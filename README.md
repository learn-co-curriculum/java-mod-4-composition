# Composition

## Learning Goals

- Explain function composition.
- Use different methods for composing functions.

## Introduction

Function composition is a method of combining functions to create new ones. We
can use this method for composing functions, operators, predicates, and consumers.

## Function and Operator Composition

The `Function<T, R>` interface provides the `compose` and the `andThen` methods
for composing new functions. Let’s look at an example to understand the
difference between the two:

```java
class Example {
    public static void main(String[] args) throws Exception {
        Function<Integer, Integer> addByFive = x -> x + 5;
        Function<Integer, Integer> mulByTwo = x -> x * 2;

        Function<Integer, Integer> composeFunc = addByFive.compose(mulByTwo);
        Function<Integer, Integer> andThenFunc = addByFive.andThen(mulByTwo);

        // compose: addByFive(mulByTwo(6))
        System.out.println("result: " + composeFunc.apply(6));
        // result: 17

        // andThen: mulByTwo(addByFive(6))
        System.out.println("result: " + andThenFunc.apply(6));
        // result: 22
    }
}
```

We have created two composed functions, `composeFunc` and `andThenFunc`, using
the `compose` and `andThen` method respectively.

The new function composed with the `compose` method applies the argument
starting with the rightmost function and passing the result to the previous
function. In mathematical terms, `a.compose(b).apply(i)` is the same as
`a(b(i))`.

The function composed using `andThen` starts applying the argument starting with
the leftmost function and then passing the result to the next function. In this
case, `a.andThen(b).apply(i)` is the same as `b(a(i))`.

We can form composite functions with more than two functions.

```java
Function<Integer, Integer> composeFunc = addByFive.compose(mulByTwo).compose((i) -> i * 10);
System.out.println("result: " + composeFunc.apply(6)); // result: 125
```

Here’s how the function takes the argument and returns the final result:

- `6 * 10` = `60`
- `60 * 2` = `120`
- `120 + 5` = `125`

The operator interfaces such as
[UnaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/UnaryOperator.html)
and
[BinaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BinaryOperator.html)
have similar methods for composing functions.

### Why  both `compose` and `andThen`?

If `addByFive.compose(mulByTwo)` produces the same value as `mulByTwo.andThen(addByFive)`, why do we have both methods?

| `Function<T, R>` composition           | Equivalent composition   | Algorithm     | Result  |
|----------------------------------------|--------------------------|---------------|---------|
| `addByFive.compose(mulByTwo).apply(6)` | `addByFive(mulByTwo(6))` | `5 + (2 * 6)` | 17      |
| `mulByTwo.andThen(addByFive).apply(6)` | `addByFive(mulByTwo(6))` | `5 + (2 * 6)` | 17      |

People who are used to mathematical functions may 
find `addByFive.compose(mulByTwo)`
easier to understand as equivalent to `addByFive(mulByTwo(....))`,
while people who are new to functional programming may
find `mulByTwo.andThen(addByFive)` easier
since it mimics the order you would use in an
English sentence (multiply by two and then add five).

Java provides both methods to support different programmer 
preferences for expressing function composition.

## Predicate Composition

All predicate functional interfaces have the `and`, `or`, and `negate` for
composing new predicate functions. They work similar to the boolean logical
operators in Java but with functions instead of values:

- `and` works like `&&`.
- `or` works like `||`.
- `negate` works like `!`.

```java
import java.util.function.Predicate;

class Example {
    public static void main(String[] args) throws Exception {
        Predicate<Integer> isEven = num -> num % 2 == 0;
        Predicate<Integer> greaterThan10 = num -> num > 10;

        Predicate<Integer> isOdd = isEven.negate();
        Predicate<Integer> isEvenAndGreaterThanTen = isEven.and(greaterThan10);
        Predicate<Integer> isOddOrLessThanTen = isOdd.or(greaterThan10.negate());

        System.out.println(isOdd.test(5)); // true
        System.out.println(isEvenAndGreaterThanTen.test(8)); // false
        System.out.println(isOddOrLessThanTen.test(5)); // true
    }
}
```

## Consumer Composition

Consider the following code, which creates two `Consumer` functions.
A consumer takes one argument and has a void return type. 
The `accept` method is called for each consumer.

```java
class Example1 {
    public static void main(String[] args) throws Exception {
        Consumer<String> consumer1 = s -> System.out.println("First consumer prints " + s);
        Consumer<String> consumer2 = s -> System.out.println("Second consumer prints " + s);

        consumer1.accept("hello");
        consumer2.accept("hello");
    }
}
```

The program prints:

```text
First consumer prints hello
Second consumer prints hello
```

The `Consumer` functional interface provides the `andThen` method
to compose functions that take in an argument and call in left to right
sequence all the functions in the composition chain with that argument.

The composition does not pass the result of one consumer
to the next in the chain, since consumers do not return a value.
Rather, each function in the composition is passed the same argument.

```java
import java.util.function.Consumer;

class Example2 {
    public static void main(String[] args) throws Exception {
        Consumer<String> consumer1 = s -> System.out.println("First consumer prints " + s);
        Consumer<String> consumer2 = s -> System.out.println("Second consumer prints " + s);
        Consumer<String> composedConsumer = consumer1.andThen(consumer2);

        composedConsumer.accept("hello");
    }
}
```

Consumer composition allows us to output values to different systems such as a database or logger.

```java
import java.util.function.Consumer;

public class Example3 {
    public static void main(String[] args) throws Exception {
        Consumer<String> consumer1 = s -> System.out.println("First consumer prints " + s);
        Consumer<String> composedConsumer = consumer1.andThen(Logger::log);
        composedConsumer.accept("hello");
    }
}

class Logger {
    public static<T> void log(T value) {
        System.out.println("Second consumer prints " + value);
    }
}
```


## Summary

We can use function composition to create new functions using existing ones.
This makes creating complex functions easier while also making them more
maintainable since the logic is broken down into simpler functions.
