# Pivacy and Data integrity in JS #
Private data in JavaScript is important as it enables a way to restrict developers into using a provided interface to access data. This allows us to control how the data is interracted with, which helps with data integrity. When a developer _must_ use a certain set of functions or methods to access data, there is less worry about unintended behavior that could result in accidental or malicious manipulation of the data being accessed. Luckily, there are a few techniques that can be leveraged to create private data in JavaScript, primarily with the use of _closure_, as well as _modules_.

## Non-Private data ##
In the code snippet below, we use a factory named `createAccount` to create objects with the properties `email` and `password`. We also provide methods that allow us to reassign the properties to different values, and a method to display their current values (if given the correct password). In this situation however, none of the properties are private, and this can an issue. The properties are not considered private because all of them are accessible by using simple dot or bracket notation. Private data would not be directly accessible outside of the object.

While it makes sense to expose `resetPassword`, `updateEmail` and `printInfo` as part of our public interface given the nature of these functions, it makes less sense to allow direct access of the properties `email` and `password`. As can be seen below, we can completely bypass the requirement of a correct password to update the values by accessing the properites directly on the object. This defeats the purpose providing an interface to do so, as we are not limiting the user to the requirements we set in place. This is detrimental to the integrity of our data. It also allows us to view `email` and `password` with a simple `console.log` as the properties are stored directly on the object, which defeats the purpose of the password requirement for `printInfo`.

```javascript
// Non-private data
function createAccount(email, password) {
  return {
    email,
    password,

    resetPassword(currentPass, newPass) {
      if (this.password === currentPass) this.password = newPass;
      else console.log('Could not update password! (Incorrect password)');
    },

    updateEmail(currentPass, newEmail) {
      if (this.password === currentPass) this.email = newEmail;
      else console.log('Could not udpate email! (Incorrect password)');
    },

    printInfo(password) {
      if (this.password === password) {
        console.log(`email: ${this.email}\npassword: ${this.password}`);
      } else console.log('Could not display account info! (Incorrect password)');
    }
  }
}

let account = new createAccount('bcorey3660@gmail.com', 'testPassword');

account.printInfo('wrongpass'); // Could not display account info! (Incorrect password)
account.resetPassword('wrongPass', 'newPass'); // Could not update password! (Incorrect password)
account.updateEmail('wrongPass', 'newEmail'); // Could not udpate email! (Incorrect password)

account.password = 'fakePassword'; // Bypasses resetPassword
account.email = 'fakeEmail@fake.com'; // Bypasses updateEmail
console.log(account); // { email: 'fakeEmail@fake.com', password: 'newPassword' ... [Functions]} // Bypases printInfo
```
## Private Data ##
One solution is to take advantage of closure in conjunction with the object factory. We can return an object who's methods "close over" variables (in this case, parameters) within the outer factory function, allowing us to access the variables through the closure instead of as properties on the object. This allows us to store private data that can only be accessed through the closure formed between our methods and the data, forcing users of the interface to abide by the requirements we set in place for each interraction. Our methods will form a closure that contains references to the `email` and `password` parameters, allowing us access to this private data strictly through these methods.

In the example below, none of the private data can be modified _or_ viewed unless a correct password is provided to the method, which enforces a level of data integrity that is superior to our previous example.
```javascript
// Private email and password using closure
function createAccount(email, password) {
  return {
    resetPassword(currentPass, newPass) {
      if (password === currentPass) password = newPass;
      else {
        console.log('Could not update password! (Incorrect password)');
      }
    },

    updateEmail(currentPass, newEmail) {
      if (password === currentPass) email = newEmail;
      else {
        console.log('Could not update email! (Incorrect password)');
      }
    },

    printInfo(passInput) {
      if (password === passInput) {
        console.log(`email: ${email}\npassword: ${password}`);
      } else {
        console.log('Could not display account info! (Incorrect password)');
      }
    }
  };
}

let account = createAccount('bcorey3660@gmail.com', 'testPassword');
console.log(account) // { ...[Functions] }
account.printInfo('wrongPass') // Could not display account info! (Incorrect password)
account.resetPassword('testPassword', 'updatedPassword');
account.updateEmail('updatedPassword', 'newEmail@new.com');
account.printInfo('updatedPassword'); // email: newEmail@new.com password: updatedPassword
account.password = 'blah';
account.printInfo('blah'); // Could not display account info! (Incorrect password)
```
# Privacy and Integrity in Node modules #

