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
