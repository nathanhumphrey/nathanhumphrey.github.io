---
title: Full Stack JavaScript Application Build Part 2 - Building the Components
description: Part 2 of a series about how to build a full stack JavaScript application. This article is about building the components for the app.
abstract: Welcome to Part 2 of the Full Stack Application series. In this part of the series, we'll build and style a few simple components to allow us to fulfill the required use cases that were presented in part 1. If you'd like to build the application but haven't worked through part 1 already, I would suggest you start there. If you'd just like some info on how to build and style components, then please, read on.
date: 2020-02-12 7:00:00.00
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

Welcome to part 2 of the Full Stack Application series. In this part of the series, we'll build and style a few simple components to allow us to fulfill the required use cases that were presented in [part 1](https://nathanhumphrey.ca/posts/2020-02-06-full-stack-javascript-app-build-pt-1/). If you'd like to build the application but haven't worked through part 1 already, I would suggest you start there. If you'd just like some info on how to build and style components, then please, read on.

**_The final project source can be found in this [GitHub repo](https://github.com/nathanhumphrey/simple-isomorphic-app/tree/components)_**

### Roadmap

The build is divided into a six-part series:

1. [Routes and pages](/posts/2020-02-06-full-stack-javascript-app-build-pt-1/)
2. [Building components](/posts/2020-02-12-full-stack-javascript-app-pt-2/) ⇐ you are here
3. [Styling components](/posts/2020-03-24-full-stack-javascript-app-pt-3/)
4. Database access and authentication
   1. [Backend implementation](/posts/2020-04-03-full-stack-javascript-app-pt-4-phase1/) ⇐ you are here
   2. ~~Frontend implementation~~ (coming soon)

## Application wireframes

To help us envision what we're going to build, let's start with some basic wireframes.

### Home Page

The Home Page must allow a user to sign into their account or sign out of their if they've already been authenticated. Basically, all functionality for this page will be implemented in the page header.

![Home page wireframe](/img/2019-02-12-full-stack-javascript-app-pt-2/home-wireframe.png)

<p class="caption">home page wireframe</p>

### Sign Up Page

This page must allow a user to sign up for a new account. This page will need to provide the user with the ability to enter their account details.

![Sign up page wireframe](/img/2019-02-12-full-stack-javascript-app-pt-2/signup-wireframe.png)

<p class="caption">sign up page wireframe</p>

### User Details Page

The page will allow a user to view their account details.

![User account view page wireframe](/img/2019-02-12-full-stack-javascript-app-pt-2/user-wireframe.png)

<p class="caption">sign up page wireframe</p>

## Required components

We'll need a few components to enable the use cases identified in part 1. The shortlist of components is as follows:

- UserSignInForm
- UserSignUpForm
- UserAccountView

The directory structure for these components will look something like this:

```bash
src/
|-- app/
    |-- components/
        |-- User/
            |-- UserSignInForm/
                |-- index.js
                |-- UserSignInForm.js
            |-- UserSignUpForm/
                |-- index.js
                |-- UserSignUpForm.js
            |-- UserAccountView/
                |-- index.js
                |-- UserAccountView.js
```

<p class="caption">User component directory and file hierarchy</p>

In addition to these application use case components, we will also build a few UI components to make building the interface a little easier (we built the starter `Page` component in part 1):

- Page
  - PageHeader
  - PageNav

```bash
src/
|-- app/
    |-- components/
        ...
        |-- Page/
            |-- index.js
            |-- Page.js
            |-- PageHeader/
                |-- index.js
                |-- PageHeader.js
            |-- PageNav/
                |-- index.js
                |-- PageNav.js
```

<p class="caption">Page component directory and file hierarchy</p>

The same directory and file structure that was describe for the user components will be used for these components as well (e.g. Page.js will exist under src/app/components/Page, PageHeader will exist under src/app/components/Page/PageHeader, etc.).

<p class="info">
NOTE: Some developers like to append <i>Component</i> to their component names, but I like to let the directory structure indicate what is a component and what is not; do whatever feels right for you (Brad Frost has a great post that covers several opinions that developers have about <a href="https://bradfrost.com/blog/post/this-or-that-component-names-index-js-or-component-js/">naming components</a>). My personal preference is to use directory pathing for identifying the grouping of components and to name files after what they represent. In order to cut down on redundancy when importing components, I also use an <code>index.js</code> for each component directory that simply exports the named component as a default (similar to what we did with our pages in part 1). Try a few options and then settle on the one that suits you and your team best.
</p>

## Building Components

Let's start with the one component we've already created, `Page`. To align with the component directory structure proposed above, create a new `Page` directory under the `src/app/components` directory and move `Page.js` into it. Now, if you try to serve the application, you will be met with an error: _Cannot find module '../components/Page_. This is because of the import statement in each page that tries to import the `Page` component directly from the `components` directory. This can easily be remedied by including an `index.js` file in the new `Page/` directory that exports the `Page` component as the default. Let's do that now.

First, create an `index.js` file in `src/app/components/Page/`, then enter the following code:

```js
export { default } from "./Page.js";
```

<p class="caption">exporting the Page component - src/app/components/Page/index.js</p>

So now we've moved our Page component and we don't have to make any updates to the other parts of our codebase that were previously importing it! Let's continue building the necessary UI components and adding them to the `Page` component as we go.

### The `PageNav` Component

The PageNav for this application will provide a user with the means to navigate to the various pages of the application. One easy way of implementing this is to create a list of links. Create a `PageNav` directory, `PageNav.js` file, and `index.js` file:

```bash/4-6
src/
|-- app/
    |-- components/
        |-- Page
            |-- PageNav/
                |-- index.js
                |-- PageNav.js
```

<p class="caption">creating the required PageNav directory and files</p>

We can create a simple component with basic HTML tags to get a general feel for the component before getting into more detail. Open the `PageNav.js` file and enter the folowing code:

```js
import React from "react";

const PageNav = () => {
  return (
    <nav className="page-nav">
      <ul>
        <li>
          <a href="#">Link 1</a>
        </li>
        <li>
          <a href="#">Link 2</a>
        </li>
        <li>
          <a href="#">Link 3</a>
        </li>
      </ul>
    </nav>
  );
};

export default PageNav;
```

<p class="caption">creating the starting point for the PageNav component - src/app/components/Page/PageNav/PageNav.js</p>

Export the `PageNav` component from `index.js` to support clean imports as well:

```js
export { default } from "./PageNav.js";
```

<p class="caption">exporting the PageNav component - src/app/components/Page/Nav/index.js</p>

To test this component, let's add it to the Page and then visit the index route in a browser. Make the following update to `Page.js`:

```js/2,7
import React from "react";
import PropTypes from "prop-types";
import PageNav from "./PageNav";

const Page = ({ children }) => {
  return (
    <div className="page-container">
      <PageNav />
      <main>{children}</main>
    </div>
  );
};

Page.propTypes = {
  children: PropTypes.node,
};

export default Page;
```

<p class="caption">testing the PageNav component in the page - src/app/components/Page/Page.js</p>

You should be able to navigate to the home page and see the navigation list rendered. If you visit any of our other defined routes (e.g. /signup or /users/:id), you should see the same list. Okay, let's actually link to our application pages.

![Testing the PageNav](/img/2019-02-12-full-stack-javascript-app-pt-2/nav-1.png)

<p class="caption">the PageNav rendered in the browser with example links</p>

Since we're using React Router, our `PageNav` component will need to implement `NavLink` components to work properly with the routers we've defined for our application. Update `PageNav` to support nav links for navigating to the signup and user pages:

<p class="info">
NOTE: more information about <a href="https://reacttraining.com/react-router/web/api/NavLink">NavLink</a>. 
</p>

```js/1,8,11,14
import React from "react";
import { NavLink } from "react-router-dom";

const PageNav = () => {
  return (
    <nav>
      <ul>
        <li>
          <NavLink to="/">Home</NavLink>
        </li>
        <li>
          <NavLink to="/signup">Sign Up</NavLink>
        </li>
        <li>
          <NavLink to="/users/:id">Account</NavLink>
        </li>
      </ul>
    </nav>
  );
};

export default PageNav;
```

<p class="caption">updating PageNav to make use of NavLink - src/app/components/Page/PageNav/PageNav.js</p>

We have replaced the anchor tags with `NavLink` components, which accept a `to` prop in place of the `href` attribute previously used. The `NavLink` also supports props for styling the currently active link, which we will implement later on.

![Testing the PageNav with NavLinks](/img/2019-02-12-full-stack-javascript-app-pt-2/nav-2.png)

<p class="caption">rendering NavLinks in the PageNav component</p>

<p class="info">
NOTE: react-router-dom also proivdes a more generic <code>Link</code> component, which simply lacks some of the additional featues of the <code>NavLink</code>.
</p>

We're just about done (for now) with our `PageNav` component. The final update will allow for simpler updating, should we decide to add more pages to our application. Let's update the `PageNav` to make use of the defined routes we created in the previous part of this series. Once again, let's edit `PageNav.js`:

```js/2,8-12
import React from "react";
import { NavLink } from "react-router-dom";
import { routes } from "../../../routes";

const PageNav = () => {
  return (
    <nav>
      <ul>
        {routes.map((route) => (
          <li key={route.name}>
            <NavLink to={route.path}>{route.name}</NavLink>
          </li>
        ))}
      </ul>
    </nav>
  );
};

export default PageNav;
```

<p class="caption">utilize routes in the PageNav - src/app/components/Pag/ePag/Nav/PageNav.js</p>

Great, except for one small problem; we are now also rendering the _no match_ route!

![Testing the PageNav with NavLinks](/img/2019-02-12-full-stack-javascript-app-pt-2/nav-3.png)

<p class="caption">rendering all routes in the PageNav component</p>

There are a couple of options available to us to solve this problem, but probably the easiest is to just add some additional properties to our routes for navigation. Open `src/app/routes.js` and make the following updates:

```js/6-8,14-16,22-24
export const routes = [
  {
    path: "/",
    method: "get",
    name: "home",
    page: "HomePage",
    exact: true,
    navigation: true,
    linkText: "Home",
  },
  {
    path: "/signup",
    method: "get",
    name: "signup",
    page: "SignUpPage",
    navigation: true,
    linkText: "Sign Up",
  },
  {
    path: "/users/:id",
    method: "get",
    name: "userdetails",
    page: "UserPage",
    navigation: true,
    linkText: "Account",
  },
  {
    path: "*",
    method: "get",
    name: "nomatch",
    page: "NoMatchPage",
  },
];
```

<p class="caption">updating routes for navigation - src/app/routes.js</p>

What we have done here is added some navigation-specific properties to the desired routes. Namely, we've identified routes that should be included in navigation with the aptly named `navigation` property, and we've also included some customizable link text to display in the `linkText` property. The best part about these updates is that they have no effect on the other parts of our application that make use of the routes. Update `PageNav.js` to make use of these new route properties:

```js/8-14
import React from "react";
import { NavLink } from "react-router-dom";
import { routes } from "../../../routes";

const PageNav = () => {
  return (
    <nav>
      <ul>
        {routes
          .filter((route) => route.navigation)
          .map((route) => (
            <li key={route.name}>
              <NavLink to={route.path}>{route.linkText}</NavLink>
            </li>
          ))}
      </ul>
    </nav>
  );
};

export default PageNav;
```

<p class="caption">utilize updated routes in the PageNav - src/app/components/Pag/ePag/Nav/PageNav.js</p>

Now we can filter to only include `navigation` routes, and we can use the desired `linkText` in our navigation links. Everything is back to normal.

![Testing the PageNav with NavLinks](/img/2019-02-12-full-stack-javascript-app-pt-2/nav-4.png)

<p class="caption">rendering only fitered navigation routes in the PageNav component</p>

The `PageNav` is looking pretty good, but we can do better. In its current state, the `PageNav` is tied to the `route.js` file, which means it can really only fulfill the single purpose of acting as an _application routes_ navigation component; we'd like to be able to use it for any type of navigation we may need. For example, we may want a navigation component in the footer of the page for all our non-application related content (e.g. privacy notice, about us information, etc.). Let's decouple the `PageNav` from the routes file and also allow for the inclusion of additional content (e.g. customized links). Make the following updates to `PageNav.js`:

```js/2,3,6,16,17
import React from "react";
import { NavLink } from "react-router-dom";

const PageNav = ({ routes, children }) => {
  return (
    <nav>
      {routes && (
        <ul>
          {routes
            .filter((route) => route.navigation)
            .map((route) => (
              <li key={route.name}>
                <NavLink to={route.path}>{route.linkText}</NavLink>
              </li>
            ))}
        </ul>
      )}
      {children && children}
    </nav>
  );
};

export default PageNav;
```

<p class="caption">improving the utility of the PageNav component - src/app/components/Page/PageNav/PageNav.js</p>

<p class="info">
NOTE: we could have modified the PageNav to support many different options (e.g. startContent, middleContent, endContent, etc.) but it is left fairly plain for this exercise. Update the component as you see fit.
</p>

Our `PageNav` component can now be extended for use anywhere we need navigation for the application. Any `routes` passed to the component will be rendered as long as they each contain the required properties (e.g. navigation, name, path. etc.) and any children in the component will be rendered in the `nav` element as well. We will have to update the `Page` component so that it works with this udpated `PageNav`. Make the following updates in `Page.js`:

```js/3,8
import React from "react";
import PropTypes from "prop-types";
import PageNav from "./PageNav";
import { routes } from "../../routes";

const Page = ({ children }) => {
  return (
    <div className="page-container">
      <PageNav routes={routes} />
      <main>{children}</main>
    </div>
  );
};

Page.propTypes = {
  children: PropTypes.node,
};

export default Page;
```

<p class="caption">updated Page to make use of routes for PageNav - src/app/components/Page/Page.js</p>

Our `PageNav` component, probably the most complex component in this build, is now fully functional. The remaining `PageHeader` component is more of a placeholder at this point and won't take nearly as long to build.

### The `PageHeader` Component

The `PageHeader` will represent exactly what it's named for, a header at the top of our pages. We will update the `Page` by moving the `PageNav` into the `PageHeader` once we've created it.

Create the necessary `PageHeader` directory files:

```bash/5-7
src/
|-- app/
    |-- components/
        |-- Page
            ...
            |-- PageHeader
                |-- index.js
                |-- PageHeader.js
```

<p class="caption">creating the required PageHeader directory and files</p>

Open the `PageHeader.js` file and enter the following code:

```js
import React from "react";
import PropTypes from "prop-types";

const PageHeader = ({ children }) => {
  return <header>{children}</header>;
};

PageHeader.propTypes = {
  children: PropTypes.node,
};

export default PageHeader;
```

<p class="caption">simple PageHeader component - src/app/components/Page/PageHeader/PageHeader.js</p>

All this component does is serve as a wrapper for any child props. Let's export from the new `index.js` file:

```js
export { default } from "./PagHeader";
```

<p class="caption">exporting the PageHeader component - src/app/components/Page/PageHeader/index.js</p>

Update the `Page` to make use of the new component:

```js/3,9-11
import React from "react";
import PropTypes from "prop-types";
import PageNav from "./PageNav";
import PageHeader from "./PageHeader";
import { routes } from "../../routes";

const Page = ({ children }) => {
  return (
    <div className="page-container">
      <PageHeader>
        <PageNav routes={routes} />
      </PageHeader>
      <main>{children}</main>
    </div>
  );
};

Page.propTypes = {
  children: PropTypes.node,
};

export default Page;
```

<p class="caption">testing the PageNav component in the page - src/app/components/Page/Page.js</p>

Your UI shouldn't appear any different, but if you inspect your dev tools, you should see that the navigation elements are now enclosed in a header element. Later on in the build, the `PageHeader` will also contain our `UserSignInForm` component.

### The `UserSignInForm` Component

Now we can begin to build the application components, starting with the `UserSignInForm`. As presented in the wireframe for the homepage, the sign-in component needs to allow a user to sign into their account by providing their email address and password. Let's begin by creating the necessary directories and files for the `UserSignInForm` component:

```bash/3-6
src/
|-- app/
    |-- components/
        |-- User/
            |-- UserSignInForm/
                |-- index.js
                |-- UserSignInForm.js
```

<p class="caption">creating the required UserSignInForm directories and files</p>

Open the `UserSignInForm.js` file and enter the following code:

<p class="info">
NOTE: all forms will be implemented as <a href="https://reactjs.org/docs/forms.html#controlled-components">controlled components</a>, with submit handlers to be completed in a later part of the series.
</p>

```js
import React, { useState } from "react";

const UserSignInForm = () => {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  return (
    <form>
      <div>
        <label>Email:</label>
        <input
          type="email"
          id="user-email"
          name="userEmail"
          value={email}
          onChange={(event) => setEmail(event.target.value)}
        />
      </div>
      <div>
        <label>Password:</label>
        <input
          type="password"
          id="user-password"
          name="userPassword"
          value={password}
          onChange={(event) => setPassword(event.target.value)}
        />
      </div>
      <div>
        <button type="submit">Sign In</button>
      </div>
    </form>
  );
};

export default UserSignInForm;
```

<p class="caption">simple UserSignInForm component - src/app/components/User/UserSignInForm/UserSignInForm.js</p>

We will worry about the implementation of the form in the next part of this build series (e.g. handling form submission and hiding the sign in form if a user is already signed in). For now, we just need to build the component for UI purposes. Let's export from the new `index.js` file:

```js
export { default } from "./UserSignInForm";
```

<p class="caption">exporting the UserSignInForm component - src/app/components/User/UserSignInForm/index.js</p>

Add the `UserSignInForm` to our `PageHeader` in the `Page`:

```js/4,10-12
import React from "react";
import PropTypes from "prop-types";
import PageNav from "./PageNav";
import PageHeader from "./PageHeader";
import UserSignInForm from "../User/UserSignInForm";
import { routes } from "../../routes";

const Page = ({ children }) => {
  return (
    <div className="page-container">
      <PageHeader>
        <PageNav routes={routes} />
        <UserSignInForm />
      </PageHeader>
      <main>{children}</main>
    </div>
  );
};

Page.propTypes = {
  children: PropTypes.node,
};

export default Page;
```

<p class="caption">testing the PageNav component in the page - src/app/components/Page/Page.js</p>

It's not pretty (yet), but it works.

![Testing the UserSignInForm in the browser](/img/2019-02-12-full-stack-javascript-app-pt-2/signin.png)

<p class="caption">rendering UserSignInForm component</p>

The next four components are page specific, that is, they will only be rendered on the `SignUpPage` or `UserPage`.

### The `UserSignUpForm` Component

Let's start with the `UserSignUpForm`, which will be rendered on the `SignUpPage`. This component will essentially be a copy and paste of the `UserSignInForm` component, so please feel free to use it to get a head start, I know I would.

Start by creating the necessary directories and files for the `UserSignUpForm` component:

```bash/4-7
src/
|-- app/
    |-- components/
        |-- User/
            ...
            |-- UserSignUpForm/
                |-- index.js
                |-- UserSignUpForm.js
```

<p class="caption">creating the required UserSignUpForm directories and files</p>

Open the `UserSignUpForm.js` file and enter the following code:

```js
import React, { useState } from "react";

const UserSignUpForm = () => {
  const [firstName, setFirstName] = useState("");
  const [lastName, setLastName] = useState("");
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [confirmPassword, setConfirmPassword] = useState("");

  return (
    <form>
      <div>
        <label>First name:</label>
        <input
          type="text"
          id="user-first-name"
          name="userFirstName"
          value={firstName}
          onChange={(event) => setFirstName(event.target.value)}
        />
      </div>
      <div>
        <label>Last name:</label>
        <input
          type="text"
          id="user-last-name"
          name="userLastName"
          value={lastName}
          onChange={(event) => setLastName(event.target.value)}
        />
      </div>
      <div>
        <label>Email:</label>
        <input
          type="email"
          id="user-email"
          name="userEmail"
          value={email}
          onChange={(event) => setEmail(event.target.value)}
        />
      </div>
      <div>
        <label>Password:</label>
        <input
          type="password"
          id="user-password"
          name="userPassword"
          value={password}
          onChange={(event) => setPassword(event.target.value)}
        />
      </div>
      <div>
        <label>Confirm password:</label>
        <input
          type="password"
          id="confirm-password"
          name="confirmPassword"
          value={confirmPassword}
          onChange={(event) => setConfirmPassword(event.target.value)}
        />
      </div>
      <div>
        <button type="submit">Sign Up</button>
      </div>
    </form>
  );
};

export default UserSignUpForm;
```

<p class="caption">simple UserSignUpForm component - src/app/components/User/UserSignUpForm/UserSignUpForm.js</p>

Again here for this component, we will complete the implementation of the form in the next part of this build series. Export the UserSignUpForm from the respective `index.js` file:

```js
export { default } from "./UserSignUpForm";
```

<p class="caption">exporting the UserSignUpForm component - src/app/components/User/UserSignUpForm/index.js</p>

Unlike the previous components we've built to this point, the `UserSignUpForm` won't be added to all pages, it only needs to appear on the `SignUpPage`. So, let's update `SignUpPage` to render our new form component:

```js/2,8-9
import React from "react";
import Page from "../components/Page";
import UserSignUpForm from "../components/User/UserSignUpForm";

const SignUpPage = () => {
  return (
    <Page>
      <h1>Sign Up Page</h1>
      <p>Sign Up for a new account by completing the form below.</p>
      <UserSignUpForm />
    </Page>
  );
};

export default SignUpPage;
```

<p class="caption">rendering the UserSignUpForm on the sign up page - src/app/pages/signup.js</p>

Navigate to [http://localhost:3000/signup](http://localhost:3000/signup) to see the sign-up page, and our sign-up form, rendered in the browser.

![Testing the UserSignInForm in the browser](/img/2019-02-12-full-stack-javascript-app-pt-2/signup.png)

<p class="caption">rendering UserSignUpForm component on the sign-up page</p>

Once again, not pretty, but it works.

### The `UserAccountView` Component

Finally, the last component for this build. The `UserAccountView` component will just be used to display the currently signed-in user's account information. You know the drill: create the required directories and files, create and export the component, and render on the user page. Let's get it!

Create the directories and files:

```bash/4-7
src/
|-- app/
    |-- components/
        |-- User/
            ...
            |-- UserAccountView/
                |-- index.js
                |-- UserAccountView.js
```

<p class="caption">creating the required UserAccountView directories and files</p>

Open the `UserAccountView.js` file and enter the following code:

<p class="info">
NOTE: the <code>user</code> prop is required for this component.
<p>

```js
import React from "react";
import PropTypes from "prop-types";

const UserAccountView = ({ user }) => {
  return (
    <div>
      <h2>User Account Details</h2>
      <p>
        <b>First Name: </b>
        {user.firstName}
      </p>
      <p>
        <b>Last Name: </b>
        {user.lastName}
      </p>
      <p>
        <b>Email: </b>
        {user.email}
      </p>
    </div>
  );
};

UserAccountView.propTypes = {
  user: PropTypes.shape({
    firstName: PropTypes.string,
    lastName: PropTypes.string,
    email: PropTypes.string,
  }).isRequired,
};

export default UserAccountView;
```

<p class="caption">simple UserAccountView component - src/app/components/User/UserAccountView/UserAccountView.js</p>

Export the component from `index.js`:

```js
export { default } from "./UserAccountView";
```

<p class="caption">exporting the UserAccountView component - src/app/components/User/UserAccountView/index.js</p>

Render the `UserAccountView` on the `UserPage`:

```js/2,5-10,15
import React from "react";
import Page from "../components/Page";
import UserAccountView from "../components/User/UserAccountView";

const UserPage = () => {
  // dummy test user
  const user = {
    firstName: "Jane",
    lastName: "Doe",
    email: "jdoe@example.com",
  };

  return (
    <Page>
      <h1>User Page</h1>
      <UserAccountView user={user} />
    </Page>
  );
};

export default UserPage;
```

<p class="caption">rendering the UserAccountView on the user page - src/app/pages/user.js</p>

Navigate to [http://localhost:3000/users/:id](http://localhost:3000/users/:id) to see the sign-up page, and our sign-up form, rendered in the browser.

![Testing the UserAccountView in the browser](/img/2019-02-12-full-stack-javascript-app-pt-2/user.png)

<p class="caption">rendering UserAccountView component on the user page</p>

That's a wrap!

## Wrap-Up

Okay, so in this part of the series, we created a few necessary components for the application:

- Page components for general UI
  - PageHeader - provides a general page header for navigation and other important top of the page details
  - PageNav - provides navigation, with support for routes, throughout the application
- Application components for performing tasks and rendering content specific to the application
  - UserAccountView - displays the currently signed-in user's details
  - UserSignInForm - allows a user to sign-in to their account
  - UserSignUpForm - allows a user to sign-up for a new account

## Up Next

In [part 3](/posts/2020-03-24-full-stack-javascript-app-pt-3), we'll look at various methods for styling the components we've just built here. From vanilla CSS to various CSS in JS options, we'll explore the pros and cons of these techniques before finally settling on one for the remainder of the build. I hope you'll continue to join me on this learning journey.
