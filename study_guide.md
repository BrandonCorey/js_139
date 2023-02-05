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
