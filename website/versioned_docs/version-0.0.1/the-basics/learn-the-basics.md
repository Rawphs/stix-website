---
title: Learn the basics
id: version-0.0.1-learn-the-basics
original_id: learn-the-basics
---

What's in it. Init... As in, `stix init`? Get it? If you don't, that means you skipped the quick start.

Which is fine... Let's just get to it then.

_**Note:** seriously though, take a look at the quick start. It's only 20 sentences including newlines_.

## Introduction

What we'll be going through here is the project structure as generated by stix cli, which is also the recommended way of setting up your project. We'll start off with the bare minimum, and work our way up to the final skeleton.

The boilerplate for a new project isn't that complicated so let's get the boring stuff over with.

> Stix fundamentally works with modules. However, modules aren't required to reap the benefits of using the framework, and we won't be using them for the duration of this guide to keep things simple.

## Foundation

Let's set up the boilerplate for any node based project.

### Directory structure

Power through it:

1. Create a new directory and navigate into it: `mkdir skeleton && cd $_`
2. Create a package.json: `yarn init -y`
3. Create the directory that will hold our code: `mkdir app`

Flex break! 💪

### tsconfig.json

Let's proceed. Create the ts config file in the root of the project.

**./tsconfig.json**

```json
{
  "include": [
    "app/**/*"
  ],
  "compilerOptions": {
    "module": "commonjs",
    "esModuleInterop": true,
    "declaration": true,
    "sourceMap": true,
    "inlineSources": true,
    "target": "ES2017",
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "noImplicitAny": true,
    "moduleResolution": "node",
    "declarationDir": "./dist",
    "outDir": "./dist"
  }
}
```

### Dependencies

Now let's add the dependencies we'll need for our project to run.

`yarn add -D typescript ts-node-dev`

We're adding 2 dependencies. The obvious one is `typescript`, the less obvious one is `ts-node-dev` which allows us to run our project with live reloads making development easier and a lot more fun. You'll see.

### Hello world

Finally, let's create our entry file so we can test the whole thing. Create a file in `app/app.ts`:

```ts
console.log('Hello world!');
```

And take it for a spin: 

`./node_modules/.bin/ts-node-dev --transpileOnly app/app.ts`

Unless I messed up writing this, you should see a kind `hello world` 👋 🌍 .

### Easy running

To make running a tad easier, let's add scripts to our `package.json`:

```json
{
  "name": "skeleton",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "scripts": {
    "dev": "ts-node-dev --transpileOnly app/app.ts",
    "build": "tsc --build tsconfig.json"
  },
  "devDependencies": {
    "ts-node-dev": "^1.0.0-pre.30",
    "typescript": "^3.1.5"
  }
}
```

And now we can run our application using `yarn dev` and build it using `yarn build`.

## Stix it up 🔥

The basics are up and we can get cracking. Let's create a server and create our first hello world route by hand.

_**Note:** this is a lot easier to do using the generator module, but I'm guessing you're reading this to learn what's what so... Let's get our hands dirty._

### Init and configuration

The first thing we'll do is add stix to our skeleton and set up our configuration.

1. Install stix: `yarn add stix`
2. Create the config entry file: `mkdir app/config && touch app/config/index.ts`
3. Create your first config file: `touch app/config/server.ts`

Just so we have something in our config, open up `app/config/server.ts` and add the following:

```ts
export const server = {
  // The port your server will be running on.
  port: 1991,

  // The url. Used by ServerService and meta modules such as stix-swagger.
  url: 'http://localhost',
};
```

Make sure it's exported from your config's entry file by adding the following to `app/config/index.ts`:

```ts
export * from './server';
```

Sweet.

### Entry point

I love a good `hello world` as much as the next developer, but I'd like a bit more than that. 

I think it's time to update our `app/app.ts` file:

```ts
import * as config from './config';
import { Application, ApplicationModes, ServerService } from 'stix';

(async () => {
  const mode: ApplicationModes = process.env.STIX_APPLICATION_MODE as ApplicationModes;
  const app: Application = await new Application(config).launch(mode);

  if (app.getMode() === ApplicationModes.Server && !app.isProduction()) {
    const url = app.getServiceManager().get(ServerService).getURL();

    console.log(`Server running on ${url}`);
  }
})();
```

I know, it's code. Let's go through it.

#### Config

First we import our config. Nothing exciting here. This is simply what we'll be passing to stix.

> Stix uses a configuration throughout the application. This configuration is merged left to right, allowing you to override defaults in order of definition. Don't worry if this sounds confusing right now, it'll make sense later.

