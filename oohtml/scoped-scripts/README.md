# Scoped Scripts
Scoped Scripts is a new feature that lets us write `<script>` elements that are scoped to their immediate host elements instead of the global browser scope. Scoped scripts make it easier to apply behaviour to modular markup as they execute in the context of their host elements.

We could also use them as a powerful abstraction over the [Observer API](../the-observer-api/README.md) to automatically keep the UI in sync with the state of an application.

> OOHTML is [being proposed as a native browser technology](https://discourse.wicg.io/t/proposal-chtml/4716) but currently available through a polyfill. Be sure to check the [Polyfill Support](#polyfill-support) section below for the features on this page.

## Scoped Scripts
Scoped scripts are defined with the `scoped` *Boolean* attribute.

```html
<div id="alert">

    <script scoped>
        // ...
    </script>

</div>
```

They have their `this` variable implicitly bound to their host element.

```html
<div id="alert">

    <script scoped>
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

    <script scoped>
        let message = alertMessage; // Task failed!
    </script>

</div>
```

They keep their variables from leaking out to the global scope.

```html
<div id="alert">

    <script scoped>
        let message = 'Task complete!';
    </script>

</div>

<script>
    // Global scope
    console.log(typeof message); // undefined
</script>
```

This lets us place behaviours of any form just where we need them!

```html
<script>
    // Global variable
    let alertMessage = 'Task failed!';
    // The element's state property
    document.querySelector('#alert').state.message = 'Task complete!';
</script>

<div id="alert">

    <script scoped>
        let message = this.state.message || alertMessage;
        console.log(message);
    </script>

</div>
```

The code above is an idea of a [stateful](../the-state-api/README.md) `#alert` component. And we could make that functional, e.g. give it a *remove* feature.

```html
<div id="alert">

    <div class="message"></div>
    <div class="close" title="Close this message.">X</div>

    <script scoped>
        let message = this.state.message || alertMessage;
        this.querySelector('.message').innerHTML = message;
        // details of how the #alert block should behave...
        this.querySelector('.close').addEventListener('click', () => {
            this.remove();
        });
    </script>

</div>
```

## Binding Scripts
Binding Scripts are a type of script elements that are automatically bound to the *live objects* that are referenced in their statements. When changes are detected on any of these objects, the specific statements that depend on them are re-run.

Binding scripts are defined with the `binding` *Boolean* attribute. And they could be used with the `scoped` attribute.

```html
<div id="alert">

    <div class="message"></div>
    <div class="exit" title="Close this message.">X</div>

    <script scoped binding>
        let message = this.state.message || alertMessage;
        this.querySelector('.message').innerHTML = message;
        this.querySelector('.close').addEventListener('click', () => {
            this.remove();
        });
    </script>

</div>
```

With the code above, updating the element's `.state.message` property will trigger a re-run of the statements that depend on it, leading to the new message being rendered.

```html
<script>
    document.querySelector('#alert').state.message = 'Task restarted!';
</script>
```

The new message gets rendered because the first statement in the script has a reference to the updated property as `this.state.message`. The result of this statement is, in turn, referenced by the second statement, getting it to re-run, thus, getting the message rendered. The third statement is left untouched as it does not depend on the updated object.

The above shows that the dependency chain within a Binding Script is followed even when broken into local variables.

### The Asynchronous Runtime and Observability
Immediately a Binding Script is parsed, every statement is run sequentially from top to bottom - as with a regular script. From this point forward, statements are kept in sync with the *live objects* they depend on where they potentially get to be called again independently on a change event. This event-based, independent runtime is called the *asynchronous runtime*.

The asynchronous runtime uses the [Observer API](../the-observer-api/README.md) to observe live, *object-like* variables for property changes. A variable is object-like when it is an object or when it works like an object, e.g arrays. A live object is any of such objects whose properties can be observed.

By default, the `this` object is observed from within an element's binding script; the `document` object is, too. Thus, any of their properties will trigger a statement in a scoped script if the property was set or removed using the Observer API.

```html
<div>

    <script scoped binding>
        console.log('Global observableProperty:', document.observableProperty);
        console.log('Local observableProperty:', this.observableProperty);
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

But, as seen in the `#alert` example above, we can more easily set or remove observable properties using an element's *state object* (or the document's *state object*) as it internally uses the Observer API to apply property assignments or mutations.

```html
<div>

    <script scoped binding>
        console.log('Global state.observableProperty:', document.state.observableProperty);
        console.log('Local state.observableProperty:', this.state.observableProperty);
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

In any of the cases above, we could get deep object mutations to be observed from within a binding script using the Observer API.

```html
<div>

    <script scoped binding>
        console.log('Global clock.time:', document.clock.time);
        console.log('Local state.clock.time:', this.state.clock.time);
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
While the `this` object and the `document` object are automatically observed from within an element's binding script, it is also possible to bind other objects to the script. This is done using a `.bind()` method on the element - to bind locally, or on the `document` object - to bind globally for all scripts in the document.

```html
<div>

    <script scoped binding>
        console.log('Globally-bound clock time:', globallyBoundClock.time);
        console.log('Locally-bound clock time:', locallyBoundClock.time);
    </script>

</div>

<script>
    let globallyBoundClock = {time: '00:00:00',};
    document.bind({globallyBoundClock});
    setInterval(() => {
        Observer.set(globallyBoundClock, 'time', (new Date).toLocaleString());
    }, 100 * 60);

    let divElement = document.querySelector('div');
    let locallyBoundClock = {time: '00:00:00',};
    divElement.bind({locallyBoundClock});
    setInterval(() => {
        Observer.set(locallyBoundClock, 'time', (new Date).toLocaleString());
    }, 100);

</script>
```

### The Binding Scripts API
The following methods are used to dynamically bind object to *binding* scripts.

+ **document.bind(bindings[, params]): Void** - This method lets us bind objects at the document-level. Objects bound here are automatically-observed in *binding* scripts across the document.

    **Parameters:**
    + `bindings: Object` - The object to bind globally for all *binding* scripts in the document.
    + `params: Object` - (Optional) Binding options:
        + `update: Boolean` - Specifies whether this binding should update previous binding or undo it. Default: `false` - undo previous binding.

    ```js
    // Undo previous binding, if exists
    document.bind({
        globallyBoundClock: {time: '00:00:00',};
    });

    // ----------

    // Update previous binding, if exists
    document.bind({
        globallyBoundClock2: {time: '00:00:00',};
    }, {update: true});
    ```

+ **document.unbind(): Void** - This method lets us unbind any existing binding from the document.

    ```js
    // Unbind existing binding
    document.unbind();
    ```
    
+ **Element.prototype.bind(bindings[, params]): Void** - This method lets us bind objects at the element-level. Objects bound here are automatically-observed in the element's *binding* script.

    **Parameters:**
    + `bindings: Object` - The object to bind to the element's *binding* script.
    + `params: Object` - (Optional) Binding options:
        + `update: Boolean` - Specifies whether this binding should update previous binding or undo it. Default: `false` - undo previous binding.

    ```js
    // Undo previous binding, if exists
    divElement.bind({
        locallyBoundClock: {time: '00:00:00',};
    });

    // ----------

    // Update previous binding, if exists
    divElement.bind({
        locallyBoundClock: {time: '00:00:00',};
    }, {update: true});
    ```

+ **Element.prototype.unbind(): Void** - This method lets us unbind any existing binding from an element's *binding* script.

    ```js
    // Unbind existing binding
    divElement.unbind();
    ```

## Error Handling
Scoped Scripts features a way to handle errors that may occur. By default, script errors are logged to the console. But they can be silently ignored by setting a `script-errors` directive on the [OOHTML meta tag](../the-oohtml-meta-tag/README.md).

```html
<html>
    <head>
        <meta name="chtml" content="script-errors=0;" />
    </head>
    <body>
        <h1></h1>
        <script scoped binding>
            this.querySelectorSelectorSelector('h1').innerHTML = headline;
        </script>
    </body>
</html>
```

Individual script tags may also be given an `errors` directive, to override the global `script-errors` directive for the script.

```html
<html>
    <head>
        <meta name="chtml" content="script-errors=0;" />
    </head>
    <body>
        <h1></h1>
        <script scoped binding errors="1">
            this.querySelectorSelectorSelector('h1').innerHTML = headline;
        </script>
    </body>
</html>
```

## Polyfill Support
The current [OOHTML polyfill implementation](../installation/README.md) has good support for Scope Scripts. But while the [proposed standard](#scoped-scripts) uses the `scoped` *Boolean* attribute to designate a *scoped* script, this implementation uses the `type` attribute.

```html
<div>

    <script type="scoped"></script>
    <script type="scoped" binding></script>

</div>
```

The polyfill additionally makes it possible to customise much of its implementation of the syntax using the [OOHTML meta tag](../the-oohtml-meta-tag/README.md). The following are areas of customization:

+ **[selector.script](#scoped-scripts)** - The CSS selector for matching the script element. The default selector is `script[type="scoped"]`. You may use a custom selector, like `script[is="my-script"][type="scoped"]`, where necessary.
        
    ```html
    <head>
        <meta name="oohtml" content="selector.script=script[is='my-script'][type='scoped'];" />
    </head>
    <body>
        <div>

            <script is="my-script" type="scoped"></script>
            <script is="my-script" type="scoped" binding></script>

        </div>
    </body>
    ```

+ **[api.bind](#the-binding-scripts-api)** - The *method name* for binding objects to scripts. The standard *method name* is `bind`, but you may use a custom method name, where necessary.
        
    ```html
    <head>
        <meta name="oohtml" content="api.bind=setBinding;" />
    </head>
    ```
    
    ```js
    document.setBinding(binding);
    ```

+ **[api.unbind](#the-binding-scripts-api)** - The *method name* for unbinding objects from scripts. The standard *method name* is `unbind`, but you may use a custom method name, where necessary.
        
    ```html
    <head>
        <meta name="oohtml" content="api.bind=clearBinding;" />
    </head>
    ```
    
    ```js
    document.clearBinding();
    ```

Learn more about customization and the OOHTML meta tag [here](../the-oohtml-meta-tag/README.md).