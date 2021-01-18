# `Observer.defineProperty()`

This method is used to define a property on an object or modifies an existing property. It corresponds to the JavaScript's `Reflect.defineProperty()` function.

`Observer.defineProperty()` brings the added benefit of driving [*observers*](../observe) and [*interceptors*](../intercept).

The `Observer.def()` function is an alias of this function and can be used interchangeably.

+ [Syntax](#syntax)
+ [Usage](#usage)
+ [Usage as a Trap's defineProperty Handler](#usage-as-a-traps-defineproperty-handler)
+ [Intercepting `Observer.defineProperty()`](#Intercepting-observer.defineproperty)
+ [Related Methods](#related-methods)

## Syntax

```js
// Define a property
Observer.defineProperty(obj, propertyName, propertyDescriptor);

// Define a list of properties
Observer.defineProperty(obj, propertyNames, propertyDescriptor);
```

**Parameters**

+ `obj:                 Object|Array` - an object or array.
+ `propertyName:        String` - the property to define.
+ `propertyNames:       Array` - a list of properties to define with the same *propertyDescriptor*.
+ `propertyDescriptor:  Object` - the property descriptor as specified for [`Reflect.defineProperty()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect).

**Return Value**

*Boolean*
*Object* - See [Returning Responses Back from Observers](#returning-responses-back-from-observers)

## Usage

### Defining a Specific Property

```js
// On an object
Observer.defineProperty(obj, 'fruit', {value:'orange'});
```

### Defining a List of Properties

```js
// On an object
Observer.defineProperty(obj, ['fruit', 'brand'], {value:'orange'});
```

### Passing a Value to Observers

The `params.detail` property can be used to pass a value specifically to observers that might be responding to the `del` event. Any type of value can be passed.

```js
Observer.defineProperty(obj, propertyName, propertyDescriptor, {
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

When a *def* operation fires an event, event handlers recieve a special object, called the *Response Object*, in addition to the standard event object. The Response Object can be used to, either *halt*, or *keep in sync* with what happens next.

**response.stopPropagation()** cancels the event, that is, prevents the event from reaching other event handlers. Returning `false` from the handler has the same effect. 

```js
Observer.observe(obj, propertyName, (event, response) => {
    response.stopPropagation();
    // Or, return false;
});
```

The initiator of the *def* operation has to flag the event as *cancellable* for the above to be honoured. It may also obtain the response object to determine the state of this response.

```js
let response = Observer.defineProperty(obj, propertyName, propertyDescriptor, {
    cancellable: true,
    responseObject: true,
});
// Determine response state...
if (response.propagationStopped) {
}
```

**response.preventDefault()** tells the initiator of the *def* operation to skip the *default action* that it takes, if any, after *def* operations. Returning `false` from the handler has the same effect. (The event still reaches other handlers.)

```js
Observer.observe(obj, propertyName, (event, response) => {
    response.preventDefault();
    // Or, return false;
});
```

The initiator of the *def* operation may obtain the response object to determine the state of this response.

```js
let response = Observer.defineProperty(obj, propertyName, propertyDescriptor, {
    responseObject: true,
});
// Determine response state...
if (response.defaultPrevented) {
}
```

**response.waitUntil(promise)** tells the initiator of the *def* operation to wait until a *Promise* is resolved before continuing with further operations. Returning a `Promise` from the handler has the same effect. (The event still reaches other handlers without waiting.)

```js
Observer.observe(obj, propertyName, (event, response) => {
    let promise = new Promise(resolve => {
        setTimeout(resolve, 2000);
    });
    response.waitUntil(promise);
    // Or, return promise;
});
```

The initiator of the *def* operation may obtain the response object to determine the state of this response. The state of this response becomes a promise when one or more handlers return a promise.

```js
let response = Observer.defineProperty(obj, propertyName, propertyDescriptor, {
    responseObject: true,
});
// Determine response state...
if (response.promises) {
    response.promises.then(() => {
    });
}
```

## Usage as a Trap's defineProperty Handler

`Observer.defineProperty()` returns a *Boolean* value by default, and can, therefore, be used as the "defineProperty" handler in Proxy traps.

```js
let _obj = new Proxy(obj, {defineProperty: Observer.defineProperty});
let _arr = new Proxy(arr, {defineProperty: Observer.defineProperty});
```

*Define* operations will now be forwarded to `Observer.defineProperty()` and [*observers*](../observe) and [*interceptors*](../intercept) that may be bound to the object will continue to respond.

```js
Reflect.defineProperty(_obj, 'fruit', {value:'apple'});
```

## Intercepting `Observer.defineProperty()`

Using [`Observer.intercept()`](../intercept), it is possible to intercept calls to `Observer.defineProperty()`. When a "def" operation triggers an interceptor, the interceptor will receive an event object containing the property name to define and the property descriptor.

```js
Observer.intercept(obj, 'def', (event, recieved, next) => {
    // What we recieved...
    console.log(event.name, event.descriptor);
    // The define operation, and the return value - Boolean
    return Reflect.defineProperty(obj, event.name, event.descriptor);
});

Observer.defineProperty(obj, 'fruit', {value:'apple'});
```

The interceptor is expected to return *true* when the deletion is successful; *false* otherwise.

## Related Methods

+ [`Observer.observe()`](../observe)
+ [`Observer.intercept()`](../intercept)
