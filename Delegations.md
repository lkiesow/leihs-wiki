## Properties

A delegation is a special type of user, so-called *impersonal* user. It is basically a group of normal users. Examples are: study course, exposition, specific space, etc.

A delegation shares many properties with a normal user like:
* Access rights for inventory pools
* Reservations
* Orders
* Contracts
* Etc.

These are the most important specifics:
* **Responsible user** 
  * Every delegation must have a responsible user.
  * It is a normal user acting as a kind of last liable instance.
  * This user is always also a member of the delegation him- or herself.
* **Members** 
  * Arbitrary amount of normal users.
  * Any of the members can place an order, pick up and return stuff in the name of a delegation. They can be different from each other.
* **`delegated_user_id` property of a reservation**
  * Every reservation created in the name of a delegation carries a special property of `delegated_user_id` which stores the information about which member of a delegation has created this or that reservation.

## Example

Given is:
* A delegation "D" with 4 members: "M1", "M2", "M3" and "M4".
* Member "M4" as the responsible user of the delegation "D".
* Delegation "D" is customer of inventory pool "IP".
* Lending manager "LM" for inventory pool "IP".

Steps:
1. Member "M1" switches to delegation "D" in the borrow section, creates couple of reservations and places an order in the name of the delegation "D". All reservations of the order have "M1" as the `delegated_user_id`.
2. Lending manager edits the order and adds another reservation to it. This new reservation gets "M1" as its `delegated_user_id` as well.
3. Lending manager approves the order.
4. Lending manager adds yet another reservation to the hand over of the delegation "D". This reservation gets the delegation's responsible user "M4" as its `delegated_user_id`.

   NOTE: This reservation does not belong to any order of the delegation "D" that's why the responsible user is chosen as the creator of the reservation in the name of the delegation "D".
5. Member "M2" picks up the stuff in the name of the delegation "D". He or she signs the contract. The user's name on the contract is that of the delegation "D", but user "M2" is noted as the *contact person* for the contract.
6. Member "M3" returns the stuff in the name of the delegation "D".

## Implementation details

* Delegations are stored in the `users` table, where `users.delegator_user_id IS NOT NULL` (responsible user).
* Delegation members are stored in the `delegations_users` table.
* `reservations.delegated_user_id` carries the information about a specific member of the delegation as its creator.
