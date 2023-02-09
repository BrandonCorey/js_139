# JS topics #
## Creation Phase / Executuion Phase ##
The two phases the JS engine goes through to run our code
- Creation --> Stores references to all variables and reecords their scope
  - Also initalizes `var` declarations to `undefined` and `function` declarations to their function objects
- Exeuction --> Interpreter executes our code, line by line

## Scope ##
- Declarative scope - Function vs Block
- Visibility scope - global vs local
- Lexical scope - Inner vs outer (relative)

## `var` statement ##
`var` keyword can be used to declare variables. Variables declared with `var` are:
- function scoped
- initalized to `undefined` during creation phase
  - The combination of function scope and initalization to undefined gives the appearance of var declarations being "hoisted" to top of function
- Allowed to be redeclared (however these subsequent declarations are discarded during creation phase)
 ### Other Notes ###
- In non-Node JS runtimes, declaring a variable in global scope with `var` adds property to global object
  - This is doesn't happen in Node because the global scope is actually the body of a wrapper function
- Because `var` declarations are initialized to `undefined` during the creation phase, in our code, we can access them anywhere within the function
  - In other words, it is impossible to get a `ReferenceError: Cannot access <variable name> before initizialization`
```javascript
function hello() {
  console.log(str); // undefined (Notoice how initalization to undefined during creation allows this to be printed without throwing an error
  if (true) {
    var str = 'hello'
    console.log(str); // 'hello';
  }
  console.log(str); // 'hello'; // Notice how function scoping/hoisting allows us to access this outside of the conditional block
}
hello();
```
 ## Hoisting ##
