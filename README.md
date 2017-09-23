# Notes on "Functional Programming Principles in Scala"

## Overview
These are the notes I took while participating Martin Odersky's Coursera course "Functional Programming Principles in Scala". This course can be found at https://www.coursera.org/learn/progfun1/.

## Credit
All the credit goes to the author of the course, Prof. Dr. Martin Odersky ([@odersky](https://github.com/odersky)).

## Week 1 -- Functions & Evaluation

### Lecture 1.1 -- Programming Paradigms

- Scala is a great tool to learn functional programming and later integrate it with more traditional object-oriented programming.
- Main programming paradigms are imperative, functional and logic programming. Object-oriented programming can be integrated well with all of those.
- Imperative programming is about mutable variables, assignments, and control structures (e.g., `if-then-else`).
- These strongly correspond to the abstractions of the Von Neumann model: Mutable variables correspond to memory cells, control structures to jumps, etc.
- Problem: How can we avoid conceptualizing programs word by word? How to scale up?
- A mathematical theory consists of data types, operations on them, and laws describing the relationship between values and operations. There is no notion of mutability whatsoever.
- Theories lead to a programming style in which operators are expressed as functions, mutations are avoided, and in which there are powerful ways for the abstraction and composition of functions.
- In a restricted sense, functional programming means programming without mutable variables and imperative control structures. In a wider sense, it means focusing on functions.
- In a functional programming language, functions are first-class citizens.
  - They can be defined anywhere.
  - They can be passed as parameters and returned as results.
  - There are operators to compose them.
- Why functional programming?
  - It's simpler to reason about programs.
  - There is better modularity.
  - It's good for exploiting parallelism.

### Lecture 1.2 -- Elements of Programming

+ Every non-trivial languages provides primitive expressions, ways to combine those into larger expressions, and ways to abstract expressions, i.e., naming them.

+ Evaluation of a non-primitive expression:

  1. Consider the leftmost operator. (This is subject to precedence rules.)
  2. Evaluate the operator's operands. (This is usually done left before right.)
  3. Apply the operator to the operands.

  The evaluation of a name is simply replacing it with the right-hand side of its definition. The process of evaluation comes to an end once a value is yielded, e.g., a number.

+ Evaluation of a function application (call-by-value):

  1. Evaluate the arguments, from left to right.
  2. Replace the application with the functions right-hand side (it's body), and, at the same time, replace the formal parameters with the evaluated arguments.

+ This scheme is known as the substitution model. The idea is that evaluation simply reduces an expression to a value. This can be done as long as the expression does not contain side effects.

+ The substitution model is formalized in the lambda calculus.

+ Not every expression reduces to a value in a finite number of steps:

  ````scala
  def loop: Int = loop
  loop
  ````

+ As an alternative, one could change the evaluation of function application by replacing the formal parameters in the function's body with un-reduced arguments (call-by-name).

+ Call-by-value and call-by-name reduce to the same values as long as the reduced expression consists of pure functions and both expressions terminate.

+ Call-by-value's advantage is that each argument is only evaluated once, call-by-name's advantage is that unused arguments are not evaluated.

### Lecture 1.3 -- Evaluation Strategies and Termination

+ If call-by-value evaluation of an expression e terminates, then call-by-name evaluation of e terminates, too. This is not true in the other direction.

+ The following function application only terminates with call-by-name:

  ````scala
  def first(x: Int, y: Int) = x
  first(1, loop) // Using the definition of `loop` from above
  ````

+ Scala uses call-by-value by default, as it is often exponentially more efficient. However, when a parameter type is prefixed with `=>`, call-by-name is used.

  ````scala
  def constOne(x: Int, y: => Int) = 1
  constOne(1 + 2, loop) // Works
  ````

### Lecture 1.4 -- Conditionals and Value Definitions

+ Unlike Java, Scala's `if-then-else` is not a statement but an expression.

  ````scala
  def abs(x: Int) = if (x >= 0) x else -x
  ````

+ Consider Boolean expressions such as `true && e` . `&&` and `||` don't always need their right operand to be evaluated. They use "short-circuit evaluation".

+ Definitions can be "by-name" and "by-value", too. In the former case, the right-hand side is evaluated on each invocation, in the latter case, the right-hand side is evaluated once, at the point of definition.

  ````scala
  def x = 2 // "by-name"
  val x = 2 // "by-value"
  ````

+ Exercise: Write functions `and` and `or` such that for all argument expressions `x` and `y` 

  ````scala
  and(x, y) == x && y
  or(x, y)  == x || y
  ````

  holds true. Do not use `&&` and `||` in your implementation.

  ````scala
  def and(x: Boolean, y: => Boolean) =
    if (x) y else false
  ````

  `or` is omitted for brevity.

### Lecture 1.5 -- Example: Square Roots with Newton's Method

+ In the following, note that `sqrtIter` is recursive, its right-hand side calls itself. Recursive functions need an explicit return type in Scala.

+ Here is the source code of the example:

  ````scala
  def abs(x: Double) = if (x > 0) x else -x

  def sqrtIter(guess: Double, x: Double): Double =
    if (isGoodEnough(guess, x)) guess
    else sqrtIter(improve(guess, x), x)

  def isGoodEnough(guess: Double, x: Double) =
    abs(guess * guess - x) / x < 0.001

  def improve(guess: Double, x: Double) =
    (guess + x / guess) / 2

  def sqrt(x: Double) =
   sqrtIter(1.0, x)
  ````

### Lecture 1.6 -- Blocks and Lexical Scope

+ We want to split up a task into many small functions while avoiding "namespace pollution". With regards to the previous example, we can do so by putting the auxiliary function inside of `sqrt`.

  ````scala
  def sqrt(x: Double) = {
    def sqrtIter(guess: Double, x: Double): Double =
      if (isGoodEnough(guess, x)) guess
      else sqrtIter(improve(guess, x), x)

    def isGoodEnough(guess: Double, x: Double) =
      abs(guess * guess - x) / x < 0.001

    def improve(guess: Double, x: Double) =
      (guess + x / guess) / 2

    sqrtIter(1.0, x)
  }
  ````

+ A block is delimited by braces. It contains a sequence of definitions and/or expressions. The last element in it is an expression defining the block's value. Blocks are themselves expressions.

+ Definitions inside a block are only visible from inside it. They shadow definitions from outside it.

+ The value of `result` is `16`.

  ````scala
  val x = 0
  def f(y: Int) = y + 1
  val result = {
    val x = f(3)
    x * x
  } + x
  ````

+ Since definitions of outer blocks are visible as long as they are not shadowed, we can further simplify `sqrt`:

  ````scala
  def sqrt(x: Double) = {
    def sqrtIter(guess: Double): Double =
      if (isGoodEnough(guess)) guess
      else sqrtIter(improve(guess))

    def isGoodEnough(guess: Double) =
      abs(guess * guess - x) / x < 0.001

    def improve(guess: Double) =
      (guess + x / guess) / 2

    sqrtIter(1.0)
  }
  ````

+ In Scala, semicolons are optional in most cases. However, multiple statements on one line must be separated by semicolons. Multi-line expressions containing infix operators should either be put into parentheses or intermediate lines should end with the operator, signaling the compiler that the expression is not yet finished.

### Lecture 1.7 -- Tail Recursion

+ Consider the following definition of Euclid's algorithm `gcd` to compute the greatest common divisor of two numbers as well as the evaluation of `gcd(14, 21)`:

  ````scala
  def gcd(a: Int, b: Int): Int = 
    if (b == 0) a else gcd(b, a % b)

  gcd(14, 21) // is evaluated as follows:
  // if (21 == 0) 14 else gcd(21, 14 % 21)
  // if (false) 14 else gcd(21, 14 % 21)
  // gcd(21, 14 % 21)
  // gcd(21, 14)
  // if (14 == 0) 21 else gcd(14, 21 % 14)
  // ...
  // gcd(14, 7)
  // ...
  // gcd(7, 0)
  // if (0 == 0) 7 else gcd(0, 7 % 0)
  // 7
  ````

+ To the contrary, consider the following definition of `factorial` as well as the evaluation of `factorial(4)`:

  `````scala
  def factorial(n: Int): Int =
    if (n == 0) 1 else n * factorial(n - 1)

  factorial(4) // is evaluated as follows:
  // if (4 == 0) 1 else 4 * factorial(4 - 1)
  // ...
  // 4 * factorial(3)
  // ...
  // 4 * (3 * factorial(2))
  // ...
  // 4 * (3 * (2 * factorial(1)))
  // ...
  // 4 * (3 * (2 * (1 * factorial(0)))
  // ...
  // 4 * (3 * (2 * (1 * 1)))
  // ...
  // 120
  `````

+ If a function's last action is to call itself, its stack frame can be used again. This is called tail recursion. Unlike `factorial`, `gcd` is tail recursive.

+ If a function's last action is calling a function (maybe itself), one stack frame would be sufficient for both. Those calls are called tail calls.

+ In Scala, only tail calls to the current function are optimized. This requires the `@tailrec` annotation.

  ````scala
  @tailrec
  def gcd(a: Int, b: Int): Int =
    // ...
  ````

+ Here is a tail-recursive version of `factorial`:

  ````scala
  def factorial(n: Int): Int = {
    def loop(acc: Int, n: Int): Int = 
      if (n == 0) acc
      else loop(acc * n, n - 1)
    loop(1, n)
  }
  ````

## Week 2 -- Higher-Order Functions 

### Lecture 2.1 -- Higher-Order Functions

+ Functional programming languages treat functions as "first-class values". This means that they can be received as arguments and returned as results. "Higher-order functions" are functions that do just that.

+ `sum` is an example of a function that takes another function as an argument.

  ````scala
  def sum(f: Int => Int, a: Int, b: Int): Int =
    if (a > b) 0
    else f(a) + sum(f, a + 1, b)

  def sumInts(a: Int, b: Int) =       sum(id, a, b)
  def sumCubes(a: Int, b: Int) =      sum(cube, a, b)
  def sumFactorials(a: Int, b: Int) = sum(fact, a, b)

  def id(x: Int): Int = x
  def cube(x: Int): Int = x * x * x
  def fact(x: Int): Int = if (x == 0) 1 else fact(x - 1)
  ````

+ `A => B` denotes the type of a function that has a parameter of type `A` and returns a `B`. Therefore, the type of `sum` is `Int => Int`.

+ As it is sometimes tedious to define and name functions using `def`, we would like to write them like literals -- without giving them a name. Anonymous functions enable this. Here is an example:

  ````scala
  (x: Int, y: Int) => x + y
  ````

  The type of a parameter can be omitted if it is inferred by the compiler.

+ An anonymous function `(x1: T1, ..., xn: Tn) => E` can always be expressed as `def f(x1: T1, ..., xn ... Tn) = E; f`. Therefore, anonymous functions are said to be syntactic sugar.

+ `sumInts` and `sumCubes` can be shortened using anonymous functions:

  ````scala
  def sumInts(a: Int, b: Int)  = sum(x => x, a, b)
  def sumCubes(a: Int, b: Int) = sum(x => x * x * x, a, b)
  ````

### Lecture 2.2 -- Currying

+ Note that above, `sumInts` and `sumCubes` simply pass `a` and `b` through. We can do better:

  ````scala
  def sum(f: Int => Int): (a: Int, b: Int) => Int = {
    def sumF(a: Int, b: Int): Int = 
      if (a > b) 0
      else f(a) + sumF(a + 1, b)
    sumF
  }

  def sumInts  = sum(x => x)         // Exemplary application: sumInts(1, 10)
  def sumCubes = sum(x => x * x * x)
  ````

  `sum` is now a function that returns another function.

+ The "middlemen" `sumInts` and `sumCubes` can be avoided. We can simply call `sum(cube)(1, 10)`, where `sum(cube)` returns the sum of cubes function and is therefore equivalent to `sumCubes`. Then the resulting function is applied to `(1, 10)`.

+ Function application associates to the left.

  ````
  sum(cube)(1, 10) == (sum(cube))(1, 10)
  ````

+ This way of defining functions is so useful, it got its own syntax in Scala. The following definitions of `sum` is equivalent to the previous one using `sumF`:

  ````scala
  def sum(f: Int => Int)(a: Int, b: Int): Int = 
    if (a > b) 0 else f(a) + sum(f)(a + 1, b)
  ````

+ The expansion of multiple parameter lists is formalized like this: A function `def f(args1)...(argsn) = E` is equivalent to a function `def f(args1)...(argsn-1) = { def g(argsn) = E; g }`, or, for short, `def f(args1)...(argsn-1) = (argsn) => E`.

+ By repeating the process n times, `def f(args1)...(argsn-1)(argsn) = E` is shown to be equivalent to `def f = (args1 => (args2 => ... (argsn => E) ... ))`.

+ This style of function definition and applications is called "currying", named after the logician Haskell Brooks Curry.

+ The type of `def sum(f: Int => Int)(a: Int, b: Int): Int = ...` is `(Int => Int) => (Int, Int) => Int`.

+ Function types associate to the right.

  ````scala
  // Int => Int => Int == Int => (Int => Int)
  ````

### Lecture 2.3 -- Example: Finding Fixed Points

+ A number is called a fixed point of a function `f` if `f(x) = x`.  For example, `2` is a fixed point of `x => 1 + (x / 2)^2` as `1 + (2 / 2)^2` is `2`.

+ For some functions `f`, we can locate fixed points by starting with an initial estimate `x` (e.g., `1`) and then repeatedly applying `f` to it until the results do not vary anymore. This works for the previously mentioned function.

  ````
  x, f(x), f(f(x)), ...
  ````

+ This leads to this function for finding a function's fixed point:

  ````scala
  val tolerance = 0.0001
  def isCloseEnough(x: Double, y: Double) =
    abs((x - y) / x) / x < tolerance
  def fixedPoint(f: Double => Double)(firstGuess: Double) = {
    def iterate(guess: Double): Double = {
      val next = f(guess)
      if (isCloseEnough(guess, next)) next
      else iterate(next)
    }
    iterate(firstGuess)
  }
  ````

+ Here is a specification of the `sqrt` function: 

  + `sqrt(x)` = the number `y` so that `y * y = x`
  + By dividing both sides of `y * y = x` with `y`, we get `y = x / y`.
  + Therefore, for a given `x`, `sqrt(x)` is a fixed point of a function `y => x / y`.
    + Another way to look at it: Dividing a number by its square root results in its square root again. Example: Let `x` be `9`. Applying `sqrt(9)`, i.e., `3`, to `y => 9 / y` results in `sqrt(9)` again.



+ Can we calculate `sqrt(x)` by iterating towards a fixed point of `y => x / y`?

  ````scala
  def sqrt(x: Double) =
    fixedPoint(y => x / y)(1.0)
  ````

  Unfortunately, no. When adding `println(next)` after the line `val next = // ... ` in the body of the `iterate` function, `sqrt(2)` prints :

  ````
  2.0
  1.0
  2.0
  ...
  ````

  + This can easily be reproduced using pen and paper.

+ These oscillation can be avoided by averaging successive values:

  ````scala
  def sqrt(x: Double) =
    fixedPoint(y => (y + x / y) / 2)(1.0)
  ````

  `sqrt(2)` now prints:

  `````
  1.5
  1.4166666666666665
  1.4142156862745097
  `````

  + The function `y => (y + x / y) / 2` still has the property that for a given `x`, `sqrt(x)` is a fixed point of it. Example: Let `x` be `9`. Applying `sqrt(9)` , i.e., `3`, to `y => (y + 9 / y) / 2` results in `sqrt(9)` again.

+ This technique of stabilizing by averaging can be factored out into its own function:

  ````scala
  def averageDamp(f: Double => Double)(x: Double) = 
    (x + f(x)) / 2
  ````

  `sqrt` can then be defined as follows:

  ````scala
  def sqrt(x: Double) =
    fixedPoint(averageDamp(y => x / y))(1.0)
  ````

### Lecture 2.4 -- Scala Syntax Summary

+ In the following, the context-free syntax of the language elements seen so far are given in Extended Backus-Naur form (EBNF), where

  + `|` denotes an alternative,
  + `[...]` and option (0 or 1),
  + `{...}` a repetition (0 or more).

+ Types:

  ````
  Type         = SimpleType | FunctionType
  FunctionType = SimpleType '=>' Type
               | '(' [Types] ')' '=>' Type
  SimpleType   = Ident
  Types        = Type {',' Type}
  ````

  A type can be:

  + A numeric type, e.g., `Int`, `Double`
  + The `Boolean` type
  + The `String` type
  + A function type, e.g., `Int => Int`, `(Int, Int) => Int`

+ Expressions:

  ````
  Expr         = InfixExpr | FunctionExpr
               | if '(' Expr ')' Expr else
  InfixExpr    = PrefixExpr | InfixExpr Operator InfixExpr
  Operator     = ident
  PrefixExpr   = ['+', '-', '!', '~'] SimpleExpr
  SimpleExpr   = ident | literal | SimpleExpr '.' ident | Block
  FunctionExpr = Bindings '=>' Expr
  Bindings     = ident [':' SimpleType]
               | '(' [Binding {',' Binding}] ')'
  Binding      = ident [':' Type]
  Block        = '{' { Def ';' } Expr '}'
  ````

  An expression can be:

  + An identifier, e.g., `x`,  `isGoodEnough`
  + A literal, e.g., `0`, `"abc"`
  + A function application, e.g., `sqrt(x)`
  + An operator application, e.g., `-x`, `y + x`
  + A selection, e.g., `math.abs`
  + A conditional expression, e.g., `if (x < 0) -x else x`
  + A block, e.g., `{ val x = math.abs(y); x * 2 }`
  + An anonymous function, e.g., `x => x + 1`

+ Definitions:

  ````
  Def        = FunDef | ValDef
  FunDef     = def ident { '(' [ Parameters ] ')' } [':' Type] '=' Expr
  ValDef     = val ident [ ':' Type ] '=' Expr
  Parameter  = ident ':' [ '=>' ] Type
  Parameters = Parameter { ',' Parameter }
  ````

  A definition can be:

  + A function definition, e.g., `def square(x: Int) = x * x`
  + A value definition, e.g., `val y = square(2)`

  A parameter can be:

  + A call-by-value parameter, e.g., `(x: Int)`
  + A call-by-name parameter, e.g., `(y: => Double)`

### Lecture 2.5 -- Functions and Data

+ In this section, we design a package for doing rational arithmetic. A rational number `x / y` is represented by its numerator `x` and its denominator `y` (both integers).

+ We define the following class `Rational` , introducing two entities, i.e., a new type `Rational` and a constructor `Rational`. Even tough they have the same name, there is no conflict, as Scala keeps the names of types and value in different namespaces.

+ Instances of a class are called objects. They are created by prefixing the application of the constructor with `new `. Their two members `numer` and `denom` are selected using an infix operator.

  ````scala
  val x = new Rational(1, 2)
  x.numer
  ````

+ In a first approach, arithmetic functions implementing standard rules are implemented as

  top-level functions that take and return `Rational`s, e.g.:

  ````scala
  def addRational(r: Rational, s: Rational): Rational = // ...
  ````

+ Then, the functions operating on the data abstraction are packaged in the abstraction itself -- as methods:

  ````scala
  class Rational(x: Int, y: Int) {
    def numer = x
    def denom = y
    
    def add(that: Rational) =
      new Rational(
        numer * that.denom +  that.numer * denom,
        denom * that.denom)
    
    def neg =
      new Rational(-numer, denom)
    
    def sub(that: Rational) =
      add(that.neg)
    
    override def toString =
      numer + "/" + denom
  }

  val x = new Rational(1, 3)
  val y = new Rational(5, 7)
  val z = new Rational(3, 2)
  x.numer // 1
  x.denom // 3
  x.add(y) // 22/1
  println(x.sub(y).sub(z)) // -79/42
  ````

### Lecture 2.6 -- More Fun with Rationals

+ In the previous lecture, rationals have not always been represented in their simplest form. They can be reduced to their smallest numerator and denominator by dividing both with their greatest common divisor.

+ In order to adhere to the DIY ("Don't repeat yourself!") principle, we do not implement this in each operation but in the constructor. Three ways of doing so are shown. However, clients of the `Rational` class always experience the same behavior.

  ````scala
  // Approach 1
  // `gcd` is calculated immediately, its value can be re-used by `numer` and `denom`.
  class Rational(x: Int, y: Int) {
    // `private` members can only be accessed from inside the class
    private def gcd(a: Int, b: Int): Int =
      if (b == 0) a else gcd(b, a % b)
    private val g = gcd(x, y)
    def numer = x / g
    def denom = y / g
    // ...
  }

  // Approach 2
  // This can be advantageous if `numer` and `denom` are called infrequently.
  class Rational(x: Int, y: Int) {
    private def gcd(a: Int, b: Int): Int =
      if (b == 0) a else gcd(b, a % b)
    def numer = x / gcd(x, y)
    def denom = y / gcd(x, y)
    // ...
  }

  // Appraoch 3
  // This can be advantageous if `numer` and `denom` are called often.
  class Rational(x: Int, y: Int) {
    private def gcd(a: Int, b: Int): Int =
      if (b == 0) a else gcd(b, a % b)
    // As `val`s, `numer` and `denom` are computed only once.
    val numer = x / gcd(x, y)
    val denom = y / gcd(x, y)
    // ...
  }
  ````

+ The ability to choose from different implementations without affecting clients is called data abstraction.

+ Inside a class, there is the self reference `this`, which represents the object on which a method is executed. A simple name `x`, referring to a class member, is an abbreviation of `x`.  `this` is used to implement the member functions `less` and `max` on the function `Rational`.

  ````scala
  class Rational(x: Int, y: Int) {
    // ...
    def less(that: Rational) =
      numer * that.denom < that.numer * denom
    
    def max(that: Rational) =
      if (this.less(that)) that else this
  }
  ````

+ The `Rational` class requires `denom` to be positive. This can be enforced like so:

  ````scala
  class Rational(x: Int, y: Int) {
    require(y > 0, ”denominator must be positive”)
    // ...
  }
  ````

+ The predefined function `require` takes a condition and an optional message string. If the condition is `false`, an `IllegalArgumentException` is thrown. There is also `assert`, which takes the same arguments as `require`, but throws an `AssertionError` whenever the condition is `false`. The two differ in intent:

  + `require` is used to enforce a requirement on the function caller.
  + `assert` is used to check the code of the function itself.

+ Scala introduces the primary constructor of the class implicitly. It takes the parameters of the class (`class Rational(x: Int, y: Int)`) and executes the statements in the class body, e.g., `require(y > 0, ”denominator must be positive”)`.

+ Auxiliary constructors may be declared. They are called `this`.

  ````scala
  class Rational(x: Int, y: Int) {
    def this(x: Int) = this(x, 1)
    // ...
  }
  ````

+ It would be a bad idea if we altered `Rational` so that numbers are kept unsimplified until they are printed, i.e., if we moved the simplification inside the `toString` method. Why? We are dealing with integers here, and we might exceed the maximal number.

### Lecture 2.7 -- Evaluation and Operators

+  The previously introduced model of evaluation based on the term of rewriting, the so-called substitution model can be extended to model classes and objects. Consider the following class definition:

  ````scala
  // The list of function parameters is optional.
  // Also, parameter types are left out for brevity.
  class C(x1, ..., xm) { ... def f(y1, ..., yn) = b ... }
  ````

  How is the following expression evaluated?

  ````scala
  new C(v1, ..., vm).f(w1, ..., wn)
  ````

  Inside the function's body,

  + `f`'s parameters `y1, ..., yn` are substituted by the arguments `w1, ..., wn`,
  + `f`'s parameters `v1, ..., vn ` are substituted by the arguments `w1, ..., wn`, and
  + the self reference `this` is substituted by the value of the object `new C(v1, ..., vn)`:

  ````
     new C(v1, ..., vm).f(w1, ..., wn)
  -> [w1/y1, ..., wn/yn][v1/x1, ..., vm/xm][new C(v1, ..., vm)/this]
     b
  =  b
  ````

+ Two more object rewriting examples are given:

  `````
     new Rational(1, 2).numer
  -> [1/x, 2/y][][new Rational(1, 2)/this]
     x
  =  1

     new Rational(1, 2).less(new Rational(2, 3))
  -> [1/x, 2/y][new Rational(2, 3)/that][new Rational(1, 2)/this]
     this.numer * that.denom < that.numer * this.denom
  =  new Rational(1, 2).numer * new Rational(2, 3).denom <
     new Rational(2, 3).numer * new Rational(1, 2).denom
  // ...
  =  1 * 3 < 2 * 2
  =  true
  `````

+ With integers, we can write `x + y`, however, with `Rational`s we must do `r.add(s)`. In Scala, we can eliminate using

  1. Infix notation
  2. Relaxed identifiers

+ Using infix notation, we can write `r add s` instead of `r.add(s)`. Any method with one parameter can be used like an infix operator.

+ In Scala, an identifier are not confined to being alphanumeric. Instead, they can be

  + Alphanumeric: Starting with a letter, followed by a sequence of letters or numbers. (`_` counts as a letter in this definition.)
  + Symbolic: Starting with an operator symbol, e.g., `+` or `-`, followed by other operator symbols.
  + Alphanumeric identifiers can also end in an underscore, followed by some operator symbols.

+ Using infix notation and relaxed identifiers, we can define the class `Rational` more naturally:

  `````scala
  class Rational(x: Int, y: Int) {
    // ...
    
    def <(that: Rational) =
      numer * that.denom < that.numer * denom
    
    def max(that: Rational) =
      if (this < that) that else this
    
    def +(r: Rational) =
      new Rational(
        numer * r.denom + r.numer * denom,
        denom * r.denom)
    
    def unary_- : Rational = 
      new Rational(-numer, denom)
    
    def -(that: Rational) = 
      this + -that
  }

  val x = new Rational(1, 2)
  val y = new Rational(1, 3)
  x * x + y * y // (x * x) + (y * y)
  `````

+ The precedence of an operator is determined by its first character. Here is a list of the characters in increasing order of precedence:

  ````
  (all letters)
  |
  ^
  &
  < >
  = !
  :
  + -
  * / %
  (all other special characters)
  ````

+ A parenthesized version of `a + b ^? c ?^ d less a ==> b | c` would be `((a + b) ^? (c ?^ d)) less ((a ==> b) | c)`.

## Week 3 -- Data and Abstraction

### Lecture 3.1 -- Class Hierarchies

+ Abstract classes contain members that are yet to be implemented. Accordingly, they cannot be instantiated. `IntSet` is an example of an abstract class:

  ````scala
  abstract class IntSet {
    def contains(x: Int): Boolean
    def incl(x: Int): IntSet
  }
  ````

+ Here is an implementation of `IntSet` in terms of binary trees:

  ````scala
  class Empty extends IntSet {
    def contains(x: Int): Boolean = false
    def incl(x: Int): IntSet = new NonEmpty(x, new Empty, new Empty)
  }

  class NonEmpty(elem: Int, left: IntSet, right: IntSet) extends IntSet {
    def contains(x: Int): Boolean =
      if (x < elem) left contains x
      else if (x > elem) right contains x
      else true

    def incl(x: Int): IntSet =
      if (x < elem) new NonEmpty(elem, left incl x, right)
      else if (x > elem) new NonEmpty(elem, left, right incl x)
      else this
  }
  ````

   + This is still a purely functional approach, there is no mutation going on. We never "change" the data structure, we always return a new one.
   + Data structures like this are called persistent data structures, since when we do "changes", e.g., by calling `incl`, the old version of the data structure is still maintained, i.e., it is not discarded.
      + Example: Let `x` be a `NonEmpty` node. If we call `incl` on it with an element that is smaller than the element `x` is carrying, the new `NonEmpty` node resulting from this call will reference  `x`'s right-hand side, i.e., `x`'s `right`. Parts of `x`'s left-hand side may also stay referenced. This can easily be reproduced using pen and paper.

+ Terminology

  + `Empty` and `NonEmpty` extend `IntSet`. As such, they conform to the `IntSet` type and can be used wherever an `IntSet` is required.
  + `IntSet` is the superclass of `Empty` and `NonEmpty`.
  + `Empty` and `NonEmpty` are subclasses of `IntSet`.
  + In Scala, any user-defined class extends another class, if no superclass is provided, the class in question simply extends `java.lang.Object`.
  + The base classes of a class are all direct or indirect superclasses of that class. E.g., `Empty`'s base classes are `IntSet` and `java.lang.Object`.
  + `incl` and `contains` in `Empty` and `NonEmpty` implement the respective abstract functions in `InSet`.
  + A class may redefine a non-abstract function of its superclass by prefixing its definition with the keyword `override`, e.g., `override def foo = 2`. Using `override` here is not optional, it is, however, optional and mostly omitted when implementing an abstract function. 

+ As it can be argued that there really is one single `Empty` node, it can be replaced by a singleton object. Singleton objects cannot be created. Behind the scenes, one instance is being created once it is first referenced. Singletons are values and evaluate to themselves.

  ````scala
  object Empty extends IntSet {
    def contains(x: Int): Boolean = false
    def incl(x: Int): IntSet =
      new NonEmpty(x, Empty, Empty) // It does not say `new Empty` anymore but `Empty`.
  }
  ````

+ Standalone Scala programs (as opposed to worksheets or the REPL) need to have a `main` method inside an object.

  ````scala
  object Hello {
    def main(args: Array[String]) =
      println("Hello, world.")
  }
  ````

+ What would a method `def union(other: IntSet): IntSet` look like?

  + For `Empty`, it would be `other`, as the union of an empty set with another set always results in the other set.
  + For `NonEmpty`, it would be `((left union right) union other) incl elem`? How can we know this actually terminates? Well, the first recursive call, `left union right`, is on `left`. Thus, the first recursive call is on something smaller than what we started with, `this`, and has to eventually reach the base case defined in `Empty`. The same goes for the second recursive call, `(left union right) union other`, which is on the union of `left` and `right`, which, in turn, is still smaller than `this`, as it doesn't contain `elem`. Therefore, the base case will eventually be reached, too.

+ Scala implements dynamic binding/dynamic method dispatch. Thus, whenever an overridden method is called through a superclass reference, e.g., `val s: IntSet = Empty; s.incl(42)`, which version of the method to execute is made based on the runtime type of the object that contains it. Thus, at run time, `Empty`'s implementation is chosen -- not `IntSet`'s (if it had one).

+ Here two examples of how dynamic binding evaluates:

  ````
     Emtpy contains 1
  -> [1/x][Empty/this]
     false
  =  false

     (new NonEmpty(7, Empty, Empty)) contains 7
  -> [7/elem][7/x][new NonEmpty(7, Empty, Empty)/this]
     if (x < elem) this.left contains x
     else if (x > elem) this.right contains x
     else true
  =  if (7 < 7) new NonEmpty(7, Empty, Empty).left contains 7
     else if (7 > 7) new NonEmpty(7, Empty, Empty).right contains 7
     else true
  =  true
  ````

+ Can we implement higher-order functions in terms of objects? Obviously. Here is my shot at it:

  ````scala
  trait Function[-Param, +Return] {
    def apply(param: Param): Return
  }

  // `f1` demonstrates why covariance and contravariance are necessary.
  val f1: Function[String, Any] = new Function[Any, String] {
    override def apply(param: Any) =
      param.toString
  }

  assert(f1.apply("Hello, world.") == "Hello, world.")

  // `f2` demonstrates how `Function` objects can represent higher-order functions.
  val f2: Function[Int, Function[Int, Int]] = new Function[Int, Function[Int, Int]] {
    override def apply(outer: Int) =
      new Function[Int, Int] {
        override def apply(inner: Int) =
          inner + outer
      }
  }

  assert(f2.apply(2).apply(40) == 42)
  ````

+ Can We implement objects is terms of higher-order functions. I'll delegate to the following blog post, which argues that objects and closures are equivalent: http://c2.com/cgi/wiki?ClosuresAndObjectsAreEquivalent

### Lecture 3.2 -- How Classes are Organized

+  Classes and objects are organized in packages. To do so, a package clause can be placed at the top of a source file.

  ````scala
  package progfun.examples
  object Hello {}
  ````

  `Hello`'s fully qualified name is `progfun.examples.Hello`.

+ A class `Rational` in package `week3` can be used via its fully qualified name, e.g., `new week3.Rational(1, 2)`. Alternatively, imports can be used.

  ````scala
  // Named imports
  import week3.Rational          // Imports just `Rational`
  import week3.{Rational, Hello} // Imports both `Rational` and `Hello`
  // Wildcard import
  import week3._                 // Imports everything in package `week3`
  ````

  One can import from packages and (singleton) objects.


+ Some entities are imported automatically into any Scala program.

  + All members of package `scala`, e.g., `Int`.
  + All members of package  `java.lang`, e.g., `Object`.
  + All members of the singleton object `scala.Predef`, e.g. `assert`.

+ Scala's standard library can be explored using the scaladoc pages: http://www.scala-lang.org/files/archive/api/current/

+ Just like in Java, classes may only inherit from one superclass. To inherit from arbitrarily many superclasses ("single inheritance language"), one can use traits.

  ````scala
  trait Planar {
    def height: Int
    def width: Int
    def surface = height * width
  }

  class Square extends Shape with Planar with Movable // ...
  ````


  Traits resemble Java's interfaces. They are, however, more powerful, as they can also contain fields and concrete methods. Unlike classes, traits cannot have (value) parameters.

+ The remainder of this lecture is devoted to Scala' class hierarchy.![Scala Class Hierarchy](scala_class_hierarchy.png)
+ Top types

  + `Any` is the base type of all types, and conceptually defines certain universal methods, e.g., `==`, `!=`, `equals`, `hashCode`, `toString`. (`==` is essentially just a forwarder that calls the Java `equals` method.)

  + `AnyVal` is the base type of all primitive types.

  + `AnyRef` is the base type of all reference types. It is an alias of `java.lang.Object`.
+ The `Nothing` type is at the bottom of the type hierarchy. It subclasses every other type. There is no value of type `Nothing`. What is it useful for?
  + It can be used to signal abnormal termination. (An example follows when exceptions are covered.)
  + It can be used as an element type of empty collections, e.g., `Set[Nothing]`.
+ The `Null` type subclasses every class that inherits from `Object`. The type of `null` is `Null`. Every reference class type has a value `null`.
+ Exceptions is Scala are similar to exceptions in Java. The expression `throw Exc` aborts the evaluation with the exception `Exc`. The type of this expression is `Nothing`.
+ The type of  `if (true) 1 else false` is `AnyVal`, as `1` is an `Int`, `false` is a `Boolean` and `AnyVal` happens to be the closest `AnyVal` of the two.

### Lecture 3.3 -- Polymorphism

+ The immutable linked list is a fundamental data structure in many functional programming languages. Such a list is either a `Nil` or a `Cons`. `Nil` represents an empty list, whereas `Cons` holds an element `head` as well as a pointer to the rest of the list `tail`.

+ At first, an implementation of an `IntList`, i.e., a definition of lists with integers, is given. By this example, the concept of value parameters is explained, which is about defining parameters and fields of a class at the same time.

  ````scala
  // Given...
  trait IntList
  // ...then the following...
  class Cons(val head: Int, val tail: IntList) extends IntList
  // ...is equalvalent to...
  class Cons(_head: Int, _tail: IntList) extends IntList {
    val head = _head
    val tail = _tail
  }
  // ...where `_head` and `_tail` are otherwise unused names. 
  ````

+ `IntList` does not scale, as we'd need `DoubleList`, and so on. Therefore, a generalized definition that makes use of type parameters is given:

  ````scala
  trait List[T] {
    def isEmpty: Boolean
    def head: T
    def tail: List[T]
  }

  class Cons[T](val head: T, val tail: List[T]) extends List[T] {
    def isEmpty = false
  }

  class Nil[T] extends List[T] {
    def isEmpty: Boolean = true
    def head: Nothing = throw new NoSuchElementException("Nil.head")
    def tail: List[T] = throw new NoSuchElementException("Nil.tail")
  }

  ````

  Type parameters are written in square brackets, e.g., `[T]`.

+ In order to demonstrate that functions can have type parameters, too, here is a function that creates `List`s containing one element of type `T`:

  ````scala
  def singleton[T](elem: T) = new Cons[T](elem, new Nil[T])

  singleton[Int](1) // new Cons[Int](1, new Nil[Int])
  singleton[Boolean](true) // new Cons[Boolean](true, new Nil[Boolean])
  ````

+ The Scala compiler can oftentimes deduce the correct type parameters from the provided value arguments. Thus, they can be omitted in a lot of cases. E.g., `singleton(1)` can be written instead of `singleton[Int](1)`.

+ Type parameters (and arguments) do not affect evaluation in Scala. In terms of our substitution model, we can assume that all types are removed before evaluating a program. Types are only important for the compiler. Languages that also do type erasure include Java, Scala, and Haskell, whereas C++, C# and F# keep them at run time.

+ Polymorphism is a Greek word with the original meaning of "having many forms". Applied to a function, it means that this function can be applied to arguments of many types. Applies to a type, it means that the type can have instances of many types. In this course, polymorhism has been seen in two forms:

  + Subtyping: Wherever there is a parameter that accepts a base class instance, a subclass instance can be passed.
  + Generics: Many instances of a function or class can be created using type parameterization, e.g., `List[Int]` or `List[String]`.

+ A function `nth` selecting the `n`th integer of a given list might look like this:

  ````scala
  def nth[T](n: Int, xs: List[T]): T =
    if (xs.isEmpty) throw new IndexOutOfBoundsException
    else if (n == 0) xs.head
    else nth(n - 1, xs.tail)
  ````

  The first list element has the number 0. If `n` is outside the range from 0 until length of the list minus one, an `IndexOutOfBoundsException` is thrown.

## Week 4 -- Types and Pattern Matching

### Lecture 4.1 -- Objects Everywhere

+ In a pure object-oriented programming language, each value is an object. If this language is based on classes, the type of each value, i.e., object, would be a class.

  + In Scala, there are not just reference types but also primitive types and functions. Does this mean Scala is not purely object-oriented?
    + Conceptually, primitive types, e.g., `Int`, do not receive any special treatment and are used like other classes. The fact that, under the hood, `Int`s are represented as 32-bit integers, can be regarded as a mere optimization. The rest of this section elaborates on this point.
    + Functions are actually represented by classes, too, just like I have demonstrated at the end of lecture 3.2. Lecture 4.2 elaborates on classes representing functions. 

+ Scala's `Boolean` type maps to the JVM's primitive type `boolean`, however, one could easily implement a purely object-oriented `Boolean`  type from scratch.

  ````scala
  abstract class Boolean {
    def ifThenElse[T](thenExpr: => T, elseExpr: T): T

    def && (x: => Boolean): Boolean = ifThenElse(x, False)
    def || (x: => Boolean): Boolean = ifThenElse(True, x)
    def unary_! : Boolean           = ifThenElse(False, True)

    def == (x: Boolean): Boolean    = ifThenElse(x, !x)
    def != (x: Boolean): Boolean    = ifThenElse(!x, x)

    // In the following, we assume `False` < `True`:
    def < (x: Boolean): Boolean     = ifThenElse(False, x)
  }

  object True extends Boolean {
    def ifThenElse[T](thenExpr: => T, elseExpr: T): T =
      thenExpr
  }

  object False extends Boolean {
    def ifThenElse[T](thenExpr: => T, elseExpr: T): T =
      elseExpr
  }

  val x: Boolean = True || False
  val y: Boolean = !x == x
  val z: AnyVal = y.ifThenElse(84 / 2, false)
  ````

+ Analogously, a partial specification of such a class `Int` is given (but omitted here for brevity). While the operations on `Int`s are expressed as methods, the question is asked whether we can actually implement this class solely based on objects, i.e., not resorting back to primitive `int`s.

+ Regarding the previous question, a class `Nat` representing non-negative integers is implemented:

  ````scala
  abstract class Nat {
    def isZero: Boolean
    def predecessor: Nat
    def successor: Nat = new Succ(this)
    def +(that: Nat): Nat
    def -(that: Nat): Nat
  }

  object Zero extends Nat {
    def isZero = true
    def predecessor = throw new Error("Zero.predecessor")
    def +(that: Nat) = that
    def -(that: Nat) = if (that.isZero) this else throw new Error("Negative number")
  }

  class Succ(n: Nat) extends Nat {
    def isZero = false
    def predecessor = n
    def +(that: Nat) = new Succ(n + that)
    def -(that: Nat) = if (that.isZero) this else n - that.predecessor
  }
  ````

### Lecture 4.2 -- Functions as Objects

+ In Scala, function values are objects. `A => B` is just a shorthand for `scala.Function1[A, B]`, which is defined similar to what I defined at the end of lecture 3.2. To represent functions with two parameters, there is `Function2`, to represent those with three parameters, there is `Function2`, and so on.

+ This is what anonymous function definitions are expanded to:

  ````scala
  (x: Int) = x * x
  // ...is expanded to...
  {
    class AnonFun extends Function1[Int, Int] {
      def apply(x: Int) = x * x
    }
    new AnonFun
  }
  // ...or, for short, using anonymous class syntax
  new Function1[Int, Int] {
    def apply(x: Int) = x * x
  }
  ````

+ A function call such as `f(a, b)`, is expanded to `f.apply(a, b)`. Remember, `f` is the value of a class type!

+ Methods are no objects, if they were, then `apply` would be an instance of some `Function` class, and a call to it would be a call to its `apply` method, so we'd end up in an infinite loop. However, if some function `f` is used in a place where a `Function` type is expected, one is automatically provided. E.g., for a method `def f(x: Int): Boolean  `, `(x: Int) => f(x)` would be provided. In lambda calculus, this conversion is known as "eta expansion"

+ Given our `List` type, we could define an object of the same name, which provides the means to obtain a list of two elements, e.g., `1` and `2`, by writing `List(1, 2)`.

  ````scala
  object List {
    // List(1, 2) == List.apply(1, 2)
    def apply[T](x1: T, x2: T): List[T] =
      new Cons(x1, new Cons(x2, new Nil))
  }
  ````

### Lecture 4.3 -- Subtyping and Generics

+ We have already covered two forms of polymorphism, i.e., subtyping, which originated in object-oriented programming, and generics, which originated in functional programming.
+ As a reminder, subtyping allows us to pass a type where a base type was required, while generics enable types to be parameterized with other types.
+ In this lecture, we will look at how subtyping and generics interact. In this context, we will look at two main areas:
  + Bounds (Subjecting type parameters to subtype constraints)
  + Variance (How do parameterized types behave under subtyping?)

+ Type bounds

  + Consider a method `assertAllPos`, which takes  an `IntSet` and returns it if all elements are positive, otherwise it will throw an error.

  + What would its signature look like if we wanted it to reflect the following two equations:

    ````
    assertAllPos(Empty)    = Empy
    assertAllPos(NonEmpty) = Either NonEmpty or Exception
    ````

    In other words, we want the signature to reflect that if we provide an instance of `IntSet`'s subtype `Empty`, we will get an `Empty` back, and if we provide an instance of `IntSet`'s subtype `NonEmpty`, we get a `NonEmpty` back (if no exception has been thrown). In no case do we provide, say, a `Empty` and receive a `NonEmpty`. One way to express this is:

    ```` scala
    def assertAllPos[S <: IntSet](r: S): S
    ````

  + `<: IntSet` is an upper bound of the type parameter `S`. This notation is also used outside of type bounds. Generally, it means this:

    + `S <: T` means `S` is a subtype of `T`
    + `S >: T` means `S` is a supertype of `T`, or `T` is a subtype of `S`

  + Lower bounds can also be used: `[S >: NonEmpty]` introduces a type parameter `S` that can range only over supertypes of `NonEmpty`. Consequently, `S` could be one of `NonEmpty`, `IntSet`, `AnyRef`, or `Any`.

  + `S` can also be bound from below (`NonEmpty`) as well as from above (`IntSet`): `[S >: NonEmpty <: IntSet]` restricts `S` to the interval between `NonEmpty` and `IntSet`, which only contains `NonEmpty` and `IntSet`.

+ Covariance

  + Should `NonEmpty <: IntSet` imply `List[NonEmpty] <: List[IntSet]`?

  + Types for which this relationship holds are called covariant, because their subtyping relationship varies with the type parameter. For example, if `List` were covariant, `List[S]` would only be a subtype of `List[T]` if `S` was a subtype of `T`. Thus, the relationship of the two `List` types would depended on the relationship of the two type parameters.

  + Covariance does not always make sense. Consider this Java code snippet:

    ````java
    NonEmpty[] a = new NonEmpty[]{new NonEmpty(1, Empty, Empty)}
    // In Scala, this line wouldn't compile, as Scala arrays aren't covariant.
    IntSet[] b = a
    b[0] = Empty
    NonEmpty s = a[0]
    ````

    In this (compiling!) example, we assign an `Empty` to variable of type `NonEmpty`! At run time, assigning `Empty` to `b[0]` will cause an `ArrayStoreException` to be thrown. (Java stores a type tag for every array, so it knows that this assignment is malicious.) Thus, making arrays covariant produced a hole in the type system (assignment compiles) that had to be patched with run time checks (comparing against type tag).

  + The following principle ("The Liskov Substitution Principle") tells us when a type can be subtype of another:

    > If `A <: B`, then everything one can do with a value of type `B`, one should also be able to do with value of type `A`.

    How would this translate to the problem described above? Well, `NonEmpty[]` should not be a subtype of `IntSet[]`, because one can put `Empty` instances into `IntSet[]` but not into `N`

    `onEmpty[]`

### Lecture 4.4 -- Variance

+ In this lecture, we are going to cover the concept of variance, i.e., how subtyping relates to genericity.

+ In the previous lecture, it has been demonstrated how some types should be covariant whereas others shouldn't.

  + Generally, a type that permits its elements to be mutated should not be covariant, e.g. arrays.
  + Analogously, immutable types can be covariant, if certain restrictions on methods are met, e.g. lists.

+ Given a type `C[T]` as well as two types `A` and `B` with `A <: B`, there are three possible relationships between `C[A]` and `C[B]`:

  + `C` is covariant: `C[A] <: C[B]`
  + `C` is contravariant: `C[A] <: C[B]`
  + `C` is nonvariant: Neither `C[A]` nor `C[B]` are subtype of the other.

+ By annotating the type parameter, a type's variance can be declared:

  + `C` is covariant: `class C[+A]`
  + `C` is contravariant: `class C[-A]`
  + `C` is nonvariant: `class C[A]`

+ Consider the following two types `A` and `B`. According to the Liskov Subsitution Principle, should `A` be a subtype of `B` (`A <: B`), `B` be a subtype of `A` (`B <: A`), or should they be unrelated?

  ````scala
  type A = IntSet => NonEmpty
  type B = NonEmpty => IntSet
  ````

  Since `NonEmpty <: IntSet` is true, `A <: B` should hold.

  + Whenever we expect a function that takes a `NonEmpty` as argument, a function that takes an `IntSet` will satisfy this requirement.
  + Whenever we expect a function that returns an `IntSet`, a function that returns a `NonEmpty` does just that.
  + To put it in terms of the aforementioned principle, you can do with `A` everything you can do with `B`.

+ Generally, there is the following subtyping rule between function types: If `A2 <: A1` and `B1 <: B2`, then ` A1 => B1 <: A2 => B2 `. Thus, you can always do the following:

  ````scala
  val f: Function1[A2, B2] = new Function[A1, B1] { /* ... */ }
  ````

+ Since we now know that functions are contravariant in their argument type(s) and covariant in their return type(s), we can revise our `Function1` trait from before:

  ````scala
  trait Function1[-T, +U] {
    def apply(x: T): U
  }
  ````

+ We have, however, seen an example, i.e., `Array[+T]`, in which covariance in combination a certain method, i.e., `update(T)`, is unsound. You may want to refer back to the Java code snippet of lecture 4.3, in which updating the element of an array -- which are covariant in Java -- causes a run time error to be thrown. The problem of the code below is that the covariant type parameter `T` is the type of a method parameter.

  ````scala
  class Array[+T] {
    def update(x: T) // ...
  }
  ````

+ Thus, the Scala compiler enforces certain rules regarding variance annotations. Roughly, these variance checks are:

  + Covariant type parameters may only appear in method results.
  + Contravariant type parameters can only appear in method parameters.
  + Invariant type parameters can appear everywhere.

+ In the following, we are going to improve our `List` implementation. The first goal is to make `Nil` a singleton object, as there really is only one empty list. We can easily achieve this by changing its signature to `object Nil extends List[Nothing]`. However, a `Cons[T]` expects a `List[T]` as its second argument (`tail`). Thus, we cannot pass `Nil`, i.e., a `List[Nothing]`, as the second argument of, say, a `List[String]`. Sure, `Nothing <: String`, but `List[Nothing]` and `List[String]` have no subtyping relationship. Obviously, we have to make `List` covariant. Here's the revised code:

  ````scala
  trait List[+T] {
    def isEmpty: Boolean
    def head: T
    def tail: List[T]
  }

  class Cons[T](val head: T, val tail: List[T]) extends List[T] {
    def isEmpty: Boolean = false
  }

  object Nil extends List[Nothing] {
    def isEmpty: Boolean = true
    def head: Nothing = throw new NoSuchElementException("Nil.head")
    def tail: Nothing = throw new NoSuchElementException("Nil.tail")
  }

  // Now, we can do this:
  val l1: List[String] = Nil
  val l2: List[String] = new Cons("Hello, world.", Nil)
  ````

+ Lastly, we want add a `prepend` method to our `List` class. However, the following does not compile, as it violates the one of the variance rules enforced by the compiler, i.e., covariant type parameters may only appear in method results.

  ````scala
  // Caution, this does not compile!
  trait List[+T] {
    def prepend(elem: T): List[T] = new Cons(elem, this)
  }
  ````

  The compiler is actually right to reject the previous definition, as it violates the Liskov Substitution Principle: Not everything you can do with a `List[IntSet]` you can do with a `List[NonEmpty]`, e.g.,  `prepend` an `Empty`. Thus, `List[NonEmpty]` cannot be a subtype of `List[IntSet]`.

+ How can we make `prepend`, which is a natural list on immutable lists, variance-correct? By the use of a lower bound:

  `````scala
  def prepend[U >: T](elem: U): List[U] = new Cons(elem, this)
  `````

  This compiles, because:

  + Covariant type parameters may appear in lower bounds of method type parameters.
  + Contravariant type parameters may appear in upper bounds of method type parameters.

+ What is the result type of `def f(xs: List[NonEmpty], x: Empty) = xs prepend x`? This function definition compiles, so we know the argument for the `prepend` method's type parameter `U` must satisfy two conditions:

  + `U >: T` must hold, and since `T` is `NonEmpty`, `U ` has to be either `NonEmpty`, `IntSet`, `AnyRef`, or `Any`.
  + `elem`, which is of type `Empty`, must conform to `U`.

  The type inferencer chooses `U` to be `IntSet`.

   + `IntSet` is a superclass of `NonEmpty`.
   + By being of type `Empty`, `elem` is also a `IntSet`.

  Thus, the return type of the function is `List[IntSet]`.

### Lecture 4.5 -- Decomposition

+ Decomposition is an important problem in programming. Suppose we have a hierarchy of classes and want to build tree-like data structures from the instances of these classes. How would we find out what kinds of elements are in this tree?

+ To find out more about this, let's start with a simple example: A small interpreter for arithmetic expressions. Expressions are represented as a class hierarchy, with a base trait `Expr` and two subclasses, `Number` and `Sum`.

  ````scala
  trait Expr {
    def isNumber: Boolean
    def isSum: Boolean
    def numValue: Int
    def leftOp: Expr
    def rightOp: Expr
  }

  class Number(n: Int) extends Expr {
    def isNumber: Boolean = true
    def isSum: Boolean = false
    def numValue: Int = n
    def leftOp: Expr = throw new Error("Number.leftOp")
    def rightOp: Expr = throw new Error("Number.rightOp")
  }

  class Sum(e1: Expr, e2: Expr) extends Expr {
    def isNumber: Boolean = false
    def isSum: Boolean = true
    def numValue: Int = throw new Error("Sum.numValue")
    def leftOp: Expr = e1
    def rightOp: Expr = e2
  }
  ````

  Now, we can easily write an evaluation function.

  ````scala
  def eval(e: Expr): Int = {
    if (e.isNumber) e.numValue
    else if (e.isSum) eval(e.leftOp) + eval(e.rightOp)
    else throw new Error("Unknown expression " + e)
  }
  ````

  However, it is tedious to write out all these classification methods, e.g., `isNumber`, as well as accessor functions, e.g., `numValue`. Furthermore,  if we want to add new expressions, e.g., the ones below, we need to add classification and accessor methods to all classes defined above.

  ````scala
  class Prod(e1: Expr, e2: Expr) extends Expr
  class Var(x: String) extends Expr
  ````

  Actually, in order to integrate `Prod` and `Var` into the hierarchy, 25 new methods would need to be defined. In fact, if we continued to add new expressions, there would be a quadratic increase in methods we would need to define.

+ A "non-solution" to this problem would be to use type tests and type casts. These can be done using the following methods defined in class `Any`:

  ````scala
  def isInstanceOf[T]: Boolean // Checks whether this object's type conforms to `T`
  def asInstanceOf[T]: T       // Treats this object as an instance of type `T`,
                               // throws a `ClassCastException` if it isn't. 
  ````

  Here is a definition of the `eval` method using type tests and type casts:

  `````scala
  def eval(e: Expr): Int =
    if (e.isInstanceOf[Number])
      e.asInstanceOf[Number].numValue
    else if (e.isInstanceOf[Sum])
      eval(e.asInstanceOf[Sum].leftOp) +
      eval(e.asInstanceOf[Sum].rightOp)
    else throw new
      Error("Unknown expression " + e)
  `````

  On the plus side, with this approach, there is no need for classification methods at all, and the accessor methods are only for the classes where the value in question is defined. However, the usage of type tests and type casts is very low-level. Generally, relying on them is error-prone, since, at run time, we never know whether a type cast will succeed or not. (Above, we can statically assure that the type casts will not fail, since we guarded them with type checks, but still.)

+ Another -- better -- approach would be object-oriented decomposition. Suppose all we wanted to do is evaluate expressions evaluate expressions.

  ````scala
  trait Expr {
    def eval: Int
  }

  class Number(n: Int) extends Expr {
    def eval: Int = n
  }

  class Sum(e1: Expr, e2: Expr) extends Expr {
    def eval: Int = e1.eval + e2.eval
  }
  ````

  However, this approach is limited. Say we want to simplify expressions using this rule:

  ````
  a * b + a * c -> a * (b + c)
  ````

  This is a non-local simplification. It cannot be encapsulated into a method of one object. In order to inspect our tree-like data structure of `Expr` instances, we need to go back to square one, using classification and accessor methods.

### Lecture 4.6 -- Pattern Matching

+ Recap

  + We are still trying to find a general and convenient way to access objects in a extensible class hierarchy.

  + Here is what we already tried:

    + Classification and accessor methods: quadratic "explosion" of number of methods to be defined for each new subtype of `Expr`
    + Type tests and type casts: unsafe, low-level
    + Object-oriented decomposition: does not work for every problem to be solved, all classes need to be touched to add a new method.

  + Observation: The purpose of the classification and accessor methods can be viewed as reversing the construction process:

    + What class was used?
    + What were the arguments to the constructor when it was invoked?

    This situation is so common in functional programming languages that many offer a construct to automate it. In Scala and other languages, e.g., Haskell, this construct is called pattern matching. It is introduced in the rest of this lecture.

+ A case class definition is like a normal class definition, except that it is preceded by the modifier `case`.

  `````scala
  trait Expr
  case class Number(n: Int) extends Expr
  case class Sum(e1: Expr, e2: Expr) extends Expr
  `````

  Other than defining a trait and two concrete subclasses of it, so-called companion objects with `apply` methods are defined implicitly.

  ````scala
  object Number {
    def apply(n: Int) =
      new Number(n)
  }

  object Sum {
    def apply(e1: Expr, e2: Expr) =
      new Sum(e1, e2)
  }
  ````

  As we saw before, terms like `Number(2)` expand into `Number.apply(2)`, so we can just write `Number(2)`

   instead of `new Number(2)`.

+ In Scala, pattern matching is expressed using the `match ` keyword.

  ````scala
  def eval(e:Expr): Int = e match {
    case Number(n) => n
    case Sum(e1, e2) => eval(e1) + eval(e2)
  }
  ````

  + The syntax can be described as follows: The `match` keyword is followed by a sequence of `case`s, each of which associate a pattern (``pat``) with an expression (`expr`): `pat => expr`.

  + Patterns are constructed from:

    + Constructors, e.g, ``Number``, ``Sum``
    + Variables, e.g., `n`, `e1`, `e2`
    + Constants, e.g. `1`, `true`
      + Besides literal constants, there can also be named constants.
    + Wildcard pattern, i.e., `_`
      + This indicates that we do not care about the value, we cannot reference it inside the expression.

    There is some "fine print":

    + Variables begin with a lowercase letter.
    + The same variable name may appear only once in a pattern, thus, ``Sum(x, x)`` is not a legal pattern.
    + Names of constants begin with a capital letter, with the exception of reserved words `null`, ``true``, ``false``. What if we some named constant, say, `val hello = "Hello, world."` is in scope and we want to use it in our pattern. Then we must either surround it with backticks (`` `hello` ``) or make it uppercase (`Hello`). See the following StackOverflow post for more information: https://stackoverflow.com/questions/7078022/why-does-pattern-matching-in-scala-not-work-with-variables

  + Patterns are evaluated as follows: An expression `e match { case p1 => e1 ... case pn => en }` matches the value of the selector `e` with the patterns ``p1, ..., pn`` in the order in which they are written. The whole match expression is rewritten to the right-hand side of the first case where the pattern matches matches `e`. Variables in the pattern are replaced by the corresponding parts in the selector.

    + When do patterns match?

      + A constructor pattern `C(p1, ..., pn)` matches all instances of type `C` (or subtypes) that were constructed with arguments matching the patterns `p1, ..., pn`.
      + A variable pattern `x` matches any value, and binds the name of the variable to it.
      + A constant pattern `c` matches values that are equal to `c` (in the sense of `==`)

    + This is what the evaluation of a pattern matching expression looks like:

      ````scala
         eval(Sum(Number(1), Number(2)))

      -> Sum(Number(1), Number(2)) match {
           case Number(n) => n
           case Sum(e1, e2) => eval(e1) + eval(e2)
         }

      -> eval(Number(1)) + eval(Number(2))

      -> Number(1) match {
           case Number(n) => n
           case Sum(e1, e2) => eval(e1) + eval(e2)
         } + eval(Number(2))

      -> 1 + eval(Number(2))

      // ...

      -> 3
      ````

  + Obviously, we could also define the evaluation function as a member function of our base trait `Expr`:

    ````scala
    trait Expr {
      def eval: Int = this match {
        case Number(n) => n
        case Sum(e1, e2) => e1.eval + e2.eval
      }
    }
    ````

    This is equivalent to `eval` as we previously defined it, except that we now pattern match over `this` and write `e1.eval` instead of `eval(e1)`.

  + Now, what is the difference between the pattern-matching-based approach of this lecture and the object-oriented decomposition approach from the lecture before? It comes down to whether we more often create new subclasses, e.g., `Number`, or do we more often create new methods, e.g., `eval`? In the former class, the object-oriented approach is favorable, as we can create a new subclass of `Expr`, e.g., `Var`, and simply implement `eval` in it without having to touch any existing code.  On the other hand, if we mostly add new functions, e.g., `show`, it's easiest to just use pattern matching and add the new function to the base trait or even put it outside of it -- no need to touch any existing classes. This two-dimensional problem (subclasses vs. functions) is called the "expression problem".

+ Lastly, an exercise is given, asking us to define two subclasses of `Expr`, `Var` for variables `x` and `Prod` for products `x * y`. Then, we are asked to implement a method ``show``, that gets operator precedence right, i.e., that it uses as few parentheses as necessary. Here's my shot at it:

  ````scala
  def show(e: Expr): String = e match {
    case Number(x) => x.toString
    case Sum(e1, e2) => show(e1) + " + " + show(e2)
    case Prod(e1, e2) =>
      val lhs = e1 match {
        case Sum(se1, se2) => "(" + show(se1) + " + " + show(se2) + ")"
        case _ => show(e1)
      }
      val rhs = e2 match {
        case Sum(se1, se2) => "(" + show(se1) + " + " + show(se2) + ")"
        case _ => show(e2)
      }
      lhs + " * " + rhs
    case Var(x) => x
  }

  show(Sum(Prod(Number(2), Var("x")), Var("y"))) // 2 * x + y
  show(Prod(Sum(Number(2), Var("x")), Var("y"))) // (2 + x) * y
  ````

### Lecture 4.7 -- Lists

+ As mentioned before, lists are a fundamental data structure in functional programming.

+ A list having `x1`, ..., `xn` as elements is written `List(x1, ..., xn)`.

  ````scala
  val fruit = List("apples", "oranges", "pears")
  val nums  = List(1, 2, 3, 4)
  ````

+ There are two important differences between lists and arrays:

  + Lists are immutable, while arrays are mutable, i.e., you can update its elements.
  + Lists are recursive, while arrays are flat.

+ Like arrays, lists are homogenous. `scala.List` (or, for short, `List`) takes one type parameter `T`, which denotes the type of all the elements of the list.

  ````scala
  val fruit = List[String]("apples", "oranges", "pears")
  val nums  = List[Int](1, 2, 3, 4)
  ````

+ All lists are constructed from the empty list `Nil` and the construction operation `::` (pronounced `cons`), so that ``x :: xs `` results in a new list with `x` being the first element of the list (head), and `xs` being the rest of the list (tail). (In our own list implementation, this would be `new Cons(x, xs)`.)

  ````scala
  val fruit = "apples" :: ("oranges" :: ("pears" :: Nil))
  val nums  = 1 :: (2 :: (3 :: (4 :: Nil)))
  val empty = Nil
  ````

+ By convention, operators ending in `:` associate to the right, thus, `A :: B :: C` can be interpreted as `A :: (B :: C)`: Consequently, we can omit the parentheses and just write:

  ````scala
  val fruit = "apples" :: "oranges" :: "pears" :: Nil
  val nums  = 1 :: 2 :: 3 :: 4 :: Nil
  ````

  Furthermore, operators ending in `:` are also different in that they are seen as method calls of the right-hand operand. So the expression `1 :: 2 :: 3 :: 4 :: Nil` is equivalent to `Nil.::(4).::(3).::(2).::(1)`. (So it really corresponds to the `prepend` method of our own list implementation.)

+ All operations on lists can be defined in terms of the three operations `head`, `tail`, and `isEmpty`, which are defined as methods of objects of the type `List`.

+ Lists can be decomposed using pattern matching:

  + A pattern matching the `Nil` constant: `Nil`
  + A pattern matching a list with a head matching `p` and a tail matching `ps`: `p :: ps` (`List(p1, ..., pn)` is equivalent to `p1 :: ... :: pn :: Nil`.)

  ````scala
  List(1, 2, xs) // Matches lists starting with the elements `1` and `2`,
                 // equivalent to `1 :: 2 :: xs`
  List(x)        // Matches lists with one element, equivalent to `x :: Nil`
  List()         // Matches the empty list, equivalent to `Nil`
  ````

+ The length of a list that is matched by the pattern `x :: y :: List(xs, ys) :: zs` is >= 3, as it matches a list with a first element `x`, a second element `y`, a third element `List(xs, ys)`, which is itself a list, and a tail `zs`.

+ Finally, the insertion sort algorithm for sorting a list of numbers in ascending order is given.

  1. Given a list of numbers to be sorted, we first recursively call the algorithm to sort the tail of the list.
  2. Then, after having obtained a sorted list containing the elements of the tail of the original list in an ascending order,  we insert the head element of the original list into the sorted list at the right position.

  ````scala
  def isort(xs: List[Int]): List[Int] = xs match {
    case List() => List()
    case y :: ys => insert(y, isort(ys))
  }

  def insert(x: Int, xs: List[Int]): List[Int] = xs match {
    case List() => List(x)
    case y :: ys => if (x < y) x :: xs else y :: insert(x, ys)
  }
  ````

  The worst-case complexity of of insertion sort relative to the length of the input list `n` is proportional to `n^2`

  + Looking at `insert` first, the worst case is that `x` is greater than all the elements of the list `xs`, so its steps would be proportional to `n`, where `n` is the length of the list.
  + `isort` calls insert `n` times, where `n` is the length of the original list.

  ​