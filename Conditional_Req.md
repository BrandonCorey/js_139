### Non-Private data ###
In the code snippet below, we use an `createAccount` factory to create objects with instance properties `email` and `password`. We also provide instance methods that allow us to reassign these properties to different values, as well as a method to display their current values, given that the correct password is passed to each method. In this situation however, none of the properties are private, and this is an issue. The properties are not private because all of them are accessible by using simple dot or bracket notation. In this situation, private data would not be accessible outside of the object.

While it makes sense to expose `resetPassword`, `updateEmail` and `printInfo` as part of our public interface given the nature of these functions, it makes little sense to allow direct access to the instance properties `email` and `password`. As can be seen below, we can completely bypass the requirement of a correct password to update the values by accessing the properites directly on an object. This defeats the purpose providing an interface to do so, as we are not limiting the user to the requirements we set in place. It also allows us to view `email` and `password` with a simple `console.log` as the properties are stored directly on the object, which defeats the purpose of the password requirement of `printInfo`.

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
### Private Data ###
One solution here is to take advantage of closure in conjunction with the object factory. We can return an object who's methods "close over" variables (in this case, parameters) within our outer function, allowing us to access the variables through the closure instead of as properties on an instance. This allows us to store private data that can only be accessed through the closure formed between our methods and the data, forcing users of the interface to abide by the requirements we set in place for each interraction, instead of letting them bypass it, as was possible in our previous code.

In the below example, none of the private data can be modified _or_ viewed unless a correct password is provided to the method.
```javascript
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
```
