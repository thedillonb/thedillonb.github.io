---
layout: post
categories: Programming
tags:
  - programming
published: true
title: The Singleton Anti-Pattern
---

The singleton is an anti-pattern. Out of all of the object-oriented patterns, the singleton pattern is one of the most widely bastardized and misused. It secretly undermines good choices in application design and introduces global state into an application which tends to bring about unnecessary restrictions. Nine out of ten times the singleton pattern is either implemented wrong, used incorrectly, or just completely unnecessary. The remaining one out of ten it *might* be appropriate - but I doubt it, and you should to.

The use of the singleton pattern brings along the inevitable violation of [dependency injection][1]. Meaning, instead of writing

```csharp
class SomeObject
{
	IThing thing;

	public SomeObject(IThing thing)
	{
		this.thing = thing;
	}

	public void Start()
	{
		this.thing.Execute();
	}
}
```

You end up writing

```csharp
class SomeObject
{
	public void Start()
	{
		Thing thing = Thing.GetInstance();
		manager.Execute();
	}
}
```

The latter works but completely ignores dependency injection. One of the strengths of dependency injection is that it promotes loosely coupled interactions between application code which, ultimately, breeds design that follows the [dependency inversion principle][2].

The latter design makes it a nightmare to test efficiently. Imagine the `Thing.Execute()` method does some number crunching and takes upwards of an hour. When I go to create tests for my `SomeObject` class how am I going to test the `Start` method without actually running that computation? There's no way you, or any member of your team, should be waiting an hour for testing feedback. The latter design will obviously not work.

The former design, however, makes testing trivial. Because of it's injection design - as well as it's use of an interface, which we'll get to in the next section - I can easily create my own implementation of the `IThing` interface and create a `Execute` method that simply returns test data immediately. Because of the loosely coupled design the dependency injection affords it has allowed me to use such techniques as [mock objects][4] to inject my own functionality.

The idea of [Design by contract][5] is an incredibly powerful, and flexible, concept that, when coupled with dependency injection, forms the basis for extremely well written object-oriented applications. As seen in the above section, the use of a `IThing` in the constructor allows me to pass in any implementation of the interface for the `SomeObject` class to operate on. This relies on the fact that we're referring to an objects contract rather than it's concrete implementation.

Let's take a simple singleton method

```csharp
public Thing GetInstance() { .. }
```

Great. It returns a `Thing` object. But, does that `Thing` object implement an interface describing it's functionality? From my experience, probably not. More often, I'll find a singleton class with no interfaces. This is most likely due to the fact that the singleton class is directly instanced rather than passed via a constructor, or method:

```csharp
public void MyMethod()
{
	// No need for interfaces if I plan to use the singleton
	// directly. Right?
	Thing thing = Thing.GetInstance();
	thing.Execute();
}
```

This lack of an interface means that referring to the singleton is always done via a concrete implementation - like seen above. Even if you follow the dependency injection described in the section above it means you must pass a concrete implementation into the constructor. There's only one place you're going to get a concrete implementation of the class: `Thing.GetInstance()`.

> A well written singleton implementation returns a concrete implementation which implements a series of interface contracts describing it's functionality.

 Application code should never refer to the concrete implementation type of the singleton; only it's interfaces. It's an easy fix: just extract out an interface from your singleton class and use that in your application code.

```csharp
// Define the interface my singleton will implement
interface IThing
{
	void Execute();
}

// Define my singleton, implementing my interface
class Thing : IThing
{
	// Notice, I'm not suggesting returning an interface here.
	// Returning the concrete is fine. But when I call this, I should
	// only refer to the return type by it's interface definition.
	public static Thing GetInstance() { .. }
	public void Execute() { .. }
}

// Define the main application, which will call the singleton
class MainApplication
{
	public void Main()
	{
		IThing t = Thing.GetInstance();

		// Pass the 'singleton' object into our other classes via
		// dependency injection. They don't need to know it's a
		// singleton and they only refer to the object
		// by it's interface. Everybody wins!
		new SomeObject(t).Execute();
	}
}
```

Check out the `Main` method in the example above: the application *just* began and we're already asking for a singleton instance which we'll use to pass to dependent classes. If there were other classes that depended on `IThing` we could easily pass them an instance from this point which begs the question: what's the point of the singleton? Can we simply rewrite this and avoid the singleton pattern all together?

```csharp
class MainApplication
{
	public void Main()
	{
		// Instantiate Operation once, perhaps at the start of the program
		// and pass it into all the dependent classes.
		IThing t = new Thing();

		// Let's create three instances of Test!
		new SomeObject(t).Execute();
		new SomeObject(t).Execute();
		new SomeObject(t).Execute();
	}
}
```

I've easily rewritten this to not use the singleton pattern, instead, relying on dependency injection to maintain operational consistency. The single instantiation of `Thing` simulates the singleton we had before and every dependent object still gets the same instance by injecting it into their construction.


[1]: https://en.wikipedia.org/wiki/Dependency_injection
[2]: https://en.wikipedia.org/wiki/Dependency_inversion_principle
[3]: https://en.wikipedia.org/wiki/Single_responsibility_principle
[4]: https://en.wikipedia.org/wiki/Mock_object
[5]: https://en.wikipedia.org/wiki/Design_by_contract
