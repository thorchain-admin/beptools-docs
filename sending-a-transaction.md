---
description: How to send a simple transfer of coins.
---

# Sending a Transaction

The user should use a variety of forms and buttons to input their intent to send a transaction, including the sending coin, amount and destination addresses. 

You should also let them add a `MEMO` to describe the transaction.

Assuming you already have a field to capture the `password, amount, coin` you then need to detect how we unlocked the wallet, using Context, and pass in the details::

```text
if (context.wallet.keystore) {
        try {
          const privateKey = crypto.getPrivateKeyFromKeyStore(
            context.wallet.keystore,
            password
          )
          binance.setPrivateKey(privateKey)

        } catch(err) {
          window.err = err
          message.error(err.message)
          return
        }
      }
```

We can then try to send a transaction by passing in the transaction variables to the Binance client:

```text
try {
        const results = window.results = await Binance.transfer(context.wallet.address, toAddress, amount, asset, memo)

        if (results.result[0].ok) {
          const txURL = Binance.txURL(results.result[0].hash)
          message.success(<Text>Sent. <a target="_blank" rel="noopener noreferrer" href={txURL}>See transaction</a>.</Text>)
        }
        
      } catch(err) {
        window.err = err
        message.error(err.message)
        setPassword(null) // clear password
      }
  }
```

We use messages to gracefully handle the transaction response and any errors. 



