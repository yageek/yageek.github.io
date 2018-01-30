---

title: A taste of CoreData - Part 1
date: '2017-11-23 10:00:00 +0100'
comments: true
published: false
tags: 
    - ios
    - coredata
    - modeling
    - graph
---

# Insights

Before 2011, I was mainly working on C++/Qt for Gnu/Linux operating systems. I really enjoyed the documentation set and the clear tutorial
the development teams provided along with the library. I guess that still today, you can find a lot of answers by simply reading the documentation or some code samples.

When I dived into the Apple world, I was glad to see that plenty of good resources was provided for the main frameworks. You usually takes the same path
when you are using a framework for the first time: reading the programmming guide, reading the the main classes documentation and may be watching
some WWDC introducton videos. Most of the times this was enough to remove the magic and to imagine the machinary behind.
I would say that this is mainly true for all the importants pieces except for CoreData. It is the lonely framework that I heard people avoid to use
because they see it as too complex and badly documented.

Poorly documented may be too harsh. I would say that information about CoreData is widespread in a lot of differents parts and it takes time to gather all of them.
I would even say, that some information are "hidden" only inside some WWDC videos about CoreData, especially regarding the concurrency and the potential stack configurations [^1].
Hopefully, different books have been published (I would recommend those [^2] and [^3]) on the subject that help you to find the missing pieces inside the puzzle.

I would like to start a series of post introducing CoreData in a different way, starting more from the code documentation and trying to provide some insights about
how the framework works internally (or at least provide an acceptable abstraction of the concepts.)

All the code related to the series will be pusblished on this github repository: [https://github.com/yageek/a-taste-of-core-data](https://github.com/yageek/a-taste-of-core-data).

# What is CoreData?

Describing CoreData is not easy as you may thought. A lot of developers I met simply describes Core Data as an **Object Relational Mapper (ORM)** library around SQLite.
You could agree with this vision but doing so you will not see the real nature of the framework. 

According to me, the best description has been provided by [Matt Thompson](https://twitter.com/@mattt) in his [post](http://nshipster.com/core-data-libraries-and-utilities/) on
[nshispter.com](https://nshispter.com):

_Contrary to popular belief, **Core Data is not an Object-Relational Mapper**, but **rather an object graph and persistence framework**, capable of much more than the Active Record pattern alone is capable of. **Using Core Data as an ORM** necessarily limits the capabilities of Core Data and muddies its conceptual purity. But for many developers longing for the familiarity of an ORM, this trade-off is a deal at twice the price!_  - Matt Thomson
{: .notice }

The key elements here are **object graph** and **persistence**. 

You use it create graphes of objects and you would eventueally persist thoses graphes into memory or on the disk.


[^1]: [Mastering Core Data - WWDC 2010 - Session 128](http://asciiwwdc.com/2010/sessions/118)
[^2]: [Core Data: Data Storage and Management for iOS, OS X, and iCloud 2nd Edition - Marcus Zarra](https://www.amazon.com/Core-Data-Storage-Management-iCloud/dp/1937785084)
[^3]: [Core Data - Florian Kugler and Daniel Eggert](https://www.objc.io/books/core-data/)