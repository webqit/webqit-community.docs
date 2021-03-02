# An SPA In OOHTML

This example makes a Single Page Application (SPA) of [*HTML Imports*](../html-imports).

Below, we're using the two `<template>` elements to each represent a route - a page. Then we point the `<body>`'s template attribute to either of the `<template>`s, depending on the current URL.

```html
<html>

    <head>

        <title>An SPA In OOHTML</title>
        <meta name="oohtml" content="element.import=import" />
        
        <script src="https://unpkg.com/@webqit/oohtml/dist/main.js"></script>

        <template name="pages">

            <!-- "home" page module -->
            <template name="home">
                <h1 exportgroup="headline">
                    Welcome Home!
                </h1>
                <p exportgroup="content">
                    <a href="#/about">About Me</a>
                </p>
            </template>

            <!-- "about" page module -->
            <template name="about">
                <h1 exportgroup="headline">
                    About Me!
                </h1>
                <p exportgroup="content">
                    <a href="#/home">Back to Home</a>
                </p>
            </template>

        </template>

    </head>

    <body template="pages/home">

        <header></header>

        <main>
            <div>
                <import name="headline">404</import>
            </div>
            <div>
                <import name="content">Page not Found!</import>
            </div>
        </main>
 
        <footer></footer>

        <script>
            window.addEventListener('popstate', e => {
                let path = document.location.hash.substr(1);
                document.body.setAttribute('template', 'pages' + path);
            });
        </script>
    </body>

</html>
```

Navigate to a route that does not begin with `#/home` or `#/about`, you should see the default content showing *404*.

[Check the live demo here](//webqit.io/tooling/.docs/oohtml/examples/spa/.demos/index.html) or copy and paste the code in a blank HTML page and view in your browser.