A mental model to describe how JS finds all declarations and stores references to their respective locations in memory
- All `var` declarations and `function` declarations are "hoisted" to top of function (have function scope)
  - Unlike other declarations, these declarations are also initalized during creation phase, allowing us to access them to be accessed anywhere in the function in which they were declared
  - `var` declarations are initalized to `undefined`
  - `function` declarations are initialized to their definition
  - If a variable/function of the same name is declared using `var` of `function` multiple times, all subseqeuent declarations are discared and are treated as reassignments during execution
  - If a `var` and `function` declaration share the same name, the function gets priority (is "hoisted), and the `declaration` is discareded, and its initilzation value will act as a reassignment during execution
- Block scoped declarations like `let`, `const` and `class` also have their declarations hoisted (references to variables recorded in creation phase)
  - Function expressions and class declaration/expressions have their **names** hoisted (references to variables), but not their definitions
```javascript
// Source Code
hello();

function hello() {
  let world = worldFunc(); 
  console.log(hello + ' ' + world); // 'undefined world'
  
  if (true) {
    var hello = 'hello';
  }
  console.log(hello + ' ' + world); // 'hello world'
}

function worldFunc() {
  return 'world';
}
```
```javascript
// "Hoisted" version
function worldFunc() {
  return 'world';
}

function hello() {
  var hello;
  let world = worldFunc(); 
  console.log(hello + ' ' + world); // 'undefined world'
  
  if (true) {
    hello = 'hello';
  }
  console.log(hello + ' ' + world); // 'hello world'
}

hello();
```
```javascript
// Same name declarations
var hello = 'hello';
var hello = () => return 1 + 1;
function hello() {
  console.log('hello');
}


hello(); // 2
```
```javascript
// Hoisted version
function hello() {
  console.log('hello');
}
hello = 'hello';
hello = () => 1 + 1;
hello() // 2;
```
### Temporal Deadzone ###
All variables declared with `let`, `const` or `class` are said to be in the temporal deadzone (TDZ) _after_ creation and _before_ initialization
- At this point, the engine is aware of the variables and their scope, but they are in a state of `not defined`, and cannot be accessed

```javascript
console.log(x); // ReferenceError: Cannot access 'x' before initialization (currently in the TDZ)
let x = 5;
```
```javascript
class Person {}
console.log(Person); ReferenceError: Cannot access 'Person' before initialization (currently in the TDZ)
```
```javascript
console.log(y); // ReferenceError: y is not defined
```

## Strict Mode ##
Makes 3 major changes to JavaScript semantics
- Eliminindates certain silent errors
- Eliminates certain code that can cause optimization issues
- Prevents using names that may conflict with future versions of JS

### Benefits ###
- Mitigates bugs
- Makes debugging easier
- Makes code faster
- Helps prevent conflicts with future changes to JS

### More Specific ###
- Cannot create global variables implicitly (by forgetting a variable declaration)
- Functions use `undefined` as their implicit context, not the `global` object
- Forgetting to use `this` raises an error
- Leading zeros on numbers is illegal
- Automatically gets enabled inside of ES6 classes
- Can be enabled globally or within the scope of a function (has to be at the top of file/function)

### When to use ###
- When creating new programs or functions from scratch
  - This includes adding new functions to an exisitng codebase
### When not to use ###
- Do not use strict mode when modifying code that was not written with strict mode enabled as it may break something unexpectedly

```javascript
"use strict"
// Implicit global variables
x = 5; // ReferenceError: x is not defined
```
```javascript
// Forgetting to use 'this' as well as implicit execution context
let person = {
  name: 'brandon',
  greet() {
    console.log(name); // ReferenceError: name is not defined
  },
  
  greetAgain() {
    console.log(this.name);
  }
}

person.greet();

let greet = person.greetAgain();
greet(); // TypeError: greet is not a function
```
```javascript
// Use of octal literals
let x = 01234; // SyntaxError: Octal literals are not allowed in strict mode
```
## Closure ##
The combination of a function and the lexical environment within which that function was defined
- Closures can be thought of as an object attached to a function that contains references to all **variables** in scope at the point of definition
- The closure is one place the JS engine looks for variables (engine looks there after local scope), and can use the references to these variables to find values
- Because closures are attached to the function object, variables referenced within them are accessible even they are out of scope for the invocation
  - Note: A new closure and variables referenced in them are recreated every time the function they live in is invoked (if they were declared in a function)
```javascript
// New counters and closures created on each invocation of makeCountByFive
const makeCountByFive = () => {
  let count = 0;
  return () => {
    return count += 5;
  }
}

const countByFive = makeCountByFive();
countByFive(); // 5
countByFive(); // 10
countByFive(); // 15

const countByFiveAgain = makeCountByFive();
countByFiveAgain(); // 5
```
```javascript
// One counter and one closure created due to a single invocation of makeCounter
const makeCountByFive = () => {
  let count = 0;
  const countByFive = () => {
    return count += 5;
  };

  const countByFiveAgain = () => {
    return count += 5;
  }

  return [countByFive, countByFiveAgain]
}

const [countByFive, countByFiveAgain] = makeCountByFive();
countByFive(); // 5
countByFive(); // 10
countByFive(); // 15

countByFiveAgain(); // 20
```
### Partial Function application ###
The use of one function to apply some arguments to a function invocation using closure to reduce the number of arguments required to invoke it
- This allows us to pass functions as arguments to other functions that may want to call our function with less arguments than our function takes
- We do this by returning a function that invokes the partially applied function
- **It is not PFA** if the total number of arguments is not reduced in the returned function
- **It is not PFA** if we hard-bind the first argument of the returned function instead of using closure

Closure in below example:
- Returned function closes over `callback` parameter in outer function, giving us access to the callback in the returned function even when `callback` is out of scope
- Returned function also closes over `sortCopy` function, allowing us to return its invocation from our inner function even when it is out of scope
```javascript
// Example of PFA
const makeSorter = (callback) => {
  const sortCopy = (func, array) => {
    return array.slice().sort(func);
  }
  return (array) => {
    return sortCopy(callback, array)
  }
}

const sortAsc = makeSorter((a, b) => a - b);
sortAsc([6, 3, 7, 2]); // [ 2, 3, 6, 7 ]
```
### Private Data ###
Private Data is data that is inaccessible from outside the scope where the data is defined
- Closures allow us to store private data that cannot be accessed  outside of the function that created the closure
- Private data is desirable as it forces users to use the methods we provide to interract with that data instead of interracting with it directly
  - This helps with data integrity
  - Abstracts away implementation details from other developers

Closure in below example
- Our returned object'S method `setID` closes over `generateID`
- This allows us to access the `generateID` function without exposing it to the public interface of our account instances
- The only way to interface with `generateID` is with the `setID` instance method
```javascript
const createAccount = (email, password) => {
  const generateID = () => {
    const LENGTH = 8;
    const CHARS = 'abcdef0123456789';
    let id = '';
    for (let count = 0; count < LENGTH; count++) {
      let idx = Math.floor(Math.random() * CHARS.length);
      id += CHARS[idx];
    }
    return id;
  }
  
  return {
    email,
    password,
    setID() {
      this.userID = generateID();
    }
  }
}
```
## IFFEs ##
An immediately invoked function expression is a function that can be defined and invoked simulataneously using specific syntax
- Can wrap functions in parenthesis and pass an argument immediately following the definition
- Can use IFFEs to create private scope and private data
  - Can be used to generate an object with fewer lines of code

### Basic syntax ###
```javascript
// Using IIFE to create a function that immediately invokes and returns another function
const print = (() => {
  return (str) => '=> ' + str;
})();

print('hello world'); // '=> hello world'
```
### Private Scope ###
```javascript
// Using IIFE to create private scope for `arr` and `total`
(() => {
  let arr = [1, 2, 3, 4, 5];
  let total = arr.reduce((sum, val)) => sum + val);
  return total;
})();
```
```javascript
// Can also use a block to create private scope
{
  let arr = [1, 2, 3, 4, 5];
  let total = arr.reduce((sum, val)) => sum + val);
  return total;
}
```
### Creating private data ###
```javascript
// Creating private data with IIFE. `roll` closer over `generateNum`
const Dice = (() => {
  const generateNum = () => {
    return Math.floor(Math.random() * 5) + 1
  }
  
  return class {
    static roll() {
      return generateNum();
    }
    constructor() {}
  }
})();
```
## Shorthand Notation ##
### Concise property initializers and concise methods ###
```javascript
// Standard JS
const createPerson = (name, age) => {
  return {
    name: name,
    age: age,
    greet: function() {
      console.log(`Hi, I am ${name} and I am ${age} years old`);
    } 
  }
}
```
```javascript
// Shorthand
const createPerson = (name, age) => {
  return {
    name,
    age,
    greet() {
      console.log(`Hi, I am ${name} and I am ${age} years old`);
    } 
  }
}
```
### Object destructuring ###
- Unlike array destructuring, objects are destructured using property names, not the order in which they appear
- Can change the name using `{ prop: newProp }` syntax
- Can also destructure within function definitions using parameters
```javascript
const person = {
  name: 'Brandon Corey',
  age: 24,
  greet() {
    console.log(`Hi, I am ${name} and I am ${age} years old`);
  }
}
```
```javascript
// Standard JS
let age = person.age;
let name = person.name;
let greet = person.greet;
```
```javascript
// Destructuring
let { age, greet, name } = person;
```
```javascript
// Destructing with parameters
const destruct = ({ name, age, greet }) => {
  // more code here....
}
```
### Array destructuring ###
- Arrays must be destructured based on the order of their elements
- Can skip elements using commas
```javascript
let arr = [1, 2, 3, 4, 5];
```
```javascript
// Standard JS
let one = arr[0];
let two = arr[1];
let five = arr[4];
```
```javascript
// Array destructuring
let [ one, two, , , five ] = arr;
```
```javascript
// Destructuring using parameters
const destruct = ([ one, two, , , five ]) = {
  // more code here...
}
```

### Spread Syntax ###
Uses `...` to spread elements of an iterable to seperate items (can visually think of them as comma seperated elements)
- Can be used on strings, arrays, or objects
```javascript
let arr = [1, 2, 3, 4, 5];
let obj = {name: 'brandon', age: 24};
const sum = (array) => array.reduce(total, val) => total + val);
```
```javascript
// Standard JS
let arrCopy = arr.slice(); // Copy of arr
let arrAdditional = arr.slice().concat(([6, 7]); // [ 1, 2, 3, 4, 5, 6, 7 ]
let largest = Math.max.apply(null, arr); // 5
sum.apply(null, arr); // 15
let objCopy = Object.assign({}, obj);
```
```javascript
// Uses of spread syntax
let arrCopy = [...arr] // Copy of arr
let arrAdditional = [...arr, 6, 7]; // [ 1, 2, 3, 4, 5, 6, 7 ]
let largest = Math.max(...arr); // 5
sum(...arr); // 15
let objCopy = {...obj};
```
### Rest syntax ###
Collects multiple items into an array or object (opposite of spread syntax)
```javascript
let arr = [1, 2, 3, 4, 5];
let obj = {name: 'brandon', age: 24, occupation: 'analyst'};
```
```javascript
// Using with declarations
let [ one, two, three, ...otherNums ] = arr;  // [ 1 ] [ 2 ] [ 3, 4, 5 ]
let { name, ...rest } = obj; // { name: 'brandon' } { age: 24, occupation: 'analyst' }
```
```javascript
// Using with functions to accept variable number of arguments
const sum = (...args) => {
  return args.reudce((total, val) => total + val);
}
```
```javascript
// Old way accpeting variable number of arguments using Arguments object
const sum = () => {
  let args = Array.from(arguments);
  return args.reudce((total, val) => total + val);
}
```

## Modules ##
A module is just a seperate file used to store code

### Benefits ###
- Each module can work work on a spereate problem (seperation of concerns)
- Each module code is less entangled with the rest of the program and can be used more easily
  - It is easier to export a module than to cut/copy and paste code from a file
- It is easier to work with private data since each module will have its own scope
  - Only things explicitly exported from the module will be accessible to other modules

## CommonJS modules ##
Native support for modules offered by **Node**
- Uses `require` function to import a module
  - NPM modules only need to pass the name of the module to `require`
  - user-created modules must pass the relative file path or absolute file path to module
    - The name exported becomes the name available to be imported to another program
- Browsers do not support commonJS modules are they are synchronous and too slow for the browser

### CommonJS syntax ###
- `require` - Used to import a module by passing argument to it
- `module` - An object that respresents the current module
- `module.exports` - A property that points to things to be exported (is an object by default, but can be reassigned to anything)
- `module.__dirname` - Absolute pathname of the directory that contains the module
- `module.__filename` - Absolute pathname of the module file name

```javascript
// Exporting one function from a module
const sum = (arr) arr.reduce((total, val) => total + val);

module.exports = sum;
```
```javascript
// Exporting two functions and a variable
const pi = 3.14159265;
const sum = (arr) => arr.reduce((total, val) => total + val);
const multiply = (arr) => arr.reduce((total, val) => total * val);

module.exports = {sum, multiply, pi}
```
## Exceptions ##
Errors messages that are thrown when JavaScript cannot recover from an error
- Program will be automatically terminated
- "throw" and "raise" can be used interchangeably
- "error" message however, does not always mean exception
  - Could be something as simple as a `console.log`

### Throwing excpetions ###
- Most we've seen so far are raised internally (i.e ReferenceError, SyntaxError, TypeError);
- They can also be defined manually
- An exception can have any value, including primitive, or object (commonly, `Error` object or instance of `Error`)
  - The internal errors are instances of the `Error` object
  - All `Error` objects have `message` and `name` instance properties that can be accessed
  - Can pass optional `message` argument to `Error` constructing when creating new error istance
  - Can throw an error using `new` keyword or without it (still creates new error object)
  - Can throw error with any following expression

```javascript
class LoadError extends Error {};
throw new LoadError('Cannot load file');

throw 'banana' // Raises exception with value banana
throw Error('this is an error'); Creates instance of error with a message property and throws it
throw new Error('this is an error'); Creates instance of error with a message property and throws it
```
### Catching exceptions ###
We can use the `try` and `catch` statements to resolve excpetions that are raised
- This is a good use case of explicitly created a new error class like above
- You generally want to know exactly the type of error that may be raised
- When an exception is thrown, it will always look for `try` blocks that contain the code that caused it before terminating
  - If it finds one, the code in the catch block will be executed
  - IF the catch throw an error, the process will repeat
```javascript
const loadFile = (filePath) => /*code here*/; 
try {
  loadFile('./testFile');
} catch (error) {
  if (error instanceof loadFile) {
    console.log('Could not load file');
  } else {
    throw error;
  }
}
```
### Exceptions should be used for exceptional behavior ###
- Do not throw exceptions for flow control type issues
  - The divide by 0 is an exmaple of when it might not be needed
  - Any type of input validation that can be controlled probably should not raise an exception

- Throw them when there is behavior that should not be ignored or is truly anomolus
  - ex) trying to load a file that may fail
  - Only way to detect this is by trying and seeing if it works

