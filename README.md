# Effective Javascript

### Chapter 1

Don't combine strict and non-strict files.  

All numbers in JavaScript are double-precision floating-point numbers, that is, the 64-bit encoding of numbers specified by the IEEE 754 standard—commonly known as “doubles.” If this fact leaves you wondering what happened to the integers, keep in mind that doubles can represent integers perfectly with up to 53 bits of precision.  

Floating-point numbers offer a trade-off between accuracy and performance. When accuracy matters, it’s critical to be aware of their limitations. One useful workaround is to work with integer values wherever possible, since they can be represented without rounding.  

Since NaN is the only JavaScript value that is treated as unequal to itself, you can always test if a value is NaN by checking it for equality to itself.  

There are exactly seven falsy values: false, 0, -0, "", NaN, null, and undefined. All other values are truthy. Since numbers and strings can be falsy, it’s not always safe to use truthiness to check whether a function argument or object property is defined.  

Objects are coerced to numbers via `valueOf` and to strings via toString.

Use typeof or comparison to undefined rather than truthiness to test for undefined values.

*Item 4: Prefer Primitives to Object Wrappers*  

You can’t compare the contents of two distinct String objects using built-in operators.

*Item 5: Avoid using == with Mixed Types*  
When the two arguments are of the same type, there’s no difference in behavior between == and ===.

Making conversions explicit ensures that you don’t mix up the coer- cion rules of ==, and—even better—relieves your readers from having to look up the coercion rules or memorize them.  

Javascript tries to be "helpful" and not crash using coercion. Just use `===` and move on.

*Item 7: Think of Strings As Sequences of 16-Bit Code Units*  
JavaScript strings consist of 16-bit code units, not Unicode code points.

### Chapter 2

*Item 8: Minimize Use of the Global Object*  
Feature detection:  

```
if (!this.JSON) { this.JSON = {
parse: ...,
        stringify: ...
    };
}
```
*Item 9: Always Declare Local Variables*

*Item 11: Get Comfortable with Closures*  
JavaScript allows you to refer to variables that were defined outside of the current function

The second fact is that functions can refer to variables defined in outer functions even after those outer functions have returned.  

```
function sandwichMaker() {
  var magicIngredient = "peanut butter";
  function make(filling) {
    return magicIngredient + " and " + filling; }
  return make;
}

var f = sandwichMaker();

f("jelly"); // "peanut butter and jelly"
```

How does this work? The answer is that JavaScript function values contain more information than just the code required to execute when they’re called. They also internally store any variables they may refer to that are defined in their enclosing scopes. _Functions that keep track of variables from their containing scopes are known as closures_. The `make` function is a closure whose code refers to two outer variables: `magicIngredient` and `filling`. Whenever the make function is called, its code is able to refer to these two variables because they are stored in the closure.

A function can refer to any variables in its scope, including the parameters and variables of outer functions. We can use this to make a more general-purpose sandwichMaker:
```
function sandwichMaker(magicIngredient) {
  function make(filling) {
    return magicIngredient + " and " + filling;
  }
  return make;
}
```
var hamAnd = sandwichMaker("ham");
hamAnd("cheese"); // "ham and cheese"
hamAnd("mustard"); // "ham and mustard"
var turkeyAnd = sandwichMaker("turkey");
turkeyAnd("Swiss"); // "turkey and Swiss"
turkeyAnd("Provolone"); // "turkey and Provolone"

The third and final fact to learn about closures is that they can update the values of outer variables. Closures actually store _references_ to their outer variables, rather than _copying_ their values. So updates are visible to any closures that have access to them.

Closures can actually update the values of outer variables

function box() {
  var val = undefined;
  return {
    set: function(newVal) { val = newVal; }, 
    get: function() { return val; },
    type: function() { return typeof val;
  }
}; }

Things to Remember
* Functions can refer to variables defined in outer scopes.
* Closures can outlive the function that creates them.
* Closures internally store references to their outer variables, and can both read and update their stored variables.

*Item 12: Understand Variable Hoisting*  
JavaScript supports lexical scoping: With only a few exceptions, a reference to a variable foo is bound to the nearest scope in which foo was declared. However, JavaScript does not support block scoping: Variable definitions are not scoped to their nearest enclosing statement or block, but rather to their containing function.  

