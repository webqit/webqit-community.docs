# Installation
Observer is usable in both browser and server environments.

## Option 1: Embed as script

```html
<script src="https://unpkg.com/@webqit/observer/dist/main.js"></script>

<script>
// The above tag loads Observer into a global "WQ" object.
const Observer = WQ.Observer;
</script>
```

## Option 2: Install via npm

```text
$ npm i -g npm
$ npm i --save @webqit/observer
```

**Import with the `import` keyword:**

```js
// Node-style import
import Observer from '@webqit/observer';

// Standard JavaScript import. (Actual path depends on where you installed Observer to.)
import Observer from './node_modules/@webqit/observer/src/index.js';
```

```js
let obj = {};
Observer.observe(obj, events => {
    console.log(events.map(e => e.type + ': ' + e.name));
});

Observer.set(obj, 'property', value);
```

## Documentation
+ [Examples](../examples)
+ [API Docs](../api)
