## Another painful day on the exchange
**2021-11-06 / exchange,server,python,sad / Sam**

I've just started slowly adding the requisite changes to support market orders (orders with no price). Despite my best efforts to avoid making decisions too early, it seems there are quite a few pervasive assumptions that have permeated through the system.
Assumptions are still mostly controlled by the domain models although some business logic has bled in to my service layers already.
Technical debt notwithstanding, there are two main technical hurdles getting in the way of handling priceless market orders:

* I'd rushed ahead in the early days and begun building the "broker" subsystem that will go on to independently handle the balances and stock holdings for each user
* I'd abandoned development of the `MarketStall` class, which still owns the idea of remembering the `last_price` (or `reference_price` as T7 calls it)

Implementing the former so early means I've had to commit to some decisions about how an investor's balance and holdings are updated.
Currently, as all orders are limit orders we can immediately check that a user is able to finance a transaction before the order is accepted by the exchange.
This is a little more complex with market orders (but not impossible), but much of the code I have written is explicitly focussed around the `order.price` attribute of the order object.
Worse still, to appropriately handle market orders we need to have a good idea of what the current market price (ie. the `reference_price`) actually is.
Unfortunately I've been neglecting development of the `MarketStall` class which is exactly what I now need in place to have controlled access to the last price a particular stock was sold for.
The only good feeling I take away from this is that perhaps the `MarketStall` abstraction was quite a good one in the first place.
