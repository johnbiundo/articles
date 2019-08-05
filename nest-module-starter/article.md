*John is a member of the NestJS core team, primarily contributing to the documentation.*

This is the first in a multi-part series on building re-usable NPM packages for NestJS. This article focuses on the mechanics of the process with a simple package.  Subsequent articles will delve into more complex NestJS modules.

### Intro
We all interact with the NPM registry on a daily basis.  Even if we're using Yarn, it's powered by that registry. It's part of the foundation that we, as JavaScript developers, rely upon.  There are plenty of tutorials on publishing packages to NPM, and this one adds to that list by focusing on publishing NestJS packages.

Publishing a package for the first time can be pretty exciting. And it's surprisingly easy. I've created this tutorial, along with a [module starter repository]() that you can quickly clone to set things up in a standard way.  This will let you publish a simple package in about 15 minutes.

As to why you want to do this, I offer two great reasons:
1) Learning how to publish a package demystifies one of the things we tend to take for granted. It probably ought to be on the resume of any self-respecting full stack developer. :smiley:
2) Whether you publish to the npmjs.com registry, or a private one, publishing an NPM package is the standard way of creating re-usable packages.

Finally, while the sample package we'll build in this tutorial is trivial, this article is a pre-requisite for a couple more I have planned that will go into detail on much more sophisticated modules.  Stay tuned!

### Pre-requisites
If you plan to follow along live with this article, make sure you have an account at [www.npmjs.com](https://www.npmjs.com).  You could also do this with a private npm registry like [verdaccio](https://github.com/verdaccio/verdaccio), but setting that up is beyond the scope of this article.

These steps use specific names for the package you're building and a companion sample app that will exercise the package. For the purposes of a tutorial, it might be helpful to stick with the names used below.  If you're building your **real** package with its own unique name, just make sure to synchronize the name you choose across the various places it occurs (folders, package.json, npmjs, git, etc.).

### Setting up for development
If you're following along, I recommend you open two command line windows.  One for the package source code, and one for a small sample app that will exercise it (we'll call that 'test-app' below).  The rest of the instructions will refer to these two command line windows.

In window 1, create a working folder that will be the parent for both the package we're building, and our small sample app.

```bash
mkdir nestmod && cd nestmod
```

#### Clone the module starter repo
Start by cloning the repo.

```bash
git clone git@github.com:nestjsplus/nestjs-module-starter.git
```

This will clone a template repo into the subfolder `nestjs-module-starter`.

Install its dependencies with
`npm install`

#### Create your test app
Now create your small sample app that will exercise the package:

```bash
nest new test-app
```

Your folder structure should look like this now:
```
nestmod
└─── nestjs-module-starter
│   └───node_modules
│   └───src
│   ...
└─── test-app
│   └───node_modules
│   └───src
│   ...
```

### Build the package

Take a moment to poke around in the `nestjs-module-starter` folder.  Here are a few things to notice:

1) The `src` folder has two files:
- `test.ts` is the entire functionality of our module; it exports our simple test function to show that we're using the module.
- `index.ts` exports that function from the folder (it's a [barrel file](https://basarat.gitbooks.io/typescript/content/docs/tips/barrel.html)).

2) The top level folder also has an `index.ts` file.  These are important, as collectively they control what gets exported as the public API of the package.

3) The `.npmignore` file in the root folder is like `.gitignore`: it excludes files from the npm package.

4) The `package.json` file has a bunch of interesting parts.  We'll delve into some of them later, but an important one to point out now is the scripts section.  In particular, `build` is where the magic happens. The combination of the build script, folder organization, and `tsconfig.json` file are what ensure that the package is built appropriately.

5) The different types of dependencies are apparent: regular dependencies, peer dependencies, and dev dependencies.  We won't go into details here, but it's worth [reading up on these](https://nodejs.org/es/blog/npm/peer-dependencies/) to make sure you understand them.

In terminal window 1, change to the folder where you cloned the nestjs-module-starter.

```bash
cd nestjs-module-starter
```

Build the package with:
```bash
npm run build
```

### Install the package into the test app
Now you can **use** the package by installing it into `test-app` with a familiar-looking `npm` command.

In terminal window 2, change to the folder where you ran the `nest new test-app` command, where our test app lives.

```bash
cd nestmod/test-app
```

Use `npm` to install the package we just built into our test app.

```bash
npm install ../nestjs-module-starter
```

The main difference from a normal `npm install`, I'm sure you've noticed, is that you are referring to a combination of a path and a name (`../nestjs-module-starter`), rather than the normal name-only.  This illustrates the brilliantly simple process used for developing packages: `npm install` works *exactly the same* for local packages as it does for remote packages.

### Use the package in the test app
The template package comes with a simple test function.  Examine `src/test.ts` in the `nestjs-module-starter` folder:

```typescript
// nestjs-module-starter/src/test.js
export function getHello(): string {
  return 'Hello from module!';
}
```

We can use the exported `getHello()` function to test our newly minted package.

Now that you've installed the package in the test app, you can use it in the normal fashion.  Open `test-app/src/app.controller.ts` and make it look like this:

```typescript
// test-app/src/app.controller.ts
import { Controller, Get } from '@nestjs/common';
import { getHello } from 'nestjs-module-starter';

@Controller()
export class AppController {
  @Get()
  getHello(): string {
    return getHello();
  }
}
```

In terminal window 2, start the app:
```bash
npm run start:dev
```

In a browser, head on over to [http://localhost:3000](http://localhost:3000) and see the message coming from the imported package function.

If you've followed along so far, let's do one more thing that shows how easy it is to continue to iterate.  Make a simple change to the `getHello()` function exported by the package (I like to change it to return `'Buon Giorno!'` :smile:).

Now, make sure the test app is still running in dev mode (e.g., `npm run start:dev`) in terminal window 2, then, in terminal window 1, rebuild the package with:

```bash
npm run build
```
![Dynamic Rebuild](https://thepracticaldev.s3.amazonaws.com/i/cd4ej6jmax30hvdzkjvj.gif)
Notice that in terminal window 2, since we're linked to the local package, the dev server will restart several times as the package is rebuilt (typically showing errors for a few seconds until the process completes).  This highlights a key thing that you will almost certainly forget until it bites you: whenever you make local changes to the package, you must rebuild **before you publish it**.  It's really easy to forget that step - trust me! I have plenty of commits with the message "actually rebuild the module". :smiley:

### Publish the package
We're now ready to push our newborn package out of the nest and onto the 'net (ugghh, sorry for the bad pun). To complete this step, you'll need a free account at [www.npmjs.com](https://www.npmjs.com). If you don't have one, go to the [signup page](https://www.npmjs.com/signup) to get one.

#### Give the package a unique scoped name for NPM

##### A few notes about NPM
- NPM names
All NPM packages have a name.  If the name starts with `@`, it's a *scoped* name. Scoped names are useful to create a unique namespace for your packages.  If you and I both publish `nodejs-module-starter` to npmjs.org, there will be a collision. Whoever publishes first *owns* the name.  But if I publish a *scoped* package name, such as `@johnbiundo/nestjs-module-starter`, you can publish your own copy at `@yournpmjsname/nestjs-module-starter`, and they both happily co-exist.  Anyone that wants mine will do `npm install @johnbiundo/nestjs-module-starter`, and anyone that wants yours will do `npm install @yournpmjsname/nestjs-module-starter`. Read [more about scopes here](https://docs.npmjs.com/misc/scope).

- NPM public packages
NPM lets you publish packages for free as long as they're public.  If you want to publish private packages (say for sharing only within your company), you'll need a private account.  Alternatively, you can set up an internal private registry with a registry server like [verdaccio]().

- NPM orgs
In addition to publishing packages scoped under your name, you can create a free NPM *org* to give your packages a more custom name (scope).  For example, I publish several packages under the org `@nestjsplus`.  Read [more about NPM orgs here](https://docs.npmjs.com/orgs/).

Once you're ready, open the `package.json` file for the package and change the following entries:
- *name*: your scoped package name, something like `@mynpmjsname/my-new-module`
- *author*: your name and email, in a format like `John Biundo <john@email.com>`
- *license*: the template uses MIT; you'll need to research this if you want to do something else
- *keywords*: change this to a list of appropriate keywords to help with searching for your package on npmjs.com

#### Once ready, publish the package
**Note:** Each time you re-publish the package, you'll have to change the `version` field in `package.json` (or the publish step will fail).  It's best practice to use [semantic versioning](https://semver.org/) for NPM packages.

In terminal window 1, run:
```bash
npm publish
```

<br/><br/>
#### Search for your package on www.npmjs.com

![search-npmjs](https://thepracticaldev.s3.amazonaws.com/i/fel4thnhlt90fg4ld02h.png)

<br/><br/>
### Install the published NPM package in your test app
Now that the package is live on npm, you can install it as you would any package.

In terminal window 2, first, uninstall the local one we created and installed about 10 minutes ago.
```bash
npm uninstall nestjs-module-starter
```

Now, install your shiny new NPM package from the internet.  Change the names as appropriate below:

```bash
npm install @yournpmjsname/your-package
```

Finally, edit your test app to reflect the new NPM package name.  For example:

```typescript
// src/app.controller.ts
import { Controller, Get } from '@nestjs/common';
import { getHello } from '@johnbiundo/test-nestjs-module-starter';

@Controller()
export class AppController {
  @Get()
  getHello(): string {
    return getHello();
  }
}
```

Fire up the app, and you'll be using your new package!

Now that you know how easy it is to publish an NPM package, stay tuned for the next articles in this series where we'll cover building more interesting NestJS modules.

Feel free to ask questions, make comments or suggestions, or just say hello in the comments below.  And join us at [Discord](https://discord.gg/G7Qnnhy) for more happy discussions about NestJS.  I post there as *Y Prospect*.