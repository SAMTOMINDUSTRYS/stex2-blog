## Hello Twitter
**2022-03-26 / hello / Sam**

I just told Twitter that we are building a stock exchange so I thought I better update the blog. If you are reading this, thank you for your continued interest in Sam and Tom Industrys produce.
We are looking for capital investment in our brand new trading platform and take any kind of capital from A to Z. We also take cheque, postal order, cash and gift cards. Please no AMEX.

It has been a little while since we last made progress... The holiday season, and both Tom and I quitting our real jobs has got in the way somewhat.
We have had an Extraordinary General Ramble today regarding the state of STEX. The headline decision is Tom has graciously admitted defeat and accepts the Python implementation of STEX is superior because it actually exists.
Tom has however, not ruled out the possibility of the Java implementation launching a counter offensive in the next 3 to 5 years, when his daughter goes to school.

In light of this, we'll likely revisit the design for the Python exchange and client to make a design in the first place.
[I left off at the tail of last year](2021-11-06-a-painfully-compliant-day-on-the-exchange.md) by adding a battery of unit tests to check that the exchange was "T7 compliant" (i.e. that it worked the same way that basic trades are carried out on the [T7 system model](https://www.eurex.com/ex-en/support/technology/t7)). Next up is polishing what we have and taking care of sharp edges and foot guns (e.g. allowing negative orders). The API is still early days and we need to have a good think about how we're going to tie up the process of matching trades to delivering timely information to clients. Sending orders and receiving information about trades is somewhat handled async currently, but querying the status of an orderbook adds a bunch of strain that should really not exist. The solution is likely separating the process of matching and administering the order book completely with independent components that exchange messages. It's going to be interesting to join forces and work collaboratively on the same piece of software -- something that is quite alien to both of us.

We are still quite far from our goal of launching this as a fun web4 game and there is still a lot to do just to get the basics working, but I've had fun connecting our [rich based](https://github.com/Textualize/rich) terminal UI to the STEX exchange. I've been looking to port the TUI to the cutting edge [Textualise project](https://github.com/Textualize/textual) but want to wait a little longer for some of the bigger API changes to land first.

My other frivolous project has been to connect the existing STEX TUI to my new [Adafruit Macropad](https://www.adafruit.com/product/5128).
If you're interested in that [you can follow the Twitter thread](https://twitter.com/samstudio8/status/1507714882598326274) for more details!
I've now got the Macropad sending hotkeys (using the excellent [macropad-hotkeys](https://learn.adafruit.com/macropad-hotkeys) library) to the TUI to control some of the basic aspects of the exchange client and I use `pyserial` to send information about the exchange back to the Macropad to render on the little OLED screen! It's very, very cute. While I am enjoying my brief period of unemployment, I'll be tinkering some more with this before tackling any real outstanding issues with STEX.
