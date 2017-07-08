# OOP in JS without this

### The Problem

Unlike other mainstream languages like Java or C++, JavaScript provides
no language level support for programming in the Object-Oriented paradigm.
There’s no concept of classes, fields or methods in JS (at least not
in the conventional OOP sense). To make things worse, JS actually provide
constructs with same name as those in OOP languages such as `this`,
`new` and as of ES6 `class` as well.

If you come from a conventional OOP language, these things might seem
to do what you think... until you start running into problems that
doesn’t seem to make sense. Many programmers who are just starting to
use OOP in JS have probably seen errors like `can't read property
"foo" of undefined` or unexpected variables being null.
Most of the time this is caused by the use of `this` in "class methods"
not being the object instance since the `this` keyword in JS actually means
"The execution context of a particular function" and not "The reference of
your object instance" like other OOP languages.

When you call `myObject.method()` in JS, it just so happens that the
execution context of the `method` function is `myObject` so it works,
but in cases like `setTimeout(myObject.method, 1000)` this is not
the case and you’ll get strange errors as a result.

With the problem identified, I’ve set myself out to see if I could
find a solution by first breaking down what we actually need when we
say we want OOP. These are the goals I need to accomplish:

- Ease code reuse
- Abstract implementation
- Encapsulate state

### CommonJS Modules

If you don’t need distributed state encapsulation, this sound exactly
like what CommonJS modules do. Take the following Counter singleton
object foe example:

```js
// Counter.js

// "this" is reserved in JS, use self instead.
const self = module.exports;

// private field
const incrementAmount = 1;

// private method
function incrementBy(x) {
  return self.value += x;
}

// public field
self.value = 0;

// public methods
self.reset = function() {
  return self.value = 0;
}
self.count = function() {
  return incrementBy(incrementAmount);
}
```

```js
var c = require('Counter.js');
c.count();
c.count();
c.count();
console.log(c.value); // => 3
c.reset();
console.log(c.value); // => 0
```

This method of emulating objects in JS also has the advantage of
solving Dependency Injection issues since the CommonJS `require` function
acts like a global object cache keyed by the string module name.

But what if we need to do async initialization like a database connection?
This is actually quite easy since JS Promises are chain-able, we just
need to keep a promise to the database connection and define all
query results in terms of that promise:

```js
// Database.js

const self = module.exports;
const sqlite = require('sqlite');

// private field: Promise<Database>
const dbPromise = sqlite.open('./demo.sqlite', { Promise }); 

// public method: get all users from DB
self.getAllUsers = function() {
  return dbPromise.then(
    db => db.all("SELECT * FROM user;")
  );  
}

// public method: execute SQL on the DB
self.run = function(sql) {
  return dbPromise.then(
    db => db.run(sql)
  );  
}
```

```js
var db = require('Database.js');

db.run(
  'CREATE TABLE user (id INT, name TEXT);'+
  'INSERT INTO user (id,name) VALUES (0,"bob");'+
  'INSERT INTO user (id,name) VALUES (1,"alice");'
).then(
  _ => db.getAllUsers()
).then(
  results => console.log(results)
);

// outputs:
// {
//   columns: ["id", "name"],
//   values: [
//     [0, "bob"],
//     [1, "alice"]
//   ]
// }
```

### Wrapping a Constructor

Now that’s all good if you only need singleton objects, but what about
classes and objects? Lexical closures to the rescue! All we need to do
is wrap the whole module in a function and return that (the constructor)
instead, here’s the previous counter as a class:

```js
// CounterClass.js

module.exports = function(initValue = 0) {
  const self = {}; 

  // private field
  const incrementAmount = 1;

  // private method
  function incrementBy(x) {
    return self.value += x;
  }

  // public fields
  self.value = initValue;

  // public methods
  self.reset = function() {
    return self.value = initValue;
  }
  self.count = function() {
    return incrementBy(incrementAmount);
  }

  return self;
}
```

```js
var Counter = require('CounterClass.js');

// the `new` keyword has no effect on this emulated constructor
// it will work either way:
var c1 = new Counter();
var c2 = Counter(42);

console.log([c1.value, c2.value]); // => [0, 42]
```

We can also emulate inheritance behavior by changing the initial
`self` object. So far we’ve always created an empty object with `{}`
but we could actually set it to an instance of the parent class to extend it,
here an example with a `Point` class and a `Circle` class which extends it:

```js
// Point.js

function Point(
  initX = 0,
  initY = 0 
) {

  const self = {}; 

  self.x = initX;
  self.y = initY;

  self.distanceFromOrigin = function() {
    const {x,y} = self;
    return Math.sqrt(x*x + y*y);
  }

  return self;
}

module.exports = Point;
```

```js
// Circle.js

const Point = require('./Point.js');

function Circle(
  initX = 0,
  initY = 0,
  initRadius = 1 
) {

  const self = Point(initX, initY);

  self.radius = initRadius;

  self.area = function() {
    const {radius:r} = self;
    return Math.PI * r*r;
  }

  self.maxY = function() {
    const {y, radius} = self;
    return y + radius;
  }

  return self
}

module.exports = Circle;
```

```js
var Point = require('Point.js');
var Circle = require('Circle.js');

var p = Point(1,2);
var c = Circle(2,3,4);

console.log([
  p,
  c,
  p instanceof Point
  c instanceof Point
  c instanceof Circle
]);

// Outputs:
// [
//   {
//     distanceFromOrigin: [Function anonymous],
//     x: 1,
//     y: 2
//   },
//   {
//     area: [Function anonymous],
//     distanceFromOrigin: [Function anonymous],
//     maxY: [Function anonymous],
//     radius: 5,
//     x: 3,
//     y: 4
//   },
//   false,
//   false,
//   false
// ]
```

### Setting up Types

I the example above, you may have noticed that none if the JS built-in
type checks work. This is because we didn’t construct the objects using
`this` inside the constructor. We can work around this by setting up the
types manually in JS’ prototype chain, here’s the `Point` and `Circle`
class again:

```js
// PointTyped.js

function Point(
  initX = 0,
  initY = 0 
) {

  const self = {};
  // set prototype on 'self'
  Object.setPrototypeOf(self, Point.prototype);

  self.x = initX;
  self.y = initY;

  self.distanceFromOrigin = function() {
    const {x,y} = self;
    return Math.sqrt(x*x + y*y);
  }

  return self;
} 

module.exports = Point;
```

```js
// CircleTyped.js

const Point = require('./PointTyped.js');

function Circle(
  initX = 0,
  initY = 0,
  initRadius = 1 
) {

  const self = Point(initX, initY);
  // set prototype on 'self'
  Object.setPrototypeOf(self, Circle.prototype);

  self.radius = initRadius;

  self.area = function() {
    const {radius:r} = self;
    return Math.PI * r*r;
  }

  self.maxY = function() {
    const {y, radius} = self;
    return y + radius;
  }

  return self
}

// set prototype on 'Circle'
Object.setPrototypeOf(Circle.prototype, Point.prototype);
module.exports = Circle;
```

```js
var Point = require('PointTyped.js');
var Circle = require('CircleTyped.js');

var p = Point(1,2);
var c = Circle(2,3,4);

console.log([
  p instanceof Point
  c instanceof Point
  c instanceof Circle
]);

// Outputs:
// [
//   true,
//   true,
//   true
// ]
```
