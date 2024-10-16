---
layout: "docs"
title: "SDK Reference"
publicationDate: "2022-09-20"
tags:
  - intro
excerpt: "Elven.js - the JavaScript SDK for the MultiversX blockchain. Compact and simplified wrapper for sdk-js!"
ogTitle: "Elven.js - JavaScript MultiversX SDK for browsers - SDK Reference!"
ogDescription: "Elven.js - the JavaScript SDK for the MultiversX blockchain. Compact and simplified wrapper for sdk-js!"
ogUrl: "https://www.elvenjs.com/docs/sdk-reference.html"
twitterTitle: "Elven.js - JavaScript MultiversX SDK for browsers - SDK Reference!"
twitterDescription: "Elven.js - the JavaScript SDK for the MultiversX blockchain. Compact and simplified wrapper for sdk-js!"
twitterUrl: "https://www.elvenjs.com/docs/sdk-reference.html"
githubUrl: "https://github.com/juliancwirko/elvenjs-website/edit/main/src/docs/sdk-reference.md"
---

The Elven.js tool will be as simple as possible. It exports a couple of helper functions. It also exports several data structures (types) from [sdk-js](https://docs.multiversx.com/sdk-and-tools/sdk-js/) libraries. Here you will find a description of all the parts, and then you can check the [recipes](/docs/recipes.html) section for real-world examples.

Worth mentioning. Remember to check the source code, written in Typescript. You will find all the source files here: [elven.js/src](https://github.com/elven-js/elven.js/tree/main/src).

### Initialization

**Function**:
```typescript
await ElvenJS.init(initOptions: InitOptions)
```

**Typings**:
```typescript
interface InitOptions {
  apiUrl?: string;
  chainType?: string;
  apiTimeout?: number;
  walletConnectV2ProjectId?: string;
  walletConnectV2RelayAddresses?: string[];
  // Login
  onLoginStart?: () => void;
  onLoginSuccess?: () => void;
  onLoginFailure?: (error: string) => void;
  // Logout
  onLogoutStart?: () => void;
  onLogoutSuccess?: () => void;
  onLogoutFailure?: (error: string) => void;
  // Qr
  onQrPending?: () => void;
  onQrLoaded?: () => void;
  // Transaction
  onTxStart?: (transaction: Transaction) => void;
  onTxSent?: (transaction: Transaction) => void;
  onTxFinalized?: (transaction: Transaction) => void;
  onTxFailure?: (transaction: Transaction, error: string) => void;
  // Signing
  onSignMsgStart?: (message: string) => void;
  onSignMsgFinalized?: (messageSignature: string) => void;
  onSignMsgFailure?: (message: string, error: string) => void;
  // Query
  onQueryStart?: (queryArgs: QueryArguments) => void;
  onQueryFinalized?: (queryResponse: ContractQueryResponse) => void;
  onQueryFailure?: (queryArgs: QueryArguments, error: string) => void;
}
```

The primary initialization function. It is responsible for synchronizing with the MultiversX network and attaching login/logout callbacks.

**Arguments**:

- `apiUrl`: MultiversX API URL - can be the public or private instance,
- `chainType`: Chain type identification - can be devnet, testnet, or mainnet,
- `apiTimeout`: The API call timeout in milliseconds. Maximum 10000,
- `walletConnectV2ProjectId`: Get yours from https://cloud.walletconnect.com/sign-in,
- `walletConnectV2RelayAddresses`: You can pass your custom WalletConnect relay adresses, by default it will use 'wss://relay.walletconnect.com'
- `onLoginStart`: Triggered when the login process started
- `onLoginSuccess`: Triggered when the login process is successful
- `onLoginFailure`: Triggered when the login process failed
- `onLogoutStart`: Triggered when the logout process started
- `onLogoutSuccess`: Triggered when the logout process is successful
- `onLogoutFailure`: Triggered when the logout process failed
- `onQrPending`: Triggered when the Qr element started loading
- `onQrLoaded`:  Triggered when the QR element finished loading
- `onTxStart`: Triggered when the transaction process started
- `onTxSent`: Triggered when the transaction is send, not finalized
- `onTxFinalized`: Triggered when the transaction is finalized
- `onTxFailure`: Triggered when the transaction sign and send filed
- `onSignMsgStart`: Triggered when the message signing process stated
- `onSignMsgFinalized`: Triggered when the message signing process finalized
- `onSignMsgFailure`: Triggered when the message signing process failed
- `onQueryStart`: Triggered when the query process started
- `onQueryFinalized`: Triggered when the query process finalized
- `onQueryFailure`: Triggered when the query process failed

You can import required types directly from elven.js

```typescript
import {
  Transaction,
  QueryArguments,
  ContractQueryResponse
} from 'https://unpkg.com/elven.js@0.20.0/build/elven.js';
```

**Usage example**:
The [demo example](https://elvenjs.netlify.app/) initialization code.

```html
<html>
<body>
  <script type="module">
    import {
      ElvenJS
    } from 'https://unpkg.com/elven.js@0.20.0/build/elven.js';

    const initElven = async () => {
      await ElvenJS.init(
        {
          apiUrl: 'https://devnet-api.multiversx.com',
          chainType: 'devnet',
          apiTimeout: 10000,
          // Remember to change it. Get yours here: https://cloud.walletconnect.com/sign-in
          walletConnectV2ProjectId: '<your_wc_project_id_here>',
          walletConnectV2RelayAddresses: ['wss://relay.walletconnect.com'],
          // All callbacks are optional
          // You could also rely on try catch to some extent, but callbacks in one place seems convenient
          // Login callbacks:
          onLoginStart: () => { uiPending(true) },
          onLoginSuccess: () => { uiLoggedInState(true); },
          onLoginFailure: (error) => { displayError(error); },
          // Logout callbacks:
          onLogoutStart: () => { uiPending(true) },
          onLogoutSuccess: () => { uiLoggedInState(false); },
          onLogoutFailure: (error) => { displayError(error); },
          // Transaction callbacks
          onTxStart: (tx) => { uiPending(true); },
          onTxSent: (tx) => { const hash = tx.getHash().toString(); hash && updateTxHashContainer(hash, true); },
          onTxFinalized: (tx) => { tx?.hash && updateTxHashContainer(tx.hash); uiPending(false); },
          onTxFailure: (tx, error) => { displayError(error); uiPending(false); },
          // Qr code callbacks:
          onQrPending: () => { uiPending(true); },
          onQrLoaded: () => { uiPending(false); },
          // Signing callbacks:
          onSignMsgStart: (message) => { uiPending(true); },
          onSignMsgFinalized: (message, messageSignature) => { messageSignature && updateOperationResultContainer(`➡️ The signature for "${message}" message:\n${messageSignature}`); uiPending(false); },
          onSignMsgFailure: (message, error) => { displayError(error); uiPending(false); },
          // Query callbacks:
          onQueryStart: (queryArgs) => { uiPending(true); },
          onQueryFinalized: (queryResponse) => {
            // Manual decoding of a simple type (number here), there will be additional tools for that using ABI
            // For now please check data converter in Buildo.dev: 
            // https://github.com/xdevguild/buildo.dev/blob/main/components/operations/utils-operations/data-converters.tsx#L103
            const hexVal = base64ToDecimalHex(queryResponse?.returnData?.[0]);
            let intVal = 0;
            if (hexVal) {
              intVal = parseInt(hexVal, 16);
            }
            updateOperationResultContainer(`➡️ The result of the query is: ${intVal}`);
            uiPending(false);
          },
          onQueryFailure: (queryArgs, error) => { displayError(error); uiPending(false); }
        }
      );
    }
  </script>
</body>
</html>
```

### Login

**Function**:
```typescript
await ElvenJS.login(loginMethod: LoginMethodsEnum, options?: LoginOptions)
```

**Typings**:
```typescript
enum LoginMethodsEnum {
  ledger = 'ledger', // not implemented yet
  mobile = 'mobile',
  webWallet = 'web-wallet',
  xAlias = 'x-alias',
  browserExtension = 'browser-extension',
}

interface LoginOptions {
  qrCodeContainer?: string | HTMLElement;
  callbackRoute?: string;
}
```

One interface for logging in with all possible auth providers. It is the core functionality in Elven.js

**Arguments**:

- `loginMethod`: one of five login methods (ledger, mobile, web-wallet, x-alias, browser-extension) (for now, four of them are implemented)
- `options` as options, you can pass the `token`, which is a unique string that can be used for signature generation and user verification. You can also define `qrCodeContainer`, the DOM element id or DOM element in which the mobile QR code will be displayed, and `callbackRoute` used for web-wallet.

**Initialization callbacks**  
Callbacks that will be triggered for that function

```typescript
onLoginStart?: () => void;
onLoginSuccess?: () => void;
onLoginFailure?: (error: string) => void;
```

**Usage example**:

```html
<html>
<body>
  <button id="button-login-extension">Login with extension</button>
  <button id="button-login-mobile">Login with xPortal</button>
  <div id="qr-code-container"></div>

  <script type="module">
    import {
      ElvenJS
    } from 'https://unpkg.com/elven.js@0.20.0/build/elven.js';

    // Initialization first (see above) ...
    
    document
      .getElementById('button-login-extension')
      .addEventListener('click', async () => {
        try {
          await ElvenJS.login('browser-extension');
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
          await ElvenJS.login('mobile', {
            // You can also use the DOM element here: 
            // qrCodeContainer: document.querySelector('#qr-code-container')
            qrCodeContainer: 'qr-code-container',
          });
        } catch (e) {
          console.log(
            'Login: Something went wrong, try again!', e?.message
          );
        }
      });

    document
      .getElementById('button-login-web')
      .addEventListener('click', async () => {
        try {
          await ElvenJS.login('web-wallet', { callbackRoute: '/' });
        } catch (e) {
          console.log('Login: Something went wrong, try again!', e?.message);
        }
      });

    document
      .getElementById('button-login-x-alias')
      .addEventListener('click', async () => {
        try {
          await ElvenJS.login('x-alias', { callbackRoute: '/' });
        } catch (e) {
          console.log('Login: Something went wrong, try again!', e?.message);
        }
      });
  </script>
</body>
</html>
```

### Logout

**Function**:
```typescript
await ElvenJS.logout()
```

Logout function will remove the localStorage entries. It will work the same with each auth provider.

**No arguments**.

**Usage example**:

```html
<html>
<body>
  <button id="button-logout">Logout</button>

  <script type="module">
    import {
      ElvenJS
    } from 'https://unpkg.com/elven.js@0.20.0/build/elven.js';

    // Initialization first (see above) ...
    
    // Some other stuff here ...

    document
      .getElementById('button-logout')
      .addEventListener('click', async () => {
        try {
          await ElvenJS.logout();
        } catch (e) {
          console.error(e.message);
        }
      });
  </script>
</body>
</html>
```

**Initialization callbacks**  
Callbacks that will be triggered for that function

```typescript
onLogoutStart?: () => void;
onLogoutSuccess?: () => void;
onLogoutFailure?: (error: string) => void;
```

### Signing and sending transactions

**Function**:
```typescript
await ElvenJS.signAndSendTransaction(transaction: Transaction)
```

**Typings**:

`Transaction` is the sdk-js exported [Transaction](https://github.com/multiversx/mx-sdk-js-core/blob/main/src/transaction.ts) class.

The sign and send transaction handle one transaction at a time. This is basic functionality that is enough in most cases. The previously prepared transaction instance will be signed and sent with that function. You will use your chosen provider (mobile app, browser extension, etc.) for signing.

**Arguments**:

- `transaction`: the sdk-js transaction class, the transaction instance is the same as the sdk-js one.

**Initialization callbacks**  
Callbacks that will be triggered for that function

```typescript
onTxStart?: (transaction: Transaction) => void;
onTxSent?: (transaction: Transaction) => void;
onTxFinalized?: (transaction: Transaction) => void;
onTxFailure?: (transaction: Transaction, error: string) => void;
```

**Usage example**: 

```html
<html>
<body>
  <button id="button-tx">Send transaction</button>

  <script type="module">
    import {
      ElvenJS,
      Transaction,
      Address,
      TransactionPayload,
      TokenTransfer
    } from 'https://unpkg.com/elven.js@0.20.0/build/elven.js';

    // Initialization first (see above) ...
    
    // Some other stuff here ...

    const egldTransferAddress = 'erd17a4wydhhd6t3hhssvcp9g23ppn7lgkk4g2tww3eqzx4mlq95dukss0g50f';

    document
      .getElementById('button-tx')
      .addEventListener('click', async () => {
        const demoMessage = 'Transaction demo from Elven.js!';

        const isGuarded = ElvenJS.storage.get('activeGuardian');
        // You will need additional 50000 when using guardians
        const gasLimit = (isGuarded ? 100000 : 50000) + 1500 * demoMessage.length;

        const tx = new Transaction({
          nonce: ElvenJS.storage.get('nonce'),
          receiver: new Address(egldTransferAddress),
          gasLimit,
          chainID: 'D',
          data: new TransactionPayload(demoMessage),
          value: TokenTransfer.egldFromAmount(0.001),
          sender: new Address(ElvenJS.storage.get('address')),
        });

        try {
          await ElvenJS.signAndSendTransaction(tx);
        } catch (e) {
          throw new Error(e?.message);
        }
      });
  </script>
</body>
</html>
```

You can also see the transaction instance here. There is a couple of classes exported from sdk-js. You can think of them as helper tools for data preparation. More about them later.

### Signing a message

**Function**:
```typescript
await ElvenJS.signMessage(message: string, options?: { callbackUrl?: string })
```

Sign a custom message using all supported providers. You will get the signature. You can verify the signature wherever you need it. It is usually done on the backend side. To verify, you can use [sdk-js](https://docs.multiversx.com/sdk-and-tools/sdk-js/sdk-js-cookbook#verifying-signatures). Or other MultiversX SDKs.

**Initialization callbacks**  
Callbacks that will be triggered for that function

```typescript
onSignMsgStart?: (message: string) => void;
onSignMsgFinalized?: (messageSignature: string) => void;
onSignMsgFailure?: (message: string, error: string) => void;
```

**Arguments**:

- `message`: message to sign
- `options`: available only for Web Wallet provider. You can set `callbackUrl`

**Usage example**: 

```html
<html>
<body>
  <button id="button-tx">Sign a message</button>

  <script type="module">
    import {
      ElvenJS,
    } from 'https://unpkg.com/elven.js@0.20.0/build/elven.js';

    // Initialization first (see above) ...
    
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

### Querying a smart contract

**Function**:
```typescript
// function
await ElvenJS.queryContract(queryArgs: SmartContractQueryArgs)
```

**Typings**:
```typescript
interface QueryArguments {
  func: IContractFunction;
  args?: TypedValue[];
  value?: ITransactionValue;
  caller?: IAddress;
}

interface SmartContractQueryArgs extends QueryArguments {
  address: IAddress;
}
```

`QueryArguments` is the sdk-js exported [QueryArguments](https://github.com/multiversx/mx-sdk-js-core/blob/main/src/smartcontracts/interface.ts) type. 
Querying smart contracts is possible with this function. You must pass the smart contract address, function name (smart contract endpoint), and arguments (if it takes any). The value and caller are optional.

**Arguments**:

- `address` - `IAddress` interface from sdk-js, you will get it by `new Address(<string_addres>)`
- `func` - `IContractFunction` interface from sdk-js, you will get it by `new ContractFunction('<function_name>')`
- `args` - `TypedValue` array, you will get it by using one of many helpers like `U32Value`, `AddressValue`, `BytesValue`, etc. You'll learn more about it later
- `value` - the value to transfer. It is optional or can be set to 0
- `caller` - also `Iaddress` interface, the same as an address, also optional

**Initialization callbacks**  
Callbacks that will be triggered for that function

```typescript
onQueryStart?: (queryArgs: QueryArguments) => void;
onQueryFinalized?: (queryResponse: ContractQueryResponse) => void;
onQueryFailure?: (queryArgs: QueryArguments, error: string) => void;
```

**Usage example**: 

```html
<html>
<body>
  <button id="button-tx">Send transaction</button>

  <script type="module">
    import {
      ElvenJS,
      Address,
      AddressValue,
      ContractFunction,
    } from 'https://unpkg.com/elven.js@0.20.0/build/elven.js';

    // Initialization first (see above) ...
    
    // Some other stuff here ...

    // You can also use currently logged in user addres: ElvenJS.storage.get('address')
    const randomUserAddress = "erd1druav0mlt7wzutla33kw80ueaalmec7mz2hus5svdmzlfj286qpstg674t";
    
    document
      .getElementById('button-query')
      .addEventListener('click', async () => {
        try {
          await ElvenJS.queryContract({
            address: new Address(nftMinterSmartContract),
            func: new ContractFunction('getMintedPerAddressTotal'),
            args: [new AddressValue(new Address(randomUserAddress))]
          });
          // You can handle results parsing here or in the initialization callback as in the example
        } catch (e) {
          throw new Error(e?.message);
        }
      });
  </script>
</body>
</html>
```

###  Storage handling

```typescript
ElvenJS.storage.get(); // all keys
ElvenJS.storage.get('address'); // single key
ElvenJS.storage.set('address', 'erd1...'); // set the value for a key
ElvenJS.storage.clear();
```

Storage is a thin wrapper over the localStorage. All data required for synchronization on page refresh is kept there.

Remember that it shouldn't be used as a custom localStorage operations replacement. It has preconfigured localStorage key and should be used only for ElvenJS related operations. Mostly reading the actual state of the application. Below you will find what data you can get:

- `signature` - auth signature when you provide the auth token
- `loginToken` - the token you pass in the login function to get the signature back (optional)
- `address` - actually logged in erd address
- `activeGuardian` - active guardian assigned for the address
- `loginMethod` - actually chosen login method, for example, `browser-extension`
- `expires` - when your current session will expire
`balance` - your actual Egld balance
- `nonce` - your actual nonce - this one is quite important because you will need it when preparing transactions objects

Example of the localStorage data:

```json
{
  "signature":"42946a91f332eb3e413bc7ac18b8246bca1cb230ef2813ed714cd0417c95a8eb8104ce4e7d7c9ce1fb03853e10e7817416f8a8e789111df89909cd973f41ee0b",
  "address":"erd1druav0mlt7wzutla33kw80ueaalmec7mz2hus5svdmzlfj286qpstg674t",
  "activeGuardian": "erd1cscs8styv2ahllym0twsanezd4v65mcf5fungfvazgsfmuhwlehstadanq",
  "loginMethod":"browser-extension",
  "expires":1663881876868,
  "nonce":166,
  "balance":"6089909418940000000"
}
```

The storage key is `elvenjs_state`.

### Destroy and cleanup

**Function**:
```typescript
ElvenJS.destroy()
```

Mostly helpful in single-page applications where you would like to do some cleanup when you don't need the ElvenJS instance anymore.

**No arguments**.

### Exported classes and types from sdk-js

Below is the list of exported helpers, classes, and types from sdk-js. You can read more about them in the [sdk-js cookbook](https://docs.multiversx.com/sdk-and-tools/sdk-js/sdk-js-cookbook), also please check the [recipes](/docs/recipes.html) section, where you will find some of the use cases related to these:

- `TokenTransfer`
- `Token`
- `TokenComputer`
- `TokenOperationsFactory`
- `TokenOperationsFactoryConfig`
- `TokenOperationsOutcomeParser`
- `TransferTransactionsFactory`
- `TransactionsFactoryConfig`
- `SmartContractTransactionsFactory`
- `TokenManagementTransactionsFactory`
- `SmartContractTransactionsOutcomeParser`
- `TokenManagementTransactionsOutcomeParser`
- `TransactionEventsParser`
- `Address`
- `Account`
- `Transaction`
- `TransactionWatcher`
- `TransactionComputer`
- `Message`
- `MessageComputer`
- `SignableMessage`
- `QueryArguments`
- `ContractQueryResponse`
- `BytesType`
- `BytesValue`
- `U16Type`
- `U16Value`
- `U32Type`
- `U32Value`
- `U64Type`
- `U64Value`
- `U8Type`
- `U8Value`
- `BigUIntType`
- `BigUIntValue`
- `BooleanType`
- `BooleanValue`
- `AddressType`
- `AddressValue`

You can import them directly from elven.js without the ElvenJS namespace. Like:

```typescript
import {
  Address,
  ContractCallPayloadBuilder,
  ContractFunction
} from 'https://unpkg.com/elven.js@0.20.0/build/elven.js';
```

There will probably be more of them, but the ElvenJS library should be as small as possible. Maybe some of them will land in separate libraries like the planned query results parser library.

### Summary

You learned about all functions from ElvenJS. The library will undoubtedly get some more functionality in the future, but for now, this is enough to let you build dApps and widgets on your existing websites.

Please check the [JS SDK](https://docs.multiversx.com/sdk-and-tools/sdk-js/) tools for more info on some of the types and classes described here, and for sure, reading the [recipes](/docs/recipes.html) section will be beneficial.
