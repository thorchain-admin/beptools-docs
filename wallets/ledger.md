---
description: How to connect to Ledger
---

# Ledger

## How to connect to Ledger

### Setup

We need to make three additions to our app to connect Ledger. 

1. Update our `Binance.js` file
2. Add a Ledger connect option to the **Unlock Page**
3. Add the ledger methods to our **Transaction Page**

**Dependencies**

Install these with NPM or Yarn:

* `@ledgerhq/hw-transport-u2f`
* `@binance-chain/javascript-sdk`

### Binance.js

In our `Binance.js` file we need to add the Ledger Signing Delegate:

```text
  useLedgerSigningDelegate(ledgerApp, preSignCb, postSignCb, errCb, hdPath) {
    return this.bnbClient.useLedgerSigningDelegate(ledgerApp, preSignCb, postSignCb, errCb, hdPath)
  }
```

### Unlock Page

On our unlock page we should import the `ledger` module, as well as `u2f_transport` package from Ledger:

```text
import { ledger, crypto } from '@binance-chain/javascript-sdk'
import u2f_transport from '@ledgerhq/hw-transport-u2f'

ledger.transports.u2f = u2f_transport
window.ledger = ledger
```

We use React Context to manage app-wide wallet state, as well as React Hooks to manage state for this page. 

Since the Ledger unlocking process requires user input to a hardware device, we add a `connecting` flag. We also need to capture the index the user chooses on their ledger, with the default value of `0`

```text
const context = useContext(Context)
const [connecting, setConnecting] = useState(false)
const [ledgerIndex, setLedgerIndex] = useState(0)
```

The ledgerConnect process is as follows:

1. Turn the `connecting` flag to true
2. Require approval from the user \(5 second timeout\)
3. Set up the `u2f` transport layer
4. Set the HDPath to `[44, 714, 0, 0, ledgerIndex]` 
5. Get the public key with the right Binance prefix \(`tbnb` or `bnb` depending on net\)
6. Set the details into context and push to the next page.
7. 
{% hint style="info" %}
The HD Path \(Hierarchial Deterministic\) is a standard of generating public-private key pairs and their associated addresses for different chains. 

* **Purpose**: 44 \(BIP44 specification from Bitcoin Core\)
* **Coin:** \(Bitcoin is 0, Ethereum is 60, Binance Chain is 714\)
* **Account**: 0...n \(This changes the account root, creating a whole new set of key-pairs\)
* **Change**: 0 or 1 \(This is used for internal \(0\)  and external \(1\) derivation.\)
* **Index**: 0...n \(Iterate through this\)

You can learn more about it here: [https://iancoleman.io/bip39/](https://iancoleman.io/bip39/)
{% endhint %}

```text
const ledgerConnect = async () => { 
setConnecting(true) 
message.success(Please approve on your ledger, 5)

  // use the u2f transport
  const timeout = 50000
  const transport = await ledger.transports.u2f.create(timeout)
  const app = window.app = new ledger.app(transport, 100000, 100000)
  
  // we can provide the hd path (app checks first two parts are same as below)
  const hdPath = [44, 714, 0, 0, ledgerIndex]
  
  // get public key
  try {
    let pk = (await app.getPublicKey(hdPath)).pk
  
    // get address from pubkey
    const address = crypto.getAddressFromPublicKey(pk, Binance.getPrefix())
  
    context.setContext({
      "wallet": {
        "address": address,
        "ledger": app,
        "hdPath": hdPath,
      }
    }, () => {
      setConnecting(false)
      //Do next after connecting
    })
  } catch (err) {
    message.error("public key error" + err.message)
    setConnecting(false)
    return
  }
}
```



We need some UI elements, which include the Index input and the button to connect. We also add some nice graphics, and some info text to match Binance Chain page. The key elements are in the input element, and the button towards the bottom. 

![](../.gitbook/assets/image%20%286%29.png)

```text
<div>
      <Row style={{ marginBottom: 20 }}>
        <Text size={18}>Connect your Ledger device</Text>
      </Row>
      <Row>
        <Col span={3}>
          <Icon icon="step1" alt="Step 1" />
        </Col>
        <Col span={8}>
          <Text bold>Enter PIN Code</Text>
        </Col>
        <Col>
          <Icon icon="pincode" style={ledgerCSS} />
        </Col>
      </Row>
      <Row style={{ marginTop: 20 }}>
        <Col span={3}>
          <Icon icon="step2" alt="Step 2" />
        </Col>
        <Col span={8}>
          <Row>
            <Text bold>Open Binance Chain</Text>
          </Row>
          <Row>
            <Text size={10}>
              “Binance Chain Ready” must be on-screen
            </Text>
          </Row>
        </Col>
        <Col>
          <Icon icon="openapp" alt="" style={ledgerCSS} />
        </Col>
      </Row>
      <Row style={{ marginTop: 20 }}>
        <Col span={6}>
          <div>
            <a
              href="https://www.binance.org/static/guides/DEX-Ledger-Documentation.html"
              rel="noopener noreferrer"
              target="_blank"
            >
              <Text size={10} color="#F0B90B">
                App Installation & Usage Instructions
              </Text>
            </a>
          </div>
          <div>
            <a
              href="https://support.ledger.com/hc/en-us/articles/115005165269-Connection-issues-with-Windows-or-Linux"
              rel="noopener noreferrer"
              target="_blank"
            >
              <Text size={10} color="#F0B90B">
                Having Connection Issues?
              </Text>
            </a>
          </div>
        </Col>
        <Col span={12}>
          <Row>
           <Col span={9}>
              <div>
                <div>
                  <Text size={12}>Index Number (default 0)</Text>
                </div>
                <InputNumber
                  min={0}
                  size='medium'
                  value={ledgerIndex}
                  onChange={(i) => { setLedgerIndex(i) }}
                  style={{verticalAlign:"text-bottom"}}
                />
              </div>
            </Col>
            <Col span={12}>
              <Button
                onClick={ledgerConnect}
                loading={connecting}
                fill={true}
                style={{marginTop:24}}
              >
                Connect to Ledger <AntIcon type="arrow-right" />
              </Button>
            </Col>
          </Row>
        </Col>
      </Row>
    </div>
```

### Transaction Page

Once the transaction is prepared by taking user input, this is the method to call:

```text
    // setup binance client for authentication
   if (context.wallet.ledger) {
        binance.useLedgerSigningDelegate(
          context.wallet.ledger,
          null, null, null,
          context.wallet.hdPath,
        )
      } else {
        throw new Error("no wallet detected")
      }

      try {
      // The transaction method (transfer, send, freeze etc)
        const results = window.results = await Binance.multiSend(context.wallet.address, transactions, memo)
        
        if (results.result[0].ok) {
          const txURL = Binance.txURL(results.result[0].hash)
          message.success(<Text>Sent. <a target="_blank" rel="noopener noreferrer" href={txURL}>See transaction</a>.</Text>)
        }
      } catch(err) {
        window.err = err
        message.error(err.message)
      }
    }

```

We handle errors in the browser in order to display them to the user. `message` is an antdesign component for throwing up messages to the user. 

### Freeze Transactions

The following is the only difference when sending a freeze transaction:

```text
  var results
  if (mode === MODE_FREEZE) {
    results = await Binance.freeze(context.wallet.address, selectedCoin, values.amount)
  } else if (mode === MODE_UNFREEZE) {
    results = await Binance.unfreeze(context.wallet.address, selectedCoin, values.amount)
  } else {
    throw new Error("invalid mode")
  }
```





