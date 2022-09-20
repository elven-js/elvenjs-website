---
layout: "docs"
title: "Getting started"
publicationDate: "2022-09-20"
tags:
  - intro
excerpt: "Elven.js - the JavaScript SDK for the Elrond blockchain. Compact and simplified wrapper for erdjs!"
ogTitle: "Elven.js - JavaScript SDK for Elrond blockchain - get started!"
ogDescription: "Elven.js - the JavaScript SDK for the Elrond blockchain. Compact and simplified wrapper for erdjs!"
ogUrl: "https://www.elvenjs.com/docs/getting-started.html"
twitterTitle: "Elven.js - JavaScript SDK for Elrond blockchain - get started!"
twitterDescription: "Elven.js - the JavaScript SDK for the Elrond blockchain. Compact and simplified wrapper for erdjs!"
twitterUrl: "https://www.elvenjs.com/docs/getting-started.html"
githubUrl: "https://github.com/juliancwirko/elvenjs-website/edit/main/src/docs/getting-started.md"
---


Elven.js is a tool designed to work in browsers without any build steps and integrate Elrond blockchain and smart contracts built on it.

The tool is a wrapper for [erdjs SDK](https://docs.elrond.com/sdk-and-tools/erdjs/erdjs/), a set of Typescript/Javascript libraries. But because the toolset is quite extensive and has a lot of Node module dependencies, there is a need for a simple browser-only library. The Elven.js aspire to be something like that - a helpful plug-and-play solution that will give you the most basic functionalities. 

## What Elven.js can do?

The fundamental functionality is connecting and logging the user using auth providers. For now, Elven.js supports two of four existing - The Maiar [browser extension](https://chrome.google.com/webstore/detail/maiar-defi-wallet/dngmlblcodfobpdpecaadgfbcggfjfnm) (Maiar Defi) and [Maiar Mobile app](https://get.maiar.com/referral/rdmfba3md2). In the future, it will also support [Web Wallet](https://wallet.elrond.com/) and [Ledger Nano](https://www.ledger.com/) hardware wallet.

So basically, the user can attach the Elven.js script and start authenticating users on the front end. There is also an option to pass a unique token and get a signature after authentication, which you can use for additional backend verification. You will read more about it in the [recipes](/docs/recipes.html) section.

Besides authentication, Elven.js will also help with all the interactions, like sending native $EGLD tokens or even ESDT tokens. It will allow you to make most transactions, including interactions with custom smart contracts. There is also a possibility to query smart contracts. For now, you need to decode the returned data using custom logic, but there will be a separate utility soon.

## How is Elven.js built?

In the end, the Elven.js library is a simple browser-based JS script. You can copy it from the repository and use it as a local JavaScript file or import it directly from the CDN.

The script is a wrapper for erdjs SDK, an official JavaScript SDK for usage in the Nodejs ecosystem. It can also be used in the browser but requires additional build steps and configuration, and the file itself will be huge. This is why Elven.js exists. 

Elven.js imports only crucial parts of the erdjs libraries. It will include the most used parts of the erdjs, which are essential for the browsers. All other functionality will be split into separate scripts, and it will be shipped as an optional script.

Internally Elven.js uses Typescript, but finally, it is a standard minified JavaScript file. It can be used in static websites and frameworks like React, SolidJS, or Vue. It is worth mentioning that there are official tools for React already, so it will probably be a better choice to check them. You could also check the [Elrond NextJs dApp template](https://github.com/ElrondDevGuild/nextjs-dapp-template).

## How to start using it?

Because Elven.js is just a JavaScript file, you can import it from CDN and start using its inner parts. You can also copy the file version from the [build](https://github.com/juliancwirko/elven.js/tree/main/build) directory in the repository.

Let's see how to import it from the CDN:

```html
<html>
  <body>
    <script type="module">
      import { ElvenJS } from 'https://unpkg.com/elven.js@0.4.0/build/elven.js';
      const initElven = async () => {
        const isInitialized = await ElvenJS.init(
          {
            apiUrl: 'https://devnet-api.elrond.com',
            chainType: 'devnet',
            apiTimeout: 10000,
            onLoginPending: () => {},
            onLoggedIn: () => {},
            onLogout: () => {},
          }
        );
      }
    </script>
  </body>
</html>
```

As you can see here, we have a simplified HTML document where we import the Elven.js SDK parts. Always check the version number (example from here: `elven.js@0.4.0`).

You can check the demo examples linked on the homepage.

If you work with a frontend framework that doesn't support the Elrond blockchain, you can also use the Elven.js tools, installing them as a dependency and then importing from it.

```bash
npm install elven.js --save
```

Example with SolidJs:
```typescript
import { ElvenJS } from 'elven.js';

(...)

onMount(() => {
  const initElven = async () => {
    const isInitialized = await ElvenJS.init({
      apiUrl: 'https://devnet-api.elrond.com',
      chainType: 'devnet',
      apiTimeout: 10000,
      onLoginPending: () => {},
      onLoggedIn: () => {},
      onLogout: () => {},
    });

    setLoggedIn(Boolean(isInitialized));
  };

  initElven();
});

(...)
```

Because of the ability to adapt to any frontend framework Elven.js is an excellent choice for all widget-like implementations. Imaging WordPress or Shopify plugins with it. Or, for example, simple donation widgets for static hobby websites. You can find such an example here: [elrond-donate-widget-demo.netlify.app](https://elrond-donate-widget-demo.netlify.app/).

## Summary

Okay, so you know what Elven.js is and how to start using it. You have a lot of links to the demo applications on the homepage, so the best would be to check the code of each one. Then you will be ready to take a look at the [SDK reference](/docs/sdk-reference.html) and ready to use [recipes](/docs/recipes.html).
