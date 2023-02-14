### Non-Private-data ###
In the code snippet below, we use an `Account` class to create objects with the instance proerties `email` and `password`. We also provide instance methods that allow us to reassign these properties to different values, given that we input the correct passowrd for the account. In this situation, none of our properties are private, and that is an issue. While it makes sense to expose `resetPassword` and `updateEmail` and `printInfo` as part of our public interface, given the nature of these functions, it makes little sense to allow for direct access to the instance properties `email` and `password`.

As can be seen below, we can completely bypass the requirement of a correct passowrd to update the values by accessing the properites directly on the instance. This defeats the purpose providing an interface to do so, as we are not limiting the user to the requirements we set in place. It also allows us to view `email` and `password` as the properties are stored directly on the instance, which defeats the purpose of the password requirement of `printInfo`.

```javascript
// // Non-private properties
// // Non-private properties
class Account {
  constructor(email, password) {
    this.email = email;
    this.password = password;
  }

  resetPassword(currentPass, newPass) {
    if (this.password === currentPass) this.password = newPass;
    else console.log('Could not update password! (Incorrect password)');
  }

  updateEmail(currentPass, newEmail) {
    if (this.password === currentPass) this.email = newEmail;
    else console.log('Could not udpate email! (Incorrect password)');
  }

  printInfo(password) {
    if (this.password === password) {
      console.log(`email: ${this.email}\npassword: ${this.password}`);
    } else console.log('Could not display account info! (Incorrect password)');
  }
}

let account = new Account('bcorey3660@gmail.com', 'testPassword');

account.printInfo('wrongpass'); // Could not display account info! (Incorrect password)
account.resetPassword('wrongPass', 'newPass'); // Could not update password! (Incorrect password)
account.updateEmail('wrongPass', 'newEmail'); // Could not udpate email! (Incorrect password)

account.password = 'fakePassword'; // Bypasses resetPassword
account.email = 'fakeEmail@fake.com'; // Bypasses updateEmail
console.log(account); // { email: 'fakeEmail@fake.com', password: 'newPassword' } // Bypases printInfo
```
### Private Data ###
One solution here is to take advantage of closure in conjunction with an IIFE. We can use the IIFE to return a class who's methods "close over" variables within our outer function, allowing us to access the variables through the closure instead of as properties on the instance. This allows us to store private data that can be accessed through the interface we provide with our methods, forcing users of the interface to abide by the requirements we set in place for the interraction, instead of letting them bypass it entirely.
```javascript
const Account = (() => {
  let userPassword;
  let userEmail;

  return class {
    constructor(email, password) {
      userPassword = password;
      userEmail = email;
    }
  
    resetPassword(currentPass, newPass) {
      if (userPassword === currentPass) userPassword = newPass;
      else console.log('Could not update password! (Incorrect password)');
    }

    updateEmail(currentPass, newEmail) {
      if (userPassword === currentPass) userEmail = newEmail;
      else console.log('Could not udpate email! (Incorrect password)');
    }

    printInfo(password) {
      if (userPassword === password) {
        console.log(`email: ${userEmail}\npassword: ${userPassword}`);
      }
      else console.log('Could not display account info! (Incorrect password)')
    }
  }
})();

let account = new Account('bcorey3660@gmail.com', 'testPassword');
console.log(account) // {}
account.printInfo('wrongPass') // Could not display account info! (Incorrect password)
account.resetPassword('testPassword', 'updatedPassword');
account.updateEmail('updatedPassword', 'newEmail@new.com');
account.printInfo('updatedPassword'); // email: newEmail@new.com password: updatedPassword
```