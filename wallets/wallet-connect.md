---
description: Adding Wallet Connect
---

# Wallet Connect

## How to integrate Wallet Connect

### Setup

We need to make three additions to our app to connect Ledger. 

1. Update our `Binance.js` file
2. Add a Ledger connect option to the **Unlock Page**
3. Add the ledger methods to our **Transaction Page**

**Dependencies**

Install these with NPM or Yarn:

* `@walletconnect/qrcode-modal`
* `@trustwallet/walletconnect`

### Wallet Connect Page

We need to add some logic to our Wallet Connect component. We import React Contect hooks and the Context for managing state. We also import the `WalletConnect`, `WalletConnectQRCodeModal` and `crypto` dependencies.

```text
import React, { useContext } from 'react'
import { Context } from '../../../context'
import WalletConnect from "@trustwallet/walletconnect";
import WalletConnectQRCodeModal from "@walletconnect/qrcode-modal";
import { crypto } from '@binance-chain/javascript-sdk'
```

The full logic is here:

```text
const WalletConnectPane = props => {
  const context = useContext(Context)
  const walletConnect = async () => { const walletConnector = window.mywallet = new WalletConnect
  ({ bridge: "https://bridge.walletconnect.org"});
  
    walletConnector.killSession()
    
    // Check if connection is already established
    if (!walletConnector.connected) {
      console.log("Creating session")
      // create new session
      walletConnector.createSession().then(() => {
        // get uri for QR Code modal
        const uri = walletConnector.uri;
        // display QR Code modal
        WalletConnectQRCodeModal.open(uri, () => {
          console.log("QR Code Modal closed");
        })
      })
  }
  
  // Subscribe to connection events
  walletConnector.on("connect", (error, payload) => {
    if (error) {
      throw error;
    }
  
    // Close QR Code Modal
    WalletConnectQRCodeModal.close();
  
    // Get provided accounts and chainId
    // const { accounts, chainId } = payload.params[0];
  
    walletConnector.getAccounts().then(result => {
      // Returns the accounts
      const account = result.find((account) => account.network === 714);
      console.log("ACCOUNT:", account)
      console.log("WALLET CONNECT ACCOUNTS RESULTS " + account.address);
      console.log("ADDR:", crypto.decodeAddress(account.address))
      
      context.setContext({
        "wallet": {
          "walletconnect": walletConnector,
          "address": account.address,
          "account": account,
        }
      }, () => {
        props.history.push("/")
      })
    })
      .catch(error => {
        // Error returned when rejected
        console.error(error);
      })
  })
  
  walletConnector.on("session_update", (error, payload) => {
    if (error) {
      throw error;
    }
  })
  
  walletConnector.on("disconnect", (error, payload) => {
    if (error) {
      throw error;
    }
  
    // Delete walletConnector
    context.forgetWallet()
  })
}
```

### Transactions

When making transactions, we need to import the [base64js library](https://www.npmjs.com/package/base64-js) to encode addresses into Base64:

`import base64js from 'base64-js'`

We also create two parameters we refer to in the Wallet Connect client that help us indentify the chain and the network. Here we also have an `isTestnet` conditional:

```text
const CHAIN_ID = isTestnet ? "Binance-Chain-Nile" : "Binance-Chain-Tigris"
const NETWORK_ID = 714
```

Then the following is the code required to sign and send a transaction:

```text
if (context.wallet.walletconnect) {
      Binance.getAccount(context.wallet.address)
        .then((response) => {
          const account = response.result
          
          const tx = window.tx = {
            accountNumber: account.account_number.toString(),
            chainId: CHAIN_ID,
            sequence: account.sequence.toString(),
          };

          // https://github.com/TrustWallet/wallet-core/blob/master/src/proto/Binance.proto#L46
          tx.send_order = {
            inputs: transfers.map((transfer) => {
              return {
                "address": base64js.fromByteArray(crypto.decodeAddress(context.wallet.address)),
                "coins": {
                  "denom": transfer.ticker,
                  "amount": transfer.free * Math.pow(10, 8),
                }
              }
            }),
            outputs: transfers.map((transfer) => {
              return {
                "address": base64js.fromByteArray(crypto.decodeAddress(transfer.address)),
                "coins": {
                  "denom": transfer.ticker,
                  "amount": transfer.free * Math.pow(10, 8),
                }
              }
            })
          }

          context.wallet.walletconnect
            .trustSignTransaction(NETWORK_ID, tx)
            .then(result => {
              // Returns transaction signed in json or encoded format
              window.result = result
              console.log("Successfully signed msg:", result);
              binance.bnbClient.sendRawTransaction(result, true)
                .then((response) => {
                  console.log("Response", response)
                  getBalances()
                })
                .catch((error) => {
                  message.error(error.message)
                  console.error(error)
                })
            })
            .catch(error => {
              // Error returned when rejected
              console.error(error);
              message.error(error.message)
            });
          return
        })
        .catch((error) => {
          window.err = error
          message.error(error.message)
          console.error(error)
          return
        })
    }
```

### FREEZE Transactions

The following is the only difference when sending a freeze transaction:

```text
if (mode === MODE_FREEZE) {
    tx.freeze_order = {
      from: base64js.fromByteArray(crypto.decodeAddress(context.wallet.address)),
      symbol: selectedCoin,
      amount: (parseFloat(values.amount) * Math.pow(10, 8)).toString(),
    }
  } else if (mode === MODE_UNFREEZE) {
    tx.unfreeze_order = {
      from: base64js.fromByteArray(crypto.decodeAddress(context.wallet.address)),
      symbol: selectedCoin,
      amount: (parseFloat(values.amount) * Math.pow(10, 8)).toString(),
    }
  } else {
    throw new Error("invalid mode")
  }
```

