---
layout: "docs"
title: "SDK Reference"
publicationDate: "2022-09-20"
tags:
  - intro
excerpt: "Elven.js - the JavaScript SDK for the Elrond blockchain. Compact and simplified wrapper for erdjs!"
ogTitle: "Elven.js - JavaScript SDK for Elrond blockchain - SDK Reference!"
ogDescription: "Elven.js - the JavaScript SDK for the Elrond blockchain. Compact and simplified wrapper for erdjs!"
ogUrl: "https://www.elvenjs.com/docs/sdk-reference.html"
twitterTitle: "Elven.js - JavaScript SDK for Elrond blockchain - SDK Reference!"
twitterDescription: "Elven.js - the JavaScript SDK for the Elrond blockchain. Compact and simplified wrapper for erdjs!"
twitterUrl: "https://www.elvenjs.com/docs/sdk-reference.html"
githubUrl: "https://github.com/juliancwirko/elvenjs-website/edit/main/src/docs/sdk-reference.md"
---

The Elven.js tool will be as simple as possible. It exports a couple of helper functions. It also exports several data structures (types) from [erdjs](https://docs.elrond.com/sdk-and-tools/erdjs/erdjs/) libraries. Here you will find a description of all the parts, and then you can check the [recipes](/docs/recipes.html) section for real-world examples.

Worth mentioning. Remember to check the source code, written in Typescript, so it should be simple to read. You will find all the source files here: [elven.js/src](https://github.com/juliancwirko/elven.js/tree/main/src).

### Initialization
The primary initialization function. It is responsible for synchronizing with the Elrond network and attaching login/logout callbacks.

Arguments:
`apiUrl`Elrond API URL can be public or private instance,
`chainType`Chain type identificator can be devnet, testnet or mainnet,
- `apiTimeout`: The API calls a timeout in milliseconds. Maximum 10000,
- `onLoginPending`: On login pending callback. It is used across all the auth providers,
- `onLoggedIn`: On logged in callback. It is used across all the auth providers,
- `onLogout`: On logout callback. It is used across all the auth providers

Usage example:

```html
<html>
<body>
  <script type="module">
    import {
      ElvenJS
    } from 'https://unpkg.com/elven.js@0.4.0/build/elven.js';

    const initElven = async () => {
      const isInitialized = await ElvenJS.init(
        {
          apiUrl: 'https://devnet-api.elrond.com',
          chainType: 'devnet',
          apiTimeout: 10000,
          onLoginPending: () => { /* do something when login pending */ },
          onLoggedIn: () => { /* do something when logged in */ },
          onLogout: () => { /* do something when logged out */ },
        }
      );
    }
  </script>
</body>
</html>
```

### Login

```typescript
// function
ElvenJS.login(loginMethod: LoginMethodsEnum, options?: LoginOptions)

// arguments types
enum LoginMethodsEnum {
  ledger = 'ledger', // not implemented yet
  maiarMobile = 'maiar-mobile',
  webWallet = 'web-wallet', // not implemented yet
  maiarBrowserExtension = 'maiar-browser-extension',
}

interface LoginOptions {
  qrCodeContainerId?: string;
  token?: string;
}
```

One interface for logging in with all possible auth providers. It is the core functionality in Elven.js

Arguments:

- `loginMethod`: one of four login methods (ledger, maiar-mobile, web-wallet, maiar-browser-extension) (for now, two of them are implemented)
- `options` as options, you can pass the `token`, which is a unique string that can be used for signature generation and user verification. You can also define `qrCodeContainerId`, the DOM element id in which the maiar-mobile QR code will be displayed

Usage example:

```html
<html>
<body>
  <button id="button-login-extension">Login with extension</button>
  <button id="button-login-mobile">Login with Maiar mobile</button>
  <div id="qr-code-container"></div>

  <script type="module">
    import {
      ElvenJS
    } from 'https://unpkg.com/elven.js@0.4.0/build/elven.js';

    // Initialization first (see above) ...
    
    document
      .getElementById('button-login-extension')
      .addEventListener('click', async () => {
        try {
          await ElvenJS.login('maiar-browser-extension');
        } catch (e) {
          console.log(
            'Login: Something went wrong, try again!', e?.message
          );
        }
      });

    document
      .getElementById('button-login-mobile')
      .addEventListener('click', async () => {
        try {
          await ElvenJS.login('maiar-mobile', {
            qrCodeContainerId: 'qr-code-container',
          });
        } catch (e) {
          console.log(
            'Login: Something went wrong, try again!', e?.message
          );
        }
      });
  </script>
</body>
</html>
```

### TODO