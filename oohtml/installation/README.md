# Installation Guide
This library is a polyfill for the [proposed](https://discourse.wicg.io/t/proposal-chtml/4716) OOHTML suite.

## On this Page
+ [Embed As Script](#embed-as-script)
+ [Install Via NPM](#install-via-npm)
  + [Server-Side Initialization](#server-side-initialization)
+ [Next Steps](#next-steps)

## Embed As Script 

+ **Embed The Complete Suite** - Embed the build below for everything about OOHTML.

  ```html
  <script src="https://unpkg.com/@webqit/oohtml/dist/main.js"></script>
  ```

+ **Embed Individual Features** - Find a build below for a specific OOHTML feature.

  + **[Namespaced HTML](../namespaced-html/README.md)** - `<script src="https://unpkg.com/@webqit/oohtml/dist/namespaced-html.js"></script>`

  + **[Named Templates](../named-templates/README.md)** - `<script src="https://unpkg.com/@webqit/oohtml/dist/named-templates.js"></script>`

  + **[The State API](../the-state-api/README.md)** - `<script src="https://unpkg.com/@webqit/oohtml/dist/state.js"></script>`

  + **[Scoped Scripts](../scoped-scripts/README.md)** - `<script src="https://unpkg.com/@webqit/oohtml/dist/scoped-scripts.js"></script>`

  + **[HTML Partials](../html-partials/README.md)** - `<script src="https://unpkg.com/@webqit/oohtml/dist/html-partials.js"></script>`

## Install Via NPM

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
Be sure to check Polyfill Support for each of [OOHTML's features](../README.md) as documented alongside each feature.



