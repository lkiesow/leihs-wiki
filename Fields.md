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
- **inventory_code**, the id of the field. This must be unique and it's the name you can use to refer to the field in other places, such as making another field depend on this one. The database will automatically make sure that all names are unique, if you try to add a field with an id that is already taken, you'll get an error message.
- **Active checkbox**: This field is active, which means it shows up on each item's item editor.
- **Reordering arrow**: You can grab fields by this arrow and reorder them via drag and drop. Actually, it doesn't matter where in the grey area you click and drag, the arrow is just there to indicate that this is draggable.

## Field definitions

Pictured next is the field definition itself, which is written in [JSON](https://en.wikipedia.org/wiki/JSON).

There are a couple of keys and values that you can use, they are described in more detail below:

Key   | Type | Description  
----- | --- | --- | ---
label | String | The human-friendly label that appears for this field on the inventory editor
attribute | String | Which DB column or hash key the data is saved in
required | Boolean | If true, field must be filled in in order to save the item
permissions | Hash | Hash of permissions and ownership restrictions
type | String | The type of the field, such as string, integer, etc.
group | String | Group of interface elements to add this field to, or null for no grouping

NOTE: This documentation is still incomplete but constantly being expanded. Expect a full list of all possible values in July 2015.
