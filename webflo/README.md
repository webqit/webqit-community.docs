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
Webflo is a command-line tool for developing and running your application. It is recommended you install it globally on your machine.

```bash
npm install -g @webqit/webflo
```

We can now use the `webflo` command from any location in the terminal. To test, run `webflo help`; an overview of available commands will be shown.

Webflo is so simple to use that you do not need any starter tool like `create-webflo-app`. It's strengths actually lie in letting you hand-craft your app!

## Overview
Every project starts on an empty directory that you can create anywhere on your machine. And every file you will create here is going to be all about your application; Webflo doesn't even make a footprint on your project.

Just for the examples below, create a directory named `webflo-app` and navigate there on your terminal:

```bash
mkdir webflo-app
cd webflo-app
```

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

So, route handlers can both return response data of their own and act as a gateway for the request/response flow. As we will see shortly, `object` responses returned from route handlers can either serve as *automatic JSON (API) responses* or get rendered as *HTML responses*.

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

Now, what happens is, when you navigate to, or try to navigate away from `http://localhost:3000/` (or `http://localhost:3000/index.html`) on your browser, this client-side route handler is hit first with the HTTP request. It then decides to either return an in-browser *response data* or simply allow the request to *flow* to the server - all while preventing the browser from reloading the current page.

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

Now, what happens is, when you navigate to, or try to navigate away from `http://localhost:3000/` (or `http://localhost:3000/index.html`) on your browser, *and the request passes the initial client-side routing layer, where exists*, this worker-level route handler is hit next with the HTTP request. It then decides to either return an in-browser *response data* or simply allow the request to *flow* to the server.

Worker-level routing lies between the client-level routing and the server-side routing. This middle routing layer is especially helpful when you're really creating a deep offline experience on your app that client-level routing alone does not satisfy.

> There are great usecases. For example, requests that are not initiated by the user cannot be caught by client-level routers but can be caught by worker-level routers. Worker-level routing is covered in [this tutorial](learn/worker-level-routing).

### Routing
As seen, Webflo lets you follow the traditional filesystem layout for your project. The concept of routing is simply drawn on this layout. It is all about the *request/response flow and the path it takes*. Webflo's skillfulness with *flows* is really the meaning of its name.

#### Layered Routing
The project layout above lets you place route handlers at a desired point along a *vertical* request/response flow between the client and the server. Here's that layout now in the order of flow.

+ `/client` - *where implemented, requests are either handled here or forwarded down*
+ `/worker` - *where implemented, requests are either handled here or forwarded down*
+ `/server` - *where implemented, requests are either handled here or forwarded down*
+ `/public` - *where implemented, requests are automatically mapped to static file responses*

The type of application you're building will determine where you choose to implement routing. It could be just client-side routing, just server-side routing, fullstack routing or any combination of it.

#### Step Routing
Webflo also lets you follow a layout pattern that maps URL paths to filesystem paths. You'd expect this behaviour for static files laid out in the `/public` directory. For example, the request URL `/` expects to find a file at `/public/index.html`, and the request URL `/products` expects to find a file at `/public/products/index.html`, and so on. (URLs with filenames, like `/css/main.css`, also work the same; i.e, expects to find a file at `/public/css/main.css`.)

Webflo draws on this traditional pattern and lets you lay out route handlers *horizontally* along URL paths this way. For example, if you're doing server-side routing, the request URL `/` expects to hit a route handler at `/server/index.js`, and the request URL `/products` expects to hit a route handler at `/server/products/index.js`, and so on. But an import concept in this horizontal layout is *step routing*, which lets a request *flow* through every handler in the route path until it hits the final handler. That means that the request URL `/products` is going to flow this way:

* `/server/index.js` --> `/server/products/index.js`

Step routing is the best way to orchestrate routes.
