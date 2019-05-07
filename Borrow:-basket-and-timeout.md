## Timer

The timer for basket is configurable: `settings.timeout_minutes`. It gets reset all the time as long as the user performs any sort of requests in borrow section. A user has thus *theoretically* unlimited time to make and submit his or her order. The main idea is to release the reserved quantities of such an inactive users for the possible reservations of other users.

## Examples

The following examples shall illustrate the idea:

### No reservation by other user after timeout 

1. User 1 makes a reservation for a model. The timer gets reset to 30 minutes.
2. User 2 tries to reserve the same model, but it is not available to him/her (it is in the basket of user 1).
3. User 1 does not perform any activity for 1 hour. In the meanwhile, the timer reaches 0 and so the reserved quantity of the model was freed for use of other users. The browser redirects user 1 to the timeout page, which notifies him/her about the fact, that potentially the model isn't available to him/her anymore. 
4. But as nobody did a reservation for the model after its reserved quantity was released from user 1, user 1 can continue with making further reservations and the timer gets reset to 30 minutes again. The reservation for the model still in basket gets reserved for him again automatically (`reservation.updated_at = now()`).

### Reservation by other user after timeout

1. User 1 makes a reservation for a model. The timer gets reset to 30 minutes.
2. User 2 tries to reserve the same model, but it is not available to him/her (it is in the basket of user 1).
3. User 1 does not perform any activity for 1 hour. In the meanwhile, the timer reaches 0 and so the reserved quantity of the model was freed for use of other users. The browser redirects user 1 to the timeout page, which notifies him/her about the fact, that potentially the model isn't available to him/her anymore. 
4. After the timeout of user 1, user 2 spots his/her chance and makes a reservation for the model.
5. User 1 cannot continue with making any further reservations until he/she fixes the problem with what has now become an unavailable (conflicting) reservation. User 1 can't leave the timeout page due to the repeated redirects until he/she resolves the problem with the reservation.




