## Admin

Admins take decisions that affect the entire system, but they don't take part in lending or inventory management. This is instead delegated to specialized managers. Admins can:

* Create and edit users.
* Create and edit inventory pools.
* Assign inventory managers to inventory pools.
* Change an inventory pool's settings.
* Change system-wide settings.
* Perform database checks and maintenance.

Note that an admin can additionally also be inventory or lending manager in case your staff shares both roles.

## Inventory manager

Inventory managers are the people responsible for procurement/purchases, they are the ones that create and maintain the inventory inside leihs. They can:

* Change their own inventory pools' settings.
* Give users access to their inventory pools, up to "inventory manager" permission.
* Create user groups inside their inventory pools.
* Create and edit models, items, packages. They can set their items to "inventory relevant".

## Lending manager

Lending managers take care of the everyday work of handing over and taking back items and printing lending contracts (if enabled). They can also create items and models, but only those that are not inventory relevant. They can:

* Give users access to their inventory pools, up to "lending manager" permission.
* Approve or reject orders.
* Take back and hand over orders.
* Print contracts, if enabled.
* Create models and items that are not inventory relevant.

## Group manager

Group managers are responsible for orders placed by users that are in a group. They can validate (not approve) orders that contain models that are exclusive to this group. They can approve an order if it contains **only** models exclusive to this group.

* Validate orders if they contain models exclusive to their groups.
* Approve orders if they contain **nothing but** models exclusive to their groups.

They cannot perform hand overs or take backs, this is the responsibility of the lending manager.

## User

Normal users only see the lending part of the system. They can:

* See what models they have access to, browse through the model list like through an online store.
* Add models to their order.
* Submit orders.
* List their open orders, reprint contracts and value lists.