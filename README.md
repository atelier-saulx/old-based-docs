<!-- > //TODO this should be written by someone that actually knows how to write. -->

# :exclamation: ARCHIVED REPO

This repo has been archived, all the documentation as been moved to the [based](https://github.com/atelier-saulx/based/) public repo.

# Documentation

**Based** as a data platform offers tool to change it's server-side behaviour, as well as tool to develop on the client side.

On the **server-side**, Based allows the user to [upload custom functions](docs/functions.md), and [manage authorization](docs/secrets.md).

On the **client-side**, we provide a set of methods which utilize the Based query language to interact with a Based instance. The syntax of these methods changes slightly depending on the type of operation requested.

Based also comes equipped **with a CLI**, find more about it [here](docs/cli.md).

## Client-side methods

**Check out the Based query language for the [schema](sdocs/chema.md), [sets](docs/set.md), and for [gets](docs/get.md).**

In summary, the main methods the Based client offers are

| Name                             | Args                                                                 | Function                                        |
| -------------------------------- | -------------------------------------------------------------------- | ----------------------------------------------- |
| [`based.set()`](docs/set.md)          | `query`                                                              | Add or modify data on the database              |
| [`based.get()`](docs/get.md)          | `query`                                                              | Extract and shape data from the database.       |
| `based.observe()`                | `query, onData: (data: any, checksum: number), onErr?: (err: Error)` | Subscribe to a query or server-side method.     |
| `based.delete()`                 | `id`                                                                 | Remove a node from the database                 |
| [`based.configure()`](docs/schema.md) | `schema query`                                                       | Change schema.                                  |
| `based.digest()`                 | `string`                                                             | Hash the input string, returning sha-64 digest. |
| [`based.call()`](docs/functions.md)   | `string`                                                             | Executes a server-side function.                |
