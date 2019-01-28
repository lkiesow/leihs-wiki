# Admin

Admins take decisions that affect the entire system, but they don't take part
in lending or inventory management. This is instead delegated to specialized
managers. Since version 5 the previous admin admin role hase been split into
_leihs-Admins_ and _System-Admins_. This separation is an ongoing process and
will be further refined in the future.


## Leihs-Admins

* Create and edit users.
* Create and edit groups.
* Associate users to groups.
* Associate users (or groups) to `authentication_systems`.
* Create and edit inventory pools.
* Assign inventory managers to inventory pools.
* Change an inventory pool's settings.
* Create procurement admin users
* Change system-wide settings.

## System-Admins

* Manage `authentication_systems`.
* Perform database management tasks such as clean-up. 


# Inventory manager

Inventory managers are the people responsible for procurement/purchases, they are the ones that create and maintain the inventory inside leihs. They can:

* Change their own inventory pools' settings.
* Give users access to their inventory pools, up to "inventory manager" permission.
* Create user groups inside their inventory pools.
* Create and edit models, items, packages. They can set their items to "inventory relevant".

# Lending manager

Lending managers take care of the everyday work of handing over and taking back items and printing lending contracts (if enabled). They can also create items and models, but only those that are not inventory relevant. They can:

* Give users access to their inventory pools, up to "lending manager" permission.
* Approve or reject orders.
* Take back and hand over orders.
* Print contracts, if enabled.
* Create models and items that are not inventory relevant.

# Group manager

Group managers are responsible for orders placed by users that are in a group. They can validate (not approve) orders that contain models that are exclusive to this group. They can approve an order if it contains **only** models exclusive to this group.

* Validate orders if they contain models exclusive to their groups.
* Approve orders if they contain **nothing but** models exclusive to their groups.

They cannot perform hand overs or take backs, this is the responsibility of the lending manager.

# User

Normal users only see the lending part of the system. They can:

* See what models they have access to, browse through the model list like through an online store.
* Add models to their order.
* Submit orders.
* List their open orders, reprint contracts and value lists.

In addition to the above roles a user may additionally have access to the procurement module as one of the following roles:

## Roles in Procurement Module

### Procurement Admin

Procurement Admins are responsible for setting up the procurement module. They:

* Create budget periods
* Create requesters
* Create categories and authorize the respective inspectors

### Procurement Inspector

Procurement Inspectors approve requests pertaining to the categories they are assigned to. They:

* Review and approve procurement requests
* Modify certain attributes of the procurement requests
* Organize requests in respect to the different categories
* Create requests in the name of particular requesters

### Procurement Viewer

Procurement Viewer simply review (read-only) requests pertaining to the categories they are assigned to. They:

### Procurement Requester

Procurement Requesters create requests for particular categories and their organizational units. They

* Create requests
