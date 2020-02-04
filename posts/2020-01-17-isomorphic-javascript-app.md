---
title: Isomorphic JavaScript App
description: A walkthrough build of a simple isomorphic JavaScript application, built using Koa and React.
abstract: So, the other day, I had this great idea to explore the land of isomorphic JavaScript and server-side rendering (SSR), because why not? Building front-end applications that run entirely on the client is fun, but sometimes ...
date: 2020-01-17 7:00:00.00
tags:
  - software
  - tutorial
  - isomorphic
  - ssr
  - react
  - koa
layout: layouts/post.njk
---

## But why?

So, the other day, I had this great idea to explore the land of [isomorphic](https://en.wikipedia.org/wiki/Isomorphic_JavaScript) JavaScript and server-side rendering (SSR), because why not? Building front-end applications that run entirely on the client is fun, but sometimes you need to execute some custom business logic or work with a secret key that you don't want to be exposed on the client. For me, the goto choice for the front-end part of this project will be React (sorry Vue fans, but I haven't been converted ... yet), and you might think that Express would be my go-to for the server, but I've been working with Koa for some time now so that's what I've chosen as my backend for this project.

Alright, what will it take to pull this off?

## Requirements

- Backend server - required for server-side rendering of course
- Front-end library that supports SSR - code sharing between client and server (could be vanilla JS, but React works nicely here)
- Application bundler - work with ES6+ features and, in this case, JSX

As this post is about isomorphic development with JavaScript, I'm not going to go into the details of the libraries/frameworks I've chosen to use in the implementation. If you'd like to learn more about them, there are many articles and courses online that can help. I've created a GitHub repo with the finished project that you can check out as well.

_**You can find the project repository [here](https://github.com/nathanhumphrey/simple-isomorphic-app).**_

### Node.js + Koa

Some of you may be thinking, "[Koa](https://koajs.com/)? Never heard of it, why not use Express?" and you're right, you could use Express but I prefer the ["simplicity" and configuration-over-convention](https://github.com/koajs/koa/blob/master/docs/koa-vs-express.md) of Koa. Everything I'm about to show you, using Koa, you could easily implement in Express, so this isn't that big an issue.

### React

These days, it seems like everything is built in [React](https://reactjs.org/); there are some promising up-and-comers, but React is still the dominant front-end library. Of course, using a front-end library is not a strict requirement, building an isomorphic app could be done via vanilla JS, but React is fairly ubiquitous and supports [SSR](https://reactjs.org/docs/react-dom-server.html), so that should save us some time.

### Webpack

For bundling, I'm still sticking with [webpack](https://webpack.js.org/). You could ask "why use a bundler at all?", and I'd respond, "because we need to [bundle all the things](http://tinselcity.net/whys/packers)... right?" Seriously though, we need some way to transpile and bundle the application for deployment, so I've chosen to go with what I know here too. As stated above regarding Koa vs. Express, you could probably get this to work with another bundler (or perhaps just straight Babel), but webpack makes it so much easier for me.

## Initial setup

First things first, we need a directory setup. For an isomorphic application, we'll need to have some code that executes on the client, some code that executes on the server, and some code that is shared between the two (essentially the application itself). So, with that in mind, let's set up our directory structure like so:

<p class="info">
NOTE: the following directory structure is one that makes sense to me. You may see similar setups where the 'app' directory is named 'shared' or something else. Choose names that make sense to you.  
</p>

```bash
project/
    |-src/
        |-client/
        |-server/
        |-app/
```

<p class="caption">project directory structure</p>

This initial directory structure may require some modification once we get started, but these directories are pretty much must-haves. All development code will be stored in the `src/` directory; specifically, the Koa server will be stored in `server/`, the client loader will be stored in `client/`, and the React application will be stored in `app/`.

### Install the requirements

From within your terminal, navigate to your `project/` directory, initialize your project with npm, and then install the required base packages:

```bash
$ npm init -y
$ npm i @babel/core @babel/preset-env @babel/preset-react \
react react-dom koa @babel/cli koa-webpack webpack webpack-cli \
babel-loader
```

<p class="caption">npm init and install commands</p>

## Complete the build

### Create the application

Create an `App.js` file in the `src/app/` directory and add the following code:

<p class="info">
NOTE: I've chosen to have this sample app implement a simple click interaction (as seen <a href="https://reactjs.org/docs/hooks-state.html">elsewhere</a>). Again here, choose whatever interaction you'd like if you want something more meaningful. I just wanted something easy to implement that would allow for testing code execution on the client.
</p>

```js
import React, { useState } from "react";

const App = () => {
  const [clicks, setClicks] = useState(0);

  return (
    <div>
      <h1>React Clicker Application</h1>
      <button onClick={e => setClicks(clicks + 1)}>
        Clicked {clicks} times
      </button>
    </div>
  );
};

export default App;
```

<p class="caption">basic React component with simple state management</p>

The simple component created above will allow us to demonstrate that the app can be rendered server-side, but also provide interaction once on the client.

### Create the server

Create an `index.js` file in the `src/server/` directory and add the following code:

<p class="info">
NOTE: I wouldn't use the following implementation in production; meaning, I wouldn't want my application sever running everything through webpack for every request. Preparing an isomorphic applicaiton for production may be covered in another post.
</p>

```js
import Koa from "koa"; // base server
import koaWebpack from "koa-webpack"; // useful for development
import webpack from "webpack"; // to bundle in development
import config from "../../webpack.config"; // import webpack config
import renderReactApp from "./render-react-app"; // performs the SSR

// create the app server
const app = new Koa();

// prepare webpack compiler
const compiler = webpack(config);

// create koa webpack middleware
koaWebpack({ compiler }).then(middleware => {
  // use the koa middleware
  app.use(middleware);
  // use renderReactApp middleware (not yet implemented)
  app.use(renderReactApp);
});

// just listen on port 3000, for now
app.listen(3000, () => {
  console.log("Server listening on port 3000");
});
```

<p class="caption">super simple koa server, only supports SSR React app</p>

What the previous code does is this:

- creates a new Koa app server
- utilize koa-webpack to build webpack middleware
- render the application (not yet implemented)
- listen for connections on port 3000

Based on this simple implementation, we need to create two things: a **webpack config** and a **renderReactApp** middleware.

#### webpack.config.js

The purpose of using webpack on this build is to bundle the project code that will be required for the client (entry will be in the `src/client/` directory. For now, all we need to do is run our client code through babel and output the bundle to a simple-named file. Create the `webpack.config.js` file in the `projet/` root directory and add the following code:

```js
const path = require("path");

module.exports = {
  mode: "development",
  target: "web",
  entry: ["./src/client/index.js"],
  resolve: {
    extensions: ["*", ".js", ".jsx"]
  },
  output: {
    path: path.resolve(__dirname, "static"),
    publicPath: "/",
    filename: "index.js"
  },
  module: {
    rules: [
      {
        test: /\.(m?js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader",
          options: {
            presets: ["@babel/preset-env", "@babel/preset-react"]
          }
        }
      }
    ]
  }
};
```

<p class="caption">webpack.config.js to create our application bundle</p>

<p class="info">
NOTE: I've used CJS module syntax for this file, which is due to it's being executed outside of any Babel pipeline; soon we should be able to use ES Modules everywhere.
</p>

#### renderReactApp.js

Create a `render-react-app.js` file in the `src/server/` directory and add the following code:

```js
import React from "react";
import { renderToString } from "react-dom/server";
import App from "../app/App";

export default ctx => {
  const renderComponent = (
    <html>
      <head>
        <title>Isomorphic React App</title>
      </head>
      <body>
        <div id="app">
          <App />
        </div>
        <script src="index.js" />
      </body>
    </html>
  );

  ctx.body = renderToString(renderComponent);
};
```

<p class="caption">code for SSR of the React component we created earlier</p>

The `render-react-app.js` file doesn't do anything fancy, it simply creates a Koa middleware that can render our App component within a div; it also includes a `script` element that will be used to deliver the client payload for app hydration (see [webpack.config.js](#webpack.config.js) above). The middleware will call the `renderToString()` function from the `react-dom/server` package, which provides the magic of SSR in this case.

### Create the client-side code

The final thing that needs to be done is to ensure that we can [hydrate](https://reactjs.org/docs/react-dom.html#hydrate) our application on the client. Create an `index.js` file in the `src/client/` directory and add the following code:

<p class="warn">
WARN: failing to hydrate the App on the client (and simply calling render) will result in React duplicating the efforts of your server. The hydrate function will attempt to work with the markup already provided.
</p>

```js
import React from "react";
import ReactDOM from "react-dom";

import App from "../app/App";

// bring the application to life!
ReactDOM.hydrate(<App />, document.getElementById("app"));
```

<p class="caption">client application launcher</p>

That's it for the application!

## Build and run the application

Finally, let's add a script that we can use to run our Koa server. Update package.json with the following:

```json
…
"scripts": {
    "serve": "node ./src/server/index.js"
  },
...
```

<p class="caption">script to execute the server, but will it work?</p>

Open your terminal, and from the project directory, run the following:

<code class="term">npm run serve</code>

Open a browser tab and navigate to `http://localhost:3000` to see… nothing, because your terminal is filled with errors. As stated earlier, my current version of node doesn't support ES modules, but that's what I've chosen to use throughout the application (which says nothing about the JSX our server is going to try to process in the render-react-app.js file). We need to do one last thing before everything is ready to go. Install the `@babel/register` package to help us with our current issue, namely, we need to run our server script through Babel in order for it to work in its current state. Run the following command in your terminal:

<p class="warn">
WARN: as mentioned earlier, this setup works well for development purposes, but please don't adopt this practice for production. It would be better to run your application through Babel as part of a production build process and then run the server.
</p>

<code class="term">npm i @babel/register</code>

Finally, put the package to use by creating an `index.js` file in the `project/` directory that will execute the package for the server:

```js
require("@babel/register")({
  presets: ["@babel/preset-env", "@babel/preset-react"],
  ignore: ["node_modules"]
});

module.exports = require("./src/server/index.js");
```

<p class="caption">script to run the server through babel on the fly</p>

Update the package.json script:

```json
...
"scripts": {
    "serve": "node index.js"
  },
...
```

<p class="caption">script to execute the server, should work now</p>

One last time, return to your terminal, and try the `serve` script again:

<code class="term">npm run serve</code>

All should be well in the terminal window, so let's see what's being served to the client. Open or return to your browser tab and navigate to `http://localhost:3000`. Now, you can marvel at your application in action! If you open your browser's dev tools, you should be able to see the original source that was sent from the server (rendered App), but also note that the application has been hydrated and responds to the user interaction as expected (click the button!). That's it, an isomorphic JavaScript app in action... but, by no means is it production-ready.

## There’s still a lot more that we could explore

This exercise is instructive as to 'how' isomorphic JavaScript apps could be built, but it has by no means tackled any of the additional requirements a typical app needs in production. Heck, we’re only running the bundler and server in development mode! Additional topics to cover include (but are not limited to):

- serving static assets
- component styling and stylesheets
- shared application routing
- cross-site request forgery
- bundling for production
- many other topics

I may cover these additional topics, or prepare a series, in the near future if I can find the time. Stay tuned.
