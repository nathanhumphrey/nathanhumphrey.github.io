---
title: Full Stack JavaScript Application Build Part 4 - Backend Database Access and Authentication
description: Part 4 of a series about how to build a full stack JavaScript application. This article is about database access and user authentication on the backend. The frontend implementation of user auth will be covered in phase 2 of Part 4.
abstract: 
date: 2020-04-06 7:00:00.00
tags:
  - software
  - tutorial
  - javascript
  - fullstack
  - koa
  - sequelize
  - passport
layout: layouts/post.njk
---

## Introduction

Welcome to Part 4: Phase 1 of the Full Stack Application series. Part 4 has been broken into two distinct phases, as the entire article had grown quite long. In this phase, we'll be adding database support and some basic user authentication to the application. Database support will be added via [Sequelize ORM](https://sequelize.org/), which will allow us to provision a testing database very easily, and provides a means to access different databases without much fuss. User authentication will be implemented via the [Passport.js](http://www.passportjs.org/) library, which again, will allow for easy initial setup and the ability to update to other auth options very quickly if needed. Phase 2 will see the implementation of this new auth system in the frontend of the application.

<p class="info">
    NOTE: Again, I feel it's warranted to state that the build is intended to provide you with a good base for developing full stack JavaScript applications, but you will need to do some additional learning in order to get this application really production-ready.
</p>

