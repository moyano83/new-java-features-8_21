# Master Java 8 to Java 21

You can manage the installation of the different versions of jdk using sdkman, available [here](https://sdkman.io/usage/). After installation of
sdk you can install the latest stable version using `sdk install java`, list the java jdks installed using `sdk list java | grep installed`.


## Java 8 declarative style

Instead of going in a for loop, you can declare how you want your operation to compute a result without the need of dealing with the how. i.e:

```java
int sum = java.util.stream.IntStream.rangeClosed(0, 100).sum(); // No mutable state thus thread safe
```


## Lambda

Synthax of Lambda in Java: `() -> {}` where `()` can have 0 or more parameters and `{}` is the body function. Their main purpose is to implement
the functional interfaces (SAM or single abstract method interfaces):

```java

@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
}

// The above can be implemented with:
Comparator<String> LambdaComparator = (s1, s2) -> s1.compareTo(s2); // No need to define the parameter types, the compiler can infer it 
```


## Functional interfaces

The 4 functional interfaces included in java 8 are: Consumer, Predicate, Function and Supplier (each one withs its extensions like BiConsumer,
BiPredicate...).

### Consumers

Operation that accepts an input and return no results, it is expected to work through side effects. Consumer defines two methods: `void accept(T t);`
and `default Consumer<T> andThen(Consumer<? super T> after)`. Consumers can be used in `foreach` methods where the accept method would be executed,
and you can chain consumer on those methods:

```java
List.of(1,2,3,4,5).foreach(consumer1.andThen(consumer2));
```

There is more versions of `Consumer<T>` for example `BiConsumer<T,U>` which accepts two parameters.

### Predicate

Evaluates a predicate on a given argument, has a method `boolean test(T t)`, it also has `and` and `or` methods to chain Predicates, i.e. 
`p1.and(p2).test(someInput)`. There is also different flavours of `Predicate`, like `BiPredicate` which accepts two arguments to the test function.

### Function

Similar to the above, a function interface represents a function, and has two main methods, `compose` and `andThen`. A function requires two types 
(input and return type), and can be invoked calling the `apply` method. `compose` executes the apply method of the function passed as a parameter 
first (fold right), as opposed to the `andThen` which executes the apply of the function to which we call the `andThen` method (fold left). Again 
there are different flavours, like the `BiFunction<T, U, R>`, which accepts two parameters (T and U). 

### Unary Operator and BinaryOperator

`UnaryOperator<T>` extends from `Function<T,T>` and defines a method `UnaryOperator<T> identity()`, and it is used when the function has the same 
type or the input and output. The `BinaryOperator<T>` extends from `BiFunction<T,T,T>` and represent a function that accepts two parameters of 
the same type and returns a value of the same type too. It also defines `minBy` and `maxBy` methods that accepts comparators to define which is 
the min/max of the two values using the provided comparator.

### Supplier
`Supplier<T>` is a functional interface that defines a method `T get()`, and it is used to provide values. It is opposite to the consumer interface.


## Constructor and method reference

Syntax of method reference:

    * ClassName :: instance-methodName
    * ClassName :: static-methodName
    * Instance :: methodName

So we can modify lambdas so instead of `Function<String, String> toUpper = (s) -> s.toUpperCase();` we can write 
`Function<String, String> toUpper = String :: toUpperCase;` in the case of inner classes we access the inner class normally, like in 
`static Consumer<String> printer = System.out::println;`. Constructors can be called in the same way like in `Supplier<MyClass> f = MyClass::new` 
but this code needs a parameterless constructor as supplier takes no input parameters (a constructor with one parameter can be passed as a 
`Function<T,U>`).


## Lambdas and Local variables

Any variable declared inside a method is a local variable, lambdas doesn't allow the same local variable than the lambda parameter name and a 
local variable can't be reassigned:

```java
int i = 0;
Consumer<Integer> c = (i) -> { // this won't compile because there is a variable on the scope named i
    i = 3; // this won't work as we are reassigning a variable on the lambda, value should be final or static (a.k.a. Effectively final)
    System.out.println(i);
};
```

## Streams API
    
Streams is  collection of elements that can be operated sequentially or in parallel. You can create a stream from an array like: 
`Arrays.asList(1,2,3).stream() # or call parallelStream()`. Streams have intermediate operations which returns another stream, and terminal 
operations which returns other type, like the return type of the Collector used to gather the results. Intermediate operations doesn't trigger 
computations, it will all occur when a terminal operation is called. Streams as the collections API, does not allow you to add or remove elements 
once is created, it is lazily constructed an can only traverse the elements in sequence (you can't access elements by passing an index for example).
Streams can only be traversed once (like an iterator), and this iteration is done using the Streams API as oppose as calling some sort of external 
operation like a for loop. Use the method `peek(Consumer<? super T>)` to print or inspect elements on the stream.

### Streams API operations

Summary of the most important streams operations:

    * .map(...): To transform elements
    * .flatMap(...): To transform and merge elements in different streams into a single one
    * .distinct(): Returns the list of distinct elements on the stream
    * .count(): Returns the count
    * .sorted(): Returns a stream with sorted elements by natural order (ascending)
    * .sorted(Comparator<? super T>): Returns a stream with sorted elements by the logic of the passed comparators
    * .filter(Predicate<T>): To discard elements from the Stream
    * .reduce(initialValue, BinaryOperator<T>): Terminal operation to reduce the contents of a Stream into a single element, if you don't pass the 
      initialValue, then the result is an Optional<T>
    * .limit(n): limits to n, the number of elements to process from the stream
    * .skip(n): skips the first n elements from the stream
    * .anyMatch(p): Returns true if any of the elements of the stream matches the predicate
    * .allMatch(p): Returns true if all elements of the stream matches the predicate
    * .noneMatch(p): Returns true if none of the elements of the stream matches the predicate, inverse to allMatch
    * .findFirst(): Returns the first element in the stream
    * .findAny(): Returns the first encountered element in the stream (uses parallel stream)

It is worth to notice the functions `limit`, `findFirst`, `findAny`, `anyMatch`, `allMatch` and `noneMatch` are short-circuited: they don't iterate 
the whole stream to evaluate the result if it is not needed.
Streams have an internal state, but not all the streams functions maintains an internal state. Example of stateful functions are: `distinct`, 
`sorted`, `skip` or `limit`. On the contrary, `map` and `filter` are stateless functions.

### Streams Factory Methods

    * of(): creates a stream of certain values passed to this method -> Stream.of(1,2,3)
    * iterate(): Used to create infinite streams -> Streams.iterate(1, x -> x * 2)
    * generate(): For infinite streams as well, but it takes a Supplier -> Streams.generate(Supplier<T>)

### Streams API: Numeric Streams

There are 3 types of numeric Streams: `IntStream`, `DoubleStream` and `LongStream` which represent the primitive values. These streams are more 
performant because they can help reduce the unboxing and boxing of primitive types, and also comes with a few handy methods like 
`IntStream.rangeClosed(1,6)` which produces numbers from 1 to 6 included (`range(1,6)` won't include the 6). It also has terminal operations like 
`sum()`, `max()`, `min()`, `average()`, `count()` and more. `DoubleStream` doesn't come with the range methods, but you can convert an `IntStream` 
to a `DoubleStream` using the `asDoubleStream()` method. You can convert a primitive stream to a wrapper class calling the `.boxed()` method (for 
example to convert from `int` to `Integer`), you can do the opposite by calling `stream().mapToInt(Integer::intValue)`.
Other important methods are `mapToObj` which converts a numeric element of the stream to an object, `mapToLong` which converts the numeric 
stream to a LongStream and `mapToDouble` which does it to a `DoubleStream`. 

### Streams API: Terminal Operations

Terminal operations start the while processing from a stream, the `Collect` method takes an input collector and returns the type of collector passed.

    * Joining: Concatenates the elements: `Stream.of("a", "b", "c").collect(Collectors.joining()); //yields "abc"`
    * Counting: Counts the total number of elements on the Stream `Stream.of("a", "b", "c").collect(Collectors.counting()); // yields 3`
    * mapping: Applies a transformation first and then collects the data: `theStream.collect(mapping(Studdent::name, Collectors.toList()))`
    * maxBy/minBy: Returns an optional with the max/min value based on a comparator passed
      `theStream.collect(Collectors.minBy(Comparators.comparing(Student::getAge)))`
    * summingInt(): returns the sum as result, there are type specific collectors: summingDouble `theStream.collect(Collectors.summingInt(Dog::age))`
    * averagingInt(): returns the average, there are type specific collectors: averagingDouble `theStream.collect(Collectors.averagingInt(Dog::age))`

#### Grouping By

equivalent to group by in SQL. The output is a `Map<K, V>`, there are several flavours. For example:

`theStream.collect(Collectors.groupingBy(Dog::getOwner)) // returns a Map<String, List<Dog>>`

But you can pass a lambda too:

`theStream.collect(Collectors.groupingBy(dog -> dog.getAge()>12 ? "OLD" : "YOUNG")`

It is also possible to pass a classifier:

`theStream.collect(Collectors.groupingBy(Dog::getOwner, Collectors.groupingBy(dog -> dog.getAge()>12 ? "OLD" : "YOUNG")))` which will return a map 
of maps. But we can pass another collector such as `summingInt()`.

The 3 argument version of grouping by acccepts a parameter to override the map type that is returned by default (a hashmap):

`theStream.collect(Collectors.groupingBy(Dog::getOwner, LinkedHashMap::new, Collectors.toList()) // return a LinkedHashMap<String, List<Dog>>`

We can also use `minBy` and `maxBy` with `groupingBy`, for example to calculate the oldest dog per owner:

`theStream.collect(Collectors.groupingBy(Dog::getOwner, Collectors.maxBy(Comparator.comparing(Dog::getAge)))) // maxBy returns an optional!`

We can avoid the odd return types with optional by using `collectingAndThen` which executes a function after the collect has happened:

`theStream.collect(groupingBy(Dog::getOwner, collectingAndThen(maxBy(Comparator.comparing(Dog::getAge))),Optional::get))`

#### partitioningBy

It is a kind of `groupingBy` that accepts a predicate as input and returns a `Map<K,V>`, also has multiple versions:

`theStream.collect(Collectors.partitioningBy(dog -> dog.getAge()>12) // returns Map<Boolean, List<Dog>>`

But also we can provide a downstream collector:

`theStream.collect(Collectors.partitioningBy(dog -> dog.getAge()>12, Collectors.toSet) // returns Map<Boolean, Set<Dog>>`

### Streams API: Parallel Processing

You can create parallel streams by calling the `.parallel()` method on a Stream. Parallel streams uses the Fork/Join framework introduced on java 
7, it creates a number of threads that matches the number of processors on the machine to compute the operations (`Runtime.getRuntime().
availableProcessors()`). There are cases were the parallelisation hurts the performance, for example if there are unboxing/boxing involved, or 
mutation of variable state as it might lead to race conditions.

### Optional

There are many ways to create an optional:

    * Optional.ofNullable(someValue): someValue might or might not be null
    * Optional.empty(): Optional object with no value
    * Optional.of(someValue): if someValue is null then it throws an exception

And also ways to deal with the value inside optional:

    * optional.get(): Gets the value and throws an exception if it is not present
    * optional.orElse(someValue): Gets the value or returns the value someValue 
    * optional.orElseGet(() -> someValue)): Gets the value or else calls the supplier to get the value
    * optional.orElseThrow(() -> ex): Gets the value or calls the function that throws the Exception ex

You can check if the optional contains a value and take some action with:

    * optional.ifPresent(consumer): passes the optional value to the consumer in case it is present 
    * optional.isPresent(): Returns a Boolean indicating if it contains a value

Other methods to deal with the value inside the optional:

    * optional.map(func): Executes func on the value contained in the optional (returns an optional)    
    * optionalOfOptional.flatMap(func): Executes func on the optional value contained in the optional and returns an optional value
    * optional.filter(func): Filters out the value that doesn't comply with the func which returns a boolean

### Default/Static methods in Interfaces

Interfaces now allow to define default implementations to methods so adding new methods to the interface doesn't break existing implementations. 
You need to start the method declaration with the `default` keyword, and can be overriden by implementing classes.
Static methods are similar to the default ones but can't be overridden. 

#### Sorting using comparators

Using the default implementation, List contains now a method called `sort(Comparator)`. Comparators can be chained with `comp1.andThen(comp2)`.
Another new feature is that we can handle nulls in comparisons with the methods `nullsFirst` or `nullsLast`: `Comparators.nullsFirst(comparator2)`.

#### Multiple inheritance

Because a class in java can implement several interfaces `class A implements I1, I2, I3`, and interfaces can extend from other interfaces 
`interface I2 extends I1` and `interface I3 extends I2`, if we chose to override the implementation of a method on an interface that extends from 
other we need a way to define the precedende. In the example used here if `I1` defines a method and `I2` overrides it, when `class A` calls that 
method, would it use the implementation from `I1` or `I2`? The answer is that the runtime will resolve the implementation of the child, in this 
case `I2`. If we have a class that implements two interfaces that has the same method signature and different default implementation but are not 
related between them, the code won't compile unless you override the method implementation in the class itself.

## New Date/Time APIs

Java 8 comes with a set of immutable classes to represent time:

    * LocalDate: Represents a date, you can get an instance with `LocalDate.now()` and returns the machine date i.e. `2025-11-03`
    * LocalTime: Represents a time, you can get an instance with `LocalDate.now()` and returns the machine date i.e. `08:31:12.654`
    * LocalDateTime: Represents a datetime, you can get an instance with `LocalDate.now()` and returns the machine date i.e. `2025-11-03T08:31:12.654`

There are other ways to get an instance of LocalDate like `LocalDate.of(2025, 11, 03)` or `LocalDate.ofYearDay(2025, 365)` the second parameter 
is the day of that year, in this case 31 dec.
How do you get values out of a `LocalDate`? Similar to Calendar `theDate.getMonth()` which returns NOVEMBER, but if you want the number of 
the month use `theDate.getMonthValue()` (similar to day and year values). `.getDayOfWeek()` returns MONDAY. 
Yet another way to get a specific field is by using `theDate.get(CronoField.DAY_OF_MONTH)`, explore the enums in `CronoField` to see the 
different values you can use.
To modify a date you can use methods such as `theDate.plusDays(13)` which returns a new instance of LocalDate (as the localDate is immutable). To 
substract you can do `minusDays(...)` or even `theDate.minus(1, CronoUnit.YEAR)`.
You can also select a value to set like `theDate.withYear(2019)` or even `theDate.with(CronoField.YEAR, 2019)`.
There are other handy ways to manipulate the date like `theDate.with(TemporalAdjusters.lastDayOfTheMonth())`. Adding or substracting a unit that 
is not supported would throw an exception, i.e. adding minutes to a Date... But you can check if the unit is supported with `thedate.isSupported
(CronoUnit.MINUTES)`. LocalTime and LocalDateTime has similar methods to LocalDate, for example you can create a LocalTime instance with 
`LocalTime.now()` or `LocalTime.of(....)`. in case of the `LocalDateTime`, you pass an instance of a localDate and one of LocalTime to the `of` 
method: `LocalDateTime.of(LocalDate.now(), LocalTime.of(20,00,00)`.

To convert a LocalDate and LocalTime to a LocalDateTime and viceversa we do:

```java
LocalDate.now().atTime(23,30,15);
LocalTime.now().atDate(LocalDate.now());
LocalDateTime.now().toLocalDate();
LocalDateTime.now().toLocalTime();
```

### Period class

Represents a period of time and it is compatible with LocalDate. Mainly used to calculate the difference between two dates: `Period.between(ld1,ld2)`.
The Period returned by the previous contains information that can be accessed with methods like `.getDays()`, `.getMonths()`, `.getYears()`. You 
can instantiate a Period with methods like `Period.ofYears(10)` and convert it later to months with `p.toTotalMonths()`. You can also get a 
period from a local date: `ld1.until(ld2)` and that would return a period between ld1 and ld2.

### Duration

Similar to Period, but compatible with `LocalTime` instead of `LocalDate`. Similarly you can access methods related with Hours, Minutes, Seconds...
There is a between method as well, but the `until` method from `LocalTime` accepts two parameters, first is the LocalTime to get the time until, 
and the second is the `ChronoUnit` desired: `lt1.until(lt2, ChronoUnit.Minutes)`. There are methods to create a duration from some unit, for example 
`Duration.ofMinutes(3)`.

### Instant

Represents the time in a machine readable format, represent the time in seconds from the epoch date. You can instantiate an Instant with the `now()` 
method, and get information back with the methods `getEpochSecond()`, `getNano()` or even `get(TemporalField)` or `getLong(TemporalField)`.

### TimeZones

Classes that represents times with a locality. We have classes like `ZonedDateTime` (represents the date time with zone information, `ZoneOffset` 
(offset from origin) and `ZoneId` identifier of the zone where the date is set. You can get the offset of id from a ZonedDateTime using the 
methods `getOffset()` or `getZone()`. You can get the list of available zones using the method `ZoneId.getAvailableZoneIds()`. To get a `ZonedDateTime`
you can use `ZonedDateTime.now(ZoneId.of("America/Detroit"))`. You can use another variant using 
`ZonedDateTime.now(Clock.system(ZoneId.of("America/Detroit")))`. You can get a local date time of a zone using `LocalDateTime.now(ZoneId.of(...))` 
but it won't contain any zoned information, just the local time.

### Converting Instant, LocalDateTime to ZonedDateTime

You can convert a LocalDateTime to a ZonedDateTime with `ldt.atZone(ZoneId.of(...))` which sets the current LocalDateTime to the zone passed. 
Similarly, the Instant class have an `atZone` method that accepts the same parameters. It is also possible to convert `LocalDateTime` to an class 
`OffsetDateTime` with the method `ldt.atOffset(ZoneOffset.ofHours(-3))`. The difference between an `OffsetDateTime` and a `ZonedDateTime` is that 
the former does not add any information about a zone id, only the offset difference.

### Converting java.util.Date or java.sql.Date to LocalDate

From a `util.Date` instance you can call the methods `toInstant().atZone(...).toLocalDate()` to get a LocalDateTime, or a LocalDate. To get a Date 
from an Instant, `Date` has a method `date.from(Instant)` and you can get the Instant from the LocalTime or LocalDate by chaining the methods 
`atZone(...)` and then `toInstant()`.
If we want to get a `sql.Date` instead, we can call the `sql.Date.valueOf(LocalDate)` method, or get a LocalDate from an sql date by calling the 
`sqlDate.toLocalDate()` method.

### Parse and formatting Dates

And example of how to parse a LocalDate with `LocalDate.parse(stringDate, DateTimeFormatter.BASIC_ISO_DATE)`, if the second parameter is not used 
then the default uses `DateTimeFormatter.ISO_LOCAL_DATE` which is `YYYY-MM-DD`. You can also define a custom DateTimeFormatter with 
`DateTimeFormatter.ofPattern("yyyyMMdd")`. To format a LocalDate we simply call the `ld.format(DateTimeFormatter.BASIC_ISO_DATE)` and it would 
return a formatted string. Similarly, we have equivalent methods to parse and format from and to a `LocalTime` and `Loca33lDateTime`.

# Java 9

From java 9 you can create a compact class which doesn't require a class name, and you can have a method `void main(){...}` on it. Also before you 
needed variables to be static to access them from the main method but that's not required anymore, reducing verbosity. Another addition is that 
public modifier in the static void method is no longer required, nor the args parameter.

## Local variable type inference using var
From java 10 you can use the `var` keyword to declare local variables, and the compiler will infer the type from the assigned value. For example:

```java
var list = List.of(1,2,3); // infers List<Integer>
for(var i: list){ // We can use var in loops
    System.out.println(i);
}
```

There are limitations in the use of var:

    * Can't be used to assign a null -> var x = null;
    * Can't change the type later, for example declare a string and then trying to assign a Long to it -> var x ="a"; x=5
    * Can't be used in class members 
    * You can't use it in function parameter definition
    * You can't use it in lambda definitions

### Textblocks

From java 13 you can define multiline strings like in scala, starting with tree double quotes and finishing again with the same.
```java
var txt = """
This is a text block with a name %s
No need to use \n
""".formatted(someName);
```

The indentation of the text is the same that the code, but you can change that using the method `stripIndent()`. 

### Enhanced Switch

Released in java 14. Switch is an expression now so it returns a value:

```java
// Before:
switch (someValue):
    case A:
        ...;
        break;
    case B:
        ...;
        break;
    default:
        ...;
        
//New Switch
var result = switch (someValue){
    case A, C -> result;
    case B -> otherResult;
    default -> {
        // in a block use yield to return a value
        yield someOtherResult;
    }
};
```

If the match is not exhaustive (all cases covered), then the code won't compile unless you provide a default case.

### Records

Records where introduced in java 16 as a way to reduce the boilerplate code needed to create data classes. A record is defined using the `record` 
keyword. Records are immutable by default, and the compiler generates the constructor, getters, `equals`, `hashcode` and `toString` methods. They 
are intended just to hold data and can't extends from other classes but can implement interfaces. Example of a record:

```java
public record Person(String name, int age){
    // You can define additional methods if needed
    public Person{
        // this is the canonical constructor, you can add validation logic here
        if(age < 0){
            throw new IllegalArgumentException("Age can't be negative");
        }
    }
}
```

What happens if you want to provide a custom constructor? You define one by anotating it with the access modifier and the record name and 
parameters required:

```java
public record Person(String name, int age){
    public Person(String name){
        this(name, 0); // calls the canonical constructor
    }
}
```

As part of the code generation, the equals compares the values of all record attributes, but you can have a different implementation of it by 
overriding the method:

```java
public record Person(String name, int age) {
    @Override
    public boolean equals(Object o) {
        // some custom implementation
    }
}
```

### Sealed Classes

Sealed classes were introduced in java 17 as a way to restrict which classes can extend or implement them. A sealed class is defined using the 
`sealed` keyword, and you need to provide the list of classes that can extend or implement it using the `permits` clause:

```java
public sealed class Vehicle permits Car, Motorbike {}
public final class Car extends Vehicle {}
public sealed class Motorbike extends Vehicle permits RaceMotorbike{}
public non-sealed class Boat extends Vehicle {}
public class Submarine extends Boat {} // Permitted as Boat is non-sealed
```

The compiler needs to guarantee that all subclasses are known at compile time, so the permitted classes must be in the same module or package as 
the sealed class. It also needs to make sure there is no further extension of the permitted classes, so they must be declared as `final`, `sealed` 
or `non-sealed`. A `final` class can't be extended, a `sealed` class can be extended but must define its own permitted classes, and a`non-sealed` 
class can be extended by any class. You can define abstract sealed classes and provide abstract method for the subclasses to implement. You can 
also define sealed interfaces, which works the same way as sealed classes but for interfaces.

### Pattern Matching

Pattern matching for `instanceoof` reduces boilerplate to check for a type and cast the object to that type. There is 2 versions:

```java
// From java 16
if(someObj instanceof String){
    String s = (String) someObj;        
    System.out.println("It's a string: " + s);
} else if (someObj instanceof Integer){ 
    Integer i = (Integer) someObj;
    System.out.println("It's an integer: " + i);
} else {
    System.out.println("Unknown type");
}
```

Or in java 21:

```java
return switch(someObj) {
    case String s -> "It's a string";
    case Integer i -> "It's an integer";
    case null, default -> "Unknown type";
};
```

It is also possible to use pattern matching with records:

```java
return switch(someObj) {
    case Person(var name, var age) when age >= 18 -> name + " is an adult"; // when allows to add extra conditions this is called guarded pattern
    case Person(String name, int age) -> name + " is a minor";
    case OtherPerson(var id) -> "Its another person"
    case null, default -> "Unknown type";
};
```

### Unamed variables

From java 21 you can use the `_` symbol to define unamed variables in patterns, for example if you don't care about a value in a record:

```java
return switch(someObj) {
    case Person(var name, _) -> name + " is a person"; // we don't care about the age
    case OtherPerson(_) -> "Its another person" // we don't care about the id so we don't capture it
}
```

There is a bunch of places where you can use unnamed variables:

    * In record patterns to ignore certain fields -> `case Person(_, age) -> {...}`
    * In array patterns to ignore certain elements -> `case int[] {_, secondElement, _} -> {...}`
    * In type patterns to ignore the casted variable -> `if(obj instanceOf Dog(_)) {...}`
    * In exception handling to ignore the exception variable -> `catch (Exception _) {...}`
    * In Lambda parameters to ignore certain parameters -> `( _, b) -> b + 1`

Note that you can't use the `_` symbol as a variable name in your code as it is reserved for unamed variables. 

### Stream Gatherers

From java 21 there is a new way to collect stream results using gatherers, which are more flexible than collectors. They bring powerful 
transformation capabilities to Streams and allows custom intermediate operations like windowing, scanning, folding, concurrent mapping, batching, 
and partitioning. Example of usage:

```java
var result = someStream.collect(gathering(toList(), // downstream collector
                                batching(3), // batches of 3 elements
                                filtering(list -> list.size() == 3) // only lists of size 3
                            ));
```

This gatherers were developed as the previous collectiors had limitations like being pull-based, not supporting stateful intermediate operations, 
and not being able to handle concurrent processing well. The new gatherers are push-based, support stateful operations, and can handle concurrent 
processing more effectively.

```java
movies.stream().gather(Gatherers.windowFixed(3)) //returns Stream<List<Movie>>
        .foreach(window -> {
            window.forEach(movie -> System.out.println(movie.title()));
        });
```

The difference between gatherers and collectors is that gatherers can handle stateful intermediate operations, support push-based processing, and 
can manage concurrent processing more effectively. Collectors are pull-based and have limitations with stateful operations and concurrency.

#### Built-in gatherers

##### windowSliding

Produces overlapping windows of specified size and step. For example if we have a window `[A,B,C,D]` and we apply a sliding of 2 with a step of 
one we get `[A,B], [B,C], [C,D]`. It is useful to calculate moving averages or trends.

##### fold

Performs stateful reduction on the stream elements, similar to `reduce` but used within the pipeline. The syntax is ` fold(Supplier<A>, 
BiFunction<A, T, A>)` where A is the accumulator type and T the stream element type.

```java
var totalDuration = movies.stream().gather(Gatherers.fold(
        () -> 0, // initial value (supplier)
        (acc, movie) -> acc + movie.duration() // accumulator function T is movie here and A is an Integer
    )).findFirst()
    .orElse(0);
```

##### scan

Similar to fold but emits intermediate results, it can be used for running totals, averages or progress tracking. You can think of it as a fold 
with visibility and the signature is the same than fold.

```java
var totalDuration = movies.stream().gather(Gatherers.scan(
        () -> 0, // initial value (supplier)
        (acc, movie) -> acc + movie.duration() // accumulator function T is movie here and A is an Integer
    )).forEach(runningTotal -> System.out.println("The total so far is: " + runningTotal + " minutes"))
    .findFirst()
    .orElse(0);
```

##### mapConcurrent

Enables concurrent transformations and process streams in parallel with a defined concurrency level.

```java
movies.stream().gather(Gatherers.mapConcurrent(2, movie -> process(movie)));
```

##### Building custom Gatherers

To build your own gatherer you can use `Gatherer.of(...)` an example of the usage can be seen below:

```java
Gatherer<Movie, Void, String> gatherer=Gatherer.of( // Void represents the state type (we don't use it so we use Void)
    Gatherer.Integrator.ofGreedy((state, movie, downstream) -> {
        if(movie.rating() >=8) { 
            String summary = movie.title + movie.duration.toString();
            downstrea.push(summary); // we need this to push the data to the final result
        }
        return true;
    })
);
```

##### Domain-Specific Logic using Gatherer.ofSequential

Imagine we want to implement a grouping and sorting operations using gatherers. We can create a gatherer that groups movies by genre and sorts 
them by rating within each group.

```java 
Gatherer<Movie, Map<String, List<Movie>>, Map<String, List<Movie>>> genreSorterGatherer = Gatherer.ofSequential(
    HashMap::new, // Initial state: empty map
    (state, movie, downstream) -> { // This is the integrator which process each movie
        state.computeIfAbsent(movie.genre(), k -> new ArrayList<>()).add(movie); // Grouping by genre
        return true; // Continue processing
    },
    (state, downstream) -> { // This is the custom gatherer, which operates over the intermediate state Map<String, List<Movie>> to produce the result
        state.forEach((genre, movies) -> {
            movies.sort(Comparator.comparingDouble(Movie::rating).reversed()); // Sorting by rating
            downstream.push(genre, movies); // Pushing grouped and sorted data
        });
    }
);
```

### Simple web server

Available from java 18, allows you to create a simple web server which server files and folders from your machine. It supports GET and HEAD only. 
We can launch the webserver like this:

```shell
jwebserver --port 8080 --root /path/to/serve 
```

This will serve the files in the root folder `/path/to/serve` at port 8080.

### New HTTP Client

Introduced in java 11, the new HTTP client support HTTP/2 and Websockets and has built in support for asynchronous and asynchronous operations. Example of a synchronous GET request:

```java 
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .connectionTimeout(10, TimeUnit.SECONDS)
    .uri(URI.create("https://api.example.com/data"))
    .build();

private final ObjectMapper objectMapper = new ObjectMapper().registerModule(new JavaTimeModule())
        .configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
// Synchronous request, the second argument specifies the handler for the response. You can also use sendAsync for asynchronous requests which 
// returns a CompletableFuture
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());  
System.out.println("Response status code: " + response.statusCode()); 
System.out.println("Response body: " + objectMapper.readValue(response.body(), Movie.class)); // Deserializes the object into a Movie instance
```

#### Asynchronous client

An example of an asynchronous request using the http client:

```java 
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .connectionTimeout(10, TimeUnit.SECONDS)
    .uri(URI.create("https://api.example.com/data"))
    .build();

private final ObjectMapper objectMapper = new ObjectMapper().registerModule(new JavaTimeModule())
        .configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false); 
CompletableFuture<List<Movie>> response = client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
        .thenApply(stringHttpResponse ->{
            try {
                return objectMapper.readValue(stringHttpResponse.body(), new TypeReference<>(){}); // We need this to parse into the List of Movies
            } catch (JsonProcessingException e) {
                throw new RuntimeException(e);
            }
        });
```

The response needs to be extracted from the `CompletableFuture` instance, for example by calling the `get()` method or chaining more operations 
using `thenApply`, `thenAccept`, etc.

### Java Platform Module System (JPMS) or project jigsaw

Introduced in Java 9, it allows to modularize your application into smaller parts called modules. Each module can declare which other modules it 
requires and which packages it exports for other modules to use. The benefits of this modularization are:

    * Better encapsulation: Modules can hide their internal implementation details and only expose the necessary APIs.
    * Smaller jar files: Reduce the process footprint and improve the application startup time.
    * Improved maintainability: Modules can be developed, tested, and deployed independently, making it easier to manage large codebases.
    * Reduced complexity: By breaking down an application into smaller modules, it becomes easier to understand and reason about the code.
    * Enhanced performance: The module system allows for better optimization of the application by only loading the necessary modules at runtime.
    * Clean separation of boundaries and stricter access control

A module is defined by creating a file named `module-info.java` in the root of the module source folder. Inside this file we define the module name,
the required modules and the exported packages. Example of a module definition:

```java
module com.example.myapp {
    requires java.sql; // Requires the java.sql module
    requires com.example.utils; // Requires another custom module
    exports com.example.myapp.api; // Exports the package com.example.myapp.api for other modules to use
    exports com.example.myapp.services;
}
```

To compile a module we need to specify the module path using the `--module-path` option and the source path using the `--source-path` option. 
Imagine you need to compile a module C which depends from another module B and which as well depends on a third module A. To define these sort of 
transitive dependencies where Module C requires Module A, we can use the `requires transitive` clause in the module B definition:

```java
module moduleB {
    requires transitive moduleA; // Module A will be available to modules that require module B
    exports moduleB.api;
}
```

#### Unnamed modules and automatic modules

If you have legacy code that is not modularized, you can still use it in a modularized application as it will be imported as an unnamed module. 
You can access the name of the module from a class using `SomeClass.class.getModule().getName()`.
All the packages in the unnamed module are accessible to all other modules, but the unnamed module can't access packages from other modules unless 
they are exported.
We can run an unnamed module as usual with `java -cp app.jar com.example.Main` or we can run a modularized application with 
`java --module-path mods -m moduleName/com.example.Main` This command will create something called automatic module and pass the `moduleName` as 
the module name to all the jars in the module path.
