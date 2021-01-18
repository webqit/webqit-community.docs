# `Observer.getObservers)`

This function is used to obtain an object's internal observers list - an `ObserversList` instance. The returned `ObserversList` instance could then be programatically fired with some custom events.

+ [Syntax](#syntax)
+ [Usage](#usage)

## Syntax

```js
let observersList = Observer.getObservers(obj, createIfNotExists = true);
```

**Parameters**

+ `obj:                 Object|Array` - an object or array.
+ `createIfNotExists:   Boolean` - when *true*, creates an empty list when no observers exist on the object.

**Return Value**

+ `ObserversList` - when observers exist on the object.
+ `undefined` - when no observers exist on the object and the `createIfNotExists` parameter is false.

## Usage

```js
// The object
let obj = {};

// We observe the 'someProperty'
Observer.observe(obj, 'someProperty', event => {
    console.log(event.type, event.name, event.path);
    return false;
});

// Get observers list
let observersList = Observer.getObservers(obj);

// Programatically fire some custom events
observersList.fire({type: 'customEvent', name: 'someProperty',});
```

### Returning Responses Back from Firing Observers

When an event is fired, event handlers recieve a special object, called the *Response Object*, in addition to the standard event object. The Response Object can be used to, either *halt*, or *keep in sync* with what happens next.

**response.stopPropagation()** cancels the event, that is, prevents the event from reaching other event handlers. Returning `false` from the handler has the same effect. 

```js
Observer.observe(obj, 'customEvent', (event, response) => {
    response.stopPropagation();
    // Or, return false;
});
```

The initiator of the event has to flag the event as *cancellable* for the above to be honoured. It may also obtain the response object to determine the state of this response.

```js
let cancellable = true;
let response = observersList.fire({type: 'customEvent', name: 'someProperty',}, cancellable);
// Determine response state...
if (response.propagationStopped) {
}
```

**response.preventDefault()** tells the initiator of the event to skip the *default action* that it takes, if any, after firing the given event. Returning `false` from the handler has the same effect. (The event still reaches other handlers.)

```js
Observer.observe(obj, 'customEvent', (event, response) => {
    response.preventDefault();
    // Or, return false;
});
```

The initiator of the event may obtain the response object to determine the state of this response.

```js
let cancellable = true;
let response = observersList.fire({type: 'customEvent', name: 'someProperty',}, cancellable);
// Determine response state...
if (response.defaultPrevented) {
}
```

**response.waitUntil(promise)** tells the initiator of the event to wait until a *Promise* is resolved before continuing with further operations. Returning a `Promise` from the handler has the same effect. (The event still reaches other handlers without waiting.)

```js
Observer.observe(obj, 'customEvent', (event, response) => {
    let promise = new Promise(resolve => {
        setTimeout(resolve, 2000);
    });
    response.waitUntil(promise);
    // Or, return promise;
});
```

The initiator of the event may obtain the response object to determine the state of this response. The state of this response becomes a promise when one or more handlers return a promise.

```js
let response = observersList.fire({type: 'customEvent', name: 'someProperty',});
// Determine response state...
if (response.promises) {
    response.promises.then(() => {
    });
}
```
