---
_after: the-state-api
---
# Subscript
Subscript is a type of JavaScript runtime that lets us create scoped, *reactive* `<script>` elements across the UI. That gives us a UI binding language and the ability to have UI logic without involving an actual JavaScript file.

> OOHTML is [being proposed as a native browser technology](https://discourse.wicg.io/t/proposal-chtml/4716) while currently available through a polyfill. Be sure to check the [Polyfill Support](#polyfill-support) section below for the features on this page.

## Convention
Subscript elements are defined with the `subscript` *MIME* type. They are scoped to their immediate host element instead of the global browser scope.

```html
<div id="alert">

    <script type="subscript">
        // ...
    </script>

</div>
```

They have their `this` variable implicitly bound to their host element.

```html
<div id="alert">

    <script type="subscript">
        let id = this.id; // alert
    </script>

</div>
```

They have their other variables resolved from the global scope.

```html
<script>
    // Global scope
    let alertMessage = 'Task failed!';
</script>

<div id="alert">

    <script type="subscript">
        let message = alertMessage; // Task failed!
    </script>

</div>
```

They keep their variables from leaking out to the global scope.

```html
<body>
        
    <div id="alert">

        <script type="subscript">
            let message = 'Task complete!';
        </script>

    </div>

    <script>
        // Global scope
        console.log(typeof message); // undefined
    </script>

</body>
```

This lets us place behaviours of any form just where across the page we need them!

```html
<body>

    <script>
        // Global variable
        let alertMessage = 'Task failed!';
        // The element's state property
        document.querySelector('#alert').state.message = 'Task complete!';
    </script>

    <div id="alert">

        <script type="subscript">
            let message = this.state.message || alertMessage;
            console.log(message);
        </script>

    </div>
    
</body>
```

Now, that was a bare-bones `#alert` component above! We could make it quite interactive by giving it a *remove* feature.

```html
<div id="alert">

    <div class="message"></div>
    <div class="close" title="Close this message.">X</div>

    <script type="subscript">
        let message = this.state.message;
        this.querySelector('.message').innerHTML = message;
        this.querySelector('.close').addEventListener('click', () => {
            this.remove();
        });
    </script>

</div>
```

## Runtime
Subscript is drastically different in behaviour from other JavaScript types (type="module", type="text/javascript", etc). The difference is that the script has the ability to observe the variables in its scope and respond to those changes. Changes that fire up the script this way are called *events*, and a script's response to these events is called the *event-based runtime*.

The event-based runtime can be understood from the code below. Take note of the first statement in the script which makes a reference to the `#alert` element's `.state.message` property.

```html
<div id="alert" namespace>

    <div id="message"></div>
    <div id="close" title="Close this message.">X</div>

    <script type="subscript">
        let message = this.state.message;
        this.namespace.message.innerHTML = message;
        this.namespace.close.addEventListener('click', () => {
            this.remove();
        });
    </script>

</div>
```

This script will, at first, run top-down as with standard JavaScript. Then, it will begin to observe changes to the `this.state.message` reference - being an observable property. And when a change is detected, that particular statement will be reevaluated, and the new value is (re)assigned to the local `message` variable.

The following update to the element's state property will trigger that *event*.

```html
<script>
    document.querySelector('#alert').state.message = 'Task restarted!';
</script>
```

Now, the same event that changed the script's local `message` variable will go further to fire up subsequent statements that depend on it, in this case, leading to the new message being rendered. The third statement in this script is left untouched as it does not depend on the current change.

Thus, when events happen, the dependency chain within the script is followed even when broken down into local variables.

### Observability
The event-based runtime uses the [Observer API](../the-observer-api) to observe objects in its scope whose properties can be observed. These are called *live objects*.

By default, the `this` object and the `document` object are observed. Thus, setting, updating or removing any of their properties using the Observer API will trigger the appropriate statement in a Subscript runtime.

```html
<div>

    <script type="subscript">
        console.log('Global observableProperty:', document.observableProperty);
        console.log('Own observableProperty:', this.observableProperty);
    </script>

</div>

<script>
    let counter = 0;
    setInterval(() => {
        Observer.set(document, 'observableProperty', counter ++);
    }, 8000);

    let counter2 = 0;
    let divElement = document.querySelector('div');
    setInterval(() => {
        Observer.set(divElement, 'observableProperty', counter2 ++);
    }, 4000);

</script>
```

But, as seen in the `#alert` example above, we can more easily set or remove observable properties using an element's *state object* (or the document's *state object*) as it internally uses the Observer API to apply property assignments and deletions.

```html
<div>

    <script type="subscript">
        console.log('Global state.observableProperty:', document.state.observableProperty);
        console.log('Own state.observableProperty:', this.state.observableProperty);
    </script>

</div>

<script>
    let counter = 0;
    setInterval(() => {
        document.state.observableProperty = counter ++;
    }, 8000);

    let counter2 = 0;
    let divElement = document.querySelector('div');
    setInterval(() => {
        divElement.state.observableProperty = counter2 ++;
    }, 4000);

</script>
```

In any of the cases above, we could get deep object mutations to be caught by the Subscript runtime using the Observer API.

