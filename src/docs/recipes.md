---
layout: "docs"
title: "Recipes"
publicationDate: "2022-09-19"
tags:
  - intro
excerpt: "Elven.js - the JavaScript SDK for the MultiversX blockchain. Compact and simplified wrapper for erdjs!"
ogTitle: "Elven.js - JavaScript MultiversX SDK for browsers - Recipes!"
ogDescription: "Elven.js - the JavaScript SDK for the MultiversX blockchain. Compact and simplified wrapper for erdjs!"
ogUrl: "https://www.elvenjs.com/docs/recipes.html"
twitterTitle: "Elven.js - JavaScript MultiversX SDK for browsers - Recipes!"
twitterDescription: "Elven.js - the JavaScript SDK for the MultiversX blockchain. Compact and simplified wrapper for erdjs!"
twitterUrl: "https://www.elvenjs.com/docs/recipes.html"
githubUrl: "https://github.com/juliancwirko/elvenjs-website/edit/main/src/docs/recipes.md"
---

In this section, we will check real-world examples. Of course, you can also check the code in many demos linked on the homepage. Let's see the most common cases here.

Remember that you can use the ElvenJS not only in static websites but let's focus only on such ones for simplicity. Check the linked demos on StackBlitz to learn how to use it, for example, with Astro or SolidJS.

### How to login and logout with auth providers

