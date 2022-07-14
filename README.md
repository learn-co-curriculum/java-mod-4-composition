# Composition

## Learning Goals

- Explain function composition.
- Use different methods for composing functions.

## Introduction

Function composition is a method of combining functions to create new ones. We
can use this method for composing functions, operators, predicates, and
consumers.

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

        // compose: addByFive(mulByTwo(5))
        System.out.println("result: " + composeFunc.apply(5));
        // result: 15

        // andThen: mulByTwo(addByFive(5)
        System.out.println("result: " + andThenFunc.apply(5));
        // result: 20
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
Function<Integer, Integer> composeFunc = addByFive
																					.compose(mulByTwo)
																					.compose((i) -> i * 10);
System.out.println("result: " + composeFunc.apply(5)); // result: 105
```

Here’s how the function takes the argument and returns the final result:

- `5 * 10` = `50`
- `50 * 2` = `100`
- `100 + 5` = `105`

The operator interfaces such as
`[UnaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/UnaryOperator.html)`
and
`[BinaryOperator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/BinaryOperator.html)`
have similar methods for composing functions.

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

Consumers provide the `andThen` method to compose functions that takes in an
argument and calls all the functions in the composition chain with that
argument. This allows us to output values to different systems such as a
database or logger.

```java
import java.util.function.Consumer;

class Example {
    public static void main(String[] args) throws Exception {
        Consumer<String> consumer = System.out::println;
        Consumer<String> composedConsumer = consumer.andThen(Logger::log);
        composedConsumer.accept("500");
    }
}

class Logger {
    public static<T> void log(T value) {
        System.out.println("The value is " + value);
    }
}
```

## Conclusion

We can use function composition to create new functions using existing ones.
This makes creating complex functions easier while also making them more
maintainable since the logic is broken down into simpler functions.