- Only handle exceptions you believe can be recovered from successfully
  - Only handle if you're confident the handler will not throw another expceiton itself
  - handler should do as little as possible (ignore exception, return error value, log a message, throw another exception)

## Pure Functions ##
Pure functions are functions that do not have side effects, and return a useful value that is not influenced by code outside the function
- Useful value means a value that means something to the calling code 
- Technically, functions themselves don't have side effects, invocations do. However, we still refer to functions themselves as having side effects
- If an invocation has side effects when *used as intended*, we say the function has side effects
  - Used as intended:
    - passed the expected arguments
    - Invoked after requesite preparations (such as after opening a connection to a remote server)

### Side effects ###
A side effect occurs if a function:
- Reassigns a non-local variable
- Mutates an object referenced by a non-local variable
- Reads or writes to any data entities (files, databases etc.)
  - This also includes accessing system hardware (mouse, trackpad, clock, random number generator, camera, speakers)
  - This means functions like `Math.random` and `new Date` produce side effects
- Raises an exception
- Calls another function that has side effects

### Always returns same value based on same arguments ###
- This second requirement states that a functions return value is soley dependent on its arguments, and not outside factors
```javascript
const sum = (a, b) => a + b; // Pure
```
```javascript
let a = 5;
const sum = (b) => (a + b); // Impure (influenced by outside values)
```
```javascript
const doNothing = (a, b) => { // Pure (will always return same value given same arguments)
  a + b;
}
```
```javascript
const printSum = (a, b) => console.log(a + b); // Impure (invokes a method that has side effects)
```

