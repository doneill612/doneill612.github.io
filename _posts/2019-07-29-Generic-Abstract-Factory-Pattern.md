---
layout: post
title: Abstract Factory Pattern
---

Hello, everyone. Tonight I'd like to go over a well studied design pattern I've been working with. I'm
going to assume that you have some knowledge of, at a minimum, the [Factory Pattern](https://en.wikipedia.org/wiki/Factory_method_pattern),
and that you have perhaps heard of the [Abstract Factory Pattern](https://en.wikipedia.org/wiki/Abstract_factory_pattern). 
I'll briefly introduce the former below, but the main focus of the post is going to be about the AFP,
including an example using generic types.

The code examples will be using my new favorite language at the moment, C#. I could make an entire post about how
I'm starting to greatly prefer .NET to Java, but I'll leave that for another time...

Most of this article is code, so I tried to make the examples as descriptive (yet simple) as possible. Feel free
to leave a comment if you need clarification on anything.

## Factory Pattern

Briefly consider an example of the traditional Factory pattern:

Let's say we're writing some (basic) software to design different types of vehicles. We might define the following
interface.

```csharp
public interface IVehicle
{
  void Move();
}
```

We can then implement our interface in some subclasses.

```csharp
public class Car : IVehicle
{
  public void Move()
  {
    Console.WriteLine("Accelerating on 4 wheels!");
  }
}

public class Plane : IVehicle
{
  public void Move()
  {
    Console.WriteLine("Ladies and gentlemen, we have lift-off!")
  }
}
```

Somewhere in our application logic, we might need to build a vehicle, but the type of vehicle we need need to build
depends on some other parameters. We can use a Factory to generate an appropriately typed object.

```csharp
public static class VehicleFactory
{
  public static IVehicle BuildVehicle(string key)
  {
    IVehicle vehicle;
    
    if (vehicleType == "car")
    {
      vehicle = new Car();
    }
    else
    {
      vehicle = new Plane();
    }
    
    return vehicle;
  }
}

// Somewhere else in our code...
string vehicleType = GetVehicleTypeId(); // let's say this returns "plane"
IVehicle vehicle = VehicleFactory.BuildVehicle(vehicleType);
vehicle.Move();
```

This program would output:

```
> Ladies and gentlemen, we have lift-off!
```

This is a perfectly fine example. However, things get interesting when we decide to expand our vehicles a bit...

## Abstract Factory Pattern

The purpose of the AFP is to create families of related or dependent objects by using interfaces 
and classes to defer the actual creation of concrete products in the concrete factory subclasses. 

Let's expand our application a bit.

```csharp
public abstract class Car : IVehicle
{
    public abstract string Model { get; protected set; }
    public abstract int Cost { get; protected set; }
    public abstract int NumberOfDoors { get; protected set; }

    public abstract void Move();
    public abstract void Park();
}

public abstract class Plane : IVehicle
{
    public abstract string Carrier { get; protected set; }
    public abstract int NumberOfSeats { get; protected set; }

    public abstract void Move();
    public abstract void Land();
}

public class Porsche : Car
{
    public override string Model { get; protected set; } = "911";
    public override int Cost { get; protected set; } = 150000;
    public override int NumberOfDoors { get; protected set; } = 2;

    public override void Move()
    {
        Console.WriteLine("Get out of the way!");
    }

    public override void Park()
    {
        Console.WriteLine("Where's the closest spot...?");
    }
}   

public class Mayflower : Car
{
    public override string Model { get; protected set; } = "Old";
    public override int Cost { get; protected set; } = 2;
    public override int NumberOfDoors { get; protected set; } = 4;

    public override void Move()
    {
        Console.WriteLine("Can't.");
    }

    public override void Park()
    {
        Console.WriteLine("Already stopped.");
    }
}

public class Seven47 : Plane
{
    public override string Carrier { get; protected set; } = "Swiss Airlines";
    public override int NumberOfSeats { get; protected set; } = int.MaxValue;

    public override void Move()
    {
        Console.WriteLine("Bon voyage!");
    }

    public override void Land()
    {
        Console.WriteLine("*Applause*");
    }
}

public class TwinEngine : Plane
{
    public override string Carrier { get; protected set; } = "Local Pilot";
    public override int NumberOfSeats { get; protected set; } = 2;

    public override void Move()
    {
        Console.WriteLine("You sure you know how to fly this thing?");
    }

    public override void Land()
    {
        throw new NotImplementedException();
    }
}
```

As you can see, we've added quite a bit of content.

- `Car` and `Plane` have become abstract
- `Car` has a new abstract method called `Park()`
- `Plane` has a new abstract method called `Land()`
- We have created concrete implementations for both super types, and the concrete implementations
have some unique characteristics (namely high-end specs vs low-budget specs).

As your application grows, families of related objects emerge. The last bullet point here indicates that we could perhaps
create an abstract factory with concrete implementations for each object family.

```csharp
public interface IVehicleFactory
{
    Car BuildCar(string key);
    Plane BuildPlane(string key);
}

public class HighendFactory : IVehicleFactory
{
    public Car BuildCar(string key)
    {
        Car vehicle;
        if (key.Equals("Porsche"))
        {
            vehicle = new Porsche();
        }
        // other cases go here with else if (...) ...
        else
        {
            throw new NotSupportedException();
        }
        return vehicle;
    }

    public Plane BuildPlane(string key)
    {
        Plane vehicle;
        if (key.Equals("747"))
        {
            vehicle = new Seven47();
        }
        // other cases go here with else if (...) ...
        else
        {
            throw new NotSupportedException();
        }
        return vehicle;
    }
}

public class LowbudgetFactory : IVehicleFactory
{
    public Car BuildCar(string key)
    {
        Car vehicle;
        if (key.Equals("Mayflower"))
        {
            vehicle = new Mayflower();
        }
        // other cases go here with else if (...) ...
        else
        {
            throw new NotSupportedException();
        }
        return vehicle;
    }

    public Plane BuildPlane(string key)
    {
        Plane vehicle;
        if (key.Equals("TwinEngine"))
        {
            vehicle = new TwinEngine();
        }
        // other cases go here with else if (...) ...
        else
        {
            throw new NotSupportedException();
        }
        return vehicle;
    }
}


// Somewhere else
IVehicleFactory highEndFactory = new HighendFactory();
IVehicleFactory lowBudgetFactory = new LowbudgetFactory();
  
Car car = highEndFactory.BuildCar("Porsche"); // returns us a brand new Porsche 911.
Plane plane = lowBudgetFactory.BuildPlane("TwinEngine"); // returns us a Twin Engine plane.

Car otherCar = lowBudgetFactory.BuildCar("Porsche"); // returns null - should probably throw instead!

car.Move();
plane.Land();
```
```
> Get out of the way!
> Program threw exception: System.NotImplementedException: The method or operation is not implemented.
```

Pretty powerful indeed. This is perhaps an oversimplified example, but as the application grows in complexity
and more diverse object families emerge, the factory interface can become quite useful.

## Generic Abstract Factories

Here's another way we can implement the Abstract Factory Pattern using generics. The logic can be a bit tough
to follow at first, but the functionality is quite amazing.

```csharp
public interface IGenericVehicle<TFactory>
{
    void Move();
}
```
This interface tells use two things:

1. Vehicles can move.
2. Vehicles are produced by some factory object of type `TFactory`. More on this shortly.

We now want to define a factory that is capable of producing any arbitrary type of vehicular product.

```csharp
public interface IVehicleFactory<TFactory>
{
    TVehicle BuildVehicle<TVehicle>() where TVehicle : IGenericVehicle<TFactory>, new();
}
```

Let's decompose this... What we're defining here is a generically-typed factory interface - it's a factory
that produces some vehicular product. More specifically, the factory method produces 
a `TVehicle` object, where `TVehicle` is some `IGenericVehicle`.

We can now implement this interface in concrete factory objects.

```csharp
public class CarFactory : IVehicleFactory<CarFactory>
{
    public TVehicle BuildVehicle<TVehicle>() where TVehicle : IGenericVehicle<CarFactory>, new()
    {
        return new TVehicle();
    }
}

public class PlaneFactory : IVehicleFactory<PlaneFactory>
{
    public TVehicle BuildVehicle<TVehicle>() where TVehicle : IGenericVehicle<PlaneFactory>, new()
    {
        return new TVehicle();
    }
}
```

Now look at the `BuildVehicle` implementations: each factory still produces a `TVehicle`; 
but recall what a `TVehicle` is. It is a subclass of `IGenericVehicle` that is produced by a particular
`TFactory`. All we're simply saying here is that the `TVehicle` objects that `CarFactory` builds are 
`IGenericVehicle<CarFactory>` objects. In plain English - *they're cars*. The same logic applies for the `PlaneFactory`.

Let's define our concrete vehicle types:

```csharp
public class Porsche : IGenericVehicle<CarFactory>
{
    public void Move()
    {
        Console.WriteLine("Get out of the way!");
    }
}

public class Mayflower : IGenericVehicle<CarFactory>
{
    public void Move()
    {
        Console.WriteLine("Can't move.");
    }
}

public class Seven47 : IGenericVehicle<PlaneFactory>
{
    public void Move()
    {
        Console.WriteLine("Bon voyage!");
    }
}

public class TwinEngine : IGenericVehicle<PlaneFactory>
{
    public void Move()
    {
        Console.WriteLine("You sure you know how to fly this thing?");
    }
}
```

`Porsche : IGenericVehicle<CarFactory>` says "A `Porsche` is a generic vehicle that can be produced by a `CarFactory`." 
Likewise: `TwinEngine : IGenericVehicle<PlaneFactory>` says "A `TwinEngine` is a generic vehicle that can be 
produced by a `PlaneFactory`."

So, how do we use our factories? 

```csharp

CarFactory carFactory = new CarFactory();
PlaneFactory planeFactory = new PlaneFactory();

var porsche = carFactory.BuildVehicle<Porsche>();
var twinEngine = planeFactory.BuildVehicle<TwinEngine>();

porsche.Move()
twinEngine.Move()
```
```
> Get out of the way!
> You sure you know how to fly this thing?
```

Pretty cool, huh? As an exercise - try implementing object-specific functionality in the cars and planes, and using
the factory to produce an appropriately typed object.

Hope you enjoyed!

