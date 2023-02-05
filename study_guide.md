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
function hello() {
  let world = world();
  console.log(hello + ' ' + world);
  
  if (true) {
    var hello = 'hello';
  }
  console.log(hello + ' ' + word);
}

function world() {
  return 'world';
}

hello();
```
### Temporarl Deadzone ###
All variables declared with `let`, `const` or `class` are said to be in the temporal deadzone (TDZ) _after_ creation and _before_ initialization
- At this point, the engine is aware of the variables and their scope, but they are in a state of `not defined`, and cannot be accessed
