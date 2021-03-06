## A painfully compliant day on the exchange
**2021-11-06 / exchange,server,python,sad / Sam**

I started off today slowly adding the requisite changes to support market orders (orders with no price). Despite my best efforts to avoid making decisions too early, it seems there are quite a few pervasive assumptions that have permeated through the system.
It sounds obvious in retrospect but I've discovered the hard way that you can spend a lot of time and effort keeping your software as malleable and flexible as possible but if you get the domain wrong in the first place, you're in for a bad time.
Assumptions are still mostly controlled by the domain models although some business logic has bled in to my service layers already.
Technical debt notwithstanding, two decisions I had made were getting in the way of handling priceless market orders:

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
However, as I move toward making changes that interact with my early-access broker functionality (eg. pre-screening and freezing order-associated assets) I am encountering uncaught unsupported operand `TypeError` cases that are not picked up by testing as there are no end-to-end tests using the real broker.

I'm OK with introducing changes that break some of the broker functionality for now, so I have hackily swapped the market order `None` price for the current `reference_price` when dealing with the broker system. This means we can still pre-screen and freeze assets but the numbers will probably be wrong in the end.
This change has [a few nasty side effects](https://github.com/SAMTOMINDUSTRYS/stex2s-python/issues/3) such as necessitating a `reference_price` keyword parameter on many of functions involved in handling new orders, but we can stomach this debt for now as ultimately we'll want to come back to handling the clearing of transactions in a much more complicated life-like way.
Amusingly, my first end to end test of the changes was "successful" in the sense that the exchange was perfectly happy filling `inf` price buy orders with `-inf` price sell orders. With market orders making it to the exchange now, I just needed to get the price handling right, which was another thing that had turned out to be much more complicated than expected.

I'm not proud of some of the hacks introduced today, but I will liken this to the sort of damaging work you must do to brace a ceiling in a house before smashing one of the walls down. Hacks aside, the afternoon has been fruitful and I'm excited to announce that [my latest change](https://github.com/SAMTOMINDUSTRYS/stex2s-python/commit/275aa2fd159f5ca7aff3cc8a1e8c6f33ed885fcb) means STEX2 is now compliant with the [Xetra Market Model](https://www.xetra.com/resource/blob/2685962/f818a7bba76ed64b9fc1bea2693b151d/data/T7_Market_Model-_Xetra_en.pdf)! STEX2 can now handle market and limit orders and the execution price reflects the same decision making seen on a real world exchange.

Rushing to get the new market order business logic working at the expense of code cleanliness is tolerable debt, as now the new tests are passing I have confidence as a developer to go back and refactor the new debt away and have immediate feedback as to whether my changes have broken the business logic.
