# Asset Bundling (Vite)

## Introduction

[Vite][vite] is a modern frontend build tool that provides an extremely fast development environment and bundles your code for production. When building applications with Laravel, you will typically use Vite to bundle your application's CSS and JavaScript files into production ready assets.

Laravel integrates seamlessly with vite by providing an official plugin and Blade directive to load your assets for development and production.

> Vite has replaced Laravel Mix in new Laravel installations. For Mix documentation, please visit the [Laravel Mix][laravel-mix] webiste. If you would like to switch to Vite, please see our [migration guide][migrating-from-laravel-mix-to-vite].

## Installation & Setup

> The following documentation discusses how to manually install and configure the Laravel Vite plugin. However, Laravel's [starter kits][starter-kits] already include all of this scaffolding and are the fastes way to get started with Laravel and Vite.

### Installing Node

You must ensure that Node.js (16+) and NPM are installed before running Vite and the Laravel plugin.

```sh
node -v
npm -v
```

### Installing Vite And The Laravel Plugin

Within a fresh installation of Laravel, you will find a `package.json` file in the root of your application's directory structure. The default `package.json` file already includes everyting you need to get started using Vite and the Laravel plugin. You may install your application's frontend dependencies via NPM.

```php
npm install
```

### Configure Vite

Vite is configured via a `vite.config.js` file in the root of your project. You are free to customize this file based on your needs, and you may also install any other plugins your application requires, such as `@vitejs/plugin-vue` or `@vitejs/plugin-react`.

The Laravel Vite plugin requires you to specify the entry points for your application. These may be JavaScript or CSS files, and include preprocessed languages such as TypeScript, JSX, TSX and Sass.

```js
import { defineConfig } from "vite";
import laravel from "laravel-vite-plugin";

export default defineConfig({
  plugins: [laravel(["resources/css/app.css", "resources/js/app.js"])],
});
```

If you are building an SPA, including applications built using inertia, Vite works best without CSS entry points.

```js
import { defineConfig } from "vite";
import laravel from "laravel-vite-plugin";

export default defineConfig({
  plugins: [laravel(["resources/js/app.js"])],
});
```

Instead, you should import your CSS via JavaScript. Typically, this would be done in your application's `resources/js/app.js` file.

```js
import "./bootstrap";
import "../css/app.css";
```

The Laravel plugin also supports multiple entry points and advanced configuration options such as [SSR entry points][ssr].

#### Working With A Secure Development Server

If your local development web server is serving your application via HTTPS, you may run into issues connecting to the Vite development server.

If you are using [Laravel Valet][valet] for local development and have run the [secure command][valet-securing-sites] against your application, you may configure the Vite development server to automatically use Valet's generated TLS certificates.

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            // ...
            valetTls: 'my-app.test',
        ]),
    ],
});
```

When using another web server, you should generate a trusted certificate and manually configure Vite to use the generated certificates.

```js
import { defineConfig } from 'vite';
import fs from 'fs';

const host = 'my-app.test';

export default defineConfig({
    // ...
    server: {
        host,
        hmr: { host },
        https: {
            key: fs.readFileSync(`/path/to/${host}.key`),
            cert: fs.readFileSync(`/path/to/${host}.crt`);
        },
    },
});
```

If you are unable to generate a trusted certificate for your system, you may install and configure the [@vitejs/plugin-basic-ssl plugin][vite-plugin-basic-ssl]. When using untrusted certificates, you will need to accept the certificate warning for Vite's development server in your browser by following the _Local_ link in your console when running the `npm run dev` command.

### Loading Your Scripts And Styles

With your Vite entry points configured, you only need reference them in a `@vite()` Blade directive that you add to the `<head>` of your application's root template.

```php
<!doctype html>
<head>
    // ...
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
```

If you're importing your CSS via JavaScript, you only need to include the JavaScript entry point.

```php
<!doctype html>
<head>
    // ...
    @vite('resources/js/app.js')
</head>
```

The `@vite` directive will automatically detect the Vite development server and inject the Vite client to enable Hot Module Replacement. In build mode, the directive will load your compiled and versioned assests, including any imported CSS.

If needed, you may also specify the build path of your compiled assets when invoking the `@vite` directive.

```php
<!doctype html>
<head>
    // Given build path is relative to public path.
    @vite('resources/js/app.js', 'vendor/courier/build')