**_The final project source can be found in this [GitHub repo](https://github.com/nathanhumphrey/simple-isomorphic-app/tree/database)_**

### Roadmap

The build is divided into a six-part series:

1. [Routes and pages](/posts/2020-02-06-full-stack-javascript-app-build-pt-1/)
2. [Building components](/posts/2020-02-12-full-stack-javascript-app-pt-2/)
3. [Styling components](/posts/2020-03-24-full-stack-javascript-app-pt-3/)
4. Database access and authentication
    1. [Backend implementation](/posts/2020-04-06-full-stack-javascript-app-pt-4-phase1/) ⇐ you are here
    2. ~~Frontend implementation~~ (coming soon)
5. ~~Protecting against CSRF attacks~~ (coming soon)
6. ~~Production bundling~~ (coming soon)

## Creating the database with Sequelize

What is Sequelize? The website tells us exactly what we're dealing with:

<q>Sequelize is a promise-based Node.js ORM for Postgres, MySQL, MariaDB, SQLite and Microsoft SQL Server. It features solid transaction support, relations, eager and lazy loading, read replication and more.</q> <cite>-- <a href="https://sequelize.org/v5/">Sequelize website</a></cite>

Couldn't have said it better myself! Using Sequelize will allow us to get up and running quickly, and without the need for directly accessing and configuring a database for development. There are many online discussions about why you should or shouldn't choose an ORM for your project, and I'm not married to either position. For this walkthrough, it allows us to move quickly without sacrificing time for discussions that are out of scope for what I want to do here.

### Install Sequelize

Install Sequelize for use in our application:

<p><code class="term">npm i sequelize sequelize-cli</code></p>

Now we can configure Sequelize to create a scaffold for database access. Create a `.sequelizerc` file in the project root directory. This file will be used to configure the initial setup for Sequelize in our project. There are four options that we can set:

- `config` - defines where Sequelize will store and look for its configuration file
- `models-path` - defines where model files will be stored (also an index.js file that exports any models that have been created)
- `seeders-path` - defines where seeder files (basically initial data insert queries) will be stored
- `migrations-path` - defines where migration files (database creation and association queries) will be stored

For our purposes, we won't be using any seeders, but we'll keep the configuration setting for reference purposes. All directories and files will be contained in the `src/db/` directory for the application.

```js
const path = require('path');

module.exports = {
  'config': path.resolve('src/db', 'database.js'),
  'models-path': path.resolve('src/db', 'models'),
  'seeders-path': path.resolve('src/db', 'seeders'),
  'migrations-path': path.resolve('src/db', 'migrations')
}
```

<p class="caption">.sequelizerc - create the sequelize configuration file</p>

### Sequelize initialization

Run the following command to initialize Sequelize:

<p><code class="term">npx sequelize-cli init</code></p>

Once the command completes, you will have a new `src/db/` directory with several child directories and the `database.js` file, which will look the like the following:

```json
{
  "development": {
    "username": "root",
    "password": null,
    "database": "database_development",
    "host": "127.0.0.1",
    "dialect": "mysql",
    "operatorsAliases": false
  },
  "test": {
    "username": "root",
    "password": null,
    "database": "database_test",
    "host": "127.0.0.1",
    "dialect": "mysql",
    "operatorsAliases": false
  },
  "production": {
    "username": "root",
    "password": null,
    "database": "database_production",
    "host": "127.0.0.1",
    "dialect": "mysql",
    "operatorsAliases": false
  }
}
```

<p class="caption">src/db/datatase.js - the default Sequelize database config file</p>

What you might notice is that `database.js` isn't JavaScript but rather JSON, which is the default format. We will make some modifications to this file so that it is valid JavaScript and also supports our development SQLite ([a small, fast, self-contained, high-reliability, full-featured, SQL database engine](https://www.sqlite.org/index.html)) database in a moment, but let's have a look at what's in the file first.

The `database.js` file contains three pretty self-explanatory configuration options:

- development - development database settings
- test -  test database settings (should closely mirror your production database for final testing before deployment)
- production - production database settings

We won't worry about testing or deployment at this stage, so we'll update the file to support our development database only. Update `database.js` with the following code, which will define the dialect for SQLite and create the database file in a new `tmp/` directory. First, create the new `tmp/` under the project root and then update `database.js`:

```js
const path = require('path');

module.exports = {
  development: {
    storage: path.resolve(__dirname, '../../tmp/test.sqlite'),
    dialect: 'sqlite'
  }
};
```

<p class="caption">src/db/datatase.js - updated database.js to support only our dev db</p>

You might be saying to yourself, "but we don't have a `test.sqlite` database file in the tmp directory we just made", and you're right! We have to create the database before it can be used. We could manually create the database with the [sqlite](https://www.sqlite.org/download.html) program and then enter all the required DML statements for tables and initial data, but we don't need to! We can use Sequelize migrations to do that heavy lifting for us. The best part of utilizing Sequelize migrations is that you only have to create the migrations once. Still, you can use them to create and initialize any of the databases that Sequelize supports (e.g. PostgreSQL, MySQL, SQLite, etc.), neat!

### Migrations

[Migrations](https://sequelize.org/v5/manual/migrations.html) allow us to perform any necessary database transition tasks (e.g. create tables, create associations, insert data, etc.) on a database, and also the power to undo those tasks. We don't need much for our application at the moment, just a table to store our user data. The `sequelize-cli` gives us the ability to create our application User model and the Users table in our database via a migration in our project.

<p class="info">
NOTE: the next couple of steps are essentially the same as what's presented on the Sequelize migration documentation page. Creating a user is a pretty standard task.
</p>

First, let's create the User model:

<p><code class="term">npx sequelize-cli model:generate --name User --attributes firstName:string,lastName:string,email:string,passwordHash:string</code></p>

The previous command will create a new `user.js` file in `src/db/models/` and a `XXXXXXXXXXXXXX-create-user.js` in `src/db/migrations/`. I've decided to store only a user's first name, last name, email address, and a password hash (it's not good practice to store plain text passwords). The effects of the command have no effect on the actual database; for that, we'll need to run the migration. Before we run the migration, let's take a look at these two new files.

```js
// src/db/models/user.js
'use strict';
module.exports = (sequelize, DataTypes) => {
  const User = sequelize.define('User', {
    firstName: DataTypes.STRING,
    lastName: DataTypes.STRING,
    email: DataTypes.STRING,
    passwordHash: DataTypes.STRING
  }, {});
  User.associate = function(models) {
    // associations can be defined here
  };
  return User;
};
```

<p class="caption">src/db/models/user.js - auto-generated user model file</p>

```js
// src/db/migrations/XXXXXXXXXXXXXX-create-user.js
'use strict';
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable('Users', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      firstName: {
        type: Sequelize.STRING
      },
      lastName: {
        type: Sequelize.STRING
      },
      email: {
        type: Sequelize.STRING
      },
      passwordHash: {
        type: Sequelize.STRING
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable('Users');
  }
};
```

<p class="caption">src/db/migrations/XXXXXXXXXXXXXX-create-user.js - auto-generated user migration file</p>

You'll notice some variance between the two files, namely that the migration file defines additional fields to the ones we specified. These default fields are:

- `id` - default primary key field added by Sequelize (auto incremented integer)
- `createdAt` - the time at which the model was originally created in the database
- `updatedAt` - the time at which the model was last updated in the database

The two timestamp fields are managed by Sequelize, so we won't have to manage them in our application.

<p class="info">
NOTE: The timestamp fields can be disabled or individually selected if desired. See the <a href="https://sequelize.org/master/manual/model-basics.html">Sequelize model</a> docs for more.
</p>

For the most part, the field definitions are fine but we'll want to make a few small updates:

- I like to use [UUID](https://tools.ietf.org/html/rfc4122)s for the id value, so we'll update that (Sequelize has good [support for UUIDs](https://sequelize.org/master/manual/model-basics.html#uuids))
- The email for each user should be unique and required
- The password hash shoule be required

Finally, in addition to the two field updates, we'll create two additional 'class' methods for our User model to generate password hash values and to check password hash values (i.e. authenticate a user).

- `generatePasswordHash` - this function will just create and return a bcrypt hash for a password
- `authenticate` - this function will find a user by their email and compare the stored hash with a passed in password

As previously mentioned, it's not good practice to store plain text passwords in a database, so we'll install the [bcryptjs](https://www.npmjs.com/package/bcryptjs) library to hash our user passwords:

<p><code class="term">npm i bcryptjs</code></p>

<p class="info">
NOTE: If you'd like to make use of async/await in place of the promise .then() call I've used below, you will need to make use of the <code>@babel/plugin-transform-runtime</code> <a href="https://babeljs.io/docs/en/babel-plugin-transform-runtime">plugin</a>.
</p>


```js/1,7-12,15-23,32-46
// src/db/models/user.js
import bcrypt from 'bcryptjs';

export default (sequelize, DataTypes) => {
  const User = sequelize.define(
    'User',
    {
      id: {
        allowNull: false,
        primaryKey: true,
        type: DataTypes.UUID,
        defaultValue: DataTypes.UUIDV4
      },
      firstName: DataTypes.STRING,
      lastName: DataTypes.STRING,
      email: {
        allowNull: false,
        unique: true,
        type: DataTypes.STRING
      },
      passwordHash: {
        allowNull: false,
        type: DataTypes.STRING
      }
    },
    {}
  );
  
  User.associate = models => {
    // associations can be defined here
  };
  
  User.generatePasswordHash = password => {
    return bcrypt.hashSync(password, 10);
  };

  User.authenticate = (email, password) => {
    return User.findOne({ where: { email } })
      .then(user => {
        if (user && bcrypt.compareSync(password, user.passwordHash)) {
          return user;
        }

        return null;
      })
      .catch(err => err);
  };
  return User;
};
```

<p class="caption">src/db/models/user.js - updated user model file</p>

For the migration file, we only need to make the updates for the `id` and `email` fields:

```js/1,7-8,11-15
// src/db/migrations/XXXXXXXXXXXXXX-create-user.js
export default {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable('Users', {
      id: {
        allowNull: false,
        primaryKey: true,
        type: Sequelize.UUID,
        defaultValue: DataTypes.UUIDV4
      },
      ...
      email: {
        allowNull: false,
        unique: true,
        type: Sequelize.STRING
      },
      ...
```

<p class="caption">src/db/migrations/XXXXXXXXXXXXXX-create-user.js - updated user migration file</p>

We're now ready to migrate the database.

### Migrate the database

Before Sequelize can create our SQLite database, we need to install the corresponding SQLite npm package:

<p><code class="term">npm i sqlite3</code></p>

After the install completes, run the following command to run our migration:

<p><code class="term">npx sequelize-cli db:migrate</code></p>

You should be greeted with an error, and that's because the `db:migrate` command requires that the file already exists; it can't create the file for us. Luckily, we can use JavaScript to programmatically create the file using Sequelize's `sync()` method.

<p class="info">
NOTE: <code>sequelize-cli</code> can create databases with the <code>db:create</code> command, but SQLite databases are not supported. With SQLite, each database is a file on the filesystem, which is different from other RDBMS servers that support the creation of multiple databases (once the server itself is installed of course).
</p>

Sequelize provides a `sync()` method that can be used to create the required database file for us. The method accepts a `force` option, that will overwrite any existing database at the specified configuration. We can update our `src/server/index.js` to make use of this method when we fire up the server.

<p class="warn">
WARN: the <code>sync({force: true})</code> method with <code>force: true</code> option will wipe out your database on every call (i.e. drop and re-create all tables), so any inserts, updates, etc. that have been carried out on the tables in our models will be lost. Use the <code>force: true</code> option with caution.
</p>

```js/1,5,9
...
import db from '../db/models/index'; // Requiring our models for syncing

...

db.sequelize.sync({ force: true }).then(() => {
  app.listen(3000, () => {
    console.log('Server listening on port 3000');
  });
});
```

<p class="caption">src/server/index.js - implement database access for the server</p>

You should now see that our `test.sqlite` file (database) has been created in the `tmp/` directory. You could test that everything worked out as planned by downloading the sqlite executable (linked to earlier) and connecting to the database, but I'll leave that as an exercise for you to carry out on your own (should you feel so inclined). At this point, if you were to re-run the `db:migrate` command from earlier (adding a flag for ES module support), there should be no error, and everything should work fine. So, we've got database access, but now we need to make use of it.

## User Authentication with Passport.js

The whole point of adding database access was so that we could add user authentication and protected routes to our application. The Passport library makes it easy to authenticate users in your node application. By default, Passport is designed to work with any Express-based web application, but there is a [koa-passport](https://www.npmjs.com/package/koa-passport) package that allows the library to work with Koa too. Passport allows you to authenticate users via a large number of supported strategies, from local (typical username/password logins) to social (e.g. facebook, Google, etc.). If you need support for a platform that doesn't already have a strategy, you can create your own!

Moving along, achieving our authentication and protection goals will require a little reworking of our current application, but not too much. The tasks we'll need to undertake are:

- Sign up new users
- Allow users to sign in to the application
- Protect desired routes from unauthorized access
- Allow users to sign out of the application

Let's get it!

### Support server-side api routes

We are currently diverting all routes to the `renderReactApp` middleware, which won't work now that we will have some dedicated server-side routes that don't require a rendered response. We can achieve the separation of server routes from client routes by creating a `src/server/routes.js` file to specifically handle only server-side routes. An example route would like the following:

```js
export const routes = [
  {
    path: '/some-path',
    method: 'get',
    middleware: [
      ctx => {
        // required middleware code here
      }
    ]
  },
];
```

<p class="caption">src/server/routes.js - api routes file</p>

In the example, we can see that we don't need quite the same level of detail that was used for our application routes. For api routes, we will include the following:

- `path` - the desired path to serve
- `method` - the HTTP method to serve
- `middleware` - an array of middleware that should be applied to the matched route (we'll see why I've chosen an array soon)

To allow the server to differentiate between application and api routes, we can create our api router with an `/api` prefix. The api router's prefix will then match incoming request URLs that begin with `/api`. Our api router setup also makes use of the [spread operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) for middleware, to ensure that multiple middleware can be run (middleware is setup as an array in our api `routes.js` file).

```js/2,7-10,17-20,26,27
...
import { routes } from '../app/routes'; // defined application routes
import { apiRoutes } from './routes'; // defined application routes
...
// create the app router
const router = new Router();

// create the api router
const apiRouter = new Router({
  prefix: '/api'
});

// define route(s)
routes.forEach(route => {
  router[route.method](route.path, renderReactApp);
});

// define api route(s)
apiRoutes.forEach(route => {
  router[route.method](route.path, ...route.middleware);
});

// create koa webpack middleware
koaWebpack({ compiler }).then(middleware => {
  app.use(middleware);
  app
    .use(apiRouter.routes())
    .use(apiRouter.allowedMethods())
    .use(router.routes())
    .use(router.allowedMethods())
    .use(koaStatic(path.join(__dirname, '../../static')));
});
```

<p class="caption">src/server/index.js - implement api routes</p>

We are almost ready to define the actual routes for our api, all that's left is to add support for authenticating our users.

### Install Passport and a local strategy

Let's install the koa-flavoured package for passport to get started:

<p><code class="term">npm i koa-passport</code></p>

Next, we will create a new `src/server/user-auth.js` file to hold code related to authenticationg a user (i.e. the default passport stuff). A typical passport requires you to define three things:
- a method for serializing a user (`passport.serializeUser()`)
- a method for deserializing a user (`passport.deserializeUser()`)
- a method for configuring a strategy (`passport.use()`)

The starter code for `user-auth.js` follows:

```js
import passport from 'koa-passport';
import db from '../db/models.js';

passport.serializeUser((user, done) => {
  done(null, user.id);
});

passport.deserializeUser((id, done) => {
  return db.User.findOne({ where: { id } })
    .then(user => {
      done(null, user);
    })
    .catch(err => {
      done(err, null);
    });
});
```

<p class="caption">src/server/user-auth.js - begin support for user authentication</p>

As you can see, serializing and deserializing the user is pretty straightforward. When serializing a user, all we need to store is the user's id, as it can be used to find any unique user in the database. The deserialization method accepts the stored id and then returns the associated user object from the database. Simple.

#### Configure the local strategy

The final thing to add to `user-auth.js` is the desired [local strategy](http://www.passportjs.org/packages/passport-local/). Install the package and then add the final method to the file:

<p><code class="term">npm i passport-local</code></p>

<p class="info">
NOTE: In this application, I've decided to use the email field rather than the default username. By passing the update as an option to Passport, we can define any field as the equivalent of username.
</p>

```js/1,4,20-34
import passport from 'koa-passport';
import { Strategy as LocalStrategy } from 'passport-local';
import db from '../db/models';

const options = { usernameField: 'email' };

passport.serializeUser((user, done) => {
  done(null, user.id);
});

passport.deserializeUser((id, done) => {
  return db.User.findOne({ where: { id } })
    .then(user => {
      done(null, user);
    })
    .catch(err => {
      done(err, null);
    });
});

passport.use(
  new LocalStrategy(options, (email, password, done) => {
    db.User.authenticate(email, password)
      .then(user => {
        if (user === null) {
          return done(null, false);
        } else {
          return done(null, user);
        }
      })
      .catch(err => {
        return done(err);
      });
  })
);
```

<p class="caption">src/server/user-auth.js - updated with configured local strategy</p>

We've configured our strategy to make use of the `User.authenticate()` method we defined in `src/db/models/user.js` earlier. Either we find a user with the supplied email and a password hash that matches the incoming password's hash, or we don't. Regardless of whether authentication is successful or not, we call the `done` callback as per the [Passport docs](http://www.passportjs.org/docs/configure/).

#### Implement passport in our server

The final steps required for our Passport implementation are to setup [session support](https://www.npmjs.com/package/koa-session) and a [body-parser](https://www.npmjs.com/package/koa-bodyparser) to grab data out of incoming requests. Start by installing the following packages:

<p><code class="term">npm i koa-session koa-bodyparser</code></p>

Then, update `src/server/index.js` with the implemenation of these packages so we can begin creating our route middleware:

```js/3-6,11-20
...
import db from '../db/models/index'; // Requiring our models for syncing

import session from 'koa-session';
import passport from 'koa-passport';
import bodyParser from 'koa-bodyparser';
import './user-auth.js';

// create the app server
const app = new Koa();

// sessions
app.keys = ['s3cur3s3cr3t'];
app.use(session(app));

// body parser
app.use(bodyParser());

// authentication
app.use(passport.initialize());
app.use(passport.session());
...
```

<p class="caption">src/server/index.js - implemented authentication support and api routes</p>

### Complete the api route middleware

As mentioned earlier, the current application requires only a few routes:

- a route to sign up (create) a new user
- a route to allow a user to sign in
- a route to allow a user to sign out
- a route to retrieve a user's profile details

#### Signup new user route

To this point, we've been able to avoid using async/await syntax in our application, which isn't a necessity, it just wasn't necessary. But now that we need to define our own Koa middleware, we're going to need async/await support. This can be achieved easily enough by updating our main `index.js` file:

<p class="info">
NOTE: as an exercise, after this update to support async/await, you could go back through the work we've done so far and convert any use of promises to async/await syntax (e.g. the authenticate() function in src/db/models/user.js). It's not required, but could be good practice.
</p>

```js/1-9
require('@babel/register')({
  presets: [
    [
      '@babel/preset-env',
      {
        targets: {
          node: true
        }
      }
    ],
    '@babel/preset-react'
  ],
  ignore: ['node_modules']
});

module.exports = require('./src/server/index.js');
```

<p class="caption">index.js - updated @babel/preset-env with target: 'node' for async/await support</p>

Update our new `src/server/routes.js` file with the following routes:

```js
import { User } from '../db/models';
export const routes = [
  {
    path: '/signup',
    method: 'post',
    middleware: [
      async ctx => {
        // TODO: validate incoming fields
        // TODO: then, check that the user doesn't already exist
        try {
          // create a new user with the password hash from bcrypt
          const user = await User.create(
            Object.assign(ctx.request.body, {
              passwordHash: User.generatePasswordHash(ctx.request.body.password)
            })
          );

          // only send back relevant details
          const { firstName, lastName, email } = user;
          ctx.status = 201; // created
          ctx.type = 'application/json';
          ctx.body = JSON.stringify({ user: { firstName, lastName, email } });
        } catch (err) {
          ctx.status = 400;
          ctx.body = JSON.stringify({
            error: `Could not create new user: ${err}`
          });
        }
      }
    ]
  }
];
```

<p class="caption">src/server/routes.js - implemented user registration route</p>

As you can see from the `TODO` comments, there are a few extras I've left as an exercise for you to work through, but this should get the job done for now. You can test that this route is working with a REST client (e.g. [Postman](https://www.postman.com/)) or even [curl](https://curl.haxx.se/) from the terminal. The following screenshot shows the result of calling this route initially for creation and then again showing the error that is generated due to the unique email constraint:

![Result of posting to api/signup](/img/2020-04-04-full-stack-javascript-app-pt-4/testing-api-signup.png) 

<p class="caption">using curl to post to the api/signup route and the user object response</p>

Let's move on to supporting user sign in.

#### User sign in route

Here will finally make use of Passport. A user must POST their sign-in credentials (e.g. email address and password), the server will try to find a matching user in the database, and then Passport will check the password. Assuming all goes well, the user will be authenticated and could then gain access to protected areas of our application.

```js/1-14
...
{
  path: '/signin',
  method: 'post',
  middleware: [
    passport.authenticate('local'),
    async ctx => {
      // only send back relevant details
      const { id, firstName, lastName, email } = ctx.state.user;
      ctx.status = 200; // ok
      ctx.type = 'application/json';
      ctx.body = JSON.stringify({ user: { id, firstName, lastName, email } });
    }
  ]
},
...
```

<p class="caption">src/server/routes.js - added signin route</p>

The following screenshot shows the result of sending the two supported requests:

1. POST `api/signup` - new user sign up
2. POST `api/signin` - user sign in

![Result of signing in](/img/2020-04-04-full-stack-javascript-app-pt-4/testing-api-signin.png)

<p class="caption">using curl to sign in and the returned user details</p>

the `passport.authenticate()` middleware will authenticate a user and, if successul, place the user into the `ctx.state.user` and then move on to the next middleware in the chain. If authentication is unsuccessful, the server will respond with status code 401 unauthorized, which can then be dealt with on the client accordingly.

#### User profile details route

```js/1-23
...
{
  path: '/users/:id',
  method: 'get',
  middleware: [
    async ctx => {
      // retrieve only fields that should be returned
      const {
        id,
        firstName,
        lastName,
        email,
        createdAt,
        updatedAt
      } = await User.findOne({ where: { id: ctx.params.id } });

      ctx.status = 200; // ok
      ctx.type = 'application/json';
      ctx.body = JSON.stringify({
        user: { id, firstName, lastName, email, createdAt, updatedAt }
      });
    }
  ]
}
...
```

<p class="caption">src/server/routes.js - basic users/:id route implementation</p>

The following screenshot shows the result of sending the now three supported requests:

1. POST `api/signup` - new user sign up
2. POST `api/signin` - user sign in
3. GET `api/users/:id` - view user details

![Result of requesting user details](/img/2020-04-04-full-stack-javascript-app-pt-4/testing-api-details.png)

<p class="caption">using curl to sign up, sign in, and request user details</p>

The problem at the moment is that any account details can be viewed (if you provide a valid user id). We only want to display user details if a user is signed in and the request is for their own details, no other user details should be returned.

### Update protected client routes

Profile route should not be accessible unless a user is authenticated and request is for their user details (i.e. can't see the details of someone else's account). Create a new `requireAuth` middleware in the api routes file to provide an easy way of requiring authentication for desired routes:

```js/3-10
import { User } from '../db/models';
import passport from 'koa-passport';

const requireAuth = async (ctx, next) => {
  if (!ctx.isAuthenticated || !ctx.isAuthenticated()) {
    ctx.status = 401;
    ctx.body = 'Unauthorized';
  } else {
    await next();
  }
};
...
```

<p class="caption">src/server/routes.js - added requireAuth middleware</p>

With this new middleware, we can easily add protection to any desired routes by placing it ahead of any route-specific middleware. For our purposes, we will protect the user details route with this new middleware:

```js/4
{
    path: '/users/:id',
    method: 'get',
    middleware: [
      requireAuth,
      async ctx => {
        const {
          id,
          firstName,
          lastName,
          email,
          createdAt,
          updatedAt
        } = await User.findOne({ where: { id: ctx.params.id } });

        ctx.status = 200; // ok
        ctx.type = 'application/json';
        ctx.body = JSON.stringify({
          user: { id, firstName, lastName, email, createdAt, updatedAt }
        });
      }
    ]
  }
```

<p class="caption">src/server/routes.js - protected users/:id route with requireAuth</p>

Now that we can affirm a user will only see the result if they are authenticated, we need to put a further user match restriction on the requst:

```js/6,21-24
{
  path: '/users/:id',
  method: 'get',
  middleware: [
    requireAuth,
    async ctx => {
      if (ctx.params.id === ctx.state.user.id) {
        const {
          id,
          firstName,
          lastName,
          email,
          createdAt,
          updatedAt
        } = await User.findOne({ where: { id: ctx.params.id } });

        ctx.status = 200; // ok
        ctx.type = 'application/json';
        ctx.body = JSON.stringify({
          user: { id, firstName, lastName, email, createdAt, updatedAt }
        });
      } else {
        ctx.status = 404;
        ctx.body = 'Not found.';
      }
    }
  ]
}
```

<p class="caption">src/server/routes.js - check for user match in users/:id route</p>

The following final screenshot demonstrates both an authenticated and matched user details request as well as an authenticated but unmatched user details request.

![Result of requesting authenticatd user details](/img/2020-04-04-full-stack-javascript-app-pt-4/testing-api-auth-details.png)

<p class="caption">using curl to demonstrate protected user details route</p>

<p class="info">
NOTE: Cookies are required for this exmple to work. Calling curl with the <code>-c &lt;cookie-file&gt;</code> and <code>-b &lt;cookie-file&gt;</code> flags allow for creating cookie files and sending cookie files with the request respectively.
</p>

#### User sign out route

The only functionality left to implement is a sign out feature. Thankfully, Passport makes this easy too. Add the `signout` route to the api routes:

```js/1-11
...
{
  path: '/signout',
  method: 'get',
  middleware: [
    async ctx => {
      ctx.logout();
      ctx.status = 204; // success, no content
    }
  ]
}
...
```

<p class="caption">src/server/routes.js - implementing the signout route</p>

The following final screenshot demonstrates both an authenticated and matched user details request as well as an authenticated but unmatched user details request.

![Result of requesting a signout](/img/2020-04-04-full-stack-javascript-app-pt-4/testing-api-signout.png)

<p class="caption">using curl to demonstrate the signout route</p>

## Conclusion

If you made it this far, congratulations! It took a lot of work to get here, even though the libraries we've used did most of the heavy lifting. At this point, we've got a decent backend that's capable of accessing a database and authenticating users. 

### Next steps

The next steps will be putting this new functionality to work on the frontend in our react application. We'll update our application to support all the new api routes we've developed here. When we're done, we'll have a pretty decent little dev application that we can use to practice or demonstrate a large number of use cases. See you in phase 2!