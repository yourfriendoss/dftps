# DFtpS - Deno Ftp Server

![alt text](./dftps_logo.png "Logo Title Text 1")
[![deno doc](https://doc.deno.land/badge.svg)](https://doc.deno.land/https/deno.land/x/dftps/mod.ts)
  
DFtpS is an FTP server based on [ftp-srv](https://github.com/autovance/ftp-srv) with Deno.

- [Install](#install)
- [Make your own](#make-your-own)
- [Deno Dependencies](#deno-dependencies)
- [List of FTP commands](#list-of-ftp-commands)
- [Contributing](#contributing)
- [References](#references)

## Usage

### Install

```sh
  curl -fsSL https://deno.land/x/dftps/install.sh | sh
```

### Install Specific Version

```sh
  curl -fsSL https://deno.land/x/dftps/install.sh | sh -s v1.0.0
```

* * *

## Make your own

### Simple

```ts
  import { Server } from "https://deno.land/x/dftps/server/mod.ts";
  import type { UsernameResolvable, LoginResolvable } from "https://deno.land/x/dftps/server/connection.ts";

  const serve = new Server({
    port: 21,
    hostname: "127.0.0.1" // (optional)
  }, {
    /** Url for passive connection. */
    pasvUrl?: string; // (optional)
    /** Minimum port for passive connection. */
    pasvMin?: number; // (optional)
    /** Maximum port for passive connection. */
    pasvMax?: number; // (optional)
    /** Handle anonymous connexion. */
    anonymous?: boolean; // (optional)
    /** Sets the format to use for file stat queries such as "LIST". */
    fileFormat?: string; // (optional)
    /** Array of commands that are not allowed */
    blacklist?: string[];
  });

  for await (const connection of serve) {
    const { awaitUsername, awaitLogin } = connection;
    /** waiting to receiving username from connection */
    awaitUsername.then(({ username, resolveUsername }: UsernameResolvable) => {
      if (!username !== "my-username") return resolveUsername.reject("Incorrect username!");
      resolveUsername.resolve();
    });
    /** waiting to receiving password from connection and finalize the user authenticate */
    awaitLogin.then(async ({ password, resolvePassword }: LoginResolvable) => {
      if (password !== "my-password") return resolvePassword.reject("Wrong password!");
      resolvePassword.resolve({ root: "my-folder", uid: 1000, gid: 1000 });
    });
  }
```

### With database

```ts
  import { Server } from "https://deno.land/x/dftps/server/mod.ts";
  import type { UsernameResolvable, LoginResolvable } from "https://deno.land/x/dftps/server/connection.ts";

  import createDb from "../db/mod.ts";
  import Users from "../db/Users.ts";

  await createDb({
    connector: "MariaDB" | "MongoDB" | "MySQL" | "PostgreSQL" | "SQLite",

    /* Maria, MySQL, PostgreSQL Example
    database: 'my-database',
    host: 'url-to-db.com',
    username: 'username',
    password: 'password',
    port: 3306 // (optional)
    */

    /* MongoDB Example
    uri: 'mongodb://127.0.0.1:27017',
    database: 'test'
    */

    /* SQLite Example
    filepath: './database.sqlite'
    */
  });

  const serve = new Server({
    port: 21,
    hostname: "127.0.0.1" // (optional)
  }, {
    /** Url for passive connection. */
    pasvUrl: string, // (optional)
    /** Minimum port for passive connection. */
    pasvMin: number, // (optional)
    /** Maximum port for passive connection. */
    pasvMax: number, // (optional)
    /** Handle anonymous connexion. */
    anonymous: boolean, // (optional)
    /** Sets the format to use for file stat queries such as "LIST". */
    fileFormat: string, // (optional)
    /** Array of commands that are not allowed */
    blacklist: string[] // (optional)
  });

  for await (const connection of serve) {
    const { awaitUsername, awaitLogin } = connection;
    /** Get all users in database */
    const users = await Users.select("username", "password", "root", "uid", "gid").all();
    let user: Model;
    /** Waiting to receiving username from connection */
    awaitUsername.then(({ username, resolveUsername }: UsernameResolvable) => {
        const found = users.find(u => u.username === username);
        if (!found) return resolveUsername.reject("Incorrect username!");
        user = found;
        resolveUsername.resolve();
    });
    /** Waiting to receiving password from connection and finalize the user authenticate */
    awaitLogin.then(async ({ password, resolvePassword }: LoginResolvable) => {
        if (!user) return resolvePassword.reject("User not found!");
        if (! await verify(password, (user.password as string))) return resolvePassword.reject("Wrong password!");

        const { root, uid, gid } = user;
        resolvePassword.resolve({ root: (root as string), uid: (uid as number), gid: (gid as number) });
    });
  }

```

* * *

## Deno Dependencies

- ### [Deno](https://deno.land)

  - [async](https://deno.land/std@0.95.0/async)
  - [io](https://deno.land/std@0.95.0/io)
  - [path](https://deno.land/std@0.95.0/path)
  - [fs](https://deno.land/std@0.95.0/fs)

- ### [Getport](https://deno.land/x/getport)

- ### [date_format_deno](https://deno.land/x/date_format_deno)

- ### [cliffy](https://deno.land/x/cliffy)

- ### [scrypt](https://deno.land/x/scrypt)

- ### [denodb](https://deno.land/x/denodb)

## [List of FTP commands](https://en.wikipedia.org/wiki/List_of_FTP_commands)

  Implemented   | Comand        | Description
  ------------- | ------------- | -------------
  🗹             | ABOR          | Abort an active file transfer.
  ☐             | ACCT          | Account information.
  ☐             | ADAT          | Authentication/Security Data.
  🗹             | ALLO          | Allocate sufficient disk space to receive a file.
  🗹             | ABOR          | Abort an active file transfer.
  🗹             | APPE          | Append (with create).
  ☐             | AUTH          | Authentication/Security Mechanism.
  ☐             | AVBL          | Get the available space.
  ☐             | CCC           | Clear Command Channel.
  🗹             | CDUP          | Change to Parent Directory.
  ☐             | CONF          | Confidentiality Protection Command.
  ☐             | CSID          | Client / Server Identification.
  🗹             | CWD           | Change working directory.
  🗹             | DELE          | Delete file.
  ☐             | DSIZ          | Get the directory size.
  ☐             | ENC           | Privacy Protected Channel.
  🗹             | EPRT          | Specifies an extended address and port to which the server should connect.
  🗹             | EPSV          | Enter extended passive mode.
  🗹             | FEAT          | Get the feature list implemented by the server.
  🗹             | HELP          | Returns usage documentation on a command if specified, else a general help document is returned.
  ☐             | HOST          | Identify desired virtual host on server, by name.
  ☐             | LANG          | Language Negotiation.
  🗹             | LIST          | Returns information of a file or directory if specified, else information of the current working directory is returned.
  ☐             | LPRT          | Specifies a long address and port to which the server should connect.
  ☐             | LPSV          | Enter long passive mode.
  🗹             | MDTM          | Return the last-modified time of a specified file.
  ☐             | MFCT          | Modify the creation time of a file.
  ☐             | MFF           | Modify fact (the last modification time, creation time, UNIX group/owner/mode of a file).
  ☐             | MFMT          | Modify the last modification time of a file.
  ☐             | MIC           | Integrity Protected Command.
  🗹             | MKD           | Make directory.
  ☐             | MLSD          | Lists the contents of a directory if a directory is named.
  ☐             | MLST          | Provides data about exactly the object named on its command line, and no others.
  🗹             | MODE          | Sets the transfer mode (Stream, Block, or Compressed).
  🗹             | NLST          | Returns a list of file names in a specified directory.
  🗹             | NOOP          | No operation (dummy packet; used mostly on keepalives).
  🗹             | OPTS          | Select options for a feature (for example OPTS UTF8 ON).
  🗹             | PASS          | Authentication password.
  🗹             | PASV          | Enter passive mode.
  🗹             | PBSZ          | Protection Buffer Size.
  🗹             | PORT          | Specifies an address and port to which the server should connect.
  🗹             | PROT          | Data Channel Protection Level.
  🗹             | PWD           | Print working directory. Returns the current directory of the host.
  🗹             | QUIT          | Disconnect.
  ☐             | REIN          | Re initializes the connection.
  🗹             | REST          | Restart transfer from the specified point.
  🗹             | RETR          | Retrieve a copy of the file.
  🗹             | RMD           | Remove a directory.
  ☐             | RMDA          | Remove a directory tree.
  🗹             | RNFR          | Rename from.
  🗹             | RNTO          | Rename to.
  🗹             | SITE          | Sends site specific commands to remote server (like SITE IDLE 60 or SITE UMASK 002). Inspect SITE HELP output for complete list of supported commands.
  🗹             | SIZE           | Return the size of a file.
  ☐             | SMNT           | Mount file structure.
  ☐             | SPSV           | Use single port passive mode (only one TCP port number for both control connections and passive-mode data connections).
  🗹             | STAT           | Returns information on the server status, including the status of the current connection.
  🗹             | STOR           | Accept the data and to store the data as a file at the server site.
  🗹             | STOU           | Store file uniquely.
  🗹             | STRU           | Set file transfer structure.
  🗹             | SYST           | Return system type.
  ☐             | THMB           | Get a thumbnail of a remote image file.
  🗹             | TYPE           | Sets the transfer mode (ASCII/Binary).
  🗹             | USER           | Authentication username.
  ☐             | XCUP           | Change to the parent of the current working directory.
  ☐             | XMKD           | Make a directory.
  ☐             | XPWD           | Print the current working directory.
  ☐             | XRCP           |
  ☐             | XRMD           | Remove the directory.
  ☐             | XRSQ           |
  ☐             | XSEM           | Send, mail if cannot.
  ☐             | XSEN           | Send to terminal.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md)

## References

- [Ftp](https://cr.yp.to/ftp.html)
- [Ftp commands](https://en.wikipedia.org/wiki/List_of_FTP_commands)
- [Ftp reply codes](https://en.wikipedia.org/wiki/List_of_FTP_server_return_codes)
