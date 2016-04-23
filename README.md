# The ECMAScript6 Functional Cookbook

This document provides a set of tools and tricks for writing functional code (as in _functional programming_) in ECMAScript6—the latest JavaScript standard.

The code in this document is licensed under the WTFPL, which basically means you're free to copy and paste it, modify it, redistribute it, relicense it, and use it however you want. This code comes with ABSOLUTELY NO WARRANTY.

## What is this, how you say, Functional Programming?

To qualify as "functional", a language must treat functions as first-class objects. That means that functions can be assigned to variables, passed to other functions as arguments, and returned from functions.

Here's how that looks in JavaScript:

```javascript
var addOne = function(x) {
  return x + 1
}

var add1 = addOne // assigning a function to a variable. After this line, `add1` and `addOne` are synonyms.

var callWithArg = function(func, arg) { // callWithArg takes a function as the first parameter
  return func(arg)
}

callWithArg(add1, 1) // returns 2
```

The functions in functional languages always have two properties that make them very versatile:

- A function can be _anonymous_—that is, it need not be bound to any particular name. When we see code like
  
  ```javascript
  var sum = 1 + 2 + 3
  ```

  We're not surprised that the values `1`, `2`, and `3` aren't referenceable by variable names. In JavaScript, functions are values just like numbers are, and don't need names to be useful:

  ```javascript
  document.body.addEventListener('click', function() {
    alert('Got clicked!')
  })
  ```
- A function's implementation can refer to any variable that's in scope when the function is defined. It holds that reference even after the variable goes out of scope, preventing its value from being garbage-collected. Such functions are called _closures_ because they "enclose" all the variables that are in scope, shielding them from the wrath of the garbage collector.

  ```javascript
  var createGreeter = function(greeting) {
      var greet = function(name) {
          return greeting + " " + name
      }

      return greet
  }

  var sayHello = createGreeter("hello")

  sayHello('satoshi') // returns 'hello satoshi'
  ```

Functions with these two properties are often called _lambdas_. This term is borrowed from the lambda calculus, a mathematical formalism that preceded the implementation of these concepts in programming languages.

## Why should you care? 

Code written in a functional style has some nice properties.

- **Data in, data out**: A _pure function_ takes a value and returns a value, with no side effects. This makes it very easy to test, since there are no side-effecting dependencies to mock.
- **Referential transparency**: Calling a pure function is equivalent to copy-pasting its implementation into the callsite and substituting the values of its arguments for their parameter names. This makes functional code easier to reason about and easier to refactor; you never have to worry about whether it's safe to inline a function call.
- **Nilpotency**: Since pure functions have no side effects, caching their results has no effect on the correctness of your code. It's easy to optimize functional code if you notice performance problems.

In addition, getting good at functional programming will make you a better programmer in general.

- You'll learn to decompose complex tasks into small building blocks that can be reused.
- You'll add new patterns to your programming vocabulary which will help you design systems in *any* language or framework, not just functional ones.

## What's so great about ES6?

It's always been possible to write functional code in JavaScript, but it hasn't been *fun* until ECMAScript 6.

ES6 introduces many, many features to JavaScript, but the ones that are most relevant for functional programming are **arrow functions**, **destructuring assignment**, and **tail call optimization**.

### Arrow Functions

