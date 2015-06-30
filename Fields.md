Fields are the location where data gets stored for each item. In leihs, we make no distinction between fields that map to columns in the database or fields whose data is serialized into a text column in the database. In the user interface when editing an item, both appear to you as input fields.

## Fields for models

Models have a fixed number of fields describing them, and the available fields cannot be customized. They are:

* **Product:** A real-world product name. Example: Xperia Z2 Tablet
* **Version:** A version number, usually used for software products. Example: 2.5.1.
* **Manufacturer**. Example: Sony.
* **Description:** A long-form text description also shown to customers when ordering/borrowing. Example: Waterproof tablet running CyanogenMod.
* **Technical Details:** Technical description of the item, shown on the detail page when ordering. Example: 12" Screen with vibrant colors, WiFi, GPS.
* **Internal Description:** Shown only to lending and inventory managers. Useful to keep information that customers don't need to see, perhaps written in internal shorthand. Example: Chargers on shelf 2.
* **Important notes for hand over:** Notes specifically for lending managers, prominently displayed during hand over. Example: Don't forget to include Micro-USB cable.

## Fields for items

Item fields can be customized to a large extent. There are however some core fields that are guaranteed to be present:

* Inventory Code: A unique alphanumeric code identifying this item. Example: INV1234.
* Model: The model (see above) that this item is one of. Example: Sony Xperia Z2 Tablet.

The rest of the fields can be customized.

TODO: This documentation is incomplete. If you want to add to it, go right ahead.