## Asymchronous programming ##
Code can be run asynchronously in JS. This means when the engine encounters the code, it does not execute it fully until a later point in time (ms, secs, mins, etc..)

```setTimeout```
- `setTimeout` is one of the easiest ways to run code asynchronously
- It accepts two arguments: a callback function to execute, and a number of milliseconds to delay the execution
- The invocation of `setTimeout` is not delayed, only the execution of the callback passed to it

### Callback passed to `setTimeout` ###
- Only runs when JS is not doing executing any other synchronous code
- Other callbacks passed to `setTimeout` will execute in order based on ms of delay
- Code with a `0` ms delay will still not run until all synchronous code is done running
```javascript
const yodaSpeak = () => {
  setTimeout(() => {
    console.log('Greetings'); // 1
  }, 1000);

  setTimeout(() => {
    console.log('my name is'); // 3
  }, 3000)

  setTimeout(() => {
    console.log('Yoda'); // 2
  }, 2000);
}

yodaSpeak();
```

### A note on asynchronous code and for loops ###
- When using for loops, JS *treats* the counter as if a new one is re-declared on each iteration
- This forms a new closure with the callback passed to the asynchronous function, and forms a new closure on each iteration
- These closures give the asynchronous callback access to each incremental value of the for loop instead of only the final
- This is special behavior exhibited in for loops to make things easier, as in theory, it would be impossible to increment a counter if we redeclared in on each iteration
```javascript
const delayLog = () => {
  for (let sec = 1; sec <= 10; sec++) {
    setTimeout(() => console.log(sec), sec * 1000); // logs 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
  }
}
```
```javascript
const delayLog = () => {
for (var sec = 1; sec <= 10; sec++) {
    setTimeout(() => console.log(sec), sec * 1000); // logs 11, 11, 11, 11, 11, 11, 11, 11, 11, 11
  }
}
```
```javascript
const delayLog = () => {
  let sec;
  for (sec = 1; sec <= 10; sec++) {
    setTimeout(() => console.log(sec), sec * 1000); // logs 11, 11, 11, 11, 11, 11, 11, 11, 11, 11
  }
}
```
### JS does some magic behind the scenes to make it work, probably something like: ###
```javascript
const delayLog = () => {
  let sec;
for (sec = 1; sec <= 10; sec++) {
    ((sec) => (setTimeout(() => console.log(sec), delay * 1000)))(sec);
  }
}
```
## `setInterval` and `clearInterval` ##
Similar to `setTimeout`, but instead allows you to execute a callback function at certain nterals of ms delays
- Can use `clearInterval` to stop the `setInterval` execution of the callback passed to it
- `setInterval` returns an ID that can be used by `clearInterval` to cancel its execution
```javascript
setInterval(() => {
  console.log('stayin alive');
}, 1000);
```
```javascript
let stayingAlive = setInterval(() => {
  console.log('stayin alive');
}, 1000);

clearInterval(stayingAlive); // Will clear the invocations of the callback by setInterval;
```
```javascript
const countDelay = () => {
  let count = 0;
  return setInterval(() => {
    count += 1;
    console.log(count);
  }, 1000)
}

let countID = countDelay(); // 1, 2, 3, 4, 5, 6, 7, 8, 9.....

clearInterval(countID) // The above count will only execute if this line is not here
                       // I cancelled it synchronously before it could run
```
# Testing #
We write tests to prevent regression
- Regression is an event that causes previously working code to not work anymore

