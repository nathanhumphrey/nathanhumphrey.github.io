---
title: Full Stack JavaScript Application Build Part 1 - Routes and Pages
description: Part 1 of a series about how to build a full stack JavaScript application. This article is about routes and pages.
abstract: I've come across a lot of articles online about how to build a front-end application or how to build a back-end REST API, but rarely do I find articles dedicated to building out a full application (both front and back), no less an isomorphic JavaScript application. So, I've decided to write the series that I was always hoping to come across.
date: 2020-02-06 7:00:00.00
tags:
  - software
  - tutorial
  - javascript
  - fullstack
  - koa
  - react
layout: layouts/post.njk
---

_Learning about what goes into building a full-stack JavaScript application can be a little daunting, but with some grit and perseverance, it can be a really rewarding experience. In this series, I'll be building a very simple isomorphic JavaScript application with some extra bells and whistles (e.g. routing, CSRF tokens, authentication, production bundling, etc.). If that all sounds interesting to you, then come along for the ride!_

## Introduction

I've come across a lot of articles online about how to build a front-end application or how to build a back-end REST API, but rarely do I find articles dedicated to building out a full application (both front and back), no less an isomorphic JavaScript application. So, I've decided to write the series that I was always hoping to come across.

**_The final project source can be found in this [GitHub repo](https://github.com/nathanhumphrey/simple-isomorphic-app/tree/routes)_**

### The Application

To keep things simple, the application we'll build won't do anything special, and by anything special I mean nothing. I've read through a lot of articles that try to teach basic, or even more advanced, concepts by building a complex application, which I find distracts from the core of what was being covered. I just want to focus on the chosen topic with as little _application_ logic as necessary to explore the details of the topic. No more, and no less.

To that end, I plan on building an application that allows a user to sign up for an account, sign in and out of their account, and to view their account details (when signed in of course). That's it. With just these simple use cases, we can explore all the relevant bits that are required when creating a standard web application without getting lost in the details of the application logic. Effectively, when we're done, you'll have a boilerplate application setup that supports user accounts for whatever specific application you'd like to build next.

All that being said, there's nothing stopping you from making the build a little more exciting by adding additional pages, components, and logic if you feel so inclined. You may even decide to work through the series once, write down some 'what if' questions, and then come back around for another walkthrough while exploring the 'what ifs' you noted the first time. Do whatever will make this journey more fun for you.

### Roadmap

The build will be divided into a six-part series:

1. [Routes and pages](/posts/2020-02-06-full-stack-javascript-app-build-pt-1/) ⇐ you are here
2. [Building components](/posts/2020-02-12-full-stack-javascript-app-pt-2/)
3. [Styling components](/posts/2020-03-24-full-stack-javascript-app-pt-3/)
4. Database access and authentication
   1. [Backend implementation](/posts/2020-04-03-full-stack-javascript-app-pt-4-phase1/) ⇐ you are here
   2. ~~Frontend implementation~~ (coming soon)

I've also included some extra stretch goals as well (should the first six parts go smoothly):

- Protecting against CSRF
- Cache busting
- Code splitting
- Progressive enhancement
- Testing
- Production bundling

<p class="info">
NOTE: there are many articles published about how to implement most of the topics in the stretch goal list. Please feel free to use this project as a base for experimenting with other walkthroughs.
</p>

Get started by working through my earlier [Isomorphic JavaScript App](/posts/2020-01-17-isomorphic-javascript-app/) post first, which builds the foundation that will be used in this series. If you'd just like to get started with this series, you can download the [starting code](https://github.com/nathanhumphrey/simple-isomorphic-app).

## Routes and Pages

Our application will need to support the following use cases:

- Sign up for an account
- Sign-in
- Sign-out
- View account details

A page will be needed to support each of these use cases, and routes will need to be configured to support them as well. Sample flow through the application may look something like the following:

Home Page ⇒ Sign Up ⇒ Sign In ⇒ View Account ⇒ Sign Out

### Endpoints

Some of the actions noted above can be supported on a single page. I've chosen the following routes and pages for us to build on:

#### Index route `/`

The index (or home) page can be used to provide a sign-in form (when not already signed in) and a sign-out link (when already signed) for the user.

#### Sign up route `/signup`

The signup page will present a basic form for gathering account details (name, email, password, etc.) from the user.

#### View account route `/users/:id`

The user page will present the currently signed in user's details. Don't mind the `:id` in the route, it simply represents a variable paramter that would be included in the URL (e.g. /users/1 or /users/jdoe).

### Implementation

The implementation of pages and routes, as we've identified them, requires that they work both on the client and the server. Every attempt will be made to reduce duplication (as much as is possible) while at the same time retaining good maintainability; changing or adding a page shouldn't require a cascade of changes throughout the application.

Since the routes are at the heart of this application, we should utilize a shared `routes.js` file that can be used to define routes everywhere we need them. The routes can be imported and then implemented for the server and for our React application. When a new page is added, we can simply define the route in our routes file.

#### Routes

Initially, our application will need to know a few things about each route:

- the path (URL path to match)
- the HTTP method (HTTP verb to accept)
- the route name (unique name to identify the route)

Finally, time to code! Since the routes are shared by both the server and client, we will create the `routes.js` file in the `src/app/` directory. Once created, open the file and export an array of routes:

```js
export const routes = [
  {
    path: "/",
    method: "get",
    name: "home",
  },
  {
    path: "/signup",
    method: "get",
    name: "signup",
  },
  {
    path: "/users/:id",
    method: "get",
    name: "userdetails",
  },
  {
    path: "*",
    method: "get",
    name: "nomatch",
  },
];
```

<p class="caption">routes for the application - src/app/routes.js</p>

<p class="info">
NOTE: I've also included a <i>nomatch</i> route, for any non-matched paths.
</p>

The array of routes in `routes.js` can now be imported into our `src/server/index.js` and used to direct requests, as well as in `App.js` to direct requests on the client. To make it easier for us to complete these two tasks, we'll make use of available packages: [koa-router](https://github.com/ZijianHe/koa-router) for the server and [react-router-dom](https://reacttraining.com/react-router/web/guides/quick-start) for the client.

Install the required packages:

<code class="term">npm i koa-router react-router-dom</code>

Now that we've installed the requirements, let's put them to use.

##### Server Routes

Using koa-router is fairly straight-forward. First, we start by creating a new `Router` object and then assign routes and handlers as necessary. Defining routes can be done with HTTP verb specific method calls or by calling the `use()` method; I've chosen to use the verb method route (pun intended) and will leave the exploration of the `use()` method to you.

As a quick example, adding the route for our index page may look something like the following:

```js
import Router from "koa-router";

const router = new Router();

router.get("/", (ctx, next) => {
  ctx.body = "requested the index route";
  next();
});

app.use(router.routes()).use(router.allowedMethods());
```

<p class="caption">exmple using koa-router</p>

In the example above, only get requests for the `/` path will be matched by the router. In response, the body of the context, which will be returned to the client, simply contains a string message to display. Let's implement this basic example before applying the routes we defined above.

Open and update `src/server/index.js` as follows:

```js/2,10-17,22-24
...

import Router from 'koa-router'; // server-side routing

// create the app server
const app = new Koa();

// prepare webpack compiler
const compiler = webpack(config);

// create the app router
const router = new Router();

// define route(s)
router.get('/', (ctx, next) => {
  ctx.body = 'requested the index route';
  next();
});

// create koa webpack middleware
koaWebpack({ compiler }).then(middleware => {
  // use the koa middleware
  app.use(middleware);
  // removed the renderReactApp middleware, use router going forward
  app.use(router.routes()).use(router.allowedMethods());
});

...

```

<p class="caption">update server/index.js to support example route - src/server/index.js</p>

Now, run the `serve` script and navigate to [http://localhost:3000/](http://localhost:3000/) in your browser. You should see the message we assigned to `ctx.body` displayed in the browser. And now we have a server-side router!

<code class="term">npm run serve</code>

<p class="info">NOTE: only matched routes will be returned. If you navigate to a path that hasn't been defined as a route, you will receive a 404 not found response.</p>

<aside>
<p>
    TIP: To make development a little easier, let's make a quick update to start the application using <a href="https://nodemon.io/">nodemon</a>. Install the `nodemon` package
</p>

<code class="term">npm i -D nodemon</code>

<p>
    then update the serve script in package.json:
</p>

<pre class="language-json">
<span class="highlight-line">...</span>
<span class="highlight-line highlight-line-active">"serve": "nodemon index.js"</span>
<span class="highlight-line">...</span>
</pre>

<p class="caption">update serve script - package.json</p>

<p>
Now, changes to the server code will be automatically reloaded and we won't have to stop and start the process whenever we make some changes. To see a more detailed setup using webpack and webpack DevServer, see this <a href="/posts/2019-12-21-simple-webpack-setup/#development-server">post</a>.
</p>

</aside>

Let's update the code to work with our defined routes. We can simply iterate over all routes in the array and apply a route to our koa-router for each one. Make the following updates to `src/server/index.js`:

```js/3,15-20
...

import Router from 'koa-router'; // server-side routing
import { routes } from '../app/routes'; // defined application routes

// create the app server
const app = new Koa();

// prepare webpack compiler
const compiler = webpack(config);

// create the app router
const router = new Router();

// define route(s)
routes.forEach(route => {
  router[route.method](route.path, (ctx, next) => {
    ctx.body = route.name;
    next();
  });
});

// create koa webpack middleware
koaWebpack({ compiler }).then(middleware => {
  // use the koa middleware
  app.use(middleware);
  // removed the renderReactApp middleware, use router going forward
  app.use(router.routes()).use(router.allowedMethods());
});

...

```

<p class="caption">update server/index.js to implement defined routes - src/server/index.js</p>

<p class="info">
NOTE: There is a lot of versatility in the way in which we've implemented our routes. Looking to the future, we can easily extend the route objects in `routes.js` to define whether they require authentication, should be included in page navigation, require initial data, and so on.
</p>

The current handler for each route is just an arrow function, for the moment. What we really want is for our routes to be rendered by our react render middleware. Update the `src/server/index.js` one final time before we move on to defining pages:

```js/4
...

// define route(s)
routes.forEach(route => {
  router[route.method](route.path, renderReactApp);
});

...

```

<p class="caption">use react renderer for all defined routes - src/server/index.js</p>

The problem now is that all the routes end up getting rendered as the default app (currently a sample clicker app). We need to define a few things for our application to render correctly: separate pages for each route and a client-side router that knows which page to render based on the current route.

#### Pages

Let's begin by defining a `Page` component that can be reused for all pages we'd like to render for the client.

Create a `components` directory under `src/app/` and then create a new file, `Page.js`, in the components directory. Open `Page.js` and insert the following code:

```js
import React from "react";

const Page = ({ children }) => {
  return (
    <div className="page-container">
      <main>{children}</main>
    </div>
  );
};

export default Page;
```

<p class="caption">Page component - src/app/components/Page.js</p>

This component is pretty plain at the moment. It simply provides exactly what the name implies, a page that we can extend for our application. Any children defined within the Page component will simply be rendered. Let's put it into action by creating our first page.

Create a `pages` directory under `src/app/` and then create a new file, `home.js`, in the pages directory. Open `home.js` and insert the following code:

```js
import React from "react";
import Page from "../components/Page";

const HomePage = () => {
  return (
    <Page>
      <h1>Home Page</h1>
      <p>Home page content will appear here.</p>
    </Page>
  );
};

export default HomePage;
```

<p class="caption">Home page component - src/app/pages/home.js</p>

Okay, we have a single page that we can use for testing purposes. Open `src/app/App.js` and let's make some updates to support routing and our new home page. Remove all existing code in the `App` function and add the following:

```js/1,4
import React from "react";
import HomePage from "./pages/home";

const App = () => {
  return <HomePage />;
};

export default App;
```

<p class="caption">Home page rendered from App - src/app/App.js</p>

Now all routes will result in the home page being rendered, progress. Let's add the required routing pieces for the App.

<p class="info">
NOTE: the remaining two pages will be defined essentially the same way as this home page (i.e. simply rendering a message of some kind). See if you can create the two remaining pages on your own; the final repo will have all three pages completed for you to review.
</p>

#### App routes

The `App` component is being shared by both the server and client, which means we need to address routing for both cases. This can be achieved by defining the routes themselves within the `App` component and then encapsulating the `App` routes with the proper router provider, so to speak. The `react-router-dom` package provides a means to achieve our goals.

```js/1,5-10
import React from "react";
import { Route, Switch } from "react-router-dom";
import Home from "./pages/home";

const App = () => {
  return (
    <Switch>
      <Route exact path="/">
        <Home />
      </Route>
    </Switch>
  );
};

export default App;
```

<p class="caption">App updated to support routing - src/app/App.js</p>

Now our App implements a Switch, that will only render one route. The Switch component will render the first Route that matches the path provided. This is only the first step we need to make to get this App routing working. Now, we need to implement the server-side router and the client-side router.

##### Server-side App routes

Server-side rendering of our application is being done in the `src/server/render-react-app.js` file; open the file and make the following udpates:

```js/2,5,16,18
import React from "react";
import { renderToString } from "react-dom/server";
import { StaticRouter } from "react-router-dom";
import App from "../app/App";

const staticContext = {}; // used to store info about the render

export default (ctx) => {
  const renderComponent = (
    <html>
      <head>
        <title>Isomorphic React App</title>
        <script src="index.js" defer />
      </head>
      <body>
        <div id="app">
          <StaticRouter location={ctx.path} context={staticContext}>
            <App />
          </StaticRouter>
        </div>
      </body>
    </html>
  );

  ctx.body = renderToString(renderComponent);
};
```

<p class="caption">Server-side render updated to support routing - src/server/render-react-app.js</p>

The server will now only render matched paths within the `StaticRouter`. Later, we'll update with our own version of a 404 page for any unmatched paths.

##### Client-side App routes

In much the same way we implemented a router for the server-side render of our app, we must do the same for the client. Open the `src/client/index.js` file and make the following updates:

```js/2,5-9
import React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter } from "react-router-dom";
import App from "../app/App";

ReactDOM.hydrate(
  <BrowserRouter>
    <App />
  </BrowserRouter>,
  document.getElementById("app")
);
```

<p class="caption">client-side routing support added - src/client/index.js</p>

That's it! All routing is setup and should be working. Now, when you run the server and navigate to the index route, you should see the home page content. But, when you navigate to the another supported but not implemented route, you will see ... nothing. That's because we haven't defined routes (or pages at this point) for our remaining pages. Let's create a `NoMatchPage` page we can render when no known paths are matched by the router.

Create a new file under `src/app/pages` named `no-match.js` and insert the following code:

```js
import React from "react";
import Page from "../components/Page";

const NoMatchPage = () => {
  return (
    <Page>
      <h1>Sorry!</h1>
      <p>The page you're looking for can't be found.</p>
    </Page>
  );
};

export default NoMatchPage;
```

<p class="caption">NoMatchPage added for unmatched routes - src/app/pages/no-match.js</p>

We can now add this new page under a no match route in our `App`. Open and edit the `src/app/App.js` file as follows:

```js/3,11-13
import React from "react";
import { Route, Switch } from "react-router-dom";
import HomePage from "./pages/home";
import NoMatchPage from "./pages/no-match";

const App = () => {
  return (
    <Switch>
      <Route exact path="/">
        <HomePage />
      </Route>
      <Route path="*">
        <NoMatchPage />
      </Route>
    </Switch>
  );
};

export default App;
```

<p class="caption">NoMatchPage added for unmatched routes - src/app/pages/no-match.js</p>

With this addition, if you navigate to [http://localhost:3000/signup](http://localhost:3000/signup), you should no longer see a blank screen but instead see our new no match page.

### Upgrading Route Declarations

We can use our `routes.js` file to define the application routes in much the same way we used it for the server. Import the `routes.js` file into `App.js`, iterate over the routes and create a Route component for each one. Open `src/app/App.js` and make the following updates:

```js/3,7-12
import React from "react";
import { Route, Switch } from "react-router-dom";
import HomePage from "./pages/home";
import { routes } from "./routes";

const App = () => {
  return (
    <Switch>
      {routes.map((route) => (
        <Route key={route.page} path={route.path}>
          {/* 
            but what do we put in here now? Can't all be HomePage
            or NoMatchPage!
          */}
        </Route>
      ))}
    </Switch>
  );
};

export default App;
```

<p class="caption">update App.js to make use of defined routes - src/app/App.js</p>

This looks a lot cleaner but, as the embedded comment above states, how do we know what page to render? And what about exact path matches (as it is now, everything will match the first index route)?

Well, we simply add requirements to the route objects! We can extend the routes, as noted earlier, to support whatever we need them to do for us. In this case, let's simply update with an optional `exact` match boolean property, and an optional `pageName` property that indicates which page to use for the route. Update `src/app/routes.js` as follows:

```js/5-6,22
export const routes = [
  {
    path: "/",
    method: "get",
    name: "home",
    page: "HomePage",
    exact: true,
  },
  {
    path: "/signup",
    method: "get",
    name: "signup",
  },
  {
    path: "/users/:id",
    method: "get",
    name: "userdetails",
  },
  {
    path: "*",
    method: "get",
    name: "nomatch",
    page: "NoMatchPage",
  },
];
```

<p class="caption">update routes to support pages and exact path matches - src/app/routes.js</p>

The second thing we'll need to do now is to export all our known pages so they too can be easily accessed from one location. Create an `index.js` in `src/app/pages` and insert the following:

```js
import HomePage from "./home";
import NoMatchPage from "./no-match";

// create 'collection' of pages
const pages = {
  HomePage: HomePage,
  NoMatchPage: NoMatchPage,
};

export { pages };
```

<p class="caption">single access for all application pages - src/app/pages/index.js</p>

<p class="warn">
WARN: the approach taken here will require that you match your page names in <code>src/app/pages/index.js</code> to the names used in <code>src/app/routes.js</code> or they won't match for creating <code>Routes</code>.
</p>

Once this has been completed, import the `pages` object, and we can then filter relevant routes for display in our app by the `page` property and include an `exact` prop for our routes as well. Update `App.js`:

```js/2,8-12
import React from "react";
import { Route, Switch } from "react-router-dom";
import { pages } from "./pages/";
import { routes } from "./routes";

const App = () => {
  return (
    <Switch>
      {routes
        .filter((route) => route.page)
        .map((route) => (
          <Route exact={route.exact} key={route.page} path={route.path}>
            {pages[route.page]}
          </Route>
        ))}
    </Switch>
  );
};

export default App;
```

<p class="caption">update routes to support pages and exact path matches - src/app/routes.js</p>

Now our app router will only render routes that have pages, and it will also match exact paths when required.

### Wrap-Up

That's quite a lot to digest for one post. Let's look back and recap what we've done:

- Identified required routes for the application
- Implemented server route handling
- Created necessary pages for rendering the application
- Implemented server-side and client-side application routing

I want to encourage you to review the code and concepts we've covered in this post, and to experiment with potential alternatives for getting everything we've built come together. I've left the implementation of the two remaining pages (user.js and signup.js) as an exercise for you, the reader. I'll include all pages in the final version of this part of the series in the [part 1 repo](https://github.com/nathanhumphrey/simple-isomorphic-app/tree/routes).

### Up Next

In the [part two](/posts/2020-02-12-full-stack-javascript-app-pt-2/) of this series, we'll build and style the necessary user interface components for this project. Specifically, We'll build a Nav component with links for navigating around the site as well as some components for viewing the user account details. A very simple user model will also be introduced to help us test our app. I hope you'll continue to join me on this learning journey.
