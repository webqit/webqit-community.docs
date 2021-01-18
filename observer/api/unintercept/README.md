# `Observer.unintercept()`

This method is used to unbind interceptors previously bound with [`Observer.intercept()`](/interceptor/api/intercept).

+ [Syntax](#syntax)
+ [Related Methods](#related-methods)

## Syntax

```js
// Unbind all interceptors bound to the following property name
// regardless of the handler function
Observer.unintercept(obj, type);

// Unbind the interceptor bound with the following handler function
Observer.unintercept(obj, type, originalHandler);

// Unbind the interceptor bound with the following handler function and tags
Observer.unintercept(obj, type, originalHandler, {tags:[...originalTags]});

// Unbind all interceptors bound with the following tags
// regardless of the handler function
Observer.unintercept(obj, type, null, {tags:[...originalTags]});
```

**Parameters**

+ `obj:             Object|Array` - an object or array.
+ `type:            String` - if not `null`, the operation type used on [`Observer.intercept()`](/interceptor/api/intercept)
+ `originalHandler: Function` - if not `null`, the *original* callback function used on [`Observer.intercept()`](/interceptor/api/intercept)
+ `params.tags:     Array` - if not `null`, the list of *tags* (in any order) used on [`Observer.intercept()`](/interceptor/api/intercept)


**Return Value**

*undefined*

## Related Methods

+ [`Observer.intercept()`](/interceptor/api/intercept)
