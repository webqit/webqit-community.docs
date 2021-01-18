# OOHTML Features Explainer

This information is being gathered to present OOHTML's design and architectural choices in the light of the problem space. We intend to maintain this as a living document; watch this space, or help us improve it by submitting a pull request or by [filing an issue](https://github.com/webqit/oohtml/issues).

## Object-Oriented Markup

*Naming and finding things* is hard; modular design makes it easier! This isn't necessarily some modern wisdom, we just haven't addressed HTML's old idea of one global scope for IDs and CSS selectors.

+ **[BEM](https://getbem.com)** has been an agreeable workaround for many people. It's, however, clunky. *Compare the [Namespaced Selectors](../namespaced-html#namespaced-selectors) in [Namespaced HTML](../namespaced-html).*

    BEM also doesn't really go beyond usage in CSS.

+ **[Stuart P.'s Parts and Walls](https://github.com/stuartpb/pwalls-spec)** proposal from 2015 is descriptive of the same need - for a modular naming convention; this time, with a JavaScript API in view. *Compare the *property-based* [Namespace API](../namespaced-html#api) in [Namespaced HTML](../namespaced-html).*

    It also looks like the DOM itself likes the idea of accessing strategic UI objects in JavaScript using a simple `object.property` convention. See [`document.head`](https://developer.mozilla.org/en-US/docs/Web/API/Document/head) and [`document.body`](https://developer.mozilla.org/en-US/docs/Web/API/Document/body). These are indeed more concise than `document.querySelector('head')` and `document.querySelector('body')`.

Overall, [Namespaced HTML](../namespaced-html) captures the gap between *how we endeavor to author HTML* and *what the current conventions allow*, and provides the ideal naming system that allows for keeping things to small-sized scopes.

## HTML Modules and Imports

On the expansive subject of [*HTML Modules*](https://github.com/WICG/webcomponents/issues/645), OOHTML takes a very, very different approach from the many existing opinions to how reusable markup should be delivered and consumed in the browser - *[OOHTML's HTML Modules](../html-modules)*. What is the most important difference?

+ OOHTML sees HTML Modules as simply a way to ship and consume **static, inert content**, and is thus more oriented towards HTML's standard primitive for *static, inert content* - the `<template>` element. OOHTML simply introduces [the **src** attribute](../html-modules#remote-content) as a way to *load a template's content from a remote HTML file*.
+ [This explainer](https://github.com/WICG/webcomponents/blob/gh-pages/proposals/html-modules-explainer.md), among others, however, follows the JavaScript route - loading **both static content and active script elements** using ES6 module infrastructure.

How do these compare?

+ One is *authoring and delivering HTML in HTML*...
    
    *File: /bundle.html*

    ```html
    <div exportgroup="blogPost">
        <p>Content...</p>
    </div>
    ```

    *Document*

    ```html
    <template name="bundle" src="/bundle.html"></template>
    ```

    *See also: [[proposal - **src** attribute]](https://discourse.wicg.io/t/add-src-attribute-to-template/2721), [[proposal - **src** attribute]](https://github.com/whatwg/html/issues/2791)*

    ...and the other is *authoring and delivering HTML with JavaScript*

    *File: /import.html*

    ```html
    <div id="blogPost">
        <p>Content...</p>
    </div>
    <script type="module">
        let blogPost = import.meta.document.querySelector("#blogPost");
        export { blogPost }
    </script>
    ```
    
    *Document*`

    ```js
    <script type="module">
        import { blogPost } from "import.html";
    </script>
    ```

+ One provides for consuming the imported HTML *both in JavaScript and in HTML*...
    
    *In JavaScript - See [API](../html-modules#api)*

    ```html
    <script>
        let [ blogPost ] = document.templates.bundle.exports.blogPost;
        document.body.appendChild(blogPost.cloneNode(true));
    </script>
    ```

    *See also: [[proposal - `document.templates`]](https://discourse.wicg.io/t/document-templates/1057)*
    
    *In HTML - See [OOHTML Imports](../html-imports)*

    ```html
    <body>
        <import name="blogPost" template="bundle"></import>
    </body>
    ```   

    *See also: [[HTML Modules - direction]](https://github.com/WICG/webcomponents/issues/863)*

    ...and the other remains based *only in JavaScript*

    ```html
    <body>
        <script type="module">
            import { blogPost } from "import.html"
            document.body.appendChild(blogPost);
        </script>
    </body>
    ```

+ One makes for *lazy-loading and load events*, as with elements like `<img>`...

    *JavaScript - See [events](../html-modules#module-events)*
    
    *HTML - See [OOHTML Imports](../html-imports)*

    ```html
    <body>
        <!-- Resolves whenever module loads -->
        <import name="blogPost" template="bundle"></import>
    </body>
    ```  

    ...and the other simply does not.

Overall, it seems more traditional to us to implement HTML Modules in HTML than in JavaScript, letting us do HTML in HTML (templates) and JavaScript in JavaScript (ES6 modules).

But then, loading **static, inert content**? What about cases where some reusable HTML has to go with some logic? [Reflex Scripts](../reflex-scripts) could be a good answer! We've had success with the following:

*File: /bundle.html*

```html
<div exportgroup="blogPost">
    <p id="content">Content...</p>
    <script type="reflex">
        this.querySelector('#content').innerHTML = document.state.content;
    </script>
    <!-- previous syntax: <script type="scoped"></script> -->
</div>
```

*Document*

```html
<body>
    <template name="bundle" src="/bundle.html"></template>
    <!-- Resolves whenever module loads, and the slotted element's scoped script activates -->
    <import name="blogPost" template="bundle"></import>
</body>
```

And how far can this go? We run this in production in <webqit.io>.

## State, Observability and Reactivity

Primitives for building *reactive* applications and keeping track of very dynamic state have not particularly found their way to native web languages.

+ **State:** We indeed have some primitives related to the concept of *state*, but they have very limited usecases. An example is [HTML5's DataSet API](https://developer.mozilla.org/en-US/docs/Web/API/HTMLOrForeignElement/dataset) that lets us keep state with HTML elements. But, it turns out to be tied to just HTML data `(data-*)` attributes - making it less useful as a general way to work with application state in the UI.

    Compare OOHTML's [State API](../the-state-api) which gives us robust *state management* - covering both document-level and element-level state, and the concept of automatic observability.

+ **Observability:** Much engineering still goes into using JavaScript's change-detection mechanisms for *reactive* UI development, and some things don't even scale. Check out a consideration of some of those difficulties in [this explainer](https://webqit.io/tooling/observer/explainer).

    Compare the [Observer API](https://github.com/webqit/observer) which provides generic APIs for observing and intercepting JavaScript objects. See its *universal* role across the rest of OOHTML, and potentially other technologies.

+ **Reactivity:** A Reactuve UI binding language? See [Reflex Scripts](../reflex-scripts). It comes bringing the full power of JavaScript for the job.

    Compare [this early idea for a template syntax by Jonathan Kingston](https://discourse.wicg.io/t/extension-of-template/447) from 2014, [this proposal](https://github.com/whatwg/html/issues/2254) from 2017, and [Apple's proposal](https://github.com/w3c/webcomponents/blob/gh-pages/proposals/Template-Instantiation.md) from 2017.
