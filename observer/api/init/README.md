# `Observer.init()`

This function is used to initialize property *setters* and *getters* that use `Observer.set()` and `Observer.get()` respectively behind the scene. This gives us the benefit of using JavaScript's assignment and accessor syntax while still driving [*observers*](../observe) and [*interceptors*](../intercept).

+ [Syntax](#syntax)
+ [Usage](#usage)

## Syntax

```js
// Init a single property
Observer.init(obj, propertyName);

// Init multiple properties
Observer.init(obj, propertyNames);
```

**Parameters**

+ `obj:             Object|Array` - an object or array.
+ `propertyName:    String` - the property to *initialize*.
+ `propertyNames:   Array` - a list of properties to *initialize*.

**Return Value**

*undefined*

## Usage

```js
// The object
let obj = {};

// We observe the 'preferences' property
Observer.observe(obj, 'preferences', changes => {
    console.log(changes);
});

// Now we virtualize this property
Observer.init(obj, 'preferences');

// We use the property and watch our console.
obj.preferences = {};
```