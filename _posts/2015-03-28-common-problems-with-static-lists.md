---
layout: post
published: true
title: Common Problems With Static Lists
categories: Programming
---


Immutability is often a subject that is not commonly understood in newer developers (or even older ones for that matter). The pros and cons of creating immutable objects are often not considered when coding which, for the most part, doesn't cause any initial concern. However, when a program reaches a certain complexity developers inevitably make mistakes. This is compounded when there is more than one member working on a programming project. Creating immutable objects should be your first line of defense in defensive programming.


## Common Use Case of Static Lists

```c#
class Test
{
	private static readonly IList<string> Countries = new List<string>() {
		"Australia", "India", "Japan", "Mexico", "United States" };
}
```

I've seen this kind of code a million times. The coder's intent here is simple: store a collection of strings which can be used in the class without incurring the cost of building the list every time the `Test` class is instantiated. While the code does do what was intended there are several issues with it that make it extremely undesirable.

To someone who is not familiar with the true power of the `readonly` keyword you might assume that this data structure and the data inside is completely immune against being changed after creation. Unfortunately, this far from the truth. While the `readonly` keyword prevents the `Countries` variable from being reassigned after instantiation, it doesn't prevent modification to the `List` data structure. Which means the following is completely valid:

```
class Test
{
	private static readonly IList<string> Countries = new List<string>() {
		"Australia", "India", "Japan", "Mexico", "United States" };

	public void DoSomething() {
		// Accidentally add a new, invalid, Country
		Countries.Add("Mars");
	}
}
```

Now, the coder might have intended to do this, but chances are that this was a mistake. (If not, then it raises a bunch of other questions about thread safety but that's a digression.) Because this was a global variable the "Mars" Country is now permanently part of the `Countries` collection. Any sort of manipulation on the `List` itself is possible: clear, add, remove, re-range, update. This is certainly not a good idea for a global list that should never change. Even worse, consider the following:

```c#
class Test
{
	private static readonly IList<string> Countries = new List<string>() {
		"Australia", "India", "Japan", "Mexico", "United States" };

	public IList<string> GetCountries() {
		return Countries;
	}
}
```

Yikes! Now the global variable, which is modifiable, has been passed back to another caller which  means they are now free to modify the static list from afar. This would be a huge pain in the ass to track down and one that is so easily avoidable.

## Help us ReadOnly Lists!

In .NET 4.5 they introduced the much needed `IReadOnlyList`/`IReadOnlyCollection` interfaces with their respective concrete implementations. With this came a bunch of oddities that I won't get into but the result was there was finally an easy way to create lists that were read-only!

Before I dive into why the `ReadOnly*` classes are preferable I want to make the following painfully clear:

```c#
var myList = new List<string>() {"One", "Two", "Three"};
IReadOnlyList<string> readonlyMyList = myList;

// Other things happen here

((List<string>)readonlyMyList).Add("Four"));
```

Just because I may return a `IReadOnlyList` doesn't mean that the caller cannot just cast back to the `List` implementation and make adjustments. This is not the way to use this class. In addition, you should also know that this is possible:

```c#
var myList = new List<string>() {"One", "Two", "Three"};
var readonlyMyList = new ReadOnlyCollection<string>(myList);

Console.WriteLine(string.Join(", ", readonlyMyList));
// Prints: One, Two, Three

myList.Add("Four");

Console.WriteLine(string.Join(", ", readonlyMyList));
// Prints One, Two, Three, Four
```

The `ReadOnlyCollection` class takes in a `IList` which it provides a read-only wrapper around. It does not prevent the original list from being modified which, in-turn, affects the read-only wrapper.

**Ok, so how do I use this to fix the original problem?**

With a few simple modifications we can turn our problematic list into a much more pleasing solution:

```c#
class Test
{
	private static readonly IReadOnlyList<string> Countries = new List<string>() {
		"Australia", "India", "Japan", "Mexico", "United States" }.AsReadOnly();
}
```

Two things have been done here:

1. Replaced `IList` with `IReadOnlyList`
2. Ended my `new List<string>()` instantiation with the `AsReadOnly` extension.

The `AsReadOnly` extension method wraps the `List` in a `ReadOnlyCollection`. However, because our `List` has no direct reference, exception through the `ReadOnlyCollection` we can forget about someone modifying our underlying list.

You could also do something like this:

```c#
class Test
{
	private static readonly IReadOnlyList<string> Countries = new 
		ReadOnlyCollection<string>(
			new [] { "Australia", "India", "Japan", "Mexico", "United States" });
}
```

This works just the same except you're using an array as the backing for the `ReadOnlyCollection` which brings me to a final thought:

## Arrays are also Problematic!

Don't think you can get away with avoiding all this by using arrays. While arrays may lack the methods to add or delete items they do still allow for entries to be replaced.

```c#
class Test
{
	private static readonly string[] Countries = new string[] { 
		"Australia", "India", "Japan", "Mexico", "United States", ... };

	public void DoSomething() {
		Countries[0] = "Moo Moo Cow";
		
		Console.WriteLine(string.Join(", ", Countries));
		// Prints: Moo Moo Cow, India, Japan,
	}
}
```

Woops! What should we do to fix this? Same thing we did before:

```c#
class Test
{
	private static readonly IReadOnlyCollection<string> Countries = new string[] { 
		"Australia", "India", "Japan", "Mexico", "United States", ... }
			.AsReadOnly();
}
```

## Always be Immutable

Always start off by creating read-only static collections via the `ReadOnlyCollection<T>` implementation and exposing the `IReadOnly*` interface. If you find you need to modify the collection then you can revert back to using `IList` and a non-read-only implementation. **You should train yourself to always create immutable collections first!**

As a final note on immutability and what was discussed above I should mention that proper immutability of a list should also extend to its item class. For example, I used `ReadOnlyCollection<string>` in all my examples above. `string` is also an immutable structure which means that it can also not be modified after creation. Thus, my whole structure, is immutable. I cannot change any part of it after creation. This is what's referred to as *deep-immutable*. However, if the item class I used was not immutable then it opens another can of worms:

```c#
public class Item {
	public string Name { get; set; }
}

class Test {
	private static readonly IReadOnlyList<Item> Items = new List<Item>() {
			new Item() { Name = "Dillon" },
			new Item() { Name = "Bob" },
			new Item() { Name = "John" },
		}.AsReadOnly();

	public void DoSomething() {
		Items[0].Name = "James";
	}
}
```

In this case, because my object itself is not immutable I am able to change it's contents. I am not replacing it in the collection but rather changing it internally. To prevent this, the object `Item` should also be immutable, thus creating *deep-immutability*. While this is sometimes tough to do it should never be overlooked.