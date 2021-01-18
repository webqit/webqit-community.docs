# `Observer.has()`

This method is used to test property presence. It corresponds to the JavaScript's built-in `Reflect.has()` function, which is itself the programmatic alternative to the JavaScript *in* operator – `property in obj`.

`Observer.has()` brings the added benefit of triggering [*interceptors*](../intercept).

+ [Syntax](#syntax)
+ [Usage](#usage)
+ [Usage as a Trap's "has" Handler](#usage-as-a-traps-has-handler)
+ [Intercepting `Observer.has()`](#Intercepting-observer.has)
+ [Related Methods](#related-methods)

## Syntax

```js
// Test the presence of a property.
let exists = Observer.has(obj, propertyName);
```

**Parameters**

+ `obj:             Object|Array` - an object or array.
+ `propertyName:    String` - the property to test.

**Return Value**

*Boolean*

## Usage

```js
let obj = {
    fruit:'orange',
    brand:'apple',
};

let exists = Observer.has(obj, 'fruit');
```

## Usage as a Trap's "has" Handler

`Observer.has()` can be used as the "has" handler in Proxy traps.

```js
let _obj = new Proxy(obj, {has: Observer.has});
let _arr = new Proxy(arr, {has: Observer.has});
```

*Exists* operations will now be forwarded to `Observer.has()` and [*interceptors*](../intercept) that may be bound to the object will continue to respond.

```js
let exists = 'fruit' in _obj;
let exists = '1' in _arr;
```

## Intercepting `Observer.has()`

Using [`Observer.intercept()`](../intercept), it is possible to intercept calls to `Observer.has()`. When a "has" operation triggers an interceptor, the interceptor will receive an event object containing the property name to check.

```js
Observer.intercept(obj, 'has', (event, recieved, next) => {
    // What we recieved...
    console.log(event.name);
    // The read operation
    return event.name in obj;
});

let exists = Observer.has(obj, 'fruit');
```

## Related Methods

+ [`Observer.observe()`](../observe)
+ [`Observer.intercept()`](../intercept)
