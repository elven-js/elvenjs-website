---
layout: "docs"
title: "Recipes"
publicationDate: "2022-09-19"
tags:
  - intro
excerpt: "Elven.js - the JavaScript SDK for the MultiversX blockchain. Compact and simplified wrapper for sdk-js!"
ogTitle: "Elven.js - JavaScript MultiversX SDK for browsers - Recipes!"
ogDescription: "Elven.js - the JavaScript SDK for the MultiversX blockchain. Compact and simplified wrapper for sdk-js!"
ogUrl: "https://www.elvenjs.com/docs/recipes.html"
twitterTitle: "Elven.js - JavaScript MultiversX SDK for browsers - Recipes!"
twitterDescription: "Elven.js - the JavaScript SDK for the MultiversX blockchain. Compact and simplified wrapper for sdk-js!"
twitterUrl: "https://www.elvenjs.com/docs/recipes.html"
githubUrl: "https://github.com/juliancwirko/elvenjs-website/edit/main/src/docs/recipes.md"
---

In this section, we will check real-world examples. Of course, you can also check the code in many demos linked on the homepage. Let's see the most common cases here.

Remember that you can use the ElvenJS not only in static websites but let's focus only on such ones for simplicity. Check the linked demos on StackBlitz to learn how to use it, for example, with Vue or SolidJS.

### How to login and logout with auth providers