A good way to think about the behavior of JavaScript variable decla- rations is to understand them as consisting of two parts: a declara- tion and an assignment. JavaScript implicitly “hoists” the declaration part to the top of the enclosing function and leaves the assignment in place. In other words, the variable is in scope for the entire function, but it is only assigned at the point where the var statement appears.  

Because redeclarations can lead to the appearance of distinct vari- ables, some programmers prefer to place all var declarations at the top of their functions, effectively hoisting their variables manually, in order to avoid ambiguity.  

* Variable declarations within a block are implicitly hoisted to the top of their enclosing function.
* Redeclarations of a variable are treated as a single variable.
* Consider manually hoisting local variable declarations to avoid
confusion.

*Item 13: Use IIFEs to Create Local Scopes*  

Closures store their outer variables by reference, not by value.

When a parameter is passed by reference, the caller and the callee use the same variable for the parameter. If the callee modifies the parameter variable, the effect is visible to the caller's variable.

When a parameter is passed by value, the caller and callee have two independent variables with the same value. If the callee modifies the parameter variable, the effect is not visible to the caller.

The solution is to force the creation of a local scope by creating a nested function and calling it right away.

This technique, known as the immediately invoked function expression, or IIFE (pronounced “iffy”), is an indispensable workaround for JavaScript’s lack of block scoping.

ES6 Block Scope is The new IIFE: e.g. let and const
http://wesbos.com/es6-block-scope-iife/


*Item 14: Beware of Unportable Scoping of Named Function Expressions*
* Use named function expressions to improve stack traces in Error objects and debuggers.

*Item 15: Beware of Unportable Scoping of Block-Local Function Declarations*
* Always keep function declarations at the outermost level of a pro- gram or a containing function to avoid unportable behavior.

* Item 16: Avoid Creating Local Variables with eval *

* If eval code might create global variables, wrap the call in a nested
function to prevent scope pollution.

To the point, let's look at the dangers in the use of eval(). There are probably many small hidden dangers just like everything else, but the two big risks - the reason why eval() is considered evil - are performance and code injection.

Performance - eval() runs the interpreter/compiler. If your code is compiled, then this is a big hit, because you need to call a possibly-heavy compiler in the middle of run-time. However, JavaScript is still mostly an interpreted language, which means that calling eval() is not a big performance hit in the general case (but see my specific remarks below).

Code injection - eval() potentially runs a string of code under elevated privileges. For example, a program running as administrator/root would never want to eval() user input, because that input could potentially be "rm -rf /etc/important-file" or worse. Again, JavaScript in a browser doesn't have that problem, because the program is running in the user's own account anyway. Server-side JavaScript could have that problem.


### Chapter 3

*Item 18: Understand the Difference between Function, Method, and Constructor Calls*  
* Constructors are called with new and receive a fresh object as their receiver.

* Item 19: Get Comfortable Using Higher-Order Functions *  
Higher-order functions are nothing more than functions that take other functions as arguments or return functions as their result.

* Item 20: Use call to Call Methods with a Custom Receiver *
* Use the call method to call a function with a custom receiver.
* Use the call method for calling methods that may not exist on a
given object.
* Use the call method for defining higher-order functions that allow clients to provide a receiver for the callback.


*Item 21: Use apply to Call Functions with Different Numbers of Arguments*
* Use the apply method to call variadic functions with a computed array of arguments.
* Use the first argument of apply to provide a receiver for variadic methods.

*Item 22: Use arguments to Create Variadic Functions*  
JavaScript provides every function with an implicit local variable called arguments.  

*Item 23: Never Modify the arguments Object*
As a consequence, it is much safer never to modify the arguments object. This is easy enough to avoid by first copying its elements to a real array. A simple idiom for implementing the copy is:
var args = [].slice.call(arguments);

* Use the implicit arguments object to implement variable-arity functions.
* Consider providing additional fixed-arity versions of the variadic functions you provide so that your consumers don’t need to use the apply method.

* Item 25: Use bind to Extract Methods with a Fixed Receiver *  
Creating a version of a function that binds its receiver to a specific object is so common that ES5 added library support for the pattern. Function objects come with a bind method that takes a receiver object and produces a wrapper function that calls the original function as a method of the receiver.  

