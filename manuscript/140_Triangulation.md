# Triangulation

TODO CHANGE THIS CHAPTER NAME TO SOMETHING ELSE!!

As one of the last topics of the core TDD techniques that do not require us to delve into the object-oriented world, I'd like to show you triangulation - one of the techniques for turning a false Statement true. I consider it an important technique because:

1. Many TDD practitioners use it and demonstrate it, so I assume you will see it sooner or later and most likely have questions regarding it.
1. It allows us to take tiny steps in arriving at the right implementation (tiniest than any you have seen so far in this book) and I find it very useful when I'm uncertain how the correct implementation and design should look like.

*Triangulation* seems to be a mysterious term at first - at least it was to me, especially that it didn't bring anything related to software engineering to my mind. The first time I encountered it was when reading Kent Beck's book [Test-Driven Development: By Example](http://www.pearsonhighered.com/educator/product/Test-Driven-Development-By-Example/9780321146533.page). In this book, Kent mentions three approaches to make a failing test pass (or, in our words, to turn a false Statement true):

1. Type the obvious implementation
1. Fake it (‘til you make it)
1. Triangulate

Don't worry if these names don't tell you anything, the techniques are quite simple and I will try to give an example for each (putting more emphasis on triangulation).

## Type the obvious implementation

The first of the three techniques is what we have done a lot throughout this book. It simply says: when you know the correct and final implementation to turn a Statement true, then just type it. If the implementation is simple, this approach makes a lot of sense - after all, the amount of Statements required to specify (and test) a functionality is about a level of confidence. If this level is very high, we can just type the correct code in response to a single Statement. Let's see it again using a trivial example of adding two numbers:

```csharp
[Fact] public void
ShouldAddTwoNumbersTogether()
{
  //GIVEN
  var sum = new Sum();

  //WHEN
  var result = sum.Of(3,5);

  //THEN
  Assert.Equal(8, result);
}
```

You may remember I told you that usually we write the simplest production code that would make the Statement true. This rule would encourage us to just return 8 from the `Of` method, because it would be sufficient to make the Statement true. Instead, we can decide that the logic is so obvious, that we can just write it based on this one Statement:

```csharp
public class Sum
{
  public int Of(int a, int b)
  {
    return a+b;
  }
}
```

and that's it. Note that I didn't use Constrained Non-Determinism here, because its use kind of enforces using "Just write obvious implementation" technique. As I mentioned before, most (if not all) Statements we wrote so far in previous chapters, uses this approach because of this fact. Let's take a look at how the above Statement would look if we used Constrained Non-Determinism:

```csharp
[Fact] public void
ShouldAddTwoNumbersTogether()
{
  //GIVEN
  var a = Any.Integer();
  var b = Any.Integer();
  var sum = new Sum();

  //WHEN
  var result = sum.Of(a,b);

  //THEN
  Assert.Equal(a + b, result);
}
```

Here, we don't have any choice. The most obvious implementation that would make this Statement true is the correct implementation. We are unable to return a constant value as we previously could (even though we chose not to), because we just don't know what the expected result is and it is strictly dependent on the input values which we don't know as well.

## Fake it (‘til you make it)

This technique is kind of funny. I don't recall myself ever using it, yet it is so interesting that I want to show it to you anyway. It is so simple you will not regret these few minutes even if just for broadening your horizons.

There are two core steps of *fake it ('till you make it)*:

1. It starts with a "fake it" step. Here, we turn a false Statement true by using the simplest implementation possible, even if it's not the correct implementation (hence the name of the step). Usually, returning a literal constant is enough at the beginning. 
1. Then we proceed with the "make it" step - we rely on your sense of duplication between Statement and (fake) implementation to gradually transform both into their more general forms that eliminate this duplication. Usually, we achieve this by introducing variables, parameters etc.

Let's apply Fake It to the same addition example as before (I promise, for triangulation, I will give you better one). The Statement looks the same as before:

```csharp
[Fact] public void
ShouldAddTwoNumbersTogether()
{
  //GIVEN
  var sum = new Sum();

  //WHEN
  var result = sum.Of(3, 5);

  //THEN
  Assert.Equal(8, result);
}
```

For the implementation, however, we are going to use the simplest code that will turn the Statement true. As I wrote, this simplest implementation is almost always to return a constant:

```csharp
public class Sum
{
  public int Of(int a, int b)
  {
    return 8;
  }
}
```

The Statement turns true (green) now, even though the implementation is obviously wrong. Now is the time to remove duplication between the Statement and the production code.

First, let's note that the number 8 is duplicated between Statement and implementation -- the implementation returns it and the Statement asserts on it. To reduce this duplication, let's break the 8 in the implementation into a sum:

```csharp
public class Sum
{
  public int Of(int a, int b)
  {
    return 3 + 5;
  }
}
```

Note the smart trick I did. I changed the duplication between implementation and *expected result* to duplication between implementation and *input values* of the Statement. I used `3 + 5` exactly because the Statement used these two values like this:

```csharp
var result = sum.Of(3, 5);
```

This kind of duplication is different from the previous one in that it can be removed using variables (this applies not only to input variables, but basically anything we have access to prior to triggering specified behavior -- constructor parameters, fields etc. in contrast to result which we normally don't know until we invoke the behavior). The duplication of number 3 can be eliminated by changing the production code to use the value passed from the Statement:

```csharp
public class Sum
{
  public int Of(int a, int b)
  {
    return a + 5;
  }
}
```

and we only have the number 5 duplicated, because we used variable to transfer the value of 3 from Statement method to the `Of` implementation, so we have it in one place now. let's do the same with 5:

```csharp
public class Sum
{
  public int Of(int a, int b)
  {
    return a + b;
  }
}
```

And that's it. I used a trivial example, since I don't want to spend too much time on this, but you can find more advanced ones in Kent Beck's book if you like.

## Triangulate

Triangulation is considered the most conservative technique of the described trio, because following it involves the tiniest possible steps to arrive at the right solution. The technique is called triangulation by analogy to [radar triangulation](http://encyclopedia2.thefreedictionary.com/radar+triangulation) where outputs from at least two radars must be used to determine the position of a unit. Also, in radar triangulation, the position is measured indirectly, by combining the following data: range (not position!) between two radars, measurement done by each radar and the positions of the radars (which we know, because we are the ones who put the radars there). From this data, we can derive a triangle, so we can use trigonometry to calculate the position of the third point of the triangle, which is the desired position of the unit (two remaining points are the positions of radars). Such measurement is indirect in nature, because we don't measure the position directly, but calculate it from other helper measurements.

These two characteristics: indirect measurement and using at least two sources of information are at the core of TDD triangulation. Basically, it translates like this:

1. **Indirect measurement**: derive the internal implementation and design of a module from several known examples of its desired externally visible behavior by looking at what varies in these examples and changing the production code so that this variability is treated in a generic way. For example, variability might lead us from changing a constant to a variable.
1. **Using at least two sources of information**: start with the simplest possible implementation and make it more general **only** when you have two or more different examples (i.e. Statements that describe the desired functionality for specific inputs). Then new examples can be added and generalization can be done again. This process is repeated until we reach the desired implementation. Robert C. Martin developed a maxim on this, saying that "As the tests get more specific, the code gets more generic.

Usually, when TDD is showcased on simple examples, triangulation is the primary technique used, so many novices mistakenly believe TDD is all about triangulation. This isn't true, although triangulation is an important tool in our toolbox.

### Example - addition of numbers

Before I show you a more advanced example of triangulation, I would like to get back to our toy example of adding two integer numbers. This will allow us to see how triangulation differs from the other two techniques mentioned earlier.

We will use the xUnit.net's feature of parameterized Statements, i.e. theories - this will allow us to give many examples of the desired functionality without duplicating the code.

The first Statement looks like this:

```csharp
[Theory]
[InlineData(0,0,0)]
public void ShouldAddTwoNumbersTogether(
  int int1,
  int int2,
  int expectedResult)
{
  //GIVEN
  var sum = new Sum();

  //WHEN
  var result = sum.Of(int1, int2);

  //THEN
  Assert.Equal(expectedResult, result);
}
```

Note that we parameterized not only the input values, but also the expected result. The first example specifies that `0 + 0 = 0`.

The implementation, similarly to *fake it ('till you make it)* is, for now, to just return a constant:

```csharp
public class Sum
{
  public int Of(int a, int b)
  {
    return 0;
  }
}
```

Now, contrary to *fake it...* technique, we don't try to remove duplication between the Statement and the code. Instead, we add another example of the same rule. What do I mean by "the same rule"? Well, we need to consider our axis of variability. In addition, there are two things that can vary - either the first argument of addition, or the second argument. For the second example, we need to keep one of them unchanged while changing the other. Let's say that we decide to keep the second input value the same (which is 0) and change the first value to 1. So this:

```csharp
[Theory]
[InlineData(0,0,0)]
```

Becomes:

```csharp
[Theory]
[InlineData(0,0,0)]
[InlineData(1,0,1)]
```

Again, not that the second input value stays the same in both examples and the first one varies. The expecied result needs to be different as well, obviously.

As for the implementation, we still try to make the Statement true by making as dumb implementation as possible:

```csharp
public class Sum
{
  public int Of(int a, int b)
  {
    if(a == 1) return 1;
    return 0;
  }
}
```

By this time, we may not yet have an idea on how to generalize the implementation, so let's add the third example:

```csharp
[Theory]
[InlineData(0,0,0)]
[InlineData(1,0,1)]
[InlineData(2,0,2)]
```

And the implementation would be:

```csharp
public class Sum
{
  public int Of(int a, int b)
  {
    if(a == 2) return 2;
    if(a == 1) return 1;
    return 0;
  }
}
```

Now, looking at this code, we may spot a pattern - for every input values so far, we return the value of the first one: for 1 we return 1, for 2 we return 2, for 0 we return 0. Thus, we can generalize this implementation to always return the first input:

```csharp
public class Sum
{
  public int Of(int a, int b)
  {
    return a;
  }
}
```

Thus, we have generalized the first axis of variability, which is the first input argument. Time to vary the second one, by leaving the first value unchanged. Our next example would be:

TODO TODO TODO TODO TODO TODO TODO 



### Example 2

Suppose we want to write a logic that creates an aggregate sum of the list. Let's assume that we have no idea how to design the internals of our custom list class so that it fulfills its responsibility. Thus, we start with the simplest example of calculating a sum of 0 elements:

```csharp
[Fact] public void 
ShouldReturn0AsASumOfNoElements()
{
  //GIVEN
  var listWithAggregateOperations 
    = new ListWithAggregateOperations();

  //WHEN
  var result = listWithAggregateOperations.SumOfElements();

  //THEN
  Assert.Equal(0, result);
}
```

Remember we want to write just enough code to make the Statement true. We can achieve it with just returning 0 from the `SumOfElements` method:

```csharp
public class ListWithAggregateOperations
{
  public int SumOfElements()
  {
    return 0;
  }
}
```

This is not yet the implementation we are happy with, which makes us add another Statement -- this time for a single element:

```csharp
[Fact] public void 
ShouldReturnTheSameElementAsASumOfSingleElement()
{
  //GIVEN
  var singleElement = Any.Integer();
  var listWithAggregateOperations 
    = new ListWithAggregateOperations(singleElement);

  //WHEN
  var result = listWithAggregateOperations.SumOfElements();

  //THEN
  Assert.Equal(singleElement, result);
}
```

The naive implementation can be as follows:

```csharp
public class ListWithAggregateOperations
{
  int _element = 0;

  public ListWithAggregateOperations()
  {    
  }

  public ListWithAggregateOperations(int element)
  {
    _element = element;
  }

  public int SumOfElements()
  {
    return _element;
  }
}
```

We have two examples, so let's check whether we can generalize now. We could try to get rid of the two constructors now, but let's wait just a little bit longer and see if this is the right path to go (after all, I wrote that we need **at least** two examples).

Let's add third example then. What would be the next more complex one? Note that the choice of next example is not random. Triangulation is about considering the axes of variability. If you carefully read the last example, you probably noticed that we already skipped one axis of variability -- the value of the element. We used `Any.Integer()` where we could use a literal value and add a second example with another value to make us turn it into variable. This time, however, I decided to **type the obvious implementation**. The second axis of variability is the number of elements. The third example will move us further along this axis -- so it will use two elements instead of one or zero. This is how it looks like:

```csharp
[Fact] public void 
ShouldReturnSumOfTwoElementsAsASumWhenTwoElementsAreSupplied()
{
  //GIVEN
  var firstElement = Any.Integer();
  var secondElement = Any.Integer();
  var listWithAggregateOperations 
    = new ListWithAggregateOperations(firstElement, secondElement);

  //WHEN
  var result = listWithAggregateOperations.SumOfElements();

  //THEN
  Assert.Equal(firstElement + secondElement, result);
}
```

And the naive implementation will look like this:

```csharp
public class ListWithAggregateOperations
{
  int _element1 = 0;
  int _element2 = 0;

  public ListWithAggregateOperations()
  {
  }

  public ListWithAggregateOperations(int element)
  {
    _element1 = element;
  }

  //added
  public ListWithAggregateOperations(int element1, int element2)
  {
    _element1 = element1;
    _element2 = element2;
  }

  public int SumOfElements()
  {
    return _element1 + _element2; //changed
  }
}
```

After adding and implementing the third example, the variability of elements count becomes obvious. Now that we have three examples, we see even more clearly that we have redundant constructors and redundant fields for each element in the list and if we added a fourth example for three elements, we'd have to add another constructor, another field and another element of the sum computation. Time to generalize!

How do we encapsulate the variability of the element count so that we can get rid of this redundancy? A collection! How do we generalize the addition of multiple elements? A foreach loop through the collection! Thankfully, C# supports `params` keyword, so let's use it to remove the redundant constructor like this:

```csharp
public class ListWithAggregateOperations
{
  int[] _elements;  

  public ListWithAggregateOperations(params int[] elements)
  {
    _elements = elements;
  }

  public int SumOfElements()
  {
    //changed
    int sum = 0;
    foreach(var element in _elements)
    {
      sum += element;
    }
    return sum;
  }
}
```

While the first Statement ("no elements") seems like a special case, the remaining two -- for one and two elements -- seem to be just two variations of the same behavior ("some elements"). Thus, it is a good idea to make a more general Statement that describes this logic to replace the two examples. After all, we don't want more than one failure for the same reason. So as the next step, I will write a Statement to replace these examples (I leave them in though, until I get this one to evaluate to true).

```csharp
[Fact]
public void 
ShouldReturnSumOfAllItsElementsWhenAskedForAggregateSum()
{
  //GIVEN
  var firstElement = Any.Integer();
  var secondElement = Any.Integer();
  var thirdElement = Any.Integer();
  var listWithAggregateOperations 
    = new ListWithAggregateOperations(
        firstElement, 
        secondElement, 
        thirdElement);

  //WHEN
  var result = listWithAggregateOperations.SumOfElements();

  //THEN
  Assert.Equal(
   firstElement + 
   secondElement + 
   thirdElement, result);
}
```

This Statement uses three values rather than zero, one or two as in the examples we had. When I need to use collections with deterministic size (and I do prefer to do it everywhere where using collection with non-deterministic size would force me to use a `for` loop in my Statement), I pick 3, which is the number I got from Mark Seemann and the rationale is that 3 is the smallest number that has distinct head, tail and middle element. One or two elements seem like a special case, while three sounds generic enough.

One more thing we can do is to ensure that we didn't write a **false positive**, i.e. a Statement that is always true due to being badly written. In other words, we need to ensure that the Statement we just wrote will ever evaluate to false if the implementation is wrong. As we wrote it after the implementation is in place, we do not have this certainty.

What we will do is to modify the implementation slightly to make it badly implemented and see how our Statement will react (we expect it to evaluate to false):

```csharp
public int SumOfElements()
{
  //changed
  int sum = 0;
  foreach(var element in _elements)
  {
    sum += element;
  }
  return sum + 1; //modified with "+1"!
}
```

When we do this, we can see our last Statement evaluate to false with a message like "expected 21, got 22". We can now undo this one little change and go back to correct implementation.

The examples ("zero elements", "one element" and "two elements") still evaluate to true, but it's now safe to remove the last two, leaving only the Statement about a behavior we expect when we calculate sum of no elements and the Statement about N elements we just wrote.

And voilà! We have arrived at the final, generic solution. Note that the steps we took were tiny -- so you might get the impression that the effort was not worth it. Indeed, this example was only to show the mechanics of triangulation -- in real life, if we encountered such simple situation we'd know straight away what the design would be and we'd start with the general Statement straight away and just type in the obvious implementation. Triangulation shows its power in more complex problems with multiple design axes and where taking tiny steps helps avoid "analysis paralysis".

## Summary

As I stated before, triangulation is most useful when you have no idea how the internal design of a piece of functionality will look like (e.g. even if there are work-flows, they cannot be easily derived from your knowledge of the domain) and it's not obvious along which axes your design must provide generality, but you are able to give some examples of the observable behavior of that functionality given certain inputs. These are usually situations where you need to slow down and take tiny steps that slowly bring you closer to the right design and functionality -- and that's what triangulation is for! 
