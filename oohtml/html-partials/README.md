# HTML Partials
HTML Partials is a new templating feature that abstracts over the [Named Template Specification](../modular-naming-and-apis/README.md#named-templates) to provide incredibly powerful composability in the simple language of tags and attributes.

> OOHTML is [being proposed as a native browser technology](https://discourse.wicg.io/t/proposal-chtml/4716) but currently available through a polyfill. Be sure to check the [Polyfill Support](#polyfill-support) section below for the features on this page.

## Templates, Imports and Exports
The Named Template Specification introduces the *import* and *export* terminology to working with `<template>` elements and their contents.

```html
<head>

    <template name="template1">
        <div export="export-1">This is export1 in template1</div>
        <div export="export-2">This is export2 in template1</div>
    </template>
    <template name="template2">
        <div export="export-3">This is export3 in template2</div>
        <div export="export-4">This is export4 in template2</div>
    </template>

</head>
```

HTML Partials makes it possible for an element in the main document to point to the `<template>` element and import its *exports*. 

```html
<body>

    <!-- Point to a template element -->
    <div template="template1">
        <h2>I have imports</h2>
        <!-- Place export-1 here -->
        <import name="export-1"></import>

        <div>
            <!-- Place export-2 here -->
            <import name="export-2"></import>
        </div>
    </div>

</body>
```

*Slotting* takes place and the `<import>` elements are replaced by the template's exports.

```html
<body>

    <div template="template1">
        <h2>I have imports</h2>
        <div export="export-1">This is export1 in template1</div>

        <div>
            <div export="export-2">This is export2 in template1</div>
        </div>
    </div>

</body>
```

The element is said to have *implemented* the `<template>`. The element is known as an *implementation block*.

HTML Partials also supports *default exports* - a template's exports without an explicit `export` attribute or with an `export` attribute of "default". Default exports are slotted into the *default import slot* in the implementation block.

```html
<body>

    <!-- Point to a template element -->
    <div template="template1">
        <!-- Place default exports here -->
        <import name="default"></import>
    </div>

</body>
```

By default, *import slots* are scoped to their containing implementation block. But the `<import>` element may also be used independently of an implementation block to point to its own `<template>`.

```html
<body>

    <!-- Point to a template element -->
    <div template="template1">
        <h2>I have imports</h2>
        <!-- Place export-1 here -->
        <import name="export-1"></import>

        <div>
            <!-- Point to template2 and place export-3 here -->
            <import name="export-3" template="template2"></import>
        </div>
    </div>

</body>
```

## Dynamicity
*Import slots* are resolved in realtime.

+ The `template` attribute can be dynamically set. Import slots are evaluated accordingly.

    ```js
    document.querySelector('div[template="template1"]').setAttribute('template', 'template1/extended/nonexistent');
    ```

+ Template elements can be added or removed dynamically. Import slots that depend on them are evaluated accordingly.

    ```js
    document.querySelector('template[name="template1"]').remove();
    ```

+ Import elements that don't find a matching *export* in their implied source templates are left in place or are automatically restored if previously resolved.

    ```html
    <div template="template1/extended/nonexistent">
        <!-- This import slot won't be resolved, or will be restored if previously resolved -->
        <import name="export-2" template-fallback="2"></import>
    </div>
    ```

+ Import elements that are resolved from nested templates may define a fallback directive. A fallback directive allows the import resolution to fall back to ancestor templates where a matching *export* is not defined in the implied source template.

    ```html
    <div template="template1/extended/nonexistent">
        <!-- Fall back up to 2 steps backwards to find export-2 -->
        <import name="export-2" template-fallback="2"></import>
    </div>
    ```

+ On deleting the last of its slotted elements, an import element will be automatically restored.

    ```js
    // The original import element for this slot will be restored
    document.querySelector('div[template="template1"] div[export="export-2"').remove();
    ```

## Slot Inheritance
An import element may have default content that displays before slotting takes place. It may also have attributes.
    
```html
<body>

    <div template="template1">
        <!-- An ixport element with default attribute and content -->
        <import name="import-1" style="color:red">No results</import>
    </div>

</body>
```

Now, these default attributes and content can be inheritted by slotted elements.

+ **Attributes** - An `<import>`'s attributes (other than the `name` and `template` attributes) are inheritable by slotted elements.
        
    ```html
    <body>

        <div template="template1">
            <!-- The slotted element now inherits the style attribute  -->
            <div export="export-1" style="color:red">This is snippet 1</div>
        </div>

    </body>
    ```

    + On inheriting single-value attributes, like the `id` attribute, a replacement will be made where the slotted element has an existing attribute.
    + On inheriting space-delimitted attributes, like the `class` attribute, new and non-duplicate values are placed after any existing values that may be on the slotted element.
    + On inheriting key/value attributes, like the `style` attribute, new declarations are placed after any existing declarations that may be on the slotted element (making CSS cascading work for the `style` attribute).

+ **Content** - To inherit the default content of an import element, the import element will have to act as a `<template>` element while the slotting element as an implementation block for the import element.

    To act as a `<template>` element, the import element will have to implicitly or explicitly define its default contents as exports.

    ```html
    <body>

        <div template="template2">
            <!--
            The import element with default content
            defined in the format of a <template>
            -->
            <import name="export-1">
                <div export="default-content">No results</div>
            </import>
        </div>

    </body>
    ```

    To act as an implementation block, the slotting element will have to define a `template` attribute but set it to the keyword `@slot` (instead of pointing to an actual `<template>` element), then define its own import elements that actually import the default content from off the slot.
         
    ```html
    <head>

        <template name="template2">

            <!--
            The slotting element
            now acting as an implementation block
            for its destination slot
            -->
            <div export="export-1" template="@slot">
                <import name="default-content"></import>
            </div>

        </template>

    </head>
    ```

     After slotting takes place, the final composition should look like:

    ```html
    <body>

        <div template="template2">
            <div export="export-1" template="@slot">
                <div export="default-content">No results</div>
            </div>
        </div>

    </body>
    ```

## Responsive Imports
Import elements may depend on templates with remote exports, and they would be resolved as soon as those contents are loaded. (See [Template Events](../named-templates/README.md#the-named-templates-api).)

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
    <template name="template2">
        <div export="export-5"></div>
        <div export="export-6"></div>
        <template name="template3" src="/templates.html"></template>
    </template>
</head>
```

Now, we would expect the second import slot to be resolved before the first.

```html
<body>

    <div template="template2/template3/extended">
        <import export="export-3"></import>
    </div>

    <div template="template2">
        <import export="export-5"></import>
    </div>

</body>
```

## Isomorphic Rendering
When rendering happens on the server and has to be serialized for the browser to take over, the browser must still be able to maintain references to all `<import>`s, even those replaced on the server. HTML Partials addresses this by serializing `<import>` elements as *comment nodes* (`<!-- <import></import> -->`) with a view to recreating the original import elements from these comment nodes on getting to the browser. This way, composition is able to continue. Thus, in the browser, deleting a server-slotted element, for example, should trigger the restoration of the original `<import>` element; changing the `template` attribute of an element should get its import slots re-evaluated.

**Before Rendering on the Server**

```html
<html>

    <head>

        <template name="template2">
            <div export="export-1"></div>
            <div export="export-2"></div>
        </template>

    </head>

    <body>

        <div template="template1">
            <import name="export-1" id="headline" style="color:red">Default Headline</import>
        </div>

        <import template="template1" name="export-1" style="color:blue"></import>

    </body>

</html>
```

**After Rendering on the Server**

```html
<html>

    <head>

        <template name="template2">
            <div export="export-1"></div>
            <div export="export-2"></div>
        </template>

    </head>

    <body>

        <div template="template1">
            <div export="export-1" id="headline" style="color:red"></div>
            <!-- <import name="export-1" id="headline" style="color:red">Default Headline</import> -->
        </div>

        <div export="export-1" style="color:blue"></div>
        <!-- <import template="template1" name="export-1" style="color:blue"></import> -->

    </body>

</html>
```

**Now on the Browser**
Find and delete the server-slotted element with ID `#headline`. The original `<import>` element should now be restored and ready to be replaced the next time composition takes place.

```html
<html>

    <head>

        <template name="template2">
            <div export="export-1"></div>
            <div export="export-2"></div>
        </template>

    </head>

    <body>

        <div template="template1">
            <import name="slot-1" id="headline" style="color:red">Default Headline</import>
            <!-- <import name="slot-1" id="headline" style="color:red">Default Headline</import> -->
        </div>

        <div import="slot-1" style="color:blue"></div>
        <!-- <import template="template1" name="slot-1" style="color:blue"></import> -->

    </body>

</html>
```

**Enabliing Import Serialization**
Since slots serialization is only necessary for isomorphic pages, this feature is designed to be explicitly turned on on the [OOHTML meta tag](../the-oohtml-meta-tag/README.md).

```html
<html>
    <head>
        <meta name="oohtml" content="isomorphic=true;" />
    </head>
    <body></body>
</html>
```

## Polyfill Support
The current [OOHTML polyfill implementation](../installation/README.md) has full support for HTML Partials. The polyfill additionally makes it possible to customise much of its implementation of the syntax using the [OOHTML meta tag](../the-oohtml-meta-tag/README.md). Much of this customization is done at the [Named Templates](../named-templates/README.md#polyfill-support) level. The following are additional areas of customization:

+ **[tag.import](#templates-imports-and-exports)** - The tag name for defining the import element. The standard element name is `import`, but this implementation currently only works with a custom element, which is `wq-import` by default. You may specify a different element name where necessary.
        
    ```html
    <head>
        <meta name="oohtml" content="tag.import=oohtml-import;" />
    </head>
    <body>
        <oohtml-import name="export-1" template="template2"></oohtml-import>
    </body>
    ```

Learn more about customization and the OOHTML meta tag [here](../the-oohtml-meta-tag/README.md).