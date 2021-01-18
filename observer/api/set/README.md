# `Observer.set()`

This method is used to set the value of an object's property. It corresponds to the JavaScript's `Reflect.set()` function, which is itself the programmatic alternative to the assignment expression – `obj.property = value`.

`Observer.set()` brings the added benefit of triggering [*observers*](../observe) and [*interceptors*](../intercept).

+ [Syntax](#syntax)
+ [Usage](#usage)
+ [Usage as a Trap's "set" Handler](#usage-as-a-traps-set-handler)
+ [Usage with Property Setters](#usage-with-property-setters)
+ [Intercepting `Observer.set()`](#Intercepting-observer.set)
+ [Related Methods](#related-methods)

## Syntax

```js
// Set or modify a specific property
Observer.set(obj, propertyName, value[, params = {}]);

// Set or modify a list of properties with the same value
Observer.set(obj, propertyNames, value[, params = {}]);

// Perform multiple key/value assignments
Observer.set(obj, keyValueMap[, = {}]);
```

**Parameters**

+ `obj:             Object|Array` - an object or array.
+ `propertyName:    String|Number` - the property to modify.
+ `propertyNames:   Array` - a list of properties to modify.
+ `value:           Any` - the value to set.
+ `keyValueMap:     Object` - an object of key/value pairs.
+ `params:          Object` - optional paramters for the operation.

**Return Value**

*Boolean*
*Object* - See [Returning Responses Back from Observers](#returning-responses-back-from-observers)

## Usage

### Assigning On a Specific Property

```js
// On an object
Observer.set(obj, 'fruit', 'orange');
// On an array
Observer.set(arr, 0, 'orange');
```

### Assigning On a List of Propertieskeys

```js
// On an object
Observer.set(obj, ['fruit', 'brand'], 'apple');
// On an array
Observer.set(arr, [0, 3], 'apple');
```

### Multiple Key/Value Assignment

```js
// On an object
Observer.set(obj, {
    fruit:'apple',
    brand:'apple'
});

// On an array
// Provide key/value as an object
Observer.set(arr, {
    0:'apple',
    3:'apple'
});
```

### Passing a Value to Observers

The `params.detail` property can be used to pass a value specifically to observers that might be responding to the `set` event. Any type of value can be passed.

```js
Observer.set(obj, propertyName, value, {
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

When a *set* operation fires an event, event handlers recieve a special object, called the *Response Object*, in addition to the standard event object. The Response Object can be used to, either *halt*, or *keep in sync* with what happens next.

**response.stopPropagation()** cancels the event, that is, prevents the event from reaching other event handlers. Returning `false` from the handler has the same effect. 

```js
Observer.observe(obj, propertyName, (event, response) => {
    response.stopPropagation();
    // Or, return false;
});
```

The initiator of the *set* operation has to flag the event as *cancellable* for the above to be honoured. It may also obtain the response object to determine the state of this response.

```js
let response = Observer.set(obj, propertyName, value, {
    cancellable: true,
    responseObject: true,
});
// Determine response state...
if (response.propagationStopped) {
}
```

**response.preventDefault()** tells the initiator of the *set* operation to skip the *default action* that it takes, if any, after *set* operations. Returning `false` from the handler has the same effect. (The event still reaches other handlers.)

```js
Observer.observe(obj, propertyName, (event, response) => {
    response.preventDefault();
    // Or, return false;
});
```

The initiator of the *set* operation may obtain the response object to determine the state of this response.

```js
let response = Observer.set(obj, propertyName, value, {
    responseObject: true,
});
// Determine response state...
if (response.defaultPrevented) {
}
```

**response.waitUntil(promise)** tells the initiator of the *set* operation to wait until a *Promise* is resolved before continuing with further operations. Returning a `Promise` from the handler has the same effect. (The event still reaches other handlers without waiting.)

```js
Observer.observe(obj, propertyName, (event, response) => {
    let promise = new Promise(resolve => {
        setTimeout(resolve, 2000);
    });
    response.waitUntil(promise);
    // Or, return promise;
});
```

The initiator of the *set* operation may obtain the response object to determine the state of this response. The state of this response becomes a promise when one or more handlers return a promise.

```js
let response = Observer.set(obj, propertyName, value, {
    responseObject: true,
});
// Determine response state...
if (response.promises) {
    response.promises.then(() => {
    });
}
```

## Usage as a Trap's "set" Handler

`Observer.set()` returns a *Boolean* value for *set* operations and can, therefore, be used as the "set" handler in Proxy traps.

```js
let _obj = new Proxy(obj, {set: Observer.set});
let _arr = new Proxy(arr, {set: Observer.set});
```

Assignment operations will now be forwarded to `Observer.set()` and [*observers*](../observe) and [*interceptors*](../intercept) that may be bound to the object will continue to respond.

```js
_obj.fruit = 'apple';
_arr[2] = 'Item 3';
```

## Usage with Property Setters

It is possible to implement *property setters* that use `Observer.set()` behind the scene. This gives us the benefit of using JavaScript's assignment syntax while still driving [*observers*](../observe) and [*interceptors*](../intercept).

This is automatically done by the [`Observer.init()`](../init) support function.

```js
// Virtualize a property or multiple properties
Observer.init(obj, 'fruit');
Observer.init(obj, ['fruit', 'brand']);

// Now we can do without Observer.set
obj.fruit = 'apple';
obj.brand = 'apple';
```

We could follow the pattern above for arrays; we could even *init* an array's prototype methods instead. The specific keys modified after calling these methods will be announced to [*observers*](../observe).

```js
// Virtualize the arr.push() and arr.splice() methods
Observer.init(arr, ['push', 'splice']);

// Now we can do without Observer.set
arr.push('Item 1');
arr.push('Item 2');
arr.splice(1);
```

## Intercepting `Observer.set()`

Using [`Observer.intercept()`](../intercept), it is possible to intercept calls to `Observer.set()`. When a "set" operation triggers an interceptor, the interceptor will receive an event object containing the property name and the assignable value.

```js
Observer.intercept(obj, 'set', (event, recieved, next) => {
    // What we recieved...
    console.log(event.name, event.value);
    // The assignment operation
    obj[event.name] = event.value;
    // The return value - Boolean
    return true;
});

Observer.set(obj, 'fruit', 'orange');
```

When the "set" operation is of multiple key/value assignments, the interceptor gets fired for each pair while also recieving the total list of properties as a hint - via `event.related`.

```js
Observer.intercept(obj, 'set', (event, recieved, next) => {
    // What we recieved...
    console.log(event.name, event.value, event.related);
    // The assignment operation
    obj[event.name] = event.value;
    // The return value - Boolean
    return true;
});

Observer.set(obj, {fruit: 'orange', brand:'apple'});
```

The above should trigger our interceptor twice with `event.related` being `['fruit', 'brand']`.

The interceptor is expected to return *true* when the operation is successful; *false* otherwise.

## Related Methods

+ [`Observer.observe()`](../observe)
+ [`Observer.intercept()`](../intercept)
