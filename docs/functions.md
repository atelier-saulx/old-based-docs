# Functions in Based

One of Based most useful features is the ability to upload and execute server-side functions, that can recursively interact with Based themselves.

It is possible to load custom server-side functions [using the CLI](cli.md) or our web interface.

These functions can be **observable**, meaning that the function will keep running as long as it's being observed, and emit an event and relative payload whenever required. Essentially, they can be observed just like a normal database query.

Non-observable functions instead only run when called, and stop when execution is completed.

An observable function is by default also **shared**, meaning that multiple clients requesting the function with the same payload will share the same instance of the function, thus optimising performance.  
The _non-shared_ behaviour is that every client gets its own instance of the function, however this is discouraged, for performance reasons, unless necessary.

## Server-side

Server-side functions can be either executed once (if non-observable), or kept running (if observables, and with active subscription).

The use of **these functions can be restricted** using the `authorize` method that Based offers.

### Syntax

When writing a server-side method, we must export it as a module using the `export default async` syntax.  
It is possible to export the **`authorize` function** from the same file by simply using a named export instead of the default one.

:exclamation: In the exported function, **the return value has to be a callback function** that will be triggered when the client (or all the clients in the chase of shared observables) stop observing the function.  
This return function offers a chance to execute a graceful exit from the function. **In any case, a few seconds after the last observer unsubscribes from the function, the worker containing the function's instance will be forcefully terminated.**

Inside the main method's module, we have three main properties available through Based, `based`, `update`, and `payload`:

### Properties

##### `based`

This gives us access to a full Based client instance limited to the env where the function is called. Other server-side functions can also be executed using this client.

##### `update`

This is the function that sends new data to the client, and it takes as argument the payload to send. It has to be called at least once to let the client exit the `await`.

##### `payload`

This is the object containg the client's payload, if it exists.

##### Example of server-side function:

```js
import { hash } from '@saulx/hash'

export default async ({ based, update, payload }) => {
  // print the user payload in the server console
  console.info('hi, here is the user payload', payload)

  const closingFunction = based.observe(
    {
      // using observe within the server-side function!
      children: {
        $all: true,
        $list: true,
      },
    },
    (data) => {
      console.info('data changed! ', data)
    }
  )
  let cnt = 0
  const int = setInterval(() => {
    // send an object to the client every 1000ms:
    update({
      cnt: ++cnt,
      y: 1321123123,
      x: hash(cnt), // use imported functions as normal
    })
  }, 1000)
  return () => {
    // exit the function cleanly
    closingFunction() // unsubscribe from the based query
    clearInterval(int) // kill interval method
  }
}

export const authorize = async ({ user, callStack, based }) => {
  const token = await user.token('blabla')
  if (callStack.length > 0) {
    return true //user is authorized!
  }

  if (token.foo == 'bar') {
    return true //user is authorized!
  }
  return false //user is not authorized.
}
```

### Authorization

Check [our page about secrets](secrets.md) to find out more.

## Client-side

The Based client includes a few methods to interact with functions on the server.

For **_observable_** functions, the `based.get()` and `based.observe()` methods can be used, similarly to a normal database query, but with a slightly different syntax.  
For non-observables, the function can be executed using the `based.call()` method, which returns the return value of the function, if there's one.

### `based.get()`

This method has a slightly different syntax compared to when it's used with a database query. It takes the following arguments:

| Arg       | Type   | Properties | Description                                   |
| --------- | ------ | ---------- | --------------------------------------------- |
| `name`    | string | -          | Name of the function required.                |
| `payload` | any    | optional   | Payload to pass to the function, if required. |

This method returns on the next call of the server-side [`update()` method](#update)

<!-- , and it's _null_ if the function has no return statement. ? -->

### `based.observe()`

This method has a slightly different syntax compared to when it's used with a database query. It takes the following arguments:

| Arg                      | Type                                                | Properties | Description                                                                    |
| ------------------------ | --------------------------------------------------- | ---------- | ------------------------------------------------------------------------------ |
| `name`                   | `string`                                            | -          | Name of the functions to get.                                                  |
| `payload`                | `any`                                               | optional   | Payload to pass to the function, if required.                                  |
| `onData(data, checksum)` | `onData: function`, `data: any`, `checksum: number` | -          | Callback function Based will execute when the function's return value changes. |
| `onErr(err)`             | `onErr: function`, `err: Error`                     | optional   | Call back function to execute in case of error.                                |

The return value of `observe` will be the function to unsubscribe from the observable.

### `based.call()`

This method has a slightly different syntax compared to when it's used with a database query. It takes the following arguments:

| Arg       | Type   | Properties | Description                                   |
| --------- | ------ | ---------- | --------------------------------------------- |
| `name`    | string | -          | Name of the function required.                |
| `payload` | any    | optional   | Payload to pass to the function, if required. |

This method returns the return value of the function itself, or _null_ if the function has no return statement.
