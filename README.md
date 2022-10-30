# Integrate-Vue3-Inertia-with-Laravel8+SSR
> **NOTICE:** This guide was written using Laravel 8. It also works with Laravel 9 that's using Laravel Mix. If you're using Laravel 9 with Vite, follow this tutorial.

Before we dive in deep, we just want to make sure we have all the tools we need. We'll be using PHP 8, so make sure you have that installed, Composer and NPM. I'll briefly go over how to install Composer and NPM.

# 1. Installing Laravel

Please note that I'm using Laravel 8 and PHP 7.4.32.

Making sure we are in the desired folder we're going to require the Laravel's installer globally and then use it to create a new app called "**laravel8-vue3-inertia-SSR**" (this will automatically create the folder with the same name).

### 1.1 Create Project

*Using this command:*

    composer create-project laravel/laravel laravel8-vue3-inertia-SSR

### 1.2 Change Directory

*Using this command:*

    cd laravel8-vue3-inertia-SSR

### 1.3 Install: npm
*Using this command:*

    npm install

### 1.4 Run

*Using this command:*

    php artisan serv

### 1.5 Result

*The result like this:*

![](https://i.ibb.co/wWNGdm4/5-Result.png)

# 2. Install Vue.js

### 2.1 Install: vue

We'll be using version 3 of Vue. Let's add Vue 3.  

*Using this command:*

    npm install vue@next

### 2.2 Run

*Using this command:*

    npm run dev

# 3. Installing Inertia

Inertia is a new approach to building classic server-driven web apps. We call it the modern monolith.

Inertia allows you to create fully client-side rendered, single-page apps, without much of the complexity that comes with modern SPAs. It does this by leveraging existing server-side frameworks.

Inertia has no client-side routing, nor does it require an API. Simply build controllers and page views like you've always done!

### 3.1 Server-side setup

The first step when installing Inertia is to configure your server-side framework. Inertia ships with official server-side adapters for Laravel and Rails.

##### Install dependencies

Install the Inertia server-side adapters using the preferred package manager for that language or framework.

*Using this command:*

    composer require inertiajs/inertia-laravel
 
##### Middleware

Next, setup the Inertia middleware. In the Rails adapter, this is configured automatically for you. However, in Laravel you need to publish the `HandleInertiaRequests` middleware to your application, which can be done using this artisan command:

*Using this command:*

    php artisan inertia:middleware

Once generated, register the `HandleInertiaRequests` middleware in your `App\Http\Kernel.php`, as the last item in your `web` middleware group.

    'web' => [
        // ...
        \App\Http\Middleware\HandleInertiaRequests::class,
    ],

This middleware provides a version() method for setting your asset version, and a share() method for setting shared data. Please see those pages for more information.

### 3.2 Client-side setup

Once you have your server-side framework configured, you then need to setup your client-side framework. Inertia currently provides support for React, Vue, and Svelte.

##### Install dependencies

Install the Inertia client-side adapters using NPM command.

*Using this command:*

    npm install @inertiajs/inertia @inertiajs/inertia-vue3

##### Progress indicator

Since Inertia requests are made via XHR, there's no default browser loading indicator when navigating from one page to another. To solve this, Inertia provides an optional progress library, which shows a loading bar whenever you make an Inertia visit. To use it, start by installing it.

*Using this command:*

    npm install @inertiajs/progress

# 4. Install Ziggy

Ziggy provides a JavaScript `route()` helper function that works like Laravel's, making it easy to use your Laravel named routes in JavaScript.

Install Ziggy into your Laravel app with.

*Using this command:*

    composer require tightenco/ziggy

Add the `@routes` Blade directive to your main layout (_before_ your application's JavaScript), and the `route()` helper function will now be available globally!

> By default, the output of the `@routes` Blade directive includes a list of all your application's routes and their parameters. This route list is included in the HTML of the page and can be viewed by end users. To configure which routes are included in this list, or to show and hide different routes on different pages, see Filtering Routes.

# 5. Connect everything together

Now we have everything installed and ready to be used. We have installed **Laravel 8, Vue 3, Inertia** and **Ziggy**.

### 5.1 Setup - app.blade.php

Let's start by setting up our one and only **blade** template. We're going to rename the `welcome.blade.php` to `app.blade.php` inside `resources/views`. We're also going to remove all its content and replace it with the following:  

    <!DOCTYPE html>
    <html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    
        <head>
            <meta charset="utf-8">
            <meta name="viewport" content="width=device-width, initial-scale=1">
    
            @routes
            <link href="{{ asset(mix('css/app.css')) }}" rel="stylesheet">
            <script src="{{ asset(mix('js/app.js')) }}" defer></script>
            <script src="{{ asset(mix('js/manifest.js')) }}" defer></script>
            <script src="{{ asset(mix('js/vendor.js')) }}" defer></script>
            <title>Integrate-Vue3-Inertia-with-Laravel8</title>
            @inertiaHead
        </head>
    
        <body>
            @inertia
        </body>
    
    </html>

So first of all you will notice we don't have any `<title>`. This is because we need it to be dynamic and we can set that using Inertia's `<Head>` component. That's why you can see that we've also added the `@inertiaHead` directive.

We have added the `@routes` directive to pass the Laravel's routes in the document's `<head>`. We are importing our `app.css` and also a bunch of `.js` we are going to take care shortly. In the `<body>` we only use the `@inertia` directive which renders a `div` element with a bunch of data passed to it using a `data-page` attribute.

### 5.2 Setup - Ziggy

Let's get back to Ziggy and generate the `.js` file that contains all of our routes. We'll gonna import this into our `app.js` a bit later.

*Using this command:*

    php artisan ziggy:generate resources/js/ziggy.js

To resolve `ziggy` in Vue, we'll have to add an alias to the Vue driver in `webpack.mix.js`:  

    const path = require("path");
    
    // Rezolve Ziggy
    mix.alias({
        ziggy: path.resolve("vendor/tightenco/ziggy/dist/vue"),
    });

### 5.3 Setup - app.js

Let's move on by setting up our app.js file. This is our main main file we're going to load in our blade template.

Now open `resources/js/app.js` and add the following chunk of code:  

    import { createApp, h } from "vue";
    import { createInertiaApp, Link, Head } from "@inertiajs/inertia-vue3";
    import { InertiaProgress } from "@inertiajs/progress";
    
    import { ZiggyVue } from "ziggy";
    import { Ziggy } from "./ziggy";
    
    InertiaProgress.init();
    
    createInertiaApp({
        resolve: async (name) => {
            return (await import(`./Pages/${name}`)).default;
        },
        setup({ el, App, props, plugin }) {
            createApp({ render: () => h(App, props) })
                .use(plugin)
                .use(ZiggyVue, Ziggy)
                .component("Link", Link)
                .component("Head", Head)
                .mixin({ methods: { route } })
                .mount(el);
        },
    });

What does this is to import Vue, Inertia, Inertia Progress and Ziggy and then create the Inertia App. We're also passing the `Link` and `Head` components as globals because we're going to use them a lot.

### 5.4 Setup - Folders & Files

Inertia will load our pages from the `Pages` directory so I'm gonna create 3 demo pages in that folder (`About.vue`, `Contact.vue`,  `Home.vue`). Like so:

![](https://i.ibb.co/jhPky0G/Folders-Files.png)

*Using this command:*
    
    mkdir resources\js\Pages
    touch resources\js\Pages\About.vue
    touch resources\js\Pages\Contact.vue
    touch resources\js\Pages\Home.vue

Each page will container the following template. The `Homepage` text will be replaced based on the file's name:  

    <template>
        <h1>Homepage</h1>
    </template> 

### 5.5 Setup - webpack.mix.js

The next step is to add the missing pieces to the `webpack.mix.js` file. Everything needs to look like this:

    const path = require("path");
    const mix = require("laravel-mix");
    
    // Rezolve Ziggy
    mix.alias({
        ziggy: path.resolve("vendor/tightenco/ziggy/dist/vue"),
    });
    
    // Build files
    mix.js("resources/js/app.js", "public/js")
        .vue({version: 3})
        .webpackConfig({
            resolve: {
                alias: {
                    "@": path.resolve(__dirname, "resources/js"),
                },
            },
        })
        .extract()
        .postCss("resources/css/app.css", "public/css", [//
        ])
        .version();


You can see that we're specifying the Vue version that we're using, we're also setting and alias (`@`) for our root js path and we're also using `.extract()` to split our code into smaller chunks (optional, but better for production in some use cases).

### 5.6 Settup - web.php

We've taken care of almost everything. Not we just need to create routes for each of the Vue pages we have created.

Let's open the `routes/web.php` file and replace everything there with the following:  

    <?php
    
    use Illuminate\Support\Facades\Route;
    use Inertia\Inertia;
    
    Route::get(
        '/',
        static function () {
            return Inertia::render(
                'Home',
                [
                    'title' => 'Homepage',
                ]
            );
        }
    )->name('homepage');
    
    Route::get(
        '/about',
        static function () {
            return Inertia::render(
                'About',
                [
                    'title' => 'About',
                ]
            );
        }
    )->name('about');
    
    Route::get(
        '/contact',
        static function () {
            return Inertia::render(
                'Contact',
                [
                    'title' => 'Contact',
                ]
            );
        }
    )->name('contact');


You can notice right away that we're not returning any traditional blade view. Instead we return an `Inertia::render()` response which takes 2 parameters. The first parameter is the name of our Vue page and the 2nd is an array of properties that will be passed to the Vue page using `$page.props`.

### 5.7 Modifying the Vue pages

Knowing this we can modify our pages to the following template and also add a navigation to them:  

    <template>
    
        <Head>
            <title>{{ $page.props.title }} - Hello World</title>
        </Head>
    
        <body>
            <span>
                <a :href="route('homepage')"><button>Homepage</button></a>
                <a :href="route('about')"><button>About </button></a>
                <a :href="route('contact')"><button>Contact </button></a>
            </span>
    
            <div>
                <h1>This is: {{ $page.props.title }}</h1>
            </div>
        </body>
    </template>

Now we have a simple navigation on each page and also a dynamic page `<title>`.

# 6. Testing

The only thing left now is to compile everything and start the server:

*Using this command:*

    npm install
    npm update vue-loader
    npm run dev
    npm install vue-loader@^16.2.0 --save-dev --legacy-peer-deps
    npm run dev
    php artisan serve
   
Result:

![](https://i.ibb.co/VWjkD7v/Result.png)

---

# 7. Server-Side Rendering (SSR)

SEO is really important so you should have SSR (server side rendering) whenever you are building a SPA (single page application). This is not an issue if you are using Laravel's default templating system, which is Blade, because everything is rendered on the server and the result gets directly shipped into the browser.

In our case the rendering is done on the client side so our DOM is pretty empty. To solve this problem and make our website work even without Javascript enabled we'll need to somehow render everything on and the server side and serve all the resulted HTML on page-load.

Luckily Inertia ships with some tools to help us tackle this issue (https://inertiajs.com/server-side-rendering).

### 7.1 Setting up server side rendering

First, we'll install the necessary server-side rendering dependencies using:

*Using this command:*

    npm install @vue/server-renderer

We'll also install the `@inertiajs/server` package, which provides a simple HTTP server. While this package isn't strictly necessary, it prevents the need to write your own HTTP service, and generally keeps things simple:

*Using this command:*

    npm install @inertiajs/server
    
We are not using `-D` or `--save-dev` flag on these 2 packages because we also need to run them on our production sever. As a general rule for `composer.json` and `package.json`, the packages that are installed inside `require-dev` respectively `devDependencies` should be used only in a development environment.

Next we need to create a server configuration file called `ssr.js` inside the same folder as our `app.js`, meaning `resources/js`. This file will look very similar to our `app.js` file, but instead of being compiled and handled by the browser, it will be handled by Node.js.

*Using this command:*

    touch resources/js/ssr.js

Our `ssr.js` file should look like this:

    import { createSSRApp, h } from "vue";
    import { renderToString } from "@vue/server-renderer";
    import { createInertiaApp, Link, Head } from "@inertiajs/inertia-vue3";
    import createServer from "@inertiajs/server";
    
    import { ZiggyVue } from "ziggy";
    import { Ziggy } from "./ziggy";
    
    createServer((page) =>
        createInertiaApp({
            page,
            render: renderToString,
            resolve: (name) => require(`./Pages/${name}`),
            setup({ app, props, plugin }) {
                return createSSRApp({
                    render: () => h(app, props),
                })
                    .use(plugin)
                    .use(ZiggyVue, Ziggy)
                    .component("Link", Link)
                    .component("Head", Head);
            },
        })
    );

Notice that instead of createApp we import `createSSRApp` from Vue and we're also wrapping our `createInertiaApp` method inside createServer method. We also no longer import Inertia's progress bar because that won't be ever used on server-side.

### 7.2 Setting up Laravel Mix

In order for our Webpack build to run properly on Node, we'll first need to install the webpack-node-externals package:

*Using this command:*

    npm install webpack-node-externals

Then, we'll create a new webpack.ssr.mix.js file for SSR. This is necessary, because at the time of writing this, Laravel Mix does not support multiple webpack configurations within the same webpack.mix.js file:

*Using this command:*

    touch webpack.ssr.mix.js
 
Now we'll create a separate Webpack file called `webpack.ssr.mix.js` which will be used by our Node.js server to compile the resources.

    const path = require("path");
    const mix = require("laravel-mix");
    const webpackNodeExternals = require("webpack-node-externals");

    // Rezolve Ziggy
    mix.alias({
        ziggy: path.resolve("vendor/tightenco/ziggy/dist/vue"),
    });
    
    // Build files
    mix.options({ manifest: false })
        .js("resources/js/ssr.js", "public/js")
        .vue({ version: 3, options: { optimizeSSR: true } })
        .alias({ "@": path.resolve("resources/js") })
        .webpackConfig({
            target: "node",
            externals: [webpackNodeExternals()],
        })
        .version();
 
 As you can see our `webpack.ssr.mix.js` file is really similar to `webpack.mix.js`. You'll need to always remember to also update your SSR webpack file everytime you update your default webpack file with something that can be used on the server-side. Now eveything that's gonna be used by our Node.js server will be bundled to `public/js/ssr.js`.
 
 Now let's publish the Inertia's SSR config file...
 
 *Using this command:*
 
    php artisan vendor:publish --provider="Inertia\ServiceProvider"
    
...and enable the SSR in `config/inertia.php` by changing the enabled flag to `true`.

    <?php
    
    return [
    
        /*
        |--------------------------------------------------------------------------
        | Server Side Rendering
        |--------------------------------------------------------------------------
        |
        | These options configures if and how Inertia uses Server Side Rendering
        | to pre-render the initial visits made to your application's pages.
        |
        | Do note that enabling these options will NOT automatically make SSR work,
        | as a separate rendering service needs to be available. To learn more,
        | please visit https://inertiajs.com/server-side-rendering
        |
        */
    
        'ssr' => [
    
            'enabled' => true,
    
            'url' => 'http://127.0.0.1:13714/render',
    
        ],
    
    // ...
 
### 7.3 Building your application

The only thing that's left to be done now is to compile and to spin up our Node.js server:

*Using this command:*
 
    npx mix
    npx mix --mix-config=webpack.ssr.mix.js

Run both of these build steps and correct any errors that are generated. Remember, you're now building an "isomorphic" app, which means your app runs both on the client (browser) and on the server (Node).
 
### 7.4 Running the Node.js service

With the builds generated, you should be able run the Node server using the following command:

*Using this command:*

    node public/js/ssr.js
    
Now you can even disable the browser's Javascript and you'll still be able to navigate through the site because everything will be rendered on the server side by the Node.js server and served to your browser on page load.

---
# Credit/Reference : 
- https://dev.to/geowrgetudor/server-side-rendering-with-laravel-inertiajs-vuejs-13ap
- https://inertiajs.com/server-side-rendering