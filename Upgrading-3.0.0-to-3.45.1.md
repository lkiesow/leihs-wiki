# Upgrading from < 3.14.0 to >= 3.14.0

In leihs 3.15.0, we fixed a lot of potential database inconsistency problems. An inconsistency can for example be that a contract exists for an inventory pool that has since been deleted. Because databases with such inconsistencies are no longer compatible with leihs 3.15.0, we introduced a report page in leihs 3.14.0 that lets you look at your inconsistent data and decide what to do with it. leihs cannot decide automatically, a human with good knowledge about the inventory in question is required.

So if you have a leihs database from a leihs version older than 3.14.0, you will first have to upgrade to 3.14.0 before continuing. This is easily done checking out version 3.5.0 as described above under "Performing Upgrades", then logging into your leihs 3.14.0 instance as an administrator. Navigate to "Database Check" -> "Consistency" (the direct path is `/admin/database/consistency`.

Each block in this report is labelled with the consistency problem it represents, for example "3 groups with missing inventory_pools". Looking at the report, you should be able to decide whether deleting any of these items is safe. If you want to delete an item directly from this screen, there is a big red "Delete from the database" button there for you to use.

You also see the SQL statement that discovered these inconsistencies, for example:

```sql
SELECT `groups`.* FROM `groups` LEFT JOIN inventory_pools AS t2 ON groups.inventory_pool_id = t2.id WHERE (`groups`.`inventory_pool_id` IS NOT NULL) AND `t2`.`id` IS NULL
```
You may use these statements as a basis for deleting the offending data, if you prefer to use SQL.

Once all the problems have been cleared, the report will be empty. You can now upgrade beyond version 3.14.0 to 3.15.0 or higher. 3.15.0 is the version that introduces foreign key checks so that inconsistencies like these are impossible.

# Upgrading from < 3.5.0 to >= 3.5.0

If you have a version older than 3.5.0, you will first have to upgrade to 3.5.0 in order to get the database into a state compatible with versions higher than 3.5.0. This is easily done checking out version 3.5.0 as described above under "Performing Upgrades", then checking out the version you want and upgrading to that.

Please note that to upgrade to 3.5.0, you need an older version of Rubygems since leihs 3.5.0 uses an old version of the acts-as-dag gem, whose gemspec is not compatible with newer version of Rubygems. Therefore you must first downgrade your Rubygems version:

```gem update --system 2.0.14```

Once you have upgraded to leihs 3.14.0 and beyond, you can freely move to a newer version or Rubygems as well.

# Upgrading from leihs 2.9.14 to leihs 3.0.0

This is a major upgrade that removes many dependencies from your leihs installation, but adds some others.

Since the guide is pretty long, we have covered this on a separate page: [[Upgrading-2.9.14-to-3.0.0]].
