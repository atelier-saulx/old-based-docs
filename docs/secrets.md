# Secrets

Based offers means to communicate an authorization token between client and server using the secrets API.

Currently the only token protocol Based supports is JWT. The user sends a JSON Web Token, and the server verifies the signature using a secret stored on the server itself.

Secrets can be uploaded to Based using the CLI, and can then be used using their assigned key name.

```sh
$ npx based secret <keyname> -f <keyfile.pub>
```

You can learn more about the CLI options [here](cli.md).

Once the token is decrypted on the server side, the developer is free to implement their own means of authorization, including making a request to a different API, through the use of the provided `authorize` function.

The

## Client-side

The client can set the token to be sent in every request using the `based.auth()` method, for example:

```js
import jwt from 'jsonwebtoken'
import { privateKey } from './secrets/keys'

// ... connect and setup based client ...

const token = jwt.sign({ foo: 'bar' }, privateKey, { algorithm: 'RS256' })
client.auth(token)
const data = await client.call('myserversidefunction', {
  custompayload: 'can I use this function pls',
})
// ...
```

Now every request for calling or observing a function will contain the token (and several more pieces of information about the user), which can be used to determine if the user has access to the requested function or not.

## Server-side

On the server, once a request to execute or observe a function comes in, the `authorize` method of that function is called first (see the [functions docs](functions.md) for how to upload one). The return value of this function determines if the function can be accessed or not.

When implementing this method, the developer has access to several props that can help determine who the user is, namely `{ based, user, payload, name, type, callStack }`.
| Prop | Type | Description |
| ---- | ---- | ----------- |
|`based`|Based client instance| This is a full Based client instance, with access to the current environment.|
|`user`|Client class instance| This class has a several public elements. The most useful when dealing with secrets is the `user.token()` method, which takes as argument the name of the secret Based needs to use to verify the token, and return the token's payload. |
|`payload`|any (optional)|This is the payload sent by the user when requesting the function.|
|`name`|string|Requested function's name|
|`type`|string|Either `'observe'` or `'function'`, depending if the user executed a `call`, `get`, or `observe` request.|
|`callStack`|string array|This list contains the callStack for the function call meaning that if the function is called from within a different server-side function, that name of that method will appear here. If the user is calling the function directly, the list will be empty.|

Here's a simple example of the `authorize` method, assuming the existance of a `'jwt_secret'` key on Based:

```js
// function name: counter
export default async ({ update, based }) => {
  // ... contents of the actual function ...
}

export const authorize = async ({ user, callStack }) => {
  // check jwt signature using the secret named 'jwt_secret'
  const token = await user.token('jwt_secret')
  if (!token) {
    // jwt authenticity cannot be verified!
    return false
  }
  const issuedAt = new Date(token.iat) // tokens payload includes issuing time
  let tokenAge = Date.now() - issuedAt //token's age in ms
  tokenAge /= 1000 // convert ms to seconds

  if (tokenAge > 259200) {
    //token is older than 3 days, expired!
    return false
  }
  if (token.userID !== 'admin') {
    // only the 'admin' userID can use this function!
    return false
  }
  if (callStack.length > 0) {
    //the function is being called from another function, not allowed!
    return false
  }
  // everything is good, authorize.
  return true
}
```
