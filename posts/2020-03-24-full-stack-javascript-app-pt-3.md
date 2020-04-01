---
title: Full Stack JavaScript Application Build Part 3 - Styling Components
description: Part 3 of a series about how to build a full stack JavaScript application. This article is about styling the pages and components for the app.
abstract: Okay, on to Part 3 of the Full Stack Application series. In Part 3, we'll look at various ways of styling the components that were created in part 2. This post will be a little shorter than the previous two, but will provide some insight into the component styling options that are available.
date: 2020-03-24 7:00:00.00
tags:
  - software
  - tutorial
  - javascript
  - fullstack
  - koa
  - react
layout: layouts/post.njk
---

## Introduction

Okay, on to Part 3 of the Full Stack Application series. In Part 3, we'll look at various ways of styling the components that were created in [part 2](/posts/2020-02-12-full-stack-javascript-app-pt-2/). This post will be a little shorter than the previous two, but will provide some insight into the component styling options that are available.

<p class="info">
NOTE: emphasis here is placed on the <i>how</i> and not the <i>why</i>; what this means is that I will focus on how styling components can be accomplished but not so much on why the components should be styled in any particular way (e.g. responsive, UX, branding, etc.).
</p>

**_The final project source can be found in this [GitHub repo](https://github.com/nathanhumphrey/simple-isomorphic-app/tree/styling)_**

### Roadmap

The build is divided into a six-part series:

1. [Routes and pages](/posts/2020-02-06-full-stack-javascript-app-build-pt-1/)
2. [Building components](/posts/2020-02-12-full-stack-javascript-app-pt-2/)
3. [Styling components](/posts/2020-03-24-full-stack-javascript-app-pt-3/) ⇐ you are here
4. ~~Database access and authentication~~ (coming soon)
5. ~~Protecting against CSRF attacks~~ (coming soon)
6. ~~Production bundling~~ (coming soon)

## Component Styling Options

To begin, the official React website has a brief intro FAQ page entry about [styling and CSS](https://reactjs.org/docs/faq-styling.html) for React components, which links out to other relevant resources. The options essentially come down to one of the following:

<p class="info">
NOTE: I suggest following up on the links presented on the React site to get a deeper understanding of the topic.
</p>

1. external CSS stylesheets and selectors (i.e. the traditional approach)

2. inline CSS via the [style attribute](https://reactjs.org/docs/dom-elements.html#style)

3. CSS in JS, for which, the [options are many](http://michelebertoli.github.io/css-in-js/)

#### External CSS

Definitely the most straight forward of the three approaches (and the one that will be adopted for this walkthrough). Linking your application to external stylesheets allows you to define your CSS as you always have, and not have to worry about any complex configuration requirements (though, there is some setup required). The trick when working with linked external CSS is to have the CSS included somehow in your bundle/build process. We will implement our CSS solution so that it works with our webpack development setup, and leave production optimization for a later article.

#### Inline CSS

Inlining CSS in your React components can also easily be done, but also spreads your CSS all over your application. Each component is responsible for its own styling, which makes it easy to know what rules apply to the component, but it also makes it difficult to manage global and shared styles too. You may find that some mix of both external global styles and inlined styles will work for you, but I strongly suggest avoiding inlining your CSS for improved performance and maintainability.

#### CSS in JS options

As the link above will show you, there are no shortage of CSS in JS options for you choose from. Typically, you will want to choose an option that best suits your needs (e.g. if you need server-side rendering, make sure the library you choose supports that). CSS in JS can offer several advantages over the two previous options mentioned, namely, code splitting and minimal CSS generation. This may be a good option for larger projects where the CSS may become very large or include rules that may not often be necessary. You will need to invest time in understanding how the library works and the best practices for your particular setup. Before choosing a CSS in JS option to move forward with, I recommend researching some reviews and tutorials to help you find one you feel comfortable using.

<p class="info">
NOTE: inlining CSS in your React components is fairly straightforward and doesn't require any additional example beyond what the official docs provide. CSS in JS can also be accomplished in many ways, far too many to cover succinctly in this article, but the setup we will be left with should provide a good base for you to experiment with CSS in JS techniques on your own. To keep things tight and focused, we'll look at how to link an external stylesheet to our application and leave the other options for you to explore.
</p>

## External Stylesheets Implementation

The simplest way to style our application is via the traditional external stylesheet. A CSS file can be added as a static asset to the application and then simply linked in the generated HTML that our server sends back to the client. Our webpack config directs bundled output to the `static/` directory, so we will need to include a `link` tag that references a CSS file in this location in our `render-react-app.js` file. But, the question remains: where should we put the development version of our CSS file?

### CSS as a module

The easiest way to get our CSS included in our bundle is to import a CSS file into one of our JS files (e.g. import some CSS file in `client/index.js`). We can specify a loader for CSS in our webpack config and then reliably link to the CSS file in our final application bundle. Simple! First, create a new CSS file in `src/app/` directory:

```bash/3
src/
|-- client/
    |-- index.js
    |-- main.css
```

<p class="caption">create the main.css file</p>

Open the new `main.css` file and add some styling:

```css
html {
  font-family: Arial, Helvetica, sans-serif;
}
```

<p class="caption">styling added to main.css</p>

Now, add an import statement for the new CSS file in `src/client/index.js`:

```js/5
import React from 'react';
import ReactDOM from 'react-dom';
import { BrowserRouter } from 'react-router-dom';

import App from '../app/App';
import './main.css';
...

```

<p class="caption">import CSS into App.js</p>

Build and run the application and ... it's broken. The problem we're running into now is that webpack doesn't know how to deal with CSS as a module (you should be seeing a parse error from main.css). The good news is that this is easily fixable, but keep in mind that we're fundamentally tying ourselves to some type of bundler going forward.

<p class="warn">
As I've written in a previous <a href="https://nathanhumphrey.ca/posts/2019-12-21-simple-webpack-setup/#css-loader">article</a>, importing CSS into JS files this way forces you to take on some technical debt.
</p>

We'll need to include a loader that will allow webpack to work with CSS as a module and a second loader to specify what we'd like to do with the CSS (i.e. output the CSS as a separate file). Start by installing the necessary webpack loaders:

<p><code class="term">npm i -D css-loader style-loader</code></p>

And then update `webpack.config.js` to use them:

```js/6-9
...
module.exports = {
  ...
  module: {
    rules: [
      ...
      {
        test: /\.css$/, // match pure CSS files
        use: ["style-loader", "css-loader"]
      }
    ]
  }
};
```

<p class="caption">update webpack.config.js for CSS loading</p>

Restart your server and everything should be working now. You might be wondering how the styling is working, given that we haven't yet explicitly linked any CSS file to our HTML. The `style-loader` we're using injects a `style` tag with any retrieved CSS directly into the DOM by default. If you inspect your page in the browser, you should see the `style` tag right before the closing `head` tag. This is fine for development, but we would definitely want to link to an external file in production.

### Extract CSS for linking

Without getting into the specifics of development vs. production best practices, let's just update our application to make use of the externally linked CSS file, just so we can see how it's done.

<p class="info">
NOTE: the setup we have right now is more than fine for development. Ideally, we could use the current setup right up until we need to build for production, and then configure our bundler to ensure proper use of external stylesheets.
</p>

We'll need to do some more work with webpack as well as make updates to our `render-react-app.js` file as mentioned earlier. Install the `mini-css-extract-plugin` plugin:

<p><code class="term">npm i -D mini-css-extract-plugin</code></p>

Then update `webpack.config.js` to use them:

```js/1,10-22
const path = require('path');
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = {
  ...
  module: {
    rules: [
      ...
      {
        test: /\.css$/, // match pure CSS files
        use: [
          // removed style-loader for mini-css-extract-plugin
          MiniCssExtractPlugin.loader,
          "css-loader"
        ]
      }
    ]
  },
  plugins: [
    new MiniCssExtractPlugin(
      {filename: "main.css"}
    )
  ]
};
```

<p class="caption">update webpack.config.js for CSS extraction</p>

If you run the server now and check the output, you will see that the styling has once again been removed. We need to link to the extracted CSS file in our rendered output. Update `render-react-app.js`:

```js/7
...
export default ctx => {
  const renderComponent = (
    <html>
      <head>
        <title>Isomorphic React App</title>
        <script src="index.js" defer />
        <link rel="stylesheet" href="main.css" />
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
...
```

<p class="caption">update render-react-app.js to link to an externall CSS</p>

You will notice that this doesn't solve problem, but it does move us one step closer. If you inspect your browser's network panel in the dev tools, you will notice that the request for the `main.css` file was sent, but that the response from the server is a full HTML page, not the CSS file. That's because we haven't configured Koa to respond to static asset requests (i.e. we don't have a route for the CSS file). We'll need to configure Koa to serve static assets when requested.

### Serving static assets

Install the `koa-static` package:

<p><code class="term">npm i koa-static</code></p>

Then configure the server in `src/server/index.js`:

```js/0,3,13
import path from 'path'; // for defining static asset path
import Koa from 'koa'; // base server
import koaWebpack from 'koa-webpack'; // useful for development
import koaStatic from 'koa-static'; // serves static assets
...

// create koa webpack middleware
koaWebpack({ compiler }).then(middleware => {
  // use the koa middleware
  app.use(middleware);
  // removed the renderReactApp middleware, use router going forward
  app.use(router.routes())
    .use(router.allowedMethods())
    .use(koaStatic(path.join(__dirname, '../../static')));
});
...
```

<p class="caption">update src/server/index.js to serve static assets</p>

Koa will now search for static assets in the `static/` directory and serve them if found. Now we can complete the styling for the application in `main.css`.

### Component styling

The process is similar for when you want to add CSS styling to a specific component. Simply create your CSS file and import in the component file. Let's add a little styling to the `PageHeader` component. Create and edit the `src/app/components/Page/PageHeader/PageHeader.css` file:

```css
/* simple layout CSS */
.page-header {
  display: flex;
  margin: 0;
  padding: 0;
}
```

<p class="caption">create src/app/components/PageHeader/PageHeader.css</p>

Now, import `PageHeader.css` into the `PageHeader.js` file:

```js/2
import React from 'react';
import PropTypes from 'prop-types';
import './PageHeader.css';
...
```

<p class="caption">import PageHeader.css into src/app/components/PageHeader/PageHeader.js</p>

Once these updates have been made, you'll find that the serve script now fails again. This happens because nodemon process is going to try to run the scripts on the server. Without any support from webpack (as is the case for the client/index.js file), the server process fails. The solution, for now, is to use the `ignore-styles` package to do just that, tell the server to ignore any style imports. Import the `ignore-styles` package:

<p><code class="term">npm i ignore-styles</code></p>

Update the serve script in `package.json`:

```json/2
...
"scripts": {
  "serve": "nodemon --require ignore-styles index.js"
},
...
```

<p class="caption">update the serve script in package.json</p>

You should now, once again, be able to run the server and view your styled pages and components. The PageNav has also been styled in a similar way (view the [repo](https://github.com/nathanhumphrey/simple-isomorphic-app/tree/styling) for the final version).

![Styled nav](/img/2020-03-24-full-stack-javascript-app-pt-3/styled-nav.png)

<p class="caption">styled navigation component</p>

![Styled header](/img/2020-03-24-full-stack-javascript-app-pt-3/styled-header.png)

<p class="caption">styled page header component</p>

## Wrap-Up

In this walkthrough, we updated our application with some basic styling via an external CSS file and also implemented the ability of our server to serve static assets. At this point, you can define any required CSS in the `main.css` file and simply link rules to your components via the `className` attribute. I'll update the project with some basic styling, which you can find in the Github repo.

## Up Next

In Part 4 (coming soon), we'll add database access and user authentication to our application; both common required tasks for many web applications.
