# Tooling In OOHTML

This example shows how we could use a DOM abstraction library, like jQuery, from within Subscript code.

Turns out that this is naturally possible!

```html
<html>

    <head>
        <title>Tooling In OOHTML</title>
        <meta name="oohtml" content="attr.id=id" />
        
        <script src="https://unpkg.com/@webqit/oohtml/dist/main.js"></script>
        <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
    </head>
        
    <body>

        <div id="alert" namespace>
            <div id="message"></div>
            <script type="subscript">
                $(this.namespace.message).html(this.state.message || 'Task pending...');
            </script>
        </div>

        <script>
            // The alert
            setTimeout(() => {
                document.querySelector('#alert').setState({
                    message: 'This task is now complete!',
                });
            }, 3000);
        </script>

    </body>

</html>
```

Tooling can also help us acheive more efficient DOM manipulation. Generally, surgically updating the DOM may have performance implications on the UI, as arising from layout thrashing (see [this article](https://developers.google.com/web/fundamentals/performance/rendering/avoid-large-complex-layouts-and-layout-thrashing) on Web Fundamentals). But we also don't need as much as a *Virtual DOM* for this. A technique like that of [fast DOM](https://github.com/wilsonpage/fastdom) could just suffice.

This technique is natively implemented by the [PlayUI](https://webqit.io/tooling/play-ui) library which has a jQuery-like API. We will now use PlayUI as a drop-in replacement for jQuery in the code above.
 
```html
<html>
    
    <head>
        <title>Tooling In OOHTML</title>
        <meta name="oohtml" content="attr.id=id" />
        
        <script src="https://unpkg.com/@webqit/oohtml/dist/main.js"></script>
        <script src="https://unpkg.com/@webqit/play-ui/dist/main.js"></script>
    </head>
            
    <body>

        <div id="alert" namespace>
            <div id="message"></div>
            <script type="subscript">
                // The .html() method is asynchronous
                $(this.namespace.message).html(this.state.message).then(() => {
                    // Do something sync
                });
            </script>
        </div>

        <script>
            // Make PlayUI available globally
            window.$ = window.WQ.$;

            // The alert
            setTimeout(() => {
                document.querySelector('#alert').setState({
                    message: 'This task is now complete!',
                });
            }, 3000);
        </script>

    </body>

</html>
```

[Check the live demo here](//webqit.io/tooling/.docs/oohtml/examples/tooling/.demos/index.html) or copy and paste the code in a blank HTML page and view in your browser.
