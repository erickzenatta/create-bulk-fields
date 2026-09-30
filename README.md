# Zoho CRM: Create Fields and Sections from a JSON

Turn a client's field list into Zoho CRM sections and fields without building them one by one in the layout editor.

A client sends a document (Excel, Word, PDF, screenshot). A prompt converts it into a standard JSON, and a Deluge function reads that JSON and creates the sections and fields in the layout you choose.

## How it works

1. **The client sends their document** in whatever format.
2. **A prompt converts it into JSON.** Paste the document into Claude along with the prompt in `field_mapping_prompt.md`. If something isn't clear (a field type, a formula written in plain language, a field with no section), it asks instead of guessing. It also flags anything the API can't create, like subforms, and warns you if the total goes over 100 fields.
3. **A Deluge function creates everything.** It uses `PATCH /settings/layouts/{layout_id}` from the Metadata API, which creates the section and its fields directly in the right layout.

## Files

| File | What it is |
|---|---|
| `zen_createFieldsFromJson.deluge` | The standalone function |
| `field_mapping_prompt.md` | The prompt that converts a client document into the JSON |
| `example_fields.json` | A sample JSON ready to paste into the function |
| `example_client_fields.xlsx` | A sample client document to try the prompt with |

## What the function does

- Sends at most **5 fields per call**, which is the API limit.
- **Skips any field whose label already exists** in the module, so you can rerun it without creating duplicates.
- **Checks which sections already exist** in the layout. A new section is created, and an existing one gets the fields added to it.
- Creates **formulas and rollups last**, since they depend on the other fields existing first.
- Only creates **multiselect lookups** if `allow_multiselectlookup` is `true`, because each one generates a new linking module.
- **Logs every step** (config, sections, request body, API response) so you can follow what happened in the console.

## Requirements

- A standalone function named `zen_createFieldsFromJson` with a `jsonText` string argument.
- A connection named `crm` (Zoho OAuth) with the `ZohoCRM.settings.fields.ALL` and `ZohoCRM.settings.layouts.ALL` scopes, authorized by a user with permission to customize modules and fields.
- If the org isn't on the US data center, replace `zohoapis.com` with the right domain. It appears in two places in the code.

## Usage

1. Run the prompt with the client's document and get the JSON.
2. Fill in `layout_id`. Open the layout under **Setup > Customization > Modules and Fields** and copy the ID from the URL, or get it with `GET /settings/layouts?module={module_API_name}`.
3. Open `zen_createFieldsFromJson` in Functions, click **Execute**, and paste the full JSON into the `jsonText` argument.
4. Check the returned report (`OK`, `SKIP` or `ERROR`) and the log for the full API responses.

## JSON format

```json
{
  "module": "Deals",
  "layout_id": "YOUR_LAYOUT_ID",
  "allow_multiselectlookup": false,
  "sections": [
    {
      "name": "Section name",
      "fields": [
        { "field_label": "Field name", "data_type": "text", "length": 255 }
      ]
    }
  ]
}
```

Each field uses the structure from the [Zoho CRM API v8 create custom field documentation](https://www.zoho.com/crm/developer/docs/api/v8/create-custom-field.html). See `example_fields.json` for a full example.

