---
title: Starter webpack Setup for Web Applications
description: A simple webpack setup to help get you started building web applications.
abstract: webpack is module bundler that can be used to prepare a static bundle of your application. While webpack isn't the only module bundler available (e.g. Parcel, Rollup, etc.), it is still the most...
date: 2019-12-21 7:00:00.00
tags:
  - software
  - tutorial
  - webpack
  - node
  - npm
  - build tools
layout: layouts/post.njk
---

[webpack](https://webpack.js.org/) is module bundler that can be used to create a static bundle of your application for deployment. While webpack isn't the only module bundler available (e.g. Browserify, Parcel, Rollup, etc.), it is still the [most prevalent bundler](https://2019.stateofjs.com/other-tools/) in use today. And, while webpack can be used without explicit configuration, you will likely need to use some custom settings rather than the webpack defaults.

This post isn't intended to inform you about modules, the differences between various build tools (e.g. task runners vs bundlers), or the inner workings of webpack; there are countless articles and posts online about these topics. This post will provide a brief walkthrough of a starter webpack config that can get you up and running quickly, and give you the confidence needed to make further configuration changes as required. A basic grasp of how to build front-end JavaScript applications is assumed (e.g. npm, package.json, basic directory structure, babel, etc.).

_**You can find the final webpack config [here](https://gist.github.com/nathanhumphrey/a41ab2a0c30d98126b422e0f8340e73c).**_

## Install webpack

Install the required packages for working with webpack:

<code class="term">npm i -D webpack webpack-cli esm</code>

## Start With the Config - webpack.config.js

There are several ways in which you can configure your webpack setup, but they all require a `webpack.config.js` file. The config file, if present in your project's root directory, will be used by webpack automatically to load your config settings. The webpack [config](https://webpack.js.org/configuration/#options) page provides a snapshot of all the available options. **The important thing to remember is that this file must export or return a config object**, whether directly or via a function (more on that later).

<p class="info">
NOTE: I'm using ES Module syntax, so be sure to use the esm package when running scripts if your current version of node doesn't support ES Modules (experimental). E.g. npx webpack -r esm
</p>

```javascript/0
export default {};
```

<p class="caption">initial export object for webpack.config.js</p>

## Entry and Output

The first thing you will want to do is define your `entry` and `output` options. These will determine where webpack will begin it's bundling process (entry) and where the results will end up (output). The official webpack docs recommend using the `path` module for the output and prefix it with the `__dirname` global (this ensures that relative paths will work as expected, regardless of the operating system being used).

```javascript/0
import path from "path";

export default {};
```

<p class="caption">import path for resolving absolute paths</p>

### entry

The `entry` defines the starting point for your application, which will be used by webpack to begin the bundling process. Any files that are imported from this entry point (and files that are imported in those files, etc.) will all be pulled into the bundling process. By default, webpack will only work with JavaScript files, so if you have any CSS, HTML, images, etc. that you'd like to be included, you will need to configure webpack to deal with them.

```javascript/4
import path from "path";

export default {
  // relative path desired entry point
  entry: "path/to/main.js"
};
```

<p class="caption">entry option for webpack.config.js</p>

### output

The `output` defines where webpack will place the results from the bundling process. webpack will copy all the files it creates to this directory, which **must be specified as an absolute path**. As mentioned previously, the `path` module is used to ensure that the path is absolute and properly defined for your operating system. The `path` property sets the directory for output, and `filename` defines the name of the file to place there. If no filename is specified, webpack will default to `main.js`.

```javascript/5-7
import path from "path";

export default {
  entry: "path/to/main.js",
  // name desired output directory
  output: {
    path: path.resolve(__dirname, "build"),
    filename: "desiredName.js"
  }
};
```

<p class="caption">output option for webpack.config.js</p>

At this point, running the following command will build your application and place the result in the build directory:

<code class="term">npx webpack -r esm</code>

Which is fine, but only if your application consists purely of JavaScript and syntax that is supported by your version of node.

## Modules and Loaders

What if you're building an application that makes use of the latest JavaScript syntax? You're probably using [Babel](https://babeljs.io/) to transpile your code down to a more widespread version. Or, if you're building a React application, you need support for JSX, which Babel also provides. If you are creating a [React](https://reactjs.org/) application, importing a file that contains JSX will cause your current bundling process to fail with an error; this is where webpack [modules](https://webpack.js.org/configuration/module/) and [loaders](https://webpack.js.org/loaders/) come in.

webpack uses a method similar to what node.js uses regarding the treatment of individual files as modules. Within the `module` property in the config file, you can specify `rules` for loaders, which allow webpack to preprocess files that are included in the import chain. You can use the `test` property to define a regex expression to match desired files in the chain and then specify the appropriate loaders that should be applied to them, along with any supported options. So, for something like JSX, we can create a rule that matches our JavaScript files and then specify any loaders we'd like to apply to them.

There are many loaders, and they all work more or less the same way. In this post, we'll look at three practical loaders for dealing with JavaScript, image files, and CSS.

### babel-loader

One of the most common loaders in use is the [babel-loader](https://webpack.js.org/loaders/babel-loader/). This loader is used to do exactly what it sounds like, run any matched files through Babel before adding them to the bundle. Install the required npm modules with the following:

<code class="term">npm i -D babel-loader @babel/core @babel/preset-env @babel/preset-react</code>

<p class="info">NOTE: only include @babel/preset-react if you're using JSX in your application.</p>

Add the required rule for your JavaScript files:

```javascript/9-22
import path from "path";

export default {
  entry: "path/to/main.js",
  // name desired output directory
  output: {
    path: path.resolve(__dirname, "build"),
    filename: "desiredName.js"
  },
  module: {
    rules: [
      {
        test: /\.(m?js|jsx)$/, // match JavaScript files .mjs, .js, .jsx
        exclude: /node_modules/, // do not process files in here
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

<p class="caption">added a rule for JavaScript files and babel-loader to webpack.config.js</p>

Now, the latest JavaScript syntax and JSX will be properly transpiled by Babel before being included in your bundle.

### file-loader

Linked files, such as images, will also need to be included in your output directory. Processing imported images is relatively straight-forward, use the [file-loader](https://webpack.js.org/loaders/file-loader/) to process any image files and have them properly copied to the output directory and linked within your CSS or JavaScript code assets.

```javascript/6-9
...
export default {
  ...
  module: {
    rules: [
      ...
      {
        test: /\.(png|svg|jpg|gif)$/, // match various images files
        use: ["file-loader"]
      }
    ]
  }
};
```

<p class="caption">added a rule for image files and file-loader to webpack.config.js</p>

### css-loader

Web applications often require CSS, so how do you get that included in the bundle? Import the required CSS file(s) directly into your JavaScript file(s) and add a loader! While this may seem a little weird, it allows you to define CSS where you need it and to tie it directly to your components. The topic of including or importing CSS into JavaScript files has been written about many times, so I'm not going to go into the pros and cons of it here. You can use the [css-loader](https://webpack.js.org/loaders/css-loader/) to handle any CSS imports in your JavaScript files.

Install css-loader and add the required config settings:

<code class="term">npm i -D css-loader</code>

<p class="warn">WARN: This requires you to really think about how you'd like to structure your application. Importing CSS into your JavaScript files for the purposes of bundling works, but is not conventional. Build this way at your own risk.</p>

```javascript/6-9
...
export default {
  ...
  module: {
    rules: [
      ...
      {
        test: /\.css$/, // match pure CSS files
        use: ["css-loader"]
      }
    ]
  }
};
```

<p class="caption">added a rule for CSS files and css-loader to webpack.config.js</p>

By default, the css-loader will pull CSS into your bundle process but not much else. You may want to look through all the available options and play around with them to see what can be achieved. For this build, let's start by using another loader that will inject your styles into the DOM.

### style-loader

As stated previously, the css-loader pulls CSS into the bundle process, but it doesn't specify what to do with them exactly, which may be fine; if you are applying the CSS directly in your code (e.g. React style attributes), then you may not need any further processing. But, if you would like to have the CSS injected into the DOM so you can apply the styles to a document, then you need more options. The [style-loader](https://webpack.js.org/loaders/style-loader/) can be used to inject your bundled CSS into a `<style>` element in an HTML document, which is pretty easy to configure for the default case (i.e. no options). Like the css-loader, the style-loader has a lot of available configuration options that I recommend you read through and experiment with.

Install style-loader and add the required config settings:

<code class="term">npm i -D style-loader</code>

<p class="info">NOTE: Loaders are processed last to first, so in this case, the css-loader will run first followed by the style-loader.</p>

```javascript/8
...
export default {
  ...
  module: {
    rules: [
      ...
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"]
      }
    ]
  }
};
```

<p class="caption">added style-loader for CSS files to webpack.config.js</p>

So now, when you run your web application, a `<style>` tag will be injected into the head of the page. Cool! Let's install a development sever so that you can see it in action.

## Development Server

There are several [development tools](https://webpack.js.org/guides/development/#choosing-a-development-tool) that webpack makes available for developing your appication. The one that we'll work with for this example is the [development server](https://webpack.js.org/configuration/dev-server/) (devServer). The devServer allows you to work on files that are in your bundle and have any saved changes bundled on-the-fly and reloaded into your running application. Like with many of the loaders we've seen so far, the devServer can be used with default options or with a custom set of user-defined options. Before you can configure your devServer, you will need to install it:

<code class="term">npm i -D webpack-dev-server</code>

Once installed, you can setup webpack to support the devServer by adding a few lines to the config file:

```javascript/6-8
...
export default {
  ...
  module: {
    ...
  },
  devServer: {
    contentBase: path.join(__dirname, "build")
  }
};
```

<p class="caption">added devServer to webpack.config.js</p>

The lone devServer option, `contentBase`, is used to set the base directory that devServer will serve content from (typically, you would use the output directory, but you could choose any directory that makes sense for your setup). You can specify a single content base as a string or several locations in an array if need be. For the current example, serving only from the output directory will work for now.

With this current setup, you will have to create an HTML file that has a relative link to the bundle in the `build` directory. You could create this file with a hard-coded script tag linked to your bundle, but there is another way to accomplish this that allows for more flexibility.

Before moving on, now would be a good time to update the scripts in your package.json with a script that will run the devServer. Add the following script to package.json:

<p class="info">NOTE: The --mode flag will be discussed later on.</p>

```json/2
"scripts": {
  ...
  "start": "npx webpack-dev-server --open --mode development -r esm"
  },
```

<p class="caption">updated scripts in package.json</p>

and then you can run the script with the following:

<code class="term">npm start</code>

The devServer should open your default browser and load your application. If you make any changes to files in the bundle import chain while the devServer is running, you should see webpack update the bundle in the terminal and then see the changes reloaded in the browser. Neat!

## Plugins

[Plugins](https://webpack.js.org/concepts/plugins/), in webpack, can be used to extend the bundling process by alllowing you to define other actions that should take place outside of the import chain (i.e. anything that cannot be done via loaders). You can specify plugins for webpack to use by creating a new instance of the desired plugin (including any options) and passing it to the `plugins` property of the config object.

### HtmlWebpackPlugin

The [HtmlWebpackPlugin](https://webpack.js.org/plugins/html-webpack-plugin/) can be used to create an index.html file in your bundle's output directory. Having an HTML file to run directly from the output directory works well with the devServer configuration we have, and it keeps necessary files for your application in one place. Like with most things in webpack, the plugin allows you to do much more.

Using HtmlWebpackPlugin with its default behaviour will produce a generic `index.html` HTML file in the output directory with a script tag linked to your bundle's JavaScript file. But, if you are building a React app, for example, you will need to have a DOM node that can be selected as the insertion point for rendering your application; the default index.html file does not make this an easy task. Luckily, the HtmlWebpackPlugin allows you to work with your own custom HTML template that can be used in the same way as the default file (e.g. script tag will be injected) but allows you to add any custom HTML you need for your application. Your template file may include injected or templated additions, based on your config settings. Along with building the file for you, you can also use the plugin to perform various other tasks with the injected values you need in your HTML file.

You can create a template HTML file anywhere in your project directory and then configure the HtmlWebpackPlugin `template` and `filename` options to use the template and create the actual file respectively. You can use [lodash templates](https://lodash.com/docs/4.17.15#template) in the template file to perform any customization during the build process.

Install HtmlWebpackPlugin:

<code class="term">npm i -D html-webpack-plugin</code>

Then configure the plugin, as shown below:

```javascript/1,8-13
import path from "path";
import HtmlWebPackPlugin from "html-webpack-plugin";

export default {
  ...
  devServer: {
    ...
  },
  plugins: [
    new HtmlWebPackPlugin({
      template: "template.html",
      filename: "index.html"
    })
  ]
};
```

<p class="caption">added HtmlWebpackPlugin to webpack.config.js</p>

With the above HtmlWebpackPlugin config settings, you would create your `template.html` in the root of your project directory and the HtmlWebpackPlugin will produce an `index.html`, based on the template, in the build directory. HtmlWebpackPlugin has many [configurable options](https://github.com/jantimon/html-webpack-plugin#options), two of which will be used in the following sections.

#### Injection

By default, HtmlWebpackPlugin is set to `inject` any scripts at the bottom of the body element in an HTML template. It will also inject any CSS files that have been extracted into the bottom of the head element of an HTML template. If you want more control over where or how JavaScript and CSS files are injected into the template, set this option to `false`.

```javascript/3
  ...
  plugins: [
    new HtmlWebPackPlugin({
      inject: true,
      template: "template.html",
      filename: "index.html"
    })
  ]
};
```

<p class="caption">explicitly set the inject option to true for HtmlWebpackPlugin in webpack.config.js</p>

#### Cache Busting

One issue that has plagued web developers for a long time is [cache busting](https://css-tricks.com/strategies-for-cache-busting-css/). HtmlWebpackPlugin provides an option for dealing with cache busting, the `hash` option, which will include a compilation hash as a query on any JavaScript or CSS URLs that are injected into the template.

```javascript/3
  ...
  plugins: [
    new HtmlWebPackPlugin({
      hash: true,
      inject: true,
      template: "template.html",
      filename: "index.html"
    })
  ]
};

```

<p class="caption">setting the hash option for HtmlWebpackPlugin in webpack.config.js</p>

### HotModuleReplacementPlugin

The default behaviour for the devServer will reload the entire page whenever a change is detected in your bundle process, which is not really great for quick and efficient development. This is where [hot module replacement](https://webpack.js.org/concepts/hot-module-replacement/), or HMR, comes in. HMR allows webpack to only update the parts of the bundle that have changed, without requiring a full reload. The [HotModuleReplacementPlugin](https://webpack.js.org/plugins/hot-module-replacement-plugin/) is a part of webpack, and can be used explicitly to support HMR for updates to code and CSS, but the webpack devServer comes with the ability to turn on HMR with the `hot` option.

<p class="info">
NOTE: You can quickly test the functionality of HMR in this example. Run the devServer and make a change to some text in the document via your browser's dev tools. Once done, make an edit to any CSS files in your bundle and save the changes. You should see that the styling update has taken effect, but that the page has reloaded and the original text is once again visible. Once the hmr option has been set (see below), you can repeat this experiment, and you should see that the styling updates while the text remains altered.
</p>

Update the devServer configuration to turn on HMR via hot:

```javascript/6
...
export default {
  ...
  devServer: {
    contentBase: path.join(__dirname, "build"),
    // set the hmr option for HMR via devServer
    hot: true
  },
  plugins: [
    new HtmlWebPackPlugin({
      hash: true,
      inject: true,
      template: "template.html",
      filename: "index.html"
    })
  ]
};
```

<p class="caption">configured devServer for HMR in webpack.config.js</p>

The style-loader automatically makes use of HMR if it's configured, so you don't have to make any further changes to the config options. Making use of [HMR for code assets](https://webpack.js.org/guides/hot-module-replacement/) is a little more detailed and won't be covered here. If you'd like to setup code splitting and define specific behaviour for certain modules as they are changed, you will need to do a [little more work](https://webpack.js.org/guides/hot-module-replacement/#other-code-and-frameworks).

### MiniCssExtractPlugin

The current setup we have here works well for development but leaves something to be desired when it comes to production. For one thing, we'd like the CSS to be extracted to a separate file to be linked in the HTML document. The [MiniCssExtractPlugin](https://webpack.js.org/plugins/mini-css-extract-plugin/) will allow for doing just that: extracting the project's CSS into files. Let's install the plugin and configure it for basic usage:

<code class="term">npm i -D mini-css-extract-plugin</code>

Update the config for MiniCssExtractPlugin:

```javascript/2,10-14,23
import path from "path";
import HtmlWebPackPlugin from "html-webpack-plugin";
import MiniCssExtractPlugin from "mini-css-extract-plugin";

export default {
  module: {
    rules: [
      ...
      {
        test: /\.css$/,
        use: [
          "style-loader",
          MiniCssExtractPlugin.loader,
          "css-loader"
        ]
      }
    ]
  },
  devServer: {
    ...
  },
  plugins: [
    ...
    new MiniCssExtractPlugin()
  ]
};
```

<p class="caption">added MiniCssExtractPlugin to webpack.config.js</p>

As you can see, MiniCssExtractPlugin defines not only a plugin, but also a loader. The default behaviour for the plugin will be to create a file named `main.css` in your output directory. You can choose the name to use by setting the `filename` option. One other thing to note is that you no longer need the `style-loader` in your config, since you are extracting CSS to a file rather than injecting it in a style tag. The plugin also supports HMR (though HMR should not be used in production, more on that later), you just need to set the `hmr` option. Read up for more information on some [advanced config settings](https://webpack.js.org/plugins/mini-css-extract-plugin/#advanced-configuration-example).

```javascript/7-12
...
export default {
  module: {
    rules: [
      ...
      {
        test: /\.css$/,
        use: [
          // removed style-loader
          {
            loader: MiniCssExtractPlugin.loader,
            options: { hmr: true }
          },
          "css-loader"
        ]
      }
    ]
  },
  ...
```

<p class="caption">updated MiniCssExtractPlugin in webpack.config.js</p>

<p class="info">
NOTE: You could uninstall style-loader now, since it's no longer being used.
</p>

### CleanWebpackPlugin

At this point, your webpack config will save files with the same name, although the content has changed. Allowing HtmlWebpackPlugin to add a hash to the file URLs is one way of managing cache busting, but you lose information about the actual file you're dealing with. You can have webpack include a compilation hash in the name of the files that get written to the output directory so that you can maintain versions of your bundle output if necessary. A few easy updates will setup filename hashing.

```javascript/6,11,18-20
...
export default {
  entry: "path/to/main.js",
  output: {
    path: path.resolve(__dirname, "build"),
    // use the name of the entry file and include a hash
    filename: "[name].[hash].js"
  },
  ...
  plugins: [
    new HtmlWebPackPlugin({
      hash: false, // no longer need hashing in URLs
      inject: true,
      template: "template.html",
      filename: "index.html"
    }),
    ...
    // choose 'main' as the name and include a content hash
    new MiniCssExtractPlugin(
      {filename: "main.[contenthash].css"}
    )
  ]
};
```

<p class="caption">updated output filenames and hash URLs in webpack.config.js</p>

<p class="warn">
WARN: Because the filenames are now being updated with a hash value, this could cause problems with HMR (e.g. CSS content changes will update the contenthash value and the updates will not be loaded into the browser). This must be handled via additional configuration (see <a href="#webpack-mode">webpack mode</a> below).
</p>

While we have removed the need for hashing URLs and can now track versions via hash values in the filenames, a new issue has been introduced. Everytime you build your bundle after code or CSS changes, you will end up with a new file in the output directory. To keep your build directory from getting cluttered, use the [CleanWebpackPlugin](https://webpack.js.org/guides/output-management/#cleaning-up-the-dist-folder) to keep things nice and tidy.

Install and configure CleanWebpackPlugin:

<code class="term">npm i -D clean-webpack-plugin</code>

```javascript/3,9
import path from "path";
import HtmlWebPackPlugin from "html-webpack-plugin";
import MiniCssExtractPlugin from "mini-css-extract-plugin";
import { CleanWebpackPlugin } from "clean-webpack-plugin";

export default {
  ...
  plugins: [
    ...
    new CleanWebpackPlugin()
  ]
};
```

<p class="caption">added CleanWebpackPlugin to webpack.config.js</p>

Whenever you build your bundle, CleanWebpackPlugin will clean the directory before writing new output files.

## Webpack Mode

With the introduction of the `hmr` option for MiniCssExtractPlugin, we have introduced an issue to the bundling process, namely that you should not use HMR in production. The [mode](<(https://webpack.js.org/configuration/mode/)>) configuration option for webpack is used to inform webpack about optimizations it should make to your build, depending on whether you're bundling for development or production. If no mode is specified, the default is `'production'`. To make decisions about what we'd like to do in the config file, depending on the mode, we need some way of accessing the value.

There are several [configuration types](https://webpack.js.org/configuration/configuration-types/) available in webpack in addition to what we have used so far (i.e. simply exporting a config object). Exporting a function allows us to access both environment variables and any arguments that may have been passed to webpack. We can use this option to determine the desired mode and then alter the config file accordingly.

Before we update the config file, let's add an additional script to package.json to build the bundle for production:

```json/2
"scripts": {
  ...
  "build": "npx webpack --mode production -r esm",
  "start": "npx webpack-dev-server --open --mode development -r esm",
  },
```

<p class="caption">updated scripts in package.json</p>

Now we have two scripts, one for running the webpack devServer in development and one to build the bundle for production. We can alter the webpack config to export a function, which will provide access to the `--mode` arg value, and then set filenames and HMR accordingly:

```javascript/2,4,8,13,24,32,35-38,42-51,54
...
 //export a function now rather than an object
export default function(env, argv) {
  // PRODUCTION const from the --mode argument
  const PRODUCTION = argv.mode && argv.mode === "production";

  // the config object doesn't have to be defined within the function
  // but I've included it here
  let config = {
    entry: ...,
    output: {
      path: path.resolve(__dirname, "build"),
      // set the filename based on PRODUCTION for HMR
      filename: PRODUCTION? "[name].[hash].js" : "[name].js"
    },
    module: {
      rules: [
        ...
        {
          test: /\.css$/,
          use: [
            {
              loader: MiniCssExtractPlugin.loader,
              // determine hmr from PRODUCTION value
              options: { hmr: !PRODUCTION }
            },
            "css-loader"
          ]
        }
      ]
    }
    ...
    // remove the devServer property, set it later based on PRODUCTION
    plugins: [
      ...
      // remove CleanWebpackPlugin plugin, set it later based on PRODUCTION
      new MiniCssExtractPlugin({
        filename: PRODUCTION? "main.[contenthash].css" : "main.css"
      }),
    ]
  };

  if (PRODUCTION) {
    // PRODUCTION config tasks
    config.plugins.push(new CleanWebpackPlugin());
  } else {
    // DEVELOPMENT config tasks
    config.devServer = {
      contentBase: path.join(__dirname, "build"),
      hot: true
    }
  }

  // here is where the config object gets returned
  return config;
};
```

<p class="caption">updated webpack.config.js to export a function and check mode</p>

Finally! We have a simple webpack configuration for web applications. The configuration supports both development and production bundles and is ready to be extended further if need be.

## Final Thoughts and Where To Go From Here

As an intro to webpack and its configuration options, the example we've put together covers pretty much everything you need to know for getting started. Experiement with the configuration (better ways to [manage modes](https://github.com/survivejs/webpack-merge)), try different loaders (what about [sass](https://github.com/webpack-contrib/sass-loader) or [minifying your CSS](https://cssnano.co/guides/getting-started/#webpack)?), explore the numerous [plugins](https://webpack.js.org/plugins/) that are available, and learn about some of the more advanced features of webpack ([code splitting](https://webpack.js.org/guides/code-splitting/), [source maps](https://webpack.js.org/configuration/devtool/), working with [environment variables](https://webpack.js.org/guides/environment-variables/), etc.).

You can view the final webpack.config.js file below, or download it directly from this [gist](https://gist.github.com/nathanhumphrey/a41ab2a0c30d98126b422e0f8340e73c). Have fun!

```javascript
/*
 * Required npm packages:
 * - esm
 * - @babel/core
 * - @babel/preset-env
 * - @babel/preset-react
 * - webpack
 * - webpack-cli
 * - css-loader
 * - babel-loader
 * - webpack-dev-server
 * - html-webpack-plugin
 * - clean-webpack-plugin
 * - mini-css-extract-plugin
 *
 * NOTE: Use esm when running scripts (if current version
 *       of node doesn"t support ES Modules
 * E.g. npx webpack --mode production -r esm
 */
import path from "path";
import HtmlWebPackPlugin from "html-webpack-plugin";
import MiniCssExtractPlugin from "mini-css-extract-plugin";
import { CleanWebpackPlugin } from "clean-webpack-plugin";

export default function(env, argv) {
  const PRODUCTION = argv.mode && argv.mode === "production";

  let config = {
    entry: "./src/index.js",
    output: {
      path: path.resolve(__dirname, "build"),
      filename: PRODUCTION ? "[name].[hash].js" : "[name].js"
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
        },
        {
          test: /\.(png|svg|jpg|gif)$/,
          use: ["file-loader"]
        },
        {
          test: /\.css$/,
          use: [
            {
              loader: MiniCssExtractPlugin.loader,
              options: { hmr: !PRODUCTION }
            },
            "css-loader"
          ]
        }
      ]
    },
    plugins: [
      new HtmlWebPackPlugin({
        inject: true,
        hash: false,
        template: "src/index.html",
        filename: "index.html"
      }),
      new MiniCssExtractPlugin({
        filename: PRODUCTION ? "main.[contenthash].css" : "main.css"
      })
    ]
  };

  if (PRODUCTION) {
    config.plugins.push(new CleanWebpackPlugin());
  } else {
    config.devServer = {
      contentBase: path.join(__dirname, "build"),
      hot: true
    };
  }

  return config;
}
```

<p class="caption">final webpack.config.js</p>
