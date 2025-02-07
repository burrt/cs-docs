# C# Tips

## Empty async delegate

Useful for unit testing for mocks or even for method signatures that require a delegate.

```cs
(x, y) => {}
```

## Some Interview Questions

> **What is the difference between an `abstract` class and an `interface`?**

* Concrete classes can implement multiple interfaces but only a single abstract class
* Abstract classes can have virtual methods that implement some functionality, interfaces cannot contain any implementation
* Semantically different; an interface is a contract an implementing class will satisfy whilst an abstract class is usually for providing and/or defining a base set of functionality

> **What is the difference between `abstract` and `virtual`?**

* Virtual methods may or may not have an implementation
* Abstract methods must be empty statements

> **Is class overloading and overriding at compile time and/or run-time?**

Class *overloading* is at (static) compile time as the compiler is aware of what method is invoked and what parameter types are set up.

Class *overriding* is at run-time and is an example of dynamic polymorphism. An example is when a method has an interface object as a parameter and the IoC container will resolve the implementing class at run-time. Thus the implementing method's behavior could change.

> **What is `IQueryable` used for?**

It provides functionalities to evaluate queries against a specific data source where the type of the data is known. It is intended for implementation by query providers and it also implements `IEnumerable` - thus enumeration of the expression tree is lazily executed.

> **If two objects are the same will they have the same hashcode?**

In C#, the default implementation of the `GetHashCode` method, inherited from the Object class, computes hash codes based on the object's memory address. However, this default behavior is not suitable for most classes, especially when you need to use instances of those classes as keys in hash-based collections like dictionaries or hash sets.

tldr; for DTOs, if they are separate instances then no.

> **What is `async` and `await`?**

The `async` keyword enables methods to be executed asynchronously with the `await` keyword. Asynchronous programming is a form of parallelizing tasks off the main thread of execution. The main thread is notified when these tasks are completed, this frees up the main thread to accept new requests and its compute is used more efficiently instead of busy waiting.

The `await` keyword yields control back to the caller - async in .NET can be coined as "Blocking Method, Non-blocking Caller."

Read more on best-practices here - [Async guidance](https://github.com/davidfowl/AspNetCoreDiagnosticScenarios/blob/master/AsyncGuidance.md)

> **What is the benefit of an asynchronous API controller if the clients/requests are all made asynchronously?**

If the requests required long-waiting database access and were handled synchronously, then the web server would block on these requests until the database access has returned. In turn, the server threads will quickly be consumed by the blocking requests.

On the other-hand, if the requests were handled asynchronously, the database access would yield control of the thread and allow the thread to be allocated back to the thread pool for serving additional requests. This also decouples from the client in such that there isn't an assumption the request would be made asynchronously and that there could be many clients with a mixture of async/sync requests.