Below, we have a few simple modules based around a report card. I have arranged these modules in a way to only expose data that is necessary, while making sure that that the exposed data does not allow unintended maniuplation of data meant to be private.

## main.js ##
This module imports four functions from two different modules. We have an `addGrade` function that lets us add a grade to our report gard, a `removeGrade` that removes the most recent grade we added, a `bestGrade` that returns the highest grade on the report card, and a `worstGrade` function that returns the lowest grade.

The report card does not live within this file, and the only way to manipulate it is using the `addGrade` and `removeGrade` functions. This provides an interface that must be used to operate on the report card. We also have functions in `bestGrade` and `worstGrade` that must be used to provide information about the highest and lowest grades within the report card.

```javascript
// main.js
const { addGrade, removeGrade } = require('./report_card.js');
const { bestGrade, worstGrade } = require('./grade_calculations.js');

addGrade(88);
addGrade(97);
addGrade(86);

removeGrade();

console.log(bestGrade()); // 97
console.log(worstGrade()); // 88
```
## report_card.js ##
This module contains our report card, which is just a simple array. We also have our `addGrade` and `removeGrade` functions. These allow us to operate directly on our `reportCard` array. Finally, we have a `getGrades` function, which returns a shallow copy of our `reportCard` array, and allows us to expose an array of grades in a safe way. Because we are only adding numbers to the array, a shallow copy should be sufficient in protecting the integrity of our data. If we were adding objects to the array, we would need to take more rigorous precautions as the elements themselves could be referenced.

We are storing these functions and `reportCard` in the same module to maintain a level of encapsulation. We want to keep the functions that operate on the data and the data itself within a single entity (in this case, a module). We also want to make sure to keep `reportCard` private to this module, as the functions in this module contain the only operations we wish to perform directly on the array.
```javascript
// report_card.js
const reportCard = [];

const addGrade = (grade) => {
  if (typeof grade === 'number') reportCard.push(grade);
}

const removeGrade = () => {
  reportCard.pop();
}

const getGrades = () => [...reportCard];

module.exports = { addGrade, removeGrade, getGrades };
```
## grade_calculations.js ##
This module imports the function `getGrades` to give us access to a copy of the `reportCard` array. I have purposely structured these functions in a way to demonstrate why keeping `reportCard` private is important. This module exports two functions, `bestGrade` and `worstGrade` so that they can be used in the main module. Both of these functions invoke `sortDesc`, which sorts our grades in descending order. This allows the other functions to reference either the first or last element of a sorted array as needed. 

Note that `sortDesc` uses the Array prototype `sort` method, which mutates the caller. This mutation isn't an issue because we use `getGrades` to access of a copy of `reportCard`, but if we did not, this would be trouble. If called after `bestGrade` or `worstGrade`, `removeGrade` (in the main module) may have ended up removing the lowest grade instead of the most recent addition. To maintain the integrity of the data, these functions were put in a seperate module as they are not intended to operate on `reportCard` directly. 
```javascript
// grade_calculations.js
const { getGrades } = require('./report_card.js');

const sortDesc = () => {
  let grades = getGrades();
  return grades.sort((a, b) => b - a);
}

const bestGrade = () => {
  return sortDesc()[0];
}

const worstGrade = () => {
  let sorted = sortDesc();
  return sorted[sorted.length - 1];
}

module.exports = { bestGrade, worstGrade }
```
