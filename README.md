---
description: A quick tutorial on a Binance Chain powered WebApp.
---

# Getting Started

## Overview

In this tutorial we will go over installing the SDK, adding a wallet, reading the wallet balance and then sending a transaction.

Code snippets will be posted along the way, and we will use the [beptools.org](https://gitlab.com/canyacoin/binancechain/beptools) codebase as a working example that you can compile.

## BEPTools Code Example

To get started, clone the BEPTools codebase in a new local directory on your machine:

```text
$ git clone https://gitlab.com/canyacoin/binancechain/beptools.git
```

{% hint style="info" %}
React is a great way to launch slick, responsive web apps. We will take a look at how BEPTools is built and from there you can build your own. You can learn more about it here:

[https://reactjs.org/docs/create-a-new-react-app.html](https://reactjs.org/docs/create-a-new-react-app.html)

We will also use YARN for package management:

[https://yarnpkg.com/lang/en/](https://yarnpkg.com/lang/en/)
{% endhint %}

Once you have cloned, you can enter the directory and start the web app using yarn:

```text
cd beptools
yarn install
yarn start
```

The webapp should launch to your `localhost`

{% hint style="info" %}
The web-app also uses Ant-design, a popular React UI framework.

[https://ant.design/](https://ant.design/)
{% endhint %}

## Quick Links

You can skip ahead to the following parts.

Add the Javascript SDK to your app and create a single `binance.js` file to manage all of our binance chain related functions:

{% page-ref page="connecting.md" %}

Add the Keystore wallet option, reading the address and save it in state.:

{% page-ref page="connect-to-wallet.md" %}

Read the wallet balances to create a web-viewable wallet:

{% page-ref page="reading-the-wallet.md" %}

Create and send a Binance Chain transaction:

{% page-ref page="sending-a-transaction.md" %}

