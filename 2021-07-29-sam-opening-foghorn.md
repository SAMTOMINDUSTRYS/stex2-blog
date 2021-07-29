## Sound the opening foghorn
**2021-07-29 / exchange,server,python / Sam**

Feeling only slightly guilty, I made a start on `stex2` without Tom last week. Having spent a bit of time chatting about the possibilities with Tom the week before, I was curious as to how a stock exchange actually works in the first place. What actually happens on an exchange? Annoyingly, most literature on the topic is aimed at non-technical prospective day-traders; focussing more on the terminology and market tips. All was not lost, as I found an excellent resource: [Technical Specifications and User Guides](https://www.lseg.com/areas-expertise/technology/group-technology/technical-user-services/technical-specifications-and-user-guides) from the London Stock Exchange group themselves. These documents are heavy on the detail and will come in handy later, but really all I wanted to know was what happens to buys and sells on an exchange?

After a bit of Googling and refining my terminology, I found the question I was trying to ask was "how are buy orders **matched** to sell orders". The keyword of "matching" led me to discover that an OMS (**order matching system**) is the key part of a system handling trades, and that the **Price-Time Priority** matching algorithm is one of the most common.

I implemented a [basic procedural Python program](https://github.com/SAMTOMINDUSTRYS/stex2s-python/commit/c0dbebfd9138e85c89fb623ad16dc029a7ad86d3) to play with order matching. I won't walkthrough it here, but to highlight:

* I used `dataclass` for the first time, skipping a bunch of boring boilerplate Python for classes that hold entities
* The entrypoint for the system was the `Exchange`, which held a map (`dict`) of stock symbols to instantiated `MarketStalls` (a place to sell a particular stock symbol)
* Each `MarketStall` had an `OrderBook`, which instantiated its own `list` of buy and sell `Orders`, as well as implementing the matching algorithm
* [My first implementation of Priority-Time matching](https://github.com/SAMTOMINDUSTRYS/stex2s-python/blob/c0dbebfd9138e85c89fb623ad16dc029a7ad86d3/stexs-py/stexs.py#L92-L147) handled all four cases of basic buy fulfillment:
    * The best sell has more volume than needed for the buy (so the remaining volume is split into a new open order)
    * The buy can be exactly fulfilled by the sell (volumes match)
    * The best sell has less volume than needed for the buy (keep iterating through the best sells under there is enough volume), unless...
    * ...the buy volume cannot be reached and the buy is not fulfilled
* The `OrderBook` additionally handled the responsibility of keeping the buy and sell orders sorted appropriately for the price-time priority matcher to work (highest buys are matched to lowest sells, additionally ensuring that older orders are served first)
* The body of the program allowed me to initialise an `Exchange`, list some `Stock` and then send messages to append buys and sell `Orders` to the correct `MarketStall`'s `OrderBook`, returning any viable `Trades` conducted as a result of the new order

I had always thought of a stock exchange as a magic mystery box, but the rules that drive basic trade are quite simple. Of course, a real exchange handles a lot more complexity than what is implemented here - the LSE documents describe a whole host of order types, rules and auction types that we'll have to wrap our heads around later - but I was really surprised at how easy it was to get this working. Clearly, the hard work isn't going to be on the business logic side, but writing code that'll stand up to tens of thousands of orders a second, in a reliable way. This is going to be fun.
