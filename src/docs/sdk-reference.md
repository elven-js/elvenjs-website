---
layout: "docs"
title: "SDK Reference"
publicationDate: "2022-09-20"
tags:
  - intro
excerpt: "Elven.js - the JavaScript SDK for the Elrond blockchain. Compact and simplified wrapper for erdjs!"
ogTitle: "Elven.js - JavaScript Elrond SDK for browsers - SDK Reference!"
ogDescription: "Elven.js - the JavaScript SDK for the Elrond blockchain. Compact and simplified wrapper for erdjs!"
ogUrl: "https://www.elvenjs.com/docs/sdk-reference.html"
twitterTitle: "Elven.js - JavaScript Elrond SDK for browsers - SDK Reference!"
twitterDescription: "Elven.js - the JavaScript SDK for the Elrond blockchain. Compact and simplified wrapper for erdjs!"
twitterUrl: "https://www.elvenjs.com/docs/sdk-reference.html"
githubUrl: "https://github.com/juliancwirko/elvenjs-website/edit/main/src/docs/sdk-reference.md"
---

The Elven.js tool will be as simple as possible. It exports a couple of helper functions. It also exports several data structures (types) from [erdjs](https://docs.elrond.com/sdk-and-tools/erdjs/erdjs/) libraries. Here you will find a description of all the parts, and then you can check the [recipes](/docs/recipes.html) section for real-world examples.

Worth mentioning. Remember to check the source code, written in Typescript. You will find all the source files here: [elven.js/src](https://github.com/juliancwirko/elven.js/tree/main/src).

### Initialization

**Function**:
```typescript
await ElvenJS.init(initOptions: InitOptions)
```

**Typings**:
```typescript
interface InitOptions {
  apiUrl: string;
  chainType: string;
  apiTimeout: number;
  onLoginPending?: () => void;
  onLoggedIn?: () => void;
  onLogout?: () => void;
}
```

The primary initialization function. It is responsible for synchronizing with the Elrond network and attaching login/logout callbacks.

**Arguments**:

- `apiUrl`Elrond API URL - can be the public or private instance,
- `chainType`Chain type identification - can be devnet, testnet, or mainnet,
- `apiTimeout`: The API call timeout in milliseconds. Maximum 10000,
- `onLoginPending`: On login pending callback. It is used across all the auth providers,
- `onLoggedIn`: On logged in callback. It is used across all the auth providers,
- `onLogout`: On logout callback. It is used across all the auth providers

**Usage example**:

```html
<html>
<body>
  <script type="module">
    import {
      ElvenJS
    } from 'https://unpkg.com/elven.js@0.5.0/build/elven.js';

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

**Function**:
```typescript
await ElvenJS.login(loginMethod: LoginMethodsEnum, options?: LoginOptions)
```

**Typings**:
```typescript
enum LoginMethodsEnum {
  ledger = 'ledger', // not implemented yet
  maiarMobile = 'maiar-mobile',
  webWallet = 'web-wallet', // not implemented yet
  maiarBrowserExtension = 'maiar-browser-extension',
}

interface LoginOptions {
  qrCodeContainer?: string | HTMLElement;
  token?: string;
}
```

One interface for logging in with all possible auth providers. It is the core functionality in Elven.js

**Arguments**:

- `loginMethod`: one of four login methods (ledger, maiar-mobile, web-wallet, maiar-browser-extension) (for now, two of them are implemented)
- `options` as options, you can pass the `token`, which is a unique string that can be used for signature generation and user verification. You can also define `qrCodeContainer`, the DOM element id or DOM element in which the maiar-mobile QR code will be displayed

**Usage example**:

```html
<html>
<body>
  <button id="button-login-extension">Login with extension</button>
  <button id="button-login-mobile">Login with Maiar mobile</button>
  <div id="qr-code-container"></div>

  <script type="module">
    import {
      ElvenJS
    } from 'https://unpkg.com/elven.js@0.5.0/build/elven.js';

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
    } from 'https://unpkg.com/elven.js@0.5.0/build/elven.js';

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

### Signing and sending transactions

**Function**:
```typescript
await ElvenJS.signAndSendTransaction(transaction: Transaction)
```

**Typings**:

`Transaction` is the erdjs exported [Transaction](https://github.com/ElrondNetwork/elrond-sdk-erdjs/blob/main/src/transaction.ts) class.

The sign and send transaction handle one transaction at a time. This is basic functionality that is enough in most cases. The previously prepared transaction instance will be signed and sent with that function. You will use your chosen provider (mobile app, browser extension, etc.) for signing.

**Arguments**:

- `transaction`: the erdjs transaction class, the transaction instance is the same as the erdjs one.

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
      TokenPayment
    } from 'https://unpkg.com/elven.js@0.5.0/build/elven.js';

    // Initialization first (see above) ...
    
    // Some other stuff here ...

    const egldTransferAddress = 'erd17a4wydhhd6t3hhssvcp9g23ppn7lgkk4g2tww3eqzx4mlq95dukss0g50f';

    document
      .getElementById('button-tx')
      .addEventListener('click', async () => {
        const demoMessage = 'Transaction demo from Elven.js!';

        const tx = new Transaction({
          nonce: ElvenJS.storage.get('nonce'),
          receiver: new Address(egldTransferAddress),
          gasLimit: 50000 + 1500 * demoMessage.length,
          chainID: 'D',
          data: new TransactionPayload(demoMessage),
          value: TokenPayment.egldFromAmount(0.001),
          sender: new Address(ElvenJS.storage.get('address')),
        });

        try {
          const transaction = await ElvenJS.signAndSendTransaction(tx);
        } catch (e) {
          throw new Error(e?.message);
        }
      });
  </script>
</body>
</html>
```

You can also see the transaction instance here. There is a couple of classes exported from erdjs. You can think of them as helper tools for data preparation. More about them later.

### Query smart contracts

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

`QueryArguments` is the erdjs exported [QueryArguments](https://github.com/ElrondNetwork/elrond-sdk-erdjs/blob/main/src/smartcontracts/interface.ts) type. 
Querying smart contracts is possible with this function. You must pass the smart contract address, function name (smart contract endpoint), and arguments (if it takes any). The value and caller are optional.

**Arguments**:

- `address` - `IAddress` interface from erdjs, you will get it by `new Address(<string_addres>)`
- `func` - `IContractFunction` interface from erdjs, you will get it by `new ContractFunction('<function_name>')`
- `args` - `TypedValue` array, you will get it by using one of many helpers like `U32Value`, `AddressValue`, `BytesValue`, etc. You'll learn more about it later
- `value` - the value to transfer. It is optional or can be set to 0
- `caller` - also `Iaddress` interface, the same as an address, also optional

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
      TokenPayment
    } from 'https://unpkg.com/elven.js@0.5.0/build/elven.js';

    // Initialization first (see above) ...
    
    // Some other stuff here ...

    const minterAddress = "erd1druav0mlt7wzutla33kw80ueaalmec7mz2hus5svdmzlfj286qpstg674t";
    document
      .getElementById('button-query')
      .addEventListener('click', async () => {
        try {
          const results = await ElvenJS.queryContract({
            address: new Address(nftMinterSmartContract),
            func: new ContractFunction('getMintedPerAddressTotal'),
            args: [new AddressValue(new Address(minterAddress))]
          });

          // Manual decoding of a simple type (number here) with aa custom helper function, 
          // there will be additional tools for that
          const hexVal = base64ToDecimalHex(results?.returnData?.[0]);
          console.log(hexVal)
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
ElvenJS.storage.get()
ElvenJS.storage.set()
ElvenJS.storage.clear()
```

Storage is a thin wrapper over the localStorage. All data required for synchronization on page refresh is kept there.

Remember that it shouldn't be used as a custom localStorage operations replacement. It has preconfigured localStorage key and should be used only for ElvenJS related operations. Mostly reading the actual state of the application. Below you will find what data you can get:

- `signature` - auth signature when you provide the auth token
- `address` - actually logged in erd address
- `loginMethod` - actually chosen login method, for example, `maiar-browser-extension`
- `expires` - when your current session will expire
`balance` - your actual Egld balance
- `nonce` - your actual nonce - this one is quite important because you will need it when preparing transactions objects

Example of the localStorage data:

```json
{
  "signature":"42946a91f332eb3e413bc7ac18b8246bca1cb230ef2813ed714cd0417c95a8eb8104ce4e7d7c9ce1fb03853e10e7817416f8a8e789111df89909cd973f41ee0b",
  "address":"erd1druav0mlt7wzutla33kw80ueaalmec7mz2hus5svdmzlfj286qpstg674t",
  "loginMethod":"maiar-browser-extension",
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

### Exported classes and types from erdjs

Below is the list of exported helpers, classes, and types from erdjs. You can read more about them in the [erdjs cookbook](https://docs.elrond.com/sdk-and-tools/erdjs/erdjs-cookbook/), also please check the [recipes](/docs/recipes.html) section, where you will find some of the use cases related to these:

- `TokenPayment`
- `Address`
- `AddressValue`
- `Account`
- `Transaction`
- `TransactionPayload`
- `TransactionWatcher`
- `BytesValue`
- `BigUIntValue`
- `U32Value`
- `BooleanValue`
- `ContractCallPayloadBuilder`
- `ESDTTransferPayloadBuilder`
- `ESDTNFTTransferPayloadBuilder`
- `ContractFunction`
- `SmartContract`

You can import them directly from elven.js without the ElvenJS namespace. Like:

```typescript
import {
  Address,
  ContractCallPayloadBuilder,
  ContractFunction
} from 'https://unpkg.com/elven.js@0.5.0/build/elven.js';
```

There will probably be more of them, but the ElvenJS library should be as small as possible. Maybe some of them will land in separate libraries like the planned query results parser library.

### Summary

You learned about all functions from ElvenJS. The library will undoubtedly get some more functionality in the future, but for now, this is enough to let you build dApps and widgets on your existing websites.

Please check the [erdjs SDK](https://docs.elrond.com/sdk-and-tools/erdjs/erdjs/) tools for more info on some of the types and classes described here, and for sure, reading the [recipes](/docs/recipes.html) section will be beneficial.
