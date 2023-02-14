### Non-Private-data ###
In the code snippet below, we use an `Account` class to create objects with the instance proerties `email` and `password`. We also provide instance methods that allow us to reassign these properties to different values, given that we input the correct passowrd for the account. In this situation, none of our properties are private, and that is an issue. While it makes sense to expose `resetPassword` and `updateEmail` as part of our public interface, given the nature of these functions, it makes little sense to allow for direct access to the instance properties `email` and `password`.

As can be seen below, we can completely bypass the requirement of a correct passowrd to update the values by accessing the properites directly on the instance. This defeats the purpose providing an interface to do so, as we are not limiting the user to the requirements we set in place.

```javascript
class Account {
  constructor(email, password) {
    this.email = email;
    this.password = password;
  }

  resetPassword(currentPass, newPass) {
    if (this.password === currentPass) this.password = newPass;
    else console.log('Could not update password! (Incorrect passowrd)');
  }

  updateEmail(currentPass, newEmail) {
    if (this.password === currentPass) this.email = newEmail;
    else console.log('Could not udpate email! (Incorrect passowrd)');
  }
}

let account = new Account('bcorey3660@gmail.com', 'testPassword');
console.log(account); // { email: 'bcorey3660@gmail.com', password: 'testPassword' }

account.resetPassword('wrongPass', 'newPass'); // Could not update password! (Incorrect passowrd)
account.updateEmail('wrongPass', 'newEmail'); // Could not udpate email! (Incorrect passowrd)

account.password = 'fakePassword';
account.email = 'fakeEmail@fake.com';

console.log(account); // { email: 'fakeEmail@fake.com', password: 'newPassword' }
```
