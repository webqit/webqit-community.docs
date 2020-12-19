# OOHTML's Relationship With Other Technologies
Here are a few things that OOHTML may relate with in some way. We will be adding more as we discover them. And you may help out by [filing an issue](https://github.com/webqit/oohtml/issues).

This information is being gathered to help everyone better understand the problem domain of this technology and to provide hints to some of its design and architectural decisions. And when we meet at the [WICG](https://discourse.wicg.io/t/proposal-chtml/4716), we have good focus areas for consideration.

## [The Observer API](../the-observer-api/README.md)
+ Mimicks the depreciated **[`Object.observe()`](https://arv.github.io/ecmascript-object-observe)** API and JavaScript's reflection APIs like **[`Reflect`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect)**.
+ Offers more general-purpose *interceptibility* than JavaScript's **property [setters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/set) and [getters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get)**, and JavaScript [Proxies](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) which specifically obscure object identity.

## [Namespaced HTML](../namespaced-html/README.md)
+ Captures the problem that the **[BEM](https://getbem.com)** naming convention tries to solve but addresses it at the language level.
+ Relates with **[Stuart P.'s Parts and Walls](https://github.com/stuartpb/pwalls-spec)** proposal from 2015.

## [Named Templates](../named-templates/README.md)
+ Defines the **src** attribute that is also being considered [here](https://discourse.wicg.io/t/add-src-attribute-to-template/2721) and [here](https://github.com/whatwg/html/issues/2791), and probbably elsewhere, but goes further in scope with its concept of *observability* using `templatecontentloaded` event and other features.
+ Defines the **document.templates** API that is also being proposed [here](https://discourse.wicg.io/t/document-templates/1057), but goes further in scope with its `HTMLTemplateElement.prototype.templates` API and other features.

## [Scoped Scripts](../scoped-scripts/README.md)
+ Captures the problem that **UI binding languages** try to solve but comes bringing the full power of JavaScript for the job. Compare [this early idea for a template syntax by Jonathan Kingston](https://discourse.wicg.io/t/extension-of-template/447) from 2014, [this proposal](https://github.com/whatwg/html/issues/2254) from 2017, and [Apple's proposal](https://github.com/w3c/webcomponents/blob/gh-pages/proposals/Template-Instantiation.md) from 2017.

## [The State API](../the-state-api/README.md)
+ Reminds us of **[HTML5's DataSet API](https://developer.mozilla.org/en-US/docs/Web/API/HTMLOrForeignElement/dataset)**. But the DataSet API turns out to be simply an API to HTML's custom data attributes `(data-*)` with its one-to-one mapping to `data-*` attribute making it quite less useful for managing application state. In contrast, the new State API gives us a robust set of *state management* APIs - covering both document-level and element-level state, and the concept of automatic observability.

