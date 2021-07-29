# stex2-blog

## Sound the opening foghorn
**2021-07-29 / exchange,server,python / Sam**

Feeling only slightly guilty, I made a start on `stex2` without Tom last week. Having spent a bit of time chatting about the possibilities with Tom the week before, I was curious as to how a stock exchange actually works in the first place. What actually happens on an exchange? Annoyingly, most literature on the topic is aimed at non-technical prospective day-traders; focussing more on the terminology and market tips. All was not lost, as I found an excellent resource: [Technical Specifications and User Guides](https://www.lseg.com/areas-expertise/technology/group-technology/technical-user-services/technical-specifications-and-user-guides) from the London Stock Exchange group themselves. These documents are heavy on the detail and will come in handy later, but really all I wanted to know was what happens to buys and sells on an exchange?

After a bit of Googling and refining my terminology, I found the question I was trying to ask was "how are buy orders **matched** to sell orders". The keyword of "matching" led me to discover that an OMS (**order matching system**) is the key part of a system handling trades, and that the **Price-Time Priority** matching algorithm is one of the most common.

I implemented a [basic procedural Python program](https://github.com/SAMTOMINDUSTRYS/stex2s-python/commit/c0dbebfd9138e85c89fb623ad16dc029a7ad86d3) to play with order matching. I won't walkthrough it here, but to highlight:

* I used `dataclass` for the first time, skipping a bunch of boring boilerplate Python for classes that hold 



## Welcome
**2021-07-29 / misc / Sam**

Content with recently [solving bioinformatics](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-021-02395-y), I have turned my attention to something much simpler and decided to build a stock exchange. This is actually an idea Tom and I had a few years ago while discussing interesting ideas for student group projects. Like many fun things during my PhD, [it was left merely as an idea](https://github.com/SAMTOMINDUSTRYS/stex).

In a rare week off a few months ago, I felt inspired to adventure on some personal improvement and bought a bunch of books.
I've been eager to apply the things I have been reading about from *Clean Architecture*, *Architecture Patterns with Python* (aka. "Cosmic Python") and most recently, the "big boy"; *Designing Data-Intensive Applications*.

Turns out our five year old idea for establishing the *Sam and Tom Stock Exchange* seemed a good fit. It was likely going to need some real domain modelling and research (because neither of us are users of such a system and not at all familiar with finance technology) and it would likely lend well to a service-orientated architecture, which is something I would like to try out. Most interestingly, by definition a stock exchange is a data intensive system, and would throw up some challenges for ensuring transactions are robust, services can scale, and are resilient to whatever we decide to throw at it.

Tom and I thought it would be good to kick off a development log. We'll see if that turns out to be true.
