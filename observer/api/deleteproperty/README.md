# `Observer.deleteProperty()`

This method is used to delete an object's property. It corresponds to the JavaScript's `Reflect.deleteProperty()` function, which is itself the programmatic alternative to the assignment expression – `delete obj.property`.

`Observer.deleteProperty()` brings the added benefit of triggering [*observers*](../observe) and [*interceptors*](../intercept).

The `Observer.del()` function is an alias of this function and can be used interchangeably.

+ [Syntax](#syntax)
+ [Usage](#usage)
+ [Usage as a Trap's "deleteProperty" Handler](#usage-as-a-traps-deleteproperty-handler)
+ [Intercepting `Observer.deleteProperty()`](#Intercepting-observer.deleteproperty)
+ [Related Methods](#related-methods)

## Syntax

```js
// Delete a specific property
Observer.deleteProperty(obj, propertyName);

// Delete a list of properties
Observer.deleteProperty(obj, propertyNames);
```

**Parameters**

+ `obj:             Object|Array` - an object or array.
+ `propertyName:    String` - the property to delete.
+ `propertyNames:   Array` - a list of properties to delete.

**Return Value**

*Boolean*
*Object* - See [Returning Responses Back from Observers](#returning-responses-back-from-observers)

## Usage

### Deleting a Specific Property

```js
// On an object
Observer.deleteProperty(obj, 'fruit');
// On an array
Observer.deleteProperty(arr, 0);
```

### Deleting a List of Properties

```js
// On an object
Observer.deleteProperty(obj, ['fruit', 'brand']);
// On an array
Observer.deleteProperty(arr, [0, 3]);
```

### Passing a Value to Observers

The `params.detail` property can be used to pass a value specifically to observers that might be responding to the `del` event. Any type of value can be passed.

```js
Observer.deleteProperty(obj, propertyName, {
    detail: 'This is observer-specific detail',
});
```

The *detail* above would now be available to every handler.

```js
Observer.observe(obj, propertyName, event => {
    console.log(event.detail);
});
```

### Returning Responses Back from Observers

When a *del* operation fires an event, event handlers recieve a special object, called the *Response Object*, in addition to the standard event object. The Response Object can be used to, either *halt*, or *keep in sync* with what happens next.

**response.stopPropagation()** cancels the event, that is, prevents the event from reaching other event handlers. Returning `false` from the handler has the same effect. 

```js
Observer.observe(obj, propertyName, (event, response) => {
    response.stopPropagation();
    // Or, return false;
});
```

The initiator of the *del* operation has to flag the event as *cancellable* for the above to be honoured. It may also obtain the response object to determine the state of this response.

```js
let response = Observer.deleteProperty(obj, propertyName, {
    cancellable: true,
    responseObject: true,
});
// Determine response state...
if (response.propagationStopped) {
}
```

**response.preventDefault()** tells the initiator of the *del* operation to skip the *default action* that it takes, if any, after *del* operations. Returning `false` from the handler has the same effect. (The event still reaches other handlers.)

```js
Observer.observe(obj, propertyName, (event, response) => {
    response.preventDefault();
    // Or, return false;
});
```

The initiator of the *del* operation may obtain the response object to determine the state of this response.

```js
let response = Observer.deleteProperty(obj, propertyName, {
    responseObject: true,
});
// Determine response state...
if (response.defaultPrevented) {
}
```

**response.waitUntil(promise)** tells the initiator of the *del* operation to wait until a *Promise* is resolved before continuing with further operations. Returning a `Promise` from the handler has the same effect. (The event still reaches other handlers without waiting.)

```js
Observer.observe(obj, propertyName, (event, response) => {
    let promise = new Promise(resolve => {
        setTimeout(resolve, 2000);
    });
    response.waitUntil(promise);
    // Or, return promise;
});
```

The initiator of the *del* operation may obtain the response object to determine the state of this response. The state of this response becomes a promise when one or more handlers return a promise.

```js
let response = Observer.deleteProperty(obj, propertyName, {
    responseObject: true,
});
// Determine response state...
if (response.promises) {
    response.promises.then(() => {
    });
}
```

## Usage as a Trap's "deleteProperty" Handler

`Observer.deleteProperty()` returns a *Boolean* value by default, and can, therefore, be used as the "deleteProperty" handler in Proxy traps.

```js
let _obj = new Proxy(obj, {deleteProperty: Observer.deleteProperty});
let _arr = new Proxy(arr, {deleteProperty: Observer.deleteProperty});
```

Delete operations will now be forwarded to `Observer.deleteProperty()` and [*observers*](../observe) and [*interceptors*](../intercept) that may be bound to the object will continue to respond.

```js
delete _obj.fruit;
delete _arr[2];
```

## Intercepting `Observer.deleteProperty()`

Using [`Observer.intercept()`](../intercept), it is possible to intercept calls to `Observer.deleteProperty()`. When a "del" operation triggers an interceptor, the interceptor will receive an event object containing the property name to delete.

```js
Observer.intercept(obj, 'del', (event, recieved, next) => {
    // What we recieved...
    console.log(event.name);
    // The delete operation
    delete obj[event.name];
    // The return value - Boolean
    return true;
});

Observer.deleteProperty(obj, 'fruit');
```

When the "del" operation is of multiple deletion, the interceptor gets fired for each pair while also recieving the total list of properties as a hint - via `event.related`.

```js
Observer.intercept(obj, 'del', (event, recieved, next) => {
    // What we recieved...
    console.log(event.name, event.related);
    // The delete operation
    delete obj[event.name];
    // The return value - Boolean
    return true;
});

Observer.deleteProperty(obj, ['orange', 'apple']);
```

The above should trigger our interceptor twice with `event.related` being `['fruit', 'brand']`.

The interceptor is expected to return *true* when the deletion is successful; *false* otherwise.

## Related Methods

+ [`Observer.observe()`](../observe)
+ [`Observer.intercept()`](../intercept)
