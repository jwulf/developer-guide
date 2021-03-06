[🏠](README.md) › [Node.js](nodejs.md) › Design patterns & best practices


# Design patterns

- [Modules](#modules)
- [Revealing module pattern](#revealing-module-pattern)
- [Prefer ES6](#prefer-es6)
- [Prefer functional over classical](#prefer-functional-over-classical)
- [Promises for async control flow](#promises-for-async-control-flow)


## Modules  
- Modules are cheap, they can never be too small but they can be too large.  
- Modules should be highly cohesive and decoupled. They should do one thing and do it well.  
- Leverage npm as the largest package management system; don't try and compete with OSS where a package has real world usage in the wild and community contributions.
- The boundaries of responsibility for each module should be informed by the underlying domain (business rules), and not from a developer centric perspective such as MVC.
- Business rules should be isolated from their dependencies (see Dependency Injection) and framework specific conventions (See Imperative Shell, Functional Core)
- Never require individual files from within a module. This violates the [Principle of Least Knowledge](https://en.wikipedia.org/wiki/Law_of_Demeter).

### Bad
The `.js` extension in the require statement is a code smell.

```javascript
const giraffe = require('animals/giraffe.js');
giraffe.create();
...
```

### Best
The animals module exposes an API with the giraffe property.

```javascript
const { giraffe } = require('animals');
giraffe.create();
...
```

- Keep function signatures flexible; once you have more than 2 or 3 arguments use a configuration object instead.

### Bad
This function signature is rigid and will be hard to refactor.  

```javascript
const clients = require(gender, age, location, height);
...
```

### Best
Using the es6 object literal shorthand, we can provide a lightweight flexible object as a single argument.  

```javascript
const clients = require({ gender, age, location, height });
...
```

**Additional reading**  
[Igor Soarez, Anti-patterns in Node.js](https://www.youtube.com/watch?v=pGFQ02qtJ7w)


## Revealing Module Pattern
- Favor the Revealing Module Pattern, as it:
  - Defaults to private assignments
  - Functions are declared in their outer most scope (Avoid nesting!)
  - The return statement is self documenting API

### Bad
Adds cognitive overhead by requiring you to scroll through the business logic to map all the Object properties to the API.

```javascript

const Obj = {
  maybeCacheList(){
    ....
  }
  ...
}

return Obj;
```

### Good
Returning an object literal, but the es5 syntax is getting in the way of readability.  

```javascript

const cacheListInDevelopment = () => ...

return {
  maybeCacheList: cacheListInDevelopment,
  maybeLogCycleData: LogCycleDataInDevelopment,
  writeDomainsList: writeDomainsList,
  ...
};
```

### Best
Take advantage of ES6 object property shorthand by keeping the naming of your functions consistent with your API naming convention.

```javascript

const maybeCacheList = () => ...

return {
  maybeCacheList,
  maybeLogCycleData,
  writeDomainsList,
  ...
};
```

**Further reading**
- [Essential Javascript Design Patterns](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#revealingmodulepatternjavascript)


## Prefer ES6
ES6 allows javascript to be even more expressive and economical in its syntax.

### Destructuring

#### Bad
es5 longhand for assigning alias to object property.  

```javascript
var myObj = require('myObj');
var foo = myObj.foo;
var bah = myObj.bah;
```

#### Best
es6 shorthand much better!

```javascript
const { foo, bah } = require('myObj');
```

#### Fat arrows  
- Are lightweight with no `function`, implied `return` and optional indentation instead of braces.
- Share the lexical this with parent scope.  
- Downside: Fat arrow functions can’t be used as generators or recursion (They're anonymous).

#### Bad
es5 longhand for anonymous callback

```javascript
getUser('Grant', function(grant){
  return grant;
})
```

#### Best
es6 shorthand much better!

```javascript
getUser('Grant', grant => grant);
```

- Fat arrows enables functions to be chained expressions, so closures can appear more readable.

This is a closure. It returns a function that has access to the options variable.
```javascript
module.exports = options => (t, cb) =>
...
```

The chained fat arrows are the equivalent of
```javascript
module.exports = function(options){
  return function(t, cb){}
}
...
```

## Prefer functional over classical

- fp (functional programming) makes programming easier to reason about then classical/OOP (Object Orientated Programming).  

> Favor composition over inheritance

> <cite>Gang of Four (Design Patterns)</cite>

### Factory functions for complex Object creation  
- Use factory functions and not constructors/ES6 classes. The `new` keyword is a [code smell in javascript](https://medium.com/javascript-scene/javascript-factory-functions-vs-constructor-functions-vs-classes-2f22ceddf33e#.7c7g0zr1t).
- Prefer `const` (es6) to assign variables over `var`. Immutable data makes programs easier to reason about as they don't change over time.

### pure functions  
- Keep functions pure where possible. Functions that are referentially transparent (can be replaced with a value) are more testable and easier to understand than functions that operate as sub routines (Performing operations on data that is not the function's input).

### Composition over inheritance  
- Compose simple functions to construct more complex behavior.  
- Avoid nested conditions prefer composition and ternary operators. [flat code is healthy code](https://twitter.com/tjholowaychuk/status/753315132982251520).
- If a functions is hard to name, it could be that it is not adhering to the [single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle).

### Avoid global state  
- Avoid global variables, keep variables local to functions. Prefer to trickle data down a promise chain than have promise handlers refer to variables outside of their scope.

### Avoid "this" and "new"  
- Avoid using "this". Javascript allows functions/methods to be invoked with dynamic context using `apply` and `call`. Likewise avoid new as it is problematic, prefer to create new objects through concatenation and factory functions.

### Unadorned data  

### Recursion > looping  
- `for in` and `for each` loops should be avoided in favor of higher order functions `map`, `reduce` and `filter`.


**additional reading**  
[Tangible Data](https://github.com/omcljs/om/wiki/Conceptual-overview#tangible-data)


## Promises for async control flow
- Promises have been widely adopted in the javascript community and are preferable to the verbose error handling of callbacks.  
- Keep implementation outside of promise chains.
- A promise chain should be higher level, sequential control flow of named functions. Keep it declarative, describe what is happening but not how it happens (implementation).

### Bad
Promise chain interrupted with implementation details

```javascript
maybeCacheList(request.params.list)
  .then(value => {
    if (isCycleData(value)){
      return Promise.resolve(value);
    }
    else {
      return Promise.resolve(cycleData(value))
    }
  })
  .then(writeDomainsList)
```

### Best
Promise chain of sequential named functions

```javascript
maybeCacheList(request.params.list)
  .then(maybeLogCycleData)
  .then(writeDomainsList)
  ...
```

- Promise chains are less useful for complex branching. Here are two ways to handle basic forms of branching:

**1. Split into two chains based on ternary evaluation**  
```javascript
const initDevelop = () =>
  fetchOrLoad()
    .then(maybeProcessMineUpdate)
    .then(flattenData)
    .then(setEnvironment);

const initProd = () =>
  fetchData()
    .then(flattenData)
    .then(setEnvironment);

const init = (isDevelopment) ? initDevelop : initProd;
```

**2. Prefix optional promise handlers with maybe**  
```javascript
maybeFecthData()
  .then(maybeProcessMineUpdate)
  .then(flattenData)
  .then(setEnvironment);
```