## Terminology ##
- test - A specific situation or context that is being attempted to test
  - Also called a **spec**
- test suite - An entire set of tests the accompanies your program or app
- assertion - A verification step that confirms that your program did what it should do
  - Typically testing return values of functions/methods
  - Also called **expectations**

## Jest ##
A testing library for javaScript
- Has a `describe` function that takes a description of your tests and a callback for multiple test arguments
- Has a `test` function that takes a descritino of the test and a callback to execute

```javascript
describe("Tests for our createBanana function", () => {
  test('Our banana is yellow', () => {
    let banana = new Banana('yellow');
    expect(banana.color).toBe('yellow');
  });
});
```

### `expect` and matcher methods ###
The expect keyword is used to create an assertion in conjunction with matcher metheods
- `expect` takes an argument of the value to be tested
  - This value is called the `actual` value
- `expect` returns an object that has a variety of **matcher methods** that compare the actual value passed to expect to an **expected** value
  - The **expected value** is the argument passed to the matcher
### Most important matchers ###
- `toBe` - Tests if the actual value is strictly equal to the expected value
- `toEqual` - Tests if the actual value is strictly equal value for primitives, but compares the values objects contain instead of their references
  - This allows us to test if an object has the values we expect
```javascript
describe("The banana class" () => {
  let banana;
  test('Our banana is yellow', () => {
    banana = new Banana('yellow');
    expect(banana.color).toBe('yellow');
  });
  test('Two newly created objects are equal', () => {
    let banana2 = new Banana('yellow');
    expect(banana2).toEqual(banana);
  });
});
```
### Other matchers ###
- `toBeUndefined`, `toEqual`, `toThrow`, `toBeNull`, `toBeTruthy`, `toContain`
- Can add modifier `not` to any of the above matchers to negate them e.g `expect(actualValue).not.toBeUndefined();`
- N**ote:** Any actual value expected `toThrow` needs to be passed returned from a callback so it doesn't throw the error before `expect` can evaluate it
```javascript
describe("The banana class" () => {
  test('banana color is assigned, () => {
    let banana = new Banana('yellow');
    expect(banana.color).not.toBeUndefined();
  });
```
```javascript
describe("The banana class" () => {
  test('banana color is assigned, () => {
    expect(() => new Banana('yellow', 'green')).toThrow();
  });
```
## SEAT approach ##
- **S**et up the necessary objects
- **E**xecute the code against the items we are testing
- **A**ssert the results of execution
- **T**ear down and clean up any lingering artifacts

