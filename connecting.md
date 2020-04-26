---
description: How to connect to the Javascript SDK
---

# Javascript SDK

## Setup

### Installing

BEPTools is powered by the Javascript SDK library. 

To add Binance Chain [Javascript SDK](https://github.com/binance-chain/javascript-sdk) to your WebApp, run the following:

```
$ npm i @binance-chain/javascript-sdk
```

{% hint style="warning" %}
Having trouble compiling?

Tips here: [https://github.com/binance-chain/javascript-sdk](https://github.com/binance-chain/javascript-sdk)
{% endhint %}

### Binance.js

To keep all the BinanceChain related functions together, we will create a `Binance.js` file located at `/src/clients/binance.js` in the BEPTools directory. 

We then need to import all the Binance functions we will use, alongside `axios` which is a popular http client. 

{% code title="/src/clients/binance.js" %}
```text
import bnbClient from '@binance-chain/javascript-sdk'
import TokenManagement, { crypto } from '@binance-chain/javascript-sdk'
import axios from 'axios'
```
{% endcode %}

We now want to initialise the client to testnet:

```text
this.baseURL = "https://testnet-dex.binance.org"
this.net = "testnet"

this.httpClient = axios.create({
      baseURL:  this.baseURL + "/api/v1",
      contentType: "application/json",
    })

this.bnbClient = new bnbClient(this.baseURL);
this.bnbClient.chooseNetwork(this.net)
this.bnbClient.initChain()
```

{% hint style="info" %}
Want to work on Mainnet instead? Use the following defaults:

```text
this.baseURL = "https://dex.binance.org"
this.net = "mainnet"
```
{% endhint %}

#### Binance Chain Queries

We can now create our Binance Chain queries to call from other parts of our app:

```text
// Get balances on a Binance Chain address
getBalances(address) {
    return this.bnbClient.getBalance(address)
  }

// Get the account number for an address
  getAccount(address) {
    return this.bnbClient.getAccount(address)
  }

// Get a list of markets
 getMarkets(limit = 1000, offset = 0) {
    return this.bnbClient.getMarkets(limit, offset)
  }
 
// Get the list of on-chain fees 
   getfees() {
    return this.httpClient.get("/fees")
  }
```

{% hint style="info" %}
There are many more functions in the Javascipt SDK that you can call to return information on Binance Chain.   
  
[https://github.com/binance-chain/javascript-sdk/wiki/API-Documentation](https://github.com/binance-chain/javascript-sdk/wiki/API-Documentation)
{% endhint %}

#### Binance Chain Transactions

Similarly, we can also use this to set our write functions:

```text
// Create a Send transaction
async transfer(fromAddress, toAddress, amount, asset, memo="") {
    const result = await this.bnbClient.transfer(fromAddress, toAddress, amount, asset, memo)
  }
}

// Create a Multi-send transaction
async multiSend(address, transactions, memo="") {
    const result = await this.bnbClient.multiSend(address, transactions, memo)
    return result
  }
}

// Token Management Functions
const manager = new TokenManagement(this.bnbClient).tokens
async freeze(address, asset, amount) {
    const result = await manager.freeze(address, asset, amount)
    return result
}

async unfreeze(address, asset, amount) {
    const result = await manager.unfreeze(address, asset, amount)
    return result
}

```

We can now make transactions from anywhere else in the app just by passing in the variable required. 

Let's now connect a wallet!

