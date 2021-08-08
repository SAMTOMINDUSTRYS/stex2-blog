## Racing toward a first viable version
**2021-08-08 / exchange,server,python / Sam**

Earlier in the week, Tom and I had a discussion about responsibilities within the application.
This came up as I was discussing how I was passing around my **Unit of Work** (UoW) to tie together the business transaction of finalising a trade and closing the orders that were fulfilled by the trade.
We talked briefly about how a `Trade` might be an **Aggregate** that provides an entrypoint to the `Order` entities that were fulfilled by it.
This led us to think about who should mark the fulfilled orders as closed. Tom suggested that perhaps a `Trade` **Aggregate** **Repository** could take care of this.
I felt it was more business behaviour and belongs over in the domain.
This had me start to wonder how much business behaviour I'd stuck in my service layer... I spent a little evening this week pulling out a couple of business rules, such as closing fulfilled orders, proposing trades and splitting up sells.
A nice artifact of pulling these things into the domain is it made it easier to unit test the actual business, and separately one-off test that the service attempts to call the domain business correctly.

Without a real aim into the weekend I finally got around to grinding out some more test code. I've managed to bring the coverage up to 80% now.
I set the repo up to [push code coverage information to codecov](https://app.codecov.io/gh/SAMTOMINDUSTRYS/stex2s-python) which neatly marks code that is hit (or not) by the test suite, helping to direct what code new tests should flex to improve coverage.
Unlike my usual experience with testing, I'm yet to have found myself backed into a corner with testing this application, which is a very nice feeling.
Keeping a separation between domain and service has helped keep the specific roles of functions clear, making them small and testable.
Additionally, as per the *Cosmic Python* book's suggestion, I can override the UoW used for a particular service function, meaning I can use my in-memory Repo/UoW utility to keep tests straightforward.

I've also been breaking out some of the bigger entities and their paraphernalia into modules of their own.
I like that Python allows for many classes and gubbins to live together in a module, while making it quite straightforward to break them apart later.
Java encourages primary classes to be in a file of their own from the off, regardless of how stubby they are.
My `Order` `dataclass` now lives in its own `domain.order` module, and has its own domain exceptions in `domain.order.order_exception`.
I've also changed the abstract base class for all the `OrderRepository` from `AbstractRepository` to `AbstractOrderRepository` which lives in `domain.order.order_repo`.
I also split out some of the `sqlalchemy` gubbins to seperate table definitions from the part of the module that instantiates the database engine and session maker.
As my final act of cleaning up, I [rewrote my generic in-memory dictionary repository](https://github.com/SAMTOMINDUSTRYS/stex2s-python/commit/90ca64fc54f74f50cd67ca5eda8627406f8fccae) to make it easier to use for multiple entities (removing a whole bunch of ugly duplicated code), which nicely [made it easier to test](https://github.com/SAMTOMINDUSTRYS/stex2s-python/commit/49ba5b5a7604c5b43bfea6f869dccd47ebf0ca1b).

With the codebase so spanking clean, it was time to ruin it with some hacky prototyping and race towards a first viable version.
Last night, I [replaced the hard-coded test messages emitted to the exchange](https://github.com/SAMTOMINDUSTRYS/stex2s-python/commit/54e0e35f6ed9fa3cd1ccc791ab98eb13e3ed539d) with a `socket` listening loop.
I wrote a [pathetically trivial broker](https://github.com/SAMTOMINDUSTRYS/stex2s-python/commit/dc0a37ea7daa5d74b4f21908ec50ae18f1144086) which connected to the exchange's localhost socket, and sent a message in the same format as before.
Lo and behold, the first true message was sent to and received by the snake-filled STEX2 exchange server.

I shared this excitement with Tom, who grounded me somehwat by sharing some documents he had found about how stock exchanges actually work (which don't necessarily match what we have gone and done).
We had a great evening ~drinking rum and~ reading about the Deutsche Börse Group's *T7 Trading Architecture*, which has [documentation](https://www.eurex.com/ex-en/support/initiatives/t7-release-9-1) that seems more clear and concise than the London Stock Exchange documents we had previously been perusing.
It was nice to confirm that some of the things we (read: Tom) had been worrying about, such as enforcing FIFO on client messages were not actually a problem (T7 only guarantees FIFO messaging ordering for high-frequency trading).
However, we became particularly annoyed with ourselves when we realised from the T7 documents that the order type we had both implemented (with a price attached) was called a **limit order**, which we had both thought was something else.
I'd also found that there are mechanisms to partially fill a buy order (if you allow orders without all-or-none rules), which was an assumption I had made in my system, requiring buy orders to be fulfilled in full before they can be removed from the order book.
Even the most basic **market order** has multiple flavours that we had not heard of (immediate or cancel, fill or kill)...
Kicking us while we were feeling down, I found a [paper about latency on stock exchanges that happens to provide a very nice overview of modern stock trading systems](https://www.bis.org/publ/work955.pdf) containing this inspiring quote:

> The continuous limit order book is at heart a simple protocol. We guess that most undergraduate computer science students could code one up after a semester or two of training.

To be fair to my self-esteem, it did only take me a few hours to get my initial version of the order book to incorrectly match some orders together.
But I fell asleep last night a bit disappointed with some of the reading we'd done, as there are a few bits of basic domain knowledge that we've got confused.

The problem with aiming for good domain driven development for a fun-times project is that investigating and mapping the domain is actually quite a lot more work (and less fun) than writing code.
Certainly if my real job was to build stock exchanges for a living, I'd be much more motivated to spend the time required to get it right, but with my precious free time I'm enjoying getting on with writing some code, trying out new patterns, and working with my view of the domain with best intentions.
This trouble has been compounded by our difficulty in finding answers, as I mentioned in a previous entry, many posts we've found to describe various market terms and processes are written for would-be investors and often provide more questions than answers.
Of course, we've also picked a project for which neither of us have any experience or any contacts.

On reflection today, despite getting some of the basics a little backward here and there, I think we've done a good job at muddling through the problem at hand and made good progress in a short time.
My aim now is to rapidly iterate towards a first version of an exchange that can send and receive messages to list buys, sells; execute trades based on price-time priority and provide quotations.
Once I have a working first version, I will have an end-to-end platform that can be tested against real-world expectations.
I'm trying to be aware of any assumptions I am making, and to defer hard decisions about how important things should work until later.
I think Tom is having trouble deferring some decisions and seems to be intent on designing things very well on the first try, whereas I'm expecting to throw bits of my system away and rewrite them with the benefit of hindsight.
I'm hoping that the design of my application will lend itself to easy heavy refactoring and swapping components, but we'll see whether Tom's tortoise approach beats my minimal effort hare in the long run.

In the mean-time as I'm so close now, I have abandoned all good practice and am racing towards a minimal viable version.
I have [added several new message types](https://github.com/SAMTOMINDUSTRYS/stex2s-python/commit/2ab84cdb8e5c06eb0a140d9b157a897135beffe4) such that the exchange can provide a list of stocks and some summary information through a socket.
More excitingly, I have written a [text driven UI application](https://github.com/SAMTOMINDUSTRYS/stex2s-python/commit/61af13434bf0df755f161586f63f0d4e47ad93df) that uses a socket to connect to the exchange, send randomly generate trades, query the state and render it all with the wonderful [`rich`](https://github.com/willmcgugan/rich) library.
It looks like this:

