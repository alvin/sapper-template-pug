# sapper-template -- pug [jade] templating edition

### NB. This fork adds pug preprocessing with a few mixins to make blocks a bit nicer
#### see routes/blog/index.html/_index.pug and the components/Nav/ folders for examples with code split into different files for template, js, and styles

An experimental [Sapper](https://github.com/sveltejs/sapper) + PUG/jade template. To clone it and get started:

```
bash
npx degit alvin/sapper-template-pug my-app
cd my-app
npm install
npm run dev
```

Open up [localhost:3000](http://localhost:3000) and start clicking around.


### Options for pug templates:

1. In src/routes/blog/_index.pug for example of a standalone pug file that's included in an index.html Note that at last check, due to the way preloading works, hot-reloading after file changes will only happen if the corresponding index.html is re-saved.   PR's for addressing this (or tweaks to the webpack config to allow just .pug files without an index.html) are also welcome.  

2. We use [https://github.com/kaisermann/svelte-preprocess] which means another way to use pug/jade in any .html file's template

```
<template lang="pug">
   p.container
     | your pug here
</template>
<style type="text/css">
   ... some styles ..
</style>
<script>
   ... some js ..
</script>
```

## PUG Mixins
##### In the interest of syntax clarity vs. embedding literal Svelte template conditions/loops, we provide several block mixins:
```
mixin each(loop)
  | {#each #{loop}}
  if block
    block
  | {/each}

mixin if(condition)
  | {#if #{condition}}
  if block
    block
  | {/if}

mixin else
  | {:else}
  if block
    block

mixin await(promise)
  | {#await #{promise}}
  if block
    block
  | {/await}

mixin then(answer)
  | {:then #{answer}}
  if block
    block

mixin catch(error)
  | {:catch #{error}}
  if block
    block
```

##### This allows the following in preprocesssed Svelte/PUG component templates
```
h1 Recent posts
ul
  +each(`posts as post`)
    li
      a(rel='prefetch' href!='blog/{post.slug}') { post.title }   
```
##### Other ideas?
Some effort was expended trying to port native pug conditionals/loops into the Svelte preprocessing system using compileClient, etc.  However, accounting for all of Svelte's $store magic, etc might prove problematic.  Other ideas and approaches are welcomed, as are PR's with fixes/omissions to our approach here.


Consult [sapper.svelte.technology](https://sapper.svelte.technology) for help getting started.

## Structure

Sapper expects to find three directories in the root of your project —  `app`, `assets` and `routes`.


### app

The [app](app) directory contains the entry points for your app — `client.js`, `server.js` and (optionally) a `service-worker.js` — along with a `template.html` file.


### assets

The [assets](assets) directory contains any static assets that should be available. These are served using [sirv](https://github.com/lukeed/sirv).

In your [service-worker.js](app/service-worker.js) file, you can import these as `assets` from the generated manifest...

```js
import { assets } from './manifest/service-worker.js';
```

...so that you can cache them (though you can choose not to, for example if you don't want to cache very large files).


### routes

This is the heart of your Sapper app. There are two kinds of routes — *pages*, and *server routes*.

**Pages** are Svelte components written in `.html` files. When a user first visits the application, they will be served a server-rendered version of the route in question, plus some JavaScript that 'hydrates' the page and initialises a client-side router. From that point forward, navigating to other pages is handled entirely on the client for a fast, app-like feel. (Sapper will preload and cache the code for these subsequent pages, so that navigation is instantaneous.)

**Server routes** are modules written in `.js` files, that export functions corresponding to HTTP methods. Each function receives Express `request` and `response` objects as arguments, plus a `next` function. This is useful for creating a JSON API, for example.

There are three simple rules for naming the files that define your routes:

* A file called `routes/about.html` corresponds to the `/about` route. A file called `routes/blog/[slug].html` corresponds to the `/blog/:slug` route, in which case `params.slug` is available to the route
* The file `routes/index.html` (or `routes/index.js`) corresponds to the root of your app. `routes/about/index.html` is treated the same as `routes/about.html`.
* Files and directories with a leading underscore do *not* create routes. This allows you to colocate helper modules and components with the routes that depend on them — for example you could have a file called `routes/_helpers/datetime.js` and it would *not* create a `/_helpers/datetime` route


## Webpack config

Sapper uses webpack to provide code-splitting, dynamic imports and hot module reloading, as well as compiling your Svelte components. As long as you don't do anything daft, you can edit the configuration files to add whatever loaders and plugins you'd like.


## Production mode and deployment

To start a production version of your app, run `npm run build && npm start`. This will disable hot module replacement, and activate the appropriate webpack plugins.

You can deploy your application to any environment that supports Node 8 or above. As an example, to deploy to [Now](https://zeit.co/now), run these commands:

```bash
npm install -g now
now
```


## Using external components

When using Svelte components installed from npm, such as [@sveltejs/svelte-virtual-list](https://github.com/sveltejs/svelte-virtual-list), Svelte needs the original component source (rather than any precompiled JavaScript that ships with the component). This allows the component to be rendered server-side, and also keeps your client-side app smaller.

Because of that, it's essential that webpack doesn't treat the package as an *external dependency*. You can either modify the `externals` option in [webpack/server.config.js](webpack/server.config.js), or simply install the package to `devDependencies` rather than `dependencies`, which will cause it to get bundled (and therefore compiled) with your app:

```bash
yarn add -D @sveltejs/svelte-virtual-list
```


## Bugs and feedback

Sapper is in early development, and may have the odd rough edge here and there. Please be vocal over on the [Sapper issue tracker](https://github.com/sveltejs/sapper/issues).
