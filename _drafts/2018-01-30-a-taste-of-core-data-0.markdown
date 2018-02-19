---

title: A taste of CoreData - Part 0
date: '2017-11-23 10:00:00 +0100'
comments: true
published: true
tags: 
    - ios
    - coredata
    - modeling
    - graph
---

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

You use it to create graphes of objects and you would eventually persist thoses graphes into memory or on the disk.

It differs from the notion of **ORM** in the sense that you should visualize a graph of connected nodes instead of a set of colums with multiple records.

{% include figure image_path="/assets/images/core_data/00/graph_vs_table.png" alt="A graph is different from a collection of tables." caption="You should see CoreData as a graph of objects, not a collection of tables." %}

The usage differ also:

-   ORM/SQLITE: involves mostly writing SQL statements that returns Record (Pure Value elements) from a sets of  tables.
-   Graph: involves looking for object instances (node) from a specific graph instance.

# CoreData as a graph framework

One graph is composed of nodes and edges. 

{% include figure image_path="/assets/images/core_data/00/graph_1.png" alt="A graph is composed of nodes and egdes." caption="A graph is composed of nodes and egdes." %}

CoreData represents this three notions by three different classes:

- Graph: `NSManagedObjectContext`
- Node: `NSManagedObject`
- Edge: `NSRelationShipDescription/NSPropertyDescription`

{% include figure image_path="/assets/images/core_data/00/graph_2.png" alt="CoreData expresses the same concepts using classes instances" caption="CoreData expresses the same concepts using classes instances" %}

## How do we create a graph?

### Graph
Now that we know the corresponding concepts within the world of CoreData, let's try to create a graph and play with it.