* Use bind as a shorthand for creating a function bound to the appro- priate receiver.

*Item 26: Use bind to Curry Functions*  
All functions in Haskell take one argument and return one result. Currying refers to the nesting of multiple functions, each accepting one argument and returning one result, to allow the illusion of multiple-parameter functions.
P126 Haskell Book

Partial application - encourages the reuse of functions.

### Chapter 4
Like many object-oriented languages, JavaScript provides support for implementation inheritance: the reuse of code or data through a dynamic delegation mechanism. But unlike many conventional languages, JavaScript’s inheritance mechanism is based on prototypes rather than classes.

In many languages, every object is an instance of an associated class, which provides code shared between all its instances. JavaScript, by contrast, has no built-in notion of classes. Instead, objects inherit from other objects. Every object is associated with some other object, known as its prototype. Working with prototypes can be different from classes, although many concepts from traditional object-oriented languages still carry over.

_What’s the Difference Between Class & Prototypal Inheritance?_

Class Inheritance: A class is like a blueprint — a description of the object to be created. Classes inherit from classes and create subclass relationships: hierarchical class taxonomies.  

The ES6 `class` keyword desugars to a constructor function:
```class Foo {}
   typeof Foo // 'function'
```
`class` keyword just uses prototypal inheritence under the hood.

JavaScript’s class inheritance uses the prototype chain to wire the child `Constructor.prototype` to the parent `Constructor.prototype` for delegation. Usually, the `super()` constructor is also called. Those steps form single-ancestor parent/child hierarchies and create the tightest coupling available in OO design.

The `super` keyword is used to access and call functions on an object's parent.
A `super` call in the constructor calls the constructor of the parent class.

Prototypal Inheritance: A prototype is a working object instance. Objects inherit directly from other objects. Instances may be composed from many different source objects, allowing for easy selective inheritance and a flat [[Prototype]] delegation hierarchy. In other words, _class taxonomies are not an automatic side-effect of prototypal OO: a critical distinction._  

Whenever a function object is created in JS, it has the `prototype` property.

Three types:  
1) Delegate prototype: if you don't find what you want on the object, you go and find the object's prototype.

Create a constructor function. Attach to functions prototype property. Then you instantiate a new object with the `new` keyword.

You technically don't need `new` to create an object. This just exists in the  language to make it look familiar to people who are used to constructors in other languages.

```
  function User(name, passwordHash) {
    this.name = name;
    this.passwordHash = passwordHash;
  }

  User.prototype.toString = function() { 
    return "[User " + this.name + "]";
  };

  User.prototype.checkPassword = function(password) {
    return hash(password) === this.passwordHash;
  };

  var u = new User("sfalken", "0ef33ae791068ec64b502d6cb0191387");
```
`new`
1) creates a new Object
2) sets the prototype of new Object to be the the prototype of the constructor function
3) it will call the constructor with the new object created in step 1 as the `this` variable. execute the constructor with `this`.
4) return the created object

```
  function new(constructor, arguments){
    var obj = {}
    Object.setPrototypeOf(obj, constructor.prototype)
    var argsArray = Array.from(arguments)
    // var argsArray = Array.prototype.slice.apply(arguments)
    return constuctor.apply(obj, argsArray.slice(1)) || obj
  }
```

with Object.create

```
var proto = {
  hello: function hello(){
    return "Hello my name is " + this.name
  }
}

var george = Object.create(proto)
george.name = "George"
```

2) Cloning and Concatenation  
The process of inheriting features directly from one object to another by copying the source objects properties. 
Great for default state, mixins.  

This is a defaults and overrides solution. 

Mixin style:
```  
var proto = {
  hello: function hello(){
    return "Hello my name is " + this.name
  }
}

Object.assign(proto, {name: "George"})
```
Common use case is to turn anything into an event emitter: `Object.assign(whatever, Backbone.events)`  

3) Functional Inheritance  
In JavaScript, any function can create an object. When that function is not a constructor (or `class`), it’s called a factory function. Functional inheritance works by producing an object from a factory, and extending the produced object by assigning properties to it directly (using concatenative inheritance).  

Good for data privacy and encapsulation.  
Functional mixins.  

