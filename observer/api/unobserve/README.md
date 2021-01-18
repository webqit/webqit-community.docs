# `Observer.unobserve()`

This method is used to unbind observers previously bound with [`Observer.observe()`](../observe).

+ [Syntax](#syntax)
+ [Related Methods](#related-methods)

## Syntax

```js
// Unbind all observers bound to the following property name
// regardless of the handler function
Observer.unobserve(obj, path);

// Unbind the observer bound with the following handler function
Observer.unobserve(obj, path, originalCallback);

// Unbind the observer bound with the following handler function and tags
Observer.unobserve(obj, path, originalCallback, {tags:[...originalTags]});

// Unbind the observer bound with the following handler function and reflex type 
Observer.unobserve(obj, path, originalCallback, {type:’set’});

// Unbind all observers bound with the following tags
// regardless of the handler function
Observer.unobserve(obj, paths, null, {tags:[...originalTags]});
```

**Parameters**

+ `obj:             Object|Array` - an object or array.
+ `path:            String|Array` - if not `null`, a path to unobserve.
+ `paths:           Array` - if not `null`, the list of paths (in any order) to unobserve.
+ `originalCallback: Function` - if not `null`, the *original* callback function used during [`observe()`](../observe)
+ `params.tags:     Array` - if not `null`, the list of the *original* tags (in any order) used during [`observe()`](../observe)


**Return Value**

*undefined*

## Related Methods

+ [`Observer.observe()`](../observe)
