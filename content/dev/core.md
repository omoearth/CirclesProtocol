---
description: ""
sidebar: "docs"
prev: "/dev/overview"
next: "/dev/contracts/"
---

# @circles/core

Common methods (sign up, transfer Circles, trust users, revoke trust) for clients & wallets to interact with the Circles ecosystem (Smart Contracts, Relay Service, Trust Graph API, etc.).

## Installation

```
npm i @circles/core web3 isomorphic-fetch
```

## Development

`circles-core` is a JavaScript module written in JavaScript, tested with [Jest](https://jestjs.io/), transpiled with [Babel](https://babeljs.io/) and bundled with [Rollup](https://rollupjs.org).

```

// Copy config file and edit it
cp .env.example .env

// Run test suite
npm run test
npm run test:watch

// Check code formatting
npm run lint

// Build it!
npm run build
```

## Usage

### Connect Dependencies

Add the circles api to your project and connect with dependencies.

```js
import CirclesCore from "@circles/core";
import Web3 from "web3";

const web3 = new Web3();

// Initialize core
const core = new CirclesCore(web3, {
  hubAddress: "0x..",
  proxyFactoryAddress: "0x..",
  safeMasterAddress: "0x..",
  apiServiceEndpoint: "https://..",
  graphNodeEndpoint: "https://..",
  relayServiceEndpoint: "https://..",
  subgraphName: "...",
});
```

### User Accounts

```js
// Create account
const account = web3.eth.accounts.create();

// Define nice username for us
const username = "margareth";
const email = "mgh@mail.org";

// Generate a nonce to predict Safe address
const nonce = new Date().getTime();

// Prepare Safe deployment and receive a predicted safeAddress
const safeAddress = await core.safe.prepareDeploy(account, { nonce });

// Register username and connect it to Safe address
await core.user.register(account, {
  nonce,
  email,
  safeAddress,
  username,
});

// Resolve public addresses to user profiles
const users = await core.user.resolve(account, {
  addresses: network.map((connection) => connection.safeAddress),
});

// Search for a user via username
const users = await core.user.search(account, {
  query: "pand",
});
```

### Web-Of-Trust

```js
// Get our current trust network
const network = await core.trust.getNetwork(account, { safeAddress });

// Example: Display our trust network
network.forEach((connection) => {
  const user = users.find(
    (item) => item.safeAddress === connection.safeAddress
  );

  if (connection.isOutgoing) {
    console.log(`${user.username} accepts your Circles.`);
  }

  if (connection.isIncoming) {
    console.log(`You accept Circles of ${user.username}.`);
  }
});

// Check if we have enough trust connections
const { isTrusted } = await core.trust.isTrusted({
  safeAddress,
});

if (!isTrusted) {
  console.log("Not enough trust connections yet ..");
} else {
  // Deploy Safe
  await core.safe.deploy(account, { safeAddress });

  // Deploy Circles Token
  await core.token.deploy(account, { safeAddress });
}

// Change trust state with users
await core.trust.removeConnection(account, {
  user: users[0].safeAddress,
  canSendTo: safeAddress,
});

// .. give user the permission to send their Token to you
await core.trust.addConnection(account, {
  user: users[0].safeAddress,
  canSendTo: safeAddress,
  limitPercentage: 20,
});
```

### Activities

```js
// Get list of my activities
const { activities } = await core.activity.getLatest(account, {
  safeAddress,
});

// Example: Display activities
const { ActivityTypes } = core.activity;

activities.forEach((activity) => {
  const { timestamp, type, data } = activity;

  if (type === ActivityTypes.HUB_TRANSFER) {
    console.log(
      `${timestamp} - ${
        data.from
      } transferred ${data.value.toString()} Circles to ${data.to}`
    );
  } else if (type === ActivityTypes.ADD_CONNECTION) {
    console.log(
      `${timestamp} - ${data.limitPercentage} ${data.canSendTo} allowed ${
        data.send
      } to transfer Circles`
    );
  } else if (type === ActivityTypes.REMOVE_CONNECTION) {
    console.log(`${timestamp} - ${data.canSendTo} untrusted ${data.user}`);
  } else if (type === ActivityTypes.ADD_OWNER) {
    console.log(
      `${timestamp} - added ${data.ownerAddress} to ${data.safeAddress}`
    );
  } else if (type === ActivityTypes.REMOVE_OWNER) {
    console.log(
      `${timestamp} - removed ${data.ownerAddress} from ${data.safeAddress}`
    );
  }
});
```

### Gnosis Safe Balances

```js
// Get current balance of all owned Circles Tokens
const tokenAddress = core.token.getAddress(account, {
  safeAddress,
});

const balance = await core.token.getBalance(account, {
  safeAddress,
});
```

### Request UBI

```js
// Request my UBI
const payout = await core.token.checkUBIPayout(account, {
  safeAddress,
});

if (!payout.isZero()) {
  await core.token.requestUBIPayout(account, {
    safeAddress,
  });
}
```

### Transfer Circles

```js
// Transfer Circles to users (directly or transitively)
await core.token.transfer(account, {
  from: safeAddress,
  to: users[0].safeAddress,
  value: 350,
});
```

### Manage Private Keys

```js
// Get current Safe owners
await core.safe.getOwners(account, {
  safeAddress,
});

// .. or get the Safe connected to an owner
const [safeAddress] = await core.safe.getAddresses(account, {
  ownerAddress: "0x123...",
});

// Manage owners of my Safe
await core.safe.removeOwner(account, {
  safeAddress,
  ownerAddress: "0x123...",
});

await core.safe.addOwner(account, {
  safeAddress,
  ownerAddress: "0x123...",
});
```