Function prototypes: can replace constructors, init functions, super  

Uses concatenative inheritance under the hood
```
const BassAmp = (options) => {
  return Object.assign({}, lowCut, volume, cabinet, options);
};

const murderRobotDog = (name) => {
  let state = {
    name,
    speed: 100,
    position: 0
  }
  return Object.assign(
    {},
    barker(state),
    driver(state),
    killer(state)
  )
}
```  
Basically what you do with higher-order components in React.  


_Inheritance: Is when you design your types around what they are_
```
Robot
  .drive()
  
  MurderRobot
    .kill()

Animal
  .poop()
  
  Dog
    .bark()
    
  Cat
    .meow()
```

_Composition: Design your types around what they do_  

```
  dog = pooper + barker
  cat = pooper + meower
  murderRobotDog = driver + killer + barker
```

Inheritance is fundamentally a code reuse mechanism: A way for different kinds of objects to share code. The way that you share code matters because if you get it wrong, it can create a lot of problems, specifically:  

* Class inheritance creates parent/child object taxonomies as a side-effect.
* The tight coupling problem (class inheritance is the tightest coupling available in oo design), which leads to the next one…
* The fragile base class problem (The fragile base class problem is a fundamental architectural problem of object-oriented programming systems where base classes (superclasses) are considered "fragile" because seemingly safe modifications to a base class, when inherited by the derived classes, may cause the derived classes to malfunction.)
* Inflexible hierarchy problem (eventually, all evolving hierarchies are wrong for new uses)
* The duplication by necessity problem (due to inflexible hierarchies, new use cases are often shoe-horned in by duplicating, rather than adapting existing code)
* The Gorilla/banana problem (What you wanted was a banana, but what you got was a gorilla holding the banana, and the entire jungle)

_Program to an interface and not an implementation_

_The solution to all of these problems is to favor object composition over class inheritance._

More flexible to use composition from the start: you can't predict the future.

* MPJ: Factories vs. `this` and `new` *  
Factories are simply functions that create objects. You can use factories instead of using classes, and factories are simpler than classes and easier to reason about.

Closures: function body has access to variables defined outside the function. Closure actually reads the variable at the time that it is called. Good use case: start a task, specify something that happens when task is done with things that are available to you when the task is started.  
 In other words, a closure gives you access to an outer function’s scope from an inner function. In JavaScript, closures are created every time a function is created, at function creation time.  
The inner function will have access to the variables in the outer function scope, even after the outer function has returned.  
In functional programming, closures are frequently used for _partial application & currying_. Partial application fixes (partially applies the function to) one or more arguments inside the returned function, and the returned function takes the remaining parameters as arguments in order to complete the function application. Arguments become fixed parameters remembered inside the closure scope.

```
const dog = () => {
  const sound = 'woof'
  return {
    talk: () => console.log(sound)
  }
}
const sniffles = dog()
sniffles.talk() // Outputs: "woof"
```

* Item 30: Understand the Difference between prototype, getPrototypeOf, and __proto__ *  

* C.prototype is used to establish the prototype of objects created by new C().
* Object.getPrototypeOf(obj) is the standard ES5 mechanism for retrieving obj’s prototype object.
* obj.__proto__ is a nonstandard mechanism for retrieving obj’s prototype object.

*Item 31: Prefer Object.getPrototypeOf to __proto__*



*Item 35: Use Closures to Store Private Data*
```
function User(name, passwordHash) {
  this.toString = function() {
    return "[User " + name + "]";
  };
  this.checkPassword = function(password) {
    return hash(password) === passwordHash;
  };
}
```

* Closure variables are private, accessible only to local references.
* Use local variables as private data to enforce information hiding
within methods.


*Item 37: Recognize the Implicit Binding of this*
The solution is straightforward enough: Just use a lexically scoped variable to save an additional reference to the outer binding of this.  
var self = this; // save a reference to outer this-binding


* The scope of this is always determined by its nearest enclosing function.
* Use a local variable, usually called self, me, or that, to make a this-binding available to inner functions.


### Chapter 5: Arrays and Dictionaries

*Item 43: Build Lightweight Dictionaries from Direct Instances of Object*