ElvenJS offers two of four auth providers for now. They are the [xPortal Mobile app](https://xportal.com/), [MultiversX browser extension](https://chrome.google.com/webstore/detail/multiversx-defi-wallet/dngmlblcodfobpdpecaadgfbcggfjfnm), and MultiversX Web Wallet. There will also be support for the [Ledger Nano](https://www.ledger.com/) and Ledger Nano.

To be able to login you need to initialize ElvenJs and then use the login function:

```html
<html>
<body>
  <button class="button" id="button-login-extension" style="display: none;">Login with Extension</button>
  <button class="button" id="button-login-mobile" style="display: none;">Login
    with xPortal</button>
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
    } from 'https://unpkg.com/elven.js@0.15.0/build/elven.js';

    // Init ElvenJs 
    const initElven = async () => {
      await ElvenJS.init(
        {
          // Define the API endpoint (can be custom one)
          apiUrl: 'https://devnet-api.multiversx.com',
          // Define the chain type (devnet, mainnet, testnet)
          chainType: 'devnet',
          // Define the API timeout, max 10 sec on public endpoint
          apiTimeout: 10000,
          // Remember to change it. Get yours here: https://cloud.walletconnect.com/sign-in
          walletConnectV2ProjectId: '<your_wc_project_id_here>',
          // WalletConnectV2RelayAddresses are required only for custom addresses
          // by default it will use ['wss://relay.walletconnect.com']
          walletConnectV2RelayAddresses: ['wss://relay.walletconnect.com'],
          // Define login callback functions
          onLoginPending: () => { uiPending(true) },
          onLoggedIn: () => { uiLoggedInState(true); uiPending(false) },
          onLogout: () => { uiLoggedInState(false); },
          // Define transactions callbacks (all are optional)
          onTxStarted: (tx) => { uiPending(true); },
          onTxSent: (tx) => { console.log('Tx sent, but not finalized on the chain!'); },
          onTxFinalized: (tx) => { 
            tx?.hash && updateTxHashContainer(tx.hash); uiPending(false);
          },
          onTxError: (tx, error) => { uiPending(false); },
          onQrPending: () => { uiPending(true); },
          onQrLoaded: () => { uiPending(false); },
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
          await ElvenJS.login('browser-extension');
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
          await ElvenJS.login('mobile', {
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
      TokenTransfer
    } from 'https://unpkg.com/elven.js@0.15.0/build/elven.js';

    // (...) Init and login logic here, check how above

    const egldTransferAddress = 'erd17a4wydhhd6t3hhssvcp9g23ppn7lgkk4g2tww3eqzx4mlq95dukss0g50f';

    // Event listener for predefined EGLD transaction
    document
      .getElementById('button-tx')
      .addEventListener('click', async () => {
        updateTxHashContainer(false);
        const demoMessage = 'Transaction demo from Elven.js!';

        const isGuarded = ElvenJS.storage.get('activeGuardian');
        
        // predefined transaction, this is how it is usually built
        const tx = new Transaction({
          // Get the actal nonce from storage
          nonce: ElvenJS.storage.get('nonce'),
          // Get the receiver of the EGLD
          receiver: new Address(egldTransferAddress),
          // Calculate gas limit (check MultiversX docs)
          // You will need additional 50000 when using guardians
          gasLimit: (isGuarded ? 100000 : 50000) + 1500 * demoMessage.length,
          // Define the chain id (D for the devnet, T for the testnet, 1 for the mainnet)
          chainID: 'D',
          // Build transaction payload data, here very simple string
          data: new TransactionPayload(demoMessage),
          // EGLD value to send
          value: TokenTransfer.egldFromAmount(0.001),
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

As you can see, more logic is involved in building the transaction here. It could look not very easy, but generally, it is just an object created with a couple of helpers exported from sdk-js SDK. So it is very similar to how you would do this with sdk-js.

Transactions are handled in very similar ways. They only need different payload structures and builders. You will find the whole list of them in the [SDK reference](/docs/sdk-reference.html).

Oh, and by the way, the transaction here is predefined, but you could have your logic that could take all the values from some form, user action etc.

### How to send ESDT 

The same here. Let's not focus on initialization and login. You can check it above in the first point.

Below you will find an example of the ESDT transfer. What is ESDT? These are tokens on the MultiversX network that you can create for yourself. Please read more about them [here](https://docs.multiversx.com/tokens/esdt-tokens/).

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
      Address,
      TokenTransfer,
      TransferTransactionsFactory,
      GasEstimator,
    } from 'https://unpkg.com/elven.js@0.15.0/build/elven.js';

    // (...) Init and login logic here, check how above 

    // ESDT address for demo purpose
    const esdtTransferAddress = 'erd17a4wydhhd6t3hhssvcp9g23ppn7lgkk4g2tww3eqzx4mlq95dukss0g50f';

    // Event listener for triggering the predefined ESDT transaction
    document
      .getElementById('button-tx-esdt')
      .addEventListener('click', async () => {
        updateTxHashContainer(false);

        // We need to build the payment here, we need to provide some data
        // Token id, amount and decimal places (check sdk-js cookbook for more info)
        const payment = TokenTransfer.fungibleFromAmount(
          'BUILDO-890d14',
          '1',
          18
        );

        // Here we are preparing the transfer factory
        // We use default GasEstimator - this way we don't have to worry about providing gas limit vale
        const factory = new TransferTransactionsFactory(new GasEstimator());

        // And here we have actual transaction
        // It doesn't need the value field, because we don't send the EGLD
        const tx = factory.createESDTTransfer({
          tokenTransfer: transfer,
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
      Transaction,
      Address,
      TokenTransfer,
      SmartContract,
      ContractFunction,
      U32Value,
    } from 'https://unpkg.com/elven.js@0.15.0/build/elven.js';

    // (...) Init and login logic here, check how above ... You need to define callbacks for handling transactions

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
        const contractAddress = new Address(nftMinterSmartContract);
        const contract = new SmartContract({ address: contractAddress });

        const isGuarded = ElvenJS.storage.get('activeGuardian');

        const tx = contract.call({
          caller: new Address(ElvenJS.storage.get('address')),
          value: TokenTransfer.egldFromAmount(0.01),
          func: new ContractFunction("mint"),
          // You need 50000 more when using guardians
          gasLimit: isGuarded ?  : 14050000 ? 14000000,
          args: [new U32Value(1)],
          chainID: "D"
        })

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

You can check more about the NFT tokens on the MultiversX blockchain in the docs [here](https://docs.multiversx.com/tokens/nft-tokens/). Also, check the [Elven Tools](https://www.elven.tools) if you want to run your own PFP NFT collection on the MultiversX blockchain. Free and open source smart contract, CLI tool, and dApp template.

### How to sign a custom message

You can sign a message using your address as the key. But you don't have to worry about internals here. There is one function for that where you need to provide the message.

```html
<html>
<body>
  <button id="button-tx">Sign a message</button>

  <script type="module">
    import {
      ElvenJS,
    } from 'https://unpkg.com/elven.js@0.15.0/build/elven.js';

    // Initialization first (see above) ... You need to define your callback for signing messages
    
    // Some other stuff here ...

    document
      .getElementById('button-tx')
      .addEventListener('click', async () => {
        try {
          await ElvenJS.signMessage('Elven Family is awesome!');
        } catch (e) {
          throw new Error(e?.message);
        }
      });
  </script>
</body>
</html>
```

### How to verify a custom message on backend

For that you can use `UserVerifier` from `@multiversx/sdk-wallet` and `SignableMessage` from `@multiversx/sdk-core`.

```ts
import { UserVerifier } from "@multiversx/sdk-wallet";
import { SignableMessage } from "@multiversx/sdk-core";

const userVerifier = UserVerifier.fromAddress(addressOfUser);
const message = new SignableMessage({ message: Buffer.from("hello") });
const serializedMessage = message.serializeForSigning();
const messageSignature = Buffer.from("{signature_hex_here}", "hex");

userVerifier.verify(serializedMessage, messageSignature)
```

You can also use other SDKs. For example [sdk-py](https://docs.multiversx.com/sdk-and-tools/sdk-py/).

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
      Address,
      AddressValue,
      ContractFunction,
    } from 'https://unpkg.com/elven.js@0.15.0/build/elven.js';

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

### How to verify the user on the backend side

<div class="docs-box docs-info-box">
You don't need to worry about this section if you don't plan to do any verification on the backend side of your application.
</div>

By default elven.js uses [@multiversx/sdk-native-auth-client](https://www.npmjs.com/package/@multiversx/sdk-native-auth-client) under the hood.

When logging in using one of the signing providers, a `loginToken` will be generated for you. It will then be used to acquire the signature. All with your account address will be then used to create `accessToken`. With that token, you can verify the user on the backend side, for example, using [@multiversx/sdk-native-auth-server](https://www.npmjs.com/package/@multiversx/sdk-native-auth-server).

### Transactions states and execution flow

When you use `ElvenJS.signAndSendTransaction`, a couple of callbacks will be called (You can define them in the `ElvenJS.init`), depending on the progress of current transactions. 

These are: 

- `onTxStarted?: (transaction: Transaction) => void;`
- `onTxSent?: (transaction: Transaction) => void;`
- `onTxFinalized?: (transaction: Transaction) => void;`
- `onTxError?: (transaction: Transaction, error: string) => void;`

They are self-explanatory. `onTxSent` will fire after sending (the transaction object will not contain the signature yet). The `onTxFinalized` will fire after the transaction is finalized on chain (the transaction object will contain the signature).

In case of the error, the `onTxError` will additionally contain the error message.  

### Styling elements

There are a couple of elements that use external styles. Feel free to copy styles if needed. You can check the examples in the `example` directory.

No styles are attached to QR code elements and WalletConnect pairings list by default. But each piece has CSS classes that you can use.

The list of classes: 
- `.elven-qr-code-deep-link`
- `.elven-wc-pairings`
- `.elven-wc-pairings-header`
- `.elven-wc-pairing-item`
- `.elven-wc-pairings-remove-btn`
- `.elven-wc-pairing-item-description`
- `.elven-wc-pairing-item-confirm-msessage`

For more info, check the demo in the `example` directory. Please let us know if you need more styling flexibility and options. Describe your use cases [here](https://github.com/elven-js/elven.js/issues).

### xPortal Hub integration

The Elven.js dApps can integrate with the xPortal Hub. You can read more on how to get your app accepted [here](https://multiversx.notion.site/How-to-submit-your-dApp-for-listing-in-the-xPortal-Hub-863f1005a56943fba38c0e0cb5b1186a).

You don't have to do anything. It should just work when your app is accepted and published in the xPortal Hub.

### Working demos

The demos are linked on the homepage, but let's bring them also here:

- [elvenjs.netlify.app](https://elvenjs.netlify.app/) - EGLD, ESDT transactions, smart contract queries and transactions
- [elrond-donate-widget-demo.netlify.app](https://elrond-donate-widget-demo.netlify.app/) - donation-like widget demo
- [StackBlitz vanilla html demo](https://stackblitz.com/edit/web-platform-d4rx5v?file=index.html)
- [StackBlitz Solid.js demo](https://stackblitz.com/edit/vitejs-vite-rbo6du?file=src/App.tsx)
- [StackBlitz React demo](https://stackblitz.com/edit/vitejs-vite-qr2u7l?file=src/App.tsx)
- [StackBlitz Vue demo](https://stackblitz.com/edit/vue-zrb8y5?file=src/App.vue)

You should be able to find the source code of each under the links.
