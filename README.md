# es6-functional-cookbook

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
