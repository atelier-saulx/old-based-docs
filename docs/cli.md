# Based-cli

It is possible to interact with Based from the command line using the **Based CLI.**

## Install

To install the CLI type in your terminal

```bash
$ npx add @based/cli
```

you can then run it by typing

```sh
$ npx based <command> [options]
```

**:exclamation: Check out [this folder](../packages/cli/examples) for some practical examples.**

## Commands

There are four commands available in the CLI:

| Command     | Description                                   |
| ----------- | --------------------------------------------- |
| `functions` | Add, remove, or change server-side functions. |
| `configure` | Configure the database                        |
| `secret`    | Add and remove secret keys                    |
| `help`      | Show help message                             |

### `functions` command

To load a function to Based using the command-line interface, use the following command:

```bash
$ npx based functions myfunctionname --file filename.js --observable
```

This command also allows to change an existing function by simply passing it with the same name. Based creates an **history of versions** of the function, up to a maximum of 10.

The options for the `based functions` command are

| Shorthand | Extended       | Details                                                                                                                         |
| --------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `-f`      | `--file`       | Takes the filepath of the file containing the function.                                                                         |
| `-o`      | `--observable` | Make function observable.                                                                                                       |
| `-S`      | `--no-sharing` | Specify that observable function is **not** shared. Observable functions are shared by default unless this option is specified. |
| `-c`      | `--code`       | Insert inline function.                                                                                                         |

### `configure` command

This command allows the user to change the database's schema, setting it either from a file or inline.

It is possible to pass an array of schemas, each one specifying the database name. Check out the [schema docs](schema.md) for more information on what that means.

The options for the `based configure` command are

| Shorthand | Extended   | Details                                                                                        |
| --------- | ---------- | ---------------------------------------------------------------------------------------------- |
| `-f`      | `--file`   | Takes the filepath to the file containg the schema. Supported formats are JavaScript and JSON. |
| `-C`      | `--config` | Takes inline schema.                                                                           |
| `-d`      | `--db`     | Specify which database this schema should be applied to.                                       |

### `secret` command

Use this to add and remove secret variables to your organization. It creates a token with the specified name, that can later be used in the code with the `based.secret()` method.

```bash
$ npx based secret mytokenname -f filename.pub
```

The options for the `based secret` command are

| Shorthand | Extended   | Details                   |
| --------- | ---------- | ------------------------- |
| `-f`      | `--file`   | Add a secret from a file  |
| `-v`      | `--value`  | Add secret's value inline |
| `-d`      | `--delete` | Removes the secret.       |
