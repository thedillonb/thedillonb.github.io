---
layout: post
categories: Programming
tags: 
  - programming
published: false
title: Benefiting With The Decorator Pattern
---

It's really tough to develop well written code. Luckily, object-oriented developers have been gifted a series of programming patterns to make their job easier. These patterns are a set of proven techniques that promote proper code reuse as well as application design. And, while there are many patterns to draw from, few are more beneficial than the [decorator pattern][3].

The decorator pattern is one of the most important object-oriented patterns a developer has available to them. For those whom are not familiar with the pattern:

> The decorator pattern is a design pattern that allows behavior to be added to an individual object, either statically or dynamically, without affecting the behavior of other objects from the same class.

The best way to illustrate the benefits of the decorator pattern is with an example. Consider this: you are tasked with developing code that will write a string to a file. Your boss asks you to not only write the data but to also log what data is being written as well as time how long it takes to write. 

Given the requirements above, you might set out to solve your boss' problem by writing the following code:

```
class FileWriter {
	public void Write(string data) {
		Log.Info("Writing data to file: " + data);
		Timer.Start();
		File.Write("c:\log.txt", data);
		Timer.End();
	}
}
```

Great! You completed each requirement given by your boss, the code works and everyone is happy. However, now imagine your boss comes back and tell you that he also needs you to develop code that will write data to two other locations: a network socket and the application console. For writing data to the network, you are asked to log the data being written and the time it takes to write - much like the file writer. However, for the application console, you are asked just to write the data being written - not the time it takes.

You're a smart coder so you throw together the following:

```
interface IWriter {
	void Write(string data);
}

class FileWriter : IWriter {
	public void Write(string data) {
		Log.Info("Writing data: " + data);
		Timer.Start();
		File.Write("c:\log.txt", data);
		Timer.End();
	}
}

class SocketWriter : IWriter {
	public void Write(string data) {
		Log.Info("Writing data: " + data);
		Timer.Start();
		_socket.Write(data);
		Timer.End();
	}
}

class ConsoleWriter : IWriter {
	public void Write(string data) {
		Log.Info("Writing data: " + data);
		Console.Write(data);
	}
}
``` 

Excellent. You've solved the problem again - nice touch with the interface by the way. Unfortunately, you don't feel good about what you've written. Look at all that repeated code around the logging and timing! What if your boss asks you to write another writer? Will you copy that again? As it currently stands, it's looking to be a maintenance issue later on.

Well, what about a base class? Putting the logging and timing in the base and implementation specific in the derived. Unfortunately, that won't work, at least not easily, as some of the implementations, such as the ConsoleWriter, do not time their writes. 

To make this code efficient we must know what we've done wrong. The current code suffers from one major problem: it violates the [single responsibility principle][1]. This principle is the first rule in the [SOLID][2] mnemonic, also known as the "five basic principles of object-oriented design". In short, the single responsibility principle states that we should aim to create classes that do one thing, and only one thing. In the example above this means one class for logging, one for timing, and one for each way we write the data. 

Knowing what principle we've violated has given as a hint at how to solve our conundrum. We know that we need to create more classes and have them efficiently interact with each other to mimic our current functionality but in a much more flexible way: accounting for additional requirements down the line. A quick look what [object-oriented patterns](https://en.wikipedia.org/wiki/Design_Patterns#Patterns_by_Type) we have available to us reveals a strong candidate: [decorator pattern][3]. 

Imagine you architected your code slightly different like so:
 
```
interface IWriter {
	void Write(string data);
}

class LogWriter : IWriter {
	IWriter _decoratedWriter;

	public LogWriter(IWriter decoratedWriter) {
		_decoratedWriter = decoratedWriter;
	}

	public void Write(string data) {
		Log.Info("Writing data: " + data);
		_decoratedWriter.Write(data);
	}
}

class TimingWriter : IWriter {
	IWriter _decoratedWriter;

	public TimingWriter(IWriter decoratedWriter) {
		_decoratedWriter = decoratedWriter;
	}

	public void Write(string data) {
		Timer.Start();
		_decoratedWriter.Write(data);
		Timer.End();
	}
}

class FileWriter : IWriter {
	public void Write(string data) {
		File.Write("c:\log.txt", data);
	}
}

class SocketWriter : IWriter {
	public void Write(string data) {
		_socket.Write(data);
	}
}

class ConsoleWriter : IWriter {
	public void Write(string data) {
		Console.Write(data);
	}
}
```

First off, two new classes were created: `LogWriter` and `TimingWriter`. As you've probably noticed, the code to log what we're writing is housed in the `LogWriter` and the code to time the write is in the `TimingWriter`. These two classes respectively contain functionality that does one thing and one thing only. Even more, the classes that do the actual writing are now brain-dead simple. These `Writer` classes only contain code that does the actual writing for whichever medium is being written to: file, socket, or console. Each class above are great examples of the [single responsibility principle][1]: they do one thing and one thing really well. 

Now the question becomes, how do we compose these objects so they they accomplish the original functionality requested? The decorator pattern will help immensely with this. If you noticed, the `LogWriter` and the `TimingWriter` are both `IWriter` implementations which also take in a `IWriter` object as a dependency. This is the decorator pattern at work. We can chain, or decorate, our original `IWriter`, whether it be the `ConsoleWriter`, `SocketWriter`, or `FileWriter`, with additional functionality: such as logging or timing. 

```
public void Main()
{
	// Boss says we need a way to write to a file, log what we wrote, and time it? Easy!
	IWriter fileWriter = new LogWriter(new TimingWriter(new FileWriter()));

	// What about a socket writer?
	IWriter socketWriter = new LogWriter(new TimingWriter(new SocketWriter()));

	// Finally, the console writer?
	IWriter consoleWriter = new LogWriter(new ConsoleWriter());
}
```

Notice how it was trivial to exclude timing from the `consoleWriter`? Now, when you are asked to develop new writing targets, perhaps to a database, the code to log and time that execution is already available to you. The decorator pattern has afforded us the ability to write clean, single purpose, code. Each class is now trivial to unit-test as the functionality for each is so specific and straight forward!

Finally, as a bonus, if you're using a language like C#, that provides the ability to write extension methods you can make the process of decorating absolutely beautiful. 

```
// Define some extension methods
static class WriterExtensions
{
	public static IWriter WithLogging(this IWriter writer) {
		return new LogWriter(writer);
	}

	public static IWriter WithTiming(this IWriter writer) {
		return new TimingWriter(writer);
	}
}

// See them in action:
public void Main() {
	// Looks fluent!
	IWriter socket = new SocketWriter().WithTiming().WithLogging();
}
```





[1]: https://en.wikipedia.org/wiki/Single_responsibility_principle
[2]: https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)
[3]: https://en.wikipedia.org/wiki/Decorator_pattern
[4]: https://en.wikipedia.org/wiki/Design_Patterns#Patterns_by_Type