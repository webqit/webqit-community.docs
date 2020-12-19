# Named Templates
HTML's standard `<template>` elements are a special type of element that helps us hold independent, reusable HTML snippets. But a standard way to name and find these templates has been missing.

Named Templates follow a naming convention that makes it easier to access them and their contents, and to organize them *meaningfully* in a document.

> OOHTML is [being proposed as a native browser technology](https://discourse.wicg.io/t/proposal-chtml/4716) but currently available through a polyfill. Be sure to check the [Polyfill Support](#polyfill-support) section below for the features on this page.

## Convention
Templates are named using the `name` attribute.

```html
<head>

    <template name="template1">
        <div>This is snippet 1</div>
        <div>This is snippet 2</div>
    </template>

</head>
```

For organizational purposes, templates may be nested. Unnested templates are called *top-level templates*.

```html
<head>

    <template name="template1">
        
        <div>This is snippet 1</div>
        <div>This is snippet 2</div>

        <template name="extended">
            <div>This is snippet 3</div>
            <div>This is snippet 4</div>
        </template>

    </template>

</head>
```

This convention also introduces the *imports* and *exports* terminology for snippets defined in a template.

### Imports and Exports
Independent snippets are kept within a template with a view to being *imported* into strategic places across the rendered document. These snippets are considered *exports*.

Exports are named using the `export` attribute.

```html
<head>

    <template name="template1">
        <div export="export-1">This is snippet 1</div>
        <div export="export-2">This is snippet 2</div>
    </template>

</head>
```

An *export ID* may span multiple snippets - usually when intended to be imported together.

```html
<head>

    <template name="template1">
        <label export="question" for="age">How old are you?</label>
        <input export="question" name="age" />
        <div export="anotherExport">This is another export</div>
    </template>

</head>
```

Snippets may be left unnamed; these are taken as *default exports*. Default exports implicitly have the export ID "default".

```html
<head>

    <template name="template1">
        <div>This is part of the default export</div>
        <div export="default">This is also part of the default export, as explicitly named</div>
    </template>

</head>
```

## Template Dependency Attribute
Regular elements may hold a reference to a `<template>` element where they depend on a template for rendering. For example, Custom Elements that implement the Shadow DOM often depend on markup stored in a template. These dependencies would be, in many cases, better maintained in the element's markup than in application code.

```html
<body>

    <my-prompt template="template1"></my-prompt>

</body>
```

A path expression is used to reference nested templates.

```html
<body>

    <my-prompt template="template1/extended"></my-prompt>

</body>
```

## Remote Content
Template elements may reference remote content using the `src` attribute.

**Remote file: http://localhost/templates.html**

```html

<div export="export-1"></div>
<div export="export-2"></div>
<template name="extended">
    <div export="export-3"></div>
    <div export="export-4"></div>
</template>

```

**Document: http://localhost**

```html
<head>
    <template name="template1" src="/templates.html"></template>
    <template name="template2">
        <div export="export-5"></div>
        <div export="export-6"></div>
    </template>
</head>
```

## The Named Templates API
*Named Templates* offers a set of APIs that puts templates and their exports up as objects, to facilitate Object-Orientend Development and to minimize selector-based queries.

+ **document.templates: Object** - This *readonly* property is a reflection of the state of the document's top-level templates.

    ```js
    let template1 = document.templates.template1; // Returns the "template1" element in the markup above
    ```

+ **HTMLTemplateElement.prototype.templates: Object** - This *readonly* property (`<template>.templates`) is a reflection of the state of a template's own nested templates.

    ```js
    let template1 = document.templates.template1;
    let extendedTemplate = template1.templates.extended; // Returns the nested "extended" element in the markup above
    ```

+ **HTMLTemplateElement.prototype.exports: Object** - This *readonly* property (`<template>.exports`) is a reflection of the state of a template's exports. Each export is given as an *array* of elements.

    ```js
    let template1 = document.templates.template1;

    // Named exports
    let questionExport = template1.exports.question; // Returns the "label" and "input" elements in the markup above
    console.log(questionExport.length); // 2

    let anotherExport = template1.exports.anotherExport; // Returns the "anotherExport" element in the markup above
    console.log(anotherExport.length); // 1

    // Default exports
    let defaultExport = template1.exports.default; // Returns the two "div" elements in the second markup above
    console.log(defaultExport.length); // 2
    ```

+ **Element.prototype.template: HTMLTemplateElement** - This *readonly* property (`<element>.template`) is a reference to a `<template>` element that may be referenced by an element as a dependency.

    ```js
    let templateDependency = myPrompt.template;
    ```

    ```js
    customElements.define('my-prompt', class extends HTMLElement {

        /**
         * Creates the Shadow DOM
         */
        constructor() {
            super();
            let templateDependency = this.template;
            let shadowContent = templateDependency.exports.question.map(el => el.cloneNode(true));
            let shadow =  this.attachShadow({mode: 'open'});
            shadow.append(...shadowContent);
        }

    });
    ```

    ```html
    <body>

        <!-- Flavour 1 of <my-prompt> -->
        <my-prompt template="template1"></my-prompt>
        <!-- Flavour 2 of <my-prompt> -->
        <my-prompt template="template2"></my-prompt>

    </body>
    ```

## Template Tree Observability
When templates - both *top-level* and nested - are added or removed on the document, events are fired on the document object.

+ **templateadded: Event** - This event is fired when a template is added to the document. The event object has a `.detail` property that gives the added template element and its nesting path.

    ```js
    document.addEventListener('templateadded', event => {
        console.log(event.detail.path, event.detail.template);
    });
    ```

    With the code above, adding a new template - `template2` - to the document would be reported in the console with path being `template2`.

    ```js
    let template = document.createElement('template');
    template.setAttribute('name', 'template2');
    document.body.append(template);
    ```

    With the code above, when the nested template - `template3` - completes loading its contents, a new template - `extended` - would have been added to the document and a report is logged to the console with path being `template2/template3/extended`.
        
    ```html
    <head>
        <template name="template2">
            <div export="export-5"></div>
            <div export="export-6"></div>
            <template name="template3" src="/templates.html"></template>
        </template>
    </head>
    ```

+ **templateremoved: Event** - This event is fired when a template is removed from the document. The event object has a `.detail` property that gives the removed template element and its nesting path.

    ```js
    document.addEventListener('templateremoved', event => {
        console.log(event.detail.path, event.detail.template);
    });
    ```

    With the code above, removing a template - the nested `template1` - from the document would be reported in the console with path being `template1/extended`.

    ```js
    document.templates.template1.templates.extended.remove();
    ```

+ **templatecontentloaded: Event** - This event is fired when a template completes loading its remote content. The event object has a `.detail` property that gives the removed template element and its nesting path.

    ```js
    document.addEventListener('templatecontentloaded', event => {
        console.log(event.detail.path, event.detail.template);
    });
    ```

    With the code above, when the nested template - `template3` - completes loading its contents, a report is logged to the console with path being `template2/template3`.
        
    ```html
    <head>
        <template name="template2">
            <div export="export-5"></div>
            <div export="export-6"></div>
            <template name="template3" src="/templates.html"></template>
        </template>
    </head>
    ```

## Polyfill Support
The current [OOHTML polyfill implementation](../installation/README.md) has full support for the Named Templates Specification. The polyfill additionally makes it possible to customise much of its implementation of the syntax using the [OOHTML meta tag](../the-oohtml-meta-tag/README.md). The following are areas of customization:

+ **[attr.templatename](#convention)** - The *template name* attribute. The standard *template name* attribute is `name`, but you may use a custom attribute name, where necessary.
        
    ```html
    <head>
        <meta name="oohtml" content="attr.templatename=data-name;" />
        <template data-name="template2">
            <div export="export-1"></div>
            <div export="export-2"></div>
        </template>
    </head>
    ```

+ **[attr.export](#imports-and-exports)** - The *export ID* attribute. The standard *export ID* attribute is `export`, but you may use a custom attribute name, where necessary.
        
    ```html
    <head>
        <meta name="oohtml" content="attr.export=data-export;" />
        <template name="template2">
            <div data-export="export-1"></div>
            <div data-export="export-2"></div>
        </template>
    </head>
    ```

+ **[attr.templatedep](#template-dependency-attribute)** - The *template dependency* attribute. The standard *template dependency* attribute is `template`, but you may use a custom attribute name, where necessary.
        
    ```html
    <head>
        <meta name="oohtml" content="attr.templatedep=data-template;" />
        <div data-template="template2">
            <import name="export-1"></import>
        </div>
    </head>
    ```

+ **[api.templates](#the-named-templates-api)** - The *templates* property exposed on the document object and on HTML template elements. The standard *templates* property is `templates`, but you may use a custom property name, where necessary.
        
    ```html
    <head>
        <meta name="oohtml" content="api.templates=templatelist;" />
    </head>
    ```
    
    ```js
    let template1 = document.templatelist.template1;
    ```

+ **[api.exports](#the-named-templates-api)** - The *exports* property exposed on HTML template elements. The standard *exports* property is `templates`, but you may use a custom property name, where necessary.
        
    ```html
    <head>
        <meta name="oohtml" content="api.exports=exportlist;" />
    </head>
    ```
    
    ```js
    let export1 = template1.exportlist.export1;
    ```

+ **[api.templatedep](#the-named-templates-api)** - The *template dependency* property exposed on HTML elements. The standard *template dependency* property is `template`, but you may use a custom property name, where necessary.
        
    ```html
    <head>
        <meta name="oohtml" content="api.templatedep=tpl;" />
    </head>
    ```
    
    ```js
    let templateDependency = myPrompt.tpl;
    ```

Learn more about customization and the OOHTML meta tag [here](../the-oohtml-meta-tag/README.md).