### Setup ###
We can use the jest function `beforeEach` to set up necessary objects/values before our tests
- `beforeEach` takes a single callback argument
- `beforeEach` is invoked outside the body of our tests
  - This means any variables used in `beforeEach`s callback must be declared outside `beforeEach` so that they are in scope for the other tests
- The callback passed to `beforeEach` is executed before every individual test (which is good as it will prevent us from mutating objects)

```javascript
describe('The banana class', () => {
  beforeEach(() => {
    let banana = new Banana('yellow');
    let otherBanana = new Banana('green');
  });
  test('Our banana is yellow', () => {
    expect(banana.color).toBe('yellow');
  });
  test('Bananas have different colors', () => {
    expect(banana).not.toEqual(otherBanana);
  });
});
```

## Execution ##
This is the code we run to produce the actual value we would like to test
- This can also be included in the setup phase as well
- I am not including a code example because there are many above

## Assertion ##
This is the actual value passed to the `expect` function being compared to the `expected` value passed to the method method
- All assertions in `jest` begin using the `expect` keyword
- Once again, there are many example above of this

## Tear Down ##
There is a function called `afterEach` that can be used for teardown
- Similar to `beforeEach`, after each is used to execute after every single test
- We don't really go over this in the curriculum too much

```javascript
describe('The banana class', () => {
  beforeEach(() => {
    let banana = new Banana('yellow');
    let otherBanana = new Banana('green');
  });
  
  afterEach(() => console.log('test complete');
  
  test('Our banana is yellow', () => {
    expect(banana.color).toBe('yellow');
  });
  test('Bananas have different colors', () => {
    expect(banana).not.toEqual(otherBanana);
  });
});
```