Creating a graph is equivalent to create an instance of the `NSManagedObjectContext` class. By taking a look at the documentation,
only the class has only [one designated initializer](https://developer.apple.com/documentation/coredata/nsmanagedobjectcontext/1506709-init)
`initWithConcurrencyType:`. 

```swift
let context = NSManagedObjectContext(concurrencyType: .mainQueueConcurrencyType)
```

Don't pay attention to the provided constant for now, we'll detail it in a future post. Just remind that after
this call, a new instance of one graph is created.

## Node

Creating a node is equivalent to create one instance of the `NSManagedObject` class. If we take a look at the [documentation](https://developer.apple.com/documentation/coredata/nsmanagedobject?language=objc), we discover that this class has two
designated initalizers, [`initWithContext:`](https://developer.apple.com/documentation/coredata/nsmanagedobject/1640602-initwithcontext?language=objc) and [`initWithEntity:insertIntoManagedObjectContext:`](https://developer.apple.com/documentation/coredata/nsmanagedobject/1506357-initwithentity?language=objc).

As we have no idea about what an `NSEntityDescription`, let's try to use the `initWithContext:` that requires 
simply a reference to a `NSManagedObjectContext` instance (aka. the graph instance).

Let's use the following code:

```swift
// Graph
let context = NSManagedObjectContext(concurrencyType: .mainQueueConcurrencyType)
// Node
let node = NSManagedObject(context: context)
```

And test is inside a playground:

{% include figure image_path="/assets/images/core_data/00/crash_code_node.png" alt="Using the initWithContext: initializer crashes" caption="Ouch! That's rude." %}

Creating the node like this does not seems to be the appropriate way because the stack crashed. If we would have read the whole
documentation of the method, we would have found this sentence:

_This method is only legal to call on subclasses of NSManagedObject that represent a single entity in the model._
{: .notice }

As for now we have no idea about why would we have subclasses of nodes, we have no other choice than continue with the 
other initializer.

The `initWithEntity:insertIntoManagedObjectContext:` method requires one instance of the `NSEntityDescription` class along with one instance 
of the `NSManagedObjectContext` class.

## Modeling the nodes and the graph

You should ask yourself why CoreData would need another piece of information than the graph instance to create our node?

Generally, the graph you're creating using CoreData matches the organization of a real concept. In this concept,
you may have differents type of nodes into the game and some specific rules on how the different nodes can or can not be combined.
Each node would hold some data we could query later.

CoreData is expecting this piece of information in the form of one instance of the `NSEntityDescription` class. 
Creating a graph without a basic set of rules would not make sense in reality and it is the same within CoreData.

Let's imagine a simple set of rules for the rest of this post. We want to create a graph representing the organization graph of
a company. We have two types of node:

-   Boss: A named person that (generally) rarely code and manages one or more people. This person has also at most one boss on top of her.
-   Employee: A named person that (generally) manages no one due to high productivity and has a speciality. This person can have multiple bosses.

From those sentences we can create a basic set of rules:
- We have two types of node `Boss` and `Employee`.
- A `Boss` node has a `name` property
- An `Employee` node has one `name` property and one `speciality` property.
- A `Boss` node has a link to several `Employee`. Let's call this link `employees`
- A `Boss` node has a link to a specific `Boss` (it's own boss). Let's call this link `boss`
- A `Boss` node has a link to the bosses under him. Let's call this link `underbosses`
- An `Employee` a link to all his own bosses. Let's call this link `bosses`

{% include figure image_path="/assets/images/core_data/00/graph_rules_description.png" alt="Description of the rules of the example graph" caption="The rules of our example graph." %}

All those rules can be splitted into two categories:

- Attribute: The data attached to the node. (`name` and `property` here)
- Relationship: The rules describing how the node can be connected together. (`boss`, `bosses`, `employees` and `underbosses`)


CoreData has exactly two classes that represent this concept: `NSAttributeDescription` and `NSRelationshipDescription` (both inherit from `NSPropertyDescription`). In fact, `NSEntityDescription` is just a container around a set of `NSPropertyDescription` subclasses.
To create the rules, we would just need to create the appropriate instances of `NSAttributeDescription` and `NSRelationshipDescription` matching
our rules.

Let's start with the different containers for our `Employee` and `Boss` nodes. Each entity is identified by her `name` property:

```swift
// Boss Description
let bossDescription = NSEntityDescription()
bossDescription.name = "Boss"

// Employee Description
let employeeDescription = NSEntityDescription()
employeeDescription.name = "Employee"
```

Then we continue with the different existing attributes:

```swift
// Attributes
let nameBossAttr = NSAttributeDescription()
nameBossAttr.name = "name"
nameBossAttr.attributeType = .stringAttributeType
nameBossAttr.defaultValue = "No Name For Boss"

let nameEmployeeAttr = NSAttributeDescription()
nameEmployeeAttr.name = "name"
nameEmployeeAttr.attributeType = .stringAttributeType

let specialityAttr = NSAttributeDescription()
specialityAttr.name = "speciality"
specialityAttr.attributeType = .stringAttributeType
```

You should notice that:
- Each attribute is identifier by its `name` property.
- Each attribute has a type (`attributeType`). You can check the [`NSAttributeType`](https://developer.apple.com/documentation/coredata/nsattributetype) enum for all the available types.
- You can provide a default value if needed.

Now we can describe the relationships:

```swift
// Relationships
let bossOneRel = NSRelationshipDescription()
bossOneRel.name = "boss"
bossOneRel.minCount = 0
bossOneRel.maxCount = 1 // To-One
bossOneRel.destinationEntity = bossDescription

let underBossesRel = NSRelationshipDescription()
underBossesRel.name = "underBosses"
underBossesRel.minCount = 0
underBossesRel.maxCount = 0 // To-Many
underBossesRel.deleteRule = .nullifyDeleteRule
underBossesRel.destinationEntity = bossDescription

bossOneRel.inverseRelationship = underBossesRel
underBossesRel.inverseRelationship = bossOneRel

let bossesMulRel = NSRelationshipDescription()
bossesMulRel.name = "bosses"
bossesMulRel.minCount = 0
bossesMulRel.maxCount = 0  // To-Many
bossesMulRel.deleteRule = .nullifyDeleteRule
bossesMulRel.destinationEntity = bossDescription

let employeeMulRel = NSRelationshipDescription()
employeeMulRel.name = "employees"
employeeMulRel.minCount = 0
employeeMulRel.maxCount = 0  // To-Many
employeeMulRel.deleteRule = .nullifyDeleteRule
employeeMulRel.destinationEntity = employeeDescription
```

Like the other, each relationship object is identified by its `name` property. You configure the relationship with the following options:
- `destinationEntity`: The name of the `NSEntityDescription` targeted by this relationship.
- `minCount`: The minimum value of nodes targeted by this relation ship.
- `maxCount`: The maximum value of nodes targeted by this relation ship. Using `0` means an infinited number.
- `deleteRule`: The policy to apply to targeted node when the node hosting the relationship is deleted.  

Now we can assign each properties to the corresponding `NSEntityDescription`:

```swift
// Assign Relationships
bossDescription.properties = [employeeMulRel, underBossesRel, bossOneRel, nameBossAttr];
employeeDescription.properties = [bossesMulRel, nameEmployeeAttr, specialityAttr];
```

If you have paid attention to the relationship's rules, you would have notice that some relationships are kind of "connected".
Getting the `bosses` property of the `Employee` node is kind of the opposite of getting the `employees` property of the `Boss` node.
By opposite I mean that if you have one `Employee` node and you get one `Boss` node through the `bosses` property, you can go back to 
the `Employee` instance by using the `employees` property of the `Boss` node.

This should be expressed in CoreData by using the `inverseRelationship` property of `NSRelationshipDescription`:

```swift
// Specifying inverse
bossesMulRel.inverseRelationship = employeeMulRel
employeeMulRel.inverseRelationship = bossesMulRel
```

Now we can try to create a graph and one `Boss` node into it:

```swift
// Create an object and insert it
let context = NSManagedObjectContext(concurrencyType: .mainQueueConcurrencyType)
let obj1 = NSManagedObject(entity: bossDescription, insertInto: context)

// Check the context status.
print("The context has now: \(context.insertedObjects.count) objects inserted.")
```
The output gives us:

```
The context has now: 1 objects inserted.
```

Great, we succeeded to create a graph instance and insert a node into it!

# Reduce the boilerplate code with `NSManagedObjectModel`

Our graph rules are basic and you would have noticed that we required a big number of lines
to describes it. Using code is one way to describes the rules. Inside the documentation of the `NSEntityDescription`,
you will find this sentence:

_You usually define entities in a Managed object model using the data modeling tool in Xcode_
{: .notice }

If you have heard about CoreData before, the *Managed Object Model* is the first tool you have been presented
to without any further information. This tool will help to create a bunch of `NSEntityDescription` with the GUI.

{% include figure image_path="/assets/images/core_data/00/managed_model_tool.png" alt="The Managed Object Model tool in Xcode" caption="The Managed Object Model tool in Xcode." %}


[^1]: [Mastering Core Data - WWDC 2010 - Session 128](http://asciiwwdc.com/2010/sessions/118)
[^2]: [Core Data: Data Storage and Management for iOS, OS X, and iCloud 2nd Edition - Marcus Zarra](https://www.amazon.com/Core-Data-Storage-Management-iCloud/dp/1937785084)
[^3]: [Core Data - Florian Kugler and Daniel Eggert](https://www.objc.io/books/core-data/)