Arrow functions are like traditional JavaScript functions, but provide a very terse syntax (they also provide lexical binding of the `this` keyword, but that's irrelevant for FP purposes).

```javascript
// before ES6

var square = function(x) {
  return x * x
}

// after
 
let square = x => x * x
```

The arrow syntax degrades gracefully in more general cases. When the body of the function has multiple statements, the curly braces and `return` keyword come back:

```javascript
var cube = x => {
  let squared = square(x)

  return squared * x
}
```

When there are multiple parameters, they need to be surrounded by parens. Parens are also required when there are no parameters.

```javascript
let add = (x, y) => x + y

let getName = () => 'satoshi'
```

As the `square` function above illustrates, the arrow syntax works best when the function has exactly one parameter and its implementation is a single expression. Keeping in mind that arrow functions are themselves expressions, can you figure out what the code below does?

```javascript
let greet = greeting => name => greeting + ' ' + name
```

The `greet` function is identical in functionality to the `createGreeter` example above. The `=>` operator is right-associative, so the example parses as:

```javascript
let greet = (greeting => (name => (greeting + ' ' + name)))
```

We can call the `greet` function like this:

```javascript
greet('hello')('satoshi') // returns 'hello satoshi'
```

Functions that use this pattern of a series of one-argument calls are called _curried_ functions, after the mathematician Haskell Curry.

Why use curried functions when we could just make a function that takes two arguments, like `greet('hello', 'satoshi')`? The advantage of currying is that the intermediate result is often useful in itself, and can be saved for later use by assigning it to a variable.

```javascript
let greetInEnglish = greet('hello')
let greetInJapanese = greet('konnichiwa')

greetInJapanese('satoshi-san') // returns 'konnichiwa satoshi-san'
```

Rather than duplicating the strings for different greetings all over your codebase, you can create a function for each type of greeting.

This is only a minor advantage of curried functions, however. Their real power lies in their _composability_. Many examples of this appear in the cookbook.

### Destructuring Assignment

**Destructuring Assignment** allows you to easily pull values out of objects and arrays and assign them to local variables.

```javascript
let formatLocation = ([lat, long]) => lat + ', ' + long

let sanFrancisco = [37.7749, -122.4194]

formatLocation(sanFrancisco)
```

The **spread operator** `...` extracts the remaining values from an array. This is extremely useful for writing recursive algorithms.

```javascript
let joinOnCommas = (items) => {
  let [first, second, ...rest] = items
  if (items.length === 0) {
    return ''
  } else if (items.length === 1) {
    return first
  } else if (items.length === 2) {
    return first + ', ' + second
  } else {
    return joinOnCommas([first + ', ' + second, ...rest])
  }
}

joinOnCommas(['Kermit', 'Piggy', 'Gonzo']) // returns 'Kermit, Piggy, Gonzo'
```

### Tail Call Optimization

Recursive algorithms in general have the unfortunate property that they can grow the call stack to arbitrary lengths, potentially causing the program to run out of stack memory and crash. Tail call optimization provides a way around this problem.

A function call to _f_ is in _tail position_ if its caller doesn't do anything after calling _f_ (returning _f_'s return value doesn't count as doing something). If a function is called in tail position, its caller's stack frame can be popped before the new stack frame is added. This keeps the stack at a constant size. This trick is called **tail call optimization**.

ES6 adds this optimization to JavaScript, allowing recursive algorithms to operate on very large data structures.

## Creating Pipelines

```javascript
let createPipeline = input => {
    let pipe = fn => createPipeline(fn(input))

    pipe.and = pipe // syntactic sugar
    pipe.value = input

    return pipe
}

let take = createPipeline
```

**now you can:**

```javascript
let add = a => b => a + b

take(3).and(add(4)).value // 7
```

## Get

```javascript
let get = property => object => object[property]
```

**now you can:**

```javascript
let muppet = {name: 'Kermit'}

take(muppet).and(get('name')).value // 'Kermit'
```

## Map

```javascript
let map = fn => array => array.map(fn)
```

**now you can:**

```javascript
let forEach = map

let muppets = [
  {name: 'Kermit'},
  {name: 'Piggy'},
  {name: 'Gonzo'}
]

take(muppets).and(forEach(get('name'))).value // ['Kermit', 'Piggy', 'Gonzo']
```

## Filter

```javascript
let filter = predicate => array => array.filter(predicate)
```

**now you can:**

```javascript
let where = filter

let candidates = [
    {name: 'Hillary', party: 'D'},
    {name: 'Ted',     party: 'R'},
    {name: 'Bernie',  party: 'D'},
    {name: 'Donald',  party: 'R'}
]

let isDemocrat = candidate => candidate.party === 'D'

take(candidates)
    .and(where(isDemocrat))
    (forEach(get('name')))
    .value // ['Hillary', 'Bernie']
```

## Fold

```javascript
let fold = combine2 => initial => array => {
    return array.reduce((a, b) => combine2(a)(b), initial)
}
```

**now you can:**

```javascript
let add = a => b => a + b
let sum = fold(add)(0)
sum([1,2,3,4]) // 10

let multiply = a => b => a * b
let product = fold(multiply)(1)
product([1,2,3,4]) // 24

let both = p => q => p && q
let all = fold(both)(true)
all([true, true, true]) // true
all([true, false, true]) // false

let either = p => q => p || q
let any = fold(either)(false)
any([false, true, false]) // true
any([false, false, false]) // false

let join2 = s1 => s2 => s1 + s2
let join = fold(join2)('')
```

## Intersperse

```javascript
let intersperse = delimiter => ([first, ...rest]) =>
    rest.length
        ? [first, delimiter, ...intersperse(delimiter)(rest)]
        : [first]
```

**now you can:**

```javascript
let str = fold(add)('') // joins together an array of strings
let expandIt = intersperse(' ')

take('hello satoshi').and(expandIt)(str).value // 'h e l l o   s a t o s h i'
```