ElvenJS offers two of four auth providers for now. They are the [Maiar Mobile app](https://get.maiar.com/referral/rdmfba3md2), [Maiar browser extension](https://chrome.google.com/webstore/detail/maiar-defi-wallet/dngmlblcodfobpdpecaadgfbcggfjfnm), and MultiversX Web Wallet. There will also be support for the [Ledger Nano](https://www.ledger.com/) and Ledger Nano.

To be able to login you need to initialize ElvenJs and then use the login function:

```html
<html>
<body>
  <button class="button" id="button-login-extension" style="display: none;">Login with Extension</button>
  <button class="button" id="button-login-mobile" style="display: none;">Login
    with Maiar mobile</button>
  <button class="button" id="button-logout" style="display: none;">Logout</button>

  <div id="qr-code-container" class="qr-code-container"></div>

  <script type="module">
    // Just for the demo - helpers
    import {
      uiLoggedInState,
      uiPending,
    } from './demo-ui-tools.js'

    // import ElvenJS parts from CDN 
    import {
      ElvenJS
    } from 'https://unpkg.com/elven.js@0.6.1/build/elven.js';

    // Init ElvenJs 
    const initElven = async () => {
      await ElvenJS.init(
        {
          // Define the API endpoint (can be custom one)
          apiUrl: 'https://devnet-api.elrond.com',
          // Define the chain type (devnet, mainnet, testnet)
          chainType: 'devnet',
          // Define the API timeout, max 10 sec on public endpoint
          apiTimeout: 10000,
          // walletConnectBridgeAddresses is required only for custom addresses
          // by default it will use https://bridge.walletconnect.org
          walletConnectBridgeAddresses: ['https://bridge.walletconnect.org'],
          // Define login callback functions
          onLoginPending: () => { uiPending(true) },
          onLoggedIn: () => { uiLoggedInState(true); uiPending(false) },
          onLogout: () => { uiLoggedInState(false); },
          // Define transactions callbacks
          onTxStarted: () => { uiPending(true); },
          onTxFinalized: (tx) => { 
            tx?.hash && updateTxHashContainer(tx.hash); uiPending(false);
          }
        }
      );
    }

    // Trigger the async init function 
    initElven();

    // Add event listener for extension login button 
    document
      .getElementById('button-login-extension')
      .addEventListener('click', async () => {
        try {
          await ElvenJS.login('maiar-browser-extension');
        } catch (e) {
          console.log('Login: Something went wrong, try again!', e?.message);
        }
      });

    // Add event listener for mobile login button 
    // You will need a container for the qr code
    document
      .getElementById('button-login-mobile')
      .addEventListener('click', async () => {
        try {
          await ElvenJS.login('maiar-mobile', {
            // You can also use the DOM element here: 
            // qrCodeContainer: document.querySelector('#qr-code-container')
            qrCodeContainer: 'qr-code-container',
          });
        } catch (e) {
          console.log('Login: Something went wrong, try again!', e?.message);
        }
      });

    // Add event listener for web login button 
    // You can pass the callback url - the landing page after login on web wallet website
    document
      .getElementById('button-login-web')
      .addEventListener('click', async () => {
        try {
          await ElvenJS.login('web-wallet', { callbackRoute: '/' });
        } catch (e) {
          console.log('Login: Something went wrong, try again!', e?.message);
        }
      });

    // Add event listener for logout button
    document
      .getElementById('button-logout')
      .addEventListener('click', async () => {
        try {
          // Trigger the ElvenJS logout
          const isLoggedOut = await ElvenJS.logout();
        } catch (e) {
          console.error(e.message);
        }
      });
  </script>
</body>
</html>
```

After using one of the login methods, your data will be kept in the localStorage for further usage and synchronization. No worries, nothing private.

From now on, you can sign and send transactions.

### How to send EGLD 

For this example, let's omit the code responsible for initialization and auth. You can check it above. Let's focus on the EGLD operations:

```html
<html>
<body>
  <button class="button" id="button-tx" style="display: none;">EGLD transaction</button>
  <div id="tx-hash-or-query-result" class="tx-hash-or-query-result"></div>

  <script type="module">
    // Just for the demo - helpers
    import {
      uiPending,
      updateTxHashContainer,
    } from './demo-ui-tools.js'

    // import ElvenJS parts from CDN 
    import {
      ElvenJS,
      Transaction,
      Address,
      TransactionPayload,
      TokenPayment
    } from 'https://unpkg.com/elven.js@0.6.1/build/elven.js';

    // (...) Init and login logic here, check how above

    const egldTransferAddress = 'erd17a4wydhhd6t3hhssvcp9g23ppn7lgkk4g2tww3eqzx4mlq95dukss0g50f';

    // Event listener for predefined EGLD transaction
    document
      .getElementById('button-tx')
      .addEventListener('click', async () => {
        updateTxHashContainer(false);
        const demoMessage = 'Transaction demo from Elven.js!';

        // predefined transaction, this is how it is usually built
        const tx = new Transaction({
          // Get the actal nonce from storage
          nonce: ElvenJS.storage.get('nonce'),
          // Get the receiver of the EGLD
          receiver: new Address(egldTransferAddress),
          // Calculate gas limit (check Elrond docs)
          gasLimit: 50000 + 1500 * demoMessage.length,
          // Define the chain id (D for the devnet, T for the testnet, 1 for the mainnet)
          chainID: 'D',
          // Build transaction payload data, here very simple string
          data: new TransactionPayload(demoMessage),
          // EGLD value to send
          value: TokenPayment.egldFromAmount(0.001),
          // Your address, we can get it from the storage, because you should be loggedin
          sender: new Address(ElvenJS.storage.get('address')),
        });

        try {
          // Send the transaction
          await ElvenJS.signAndSendTransaction(tx);
        } catch (e) {
          throw new Error(e?.message);
        }
      });
  </script>
</body>
</html>
```

As you can see, more logic is involved in building the transaction here. It could look not very easy, but generally, it is just an object created with a couple of helpers exported from erdjs SDK. So it is very similar to how you would do this with erdjs.

Transactions are handled in very similar ways. They only need different payload structures and builders. You will find the whole list of them in the [SDK reference](/docs/sdk-reference.html).

Oh, and by the way, the transaction here is predefined, but you could have your logic that could take all the values from some form, user action etc.

### How to send ESDT 

The same here. Let's not focus on initialization and login. You can check it above in the first point.

Below you will find an example of the ESDT transfer. What is ESDT? These are tokens on the Elrond network that you can create for yourself. Please read more about them [here](https://docs.elrond.com/tokens/esdt-tokens/).

```html
<html>
<body>
  <button class="button" id="button-tx-esdt" style="display: none;">ESDT transaction*</button>
  <div id="tx-hash-or-query-result" class="tx-hash-or-query-result"></div>

  <script type="module">
    // Just for the demo - helpers
    import {
      uiPending,
      updateTxHashContainer,
      updateQueryResultContainer
    } from './demo-ui-tools.js'

    // import ElvenJS parts from CDN 
    import {
      ElvenJS,
      ESDTTransferPayloadBuilder,
      Transaction,
      Address,
      TokenPayment,
    } from 'https://unpkg.com/elven.js@0.6.1/build/elven.js';

    // (...) Init and login logic here, check how above 

    // ESDT address for demo purpose
    const esdtTransferAddress = 'erd17a4wydhhd6t3hhssvcp9g23ppn7lgkk4g2tww3eqzx4mlq95dukss0g50f';

    // Event listener for triggering the predefined ESDT transaction
    document
      .getElementById('button-tx-esdt')
      .addEventListener('click', async () => {
        updateTxHashContainer(false);

        // We need to build the payment here, we need to provide some data
        // Token id, amount and decimal places (check erdjs cookbook for more info)
        const payment = TokenPayment.fungibleFromAmount(
          'BUILDO-890d14',
          '1',
          18
        );

        // Here we are preparing the transaction data payload
        const data = new ESDTTransferPayloadBuilder().setPayment(payment).build();

        // And here finally is our transaction. 
        // It doesn't need the value field, because we don't send the EGLD
        const tx = new Transaction({
          data,
          gasLimit: 50000 + 1500 * data.length() + 300000,
          receiver: new Address(esdtTransferAddress),
          sender: new Address(ElvenJS.storage.get('address')),
          chainID: 'D',
        });

        try {
          // We use the same function as previously
          const transaction = await ElvenJS.signAndSendTransaction(tx);
        } catch (e) {
          throw new Error(e?.message);
        }
      });
  </script>
</body>
</html>
```

### How to mint NFT

Again, let's not focus on initialization and login. Check these above in the first point.

Here we will mint an NFT on the [Elven Tools Minter Smart Contract](https://www.elven.tools) deployed on the devnet.

```html
<html>
<body>
  <button class="button" id="button-mint" style="display: none;">Mint NFT</button>
  <div id="tx-hash-or-query-result" class="tx-hash-or-query-result"></div>

  <script type="module">
    // Just for the demo - helpers
    import {
      uiPending,
      updateTxHashContainer,
      updateQueryResultContainer
    } from './demo-ui-tools.js'

    // import ElvenJS parts from CDN 
    import {
      ElvenJS,
      ESDTTransferPayloadBuilder,
      Transaction,
      Address,
      TokenPayment,
    } from 'https://unpkg.com/elven.js@0.6.1/build/elven.js';

    // (...) Init and login logic here, check how above ...

    // Here is the Elven Tools demo minter smart contract on the devnet
    // The one we will be calling to mint the NFT
    const nftMinterSmartContract = 'erd1qqqqqqqqqqqqqpgq5za2pty2tlfqhj20z9qmrrpjmyt6advcgtkscm7xep';
    
    // We need an event to trigger the mint
    document
      .getElementById('button-mint')
      .addEventListener('click', async () => {
        updateTxHashContainer(false);

        // Again, we are preparing the data payload using different helpers
        // The function on smart contract is called 'mint'
        // It also takes one argument which is the amount to mint
        const data = new ContractCallPayloadBuilder()
          .setFunction(new ContractFunction('mint'))
          .setArgs([new U32Value(1)])
          .build();

        // Then we need our transaction instance
        const tx = new Transaction({
          data,
          gasLimit: 14000000,
          // The cost of minting on smart contract is set to 0.01 EGLD
          value: TokenPayment.egldFromAmount(0.01),
          chainID: 'D',
          receiver: new Address(nftMinterSmartContract),
          sender: new Address(ElvenJS.storage.get('address')),
        });

        try {
          // We still use the same ElvenJS function for that, 
          // only the transaction instance is different
          const transaction = await ElvenJS.signAndSendTransaction(tx);
        } catch (e) {
          throw new Error(e?.message);
        }
      });
  </script>
</body>
</html>
```

You can check more about the NFT tokens on the MultiversX blockchain in the docs [here](https://docs.elrond.com/tokens/nft-tokens/). Also, check the [Elven Tools](https://www.elven.tools) if you want to run your own PFP NFT collection on the MultiversX blockchain. Free and open source smart contract, CLI tool, and dApp template.

### How to query a smart contract

Smart contracts can offer read-only endpoints/functions that you can query. Let's omit the initialization and login using ElvenJS here. Check it in the first point.

We will query the minter smart contract to get the number of NFTs already minted by the wallet address. Such a functionality is programmed in the Elven Tools Smart Contract, and everyone can do the query.

```html
<html>
<body>
  <button class="button" id="button-query" style="display: none;">Query SC</button>
  <div id="tx-hash-or-query-result" class="tx-hash-or-query-result"></div>

  <script type="module">
    // Just for the demo - helpers
    import {
      uiPending,
      updateTxHashContainer,
      updateQueryResultContainer
    } from './demo-ui-tools.js'

    // import ElvenJS parts from CDN 
    import {
      ElvenJS,
      ESDTTransferPayloadBuilder,
      Transaction,
      Address,
      TokenPayment,
    } from 'https://unpkg.com/elven.js@0.6.1/build/elven.js';

    // (...) Init and login logic here, check how above ...

    // Here the same minter smart contract address as above. We will qery its function
    const nftMinterSmartContract = 'erd1qqqqqqqqqqqqqpgq5za2pty2tlfqhj20z9qmrrpjmyt6advcgtkscm7xep';
    
    document
      .getElementById('button-query')
      .addEventListener('click', async () => {
        try {
          updateQueryResultContainer();
          uiPending(true);

          // Here we use the queryContract function from ElvenJS
          // we need to pass required values
          const results = await ElvenJS.queryContract({
            address: new Address(nftMinterSmartContract),
            // The function on smart contract is called 'getMintedPerAddressTotal'
            func: new ContractFunction('getMintedPerAddressTotal'),
            // As an argument we need to pass our address to check in TypedValue type
            // Check whole list in the SDK reference section
            args: [new AddressValue(new Address(ElvenJS.storage.get('address')))]
          });

          uiPending(false);

          // Manual decoding of a simple type (number here), 
          // there will be additional tools for that
          // We know that it should be a number, so we can simply decode it using
          // helper functions like base64 to decimal hex
          // and then parse the hex number
          // You'll find an example of such helper tool in the example directory in the repository
          const hexVal = base64ToDecimalHex(results?.returnData?.[0]);
          updateQueryResultContainer(`➡️ The result of the query is: ${parseInt(hexVal, 16)}`);
        } catch (e) {
          uiPending(false);
          throw new Error(e?.message);
        }
      });
  </script>
</body>
</html>
```

### How to use the login token

First of all, using the login token in only the frontend-based app isn't required. This is more like additional verification of the user on the backend side of your application.

The login token is your unique hash or whatever you like that is passed when logging in and signed by your wallet using login auth providers. The signature is returned to your browser and saved in localStorage. Such a signature and token can then be sent to your custom backend, and with your public address, the backend can verify it. Then, for example, you could have a logic that will prepare the JWT token and send it back to the frontend app.

The tools and logic required for verification on the backend are out of the scope of this library, but you can use erdjs for that on your Node backend. Here is an example and docs on how to do that:  [Elven Tools Dapp backend integration](https://www.elven.tools/docs/dapp-backend-integration.html#dapp-backend-integration).

On the frontend part with Elven.js, you would only need to pass your token when logging in. This will look like that:

```typescript
(...)

// The token is just an example, it can be anything
ElvenJS.login('maiar-browser-extension', { token: "d052ee8c9acb023d521ef3" })

(...)
```

Then you can get the signature:

```typescript
ElvenJS.storage.get('signature')
```

Then you will need to send three things to your backend, the signature, the token, and your public wallet address. On the backend, you will need to use erdjs and a couple of operations described in the link above to verify the signature.

### Working demos

The demos are linked on the homepage, but let's bring them also here:

- [elvenjs.netlify.app](https://elvenjs.netlify.app/) - EGLD, ESDT transactions, smart contract queries and transactions
- [elrond-donate-widget-demo.netlify.app](https://elrond-donate-widget-demo.netlify.app/) - donation-like widget demo on Astro based blog example
- [StackBlitz vanilla html demo](https://stackblitz.com/edit/web-platform-d4rx5v?file=index.html)
- [StackBlitz Astro demo](https://stackblitz.com/edit/withastro-astro-pwareu?file=src%2Fpages%2Findex.astro)
- [StackBlitz Solid.js demo](https://stackblitz.com/edit/vitejs-vite-rbo6du?file=src/App.tsx)
- [StackBlitz React demo](https://stackblitz.com/edit/vitejs-vite-qr2u7l?file=src/App.tsx)
- [StackBlitz Vue demo](https://stackblitz.com/edit/vue-zrb8y5?file=src/App.vue)

You should be able to find the source code of each under the links.
