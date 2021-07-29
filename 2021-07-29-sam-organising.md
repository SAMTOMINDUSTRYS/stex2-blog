## Organising for Architecture
**2021-07-29 / exchange,server,python / Sam**

Since my last update I had added the concept of a `Client` (user) which worked similarly to `Stocks` (make a list of them and stuff it into the `Exchange`), and wanted to see how access to the Clients and Stocks could be provided by a Repository (before attacking anything more complex like the `OrderBook`). Before that, I had to [pull apart my single script](https://github.com/SAMTOMINDUSTRYS/stex2s-python/commit/acf1031b25313478074ad7899b8a7eeef5eefdb7) and set a layout to help organise the real ARCHITECTURE that was about to happen. I settled on the following top-level modules:

* `domain` -- a sacred realm for storing domain entities and very little else (this is the centre of the Clean Architecture diagram, it should not depend on anything outside of itself), I left all the dataclasses in here with the exception of the `Exchange` class
* `entrypoints` -- the welcome mat of the application, I moved all the object instantiation and example code here to `main.py`
* `services` -- a placeholder for utilities that offer the application ways to do things, effectively a routing layer to cause the entrypoints to make something happen. This is the new home of the `Exchange` class, but eventually much more will be moved here
* `io` -- I didn't like the suggestion of using the `services` module to house Repositories and UoWs as found in the Cosmic Python examples as they didn't seem to fit my personal interpretation of a service, so I created an `io` module to organise "input and output things". I imagine message parsing and file handling will all live in here too
* `adapters` -- following the "Ports and Adapters" ethos, this code is responsible for plugging frameworks, ORMs and other such external matters into the application

The most interesting thing about this exercise was that I quite quickly realised the `Exchange` domain entity was in fact a service layer. The `Exchange` offers exchange-related functionality to the `main.py` application entrypoint. If nothing else, merely thinking about what a cleaner architecture looks like in your project tree will encourage better domain abstractions and make the responsibilities of intra-application services more obvious.