#### Stix Application

The `Application` class is the orchestrator of your application. It makes sure the Config gets created, the ModuleManager is loaded and more.

#### Stix ApplicationModes

Stix offers two modes:

- `ApplicationModes.Server`
- `ApplicationModes.Cli`

The reason for this separation is both performance and a way to perform different actions in different modes: a cli command doesn't need a server, and doesn't require all configurations to be loaded. Likewise, a server doesn't care about your CLI commands.

For now we'll be focusing on the server mode, but rest assured the cli mode will come up at a later time.

_**Note:** the default mode is Server._

#### Stix ServerService

Stix's architecture heavily relies on the service manager pattern. One of the services available is the ServerService which, as you might have guessed, is responsible for anything server related.

We'll go more in-depth on the service manager at a later time.

#### Async

The only reason we're using a self-executing function is so that we have a clean way of waiting for our application to finish loading using async/await.

#### Default mode

We're allowing the environment variable `STIX_APPLICATION_MODE` to override the default mode (Server) to be overridden. This is also used by stix cli.

#### The launch

Next up, we create a new Application instance and launch it. We prepend it with `await` so that we know the server has finished booting up in the next lines of code.


#### Mode and is production

If our application is running in server mode and is not in production, we want to log the URL of our dev server. This isn't required of course, but it's nice to have some feedback telling us the server is ready to serve _(get it? serve?)_ our needs.

> Stix uses the environment variable `NODE_ENV` to decide whether or not to run in production. When it's set to production, it's of course production.

#### Url

Next up, we ask stix to get us the service manager, so we can ask it to give us the server service which in turn has a `getURL()` method that constructs the server url based on configuration _(the only configuration we currently have)_.

Finally, we log that url.

### Action time

Yes. I love puns. I'm not going to apologize for that.

> Actions are methods on controllers. They're responsible for producing a response on (generally) http requests.

We're going to be adding our first action! To get this going, there are three things we need:

- Controller config
- Controllers
- Route

Let's start with the config.

#### Controller config

Let's create our controller config by adding the following to a new config file in `app/config/controller.ts`:

```ts
import path from 'path';
import { ControllerManagerConfigType } from 'stix';

export const controller: ControllerManagerConfigType = {
  locations: [ path.resolve(__dirname, '..', 'src', 'Controller') ],
};
```

And export it in `app/config/index.ts`:

```ts
export * from './controller';
```

The ControllerManager _(another service manager)_ allows you to define one or more directories to load your your controllers from. This is a convenient way to load them without having to define each and every one of them individually _(which we'll do at a later time)_.

_**Note:** controllers are also services. The controllers loaded from directories are registered as invokables giving them all the benefits without the hassel. We'll get into what invokables are at a later time_

#### Controller

Now it's time to create that controller we were talking about earlier. Create a file in `app/src/Controller/MyController.ts` with the following contents:

```ts
import { AbstractActionController } from 'stix';

export class MyController extends AbstractActionController {
  public async myAction() {
    return this.okResponse({ Hello: 'world!' });
  }
}
```

Now we have a controller and an action, we can add a route to it and test our application.

> Controllers can optionally extend the `AbstractActionController` which provides a couple of handy methods. This is what you'll need 9/10 times.

#### Route

To add a route, we'll be adding another config file.
Create a file at `app/config/router.ts` with the following content:

```ts
import { Route } from 'stix';
import { MyController } from '../src/Controller/MyController';

export const router = {
  routes: [
    Route.get('/', MyController, 'myAction'),
  ],
};
```

And of course don't forget to add the export to `app/config/index.ts`:

```ts
export * from './router';
```

Routes is simply an array of route objects, and `Route` is a helper to make formatting those easier.

The line `Route.get('/', MyController, 'myAction')` returns the following:

```ts
{ method: 'get', route: '/', controller: MyController, action: 'myAction' }
```

You have access to the following methods on Route:

- `Route.method(method, route, controller, action)`
- `Route.get(route, controller, action)`
- `Route.post(route, controller, action)`
- `Route.put(route, controller, action)`
- `Route.patch(route, controller, action)`
- `Route.delete(route, controller, action)`

### Give it go

With everything set up we should be able to access our new endpoint. Let's try.

1. Start the server: `yarn dev`
2. Check out our endpoint by visiting [http://localhost:1991](http://localhost:1991).

And there you have it, your very first endpoint.

## What's next?

Play around with the skeleton a bit. 
