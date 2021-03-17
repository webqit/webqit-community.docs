---
icon: app-indicator
desc: The universal JavaScript framework for Web, Mobile, and API Backends; one tool for every step of the way - from development, to deployment, to continous delivery.
desc2: Start on a clean slate, in zero-abstraction, zero-config, and zero-dependency. Develop and deploy anything from a simple <code>'Hello World!'</code> to a rich web and mobile experience.
categories: Featured, Frameworks
tags: Web, Mobile, API Backends, Dev Server, Production Server, GIT Workflow
_after: oohtml
---
# Webflo

<!-- BADGES/ -->

<span class="badge-npmversion"><a href="https://npmjs.org/package/@webqit/webflo" title="View this project on NPM"><img src="https://img.shields.io/npm/v/@webqit/webflo.svg" alt="NPM version" /></a></span>
<span class="badge-npmdownloads"><a href="https://npmjs.org/package/@webqit/webflo" title="View this project on NPM"><img src="https://img.shields.io/npm/dm/@webqit/webflo.svg" alt="NPM downloads" /></a></span>

<!-- /BADGES -->

[Webflo](https://github.com/webqit/webflo) is the universal JavaScript framework for Web, Mobile, and API Backends; one tool for every step of the way - from development, to deployment, to continous delivery.

*Start on a clean slate, in zero-abstraction, zero-config, and zero-dependency. Develop and deploy anything from a simple `'Hello World!'` to a rich web and mobile experience.*

> [Visit project repo](https://github.com/webqit/webflo).

## Prerequisites
Webflo is a command-line tool for developing and running your application. We recommended you install it globally on your machine.

```bash
npm install -g @webqit/webflo
```

We can now use the `webflo` command from any location in the terminal. To test, run `webflo help`; an overview of available commands will be shown.

Webflo is so simple to use that you do not need any starter tool like `create-webflo-app`. It's strengths actually lie in letting you hand-craft your app!

## Concepts
Every project starts on an empty directory that you can create anywhere on your machine. And every file you will create here is going to be all about your application; Webflo doesn't even make a footprint on your project.

Just for the examples below, create a directory named `webflo-app` and navigate there on your terminal:

```bash
mkdir webflo-app
cd webflo-app
```

We should now learn the following concepts on which Webflo is built:

+ [Project Layout](#project-layout)
+ [Routing](#routing)
+ [HTTP Requests and Responses](#http-requests-and-responses)

### Project Layout
It's a good practice to locate certain project files in conventional places. Webflo is thus able to automatically identify them at runtime. Here's an overview (keep in mind that everything below is optional, and/or can be renamed):

+ `/public`
+ `/server`
+ `/client`
+ `/worker`

Let's now see what goes into each directory, and how they're all related.

#### The `/public` Directory
If you intend to have static files (like images or CSS files) that should be served automatically, place them in this directory.

Your application's start page - `index.html` - should also be here.

+ `/public`
    + `/index.html` - *Try add some Hello World greeting.*

Now, when you start the Webflo server and navigate to `http://localhost:3000/` (or `http://localhost:3000/index.html`) on your browser, the start page is shown.

```bash
webflo start
```

> Ensure that you are at your project root `/webflo-app` in the terminal to run this and subsequent Webflo commands.

> Now, if all you're creating is a static site, your work ends in this directory! Static file serving is covered in [this tutorial](learn/static-files).

#### The `/server` Directory
If you intend to have JavaScript files that handle dynamic routing on the server, place them in this directory.

+ `/server`
    + `/index.js` - *This is a server-side route handler.*

Now, what happens is, when you navigate to `http://localhost:3000/` (or `http://localhost:3000/index.html`) on your browser, the route handler in `index.js` is hit first with the HTTP request. It then decides to either return a certain *response data* or simply allow the request to *flow* to the intended `index.html`.

So, route handlers can both return response data of their own and act as a gateway for the request/response flow. As we will see shortly, response data returned from route handlers can either serve as *automatic JSON (API) responses* or get rendered into an HTML file and return as an *HTML responses*.

> Now, if all you're creating is a traditional server-side application or simply an API backend, your work ends in this directory! Server-side routing is covered in [this tutorial](learn/server-side-routing).

#### The `/client` Directory
If you intend to have JavaScript files that handle routing in the browser, place them in this directory.

+ `/client`
    + `/index.js` - *This is a client-side route handler.*

Next, run a Webflo command that automatically builds these files into a script that you can include on your page.

```bash
webflo build
```

> Client builds are covered later on. But let's assume for now that the generated JavaScript file is now linked to in the HTML page.

Now, what happens is, when you navigate to, or try to navigate away from `http://localhost:3000/` (or `http://localhost:3000/index.html`) on your browser, this client-side route handler is hit first with the HTTP request. It then decides to either return an in-browser *response data* or simply allow the request to *flow* to the server - all while preventing the browser doing a page reload.

As we will see, being able to either return an in-browser response data or act as a gateway for the request/response flow is a powerful way to create fast and smooth client-side experiences.

> If all you're creating is a client-side application, your work ends in this directory! Client-side routing is covered in [this tutorial](learn/client-side-routing).

#### The `/worker` Directory
What happens here is quite advanced and you can ignore this until you really need it. But if you already know about application Service Workers and intend to implement routing at the service-worker level, place your route handlers in this directory.

+ `/worker`
    + `/index.js` - *This is a worker-level route handler.*

Next, run a Webflo command that automatically builds these files, *along with any client-side routers* above, into a script that you can include on your page.

```bash
webflo build
```

> Client builds are covered later on. But let's assume for now that the generated JavaScript file is now linked to in the HTML page.

Now, what happens is, when you navigate to, or try to navigate away from `http://localhost:3000/` (or `http://localhost:3000/index.html`) on your browser, *and the HTTP navigation request is passed on from the initial client-side routing layer, where exists*, this worker-level route handler is hit next with the request. It then decides to either return an in-browser *response data* or simply allow the request to *flow* to the server.

Worker-level routing lies between the client-level routing and the server-side routing. This middle routing layer is especially helpful when you're really creating a deep offline experience in your application that client-level routing alone does not satisfy. For example, certain requests are not initiated by the end user and cannot be caught by client-level routers. These requests can only be caught by worker-level routers.

> Worker-level routing is covered in [this tutorial](learn/worker-level-routing).

### Routing
As seen, Webflo lets us follow the traditional filesystem layout for a project. The concept of routing is simply drawn on this layout. It is all about the *request/response flow and the path it takes*. Webflo's skillfulness with *flows* is probabbly the best thing about its name.

#### Layered Routing
The project layout above lets us place route handlers at a desired point along a *vertical* request/response flow between the client and the server. Here's that layout now in the order of flow.

+ `/client` - (where implemented) *requests are either handled here or forwarded down*
+ `/worker` - (where implemented) *requests are either handled here or forwarded down*
+ `/server` - (where implemented) *requests are either handled here or forwarded down*
+ `/public` - (where implemented) *requests are automatically mapped to static files*

As seen in the [Project Layout](#project-layout) section above, the type of application you're building will determine where you choose to implement routing. It could be just client-side routing, just server-side routing or fullstack routing in any combination of it, as we will see soon.

#### Step Routing
Webflo also lets us follow a layout pattern that maps URL paths to filesystem paths. We'd expect this behaviour for static files laid out in the `/public` directory. For example, the request URL `/` would be expected to find a file at `/public/index.html`, and the request URL `/products` would be expected to find a file at `/public/products/index.html`, and so on. (URLs with filenames, like `/assets/main.css`, would also be expected to work the same; i.e, find a file at `/public/assets/main.css`.)

Webflo draws further on this convention to let us lay out route handlers *horizontally* along URL paths. For example, if you're doing server-side routing, the request URL `/` would be mapped to a route handler at `/server/index.js`, and the request URL `/products` would be mapped to a route handler at `/server/products/index.js`, and so on. But an important concept in this horizontal layout is *step routing*, which is the idea of making a request *flow* through every handler in the route path until it hits the final handler. That means that the request URL `/products` would actually flow like this:

+ -> enter `/server`
    + -> enter `/index.js`
    + -> enter `/products`
        + -> enter `/index.js`

As you will see, step routing is the most-empowering way to orchestrate routes.

#### Route Handlers
Route handlers are `index.js` files that are laid out in the routing directory to handle the application's request/response flow. The most important content of these files are a simple function that is exported as the *default export* of the file.

```js
export default function(flo, recieved, next) {
}
```

This function is what is called when a request flow hits a route.

Of the three parameters of a route handler, the `next` parameter is what it uses to forward a request, if it chooses to. This is the concept of flow control.

##### Flow Control
Given the following route hierarchy...

+ `/server`
    + `/index.js`
    + `/products`
        + `/index.js`

Here is how our route handlers could look:

`file: /server/index.js`

```js
export default function(process, recieved, next) {
    if (next.pathname) {
        return next();
    }
    return { title: 'Hello World', };
}
```

`file: /server/products/index.js`

```js
export default function(process, recieved, next) {
    if (next.pathname) {
        return next();
    }
    return { title: 'Our Products', };
}
```

In the first handler, we started by asking if the route path has another step ahead. For a path like `/products`, the answer is yes, and the flow is forwarded to the next handler in the path.

In the second handler, we again started by asking if the route path has another step ahead. This time, the answer is no, and *reponse data* is returned here for the request.

Flow control is especially important for root-level handlers, i.e handlers that handle the application's root URL `/`, since they act as the *gateway* to all of the application's routes - both static and dynamic routes.

Given the following layout...

+ `/server`
    + `/index.js`
+ `/public`
    + `/index.html`
    + `/assets`
        + `/main.css`

The request URL `/index.html` would flow this way:

+ -> enter `/server`
    + -> enter `/index.js`; test `next.pathname`: `'index.html'`; then `next()`
+ -> enter `/public`
    + -> serve `/index.html`

While, the client request URL `/` would flow this way:

+ -> enter `/server`
    + -> enter `/index.js`; test `next.pathname`: `''`; return `{ title: 'Hello World', }`

This way, *static file URLs* are properly allowed at this critical point in the application's URL-handling. Also noteworthy is that the *static file URL*  `/index.html` is still handled differently from the *path URL* `/`. Thanks to `next.pathname` and `next()`.

> As we would expect, if we had no route handlers intercepting the request, both URLs - `/index.html` and `/` - would be mapping directly to `/public/index.html`.

> Try figuring out on your own the flow for the URL `/assets/main.css`.

As a general rule, it is good to always use `next.pathname`and `next()` to properly manage the flow, even when writing the last handler for a given route hierarchy. There are many important advantages to this:

+ Given an earlier example, while the last handler in our route hierarchy is designed for the URL `/products`, our flow control code in there would make an unexpected, extended URL like `/products/specials` trigger the `next()` function; and since nothing like that really exists ahead, an empty respone would be returned, which Webflo gracefully translates into a `404` HTTP response. Without that check in place, an invalid URL would be returning a *falsely-valid* response.
+ Furthermore, if, or when, we do eventually place a handler there for that level of the route path, our flow control code in the current handler would have already ensured that requests can flow through to the new level.
+ Or, where another routing layer exists below the given routing layer, our flow control code in this last handler for the route would be forwarding those requests to the next layer.

    In the layout below, the request URL `/products/specials` that is non-existent in the upper routing layer is actually existent in the lower routing layer.

    + `/server`
        + `/index.js`
        + `/products`
            + `/index.js`
    + `/public`
        + `/products`
            + `/specials`
                + `/index.html`

    Calling `next()` at the last handler for the URL will forward the flow off the `/server` routing layer into the `/public` routing layer, and the HTML file is served for the URL.

    Here is how that would flow:
        
    + -> enter `/server`
        + -> enter `/index.js`; test `next.pathname`: `'products/specials/index.html'`; then `next()`
        + -> enter `/products`
            + -> enter `/index.js`; test `next.pathname`: `'specials/index.html'`; then `next()`
    + -> enter `/public`
        + -> serve `/products/specials/index.html`

    > A side benefit we've enjoyed with this `products/specials` URL is the convenience of having it begin life as a static route until we can make it dynamic by creating a route handler for it.

##### Parameter Passing
While being used to forward a request, the `next()` function can help to pass on anything to the next handler.

`file: /server/index.js`

```js
export default function(process, recieved, next) {
    if (next.pathname) {
        return next({ userId: 1 });
    }
    return { title: 'Hello World', };
}
```

`file: /server/products/index.js`

```js
export default function(process, recieved, next) {
    if (next.pathname) {
        return next(recieved);
    }
    if (recieved.userId) {
        // Show recommended products
        return { title: 'Recommended Products For You', };
    }
    return { title: 'Our Products', };
}
```

In the first handler, we passed an object via the `next()` function. In the second handler, we recieved it on the `recieved` parameter.

Parameter passing is a great way to implement one source of truth for subsequent handlers in the route hierarchy. Think authentication, certain database or external API queries, etc. Doing these at the earliest level in the route hierarchy and passing tokens or objects down is a perfect way to avoid repeating code down the hierarchy.

##### The `process` Object
When called, route handlers recieve very useful information about the ongoing HTTP process. This and a few other metadata are passed together as an object into the first parameter of the handler - the `process` parameter.

### HTTP Requests and Responses