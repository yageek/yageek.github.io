+++
title="Migrating smoothly to modern swift asynchronous APIs"
date="2022-05-21"
comments = true
published = false
[taxonomies]
tags = ["ios", "macOS", "concurrency", "nsoperation"]
categories = ["programming", "apple"]
+++

As old as I remember, I always managed concurrency on iOS/macOS before the arrival of swift with `NSOperationQueue` and
`NSOperation` subclasses. The API is not simple to understand and it's threrading behaviors changed over time but once
you understood how it was working, it was a pleasure to use to encapsulate and manage long running tasks.
 
In this post, we will the basics about those API and how you could use them to provide, with the same base code, asynchronous APIS for
Objective-C, Swift, Combine and `async` swift.

## Basics

Is see `NSOperation` as more powerfull closures: it encapsulates a piece of work whose execution is deferred for later with the 
advantage of beeing an object instance. Using dependency injection and proper abstraction, you can constitute a set of 
reusable components all accross your app.

Moreover, the native API provides the ability to chain different `NSOperation` by creating dependencies between them.
It helps a lot to mangage the concurrency properly without having to deal directly with GCD. (Behind the scene `NSOperation` uses the GCD primitives.)

**NOTE**: The corresponding types in Swift for `NSOperation` and `NSOperationQueue` are simply `Operation` and `OperationQueue`.

The API has been here from the day I started woking on  platforms. It's behavior and usage has evolved a bit [^1] but the core
idea stayed the same. 

The best to understand how one API is working, is actually to write some code.

Let's work on one simple task:

Given a sprcific range, increment one a variable from the lower to the upper bound.
After each increment, the code should sleep for some specified amount of time.

#### Create one `NSOperation`/`Operation` subclass

We will create a basic class with the different element we need.
```objective-c
#pragma mark - Interface

@interface WaitCounterOperation : NSOperation {
	NSRange _countingRange;
	NSTimeInterval _sleepTime;
}

-(instancetype) initWithRange:(NSRange) countingRange sleepTime:(NSTimeInterval) interval;
@end

#pragma mark - Implementation

@implementation WaitCounterOperation
-(instancetype) initWithRange:(NSRange) countingRange sleepTime:(NSTimeInterval) interval {
	
	self = [super init];
	if (self) {
		_countingRange = countingRange;
		_sleepTime = interval;
	}
	return self;
}

@end
```

In swift this part become less verbose:

```swift
final class WaitCounterOperation: Operation {

	// MARK: - iVar | Inputs
	private let countingRange: Range<Int>
	private let sleepTime: TimeInterval

	// MARK: - Init
	init(countingRange: Range<Int>, sleepTime: TimeInterval) {
		self.countingRange = countingRange
		self.sleepTime = sleepTime
	}
}
```

#### Encapsulate work to be done

The documentation of this class [^2] will directly speaks about asynchronous and synchronous operation. 
Let's put this question apart for now and simply talk about encapsulating the work to be done.

Every `NSOperation` has one `main` method that should contain the work to be done.

Let's implement it:

```objective-c
- (void)main {

	NSLog(@"Starting WaitCounterOperation instance: %@", self);
	for (NSInteger i = _countingRange.location; i < NSMaxRange(_countingRange); i++) {
		NSLog(@"Counter Value: %ld", i);
		[NSThread sleepForTimeInterval:_sleepTime];
	}
}
```

For swift: 

```swift
override func main() {
    print("Starting WaitCounterOperation instance: \(self)")
    for i in self.countingRange {
        print("Counter Value: \(i)")
        Thread.sleep(forTimeInterval: self.sleepTime)
    }
}
```

### Starts the work on a queue.

You can schedule the work "manually" by keeping references to the different `NSOperation` subclasses' instances and call the `main` method your self but I have to admit I never did that.

The easiest way to schedule one operation is to attach it to one `NSOperationQueue` instance.

This class has several interesting properties:

- `qualityOfService`: This is the same property you will find on GCD queues (aka. `dispatch_queue_t`/`DispatchQueue`) element and allows the developer to attribute a priority to the queue. Note that using some values could lead to some unwanted behavior [^3]. See this  guide for how to choose the appropriate mode [^4]
- `maxConcurrentOperationCount`: The maximum number of operation this queue can handle at the same time. For serial queue, set it to `1`
- `suspended`/`isSuspended`: you can ask the queue to pause/unpause.
- `underlyingQueue`: You can change the underlying GCD queue if you require specific customization. See the 2010 WWDC 210 session for more information [^5]


Let's start a queue and dispatch our task to it:

```objective-c
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
queue.qualityOfService = NSQualityOfServiceUtility;
queue.name = @"SimpleTestingQueue";
queue.maxConcurrentOperationCount = 1; // Only one task is available

WaitCounterOperation *op = [[WaitCounterOperation alloc] initWithMaxValue:10 
                                                                sleepTime:1.0f];

// Adding the operation to the queue
[queue addOperation:op]
```

In swift: 

```swift
let queue = OperationQueue()
queue.qualityOfService = .utility
queue.name = "SimpleTestingQueue"
queue.maxConcurrentOperationCount = 1; // Only one task is available

let op = WaitCounterOperation(maxValue: 10, sleepTime: 1.0)
queue.addOperation(op)
```

In both case you should see the following inside the terminal a new line every seconds:

```shell
Starting WaitCounterOperation instance: <QueueSample.WaitCounterOperation: 0x1497050e0>
Counter Value: 0
Counter Value: 1
Counter Value: 2
Counter Value: 3
Counter Value: 4
Counter Value: 5
Counter Value: 6
Counter Value: 7
Counter Value: 8
Counter Value: 9
```

It seems to work as expected.

### Splitting the work

Let's try to split the work: let's count on a larger range with two operations. The only requirement, we want to count on order.

For now we have one queue with the a value of `maxConcurrentOperationCount` of `1`. As only one operation can run at the same time,
we could simply add two operations to the queue

```objective-c
// Let's count from 0 to 19 with two operations
WaitCounterOperation *op1 = [[WaitCounterOperation alloc] initWithRange:NSMakeRange(0, 10) sleepTime:1.0];
WaitCounterOperation *op2 = [[WaitCounterOperation alloc] initWithRange:NSMakeRange(10, 10) sleepTime:1.0];

// Adding operations to the queue
[queue addOperation:op1];
[queue addOperation:op2];
```

And in Swift:

```swift
// Let's count from 0 to 19 with two operations
let op1 = WaitCounterOperation(countingRange: 0..<10, sleepTime: 1.0)
let op2 = WaitCounterOperation(countingRange: 10..<20, sleepTime: 1.0)

// Adding operations to the queue
self.queue.addOperation(op1)
self.queue.addOperation(op2)
```

Let's observe the results:

```shell
Starting WaitCounterOperation instance: <WaitCounterOperation: 0x136804f80>
Counter Value: 0
Counter Value: 1
Counter Value: 2
Counter Value: 3
Counter Value: 4
Counter Value: 5
Counter Value: 6
Counter Value: 7
Counter Value: 8
Counter Value: 9
Starting WaitCounterOperation instance: <WaitCounterOperation: 0x136805500>
Counter Value: 10
Counter Value: 11
Counter Value: 12
Counter Value: 13
Counter Value: 14
Counter Value: 15
Counter Value: 16
Counter Value: 17
Counter Value: 18
Counter Value: 19
```

Works perfectly

### Create dependencies between operations

Imagine now that we would like to start with one operation of counting from 0 to 9.
Once that's done, we would like to run two counter operations simulatneously.

We first need to allow the queue to run at least 2 operation concurrently.

```objective-c
queue.maxConcurrentOperationCount = 2;
```

In swift:

```swift
```objective-c
queue.maxConcurrentOperationCount = 2
```

Then we declare a third operation:
```
objective-c
WaitCounterOperation *op1 = [[WaitCounterOperation alloc] initWithRange:NSMakeRange(0, 10) sleepTime:1.0];
WaitCounterOperation *op2 = [[WaitCounterOperation alloc] initWithRange:NSMakeRange(10, 10) sleepTime:1.0];

// New operation counting from 50 to 100 faster
WaitCounterOperation *op3 = [[WaitCounterOperation alloc] initWithRange:NSMakeRange(50, 50) sleepTime:0.2];

[queue addOperation:op1];
[queue addOperation:op2];
[queue addOperation:op3];
```

In swift:
```swift

```

[^1]: [Concurrent Operations on Snow Leopard
](https://www.dribin.org/dave/blog/archives/2009/05/05/concurrent_operations/)
[^2]: [NSOperation documentation](https://developer.apple.com/documentation/foundation/operation#)
[^3]: [GregHeo Tweet](https://twitter.com/gregheo/status/1001501337907970048?s=21&t=xznouldW09Y9ts9Civ_n3w)
[^4]: [Energy Efficiency Guide for iOS Apps](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/EnergyGuide-iOS/PrioritizeWorkWithQoS.html)
[^5]: [Mastering Grand Central Dispatch](https://developer.apple.com/devcenter/download.action?path=/videos/wwdc_2011__hd/session_210__mastering_grand_central_dispatch.m4v)