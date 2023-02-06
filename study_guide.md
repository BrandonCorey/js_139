## Creation Phase / Executuion Phase ##
The two phases the JS engine goes through to run our code
- Creation --> Stores references to all variables and notes their scope
  - Also initalizes `var` declarations to `undefined` and `function` declarations to their function objects
- Exeuction --> Interpreter executes our code, line by line

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
- In Node JS, since the entire program is wrapped in a function, the these declarations are still hoisted, even in the global scope.
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
