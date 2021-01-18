# `Observer.observe()`

This method is used to observe changes to an object or array.

+ [Syntax](#syntax)
+ [Usage](#usage)
+ [Specifying an Event Type](#specifying-an-event-type)
+ [Tagging an Observer](#tagging-an-observer)
+ [The Returned Observer Instance](#the-returned-observer-instance)
+ [Related Methods](#related-methods)

## Syntax

```js
// Observe all properties
Observer.observe(obj, callback[, params = {}]);

// Observe a specific path
Observer.observe(obj, path, callback[, params = {}]);

// Observe a list of paths
Observer.observe(obj, paths, callback[, params = {}]);
```

**Parameters**

+ `obj:         Object|Array` - an object or array.
+ `path:        String|Array` - a path to observe.
+ `paths:       Array` - the list of paths to observe.
+ `callback:    Function` - a callback function that receives the change notifications. This recieves:
    + `changes/change` - an *event* or a list of *events* (in the case of the last syntax above.)
+ `params:      Object` - Additional parameters.

**Return Value**

An [*Observer* instance](#the-returned-observer-instance).

## Usage

+ [Observing All Properties](#observing-all-properties)
+ [Observing a Path](#observing-a-path)
+ [Observing a List of Paths](#observing-a-list-of-paths)

### Observing All Properties

```js
let callback = events => {
    events.forEach(event => {
        console.log(event.type, event.name, event.path, event.value, event.oldValue);
    });
};
let obj = {};
Observer.observe(obj, callback);
```

The code above will report changes as the *object* gets modified on any of its properties.

```js
let arr = [];
Observer.observe(arr, callback);
```

The code above will report changes as the *array* gets modified with entries.

```js
Observer.set(obj, 'fruit', 'apple');
```

With the *set* operation above, the value of `event.name` and `event.path` in the console will be `fruit` and `[fruit]` respectively.

#### Using the `params.sutree` Parameter

To observe changes on nested objects or arrays, we would use *the `params.sutree` parameter*.

```js
let obj = {
    preferences: {},
};
Observer.observe(obj, events => {
    events.forEach(event => {
        console.log(event.type, event.name, event.path, event.value, event.oldValue);
    });
}, {subtree:true});
```

The code above will report changes happening down the object tree.

```js
Observer.set(obj.preferences, 'fruit', 'apple');
```

With the *set* operation at level 2 above, the value of `event.name` and `event.path` in the console will be `preferences` and `['preferences', 'fruit']` respectively.

### Observing a Path

It is possible to observe a specific path on an object or array.

```js
let callback = event => {
    console.log(event.type, event.name, event.path, event.value, event.oldValue);
};

let obj = {};
Observer.observe(obj, 'fruit', callback);
```

The code above will report changes as the "fruit" property gets created, modified or deleted on the object.

```js
let arr = [];
Observer.observe(arr, 0, callback);
```

The code above will report changes as the array's first entry gets created, modified or deleted.

```js
Observer.set(obj, 'fruit', 'apple');
```

With the *set* operation above, the value of `event.name` and `event.path` in the console will be `fruit` and `['fruit']` respectively.

#### Using Array Path Expressions

Array path expressions are used to observe changes on nested objects or arrays.

```js
let obj = {
    preferences: {},
};
Observer.observe(obj, ['preferences', 'fruit'], event => {
    console.log(event.type, event.name, event.path, event.value, event.oldValue);
});
```

The code above will report changes that occur at any level along the specified path - `['preferences', 'fruit']`.

```js
Observer.set(obj.preferences, 'fruit', 'orange');
```

With the *set* operation at level 2 above, the value of `event.name` and `event.path` in the console will be `preferences` and `['preferences', 'fruit']` respectively.

```js
Observer.set(obj.preferences, 'brand', 'apple');
```

With the *set* operation at level 2 above, the value of `event.name` and `event.path` will be `preferences` and `['preferences', 'brand']` respectively. This time, the change isn't happening along the path we're observing. And we don't see anything in the console.

### Observing a List of Paths

It is possible to observe multiple paths at once by giving an array of path arrays. The observer is called with the paths along which an event happened.

```js
let callback = events => {
    events.forEach(event => {
        console.log(event.type, event.name, event.path, event.value, event.oldValue);
    });
};

let obj = {};
Observer.observe(obj, [ ['fruit'], ['brand'] ], callback);
```

The code above will report changes as any of "fruit" or "brand" properties get created, modified or deleted on the object.

```js
let arr = [];
Observer.observe(arr, [ [0], [3] ], callback);
```

The code above will report changes as the array's first or fourth entry gets created, modified or deleted.

```js
Observer.set(obj, 'fruit', 'orange');
```

If we made multiple changes in one batch, our observer would recieve multiple events in one call.

```js
Observer.set(obj, {
    fruit: 'orange',
    brand: 'apple',
});
```

#### Using Wildcard Paths

It is possible to observe multiple paths dynamically using one *wildcard path* expression. Wildcard paths are paths with empty slots, designed to match *event paths* as they happen.

Notice below that our path is a 3-level path, with the second level being the wildcard.

```js
let obj = {
    preferences: {
        favourites: {},
    },
};
Observer.observe(obj, ['preferences', null, 'fruit'], events => {
    events.forEach(event => {
        console.log(event.type, event.name, event.path, event.value, event.oldValue);
    });
});
```

The code above will report changes when an event fires at a path that fulfills our wildcard path. A path like `['preferences', 'favourites', 'fruit']` would do that.

```js
Observer.set(obj.preferences.favourites, 'fruit', 'mango');
```

If we fired multiple matching events in one batch, our observer would recieve multiple events in one call.

```js
Observer.set(obj.preferences.favourites, {
    fruit: 'orange',
    brand: 'apple',
});
```

## Specifying an Event Type

The `params.type` parameter can be used to tell an observer to respond to a specific mutation type like "set" and "del".

```js
Observer.observe(obj, path, callback, {type:'del'});
```

## Tagging an Observer

The `params.tags` parameter can be used to tag an observer. Tags are an *array* of values (*strings*, *numbers*, *objects*, etc) that can be used to identify the observer for later use.

```js
Observer.observe(obj, path, callback, {tags:['#tag']});
```

## The Returned Observer Instance

The `Observer.observe()` method returns an *Observer* instance that gives us per-instance control.

```js
// Obtain the Observer instance
let instance = Observer.observe(obj, callback);

// Synthetically fire the observer
instance.fire({
    type:'customType',
});

// Disconnect the observer
instance.disconnect();
```

## Related Methods

+ [`Observer.unobserve()`](../unobserve)
+ [`Observer.set()`](../set)
+ [`Observer.deleteProperty()`](../deleteproperty)
