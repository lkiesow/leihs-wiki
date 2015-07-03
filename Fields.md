Fields are the location where data gets stored for each item. In leihs, we make no distinction between fields that map to columns in the database or fields whose data is serialized into a text column in the database. In the user interface when editing an item, both appear to you as input fields on the item edit form.

# Fields for models
Models have a fixed number of fields describing them, and the available fields cannot be customized. They are:
- **Product:** A real-world product name. Example: Xperia Z2 Tablet
- **Version:** A version number, usually used for software products. Example: 2.5.1.
- **Manufacturer**. Example: Sony.
- **Description:** A long-form text description also shown to customers when ordering/borrowing. Example: Waterproof tablet running CyanogenMod.
- **Technical Details:** Technical description of the item, shown on the detail page when ordering. Example: 12" Screen with vibrant colors, WiFi, GPS.
- **Internal Description:** Shown only to lending and inventory managers. Useful to keep information that customers don't need to see, perhaps written in internal shorthand. Example: Chargers on shelf 2.
- **Important notes for hand over:** Notes specifically for lending managers, prominently displayed during hand over. Example: Don't forget to include Micro-USB cable.

# Fields for items
Item fields can be customized to a large extent. There are however some core fields that are guaranteed to be present:
- **Inventory Code:** A unique alphanumeric code identifying this item. Example: INV1234. This is a string.
- **Model:** The model (see above) that this item is one of. Example: Sony Xperia Z2 Tablet. What actually gets stored in the database is the `model_id`, pointing to the `id` field of the `models` table.

The rest of the fields can be customized.

## Field editor
The field editor can be reached through Admin -> Fields. Each text area there represents and configures one field on the item editor. You can add new fields with the "Create Field" button and deactivate existing fields using the "Active" checkbox on the left. Deleting fields is not yet possible as implementing a safe way to do this without discarding any data is non-trivial.

This is a typical field. Here it's the rather important `inventory_code`:

![Screenshot of a field in leihs' field editor](images/field_editor_01.png)

From left to right we have:
- **inventory_code**, the id of the field. This must be unique and it's the name you can use to refer to the field in other places, such as when making another field depend on this one. The database will automatically make sure that all names are unique, if you try to add a field with an id that is already taken, you'll get an error message.
- **Active checkbox**: This field is active, which means it shows up on each item's item editor.
- **Reordering arrow**: You can grab fields by this arrow and reorder them via drag and drop. Actually, it doesn't matter where in the grey area you click and drag, the arrow is just there to indicate that this is draggable.

## Field definitions

The following is a typical field definition, which is written in [JSON](https://en.wikipedia.org/wiki/JSON).

```json
{
	"label":"Inventory Code",
	"attribute":"inventory_code",
	"required":true,
	"permissions": { "role":"inventory_manager", "owner":true },
	"type":"text",
	"group":null
}
```

These definitions are edited in simple text areas in leihs. You may of course use an external editor to syntax highlight and format your JSON more easily. If you don't want to install any software, a simple online editor we can recommend is [JSON Editor Online](http://jsoneditoronline.org/). Click the ![Screenshot of JSON Editor Online's indent button button](images/indent_button.png) button in the top left corner of that editor to have your code indented beautifully. With the left and right arrows you can convert your JSON into a graphically editable tree on the right and back to code on the left. This can help with e.g. comma errors when listing properties. You can then copy and paste your definition back into leihs' textareas when you're done.

Using such a JSON-specific editor helps make sure that you don't introduce syntax errors or other problems. If you do create a syntax error, leihs will not let you save any fields at all until you fix it, thus preventing problems with your database.

## Available keys and values

There are the keys and values that you can use, they are described in more detail below:

Key   | Type | Description  
----- | --- | --- | ---
attribute | String | Which DB column or hash key the data is saved in
data_dependency_field_id | Field ID | What data to use for a composite field
form_name | String | ?
group | String | Group of interface elements to add this field to, or null for no grouping
label | String | The human-friendly label that appears for this field on the inventory editor
permissions | Hash | Hash of permissions and ownership restrictions
required | Boolean | If true, field must be filled in in order to save the item
target_type | String | Restrict the field to phsyical items or software licenses
type | String | The type of the field, such as string, integer, etc.
type | String | Type of the field
value_label | Array | ?
value_label_ext | Array | ?
visibility_dependency_field_id | Field ID | The field only appears if the field of the given id has the value specified by `visibility_dependency_value`
visibility_dependency_value | String | Combine with `visibility_dependency_field_id` to make a field only appear when another field has a certain value

### type

Type determines the type of the field, in the sense of datatype. Possible values are:

* **autocomplete-search**: A search field that autocompletes its selection with values from the database.
* **text**: Plain text, stored in UTF-8.
* **select**: A dropdown menu that shows values you can predefine. Requires `values` to be defined to be useful.
* **composite**: A widget that allows assigning quantities and strings to count down from a value specified in a dependent field. Slightly more complex, therefore [explained on a separate page](Composite-field-type)
* **checkbox**: A group of checkboxes with values defined in the `values` array.
* **radio**: A group of radio buttons with values defined in the `values` array.

### target_type

One of `item` to make the field only appear on edit forms for physical items or `license` to appear only on software licenses. Not setting `target_type` at all makes fields appear on both items and software licenses.


NOTE: This documentation is still incomplete but constantly being expanded. Expect a full list of all possible values in July 2015.
