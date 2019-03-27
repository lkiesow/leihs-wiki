## Availability

* [Abstract](#user-content-abstract)
* [Definition](#user-content-definition)
* [Computation](#user-content-computation)
    * [Running reservations](#user-content-running-reservations)
    * [Changes of available quantities](#user-content-changes-of-available-quantities)
    * [General group](#user-content-general-group)
    * [Example](#user-content-example)
* [Use Cases](#user-content-use-cases)
    * [Booking calendar](#user-content-booking-calendar)
    * [Model's timeline](#user-content-models-timeline)
* [Overbooking](#user-content-overbooking)
    * [Soft overbooking](#user-content-soft-overbooking)
    * [Hard overbooking](#user-content-hard-overbooking)
* [Self-blocking](#user-content-self-blocking)
* [Factors of influence](#user-content-factors-of-influence)
* [Notes](#user-content-notes)

### Abstract

*Availability* is the core concept of the reservation system provided by leihs. It is the basis for optimal distribution of items of some particular model among customers of a particular inventory pool. It enables a customer as well as a lending manager an accurate planning of the borrow resp. lending process in short- and middle term manner.

### Definition

Given a model, an inventory pool, entitlements for groups within the inventory pool and reservations done for the model:

*The availability of the model within the inventory pool represents time-clustered distribution of available quantities of the model across the entitlement groups of the inventory pool considering their entitled quantities. These time-clusters together with their available quantities are derived from the start-/end-dates as well as the quantities of the reservations created by the people pertaining to the respective entitlement groups and some other factors.*

### Computation

https://github.com/leihs/leihs_legacy/blob/master/app/models/availability/main.rb

#### Running reservations

The starting position for the computation are the so-called **running reservations** for the model in the inventory pool. Each of these reservations must fulfill the following conditions:
* The type is an `ItemLine`.
* The status is not `rejected` or `closed`.
* The `unsubmitted` reservation is not from an expired basket (`settings.timeout_minutes`).
* The return back date of the reservation is *not overdue*. It means that the end date is not in the past and at the same time an item is assigned.
     * Case: `status = 'signed' AND item_id IS NOT NULL` is clear (item handed over, in contract and not returned back yet).
     * But what about: `status = 'approved' AND item_id IS NOT NULL` ???

They are sorted as follows:
1. `start_date ASC`
2. `end_date ASC`
3. `created_at ASC`

***This order has impact on the outcome of the computation of the available quantities across entitlement groups. This means that in case of a conflict (overbooking), reservations of users starting sooner and lasting shorter are prioritized over those created sooner (see soft overbooking example; model's timeline; unavailable/not guaranteed item line).***

#### Changes of available quantities

The main idea of the availability computation is based on so-called **changes of available quantity** (from now on referred to simply as *changes*). Such a change comprises of two parts:
1. A date on which some quantity of the model is either returned back or can be handed over. 
2. A new distribution of the available quantities among entitlement groups starting from the given date upto the succeeding one.

The first change is always today's. Changes of the past are irrelevant.

The *changes* are computed as follows:
* For every reservation in *running reservations* do:
    1. Determine the time span in which the quantity of one item of the model is unavailable for. Factors to consider:
        * Is an item already assigned?
        * Do we deal with a delayed, outstanding return of the item? If yes, what is the replacement interval? <mark>(What is it exactly??? Currently it's 1 month.)</mark>
        * Is start or end date in the past?
        * What is the maintenance period of the model?

       NOTE: This time span does not necessarily needs to be the same as `reservation.start_date` - `reservation.end_date`.

    2. Check and ensure existence of changes in respect to the lower and upper bound dates of the time span:
        1. Unless a change exists for the lower bound date, then create one.
        2. Unless a change exists for the date *just 1 day after* the lower bound date, then create one.
       
       The newly created changes get the copy of the group allocations from those preceding them.

    3. Determine the *inner changes* for the time span whose available quantities need to be updated. They are the ones, which lie between the lower and upper bound dates of the time span (`lower_bound_date <= inner_change_date < upper_bound_date`).

    4. Determine the entitlement group, whose available quantities will be subtracted from for each of the *inner changes*:
        1. Get the group candidates. They are prioritized and sorted as follows:
            1. Entitlement groups where the user of the reservation is member or; **sorted by name ascending**.
            2. General group, all users are member of.
            3. Entitlement groups where the user of the reservation is NOT member of; **sorted by name ascending**.
        2. For each of the group candidates, get the minimum of the available quantities among all the inner changes. This is the maximum possible quantity which can be taken from a group candidate for the whole time-span.
        3. Among the group candidates find the first one, whose maximum available quantity for the time-span is at least 1.
        4. If still no group candidate could be chosen, then fallback to the general group anyway.

    6. For each *inner change* do:
        1. Decrease the available quantity for the selected group candidate by 1.
        2. Assign the reservation to *running reservations* of the selected group candidate.

#### General group

*General group* is a virtual group every customer of an inventory pool is member of. Its available quantity results from model's *total available quantity* minus the sum of all entitled group quantities for the given model and inventory pool.

*Total available quantity* of a model includes items with the following properties:
* `is_borrowable = TRUE`
* `retired IS NULL`
* `parent_id IS NULL`

#### Example

```ruby
module Example
  @model = FactoryGirl.create(:model, product: 'Example Model')
  @pool = FactoryGirl.create(:inventory_pool)

  @inventory_manager = FactoryGirl.create(:inventory_manager,
                                          inventory_pool: @pool,
                                          email: 'inventory_manager@example.com')

  4.times { FactoryGirl.create(:item, model: @model, inventory_pool: @pool) }

  @group_1 = FactoryGirl.create(:group, inventory_pool: @pool, name: 'Group 1')
  FactoryGirl.create(:entitlement, model: @model, entitlement_group: @group_1, quantity: 2)

  @group_2 = FactoryGirl.create(:group, inventory_pool: @pool, name: 'Group 2')
  FactoryGirl.create(:entitlement, model: @model, entitlement_group: @group_2, quantity: 1)

  @user_A = FactoryGirl.create(:customer, inventory_pool: @pool, email: 'user_a@example.com',
                               firstname: 'User', lastname: 'A') 
  @group_1.users << @user_A
  @group_2.users << @user_A

  @user_B = FactoryGirl.create(:customer, inventory_pool: @pool, email: 'user_b@example.com',
                               firstname: 'User', lastname: 'B') 
  @group_2.users << @user_B

  @user_C = FactoryGirl.create(:customer, inventory_pool: @pool, email: 'user_c@example.com',
                               firstname: 'User', lastname: 'C') 

  # Timecop.travel('2018-06-27') # personas travel date

  @r1 = FactoryGirl.create(:reservation, model: @model, status: :approved, user: @user_A, inventory_pool: @pool,
                           start_date: Date.parse('2018-06-26'), end_date: Date.parse('2018-07-05'))
  @r2 = FactoryGirl.create(:reservation, model: @model, status: :approved, user: @user_B, inventory_pool: @pool,
                           start_date: Date.parse('2018-06-27'), end_date: Date.parse('2018-06-28'))
  @r3 = FactoryGirl.create(:reservation, model: @model, status: :approved, user: @user_C, inventory_pool: @pool,
                           start_date: Date.parse('2018-06-27'), end_date: Date.parse('2018-07-11'))
  @r4 = FactoryGirl.create(:reservation, model: @model, status: :approved, user: @user_A, inventory_pool: @pool,
                           start_date: Date.parse('2018-07-02'), end_date: Date.parse('2018-07-03'))
end

##############################################################

$ Example.instance_eval { @model.availability_in(@pool) } # =>

### binding.pry for initial changes ##########################

$ self.changes # =>

{Wed, 27 Jun 2018=>
  {"group_1"=>{:in_quantity=>2, :running_reservations=>[]},
   "group_2"=>{:in_quantity=>1, :running_reservations=>[]},
   nil=>{:in_quantity=>1, :running_reservations=>[]}}}

$ exit

### binding.pry after processing @r1 ##########################

$ self.changes.sort.to_h # =>

{Wed, 27 Jun 2018=>
  {"group_1"=>{:in_quantity=>1, :running_reservations=>["@r1 (user_A)"]},
   "group_2"=>{:in_quantity=>1, :running_reservations=>[]},
   nil=>{:in_quantity=>1, :running_reservations=>[]}},
 Fri, 06 Jul 2018=>
  {"group_1"=>{:in_quantity=>2, :running_reservations=>[]},
   "group_2"=>{:in_quantity=>1, :running_reservations=>[]},
   nil=>{:in_quantity=>1, :running_reservations=>[]}}}

$ exit

### binding.pry after processing @r2 ##########################

$ self.changes.sort.to_h # =>

{Wed, 27 Jun 2018=>
  {"group_1"=>{:in_quantity=>1, :running_reservations=>["@r1 (user_A)"]},
   "group_2"=>{:in_quantity=>0, :running_reservations=>["@r2 (user_B)"]},
   nil=>{:in_quantity=>1, :running_reservations=>[]}},
 Fri, 29 Jun 2018=>
  {"group_1"=>{:in_quantity=>1, :running_reservations=>["@r1 (user_A)"]},
   "group_2"=>{:in_quantity=>1, :running_reservations=>[]},
   nil=>{:in_quantity=>1, :running_reservations=>[]}},
 Fri, 06 Jul 2018=>
  {"group_1"=>{:in_quantity=>2, :running_reservations=>[]},
   "group_2"=>{:in_quantity=>1, :running_reservations=>[]},
   nil=>{:in_quantity=>1, :running_reservations=>[]}}}

$ exit

### binding.pry after processing @r3 ##########################

$ self.changes.sort.to_h # =>

{Wed, 27 Jun 2018=>
  {"group_1"=>{:in_quantity=>1, :running_reservations=>["@r1 (user_A)"]},
   "group_2"=>{:in_quantity=>0, :running_reservations=>["@r2 (user_B)"]},
   nil=>{:in_quantity=>0, :running_reservations=>["@r3 (user_C)"]}},
 Fri, 29 Jun 2018=>
  {"group_1"=>{:in_quantity=>1, :running_reservations=>["@r1 (user_A)"]},
   "group_2"=>{:in_quantity=>1, :running_reservations=>[]},
   nil=>{:in_quantity=>0, :running_reservations=>["@r3 (user_C)"]}},
 Fri, 06 Jul 2018=>
  {"group_1"=>{:in_quantity=>2, :running_reservations=>[]},
   "group_2"=>{:in_quantity=>1, :running_reservations=>[]},
   nil=>{:in_quantity=>0, :running_reservations=>["@r3 (user_C)"]}},
 Thu, 12 Jul 2018=>
  {"group_1"=>{:in_quantity=>2, :running_reservations=>[]},
   "group_2"=>{:in_quantity=>1, :running_reservations=>[]},
   nil=>{:in_quantity=>1, :running_reservations=>[]}}}

$ exit

### binding.pry after processing @r4 ##########################

$ self.changes.sort.to_h # =>

{Wed, 27 Jun 2018=>
  {"group_1"=>{:in_quantity=>1, :running_reservations=>["@r1 (user_A)"]},
   "group_2"=>{:in_quantity=>0, :running_reservations=>["@r2 (user_B)"]},
   nil=>{:in_quantity=>0, :running_reservations=>["@r3 (user_C)"]}},
 Fri, 29 Jun 2018=>
  {"group_1"=>{:in_quantity=>1, :running_reservations=>["@r1 (user_A)"]},
   "group_2"=>{:in_quantity=>1, :running_reservations=>[]},
   nil=>{:in_quantity=>0, :running_reservations=>["@r3 (user_C)"]}},
 Mon, 02 Jul 2018=>
  {"group_1"=>{:in_quantity=>0, :running_reservations=>["@r1 (user_A)", "@r4 (user_A)"]},
   "group_2"=>{:in_quantity=>1, :running_reservations=>[]},
   nil=>{:in_quantity=>0, :running_reservations=>["@r3 (user_C)"]}},
 Wed, 04 Jul 2018=>
  {"group_1"=>{:in_quantity=>1, :running_reservations=>["@r1 (user_A)"]},
   "group_2"=>{:in_quantity=>1, :running_reservations=>[]},
   nil=>{:in_quantity=>0, :running_reservations=>["@r3 (user_C)"]}},
 Fri, 06 Jul 2018=>
  {"group_1"=>{:in_quantity=>2, :running_reservations=>[]},
   "group_2"=>{:in_quantity=>1, :running_reservations=>[]},
   nil=>{:in_quantity=>0, :running_reservations=>["@r3 (user_C)"]}},
 Thu, 12 Jul 2018=>
  {"group_1"=>{:in_quantity=>2, :running_reservations=>[]},
   "group_2"=>{:in_quantity=>1, :running_reservations=>[]},
   nil=>{:in_quantity=>1, :running_reservations=>[]}}}
```

### Use Cases

#### Booking calendar

Given User C wants to reserve one more item of the model from 2018-06-27 to 2018-07-12, then the booking calendar looks like follows:

![Image of booking calendar of User B](https://raw.githubusercontent.com/leihs/leihs_documentation/mk/master/sources/business_logic/availability_example/user_b_booking_calendar.png)

Explanation of the time-clustered quantities:
* Wed, 27 Jun 2018 - Wed, 28 Jun 2018
    * 0 (`in_quantity` for Group 2) + 0 (`in_quantity` for Generel group `nil`) = 0
* Fri, 29 Jun 2018 - Sat, 30 Jun 2018
    * 1 (`in_quantity` for Group 2) + 0 (`in_quantity` for Generel group `nil`) = 1
* Mon, 02 Jul 2018 - Tue, 03 Jul 2018
    * 1 (`in_quantity` for Group 2) + 0 (`in_quantity` for Generel group `nil`) = 1
* Wed, 04 Jul 2018 - Thu, 05 Jul 2018
    * 1 (`in_quantity` for Group 2) + 0 (`in_quantity` for Generel group `nil`) = 1
* Wed, 04 Jul 2018 - Thu, 05 Jul 2018
    * 1 (`in_quantity` for Group 2) + 0 (`in_quantity` for Generel group `nil`) = 1
* Fri, 06 Jul 2018 - Wed, 11 Jul 2018
    * 1 (`in_quantity` for Group 2) + 0 (`in_quantity` for Generel group `nil`) = 1
* Thu, 12 Jul 2018 - ...
    * 1 (`in_quantity` for Group 2) + 1 (`in_quantity` for Generel group `nil`) = 2

#### Model's timeline

The lending and inventory managers can open the timeline for the model. This timeline is kind of a graphical representation of the availability object explained before. 

![Image of model's timeline](https://raw.githubusercontent.com/leihs/leihs_documentation/mk/master/sources/business_logic/availability_example/model_timeline.png)

One sees within it:
* *Running reservations* for the model.
* Assignments of the reservations to the particular entitlement groups.
* Available total quantities for each day.
* Available quantities of the entitlement groups for each day.
* Properties of the reservations.

### Overbooking

There is a difference between dealing with the availability data in borrow and lending sections. The availability imposes a hard constraint on the former one, but provides only orientation for the latter one. In other words, a customer can only reserve the available quantities provided by the computed values, but a lending manager can do whatever he or she likes and violate the total available quantities of the model or the entitled quantities of the entitlement groups.

#### Soft overbooking

It was described before, how the entitlement groups are prioritized and how a group candidate for the quantity allocation is determined. If there isn't sufficient quantity (at least 1) available in user's entitlement groups or the general one, then one of the other model's entitlement groups is selected, given it has the sufficient quantity available (at least 1).

```ruby
module Example
  @r5 = FactoryGirl.create(:reservation, model: @model, status: :approved, user: @user_C, inventory_pool: @pool,
                           start_date: Date.parse('2018-06-27'), end_date: Date.parse('2018-06-28'))
end

$ Example.instance_eval { @model.availability_in(@pool) } # =>

{Wed, 27 Jun 2018=>
  {"group_1"=>{:in_quantity=>0, :running_reservations=>["@r1 (user_A)", "@r5 (user_C)"]},
   "group_2"=>{:in_quantity=>0, :running_reservations=>["@r2 (user_B)"]},
   nil=>{:in_quantity=>0, :running_reservations=>["@r3 (user_C)"]}},
...
}
```

As one can see looking at the respective change date, the new reservation (`@r5`) was assigned to Group 1, which the User C is not member of.

![Image of model's timeline](https://raw.githubusercontent.com/leihs/leihs_documentation/mk/master/sources/business_logic/availability_example/soft_overbooking_timeline.png)

#### Hard overbooking

If the available quantity (`in_quantity`) for some group and time span is **negative**, then we are dealing with so-called *hard overbooking*. This happens when the total reserved quantity of the model exceeds its *total available quantity*. There are basically 2 possible reasons for that:
1. Some items of the model become excluded from its *total available quantity*, for example by being retired.
2. Lending manager creates reservations for some customer despite of non-available total quantity for some particular time span.

```ruby
module Example
  @model.items.take(2).each do |i|
    i.update_attributes! \
      retired: Date.today,
      retired_reason: Faker::Lorem.sentence
  end
end

$ Example.instance_eval { @model.availability_in(@pool) } # =>

{Wed, 27 Jun 2018=>
  {"group_1"=>{:in_quantity=>0, :running_reservations=>["@r1 (user_A)", "@r3 (user_C)"]},
   "group_2"=>{:in_quantity=>0, :running_reservations=>["@r2 (user_B)"]},
   nil=>{:in_quantity=>-1, :running_reservations=>[]}},
 Fri, 29 Jun 2018=>
  {"group_1"=>{:in_quantity=>0, :running_reservations=>["@r1 (user_A)", "@r3 (user_C)"]},
   "group_2"=>{:in_quantity=>1, :running_reservations=>[]},
   nil=>{:in_quantity=>-1, :running_reservations=>[]}},
 Mon, 02 Jul 2018=>
  {"group_1"=>{:in_quantity=>0, :running_reservations=>["@r1 (user_A)", "@r3 (user_C)"]},
   "group_2"=>{:in_quantity=>0, :running_reservations=>["@r4 (user_A)"]},
   nil=>{:in_quantity=>-1, :running_reservations=>[]}},
 Wed, 04 Jul 2018=>
  {"group_1"=>{:in_quantity=>0, :running_reservations=>["@r1 (user_A)", "@r3 (user_C)"]},
   "group_2"=>{:in_quantity=>1, :running_reservations=>[]},
   nil=>{:in_quantity=>-1, :running_reservations=>[]}},
 Fri, 06 Jul 2018=>
  {"group_1"=>{:in_quantity=>1, :running_reservations=>["@r3 (user_C)"]},
   "group_2"=>{:in_quantity=>1, :running_reservations=>[]},
   nil=>{:in_quantity=>-1, :running_reservations=>[]}},
 Thu, 12 Jul 2018=>
  {"group_1"=>{:in_quantity=>2, :running_reservations=>[]},
   "group_2"=>{:in_quantity=>1, :running_reservations=>[]},
   nil=>{:in_quantity=>-1, :running_reservations=>[]}}}
```

*Total available quantity* of the model is 2 now. Because quantity of 3 items of the model is already entitled to groups, the *general group* gets -1.

The case of 1st change (Wed, 27 Jun 2018) and 3rd change (Mon, 02 Jul 2018):
* When determining the group candidate for assigning User C's reservation, Group 1 is selected, because it has *theoretically* still quantity of 1 available.
* It is a soft overbooking, that's why the reservation is highlighted with a red border.
* The total available quantities are -1 and highlighted with red for the respective time spans.

![Image of model's timeline](https://raw.githubusercontent.com/leihs/leihs_documentation/mk/master/sources/business_logic/availability_example/hard_overbooking_timeline.png)

### Self-blocking

The *self-blocking* is a particular problem of the availability calculation dealt with in the context of the customer's booking calendar.

Given:
* The available quantity for a user in a given time span is 2.
* The user has ordered the quantity of 1 for a model in a given time span.  

Under usual circumstances the customer would see available quantity 1 for a given time span. The customer wouldn't be able to increase the quantity to 2. But according to the availability context he or she can have 2. That's why this particular reservation already created by the user (or more if quantity > 1) have to be excluded from the availability computation in order to resolve the *self-blocking* problem.

### Factors of influence

Availability computation:
* Replacement interval (?)
* Maintenance period (DB: `models.maintenance_period`)

Customer's booking calendar:
* Open dates of the inventory pool (DB: `workdays`, `holidays`)
* Reservation advance days (DB: `workdays.reservation_advance_days`)
* Maximal allowed number of visits on workdays (DB: `workdays.max_visits`)

### Notes

All the examples can be reproduced locally by:

```
cd legacy
bundle exec rspec -r ./spec/steps/availability_example_steps.rb spec/features/availability_example.feature
```