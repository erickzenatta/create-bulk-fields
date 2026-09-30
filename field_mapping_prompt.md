# Prompt: client document to fields JSON

Paste this into Claude along with the document the client sent (Excel, Word, PDF, screenshot, anything).

```
I'm going to give you a document where the client describes the fields they need in Zoho CRM.
Convert it into a JSON with this exact schema, with no extra text and no backticks:

{
  "module": "<Module API name, e.g. Deals>",
  "layout_id": "<Layout ID, I'll fill it in if you don't know it>",
  "allow_multiselectlookup": false,
  "sections": [
    {
      "name": "<Section name>",
      "column_count": 2,
      "fields": [ <fields in Zoho CRM API v8 format> ]
    }
  ]
}

Every field always has "field_label" and "data_type", plus the keys for its type:
- text: "length": 255
- textarea: "textarea": {"type": "small" | "large" | "rich_text"}
- email, phone, website, date, datetime, boolean: nothing extra
- integer: "length": 9
- double / currency: "length": 16, "decimal_place": 2
- percent: nothing extra
- picklist / multiselectpicklist: "pick_list_values": [{"display_value": "X", "actual_value": "X"}]
- lookup: "lookup": {"module": {"api_name": "<Module>"}, "display_label": "<Related list name>"}
- userlookup, formula, rollup_summary, autonumber, multiselectlookup: use the structure
  from the official Zoho CRM API v8 documentation for creating custom fields.

Rules:
1. Use the labels exactly as the client wrote them. Don't translate or invent any.
2. If a field type isn't clear, do NOT guess. List your questions before the JSON.
3. Formulas must use Zoho syntax (${Field}), not natural language.
   If the client didn't specify the formula, ask me.
4. If the document doesn't say which section a field belongs to, ask me.
```

## Example output

```json
{
  "module": "Deals",
  "layout_id": "4901554000000091055",
  "allow_multiselectlookup": false,
  "sections": [
    {
      "name": "Billing Info",
      "column_count": 2,
      "fields": [
        { "field_label": "Billing Method", "data_type": "picklist",
          "pick_list_values": [
            { "display_value": "Monthly", "actual_value": "Monthly" },
            { "display_value": "Annual", "actual_value": "Annual" }
          ]
        },
        { "field_label": "Contract Start", "data_type": "date" },
        { "field_label": "Retainer Amount", "data_type": "currency", "length": 16, "decimal_place": 2 },
        { "field_label": "Billing Contact", "data_type": "lookup",
          "lookup": { "module": { "api_name": "Contacts" }, "display_label": "Billed Deals" }
        }
      ]
    }
  ]
}
```
