---
layout: post
title: Functional Programming With Javascript. Part 1
modified:
categories:
excerpt:
tags: []
image:
  feature:
date: 2015-10-04T23:11:25+03:00
---

In these series of articles we are going to go into deeply into the functional programming language concepts using the Javascript programming language.


#What is Functional Programming

Functional Programming is not something new at all. We can track functional programming back to 1930s with the introduction of Lambda Calculus by Alonzo Church, far before there was any computer programming language at all. You can read more about lambda calculus here.

So what exactly is functional programming? Functional Programming is a declarative style of programming which avoids changing state and mutation of data. In order for us to better understand the benefits of that kind of programming let us first introduce the pitfalls of stateful style of programming.

#Mutable State

As you probably already know, in imperative (procedural) programming languages we have a way of storing state into variables and later reasigning new values into those variables.

{% highlight js %}
  var car = {
   speed: 140,
   slowdown: function (decrease) {
     this.speed -= decrease;
   }
  };
  // later we can change that amount using assignment
  car.slowdown(20); // making a simple withdraw

  module.export = car;
{% endhighlight%}

As this is a really powerful feature of our programs it can introduce some bugs when multiple clients are requesting the same object.
This can really lead to unexpected behaviour when the same account is accessed by many clients in different modules.
```javascript
  // in a separate module without knowing of the first decrease
  var car = require('car');
  car.slowdown(140);

  car.speed;
  > -20
```

In this case we make a modification without having the intel of other modifications on the same object. Imagine what happens if we have dozens of modules requiring the same object and changing it's state. This can really lead to bizzaire behaviour which will be hard to debug and patch up in the end. On help comes the pure function.

#Pure Function
A pure function is a function where the return value is only determined by its input values, without observable side effects. In other words, if we pass the same parameters more than once, the function should always behave the same. For example all math functions are pure, they take an input and return an output ( sinus, cosinus, tangent, etc). Also a pure function doesn't make any modifications on a state. Lets try with some examples:

## Pure Function
function union(a, b) {
  return b.reduce((accum, curr) => {
    var found = а.indexOf(el) !== -1;
    return accum.concat(!found ? [curr] : []);
  }, a);
}

## Impure Function
``` javascript
function union(a, b) {
  b.forEach(el => {
    if (a.indexOf(el) !== -1) {
      a.push(el);
    }
  });
  return a;
}
```

###Why are pure functions good?
Pure functions are really good tool in our coding instrumentarium since they are a lot less to cause bugs. They always act the same and they don't modify existing state.

# High Order Functions
Another great tool of the functional and programming at all is the high order function. A high order function is a function which returns or takes as a parameter another function.

``` javascript
function forEachProperty(obj, callback) {
  for (let prop in obj) {
    if (obj.hasOwnProperty(prop)) {
      // callback is a function so we can call it
      callback(obj[prop], prop, obj);
    }
  }
}

// using it to iterate the window object
forEachProperty(window, (val) => console.log(val))
```

## In this example we create a function forEachProperty which iterates an object and executes a particular callback function on each property and value. You've probably used many functions like this but trying to create one might be a completely different experience.

```javascript
  function complement(predicate) {
    return function () {
      return !predicate.apply(this, arguments);
    };
  }
```

### Complement is a simple switcher for predicate functions, for example we can use it for a simple odd\even checking.

```javascript
  function even(number) {
    return number % 2 === 0;
  }
  let odd = complement(even);
```

### As you can see high order functions can be really handy and you are probably using them all the time in your programming. Creating your own high order functions can be mind changing and really helps in your future programming.
