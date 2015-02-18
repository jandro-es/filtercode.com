Title: Thread Safe Singleton pattern in Swift
Date: 2015-02-18 10:20
Category: swift
Tags: swift, patterns, multithreading, bluebird
Slug: logger-thread-safe-swift
Authors: jandro_es
Summary: Turning our Logger class, into a **Thread safe** Singleton.

Our previous iteration of the Logger class, used this code to initialise itself as a Singleton:

~~~~{.language-swift}
	// MARK: - Singleton Pattern

    class var sharedInstance: Logger {

        struct Singleton {
            static let instance = Logger()
        }

        return Singleton.instance
    }
~~~~

It works great, and in fact is one of the most common ways of creating them. But let's add some simple code to make it **thread safe**, even taking into account that we don't need a Logger to be thread safe, i think it's a good practice.

In order to accomplish it, we need to make sure that our *struct* is only instantiated once, no matter the caller thread. The best way, is to dispatch ourselves an unique thread using as name, a unique token inside the struct. The initialiser will look something like this:

~~~~{.language-swift}
	// MARK: - Thread Safe Singleton Pattern
    
    class var sharedInstance: Logger {
        
        struct StaticStruct {
            static var instance: Logger?
            static var token: dispatch_once_t = 0
        }
        
        dispatch_once(&StaticStruct.token) {
            StaticStruct.instance = Logger()
        }
        
        return StaticStruct.instance!
    }
~~~~

Now our Logger is thread safe and we don't need to worry ourselves in which thread is being called. Several more types of Singleton Initialisation Patterns can be found in this great [Github](https://github.com/hpique/SwiftSingleton).