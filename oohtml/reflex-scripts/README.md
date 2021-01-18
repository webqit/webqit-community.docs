# Reflex
Reflex is a new type of `<script>` elements that works as a *data binding* language for the UI. They are *scoped* to their immediate host elements instead of the global browser scope. These two important features make it all interesting to apply behaviour to modular markup, that is, give us the ability to have some little logic without involving an actual JavaScript file.

> OOHTML is [being proposed as a native browser technology](https://discourse.wicg.io/t/proposal-chtml/4716) while currently available through a polyfill. Be sure to check the [Polyfill Support](#polyfill-support) section below for the features on this page.

## Convention
Reflex Scripts are defined with the `reflex` *MIME* type.

```html
<div id="alert">

    <script type="reflex">
        // ...
    </script>

</div>
```

They have their `this` variable implicitly bound to their host element.

```html
<div id="alert">

    <script type="reflex">
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

    <script type="reflex">
        let message = alertMessage; // Task failed!
    </script>

</div>
```

They keep their variables from leaking out to the global scope.

```html
<body>
        
    <div id="alert">

        <script type="reflex">
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

        <script type="reflex">
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

    <script type="reflex">
        let message = this.state.message;
        this.querySelector('.message').innerHTML = message;
        this.querySelector('.close').addEventListener('click', () => {
            this.remove();
        });
    </script>

</div>
```

## Reflex Actions
Reflexive scripts are drastically different in behaviour from other JavaScript types (type="module", type="text/javascript", etc). The difference is that the script has the ability to observe the variables in its scoped and respond to those changes. Changes that fire up the script this way are called *Reflex Actionss*.

Reflex actions and a script's reponse to them can be understood from the code below. Take note of the first statement in the script which makes a reference to the `#alert` element's `.state.message` property.

```html
<div id="alert">

    <div class="message"></div>
    <div class="close" title="Close this message.">X</div>

    <script type="reflex">
        let message = this.state.message;
        this.querySelector('.message').innerHTML = message;
        this.querySelector('.close').addEventListener('click', () => {
            this.remove();
        });
    </script>

</div>
```

This script will, at first, run top-down as with standard JavaScript. Then, it will begin to observe changes to the `this.state.message` reference - being an observable property. And when a change is detected, that particular statement will be reevaluated, and the local `message` variable will take on the new value.

The following update to the element's state property will trigger that *Reflex Action*.

```html
<script>
    document.querySelector('#alert').state.message = 'Task restarted!';
</script>
```

Now, the same *Reflex Actions* that changed the script's local `message` variable will go further to fire up subsequent statements that depend on it, in this case, leading to the new message being rendered. The third statement in this script is left untouched as it does not depend on the current Reflex Action.

Thus, when Reflex Actions happen, the dependency chain within script is followed even when broken down into local variables.

This event-based runtime is called the *Reflex Runtime*.

### Observability
The Reflex Runtime uses the [Observer API](../the-observer-api) to observe objects in its scope whose properties can be observed. These are called *live objects*.

By default, the `this` object and the `document` object are observed. Thus, setting, updating or removing any of their properties using the Observer API will trigger the appropriate statement in a Reflex script.

```html
<div>

    <script type="reflex">
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

    <script type="reflex">
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

In any of the cases above, we could get deep object mutations to be caught by a Reflex script using the Observer API.

```html
<div>

    <script type="reflex">
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
While the `this` object and the `document` object are automatically observed from within an element's Reflex script, it is also possible to bind other objects to the script's scope. This is done using a `.bind()` method on the element - to bind locally, or on the `document` object - to bind globally for all scripts in the document.

```html
<div>

    <script type="reflex">
        console.log('Globally-bound clock time:', globallyBoundClock.time);
        console.log('Own-bound clock time:', locallyBoundClock.time);
    </script>

</div>

<script>
    // Create a collection of variables
    let globallyBoundClock = {time: '00:00:00',};
    // Bind them to all Reflex scripts in the document
    document.bind({globallyBoundClock});
    setInterval(() => {
        // Update existing binding
        Observer.set(globallyBoundClock, 'time', (new Date).toLocaleString());
        // Create new ones on the fly anytime
    }, 100 * 60);

    let divElement = document.querySelector('div');
    // Create a collection of variables
    let locallyBoundClock = {time: '00:00:00',};
    // Bind them to the current script
    divElement.bind({locallyBoundClock});
    setInterval(() => {
        // Update existing binding for the current script
        Observer.set(locallyBoundClock, 'time', (new Date).toLocaleString());
        // Create new ones on the fly anytime
    }, 100);

</script>
```

#### API
The following methods are used to dynamically bind observable variables to Reflex scripts.

+ **document.bind(bindings[, params]): Void** - This method lets us bind objects at the document-level for all Reflex scripts across the document.

    **Parameters:**
    + `bindings: Object` - The object to bind globally for all Reflex scripts in the document.
    + `params: Object` - (Optional) Binding options:
        + `update: Boolean` - Specifies whether to simply update existing variables in the global Reflex scope from properties of the given object or establish the given object as Reflex's new global scope. Default: `false` - establish as new global scope.

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
    + `bindings: Object` - The object to bind to the element's Reflex script.
    + `params: Object` - (Optional) Binding options:
        + `update: Boolean` - Specifies whether to simply update existing variables in the element's local scope from properties of the given object or establish the given object as the element's local scope. Default: `false` - establish as new scope.

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
Reflex features a way to handle errors that may occur within scripts. By default, script errors are logged to the console. But they can be silently ignored by setting a `script.errors` directive on the [OOHTML META tag](../the-oohtml-meta-tag).

```html
<html>
    <head>
        <meta name="chtml" content="script.errors=0;" />
    </head>
    <body>
        <h1></h1>
        <script type="reflex">
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
        <script type="reflex" binding errors="1">
            this.querySelectorSelectorSelector('h1').innerHTML = headline;
        </script>
    </body>
</html>
```

## Polyfill Support
The current [OOHTML polyfill implementation](../polyfill) has good support for Reflex scripts. The polyfill additionally makes it possible to customise the follwoing areas of its implementation of the syntax using the [OOHTML META tag](../the-oohtml-meta-tag):

+ **[selector.script](#convention)** - The CSS selector for matching the script element. The default selector is `script[type="reflex"]`. You may use a custom selector, like `script[is="my-script"][type="reflex"]`, where necessary.
        
    ```html
    <head>
        <meta name="oohtml" content="selector.script=script[is='my-script'][type='reflex'];" />
    </head>
    <body>
        <div>

            <script is="my-script" type="reflex"></script>
            <script is="my-script" type="reflex"></script>

        </div>
    </body>
    ```

+ **[api.bind](#api)** - The *method name* for binding objects to scripts. The standard *method name* is `bind`, but you may use a custom method name, where necessary.
        
    ```html
    <head>
        <meta name="oohtml" content="api.bind=setBinding;" />
    </head>
    ```
    
    ```js
    document.setBinding(binding);
    ```

+ **[api.unbind](#api)** - The *method name* for unbinding objects from scripts. The standard *method name* is `unbind`, but you may use a custom method name, where necessary.
        
    ```html
    <head>
        <meta name="oohtml" content="api.bind=clearBinding;" />
    </head>
    ```
    
    ```js
    document.clearBinding();
    ```

Learn more about customization and the OOHTML meta tag [here](../the-oohtml-meta-tag).