* Use object literals to construct lightweight dictionaries.
* Lightweight dictionaries should be direct descendants of Object.prototype to protect against prototype pollution in for...in loops.

*Item 44: Use null Prototypes to Prevent Prototype Pollution*
* In ES5, use Object.create(null) to create prototype-free empty objects that are less susceptible to pollution.

*Item 46: Prefer Arrays to Dictionaries for Ordered Collections*  
* Avoid relying on the order in which for...in loops enumerate object properties.

*Item 47: Never Add Enumerable Properties to Object.prototype*
* Avoid adding properties to Object.prototype.
* Consider writing a function instead of an Object.prototype method.
* If you do add properties to Object.prototype, use ES5’s Object.defineProperty to define them as nonenumerable properties.

*Item 48: Avoid Modifying an Object during Enumeration*
* Avoid relying on the order in which for...in loops enumerate object properties.
* If you aggregate data in a dictionary, make sure the aggregate operations are order insensitive.
* Use arrays instead of dictionary objects for ordered collections.

* Item 49: Prefer for Loops to for...in Loops for Array Iteration *
A corallary is to use the ES6 `for-of` loop:  

```
let list = [8, 3, 11, 9, 6];

for (let value of list) {
  console.log(value);
}

for (const x of ['a', '', 'b']) {
    if (x.length === 0) break;
    console.log(x);
}

const arr = ['a', 'b'];
for (const [index, element] of arr.entries()) {
    console.log(`${index}. ${element}`);
}
```

We get the succinct syntax of for-in, the run-to-completion of forEach, and the ability to break, continue, and return of the simple for loop. Now JavaScript has a loop control structure that is just as succinct as what you will find in Python, C# or Java.


* Always use a for loop rather than a for...in loop for iterating over the indexed properties of an array.
* Consider storing the length property of an array in a local vari- able before a loop to avoid recomputing the property lookup.

*Item 50: Prefer Iteration Methods to Loops*
```
// If you need access to the index
const arr = ['a', 'b'];
for (const [index, element] of arr.entries()) {
    console.log(`${index}. ${element}`);
}
```
* use `map` `filter` `for-of` instead of `for` loops

*Item 51: Reuse Generic Array Methods on Array-Like Objects*
`for-of` works with non-Array objects. Other existing collections like the DOM NodeList object, the arguments object, and strings also work with for-of. Just like with arrays, this makes it a little bit easier to iterate over these non-array objects.

Convert `arguments` to an array:
`[].slice.call(arguments)`

*Item 52: Prefer Array Literals to the Array Constructor*  


### Chapter 6: Library and API design 

*Item 53: Maintain Consistent Conventions*  
One of the key conventions is argument order.  
Unless you have a really strong reason for needing to vary from universal practice, stick with what’s familiar.   
If your API uses options objects (see Item 55), you can avoid the dependence on argument order.  

* Use consistent conventions for variable names and function signatures.
* Don’t deviate from conventions your users are likely to encounter in other parts of their development platform.

*Item 54: Treat undefined As "No Value"*

Undefined value means the:

• Variable used in the code doesn’t exist
• Variable is not assigned to any value
• Property doesn’t exist

es6 default parameters can solve the undefined argument issue:  
```
function multiply(a, b = 1) {
  return a * b;
}
```

* Avoid using undefined to represent anything other than the absence of a specific value.
* Use descriptive string values or objects with named boolean proper- ties, rather than undefined or null, to represent application-specific flags.
* Test for undefined instead of checking arguments.length to provide parameter default values.
* Never use truthiness tests for parameter default values that should allow 0, NaN, or the empty string as valid arguments.

*Item 55: Accept Options Objects for Keyword Arguments*
* Use options objects to make APIs more readable and memorable.
* The arguments provided by an options object should all be treated
as optional.

*Item 56: Avoid Unnecessary State*  

While state is sometimes essential, stateless APIs tend to be easier to learn and use, more self-documenting, and less error-prone.

 The stateful API requires modifying the internal state of a canvas in order to do anything custom, and this causes one drawing operation to affect another one, even if they have nothing to do with each other. For example, the default fill style is black. But you can only count on getting the default value if you know that no one has changed the defaults already.

A stateless API leads to more modular code, which avoids bugs based on surprising interactions between different parts of your code, while simultane- ously making the code easier to read.

