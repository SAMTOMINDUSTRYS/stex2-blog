## Another painful day on the exchange
**2021-11-06 / exchange,server,python,sad / Sam**

I've just started slowly adding the requisite changes to support market orders (orders with no price). Despite my best efforts to avoid making decisions too early, it seems there are quite a few pervasive assumptions that have permeated through the system.
Assumptions are still mostly controlled by the domain models although some business logic has bled in to my service layers already.
Technical debt notwithstanding, there are two main technical hurdles getting in the way of handling priceless market orders:

* I'd rushed ahead in the early days and begun building the "broker" subsystem that will go on to independently handle the balances and stock holdings for each user
* I'd abandoned development of the `MarketStall` class, which still owns the idea of remembering the `last_price` (or `reference_price` as T7 calls it)

Implementing the former so early means I've had to commit to some decisions about how an investor's balance and holdings are updated.
Currently, as all orders are limit orders we can immediately check that a user is able to finance a transaction before the order is accepted by the exchange.
We can also "freeze" the particular assets that a user is trying to trade. For example, if a user wants to buy £100 of stock then we can debit and hold £100 from their account until the order is fulfilled or cancelled.
This becomes a little more complex with market orders (but not impossible), but much of the code I have written is explicitly focussed around the `order.price` attribute of the order object.
Worse still, to appropriately handle market orders we need to have a good idea of what the current market price (ie. the `reference_price`) actually is.
Unfortunately I've been neglecting development of the `MarketStall` class which is exactly what I now need in place to have controlled access to the last price a particular stock was sold for.
The only good feeling I take away from this is that perhaps the `MarketStall` abstraction was quite a good one in the first place and I shouldn't have tried to ditch it.

The inertia for these changes has been quite high and really drained some of the initial excitement that Tom and I had to get started, but I'm keen to continue.
To my surprise, many of the initial changes have been quite simple and my test coverage provides good confidence that things are indeed working.
However, as I move toward making changes that interact with my early-access broker functionality (eg. pre-screening and freezing order-associated assets) I am encountering uncaught unsupported operand `TypeError` cases that are not picked up by testing (as there are no end-to-end tests using the real broker).

For now, I have hackily swapped the market order `None` price for the current `reference_price` when dealing with the broker system.
This has the nasty side effect of adding a `reference_price` keyword parameter to many of functions involved in handling new orders, but we can stomach this for now.
Ultimately, I'm sure we'll want to come back to handling the clearing of transactions in a much more complicated way.
