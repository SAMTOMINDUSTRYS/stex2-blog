# stex2-blog

## Trying out some Architecture: Repository and the Unit of Work collaborators
**2021-07-29 / exchange,server,python / Sam**

*Clean Architecture* talked a little about how a framework is merely a development detail and should be deferred just like any other detail in your system. On first reading I found this quite confusing and unhelpful. I understood the sentiment, but as someone who has just spent the pandemic year working on a "Django application", I couldn't see how one could possibly engineer applications and use a framwork without making a decision early in the process, and conforming to that framework.

The reason I bought *Architecture Patterns with Python* ("Cosmic Python") was for its [appendix showing how you might integrate an applocation into Django](https://www.cosmicpython.com/book/appendix_django.html). This was particularly helpful as Django is the only tool I've used to build large service-like applications, so I could see the boundaries of the example application and Django's responsibilities in much clearer terms. Seeing is believing and here was proof that a framework really can live on the "outside" of your application. Still, this begged the question: what's the point in all these lovely frameworks if you're going to write a bunch of code to keep them at bay?

*Architecture Patterns with Python* also introduced me to the **Repository** and closely related **Unit of Work** patterns.
I won't go into detail here (because every person and their dog seems to have their own personal interpretation of each pattern), but my brief interpretation at this time is:

* A **Repository** offers an interface for an application manipulate a collection of objects (eg. add, get) while hiding how and where the data is stored, effectively keeping your application ignorant of how data is persisted (eg. in memory, a database, a file)
* A **Unit of Work** (UoW) offers a context in which objects that have changed are noted, and those changes can be persisted (or discarded) as part of a transaction in your application

Since my last update I had added the concept of a `Client` (user) which worked similarly to `Stocks` (make a list of them and stuff it into the `Exchange`), and wanted to see how access to the Clients and Stocks could be provided by a Repository (before attacking anything more complex like the `OrderBook`). Before that, I had to [pull apart my single script](https://github.com/SAMTOMINDUSTRYS/stex2s-python/commit/acf1031b25313478074ad7899b8a7eeef5eefdb7) and set a layout to help organise the real ARCHITECTURE that was about to happen. I settled one a few modules:

* `domain` -- a sacred realm for storing domain entities and very little else (this is the centre of the Clean Architecture diagram, it should not depend on anything outside of itself), I left all the dataclasses in here with the exception of the `Exchange` class
* `entrypoints` -- the welcome mat of the application, new home of the `Exchange` class
* `services` -- a placeholder for utilities that will offer the application ways to do things
* `io` -- I didn't like the suggestion of using the `services` module to house Repositories and UoWs as found in the Cosmic Python examples as they didn't seem to fit my personal interpretation of a service, so I created an `io` module to organise "input and output things". I imagine message parsing and file handling will all live in here too

I figured it would be easiest to abstract how the `Stocks` are stored first as they aren't touched very often. There are plenty of (sometimes conflicting) examples of writing a Repository and UoW, and I became a little frustrated with trying to do "the right thing" first time.



The main thing that sticks from the three books I have read so far is the principle of inverting dependencies.

I've learned two things this week:
* Until now, I have failed to harness the real power of object orientated programming, instead writing procedural programs that happened to have objects
* Less code is not clean code, indeed more code seems to be the recipe to adaptable systems

The main point of this is to do things properly rather than quickly! I haven't made visible progress in terms of features this week, but the storage of Stocks, Users and Orders has been abstracted and we can move it around to anything we like with a bit of grunt code.


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


## Welcome
**2021-07-29 / misc / Sam**

Content with recently [solving bioinformatics](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-021-02395-y), I have turned my attention to something much simpler and decided to build a stock exchange. This is actually an idea Tom and I had a few years ago while discussing interesting ideas for student group projects. Like many fun things during my PhD, [it was left merely as an idea](https://github.com/SAMTOMINDUSTRYS/stex).

In a rare week off a few months ago, I felt inspired to adventure on some personal improvement and bought a bunch of books.
I've been eager to apply the things I have been reading about from *Clean Architecture*, *Architecture Patterns with Python* (aka. "Cosmic Python") and most recently, the "big boy"; *Designing Data-Intensive Applications*.

Turns out our five year old idea for establishing the *Sam and Tom Stock Exchange* seemed a good fit. We're likely going to need some real domain modelling and research (because neither of us are users of such a system and not at all familiar with finance technology) and we think the idea will lend well to a service-orientated architecture, which is something I would like to try out. Most interestingly, by definition a stock exchange is a data intensive system, and will throw up some challenges for ensuring transactions are robust, services can scale, and are resilient to whatever we decide to throw at it.

Tom and I thought it would be good to kick off a development log. We'll see if that turns out to be true.
