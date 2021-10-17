## Long time no sale
**2021-10-17 / exchange,server,python / Sam**

Work has been [pretty busy](https://twitter.com/samstudio8/status/1438834677893287937) and for a variety of other reasons neither Tom or I have had much time for fun and frivolity on the stock exchange.
I'd meant to write an update in September as I was very pleased to have assembled a test suite with just shy of 95% test coverage of the STEX2 Python Exchange!
Having always struggled to work out which tests to write I have found that like most things, practice makes perfect.
Writing tests alongside code (or even before writing code if you're mentally able) is yielding better software. It's forcing me to write smaller functions that do their job well, and abstractions that are simple to mock for tests.
More excitingly, these skills are bleeding into my real work already.

My testing epiphany aside, I wanted to generally update our avid reader on the state of play.
A little while after my last post two months ago, Tom and I found what is quite possibly the most useful and concise document yet: [T7's Market Model for the Trading Venue Xetra](https://www.xetra.com/resource/blob/2685962/f818a7bba76ed64b9fc1bea2693b151d/data/T7_Market_Model-_Xetra_en.pdf).
It's from the same bundle of documentation I'd touted before (about the [T7 trading platform](https://www.xetra.com/xetra-en/technology/t7)).
Of particular interest is Section 11 ("Illustration of Price Determination Processes") which describes *with examples* how the matcher works under a variety of scenarios -- finally answering the question of how the execution price for an order is determined!
These examples were ripe for forming the basis of a test suite to determine whether the Python and Java based matchers were working correctly.
However, it wasn't until attempting to implement these tests in the Python Exchange, I discovered the extent of the misunderstanding between market and limit orders I mentioned in my last post.

Tom and I have both implemented an order book and matcher that handles limit orders exclusively (that is, any new order must have a price).
We'll need to factor in the possibility of a `null` price to allow for orders to be handled at the "market price".
Armed with the new T7 document we'll be able to start making these changes but the activation energy required explains part of the reason it's taken so long to write a new blog post, let alone new code!

My current implementation of the Exchange has an old class that was being phased out (called the `MarketStall`) which still holds the market reference price (the last traded price) for a given symbol.
Amusingly, my first version had the orderbook as a member of this `MarketStall` class, but I split it out into its own entity as Tom and I were under the impression the matcher just took orders and returned trades without needing to know any additional state.
I'm undecided as to whether the standalone matcher's `OrderBook` and the old `MarketStall` should be reunited, or whether the last traded price needs to be given to the matcher to decide execution prices.
I'm not even sure if the matcher should be decising on the execution price or just proposing order matches to be executed at a price determined by another process later!

I think my first goal will be to retrofit the correct execution pricing into the existing matcher with as little damage as possible, that will allow my exchange to pass the new T7 test suite for the few cases that only consider limit orders.
Even if the matcher is not the right place to determine the sale price, it is where it is currently decided, we can always move it later.
Once that works, I'll have to implement market orders (null-price orders). I don't think this will actually be too much trouble, as my exchange isn't particularly heavy on validation.
The only issue will come when trying to sort the orderbook and we'll just need to ensure that `None` price orders appear on top. We'll still need to keep track of the largest bid and smallest ask (regardless of market orders) as those are used in determining execution price.

I'm optimistic that I can still salvage my implementation but the outlook doesn't look so good for Tom's Java based exchange.
I had a fun discussion with Tom yesterday where we discussed the pitfalls of both our implementations.
My exchange has been subject to feature creep (incorporating the broker, rushing ahead with sockets and messages) whereas Tom's has been subject to overengineering (the matcher has been rigidly designed to support the use cases that we have later discovered to be wrong).
Tom anticipates having to start the matcher class again, whereas I'm confident there is still enough elasticity in my design to fit these new aspects in.

With a large milestone out of the way in the real world, I'm hoping for a bit of time this coming week or two to try and get my system to pass the new T7 suite in full.