</head>
```

## Running Vite

There are two ways you can run Vite. You may run the development server via the `dev` command, which is useful while developing locally. The development server will automatically detect changes to your files and instantly reflect them in any open browser windows.

Or, running the `build` command will version and bundle your application's assets and get them ready for you to deploy to production.

```sh
# Run the Vite development server...
npm run dev

# Build and version the assets for production...
npm run build
```

## Working With JavaScript

### Aliases

By default, The Laravel plugin provides a common alias to help you hit the ground running and conveniently import your application's assets:

```php
{
    '@' => '/resources/js'
}
```

You may overwrite the `@` alias by adding your own to the `vite.config.js` configuration file.

```js
import { defineConfig } from "vite";
import laravel from "laravel-vite-plugin";

export default defineConfig({
  plugins: [laravel(["resources/ts/app.tsx"])],
  resolve: {
    alias: {
      "@": "/resources/ts",
    },
  },
});
```

### Vue

There are a few additional options you will need to include in the `vite.config.js` configuration file when using the Vue plugin with the Laravel plugin.

```js
import { defineConfig } from "vite";
import laravel from "laravel-vite-plugin";
import vue from "@vitejs/plugin-vue";

export default defineConfig({
  plugins: [
    laravel(["resources/js/app.js"]),
    vue({
      template: {
        transformAssetUrls: {
          // The Vue plugin will re-write assets URLs, when referenced
          // in single File Components, to point to the Laravel web
          // server. Setting this to `null` allows the Laravel plugin
          // to instead re-write asset URLs to point to the vite
          // server instead.
          base: null,

          // The Vue plugin will parse absolute URLs and treat them
          // as absolute paths to files on disk. Setting this
          // `false` will leave absolute URLs un-touched so they can
          // reference assets in the public directory as expected.
          includeAbsolute: false,
        },
      },
    }),
  ],
});
```

> Laravel's [starter kits][starter-kits] already include the proper Laravel, Vue, and Vite configuration. Check out [Laravel Breeze][laravel-breeze] for the fastest way to get started with Laravel, Vue, and Vite.

### React

When using Vite with React, you will need to ensure that any files containing JSX have a `.jsx` or `.tsx` extension, remembering to update your entry point, if required, as [shown above][configure-vite]. You will also need to include the additional `@viteReactRefresh` Blade directive alongside your existing `@vite` directive.

```php
@viteReactRefresh
@vite('resources/js/app.jsx')
```

The `@viteReactRefresh` directive must be called before the `@vite` directive.

> Laravel's [starter kits][starter-kits] already include the proper Laravel, React, and Vite configuration. Check out [Laravel Breeze][laravel-breeze] for the fastest way to get started with Laravel, React, and Vite.

### Inertia

The Laravel Vite plugin provides a convenient `resolvePageComponent` function to help you resolve your Inertia Page components. Below is an example of the helper in use with Vue 3; however; you may also utilize the function in other frameworks such as React -

```js
import { createApp, h } from "vue";
import { createInertiaApp } from "@inertiajs/vue3";
import { resolvePageComponent } from "laravel-vite-plugin/inertia-helpers";

createInertiaApp({
  resolve: (name) =>
    resolvePageComponent(
      `./Pages/${name}.vue`,
      import.meta.glob("./Pages/**/*.vue")
    ),
  setup({ el, App, props, plugin }) {
    return createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el);
  },
});
```

> Laravel's [starter kits][starter-kits] already include the proper Laravel, Inertia, and Vite configuration. Check out [Laravel Breeze][laravel-breeze] for the fastest way to get started with Laravel, Inertia, and Vite.

### URL Processing

When using Vite and referencing assets in your aplication's HTML, CSS or JS, there are a couple of caveats to consider.

[vite]: https://vitejs.dev
[laravel-mix]: https://laravel-mix.com
[migrating-from-laravel-mix-to-vite]: https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-laravel-mix-to-vite
[starter-kits]: https://laravel.com/docs/9.x/starter-kits
[ssr]: https://laravel.com/docs/9.x/vite#ssr
[valet]: https://laravel.com/docs/9.x/valet
[valet-securing-sites]: https://laravel.com/docs/9.x/valet#securing-sites
[vite-plugin-basic-ssl]: https://github.com/vitejs/vite-plugin-basic-ssl
[laravel-breeze]: https://laravel.com/docs/9.x/starter-kits#breeze-and-inertia
[configure-vite]: #configure-vite
