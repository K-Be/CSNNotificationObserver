# CSNNotificationObserver

[![CocoaPods](http://img.shields.io/cocoapods/v/CSNNotificationObserver.svg)](http://cocoadocs.org/docsets/CSNNotificationObserver/)
![](http://img.shields.io/badge/license-MIT-green.svg)

## Overview

Simple manage NotificationObserver inspired by [cockscomb](https://github.com/cockscomb/CXCKeyValueObserver). you won't forget to remove notification.

## Requirements

* iOS 6 or Later
* ARC

## Usage

### Traditional implementation for selector style

```objc
@implementation SampleViewController

- (void)viewDidLoad
{
    [super viewDidLoad];
    
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(receiveDidEnterBackgroundNotification:) name:UIApplicationDidEnterBackgroundNotification object:nil];
}

- (void)receiveDidEnterBackgroundNotification:(NSNotification *)notification
{
    // do something
}

- (void)dealloc
{
    // You have to remove observer, but sometimes forget about it.
    [[NSNotificationCenter defaultCenter] removeObserver:self];
}

@end
```

### New implementation for selector style

```objc
@implementation SampleViewController {
    CSNNotificationObserver *_observer;
}

- (void)viewDidLoad
{
    [super viewDidLoad];
    _observer = [[CSNNotificationObserver alloc] initWithObserver:self selector:@selector(receiveDidEnterBackgroundNotification:) name:UIApplicationDidEnterBackgroundNotification object:nil];
}

- (void)receiveDidEnterBackgroundNotification:(NSNotification *)notification
{
    // do something
}

- (void)dealloc
{
    // You don't have to worry about to remove observer
}

@end
```

### Traditional implementation for blocks style

```objc
@implementation SampleViewController {
    id _didEnterBackgroundNotificationObserver;
    id _willEnterForegroundNotificationObserver;
}

- (void)viewDidLoad
{
    [super viewDidLoad];
    _didEnterBackgroundNotificationObserver = [[NSNotificationCenter defaultCenter] addObserverForName:UIApplicationDidEnterBackgroundNotification object:nil queue:[NSOperationQueue mainQueue] usingBlock:^(NSNotification *notification) {
        // do something
    }];
    
    _willEnterForegroundNotificationObserver = [[NSNotificationCenter defaultCenter] addObserverForName:UIApplicationWillEnterForegroundNotification object:nil queue:[NSOperationQueue mainQueue] usingBlock:^(NSNotification *notification) {
        // do something
    }];
}

- (void)dealloc
{
    // You have to remove observer, but sometimes forget about it.
    [[NSNotificationCenter defaultCenter] removeObserver:_didEnterBackgroundNotificationObserver];
    [[NSNotificationCenter defaultCenter] removeObserver:_willEnterForegroundNotificationObserver];
}

@end

```

### New implementation for blocks style

```objc
@implementation SampleViewController {
    CSNNotificationObserver *_observer;
}

- (void)viewDidLoad
{
    [super viewDidLoad];
    _observer = [[CSNNotificationObserver alloc] initWithName:UIApplicationDidEnterBackgroundNotification object:nil queue:[NSOperationQueue mainQueue] usingBlock:^(NSNotification *notification) {
        // do something
    }];
    
    [_observer addObserverForName:UIApplicationWillEnterForegroundNotification object:nil queue:[NSOperationQueue mainQueue] usingBlock:^(NSNotification *notification) {
        // do something
    }];
}

- (void)dealloc
{
    // You don't have to worry about to remove observer
}

@end
```

Especially useful when you choose block based notification observing. You don't need to have ivars, you need just 1 ivar, and it's automatically removed as observer when deallocated.

## Caution

following sample code does not work that you expect.

```objc
/// this method is called more than twice.
- (void)addObserver
{
    self.notificationObserver = [[CSNNotificationObserver alloc] initWithObserver:self selector:@selector(respondsNotification:) name:@"SomeNotificationName" object:nil];
}
```

Reason

1. `initWithObserver:selector:name:object:` add observer to NSNotificationCenter
2. returns new instance of `CSNNotificationObserver`
3. `_notificationObserver` instance variable deallocate.
4. remove observer from NSNotification.
5. new instance of `CSNNotificationObserver` will be assign to `_notificationObserver`

this is by design.


### Workaround

#### 1. Deallocate exist instance first.

```objc
/// this method is called more than twice.
- (void)addObserver
{
    self.notificationObserver = nil;
    self.notificationObserver = [[CSNNotificationObserver alloc] initWithObserver:self selector:@selector(respondsNotification:) name:@"SomeNotificationName" object:nil];
}
```

#### 2. Separate initialize and adding observer.

```objc
/// this method is called more than twice.
- (void)addObserver
{
    self.notificationObserver = [[CSNNotificationObserver alloc] init];
    [self.notificationObserver addObserver:self selector:@selector(respondsNotification:) name:@"SomeNotificationName" object:nil];
}
```

#### 3. Use block style

```objc
/// this method is called more than twice.
- (void)addObserver
{
    self.notificationObserver = [[CSNNotificationObserver alloc] initWithName:@"SomeNotificationName" object:nil queue:[NSOperationQueue mainQueue] usingBlock:^(NSNotification *notification) {
        [self respondsNotification:notification];
    }];
}
```

## Install

Use CocoaPods,

```ruby
pod 'CSNNotificationObserver', '~> 0.9.2'
```


## License

The MIT License (MIT)

Copyright (c) 2014 griffin-stewie

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
the Software, and to permit persons to whom the Software is furnished to do so,
subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
