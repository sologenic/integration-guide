# SOLO Integration Guide

# Technical Support for listing

Technical support and integration assistance is provided free of charge by the core team when listing is required. Please contact integration@sologenic.com for more information on integration assistance.

##### Easy and Fast Guide to List SOLO

The integration of SOLO coins to exchanges are simply done via the many methods provided below. It is important to note that SOLO is an asset on the XRP Ledger blockchain and therefore it uses XRPL native commands and API.
SOLO is an IOU that is issued by the Sologeinc Gateway with the following information:

| Property                                                                                                                                                                                                                                                                                                                                                                                                                                                            | Value                                                              |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| `Currency Code (XRP Ledger)` Please note that currency codes on the XRP Ledger are either 3 letter standard currency formats such as the USD, or HEX representation of longer text. In SOLOs case “534F4C4” represents the text SOLO if converted from HEX to ASCII.                                                                                                                                                                                                | `534F4C4F00000000000000000000000000000000`                         |
| `Issuer` Issuer is the Gateway that issued SOLO IOU on the XRPL. It is important not to accept any other issuer other than Sologenics.                                                                                                                                                                                                                                                                                                                              | `rsoLo2S1kiGeCcn6hCUXVrCpGMWLrRrLZz`                               |
| `Symbol`                                                                                                                                                                                                                                                                                                                                                                                                                                                            | `Ƨ`                                                                |
| `Decimals`                                                                                                                                                                                                                                                                                                                                                                                                                                                          | `15`                                                               |
| `TX Fee`                                                                                                                                                                                                                                                                                                                                                                                                                                                            | `~0.00001 XRP` (10 drops) (https://xrpl.org/transaction-cost.html) |
| `Burn Amount` It is important to know that SOLO coins use a deflationary mechanism to bring the total supply down. For every transaction sent and received from XRP wallet addresses, 0.01% of that transaction is destroyed. For example if user A wants to send user B a total of 10 SOLO, your system should calculate 0.01% of 10 and deduct it from the amount to be sent. This is due to XRPL adding the burn fee on top of the amount that needs to be sent. | `0.01%`                                                            |

# Integration guide:

There are two main ways to integrate SOLO into your exchange.

- 1. sologenic-api: Use the sologenic-api JSON-RPC to and use endpoints provided (Easy and fast)
- 2. XRP Ledger: Use XRP Ledger directly (Complex)

For both methods, you have to make sure you either use a public ripple node or run your own rippled if you prefer (there are some benefits to running your own node such as speed, availability, and security).
Your hot wallet and cold wallets must have a trustline setup to the Gateway.

| Property    | Value                                      |
| ----------- | ------------------------------------------ |
| Currency    | `534F4C4F00000000000000000000000000000000` |
| Issuer      | `rsoLo2S1kiGeCcn6hCUXVrCpGMWLrRrLZz`       |
| Limit Value | `400,000,000`                              |
| Flags       | `131072`                                   |

A sample trustset transaction is as follows:

```json
{
  "TransactionType": "TrustSet",
  "Account": xxx,
  "LimitAmount": {
    "currency": "534F4C4F00000000000000000000000000000000",
    "issuer": "rsoLo2S1kiGeCcn6hCUXVrCpGMWLrRrLZz",
    "value": "400000000"
  },
  "Flags": 131072
}
```

Please make sure to also set requiredDest flags if you are managing 1 XRP wallet as your main cold or hot wallet. (See XRPL notes on Destination Tags)

# 1- sologenic-api

The best way to integrate SOLO coins into an exchange is to use the sologenic-api. This JSON-RPC api provides endpoints to users that are standardized to work with the underlying XRP Ledger.

## Configuration

Use environment variables for configuration.

`JSON_RPC_ENDPOINT` – rippled server URL (for testnet use https://s.altnet.rippletest.net:51234).
`APP_PORT` – server port (8080 by default).

You can pass `TESTNET=true` to automatically configure server to use testnet.

## Run the server

There are 3 ways you can run the server.

Using Docker:

```shell script
docker run -it -p 8080:8080 -e TESTNET=true sologenic/sologenic-api
```

docker-compose (by default runs in testnet):

```shell script
docker-compose up -d
```

Using locally installed Node.js:

```shell script
git clone git@github.com:sologenic/sologenic-api.git
cd sologenic-api
npm i && TESTNET=true node index.js
```

## API reference

### `get_balance`

Returns XRP and SOLO balance of an address.

```shell script
curl -H "Content-Type: application/json" -X POST \
     -d '{ "method": "get_balance", "params": { "address": "rLPA9rWx4WF3JJWN2QnrZE1fkahptc9Jn8" } }' \
     http://localhost:8080
```

```json
{
  "result": {
    "xrp": "999.999768",
    "solo": "4905.7437"
  }
}
```

### `generate_address`

Returns new address and secret generated offline.

```shell script
curl -H "Content-Type: application/json" -X POST \
     -d '{ "method": "generate_address", "params": {} }' \
     http://localhost:8080
```

```json
{
  "result": {
    "address": "r9zkrS5HKSYgpNVE1Gs9Gv5736DEUvik5v",
    "secret": "snra7wtNULsjdSwnGRS2uMXGGKi1q"
  }
}
```

### `get_transaction`

Returns transaction details.

Returns `null` if transaction currency isn't SOLO.

```shell script
curl -H "Content-Type: application/json" -X POST \
     -d '{ "method": "get_transaction", "params": { "id": "4A34D30DB25D6BF02F80CC714AD12F236DE4D2AA9B222FC6BF44780EE4364200" } }' \
     http://localhost:8080
```

```json
{
  "result": {
    "id": "4A34D30DB25D6BF02F80CC714AD12F236DE4D2AA9B222FC6BF44780EE4364200",
    "confirmations": 37,
    "time": 1580390510,
    "ledger": 4194290,
    "amount": "300.89",
    "sender": "rLPA9rWx4WF3JJWN2QnrZE1fkahptc9Jn8",
    "recipient": "rnuBmbM6tVv6R5FXYGr7cCgPoXKQBzMCBK?dt=7399"
  }
}
```

### `get_transactions`

Returns transactions belonging to an address (both incoming and outgoing).

You may specify `minimum_ledger` and `maximum_ledger` (both inclusive). If specified only transactions from these ledgers will be returned.

You can use this feature to paginate transactions.

By default if `minimum_ledger` and `maximum_ledger` are not specified API method returns transactions for past 1000 ledgers.

```shell script
curl -H "Content-Type: application/json" -X POST \
     -d '{ "method": "get_transactions", "params": { "address": "rnuBmbM6tVv6R5FXYGr7cCgPoXKQBzMCBK" } }' \
     http://localhost:8080
```

```json
{
  "result": {
    "transactions": [
      {
        "id": "431C405A107BFDA7251386055A9A31C1C2370F2FF2FF15E0F710F09BDDF29E5A",
        "confirmations": 17,
        "time": 1580390952,
        "ledger": 4194437,
        "amount": "30.33",
        "sender": "rLPA9rWx4WF3JJWN2QnrZE1fkahptc9Jn8",
        "recipient": "rnuBmbM6tVv6R5FXYGr7cCgPoXKQBzMCBK?dt=766"
      },
      {
        "id": "4A34D30DB25D6BF02F80CC714AD12F236DE4D2AA9B222FC6BF44780EE4364200",
        "confirmations": 164,
        "time": 1580390510,
        "ledger": 4194290,
        "amount": "300.89",
        "sender": "rLPA9rWx4WF3JJWN2QnrZE1fkahptc9Jn8",
        "recipient": "rnuBmbM6tVv6R5FXYGr7cCgPoXKQBzMCBK?dt=7399"
      }
    ],
    "minimum_ledger": 4193455,
    "maximum_ledger": 4194454
  }
}
```

### `send_transaction`

Creates, signs and broadcasts SOLO transaction to network.

```shell script
curl -H "Content-Type: application/json" -X POST \
     -d '{ "method": "send_transaction", "params": { "sender_address": "rLPA9rWx4WF3JJWN2QnrZE1fkahptc9Jn8",
                                                     "sender_secret": "SECRET",
                                                     "recipient_address": "rnuBmbM6tVv6R5FXYGr7cCgPoXKQBzMCBK?dt=7399",
                                                     "amount": "300.89" } }' \
     http://localhost:8080
```

```json
{
  "result": "4A34D30DB25D6BF02F80CC714AD12F236DE4D2AA9B222FC6BF44780EE4364200"
}
```

### `get_status`

Returns various status information (currently returns only last synced ledger).

```shell script
curl -H "Content-Type: application/json" -X POST \
     -d '{ "method": "get_status", "params": {} }' \
     http://localhost:8080
```

```json
{
  "result": {
    "latest_ledger": 4194744
  }
}
```

# 2- XRP Ledger

If you plan on building your own mechanisms for SOLO transactions, we recommend following the guides provided by the XRPL.org website.
Please note that the Amount object in SOLO is no longer a string with the number of drops (as in XRP). It is an object as such:

```json
Amount: {
    currency: "534F4C4F00000000000000000000000000000000",
    issuer: "rLZP3dxycQVsi7R5UuvJVRLYfDLG4KUfWQ",
    value: "AMOUNT TO SEND"
}
```

In order to create a payment transaction, you must first make sure the recipient has a trustline setup with the SOLO Gateway. To do so, they can use the open-source SOLO Wallet app or activate they trustline manually.
To send SOLO to a recipient, you must make a “Payment” transaction with the XRP ledger and specify the SendMax in addition to the Amount field. SendMax is also an object similar to the Amount object, but the value in SendMax must be the original amount to be transferred and the value in the Amount field, must be original x 0.9999. This account for the Burn Mechanism.
Example:
Send 100 SOLO from A to B

| Property | Value                |
| -------- | -------------------- |
| Amouont  | 100 x 0.9999 = 99.99 |
| SendMax  | 100                  |

A sample transaction would be like:

```json
{
  "TransactionType": "Payment",
  "Account": "SENDER ACCOUNT",
  "Sequence": 1,
  "Destination": "DESTINATION ACCOUNT",
  "LastLedgerSequence": 53836257,
  "Fee": "104",
  "Amount": {
    "currency": "534F4C4F00000000000000000000000000000000",
    "issuer": "rsoLo2S1kiGeCcn6hCUXVrCpGMWLrRrLZz",
    "value": "10"
  },
  "SendMax": {
    "currency": "534F4C4F00000000000000000000000000000000",
    "issuer": "rsoLo2S1kiGeCcn6hCUXVrCpGMWLrRrLZz",
    "value": "400000000"
  },

  "Memos": [
    {
      "Memo": {
        "MemoType": "remark",
        "MemoData": "Test"
      }
    }
  ]
}
```

# SDK(s)

## sologenic-xrpl-stream-js

Sologenic developer community has implemented a Javascript SDK that would manage transactions submitted to the XRP Ledger. This SDK uses persistent storage such as Redis and will queue transactions and makes sure they are validated or failed using a Transaction Management Queue (TXMQ).
Full documentation can be found: (https://github.com/sologenic/sologenic-xrpl-stream-js)

<!--ts-->

- [<a href="https://github.com/sologenic/sologenic-xrpl-stream-js">ƨ Sologenic XRPL Stream</a>](#ƨ-sologenic-xrpl-stream)
  - [Purpose](#purpose)
  - [How to Participate](#how-to-participate)
  - [Install](#install)
  - [Node Package Manager Scripts](#node-package-manager-scripts)
    - [Creating a packagable distribution](#creating-a-packagable-distribution)
    - [Testing](#testing)
      - [One-time execution of the unit tests](#one-time-execution-of-the-unit-tests)
      - [Persistent execution of unit tests (watching for changes)](#persistent-execution-of-unit-tests-watching-for-changes)
    - [Generating documentation](#generating-documentation)
      - [Generating HTML documentation only](#generating-html-documentation-only)
      - [Generating markdown documentation only](#generating-markdown-documentation-only)
  - [Typescript Example](#typescript-example)
    - [Intializing the Sologenic XRPL stream with a hash-based queue](#intializing-the-sologenic-xrpl-stream-with-a-hash-based-queue)
    - [Intializing the Sologenic XRPL stream with a redis-based queue](#intializing-the-sologenic-xrpl-stream-with-a-redis-based-queue)
    - [Sending a Payment with XRPL account and secret](#sending-a-payment-with-xrpl-account-and-secret)
    - [Sending a Payment with XRPL account and keypair](#sending-a-payment-with-xrpl-account-and-keypair)
  - [Event Emitter and Listeners](#event-emitter-and-listeners)
  - [Transactions](#transactions)

<!-- Added by: pmcconna, at: Fri Feb 14 13:37:46 PST 2020 -->

<!--te-->

![GitHub code size in bytes](https://img.shields.io/github/languages/code-size/sologenic/sologenic-xrpl-stream-js)
![GitHub contributors](https://img.shields.io/github/contributors/sologenic/sologenic-xrpl-stream-js)
![GitHub commit activity](https://img.shields.io/github/commit-activity/w/sologenic/sologenic-xrpl-stream-js)

# [&#x1a8; Sologenic XRPL Stream](https://github.com/sologenic/sologenic-xrpl-stream-js)

## Purpose

The sologenic-xrpl-stream-js enables clients to communicate and submit transactions to the XRP Ledger seamlessly and reliably.

This library provides reliable transaction handling following the guide provided by XRPL.org [reliable transaction submissions](https://xrpl.org/reliable-transaction-submission.html).

![sologenic-xrpl-stream-js.png](assets/img/sologenic-xrpl-stream-js.png)

Once a transaction is submitted, it is queued either using a Hash-based in-memory queue (non-persistent, ideal for front-end) or a Redis (persistent, ideal for backend). Transactions are queued and dispatched in sequence. Account sequence numbers, ledgerVersions and fees are also handled for each transaction that is being dispatched.

Events are reported back to the client using a global EventEmitter and transaction-specific EventEmitter. This allows clients to track the statuses of their transactions and take actions based on the results.

**Production uses**

- SOLO ReactNative Wallet
- SOLO React Electron Wallet

In general, there are two types of users who would benefit from using this library:

1. Exchanges or users with large volumes of transactions who want to ensure they receive reliable delivery and can receive event notifications throughout the transactions validation process.

2. Users who do not want to deal with transaction dispatching, validations and errors on the XRPL.

## How to Participate

We have a community for questions and support at [sologenic-dev.slack.com](https://sologenic-dev.slack.com). To receive an invite for the community please fill out the [form](https://docs.google.com/forms/d/e/1FAIpQLSdcpIL-u2FsqBZj0ikG7UyJe3l9If7sVr7MdTpVnINQJJbsQg/viewform) and we'll send you your invite link.

## Install

```bash
$ npm install sologenic-xrpl-stream-js
```

## Node Package Manager Scripts

Each `npm` script defined in `package.json` can be run by simply running the command `npm run-script <script name>`

For example, notably the following scripts are frequently used when creating a packagable distribution, testing,
and generating documentation.

### Creating a packagable distribution

`npm run-s build`

### Testing

#### One-time execution of the unit tests

The following command will iterate over all test cases (files with `.spec.ts`) as their suffix.

`npm run-s test`

#### Persistent execution of unit tests (watching for changes)

The following command will iterate over all test cases (files with `.spec.ts`) as their suffix. But it will also watch for changes to the `.spec.ts` and `.ts` scripts and run tests on change.

`npm run-s watch`

### Generating documentation

The following command will generate both HTML and markdown documentation in the `docs/` folder.

`npm run-s doc`

#### Generating HTML documentation only

The following command will generate HTML only documentation.

`npm run-s doc:html`

#### Generating markdown documentation only

The following command will generate markdown only documentation.

`npm run-s doc:markdown`

## Typescript Example

When using the `sologenic-xrpl-stream-js` library on the server side, we recommend using `redis` as the queue storage mechanism, whereas when using the library on the client side, we recommend using `hash` as on the client side you will most likely not have a `redis` server accessible to you.

### Intializing the Sologenic XRPL stream with a hash-based queue

```typescript
'use strict';
const ƨ = require('sologenic-xrpl-stream-js');

(async () => {
  try {
    const sologenic = await new ƨ.SologenicTxHandler(
      // RippleAPI Options
      {
        server: 'wss://testnet.xrpl-labs.com', // Kudos to Wietse Wind
      },
      // Sologenic Options, hash or redis (see SologenicOptions in documentation)
      {
        // Clear the cache before accessing the queue, since this is a hash-based
        // queue it will be initialized empty, so this will have no effect.
        clearCache: true,
        queueType: "hash",
        hash: {}
      }
    ).connect();
);
```

### Intializing the Sologenic XRPL stream with a redis-based queue

```typescript
'use strict';
const ƨ = require('sologenic-xrpl-stream-js');

(async () => {
  try {
    const sologenic = await new ƨ.SologenicTxHandler(
      // RippleAPI Options
      {
        server: 'wss://testnet.xrpl-labs.com', // Kudos to Wietse Wind
      },
      // Sologenic Options, hash or redis
      {
        // Clear the cache before accessing the queue, since this is a redis-based
        // queue it will be emptied before after connecting.  Please make sure there
        // is no data in the database you're accessing you want to preserve.
        clearCache: true,
        queueType: 'redis',
        redis: {
          // The IP address or hostname of the redis queue
          // host: '127.0.0.1',

          // The port of the redis queue
          // port: 6379,

          // The password to access the redis queue
          // password: 'password',

          // The database number of the redis queue (if multiple database support is active)
          // database: 1
        }
      }
    ).connect();
);
```

Note: When using a redis queue, you must have an active redis server which the transactional handler can connect.

### Sending a Payment with XRPL account and secret

```js
'use strict';
const ƨ = require('sologenic-xrpl-stream-js');

(async () => {
  try {
    const sologenic = await new ƨ.SologenicTxHandler(
      // RippleAPI Options
      {
        server: 'wss://testnet.xrpl-labs.com', // Kudos to Wietse Wind
      },
      // Sologenic Options, hash or redis
      {
        clearCache: true,
        queueType: "hash",
        hash: {}
      }
    ).connect();

    // Events have their own types now.
    sologenic.on('queued', (event: SologenicTypes.QueuedEvent) => {
      console.log('GLOBAL QUEUED: ', event);
    });
    sologenic.on('dispatched', (event: SologenicTypes.DispatchedEvent) => {
      console.log('GLOBAL DISPATCHED:', event);
    });
    sologenic.on('requeued', (event: SologenicTypes.RequeuedEvent) => {
      console.log('GLOBAL REQUEUED:', event);
    });
    sologenic.on('warning', (event: SologenicTypes.WarningEvent) => {
      console.log('GLOBAL WARNING:', event);
    });
    sologenic.on('validated', (event: SologenicTypes.ValidatedEvent) => {
      console.log('GLOBAL VALIDATED:', event);
    });
    sologenic.on('failed', (event: SologenicTypes.FailedEvent) => {
      console.log('GLOBAL FAILED:', event);
    });

    await sologenic.setAccount({
      address: 'rNbe8nh1K6nDC5XNsdAzHMtgYDXHZB486G',
      secret: 'ssH5SSKYvHBynnrYoCnmvsbxrNGEv'
    });

    const tx = sologenic.submit({
      TransactionType: 'Payment',
      Account: 'rNbe8nh1K6nDC5XNsdAzHMtgYDXHZB486G',
      Destination: 'rUwty6Pf4gzUmCLVuKwrRWPYaUiUiku8Rg',
      Amount: {
        currency: '534F4C4F00000000000000000000000000000000',
        issuer: 'rNbe8nh1K6nDC5XNsdAzHMtgYDXHZB486G',
        value: '100000000'
      }
    });

    // Events have their own types now.
    tx.events.on('queued', (event: SologenicTypes.QueuedEvent) => {
      console.log('TX QUEUED: ', event);
    }).on('dispatched', (event: SologenicTypes.DispatchedEvent) => {
      console.log('TX DISPATCHED:', event);
    }).on('requeued', (event: SologenicTypes.RequeuedEvent) => {
      console.log('TX REQUEUED:', event);
    }).on('warning', (event: SologenicTypes.WarningEvent) => {
      console.log('TX WARNING:', event);
    }).on('validated', (event: SologenicTypes.ValidatedEvent) => {
      console.log('TX VALIDATED:', event);
    });.on('failed', (event: SologenicTypes.FailedEvent) => {
      console.log('TX FAILED:', event);
    });

    console.log(await tx.promise);
  } catch (error) {
    console.log('Error:', error);
  }
})();
```

### Sending a Payment with XRPL account and keypair

```js
'use strict';
const ƨ = require('sologenic-xrpl-stream-js');

(async () => {
  try {
    const sologenic = await new ƨ.SologenicTxHandler(
      // RippleAPI Options
      {
        server: 'wss://testnet.xrpl-labs.com', // Kudos to Wietse Wind
      },
      // Sologenic Options, hash or redis
      {
        clearCache: true,
        queueType: "hash",
        hash: {}
      }
    ).connect();

    // Events have their own types now.
    sologenic.on('queued', (event: SologenicTypes.QueuedEvent) => {
      console.log('GLOBAL QUEUED: ', event);
    });
    sologenic.on('dispatched', (event: SologenicTypes.DispatchedEvent) => {
      console.log('GLOBAL DISPATCHED:', event);
    });
    sologenic.on('requeued', (event: SologenicTypes.RequeuedEvent) => {
      console.log('GLOBAL REQUEUED:', event);
    });
    sologenic.on('warning', (event: SologenicTypes.WarningEvent) => {
      console.log('GLOBAL WARNING:', event);
    });
    sologenic.on('validated', (event: SologenicTypes.ValidatedEvent) => {
      console.log('GLOBAL VALIDATED:', event);
    });
    sologenic.on('failed', (event: SologenicTypes.FailedEvent) => {
      console.log('GLOBAL FAILED:', event);
    });

    await sologenic.setAccount({
      address: 'rNbe8nh1K6nDC5XNsdAzHMtgYDXHZB486G',
      keypair: {
        publicKey: 'my public key with permission to sign transaction for address',
        privateKey: 'my private key with permission to sign transactions for address'
      }
    });

    const tx = sologenic.submit({
      TransactionType: 'Payment',
      Account: 'rNbe8nh1K6nDC5XNsdAzHMtgYDXHZB486G',
      Destination: 'rUwty6Pf4gzUmCLVuKwrRWPYaUiUiku8Rg',
      Amount: {
        currency: '534F4C4F00000000000000000000000000000000',
        issuer: 'rNbe8nh1K6nDC5XNsdAzHMtgYDXHZB486G',
        value: '100000000'
      }
    });

    // Events have their own types now.
    tx.events.on('queued', (event: SologenicTypes.QueuedEvent) => {
      console.log('TX QUEUED: ', event);
    }).on('dispatched', (event: SologenicTypes.DispatchedEvent) => {
      console.log('TX DISPATCHED:', event);
    }).on('requeued', (event: SologenicTypes.RequeuedEvent) => {
      console.log('TX REQUEUED:', event);
    }).on('warning', (event: SologenicTypes.WarningEvent) => {
      console.log('TX WARNING:', event);
    }).on('validated', (event: SologenicTypes.ValidatedEvent) => {
      console.log('TX VALIDATED:', event);
    });.on('failed', (event: SologenicTypes.FailedEvent) => {
      console.log('TX FAILED:', event);
    });

    console.log(await tx.promise);
  } catch (error) {
    console.log('Error:', error);
  }
})();
```

## Event Emitter and Listeners

There are 6 different events that you'll receive while a transaction is being processed by the library. Each event has its own type associated with it as you can see in the `src/types/sologenicoptions.d.ts`. See below for a list of events and the emitted objects.

- Event (validated): `SologenicTypes.ValidatedEvent`
- Event (warning): `SologenicTypes.WarningEvent`
- Event (requeued): `SologenicTypes.RequeuedEvent`
- Event (queued): `SologenicTypes.QueuedEvent`
- Event (failed): `SologenicTypes.FailedEvent`
- Event (dispatched): `SologenicTypes.DispatchedEvent`

As you can see below in the `transactions` section there is a snippet of code there. The reason we added this is because we wanted to outline that we have two separate event emitters that emit events based on the transactions. One of the event emitters being a global emitter that receives all events, and then a transaction based event emitter which receives events for only the transaction it has subscribed to.

For example, the `sologenic.on()` subscriptions (event listeners) to events are global, meaning you'll receive events for every transaction you submit. Whereas, the `tx.events.on()` subscriptions (event listeners) are specific to the transactions themselves, once the transaction has been `validated` or `failed` you will no longer receive transactions as the listeners are all unsubscribed automatically.

```typescript
// Events have their own types now.
sologenic.on('queued', (event: SologenicTypes.QueuedEvent) => {
  console.log('GLOBAL QUEUED: ', event);
});
sologenic.on('dispatched', (event: SologenicTypes.DispatchedEvent) => {
  console.log('GLOBAL DISPATCHED:', event);
});
sologenic.on('requeued', (event: SologenicTypes.RequeuedEvent) => {
  console.log('GLOBAL REQUEUED:', event);
});
sologenic.on('warning', (event: SologenicTypes.WarningEvent) => {
  console.log('GLOBAL WARNING:', event);
});
sologenic.on('validated', (event: SologenicTypes.ValidatedEvent) => {
  console.log('GLOBAL VALIDATED:', event);
});
sologenic.on('failed', (event: SologenicTypes.FailedEvent) => {
  console.log('GLOBAL FAILED:', event);
});

await sologenic.setAccount({
  address: 'rNbe8nh1K6nDC5XNsdAzHMtgYDXHZB486G',
  keypair: {
    publicKey: 'my public key with permission to sign transaction for address',
    privateKey: 'my private key with permission to sign transactions for address'
  }
});

const tx = sologenic.submit({
  TransactionType: 'Payment',
  Account: 'rNbe8nh1K6nDC5XNsdAzHMtgYDXHZB486G',
  Destination: 'rUwty6Pf4gzUmCLVuKwrRWPYaUiUiku8Rg',
  Amount: {
    currency: '534F4C4F00000000000000000000000000000000',
    issuer: 'rNbe8nh1K6nDC5XNsdAzHMtgYDXHZB486G',
    value: '100000000'
  }
});

// Events have their own types now.
tx.events.on('queued', (event: SologenicTypes.QueuedEvent) => {
  console.log('TX QUEUED: ', event);
}).on('dispatched', (event: SologenicTypes.DispatchedEvent) => {
  console.log('TX DISPATCHED:', event);
}).on('requeued', (event: SologenicTypes.RequeuedEvent) => {
  console.log('TX REQUEUED:', event);
}).on('warning', (event: SologenicTypes.WarningEvent) => {
  console.log('TX WARNING:', event);
}).on('validated', (event: SologenicTypes.ValidatedEvent) => {
  console.log('TX VALIDATED:', event);
});.on('failed', (event: SologenicTypes.FailedEvent) => {
  console.log('TX FAILED:', event);
});
```

## Transactions

Each transaction that is created by the `sologenic-xrpl-stream-js` library will have a unique `uuidv4` that is used for tracking purposes in the queue system. The `tx.id` is not the same as the XRPL ledger hash.

Example snippet from `src/lib/sologenictxhandler.ts`

```typescript
public submit(tx: SologenicTypes.TX): SologenicTypes.TransactionObject {
  try {
    // Generate a unique ID using the uuid library
    const id = uuid();

    // Add a new EventEmitter to txEvents array identifiable with the generated id.
    this.txEvents![id] = new EventEmitter();
    this._initiateTx(id, tx);
```

When you are receiving events while the transaction is undergoing processing in the XRPL, you'll receive your transaction ID which can be then used to verify your transaction has been completed. In addition, once the transaction has been validated in the XRPL, you will notice that the `tx.id` is appended to a memo-field within the transaction itself.

The `tx.id` field is non-async and does not change if the transaction is requeued or failed so that you have the option of going back to check the state.

# Technical Support for listing

Technical support and integration assistance is provided free of charge by the core team when listing is required. Please contact integration@sologenic.com for more information on integration assistance.
