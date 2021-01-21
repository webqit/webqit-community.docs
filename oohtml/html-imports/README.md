---
_after: html-modules
---
# HTML Imports
HTML Imports are a declarative way to place reusable HTML snippets on any location in the DOM. This feature abstracts over the [HTML Modules API](../html-modules) to provide incredibly powerful composability in just a matter of tags and attributes.

> OOHTML is [being proposed as a native browser technology](https://discourse.wicg.io/t/proposal-chtml/4716) while currently available through a polyfill. Be sure to check the [Polyfill Support](#polyfill-support) section below for the features described on this page.

## Convention
The [HTML Modules Specification](../html-modules) introduces the *import* and *export* terminology in working with `<template>` elements and their contents.

The `<template>` elements below are HTML modules. Exports are defined using the two standard conventions.

```html
<head>

    <template name="module2">

        <export name="export-1">
            <img src="logo.png" />
        </export>

        <export name="export-2">
            <label for="email">Enter your email address</label>
            <input id="email" type="email" />
        </export>

    </template>

    <template name="module1">

        <img exportgroup="export-1" src="logo.png" /> <!-- part of export-1 -->

        <label exportgroup="export-2" for="email">Enter your email address</label> <!-- part of export-2 -->
        <input exportgroup="export-2" id="email" type="email" /> <!-- part of export-2 -->

    </template>

</head>
```

HTML Imports introduces an `<import>` element that lets us place these *exports* into any location on the DOM.

```html
<body>
 
    <!-- Place all of export-1 here -->
    <import name="export-1" template="module1"></import>

</body>
```   

An import element uses the `name` attribute as the *export* identifier and the `template` attribute as the *module* identifier.

*Resolution* takes place and the `<import>` element is replaced by the *export*.

```html
<body>
 
    <img exportgroup="export-1" src="logo.png" />
    <!-- import element replaced -->

</body>
```   

And all elements of an *export ID* go together into the same *import slot*.

```html
<body>
 
    <question-element>
        <!-- Place all of export-2 here -->
        <import name="export-2" template="module1"></import>
    </question-element>

</body>
```   

```html
<body>
 
    <question-element>
        <label exportgroup="export-2" for="email">Enter your email address</label>
        <input exportgroup="export-2" id="email" type="email" />
        <!-- import element replaced -->
    </question-element>

</body>
```   

The module identifier may also be defined on any element to establish a *resolution scope* for all import elements within its subtree.

```html
<body template="module1">
 
    <!-- Place all of export-1 here -->
    <import name="export-1"</import>
    <question-element>
        <!-- Place all of export-2 here -->
        <import name="export-2"></import>
    </question-element>

</body>
```

Other import elements within a *resolution scope* can still reference their own module.

```html
<body template="module1">
 
    <!-- Place all of export-1 here -->
    <import name="export-1"</import>
    <question-element>
        <!-- Place all of export-2 here -->
        <import name="export-2" template="module2"></import>
    </question-element>

</body>
```

### Default Exports and Imports
HTML Modules may define contents without an export identifier - [Default Exports](../html-modules#default-exports). These arbitrary contents are also imported without an identifier.

```html
<head>

    <template name="module3">

        <label for="email">Enter your email</label> <!-- This is part of the default export -->
        <input id="email" type="email" /> <!-- This is part of the default export -->

    </template>

</head>
```

```html
<body>
 
    <!-- Place all of the default export here -->
    <import template="module3"></import>

</body>
```   

### Module Reference Specificity
Import elements that reference nested modules may define a fallback directive using the `template-specificity` attribute. A fallback directive allows the import resolution to fall back to ancestor modules to find the specified *export*.

The directive below is to fall back up to 2 steps from the right to find `export-2`.

```html
<body>

    <import name="export-2" template="module1/nonexistent/nonexistent" template-specificity="-2"></import>

</body>
```

The directive below is to fall back up to 2 steps from the right, to at most 1 step from the left, to find `export-2`.

```html
<body>

    <import name="export-2" template="module1/nonexistent/nonexistent/nonexistent" template-specificity="1-2"></import>

</body>
```

> The resolution of the above ends up at `module1/nonexistent` which is still *nonexisitent*. The import element is left unresolved.

## Dynamicity
*Import slots* are resolved in realtime in response to a number of events.

+ When the module reference - the `template` attribute - is changed, the import element is resolved again.

    ```js
    document.querySelector('import[template="module1"]').setAttribute('template', 'module2');
    ```
    
    This also applies to all import elements within resolution scopes. Below, all import elements scoped to `<body>` will be resolved again.

    ```js
    document.querySelector('body').setAttribute('template', 'module2');
    ```

+ On deleting the last of its slotted elements, an import element is automatically restored.

    ```js
    // Remove the first element slotted
    document.querySelector('label').remove();
    // Remove the last element slotted
    document.querySelector('input').remove();
    // The original import element should now be restored
    ```

+ When exports and modules are added or removed, all import elements referencing them are reevaluated accordingly.

    With the module addition below, all import elements referencing `module1/nonexistent` will now be resolved.

    ```js
    let nestedModule = document.createElement('template');
    nestedModule.setAttribute('name', 'nonexistent');
    nestedModule.content.append(document.createElement('div'));

    let module1 = document.querySelector('template[name="module1"]');
    module1.content.append(nestedModule);
    ```
    
    With the module removal below, all import elements currently *resolved* from `module1` and its nested modules will now be restored.

    ```js
    module1.remove();
    ```

+ When the external contents of a remote module become available, all import elements that depend on the loaded content will be resolved. (See [Module Events](../html-modules#module-events).)

    **Remote file: /bundle.html**

    ```html

    <div exportgroup="export-2"></div>
    <template name="module-loaded">
        <div exportgroup="export-3"></div>
    </template>

    ```

    **Document:**

    ```html
    <head>

        <template name="module1">
            <div exportgroup="export-1"></div>
            <template name="module-remote" src="/bundle.html"></template>
        </template>

    </head>
    ```

    ```html
    <body>

        <!-- This is resolved immediately -->
        <import name="export-1" template="module1"></import>

        <!-- This is resolved as the targetted export becomes available -->
        <import name="export-2" template="module1/module-remote"></import>

        <!-- This is resolved as the targetted export becomes available -->
        <import name="export-3" template="module1/module-remote/module-loaded"></import>

    </body>
    ```

## Slot Inheritance
An import element will work as a regular element until it is resolved, that is, replaced. Useful default content, along with attributes, may be displayed on its slot this way.
    
```html
<body>

    <div template="module1">
        <!-- An ixport element with default attribute and content -->
        <import name="export-1" style="max-width:50px">No image</import>
    </div>

</body>
```

A slot's default content and attributes may also be inheritted by slotted elements to persist the semantics of the slot.

+ **Attributes** - All attributes of an `<import>` element (other than the `name`, `template` and `template-specificity` attributes) are inheritted by slotted elements.
        
    ```html
    <body>

        <div template="module1">
            <!-- The slotted element now inherits the style attribute  -->
            <img exportgroup="export-1" src="logo.png" style="max-width:50px" />
        </div>

    </body>
    ```

    When the attribute to inherit already exists on the element being slotted, one of the following happens:
    + If the attribute is of space-delimitted attributes, like the `class` attribute, inheritted non-duplicate values are placed after existing values on the element being slotted.
    + If the attribute is of key/value attributes, like the `style` attribute, inheritted declarations are placed after existing declarations on the element being slotted (making CSS cascading work for the `style` attribute).
    + If the attribute is of single-value attributes, like the `id` attribute, the inheritted attribute value is made to replace exisiting value on the element being slotted.
    + All other attribute types are treated as single-value attributes.

+ **Content** - To inherit the default content of an import element, the import element will have to define its contents as exports and the element being slotted will have to reference this import element as its module and import its exports.

    The import element will have to implicitly or explicitly define its contents as exports, thus acting as a module.

    ```html
    <body>

        <div template="module2">
            <!--
            The import element as a module
            -->
            <import name="export-1">
                <div exportgroup="default-content">No results</div>
            </import>
        </div>

    </body>
    ```

    The element being slotted will have to reference this import element as its module by defining a `template="@slot"` attribute (instead of pointing to an actual `<template>` element). Import elements defined within this element being slotted are resolved from the destination slot.
         
    ```html
    <head>

        <template name="module2">

            <!--
            The element being slotted pointing to its destination slot
            -->
            <div exportgroup="export-1" template="@slot">
                <!--
                The import element that actually imports the contents of its destination slot
                -->
                <import name="default-content"></import>
            </div>

        </template>

    </head>
    ```

    The default content is thus persisted on resolving the above.

    ```html
    <body>

        <div template="module2">
            <div exportgroup="export-1" template="@slot">
                <div exportgroup="default-content">No results</div>
            </div>
        </div>

    </body>
    ```

## Isomorphic Rendering
When rendering happens on the server and has to be serialized for the browser to take over, import elements that are replaced on the server will need to be kept in some way in the serialized HTML output. This would enable the browser, on load, to *hydrate* the original import elements and map them in their replaced state to their currently slotted contents. With this, deleting a slot's contents, for example, can trigger the restoration of the hydrated `<import>` element.

HTML Imports makes it possible to serialize `<import>` elements as *comment nodes* (`<!-- <import></import> -->`) when rendering on the server. It can also be told in the browser to rehydrate these import elements on loading the serialized HTML output. This feature is turned on by setting the `isomorphic` directive to `1` or `true` on the [OOHTML META tag](../the-oohtml-meta-tag).

**HTML to be rendered on the server**

```html
<html>

    <head>

        <meta name="oohtml" content="isomorphic=true;" />
        <template name="module2">
            <div exportgroup="export-1"></div>
            <div exportgroup="export-2"></div>
        </template>

    </head>

    <body>

        <div template="module1">
            <import name="export-1" id="headline" style="color:red">Default Headline</import>
        </div>

        <import template="module1" name="export-1" style="color:blue"></import>

    </body>

</html>
```

**The serialized HTML output**

```html
<html>

    <head>

        <meta name="oohtml" content="isomorphic=true;" />
        <template name="module2">
            <div exportgroup="export-1"></div>
            <div exportgroup="export-2"></div>
        </template>

    </head>

    <body>

        <div template="module1">
            <div exportgroup="export-1" id="headline" style="color:red"></div>
            <!-- <import name="export-1" id="headline" style="color:red">Default Headline</import> -->
        </div>

        <div exportgroup="export-1" style="color:blue"></div>
        <!-- <import template="module1" name="export-1" style="color:blue"></import> -->

    </body>

</html>
```

**Hydrated slots in the browser**
On loading the above serialized HTML output in the browser, find and delete the server-slotted element with ID `#headline`. The original `<import>` element should now be restored.

```html
<html>

    <head>

        <meta name="oohtml" content="isomorphic=true;" />
        <template name="module2">
            <div exportgroup="export-1"></div>
            <div exportgroup="export-2"></div>
        </template>

    </head>

    <body>

        <div template="module1">
            <import name="slot-1" id="headline" style="color:red">Default Headline</import>
            <!-- <import name="slot-1" id="headline" style="color:red">Default Headline</import> -->
        </div>

        <div import="slot-1" style="color:blue"></div>
        <!-- <import template="module1" name="slot-1" style="color:blue"></import> -->

    </body>

</html>
```

## Polyfill Support
The current [OOHTML polyfill implementation](../polyfill) has full support for HTML Imports. The polyfill additionally makes it possible to customise the following areas of its implementation of the syntax using the [OOHTML META tag](../the-oohtml-meta-tag):

+ **[element.import](#convention)** - The tag name for the import element. The standard import element is `<import>`, but the polyfill uses `<html-import>` by default. This can be reset to the standard tag name or to something else.
        
    ```html
    <head>
        <meta name="oohtml" content="element.import=import;" />
    </head>
    <body>
        <import name="export-1" template="module2"></import>
    </body>
    ```

+ **[attr.importid](#convention)** - The *import ID* attribute. The standard attribute is `name`, but you may use a custom attribute name, where necessary.
        
    ```html
    <head>
        <meta name="oohtml" content="attr.importid=data-name;" />
    </head>
    <body>
        <import data-name="export-1" template="module2"></import>
    </body>
    ```

+ **[attr.templatespec](#module-reference-specificity)** - The *module specificity* attribute. The standard attribute is `template-specificity`, but you may use a custom attribute name, where necessary.
        
    ```html
    <head>
        <meta name="oohtml" content="attr.templatespec=module-specificity;" />
    </head>
    <body>
        <import name="export-1" template="module2" module-specificity="0-2"></import>
    </body>
    ```

Learn more about customization and the OOHTML META tag [here](../the-oohtml-meta-tag).