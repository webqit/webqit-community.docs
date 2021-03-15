---
icon: layers
desc: Fullstack, object-oriented SQL for modern apps; one language and one API over diverse data storage technologies - from in-memory storage, to the client-side IndexedDB, to the server-side database like MySQL.
desc2: Speak the same familiar, powerful SQL language now with object-oriented conventions for relationships. Or query your data programmatically using an API.
categories: Featured, Libraries
tags: SQL, Indexed DB, In-Memory Store, Object Store
_after: webflo
---
# Objective SQL

<!-- BADGES/ -->

<span class="badge-npmversion"><a href="https://npmjs.org/package/@webqit/objective-sql" title="View this project on NPM"><img src="https://img.shields.io/npm/v/@webqit/objective-sql.svg" alt="NPM version" /></a></span>
<span class="badge-npmdownloads"><a href="https://npmjs.org/package/@webqit/objective-sql" title="View this project on NPM"><img src="https://img.shields.io/npm/dm/@webqit/objective-sql.svg" alt="NPM downloads" /></a></span>

<!-- /BADGES -->

[Objective SQL](https://github.com/webqit/objective-sql) is a fullstack, object-oriented SQL for modern apps; one language and one API over diverse data storage technologies - from in-memory storage, to the client-side IndexedDB, to the server-side database like MySQL.

*Speak the same familiar, powerful SQL language now with object-oriented conventions for relationships. Or query your data programmatically using an API.*

> [Visit project repo](https://github.com/webqit/objective-sql).

## Basic Usage
The following code works both in node.js and in browsers.

```js
// Import the inbuilt Object Storage library as the query client
import { ODB as Client } from '@webqit/objective-sql';

// Run a query
Client.query('SELECT fname, lname FROM users').then(result => {
    console.log(result);
});
```

## The Language
Objective SQL is the same familiar, powerful SQL language...

```sql
SELECT post_title, users.fname AS author_name FROM posts
LEFT JOIN users ON users.id = posts.author_id;
```

...but with an object-oriented syntax for relationships...

```sql
SELECT post_title, author_id->fname AS author_name FROM posts;
```

...and that's SQL without the query complexity!

[Learn more about the language](the-language) and see just what's possible. (DOCS coming soon.)

## The API
Objective SQL also lets you work programmatically using a promise-based API.

Here's the API version of the *[Basic Usage](#basic-usage)* query earlier:

```js
// The Client.open() method below opens the "default" database at version "0"
// More on this in the docs
Client.open().then(async DB => {

    // Next we open the "users" store (or table)
    let userStore = await DB.open('users');
    
    // Then we run a query
    let result = await userStore.getAll();
    console.log(result);
});
```

[Learn more about the API](the-api) and see just what's possible. (DOCS coming soon.)

## Storage
Objective SQL lets you decide between underlying storage technologies without changing code.

While we've used the inbuilt in-memory store in the examples above, we could easily switch to a persistent storage engine, like IndexedDB, by simply swiping in the appropriate client...

```js
import {
    //ODB as Client,
    IDB as Client,
    //SQL as Client,
} from '@webqit/objective-sql';
```

...and the rest of the code can remain functional. 

[Learn more about Storage](storage) and see just what's possible. (DOCS coming soon.)

## Schemas
Objective SQL completely embraces the schema idea of SQL, and offers one simple, universal way to write these schemas that describe your data - whatever the underlying storage engine.

Schema declaration is usually the first step to working with SQL. So, the `users` table we've used in the examples above should already have been declared. Here's how easy it is to do that:

```js
Client.create([
    {
        name: 'users',
        primaryKey: 'id',
        autoIncrement: true,
        fields: {id: 'number', fname: 'text', lname: 'text',},
    },
    {name: 'posts', ...},
]).then(DB => {
    // Schema created. DB ready for use
});
```

[Learn more about Schemas](schemas) and see just what's possible. (DOCS coming soon.)