```html
<div>

    <script type="subscript">
        console.log('Global clock.time:', document.clock.time);
        console.log('Own state.clock.time:', this.state.clock.time);
    </script>

</div>

<script>
    // Initial binding
    Observer.set(document, 'clock', {});
    setInterval(() => {
        Observer.set(document.clock, 'time', (new Date).toLocaleString());
    }, 100 * 60);

    let divElement = document.querySelector('div');
    setInterval(() => {
        Observer.set(divElement.state.clock, 'time', (new Date).toLocaleString());
    }, 100);

</script>
```

### Bindings
While the `this` object and the `document` object are automatically observed from within an element's Subscript runtime, it is also possible to bind other objects to the script's scope. This is done using a `.bind()` method on the element - to bind locally, or on the `document` object - to bind globally for all scripts in the document.

```html
<div>

    <script type="subscript">
        console.log('Globally-bound clock time:', globallyBoundClock.time);
        console.log('Own-bound clock time:', locallyBoundClock.time);
    </script>

</div>

<script>
    // Create a collection of variables
    let globallyBoundClock = {time: '00:00:00',};
    // Bind them to all Subscript scopes in the document
    document.subscript.bind({globallyBoundClock});
    setInterval(() => {
        // Update existing binding
        Observer.set(globallyBoundClock, 'time', (new Date).toLocaleString());
        // Create new ones on the fly anytime
    }, 100 * 60);

    let divElement = document.querySelector('div');
    // Create a collection of variables
    let locallyBoundClock = {time: '00:00:00',};
    // Bind them to the current script
    divElement.subscript.bind({locallyBoundClock});
    setInterval(() => {
        // Update existing binding for the current script
        Observer.set(locallyBoundClock, 'time', (new Date).toLocaleString());
        // Create new ones on the fly anytime
    }, 100);

</script>
```

#### API
The following methods are used to dynamically bind observable variables to Subscript scopes.

+ **document.subscript.bind(bindings[, params]): Void** - This method lets us bind objects at the document-level for all Subscript scopes across the document.

    **Parameters:**
    + `bindings: Object` - The object to bind globally for all Subscript scopes in the document.
    + `params: Object` - (Optional) Binding options:
        + `update: Boolean` - Specifies whether to simply update existing variables in Subscript's global scope from properties of the given object or establish the given object as Subscript new global scope. Default: `false` - establish as new global scope.

    ```js
    // Undo previous binding, if exists
    document.subscript.bind({
        globallyBoundClock: {time: '00:00:00',};
    });

    // ----------

    // Update previous binding, if exists
    document.subscript.bind({
        globallyBoundClock2: {time: '00:00:00',};
    }, {update: true});
    ```

+ **document.subscript.unbind(): Void** - This method lets us unbind any existing binding from the document.

    ```js
    // Unbind existing binding
    document.subscript.unbind();
    ```
    
+ **Element.prototype.subscript.bind(bindings[, params]): Void** - This method lets us bind objects at the element-level. Objects bound here are automatically-observed in the element's *binding* script.

    **Parameters:**
    + `bindings: Object` - The object to bind to the element's Subscript scope.
    + `params: Object` - (Optional) Binding options:
        + `update: Boolean` - Specifies whether to simply update existing variables in the element's local scope from properties of the given object or establish the given object as the element's local scope. Default: `false` - establish as new scope.

    ```js
    // Undo previous binding, if exists
    divElement.subscript.bind({
        locallyBoundClock: {time: '00:00:00',};
    });

    // ----------

    // Update previous binding, if exists
    divElement.subscript.bind({
        locallyBoundClock: {time: '00:00:00',};
    }, {update: true});
    ```

+ **Element.prototype.subscript.unbind(): Void** - This method lets us unbind any existing binding from an element's *binding* script.

    ```js
    // Unbind existing binding
    divElement.subscript.unbind();
    ```

## Error Handling
Subscript features a way to handle errors that may occur within scripts. By default, script errors are logged to the console. But they can be silently ignored by setting a `script.errors` directive on the [OOHTML META tag](../the-oohtml-meta-tag).

```html
<html>
    <head>
        <meta name="chtml" content="script.errors=0;" />
    </head>
    <body>
        <h1></h1>
        <script type="subscript">
            this.querySelectorSelectorSelector('h1').innerHTML = headline;
        </script>
    </body>
</html>
```

Individual script tags may also be given an `errors` directive, to override the global `script.errors` directive for the script.

```html
<html>
    <head>
        <meta name="chtml" content="script.errors=0;" />
    </head>
    <body>
        <h1></h1>
        <script type="subscript" binding errors="1">
            this.querySelectorSelectorSelector('h1').innerHTML = headline;
        </script>
    </body>
</html>
```

## Polyfill Support
The current [OOHTML polyfill implementation](../polyfill) has good support for Subscript. The polyfill additionally makes it possible to customise the follwoing areas of its implementation of the syntax using the [OOHTML META tag](../the-oohtml-meta-tag):

+ **[selector.script](#convention)** - The CSS selector for matching the script element. The default selector is `script[type="subscript"]`. You may use a custom selector, like `script[is="my-script"][type="subscript"]`, where necessary.
        
    ```html
    <head>
        <meta name="oohtml" content="selector.script=script[is='my-script'][type='reflex'];" />
    </head>
    <body>
        <div>

            <script is="my-script" type="subscript"></script>
            <script is="my-script" type="subscript"></script>

        </div>
    </body>
    ```

Learn more about customization and the OOHTML meta tag [here](../the-oohtml-meta-tag).
