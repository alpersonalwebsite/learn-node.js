# BCrypt
A library to hash our passwords.

---

* [Salt, hash and compare](#salt--hash-and-compare)

---

Install bcrypt: `npm install bcrypt`

<!-- 
TODO: definition of
hash
salt
-->

## Salt, hash and compare

```js
const bcrypt = require('bcrypt');

async function generateSalt(saltRounds) {
  return await bcrypt.genSalt(saltRounds);
}


async function generateHash() {
  const salt = await generateSalt(10);
  console.log(salt);
  // $2b$10$0Cduq8SCxR2dp7eCXtAo8e

  const hash = await bcrypt.hash('mypassword', salt);
  console.log(hash);
  // $2b$10$0Cduq8SCxR2dp7eCXtAo8e$2b$10$0Cduq8SCxR2dp7eCXtAo8eOeZJ5dSZCUpTJE3vS5cCIFYzCJWiQBC

  return hash;
}

async function compareHashes(password) {
  const isValidPassword = await bcrypt.compare(password, await generateHash());
  console.log(isValidPassword);

  return isValidPassword;
}

compareHashes('mypassword'); // true
compareHashes('mypasswor'); // false
```

Sample output:

```
$2b$10$0Cduq8SCxR2dp7eCXtAo8e
$2b$10$0Cduq8SCxR2dp7eCXtAo8eOeZJ5dSZCUpTJE3vS5cCIFYzCJWiQBC
```