# Installation Guide
Subscript can be used both on the server and in the browser.

## On this Page
+ [Embed As Script](#embed-as-script)
+ [Install Via NPM](#install-via-npm)
+ [Next Steps](#next-steps)

## Embed as script

```html
<script src="https://unpkg.com/@webqit/subscript/dist/main.js"></script>

<script>
// The above tag loads JSEN into a global "WQ" object.
const Subscript = window.WQ.Subscript;
</script>
```

## Install via npm

```text
$ npm i -g npm
$ npm i --save @webqit/subscript
```

Import the library.

```js
// Node-style import
import Subscript from '@webqit/subscript';

// parse() an expression
var exprObj = Subscript.parse(expr);

// get result with eval()
console.log(exprObj.eval());

// convert to string anytime
console.log(exprObj.toString());
```

## Next Steps
Be sure to check the [usage examples](../README.md).