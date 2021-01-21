---
_after: examples
---
# Installation Guide
This library is a polyfill for OOHTML.

## Option 1: Embed As Script 

+ **Embed The Complete Suite** - Embed the build below for everything about OOHTML.

  ```html
  <script src="https://unpkg.com/@webqit/oohtml/dist/main.js"></script>
  ```

+ **Or Embed Individual Features** - Find a build below for a specific OOHTML feature.

  + **[HTML Modules](../html-modules)** - `<script src="https://unpkg.com/@webqit/oohtml/dist/html-modules.js"></script>`
  + **[HTML Imports](../html-imports)** - `<script src="https://unpkg.com/@webqit/oohtml/dist/html-imports.js"></script>`
  + **[Namespaced HTML](../namespaced-html)** - `<script src="https://unpkg.com/@webqit/oohtml/dist/namespaced-html.js"></script>`
  + **[The State API](../the-state-api)** - `<script src="https://unpkg.com/@webqit/oohtml/dist/state-api.js"></script>`
  + **[Subscript](../subscript)** - `<script src="https://unpkg.com/@webqit/oohtml/dist/subscript.js"></script>`

## Option 2: Install Via NPM

```text
$ npm i -g npm
$ npm i --save @webqit/oohtml
```

The installed package is designed to be *initialized* with the *window* object of the current browser or server evironment.

+ **Initialize the Complete Suite** - Initialize the module below for everything about OOHTML.

  ```js
  // Import
  import OOHTML from '@webqit/oohtml';
  // Initialize
  OOHTML(window);
  ```

### Server-Side Initialization
To use OOHTML for server-side rendering, a library like [jsdom](https://github.com/jsdom/jsdom) would be used to create a DOM instance, and the instance's `window` object would be used to initialize OOHTML.

```js
// Import OOHTML
import OOHTML from '@webqit/oohtml';
// Import jsDom
import jsdom from 'jsdom';
// Utilities we'll need
import fs from 'fs';
import path from 'path';

// Read the HTML document file from the server
const documentFile = fs.readFileSync(path.resolve('./index.html'));
// Instantiate jsdom so we can obtain the "window" for OOHTML
// Detailed instruction on setting up jsdom is available in the jsdom docs
const JSDOM = new jsdom.JSDOM(documentFile.toString());

// Initialize OOHTML...
OOHTML(JSDOM.window);

// And we can use OOHTML as normal
const document = JSDOM.window.document;
// The document's templates object...
let template1 = document.templates.template1;
// etc
```

## Next Steps
Be sure to check Polyfill Support for each of [OOHTML's features](../) as documented alongside each feature. Then go native and build something!



