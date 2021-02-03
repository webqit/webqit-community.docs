---
icon: braces
title: Observer
desc: Observability and Interceptability API for JavaScript Objects and Arrays.
categories: Web-Native
tags: observer
_after: oohtml
---
# The Observer API

<!-- BADGES/ -->

<span class="badge-npmversion"><a href="https://npmjs.org/package/@webqit/observer" title="View this project on NPM"><img src="https://img.shields.io/npm/v/@webqit/observer.svg" alt="NPM version" /></a></span>
<span class="badge-npmdownloads"><a href="https://npmjs.org/package/@webqit/observer" title="View this project on NPM"><img src="https://img.shields.io/npm/dm/@webqit/observer.svg" alt="NPM downloads" /></a></span>

<!-- /BADGES -->

*[Observer](https://github.com/webqit/observer)* is an API for intercepting and observing JavaScript objects and arrays.

> [Visit project repo](https://github.com/webqit/observer).
> [Visit project homepage](https://webqit.io/tooling/observer).

```js
let obj = {};
Observer.observe(obj, events => {
    events.forEach(e => {
        console.log(e.type, e.name, e.path, e.value, e.oldValue);
    });
});

Observer.set(obj, path, value);
```

Follow the [installation guide](installation) to add the Observer API to a page or project.

## Documentation
+ [Examples](examples)
+ [API](api)

## Design Goals
See the [API explainer](explainer).

## Issues
To report bugs or request features, please submit an [issue](https://github.com/webqit/observer/issues).

## License
MIT.
