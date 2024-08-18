+++
title = 'Functional Programming With Javascript'
date = 2020-08-18T12:19:57+03:00
draft = false
featured_image = '/images/bg.png'
type = "post"
+++

    
In this series of articles we are going to go deeply into functional programming language concepts using the Javascript programming language.

## What is Functional Programming

Functional Programming is not something new. We can track functional programming back to the 1930s with the introduction of Lambda Calculus by Alonzo Church, far before there was any computer programming language at all. You can read more about lambda calculus here.

### So what exactly is functional programming?

Functional Programming is a declarative style of programming which avoids changing state and mutation of data. In order for us to better understand the benefits of that kind of programming let us first introduce the pitfalls of the stateful style of programming.

## High Order Functions
In functional programming functions are considered first class citizens, i.e. they can be assigned to variables and returned from other functions. A great tool of functional programming is higher order function. A high order function is a function which returns or takes as a parameter another function.

```javascript
function forEachProperty(obj, callback) {
  for (let prop in obj) {
    if (obj.hasOwnProperty(prop)) {
      // callback is a function so we can call it
      callback(obj[prop], prop, obj);
    }
  }
}
```

```javascript
// using it to iterate the window object
forEachProperty(window, val => console.log(val))
```

In this example we create a function **forEachProperty** which iterates an object and executes a particular callback function on each property and value. You've probably used many functions like this but trying to create one might be a completely different experience.


```javascript
function complement(predicate) {
  return function () {
    return !predicate.apply(this, arguments);
  };
}
```

Complement is a simple switcher for predicate functions, for example we can use it for a simple odd\even checking.

```javascript
function even(number) {
  return number % 2 === 0;
}
let odd = complement(even);
```

As you can see, high order functions can be really handy and you probably use them all the time in your programming. Creating your own high order functions can be mind changing and really helps for your future programming goals. Let's review some high order functions we can use in our daily javascript programming.


## Map, Filter, Reduce

**map** is a simple relation from one list to another. It converts a list of elements applying a function f to every element of the list.
```javascript
let list = [1, 2, 3, 4, 5];
// we convert a list applying a square function to every element
let squared = list.map(el => el * el);
squared
> 1, 4, 9, 16, 25
```
**filter** is a function which takes a list and applies a predicate function to every element and returns a new list with only the elements which pass the predicate function
```javascript
let list = [1, 2, 3, 4, 5];
let oddList = list.filter(el => el % 2 !== 0);
oddList
> [1, 3, 5]
```

**reduce** takes a list and reduces it into a single value applying a accumulation function.
```javascript
let list = [1, 2, 3, 4, 5];
let sum = list.reduce((accumulation, current) => accumulation + current, 0);
sum
> 15
```

We also apply a second argument after the lambda function which is the initial value in our reduction.

## Mutable State

As you probably already know, in imperative (procedural) programming languages we have a way of storing state into variables and later reassigning new values into those variables.

```javascript
let car = {
  speed: 140,
  slowdown: function (decrease) {
    this.speed -= decrease;
  }
};
// later we can change that amount using assignment
car.slowdown(20); // making a simple withdraw

module.export = car;
```

As this is a really powerful feature of our programs, it can introduce some bugs when multiple clients are requesting the same object.
This can really lead to unexpected behavior when the same account is accessed by many clients in different modules.
```javascript
// in a separate module without knowing of the first decrease

let car = require('car');
car.slowdown(140);

car.speed;
> -20
```

In this case we make a modification without having the intel of other modifications on the same object. Imagine what happens if we have dozens of modules requiring the same object and changing it's state. This can really lead to bizzare behaviour which will be hard to debug and patch up in the end. To help comes the pure function.

## Pure Function
A pure function is a function where the return value is only determined by its input values, without observable side effects. In other words, if we pass the same parameters more than once, the function should always behave the same. For example all math functions are pure, they take an input and return an output ( sin, cosin, tangent, etc). Also a pure function doesn't make any modifications on a state. Let's try with some examples:

### Pure Function
```javascript
function union(list1, list2) {
  return list2.reduce((accum, curr) => {
    let found = list1.indexOf(el) !== -1;
    return accum.concat(!found ? [curr] : []);
  }, list1);
}
```
### Impure Function

```javascript
function union(list1, list2) {
  list2.forEach(el => {
    if (list1.indexOf(el) !== -1) {
      list1.push(el);
    }
  });
  return list1;
}
```
### Why are pure functions good?
Pure functions are a really good tool in our coding instrumentarium since they are a lot less likely to cause bugs. They always act the same and they don't modify the existing state.

### Functions as first-class objects (citizens)
You've probably heard that in javascript functions are first-class citizens. But what does that mean?
It's the simple fact that functions are also objects and have their own properties, they can be returned as a result from a function and be set to variables.

```javascript
function getCallback(a, b) {
  return c => a + b + c;
}
const sayHello = () => 'hello';

function fibonacci(n) {
  return n < 2 ? n : fibonacci(n - 1) + fibonacci(n - 2);
}

fibonacci.owner = 'Leonardo of Pisa';
fibonacci.foo = 'bar';
```

How is this helpful you may ask. Well, we can use it for a method called memoization.
Memoization is a method of storing already calculated results of a function and already returning those stored values when the user requests them. In our previous example that would be:
```javascript
function fibo(n) {
  if (typeof fibo.memo[n] !== 'undefined') {
    return fibo.memo[n];
  }
  fibo.memo[n] = n < 2 ? n : fibo(n - 1) + fibo(n - 2);
  return fibo.memo[n];
}
```

This technique can come really handy when we are dealing with calculations of high values since most of the calculated results in our recursive tree are already cached in the **memo** array


### Conclusion

In the next post In the next article we are going to look into what closures are and pay attention to more advanced concepts of functional programming.
