---
layout: "docs"
title: "Getting started"
publicationDate: "2022-09-20"
tags:
  - intro
excerpt: "Elven.js - the JavaScript SDK for the MultiversX blockchain. Compact and simplified wrapper for sdk-js!"
ogTitle: "Elven.js - JavaScript MultiversX SDK for browsers - get started!"
ogDescription: "Elven.js - the JavaScript SDK for the MultiversX blockchain. Compact and simplified wrapper for sdk-js!"
ogUrl: "https://www.elvenjs.com/docs/getting-started.html"
twitterTitle: "Elven.js - JavaScript MultiversX SDK for browsers - get started!"
twitterDescription: "Elven.js - the JavaScript SDK for the MultiversX blockchain. Compact and simplified wrapper for sdk-js!"
twitterUrl: "https://www.elvenjs.com/docs/getting-started.html"
githubUrl: "https://github.com/juliancwirko/elvenjs-website/edit/main/src/docs/getting-started.md"
---


Elven.js is a tool designed to work in browsers without any build steps and integrate MultiversX blockchain and smart contracts built on it.

The tool is a wrapper for [sdk-js](https://docs.multiversx.com/sdk-and-tools/sdk-js/) - a set of Typescript/Javascript libraries. But because the sdk-js toolset is quite extensive and has a lot of Node module dependencies, there is a need for a simple browser-only library. The Elven.js aspire to be something like that - a helpful plug-and-play solution that will give you the most basic functionalities. 

<div class="docs-box docs-info-box">
The quickest way to learn how to use elven.js is to check the source code of the simple example: <a href="https://elvenjs.netlify.app/" target="_blank">elvenjs.netlify.app</a>
</div>

## What Elven.js can do?

The fundamental functionality is connecting and logging in the user using auth providers. For now, Elven.js supports three of four existing - The MultiversX [browser extension](https://chrome.google.com/webstore/detail/multiversx-defi-wallet/dngmlblcodfobpdpecaadgfbcggfjfnm) (MultiversX Defi), [xPortal Mobile app](https://xportal.com/), and MultiversX Web Wallet. In the future, it will also support [Ledger Nano](https://www.ledger.com/) hardware wallet. It also supports [xPortal](https://xportal.com/) Hub integration and [xAlias](https://xalias.com/).

So basically, the user can attach the Elven.js script and start authenticating users on the front end. There is also an option to pass a unique token and get a signature after authentication, which you can use for additional backend verification. You will read more about it in the [recipes](/docs/recipes.html) section.

Besides authentication, Elven.js will also help with all the interactions, like sending native $EGLD tokens or even ESDT tokens. It will allow you to make most transactions, including interactions with custom smart contracts. It allows to sign custom messages. There is also a possibility to query smart contracts. For now, you need to decode the returned data using custom logic, but there will be a separate utility soon.

## How is Elven.js built?

In the end, the Elven.js library is a simple browser-based JS script. You can copy it from the repository and use it as a local JavaScript file or import it directly from the CDN.

The script is a wrapper for sdk-js, an official JavaScript SDK for usage in the Nodejs ecosystem. JS SDK can also be used in the browser but requires additional build steps and configuration, and the file itself will be huge. This is why Elven.js exists. 

Elven.js imports only crucial parts of the sdk-js libraries. It will include the most used parts of the sdk-js, which are essential for the browsers. All other functionality will be split into separate scripts, and it will be shipped as optional scripts in the future.

Internally Elven.js uses Typescript, but finally, it is a standard minified JavaScript file. It can be used in static websites and frameworks like React, SolidJS, or Vue. It is worth mentioning that there are official tools for React already, so it will probably be a better choice to check them. You could also check the [MultiversX NextJs dApp template](https://github.com/xdevguild/nextjs-dapp-template).

## How to start using it?

Because Elven.js is just a JavaScript file, you can import it from CDN and start using its inner parts. You can also copy the file version from the [build](https://github.com/elven-js/elven.js/tree/main/build) directory in the repository.

Let's see how to import it from the CDN:

```html
<html>
  <body>
    <script type="module">
      import { ElvenJS } from 'https://unpkg.com/elven.js@0.16.0/build/elven.js';

      const initElven = async () => {
        await ElvenJS.init(
          {
            apiUrl: 'https://devnet-api.multiversx.com',
            chainType: 'devnet',
            apiTimeout: 10000,
            // Remember to change it. Get yours here: https://cloud.walletconnect.com/sign-in
            walletConnectV2ProjectId: '<your_wc_project_id_here>',
            walletConnectV2RelayAddresses: ['wss://relay.walletconnect.com'],
            onLoginStart: () => {},
            onLoginEnd: () => {},
            onLoginSuccess: () => {},
            onLoginFailure: (error) => {},
            onLogoutStart: () => {},
            onLogoutEnd: () => {},
            onLogoutSuccess: () => {},
            onLogoutFailure: (error) => {},
            onTxStart: (tx) => {},
            onTxSent: (tx) => {},
            onTxFinalized: (tx) => {},
            onTxFailure: (tx, error) => {},
            onQrPending: () => {},
            onQrLoaded: () => {},
            onSignMsgStart: (message) => {},
            onSignMsgFinalized: (message, messageSignature) => {},
            onSignMsgFailure: (message, error) => {},
            onQueryStart: (queryArgs) => {},
            onQueryFinalized: (queryResponse) => {},
            onQueryFailure: (queryArgs, error) => {}
          }
        );
      }

      initElven();
    </script>
  </body>
</html>
```

As you can see here, we have a simplified HTML document where we import the Elven.js SDK parts. Always check the version number (the example from here is v0.16.0: `elven.js@0.16.0`).

You can check the demo examples linked on the homepage.

If you work with a frontend framework for which no tools support the MultiversX blockchain, you can also use the Elven.js tools, installing them as a dependency and then importing from it.

```bash
npm install elven.js --save
```

Example with SolidJs:
```typescript
import { ElvenJS } from 'elven.js';

(...)

onMount(() => {
  const initElven = async () => {
    await ElvenJS.init({
      apiUrl: 'https://devnet-api.multiversx.com',
      chainType: 'devnet',
      apiTimeout: 10000,
      // Remember to change it. Get yours here: https://cloud.walletconnect.com/sign-in
      walletConnectV2ProjectId: '<your_wc_project_id_here>',
      walletConnectV2RelayAddresses: ['wss://relay.walletconnect.com'],
      onLoginStart: () => {},
      onLoginEnd: () => {},
      onLoginSuccess: () => {},
      onLoginFailure: (error) => {},
      onLogoutStart: () => {},
      onLogoutEnd: () => {},
      onLogoutSuccess: () => {},
      onLogoutFailure: (error) => {},
      onTxStart: (tx) => {},
      onTxSent: (tx) => {},
      onTxFinalized: (tx) => {},
      onTxFailure: (tx, error) => {},
      onQrPending: () => {},
      onQrLoaded: () => {},
      onSignMsgStart: (message) => {},
      onSignMsgFinalized: (message, messageSignature) => {},
      onSignMsgFailure: (message, error) => {},
      onQueryStart: (queryArgs) => {},
      onQueryFinalized: (queryResponse) => {},
      onQueryFailure: (queryArgs, error) => {}
    });
  };

  initElven();
});

(...)
```

Because of the ability to adapt to any frontend framework Elven.js is an excellent choice for all widget-like implementations. Imaging WordPress or Shopify plugins with it. Or, for example, simple donation widgets for static hobby websites. You can find such an example here: [elrond-donate-widget-demo.netlify.app](https://elrond-donate-widget-demo.netlify.app/).

## Demo video

The video is a little bit outdated. Check the version and [changelog](https://github.com/elven-js/elven.js/blob/main/CHANGELOG.md). Update soon.

<div class="embeded-media-container">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/tcTukpkjcQw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

## Summary

Okay, so you know what Elven.js is and how to start using it. You are now ready to look at the [SDK reference](/docs/sdk-reference.html) and [recipes](/docs/recipes.html).
 
Also, there are a lot of links to the demo applications on the homepage, so you could also check the code of each one.
