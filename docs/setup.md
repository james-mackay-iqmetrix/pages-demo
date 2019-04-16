We at the Hub team think [create-react-app](https://github.com/facebook/create-react-app#creating-an-app) (CRA) is a great way to get started, but your team has complete control over how you build your app. For the purposes of this doc, we're going to assume CRA, but the concepts should apply to most applications.

_**All commands are intended to be ran from your application's root directory**_

## Prerequisites

- Node.js v10.14.0+ -- _untested below this version_

- yarn v1.12.3+ -- _untested below this version_

- Add`@iqmetrix:registry=https://packages.iqmetrix.com/npm/npm` to a new line in your [`.npmrc` file](https://docs.npmjs.com/files/npmrc). Create the file if it doesn't exist. This tells npm/yarn where to find our private npm registry.

## Run the Hub shell locally

The Hub shell is the core of the Hub website. The Hub shell handles navigation, authentication, and several other key functions. This shell also contains an iframe which is responsible for displaying applications. In order to build these iframed applications, developers need the ability to emulate the Hub shell during development. Here's our recommended strategy for getting your application running within the Hub shell during development:

1. Install the `@iqmetrix/hub-dev-server` package and saves it as a dev dependency.

   ```code
   lang: bash
   ---
   yarn add @iqmetrix/hub-dev-server --dev
   ```

2. Make sure the package installed correctly

   ```code
   lang: bash
   ---
   yarn hub-dev-server --help
   ```

3. Start your application server at `localhost:3000`. If you're not using CRA, you'll need to check your `package.json` scripts to see how to start your server. You'll need to know the hosted address for the next step.

   ```code
   lang: bash
   ---
   yarn start
   ```

4. In a second shell, start a local copy of the v2 shell at `localhost:9000`. Navigate to `localhost:9000` to view your app running within the v2 shell.

   ```code
   lang: bash
   ---
   yarn hub-dev-server app-host=localhost:3000
   ```

5. We'd recommend looking into [concurrently](https://www.npmjs.com/package/concurrently#usage), or [npm-run-all](https://github.com/mysticatea/npm-run-all) to create a single NPM script to run your entire development environment. Using Concurrently and CRA, you could start your application with the following NPM script:

   ```code
   lang: javascript
   ---
   "dev": "concurrently \"hub-dev-server app-host=localhost:3000\" \"yarn start\""
   ```

   _-- package.json_

## Adding frontend-packages

Once you've gotten your application running within the Hub shell, you'll be able to start using frontend-packages. These packages will allow your application to "talk" to the Hub shell, and get things like an authentication token, or the current environment. Here's how you would modify a CRA app to be able to make authenticated fetch calls to an iqmetrix API:

1. Ensure Promises and an AMD loader are globally available. _We've chosen [Alameda](https://github.com/requirejs/alameda) to be our AMD loader and configured it to load `@iqmetrix` namespaced packages from our CDN._

   ```code
   lang: html
   ---
   <body>

     <!-- include a Promise polyfill if IE11 support is required -->
     <script src="https://cdnjs.cloudflare.com/ajax/libs/es6-promise/4.1.1/es6-promise.auto.js"></script>
     <!-- load and configure AMD loader -->
     <script src="https://cdn.jsdelivr.net/npm/alameda@1.3.0/alameda.min.js"></script>
     <script>
       require.config({
         paths: { "@iqmetrix": "https://hub-cdn.iqmetrix.net/packages" }
       });
     </script>
   </body>
   ```

   _-- index.html_

2. Load the `authenticated-fetch` package from our CDN and assign it to the `authenticatedFetch` variable. _The `authenticatedFetch` function knows how to extract the auth-token from the shell, and how to inject it into fetch calls, so we can use it to make authenticated requests to our services._
   See [here](../packages/functions/authenticated-fetch/README.md) for details on `authenticated-fetch`.
   See [here](./Functions.md) for high-level details on how all of our functions work.
   See [here](../packages/functions) for a complete list of available functions.

   ```code
   lang: javascript
   ---
   // most transpilers don't support top-level await, so we wrap our app in an async function
   (async () => {
     // use window.require to ensure our bundler ignores this at build time
     const [authenticatedFetch] = await window.require([
       "@iqmetrix/authenticated-fetch"
     ]);

     const response = await authenticatedFetch(
       "https://accountsint.iqmetrix.net/v1/me"
     );

     const data = await response.json();

     console.log(data);
   })();
   ```

   _-- index.js_

3. Add authenticated-fetch as a dev dependency. _You may get warnings about unresolved peer dependencies, but they can be disregarded._

   ```code
   lang: bash
   ---
      yarn add @iqmetrix/authenticated-fetch --dev
   ```

4. Use Typescript's [import()](https://davidea.st/articles/typescript-2-9-import-types) functionality to load only type information from the `@iqmetrix/authenticated-fetch` package. _The module's code is never imported into the resulting bundle, but the function signature is now available to develop against._

   ```code
   lang: javascript
   ---
   // use Typescript's import() to import and assign type information
   const [authenticatedFetch] : [typeof import("@iqmetrix/authenticated-fetch")] = await window.require([
     "@iqmetrix/authenticated-fetch"
   ]);
   ```

   _-- index.js_

## Gotchas

1. Once you've added frontend-packages to your application, you'll need to preview your app through the `hub-dev-server`. Many frontend-packages assume the Hub shell is available, and they'll fail if the expected environment hasn't been duplicated.

2. The `hub-dev-server` proxies _all_ application requests to the specified app host. This means requests for "/home", "/reporting", "/templating", etc. will all be proxied to the app host and the sidebar navigation will appear incorrect.
