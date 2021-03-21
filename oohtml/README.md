---
icon: code-slash
desc: A suite of new DOM features that particularly facilitates writing modular, reusable, and reactive HTML and JavaScript natively and more conveniently.
desc2: Handcraft your HTML, CSS and JavaScript into a powerful UI without writing a line of framework-specific code! OOHTML comes with all modern UI development paradigms built into the plain old HTML so you can go without a tool.
categories: Featured, Web-Native
tags: OOHTML, CHTML, Proposal
_before: webflo
---
# OOHTML

<!-- BADGES/ -->

<span class="badge-npmversion"><a href="https://npmjs.org/package/@webqit/oohtml" title="View this project on NPM"><img src="https://img.shields.io/npm/v/@webqit/oohtml.svg" alt="NPM version" /></a></span>
<span class="badge-npmdownloads"><a href="https://npmjs.org/package/@webqit/oohtml" title="View this project on NPM"><img src="https://img.shields.io/npm/dm/@webqit/oohtml.svg" alt="NPM downloads" /></a></span>

<!-- /BADGES -->

[Object-Oriented HTML (OOHTML)](https://github.com/webqit/oohtml) is a suite of new DOM features that particularly facilitates writing *modular*, *reusable*, and *reactive* HTML and JavaScript *natively* and *more conveniently*.

*Handcraft your HTML, CSS and JavaScript into a powerful UI without writing a line of framework-specific code! OOHTML comes with all modern UI development paradigms built into the plain old HTML so you can go without a tool.*

> OOHTML is being proposed as a [W3C standard at the Web Platform Incubator Community Group](https://discourse.wicg.io/t/proposal-chtml/4716). Consider bringing your ideas to the discussion.

> [Visit project repo](https://github.com/webqit/oohtml).

> This entire documentation is currently being improved on.

## Features
OOHTML brings five new features that make common UI design terminologies possible without external tooling. These features may be used individually or together to improve how we author the UI. Here is an overview:

+ [HTML Modules](#html-modules)
+ [HTML Imports](#html-imports)
+ [Namespaced HTML](#namespaced-html)
+ [The State API](#the-state-api)
+ [Subscript](#subscript)

### HTML Modules
HTML Modules are a way to implement reusable HTML markup using the *module*, *import* and *export* paradigm. This feature establishes the standard `<template>` element as the module infrastructure for HTML and introduces new attributes, properties and events that make that work.

Every module is a regular `<template>` element with a `name` attribute - *the module ID*, and the contents if a module are simply regarded as *exports*.

```html
<head>

    <template name="module1">

        <label for="age">How old are you?</div>
        <input id="age" />

    </template>

</head>
```

Exports may be more properly wrapped within an `<export>` element of a designated name.

```html
<head>

    <template name="module1">

        <export name="question">
            <label for="age">How old are you?</label>
            <input id="age" />
        </export>

        <div>This is another export</div>

    </template>

</head>
```

Or they may be individually *tagged* to an export identifier using the `exportgroup` attribute.

```html
<head>

    <template name="module1">

        <label exportgroup="question" for="age">How old are you?</label>
        <input exportgroup="question" name="age" />
        
        <div>This is another export</div>

    </template>

</head>
```

Either way, they are accessed the same way using the *Modules API*.

```js
// Access module1 from document.templates
let module1 = document.templates.module1;

// Import module1's exports
let questionExport = module1.exports.question; // Array

// Clone the elements in the export
let questionExportClone = questionExport.map(el => el.cloneNode(true));
```

Taking things further, template elements may reference remote content using the `src` attribute.

```html
<head>

    <template name="module-remote" src="/bundle.html"></template>

</head>
```

The contents of the remote file automatically become the template's content on load.

*Details are in the [HTML Modules](html-modules) documentation. Learn more about the convention, API, events, and the polyfill support.*

### HTML Imports
HTML Imports are a declarative way to *import* reusable snippets from HTML Modules into slots in the main document.

> Note that this is not the same as the HTML Imports that earlier shipped with Web Components.

Here, an `<import>` element in the `<body>` area simply points to a module to *place* one of its exports.

```html
<body>

    <!-- Import question here from module1 -->
    <import name="question" template="module1"></import>

</body>
```

Resolution takes place and the `<import>` element is replaced by all of the imported contents.

```html
<body>

    <!-- import element replaced -->
    <label for="age">How old are you?</label>
    <input id="age" />

</body>
```

The `<import>` stands by somewhere so that it can return to its slot in the event that all of the imported ccontents are removed. Details ahead.

Now, it is possible to define one *module ID* on a regular element for use by one or more `<import>` elements within the block.

```html
<body>

    <!-- Point to a module; one module ID for all imports within -->
    <div template="module1">

        <!-- Import question here from module1 -->
        <import name="question"></import>

        <div>
            <!-- Import another export here from module1 -->
            <import name="export-2"></import>
        </div>

    </div>

</body>
```

*Imports* maintain a *live* relationship with the modules they point to. For example, they are resolved again when the module ID is dynamically pointed at another module element.

```js
// Changing the module ID on the DIV container above will see all associated imports resolved again
document.querySelector('div[template="module1"]').setAttribute('template', 'module2');
```

This opens up new simple ways to create very dynamic applications.

*Details are in the [HTML Imports](html-imports) documentation. Learn more about the convention, dynamicity, Slot Inheritance, isomorphic rendering, and the polyfill support.*

### Namespaced HTML
Namespacing is a way to let an element establish its own naming context for descendant elements. This makes it possible to keep IDs scoped to a context other than the document's global scope; thus the ability to write collision-free IDs across a page.

The following modular markup implements its IDs in namespaces:

```html
<article id="continents" namespace>
    <section id="europe" namespace>
        <div id="about">About Europe</b></div>
        <div id="countries">Countries in Europe</div>
    </section>
    <section id="asia" namespace>
        <div id="about">About Asia</b></div>
        <div id="countries">Countries in Asia</div>
    </section>
</article>
```

Namespaced HTML gives us many benefits. For example, the above gives us a conceptual model of repeating objects:

```html
continents
|- europe
|   |- about
|   |- countries
|- asia
    |- about
    |- countries
```

The *namespace API* translates that model into a real object tree:

```js
// Get the "continents" article
let continents = document.namespace.continents;

// Access scoped IDs with the new "namespace" DOM property
let europe = continents.namespace.europe;
let asia = continents.namespace.asia;

// And for deeply-nested IDs...
let aboutAsia = continents.namespace.asia.namespace.about;
```

We get a document structure that's easier to reason about and that translates to a more consistent, bankable API.

Namespaced HTML also helps us to (1) write collision-free IDs and selectors, and (2) by means of the Namespace API, minimize selector-based DOM traversal (e.g `el.namespace.some_id` vs `el.querySelector('#some_id')`). Much of our code in the examples below will now use the `namespace` attribute in markup and the `.namespace` property in JS.

*Details are in the [Namespaced HTML](namespaced-html) documentation. Learn more about the convention, Namespaced Selectors, API, observability, and the polyfill support.*

### The State API
The State API is a DOM API that lets us maintain application state at both the document level and the individual element level. It makes it easy to think about application state at different levels in the DOM tree and to keep track of changes at each level.

This API exposes a `.state` property on the document object and on elements. Arbitrary values can be set and read here the same way we would with regular objects.

```js
// At the document level
document.state.pageTitle = 'Hello World!';
console.log(document.state.pageTitle); // Hello World!

// At the element level
element.state.collapsed = true;
console.log(element.state.collapsed); // true
```

*State Objects* are *live objects* that can be observed for changes using the [Observer API](the-observer-api).

```js
Observer.observe(document.state, 'pageTitle', e => {
    console.log('New Page Title: ' + e.value);
    // Or we could reflect this state on the document title element
    document.querySelector('title').innerHTML = e.value;
});
```

This lets us build very reactive applications natively.

Using an element's state API, here's how we could make a practical *collapsible* component.

```html
<my-collapsible namespace>
    <div id="control">Toggle Me</div>
    <div id="content" style="height: 0px">
        Some content
    </div>
</my-collapsible>
```

```js
customElements.define('my-collapsible', class extends HTMLElement {

    /**
     * Creates the Shadow DOM
     */
    constructor() {
        super();
        // Observe state and get the UI synced
        Observer.observe(this.state, 'collapsed', e => {
            this.namespace.content.style.height = e.value ? '0px' : 'auto';
            this.setAttribute('data-collapsed', e.value ? 'true' : 'false');
        });

        // Implement the logic for toggling collapsion
        this.namespace.control.addEventListener('click', function() {
            this.state.collapsed = !this.state.collapsed;
        });
    }

});
```

Now, other parts of the application are able to work in sync with the state of this element.

```js
let collapsible = document.querySelector('my-collapsible');
Observer.observe(collapsible.state, 'collapsed', e => {
    console.log(e.value ? 'element collapsed' : 'element expanded');
});
```

*Details are in the [State API](the-state-api) documentation. Learn more about the API, deep observability, and the polyfill support.*

### Subscript
Subscript is a type of JavaScript runtime that lets us create scoped, *reactive* `<script>` elements across the UI. That gives us a UI binding language and the ability to have UI logic without involving actual JavaScript classes or files.

The following `<script>` element is scoped to the `#alert` element - its host element - instead of the global browser scope:

```html
<div id="alert">
    <script type="subscript">
        console.log(this.id); // alert
    </script>
</div>
```

As seen, the `this` variable is a reference to the script's host element. In addition, variables declared within the script are available only within the script, while global variables are always available across all scripts.

```html
<div id="alert" namespace>

    <div id="message"></div>
    <div id="exit" title="Close this message.">X</div>

    <script type="subscript">
        this.namespace.exit.addEventListener('click', () => {
            this.remove();
        });
    </script>

</div>
```

Below is how we could render something - a message - from the element's state object - `.state.message`.

```html
<body>

    <div id="alert" namespace>

        <div id="message"></div>
        <div id="exit" title="Close this message.">X</div>

        <script type="subscript">
            // Render the "message" property from the element's state object

            this.namespace.message.innerHTML = this.state.message;
            this.namespace.exit.addEventListener('click', () => {
                this.remove();
            });
        </script>

    </div>

<body>
```

And here is one of the most important features of Subscript: reactivity! While Subscript works like regular JavaScript, it is also able to observe changes in its scope and respond to them. And it does this at the statement level such that a given statement is evaluated again and independent of other statements whenever the observable references on the statement change. Thus, in the code above, any changes made to the observable reference`.state.message` will trigger that particular statement to run again.

```js
let alertElement = document.querySelector('#alert');
alertElement.state.message = 'Task started!';
setTimeout(() => {
    alertElement.state.message = 'Task complete!';
}, 1000);
```

Leveraging this reactivity, here's how we could implement the `<my-collapsible>` component we created in the *State API* section above - this time, without a JavaScript class.

```html
<div id="collapsible" namespace>
    
    <div id="control">Toggle Me</div>
    <div id="content" style="height: 0px">
        Some content
    </div>

    <script type="subscript">
        // Observe state and get the UI synced, without requiring Observer.observe() here...

        this.setAttribute('data-collapsed', this.state.collapsed ? 'true' : 'false');
        this.namespace.content.style.height = this.state.collapsed ? '0px' : 'auto';
        this.namespace.control.addEventListener('click', () => {
            // Toggle collapsion state
            this.state.collapsed = !this.state.collapsed;
        });
    </script>

</div>
```

> We can see that with Subscript, we sometimes don't need as much as a custom element to bring life to some areas in the UI.

*Details are in the [Subscript](subscript) documentation. Learn more about the event-based runtime, deep observability, bindings, the API, error handling, and the polyfill support.*

## Getting Started
All it takes to use OOHTML is include the [polyfill](polyfill) on a page. You can begin writing modular, reusable, reactive HTML right away! You definitely want to visit the documentation for each feature and try everything out by pasting the code examples and running them right on your browser.

We're putting together a collection of examples in the [examples](examples) section.

## Getting Involved
*Platform feature* proposals aren't the easiest thing in the world!

+ They have to be something everyone can agree on! This makes us want to hear from you.

    If you indeed have a usecase for all, or aspects, of OOHTML, or have some opinions, you should please join the discussion at the [WICG](https://discourse.wicg.io/t/proposal-chtml/4716).
    
    If you are building something early with it (just as we are building [webqit.io](//webqit.io) with it), we'd like to hear from you via any means - [WICG](https://discourse.wicg.io/t/proposal-chtml/4716), [email - webqit.io@gmail.com], [Github Discussions](https://github.com/webqit/oohtml/discussions). Your personal experience working with OOHTML may have something to give back in some way to its development. You may also like us to feature your work with OOHTML.
+ Platform feature proposals also have to go through a million iterations! And much in dollars go into that!

    If you could help in some way, we'd be more than glad! If you'd like to find out what your $1 could do for us and what's in for you, do indeed reach out at webqit.io@gmail.com.

## Design Goals
See the [features explainer](explainer).

## Issues
To report bugs or request features, please submit an [issue](https://github.com/webqit/oohtml/issues).

## Discusions (new)
We now have [Github Discussions](https://github.com/webqit/oohtml/discussions) for OOHTML. General feedback may be taken here instead of filing an issue.


## License
MIT.