## Code coverage ##
Testing libraries usually determine coverage one of two ways
- number of functions called by our tests / total functions in our program --> as a percentage
- number of lines executed by our tests / total lines in our program --> as a percentage

### Important things to remember ###
- This does not tell us whether the code is working correctly or not
  - Only tells us how if they it is working with the our test cases

jest has a built in coverage tool --> `jest --coverage <test program name>`

## Packaging Code ##

## Node Pojects ##
Should always create a new local directory, new github repository, and new Git repository for projects
- Make sure that all directories and parent directory names **do not have spaces**
- Make sure parent directories do not have `node_modules` directory (except maybe at the global level)

### Necessary files/directories for now ###
- `test` directory for storing all of our test files
- `lib` directory for storing our source code
- `dist` directory for transpiled code
- `index.js` main file referenced in our `package.json` that imports the relevant modules (in this case, our transpiled code modules)
- `.gitignore` file containing the text `node_modules` (this file is hidden)
- `README.md` containg whatever text we want to describe project

Example:
```
todolist_project
├── README.md
├── index.js
├── dist
│   ├── todo.js
│   └── todolist.js
├── lib
│   ├── todo.js
│   └── todolist.js
└── test
    └── todolist.test.js
```

### Node Packages ###
Packages of code that can be downloaded, installed, and used in your code or system
- **packages are made up of modules**
- Some can be imported into programs
- Some can be used from the command line
- Some can do both

The `npm` command is used to manage packages

### Examples of packages I've alreasy used ### 
- `eslint` (command-line)
- `jest` (command line)
- `readline-sync` (imported into program)

### npm vs npx ###
- Things like `readline` are programming interfaces, it is a module that we require and allows us to use its methods/functions
- Things like `jest` and `eslnt` are command line executables
  - `npx` is used to run local executables, global ones do not require the `npx` prefix
- NOTE: When using `require` for node modules, we do not need to specify the relative path. Node will always look through your `node_modules` directory

### Local vs Global Packages ###
Local packages are installed to your project directory
- They will be installed in your `node_modules` directory. If you do not have one, one will be created
  - This is why no parent directories should have a node modules folder, because node will use that instead of creating a new one

Packages should almost always be installed locally so you/npm can install the correct version of the package for your project
- A version installed globally may be compatible with one project, but not another

Some packages don't usually require different versions for different projects and be installed globally
- Examples of this are `jest` and `heroku`
- These can be installed using the global flag `npm install heroku -g` or `npm install jest -g`

```javascript
const _ = require('lodash'); // lodash library is an object with methods
const chunk = require('lodash/chunk'); // Can import functions we want directly (faster loading, less memory used)

chunk = require('lodash').chunk; // Note: Not all libraries independently export their functions like lodash. We can use this syntax for those
                                 // This still imports all the methods, but they get garbage collected quickly

console.log(_.chunk([1, 2, 3, 4, 5, 6, 7, 8], 2));
// logs [[1, 2], [3, 4], [5, 6], [7, 8]]
console.log(chunk([1, 2, 3, 4, 5, 6, 7, 8], 2));
```

## `package.json` and `package-lock.json` ##
The `package.json` file contains important information about your Node project, including all of your projects dependncies
- A `package.json` can be created using `npm init`

### `package.json` ###
While Node will add a `package.json` automatically when you install a package, it can also be created manually
- You can manually add a `dependencies` key to the JSON with the relevent information of the module you want to install
- After you add your `dependencies`, you can use `npm install` to find the version you specified and install it into `node_modules`
- Can also do it automatically using `npm install <packagename> --save`
  - `--save` flag tells npm to save the package to the `dependencies` list in `package.json`. Can also use `-S` flag

Some packages may only be relevant in the context of a development enviroment, and not suitable for production (e.g `eslint`)
- There is a section within a `package.json` for these dependecies as well called `devDependencies`
- Can manually add this to a `package.json` or use `npm install <packagename> --save-dev`

### `package-lock.json` ###
Installing modules will create a `package-lock.json`, which will contain specific info about which major, minor, and patch version of a depency your project is using
- IMPORTANT: If you make ANY changes to `package.json` dependencies, you MUST update `package-lock.json` using `npm intall`
- The `package-lock.json` will also show all the dependencies that your project has
  - NOTE: Verson is considered the major number that we specify in our `package.json`, but Node will determine the minor version and patch for us
  - Ex) Version 4.17.1 --> Major version: 4, Minor version: 17, Patch version: 1
