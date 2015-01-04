---
layout: post
published: true
title: "Dependency Injection: Constructor vs Property"
categories: Programming
tags: 
  - design
---

Dependency injection is one of my favourite programming patterns for large projects. Among other things, it's great for promoting loosely coupled components which, when projects get large, is a worth it's weight in gold as re-factoring becomes a much easier and testing these independent modules is cake since they typically follow the [dependency inversion principle](http://en.wikipedia.org/wiki/Dependency_inversion_principle). Regardless, code typically has two fundamental ways of taking a dependency: via the constructor or via a property (or method). Unfortunately, one of these should be avoided at all cost.

## Constructor Injection
The following is an example of constructor injection. As the name suggests, dependencies are injected via the objects constructor and typically saved away for use in any method.

```c#
class Stuff
{
    private readonly IDataRepository _dataRepository;
	
	// The dependency injector will inject a concrete implementation
	// of the IDataRepository into the constructor of this object
	// during instantiation
    public Stuff(IDataRepository dataRepository)
	{
	    _dataRepository = dataRepository;
	}
	
	// Get an item from the IDataRepository
	public string GetItem(int id)
	{
	    return _dataRepository.Get(id);
	}
}
```


### Property Injection
The following is an example in property injection. Note how this differs from the constructor injection above. A property (or method) is called by the dependency injector after the object is instantiated, passing the dependency into the object to be saved and utilized later.

```c#
class Stuff
{
    private IDataRepository _dataRepository;
	
	// The dependency injector will inject a concrete implementation
	// of the IDataRepository into the property after instantiation
	public IDataRepository DataRepository
	{
	    get { return _dataRepository; }
		set { _dataRepository = value; }
	}
	
	// Just an empty constructor
    public Stuff()
	{
	}
	
	// Get an item from the IDataRepository
	public string GetItem(int id)
	{
	    return _dataRepository.Get(id);
	}
}
```

## What's the big deal?

Ok, so which one should you prefer? Always **constructor** over **property**. Constructor injection has a huge amount of benifits that property injection does not have. Constructor injection allows objects to be immutable. Immutable objects should always be preferred as they tend to be simplier to reason about. 

For example, during construction the object can check every dependency to make sure it exists and not null. If something is wrong, the object will fail to instantiate immediately at the time of creation. Otherwise, the object knows that all dependecies are met and that it is ready to rock. It also means that no-one can accidently swap out the dependency or make a reference change to it. In property injection, there's no way to tell when the injector is done wiring up dependencies. That may mean that one of your dependencies does not resolve to a value and stays null. Do you then wrap every dependency call in a `If` statement to check if there's a valid reference? What happens when there's not? 

```c#
class Stuff
{
    // Truncated from Property injection example above.

	// Get an item from the IDataRepository
	public string GetItem(int id)
	{
	    if (_dataRepository == null)
		    throw new InvalidOperationException("No data repository");
	    return _dataRepository.Get(id);
	}
}
```

Yikes, that's already getting ugly and that's a trivial example. Now my calling code has to brace for this stupid exception - complexity rises. Even worst, I didn't figure out that my object was wired inappropriately until I called the `GetItem` method. It's definitely easier to just avoid that situation by avoiding property injection.


> But property injection avoids constructors with tons of arguments!

That should not be an excuse. Just because you're anxious about the number of arguments does not mean you should forgo the advantages described above. In fact, this is typically a problem of an object that does not follow the [single responsibility principle](http://en.wikipedia.org/wiki/Single_responsibility_principle). Most of the time, objects that require a huge number of dependencies are trying to do too much. These objects should be broken down into smaller functional pieces - it'll really help during testing too. However, if you're certain that this object needs all these dependencies you should look into creating dependencies that aggregate other dependencies. Between these two approached, this problem should never really arise.


> What about if you're using inheritance and you add a dependency at the base class, can I avoid touching the super-classes by using property injection?

Uh, yes. But should you? No. You made your bed with choosing inheritance over composition and now you're going to pay with the extra work you have to do. Honestly though, if you're working with UI frameworks that pretty much lock you into inheritance it still doesn't make sense to cheat and add a property injection to make your life easier now and potentially more difficult in the future. Again, you'd be giving up the advantages of constructor injection because you're lazy.


> I have an object that depends on another which in-turn depends on this object... Property injection?

This is the **only** reason I would potentially even think about property injection. This problem is typically one of bad object design which you should be able to rectify with some clever thinking. However, if you absolutely can't change those objects then you're stuck. This is where constructor injection fails since neither would be able to instantiate themselves. In this rare case, property injection would be you're only solution :(