stateless: you don’t have to understand all the modifications that precede it.

Another benefit of stateless APIs is conciseness. A stateful API tends to lead to a proliferation of additional statements just to set the inter- nal state of an object before calling its methods. 

* Prefer stateless APIs where possible.
* When providing stateful APIs, document the relevant state that
each operation depends on.

*Item 57: Use Structural Typing for Flexible Interfaces*__
This kind of interface is sometimes known as structural typing or duck typing: Any object will do so long as it has the expected structure (if it looks like a duck, swims like a duck, and quacks like a duck...). It’s an elegant programming pattern and especially lightweight in dynamic languages such as JavaScript, since it doesn’t require you to write anything explicit. A function that calls methods on an object will work on any object that implements the same interface. Of course, you should list out the expectations of an object interface in your API documentation. 


* Use structural typing (also known as duck typing) for flexible object interfaces.
* Avoid inheritance when structural interfaces are more flexible and lightweight.
* Use mock objects, that is, alternative implementations of interfaces that provide repeatable behavior, for unit testing.


*Item 58: Distinguish between Array and Array-Like*
overloaded: You can pass it either an index or an array of indices.


APIs should never overload structural types with other overlapping types. - e.g. don't take both arrays and objects as arguments, since they are both technically objects.

* Never overload structural types with other overlapping types.
* When overloading a structural type with other types, test for the
other types first.
* Accept true arrays instead of array-like objects when overloading with other object types

*Item 59: Avoid Excessive Coercion*
Coercions can certainly be convenient. They can also cause trouble, hiding errors and leading to erratic and hard-to-diagnose behavior.

Defensive programming attempts to defend against potential errors with additional checks.

Functional Note: You wouuldn't have to do this with static types.

* Avoid mixing coercions with overloading.
* Consider defensively guarding against unexpected inputs.

*Item 60: Support Method Chaining*
Method chaining can be used whenever an API produces objects of some interface (see Item 57) with methods that produce more objects, often of the same interface. 

```
var users = records.map(function(record) { 
                      return record.username;
                    })
                    .filter(function(username) {
                      return !!username; 
                    })
                    .map(function(username) {
                      return username.toLowerCase();
                    });
```
Functional Note: This is the idea behind composition. Underscore gets it wrong by not thinking about currying and partial function application.


* Use method chaining to combine stateless operations.
* Support method chaining by designing stateless methods that pro-
duce new objects.
* Support method chaining in stateful methods by returning this.

### Chapter 7 Concurrency

JavaScript’s approach to writing programs that respond to multiple concurrent events is remarkably user-friendly and powerful, using a combination of a simple execution model, sometimes known as _event-queue_ or _event-loop_ concurrency, with what are known as asynchronous APIs.

*Item 61: Don’t Block the Event Queue on I/O*

Functions such as downloadSync are known as synchronous, or blocking: The program stops doing any work while it waits for its input.  

In JavaScript, most I/O operations are provided through asynchronous, or nonblocking APIs. Instead of blocking a thread on a result, the programmer provides a callback for the system to invoke once the input arrives.  

The system does not just jump right in and call the callback the instant the download completes. JavaScript is sometimes described as providing a run-to-completion guarantee: Any user code that is currently running in a shared context, such as a single web page in a browser, or a single running instance of a web server, is allowed to finish executing before the next event handler is invoked. In effect, the system maintains an internal queue of events as they occur, and invokes any registered callbacks one at a time.

The benefit of the run-to-completion guarantee is that when your code runs, you know that you have complete control over the application state: You never have to worry that some variable or object property will change out from under you due to concurrently executing code.

The single most important rule of concurrent JavaScript is never to use any blocking I/O APIs in the middle of an application’s event queue. 

Asynchronous APIs are safe for use in an event-based setting, because they force your application logic to continue processing in a separate “turn” of the event loop.

In the asynchronous version, the JavaScript code registers an event handler and returns immediately, allowing other event handlers to process intervening events before the download completes.

In a server setting, blocking APIs are unproblematic during startup, that is, before the server begins responding to incoming requests. But when servicing requests, blocking APIs are every bit as catastrophic as in the event queue of the browser.