- The `package-lock.json` file is important as the next time `npm install` is ran, it will install the exact depencies needed, down to patch number
- This is incredibly helpful if a team of developers are working on something as it makes it possible to be using the exact same versions for each dependency

```json
// package.json example
{
  "name": "todolist",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "directories": {
    "lib": "lib",
    "test": "test"
  },
  "dependencies": {
    "express": "4",
    "http-errors": "1",
    "lodash": "^4.17.15",
    "morgan": "1.9"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "eslint": "^8.32.0"
  }
}
```

### Global vs Local executable packages ###
- To run an executable globally, call it using its name i.e `eslint <filename.js>`
- To run an executable locally, call it using npx + its name i.e `npx esilnt <filename.js>`
  - **Note:** if you don't have a local version of a package installed, npx will check globally, and if not found, downloads a temporary version to use

### Deleting a dependency ###
- Use `npm uninstall <packagename>` (i.e `npm uninstall lodash`)
- To uninstall and remove from `package.json` dependencies, use `npm uninstall <packagename> --save`
- To uninstall from dev dependencies, use `npm uninstall <packagename> --save-dev`
- Can also use `npm prune` to remove any packages from `node_modules` that are not listed in the `package.json`

## Transpilation ##
Transpilation is the process of converting source code written in one language into another language with a similar level of abstraction to the original code.
- In JS, this often means taking code written in a superset of JS and rewriting it in JS
  - This typically means converting ES6+ code to ES5
  - We do this so new JS features work in all runtime enviroment

### Babel ###
This is by far the best and most poppular javaScript transpiler

To install:
- `npm install --save-dev @babel/core @babel/cli`
  - Recommended to install babel locally despite being a command-line utility
  - `@babel/core` is the core library that provides us the ability to transpile
  - `@babel/cli` is the cli provided to us to interface with the library

We need to install the `env` preset (a type of plugin) to transpile our code to ES5:
`npm install --save-dev @babel/preset-env`
- The `env` preset is used to transpile JS code to ES5

Finally, to **transpile our ES6+ to ES5**:
`npx babel lib --out-dir dist --preset=@babel/preset-env`
- This will transpile all of our js files in the `lib` directory of our current directory
- It will output files of the same name to a `dist` directory

## npm scripts ##
We can add scripts to our `package.json` within the "scripts" object

### Example of adding Babel script ###
The syntax to run a script is `npm run <scriptname>`

Lets make using babel easier:

We can add `npx babel lib --out-dir dist --presets=@babel/preset-env` to the script called `babel`
- Note, we can exclude `npx` within scripts, as they will automatically run locally as long as your package is installed locally
  - We must already have ran both `npm install --save-dev @babel/core @babel/cli` and `npm install --save-dev @babel/preset-env` for this script to work

```json
{
  "name": "todolist_project",
  "version": "1.0.0",
  "description": "This repo is to learn how to format node projects",
  "main": "index.js",
  "directories": {
    "lib": "lib",
    "test": "test"
  },
  "dependencies": {
    "express": "4",
    "http-errors": "1",
    "morgan": "1.9"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "babel": "babel lib --out-dir dist --presets=@babel/preset-env"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/BrandonCorey/todolist_project.git"
  },
  "author": "",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/BrandonCorey/todolist_project/issues"
  },
  "homepage": "https://github.com/BrandonCorey/todolist_project#readme",
  "devDependencies": {
    "@babel/cli": "^7.20.7",
    "@babel/core": "^7.20.12",
    "@babel/preset-env": "^7.20.2",
    "eslint": "^8.32.0"
  }
}

```

## How to publish to npm ##
Make sure `package.json` contains correct information
- Remove any dependencies that you are not using from the file
- After they are removed, use `npm prune` to remove them from your `node_modules`
- After that, use `npm install` to update your `package-lock.json`

**The three main things that need to be filled out in `package.json`**
- Change `name` of package if needed
- Change `version` of package if needed
- Make sure `main` is pointing to your main file
  - Make sure your is referncing the transpiled versions of your source code (if you have them)

When ready:
```npm publish --access public```

### If you are not logged in... ###
- Use `npm login`. When prompted, enter login info
- If you don't have an account, use `npm adduser` to create one. Enter info when prompted
