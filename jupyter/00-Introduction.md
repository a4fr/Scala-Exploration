# Introduction

## Programming in data science

This book is not a book about data science. It is a book about how to use [Scala](https://en.wikipedia.org/wiki/Scala_(programming_language)), a programming language, for data science. So, where does programming come in when processing data?

Computers are involved at every step of the data science pipeline, but not necessarily in the same manner. The style of programs that we build will be drastically different if we are just writing throwaway scripts to explore data or trying to build a scalable application that pushes data through a well-understood pipeline to continuously deliver business intelligence.

Let's imagine that we work for a company making games for mobile phones in which you can purchase in-game benefits. The majority of users never buy anything, but a small fraction is likely to spend a lot of money. We want to build a model that recognizes big spenders based on their play patterns.

The first step is to explore data, find the right features, and build a model based on a subset of the data. In this exploration phase, we have a clear goal in mind but little idea of how to get there. We want a light, flexible language with strong libraries to get us a working model as soon as possible.

Once we have a working model, we need to deploy it on our gaming platform to analyze the usage patterns of all the current users. This is a very different problem: we have a relatively clear understanding of the goals of the program and of how to get there. The challenge comes in designing software that will scale out to handle all the users and be robust to future changes in usage patterns. 

In practice, the type of software that we write typically lies on a spectrum ranging from a single throwaway script to production-level code that must be proof against future expansion and load increases. Before writing any code, the data scientist must understand where their software lies on this spectrum. Let's call this the permanence spectrum.

## Why Scala?

You want to write a program that handles data. Which language should you choose?

There are a few different options. You might choose a dynamic language such as 
Python or R or a more traditional object-oriented language such as Java. In this 
section, we will explore how Scala differs from these languages and when it might 
make sense to use it.

When choosing a language, the architect's trade-off lies in a balance of provable correctness versus development speed. Which of these aspects you need to emphasize will depend on the application requirements and where on the permanence spectrum your program lies. Is this a short script that will be used by a few people who can easily fix any problems that arise? If so, you can probably permit a certain number of bugs in rarely used code paths: when a developer hits a snag, they can just fix the problem as it arises. By contrast, if you are developing a database engine that you plan on releasing to the wider world, you will, in all likelihood, favor correctness over rapid development. The SQLite database engine, for instance, is famous for its extensive test suite, with 800 times as much testing code as application code (https://www.sqlite.org/testing.html). 

What matters, when estimating the correctness of a program, is not the perceived absence of bugs, it is the degree to which you can prove that certain bugs are absent.

There are several ways of proving the absence of bugs before the code has even run:
• Static type checking occurs at compile time in statically typed languages, 
but this can also be used in strongly typed dynamic languages that support 
type annotations or type hints. Type checking helps verify that we are using 
functions and classes as intended.
• Static analyzers and linters that check for undefined variables or suspicious 
behavior (such as parts of the code that can never be reached).
• Declaring some attributes as immutable or constant in compiled languages.
• Unit testing to demonstrate the absence of bugs along particular code paths.

There are several more ways of checking for the absence of some bugs at runtime:
• Dynamic type checking in both statically typed and dynamic languages
• Assertions verifying supposed program invariants or expected contracts

In the next sections, we will examine how Scala compares to other languages in 
data science.

## Static typing and type inference

Scala's static typing system is very versatile. A lot of information as to the program's behavior can be encoded in types, allowing the compiler to guarantee a certain level of correctness. This is particularly useful for code paths that are rarely used. A dynamic language cannot catch errors until a particular branch of execution runs, so a bug can persist for a long time until the program runs into it. In a statically typed language, any bug that can be caught by the compiler will be caught at compile time, before the program has even started running.

Statically typed object-oriented languages have often been criticized for being 
needlessly verbose. Consider the initialization of an instance of the Example
class in Java:

```java
Example myInstance = new Example() ;
```

We have to repeat the class name twice—once to define the compile-time type of 
the myInstance variable and once to construct the instance itself. This feels like 
unnecessary work: the compiler knows that the type of myInstance is Example (or a superclass of Example) as we are binding a value of the Example type.

Scala, like most functional languages, uses type inference to allow the compiler to infer the type of variables from the instances bound to them. We would write the equivalent line in Scala as follows:

```scala
val myInstance = new Example()
```

The Scala compiler infers that myInstance has the Example type at compile time. A lot of the time, it is enough to specify the types of the arguments and of the return value of a function. The compiler can then infer types for all the variables defined in the body of the function. Scala code is usually much more concise and readable than the equivalent Java code, without compromising any of the type safety.

## Scala encourages immutability

Scala encourages the use of immutable objects. In Scala, it is very easy to define an attribute as immutable:

```scala
val amountSpent = 200
```

The default collections are immutable:

```scala
val clientIds = List("123", "456") // List is immutable
clientIds(1) = "589" // Compile-time error
```

Having immutable objects removes a common source of bugs. Knowing that some 
objects cannot be changed once instantiated reduces the number of places bugs 
can creep in. Instead of considering the lifetime of the object, we can narrow in 
on the constructor.

## Scala and functional programs

Scala and functional programs Scala encourages functional code. A lot of Scala code consists of using higher-order functions to transform collections. You, as a programmer, do not have to deal with the details of iterating over the collection. Let's write an occurrencesOf function that returns the indices at which an element occurs in a list:

```scala
def occurrencesOf[A](elem:A, collection:List[A]):List[Int] = {
 for { 
 (currentElem, index) <- collection.zipWithIndex
 if (currentElem == elem)
 } yield index
}

```

How does this work? We first declare a new list, collection.zipWithIndex, whose 
elements are (collection(0), 0), (collection(1), 1), and so on: pairs of the 
collection's elements and their indexes.

We then tell Scala that we want to iterate over this collection, binding the 
currentElem variable to the current element and index to the index. We apply 
a filter on the iteration, selecting only those elements for which `currentElem== elem`. We then tell Scala to just return the index variable.

We did not need to deal with the details of the iteration process in Scala. The syntax is very declarative: we tell the compiler that we want the index of every element equal to elem in collection and let the compiler worry about how to iterate over collection.

Consider the equivalent in Java:

````java
static <T> List<Integer> occurrencesOf(T elem, List<T> collection) {
 List<Integer> occurrences = new ArrayList<Integer>() ;
 for (int i=0; i<collection.size(); i++) {
 if (collection.get(i).equals(elem)) {
 occurrences.add(i) ;
 }
 }
 return occurrences ;
}
````

In Java, you start by defining a (mutable) list in which to put occurrences as you find them. You then iterate over the collection by defining a counter, considering each element in turn and adding its index to the list of occurrences, if need be. There are many more moving parts that we need to get right for this method to work. These moving parts exist because we must tell Java how to iterate over the collection, and they represent a common source of bugs.

Furthermore, as a lot of code is taken up by the iteration mechanism, the line that defines the logic of the function is harder to find:

```java
static <T> List<Integer> occurrencesOf(T elem, List<T> collection) {
 List<Integer> occurences = new ArrayList<Integer>() ;
 for (int i=0; i<collection.size(); i++) {
 if (collection.get(i).equals(elem)) { 
 occurrences.add(i) ;
 }
 }
 return occurrences ;
}
```

Note that this is not meant as an attack on Java. In fact, Java 8 adds a slew of 
functional constructs, such as lambda expressions, the Optional type that mirrors 
Scala's Option, or stream processing. Rather, it is meant to demonstrate the benefit of functional approaches in minimizing the potential for errors and maximizing clarity.

## Null pointer uncertainty

We often need to represent the possible absence of a value. For instance, imagine that we are reading a list of usernames from a CSV file. The CSV file containsname and e-mail information. However, some users have declined to enter their e-mail into the system, so this information is absent. In Java, one would typically represent the e-mail as a string or an Email class and represent the absence of e-mail information for a particular user by setting that reference to null. Similarly, in Python, we might use None to demonstrate the absence of a value.

This approach is dangerous because we are not encoding the possible absence 
of e-mail information. In any nontrivial program, deciding whether an instance 
attribute can be null requires considering every occasion in which this instance is 
defined. This quickly becomes impractical, so programmers either assume that a 
variable is not null or code too defensively.

Scala (following the lead of other functional languages) introduces the Option[T] type to represent an attribute that might be absent. We might then write the following:

```scala
class User {
 ...
 val email:Option[Email]
 ...
}
```

We have now encoded the possible absence of e-mail in the type information. It is 
obvious to any programmer using the User class that e-mail information is possibly absent. Even better, the compiler knows that the email field can be absent, forcing us to deal with the problem rather than recklessly ignoring it to have the application burn at runtime in a conflagration of null pointer exceptions.

All this goes back to achieving a certain level of provable correctness. Never using 
null, we know that we will never run into null pointer exceptions. Achieving the 
same level of correctness in languages without Option[T] requires writing unit tests on the client code to verify that it behaves correctly when the e-mail attribute is null.

Note that it is possible to achieve this in Java using, for instance, Google's 
Guava library (https://code.google.com/p/guava-libraries/wiki/UsingAndAvoidingNullExplained) or the Optional class in Java 8. It is more a matter of convention: using null in Java to denote the absence of a value has long 
been the norm.

## Easier parallelism

Writing programs that take advantage of parallel architectures is challenging. It is nevertheless necessary to tackle all but the simplest data science problems.

Parallel programming is difficult because we, as programmers, tend to think sequentially. Reasoning about the order in which different events can happen in a concurrent program is very challenging.

Scala provides several abstractions that greatly facilitate the writing of parallel code. These abstractions work by imposing constraints on the way parallelism is achieved. For instance, parallel collections force the user to phrase the computation as a sequence of operations (such as map, reduce, and filter) on collections. Actor systems require the developer to think in terms of actors that encapsulate the application state and communicate by passing messages.

It might seem paradoxical that restricting the programmer's freedom to write parallel code as they please avoids many of the problems associated with concurrency. However, limiting the number of ways in which a program behaves facilitates thinking about its behavior. For instance, if an actor is misbehaving, we know that the problem lies either in the code for this actor or in one of the messages that the actor receives.

As an example of the power afforded by having coherent, restrictive abstractions, 
let's use parallel collections to solve a simple probability problem. We will calculate the probability of getting at least 60 heads out of 100 coin tosses. We can estimate  this using Monte Carlo: we simulate 100 coin tosses by drawing 100 random Boolean values and check whether the number of true values is at least 60. We repeat this until results have converged to the required accuracy, or we get bored of waiting.

Let's run through this in a Scala console:

```
scala> val nTosses = 100
nTosses: Int = 100
scala> def trial = (0 until nTosses).count { i =>
 util.Random.nextBoolean() // count the number of heads
}
trial: Int
```

The trial function runs a single set of 100 throws, returning the number of heads:

```
scala> trial
Int = 51
```

To get our answer, we just need to repeat trial as many times as we can and 
aggregate the results. Repeating the same set of operations is ideally suited to 
parallel collections:

```
scala> val nTrials = 100000
nTrials: Int = 100000
scala> (0 until nTrials).par.count { i => trial >= 60 }
Int = 2745
```

The probability is thus approximately 2.5% to 3%. All we had to do to distribute the calculation over every CPU in our computer is use the par method to parallelize the range (0 until nTrials). This demonstrates the benefits of having a coherent abstraction: parallel collections let us trivially parallelize any computation that can be phrased in terms of higher-order functions on collections. Clearly, not every problem is as easy to parallelize as a simple Monte Carlo problem. However, by offering a rich set of intuitive abstractions, Scala makes writing parallel applications manageable.

## Interoperability with Java

Scala runs on the Java virtual machine. The Scala compiler compiles programs to 
Java byte code. Thus, Scala developers have access to Java libraries natively. Given the phenomenal number of applications written in Java, both open source and as part of the legacy code in organizations, the interoperability of Scala and Java helps explain the rapid uptake of Scala.
Interoperability has not just been unidirectional: some Scala libraries, such as the 
Play framework, are becoming increasingly popular among Java developers.

## When not to use Scala

In the previous sections, we described how Scala's strong type system, preference 
for immutability, functional capabilities, and parallelism abstractions make it easy to write reliable programs and minimize the risk of unexpected behavior.

What reasons might you have to avoid Scala in your next project? One important 
reason is familiarity. Scala introduces many concepts such as implicits, type classes, and composition using traits that might not be familiar to programmers coming from the object-oriented world. Scala's type system is very expressive, but getting to know it well enough to use its full power takes time and requires adjusting to a new programming paradigm. Finally, dealing with immutable data structures can feel alien to programmers coming from Java or Python.

Nevertheless, these are all drawbacks that can be overcome with time. Scala does 
fall short of the other data science languages in library availability. The IPython 
Notebook, coupled with matplotlib, is an unparalleled resource for data exploration. There are ongoing efforts to provide similar functionality in Scala (Spark Notebooks or Apache Zeppelin, for instance), but there are no projects with the same level of 
maturity. The type system can also be a minor hindrance when one is exploring data or trying out different models.

Thus, in this author's biased opinion, Scala excels for more permanent programs. If you are writing a throwaway script or exploring data, you might be better served with Python. If you are writing something that will need to be reused and requires a certain level of provable correctness, you will find Scala extremely powerful.