* Asynchronous APIs take callbacks to defer processing of expensive operations and avoid blocking the main application.
* JavaScript accepts events concurrently but processes event handlers sequentially using an event queue.
* Never use blocking I/O in an application’s event queue.

*Item 62: Use Nested or Named Callbacks for Asynchronous Sequencing*
AKA callback hell

* Use nested or named callbacks to perform several asynchronous operations in sequence.

*Item 63: Be Aware of Dropped Errors*
One of the more difficult aspects of asynchronous programming to manage is error handling. In synchronous code, it’s easy to handle errors in one fell swoop by wrapping a section of code with a try block.

In fact, asynchronous APIs cannot even throw exceptions at all, because by the time an asynchronous error occurs, there is no obvious execution context to throw the exception to! Instead, asynchronous APIs tend to represent errors as special arguments to callbacks, or take additional error-handling callbacks (sometimes referred to as errbacks).

```
function asyncTask(cb) {

   asyncFuncA.then(AsyncFuncB)
      .then(AsyncFuncC)
      .then(AsyncFuncD)
      .then(data => cb(null, data)
      .catch(err => cb(err));
}
```

```
async function asyncTask(cb) {
    try {
       const user = await UserModel.findById(1);
       if(!user) return cb('No user found');
    } catch(e) {
        return cb('Unexpected error occurred');
    }
}
```

One of the practical differences between try...catch and typical error-handling logic in asynchronous APIs is that try makes it eas- ier to define “catchall” logic so that it’s difficult to forget to handle errors in an entire region of code. With asynchronous APIs like the one above, it’s very easy to forget to provide error handling in any of the steps of the process.  Often, this results in an error getting silently dropped. 

* Avoid copying and pasting error-handling code by writing shared error-handling functions.
* Make sure to handle all error conditions explicitly to avoid dropped errors.

*Item 64: Use Recursion for Asynchronous Loops*
JavaScript environments usually reserve a fixed amount of space in memory, known as the call stack, to keep track of what to do next after returning from function calls.

When a program is in the middle of too many function calls, it can run out of stack space, resulting in a thrown exception. This condition is known as stack overflow. 

* Loops cannot be asynchronous.
* Use recursive functions to perform iterations in separate turns of
the event loop.
* Recursion performed in separate turns of the event loop does not overflow the call stack.

*Item 65: Don’t Block the Event Queue on Computation*

This has the effect of adding the callback to the event queue almost right away:
setTimeout(next, 0); // schedule the next iteration in the event queue


*Item 66: Use a Counter to Perform Concurrent Operations*
Concurrent events are the most important source of nondeterminism in JavaScript. Specifically, the order in which events occur is not guaran- teed to be the same from one execution of an application to the next.

* Events in a JavaScript application occur nondeterministically, that is, in unpredictable order.
* Use a counter to avoid data races in concurrent operations.

*Item 67: Never Call Asynchronous Callbacks Synchronously*
The purpose of asynchronous APIs is to maintain the strict separation of turns of the event loop. As Item 61 explains, this simplifies concur- rency by alleviating code in one turn of the event loop from having to worry about other code changing shared data structures concur- rently. An asynchronous callback that gets called synchronously vio- lates this separation, causing code intended for a separate turn of the event loop to execute before the current turn completes.

We use the common library function setTimeout to add a callback to the event queue after a minimum timeout. There may be preferable alternatives to setTimeout for scheduling immediate events, depending on the platform.

```
var cache = new Dict();
function downloadCachingAsync(url, onsuccess, onerror) {
  if (cache.has(url)) {
    var cached = cache.get(url);
    // adds the callback to the end of the event queue so that it behaves like a predictable async function.
    setTimeout(onsuccess.bind(null, cached), 0); return;
  }
  return downloadAsync(url, function(file) {
    cache.set(url, file);
    onsuccess(file);
  }, onerror);
}
```

* Never call an asynchronous callback synchronously, even if the data is immediately available.
* Calling an asynchronous callback synchronously disrupts the expected sequence of operations and can lead to unexpected inter- leaving of code.
* Calling an asynchronous callback synchronously can lead to stack overflows or mishandled exceptions.
* Use an asynchronous API such as setTimeout to schedule an asyn- chronous callback to run in another turn.




















