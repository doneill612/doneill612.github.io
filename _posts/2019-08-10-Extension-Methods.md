---
layout: post
title: Extension Methods in `C#`
---

This one will be (kind of) short and (definitely) sweet: I want to talk about extension methods in C#. What are they?
How are they built? When should you build them? We'll address each of the questions!

### What is an Extension Method?

Extension methods allow us to add methods to existing types without creating new derived types, and without
modifying the original type in any way. As you'll see, they're declared statically, but are called
on instance variables of the underlying type.

If you're relatively new to C#, you might not have known that these types of methods exist - but I guarantee you've used them.
Ever hear of LINQ? You know - you declare your `using System.Linq` directive, and then you can write some really elegant
looking code to manipulate your `IEnumerable` objects...

```csharp
using System.Linq;

Dictionary<DateTime, float> dict = GetFloatDict();

// Exponential moving average never looked so good...
var ema = myFloatVals
			.Select(kv => kv.Value)
            .DefaultIfEmpty()
			.Aggregate((prev, curr) => weight * curr + (1 - weight) * prev);
```

But if you remove your `using` directive, what happens? None of those methods work anymore! This is because
LINQ is a collection of **extension methods** for the family of `IEnumerable` objects. Without importing it,
methods like `Select` and `Aggregate` aren't recognized because *they aren't actually declared* in the `IEnumerable` interface.

### When should you build Extension Methods?

Now that you know what Extension Methods are, I want to first explain when you should actually build and implement them,
before actually showing you how to do so. 

As per Microsoft's documentation: "In general, we recommend that you implement extension methods **sparingly and only when you have to**."
In my opinion (this is of course debatable) there are *exactly two* situations when you should write extension methods:

##### When you don't control the type being extended

I'll give a concrete example of this in the next section, but this essentially means that if you are working
with a data type that is defined outside the bounds of your project/solution, and if you need to add functionality
for that data type that can be used elsewhere in your application, you can write an extension method. **Don't** start building
extension methods for your own data types if you can implement the functionality directly in the type, or through inheritence.

##### When you don't want to force an implementor to construct new methods that can achieve the same functionality of existing methods.

For example, let's say you have an interface `IFoo`, and it contains a collection of floating point values. It has a method 
`SumElements()`. Now you want to implement the interface in a concrete class, `Foo : IFoo`. You realize that `Foo` should be able
to compute the mean of the floating point values it contains. You then realize that some (but not all) other implementors of `IFoo` *also* need
to compute the mean of the values. Now would be a good time to make an extension method `CalculateMean()` that does so; all of the functionality
required to compute the mean is already present in the `IFoo` interface. Your extension method could do something like `SumElements() / Count()`,
and any implementors can now call this extension method when needed.

Okay, let's clarify these two use cases with some examples (and more importantly show you how to actually build an extension method).

### How to build an Extension Method

Let's start with building an extension method for our own type. We'll first consider the last use-case of when to build an extension method discussed above.

**Note**: *For this example, please ignore the fact that LINQ already provides extension methods for summing elements in an* `IEnumerable`...

```csharp
namespace DefaultNamespace
{
	interface IFoo
	{
		float[] Values { get; }

		float SumElements();
	}
}
```

There's our `IFoo` interface - nice and simple. Now, we want to decalre a concrete type `Foo` that implements this interface and provides a method
for computing the mean of the elements.

```csharp
namespace DefaultNamespace
{
	class Foo : IFoo
	{
		float[] Values { get; set; }

		float SumElements()
		{
			float sum = float.Zero;
			for (int i = 0; i < values.Length; ++i)
			{
				sum += values[i];
			}
			return sum;
		}

		float CalculateMean()
		{
			return SumElements() / values.Length;
		}
	}
}
```

Now down the line, you realize, "Hey, I need to make another class `Bar` that also needs to calculate the mean of the elements... Guess I'll just write it again!"

```csharp
namespace DefaultNamespace
{
	class Bar : IFoo
	{
		float[] Values { get; set; }

		// This time Bar sums elements in a weird way...
		float SumElements()
		{
			float sum = float.Zero;
			for (int i = 0; i < values.Length; ++i)
			{
				sum -= values[i];
			}
			return sum;
		}

		float CalculateMean()
		{
			return SumElements() / values.Length;
		}
	}
}
```

"Oh wait, this new class `FooBar` also needs to calculate the mean... Guess I'll just write it again!" ... And so on, and so forth. The issue here is that not all concrete implementations of
`IFoo` will need to calculate the mean, but enough do - and we're re-writing a lot of code here (not really, but you get the point for more expensive operations).

##### Enter the Extension method

Okay, we know we want to make an extension method for the mean. Here's how we do it:

```csharp
namespace ExtensionsNamespace
{
	static class FooExtensions
	{
		static float CalculateMean(this IFoo foo)
		{
			return foo.SumElements() / foo.Values.Length;
		}
	}
}
```

So what do we have:

- A static class that ends with the word "Extensions"
- A static method `CalculateMean()` that takes as an argument an `IFoo` object
- The `this` modifier

`FooExtensions` is defined statically, in its own class, in a different namespace. The method `CalculateMean()` **is an extension method**. The first parameter of an extension method
is always preceded by the `this` modifier and indicates what type the extension method operates on. So, what we're saying here with this new definition is that
we are creating an extension method called `CalculateMean()` for all `IFoo` objects. What we can do now is similar to how we use LINQ. Let's assume our `Foo` and `Bar` classes no longer provide
the `CalculateMean()` method - we are instead going to use our newly created extension method.

```csharp
using DefaultNamespace;
using ExtensionsNamespace;

namespace ApplicationNamespace
{
	class Program
	{
		static void Main(string[] args)
		{
			Bar bar = new Bar();
			Foo foo = new Foo();

			// initialize bar's float array somewhere

			// using our extension method!
			var fooMean = foo.CalculateMean();
			
			// using our extension method!
			var barMean = bar.CalculateMean();
		}
		
	}
}
```

#### A more concrete example...

Now that the `FooBar` talk is over, let's consider a more concrete (yet perhaps overkill) example. Let's consider the following code:

```csharp
IDictionary<string, decimal> shoppingCart = GetShoppingCart();

foreach (var kvPair in shoppingCart)
{
	string itemName = kvPair.Key;
	decimal price = kvPair.Value;

	Console.WriteLine($"{itemName} costed me {price} dollars.");
}
```

We want to be able to make this code more compact, and perhaps do similar operations on another `IDictionary` later. Let's make an extension 
method that will allow for a LINQ-style `foreach` loop on dictionary objects.

```csharp
namespace Extensions
{
	static class DictionaryExtensions
	{
		static void ForEach<TKey, TValue>(this IDictionary<TKey, TValue> dict, 
										  Action<TKey, TValue> action)
		{
			foreach (var kvPair in dict)
			{
				var key = kvPair.Key;
				var value = kvPair.Value;

				action.Invoke(key, value);
			}
		}
	}
}
```

Let's rewrite that code from above...

```csharp
using Extensions;

GetShoppingCart().ForEach((item, price) => Console.WriteLine($"{itemName} costed me {price} dollars."));
```

There we have it! We've added an extension method to a type (`IDictionary`) that we have no control over.

I hope this was a good explanation of a really cool and useful feature. Remember - don't overuse extension methods! There's always a correct
time and place to build them. The examples presented here were perhaps overly simplistic, but I hope they can help inspire you to clean up your
code in more complex applications.