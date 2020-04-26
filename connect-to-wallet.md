---
description: How to connect to a Keystore Wallet.
---

# Wallets

## Connecting to Different Wallets

There are three different wallets that we can connect, but we'll start with Keystore. Keystore is simple and you can get up and running fast. To create a keystore: [https://www.binance.org/en/create](https://www.binance.org/en/create)

{% hint style="warning" %}
Whilst we can add **Mnemonic Phrase** option, it is not recommended for security reasons so we simply link our users to this page, where they re-build their Keystore:

[https://www.binance.org/en/recover](https://www.binance.org/en/recover)
{% endhint %}

### Wallet State Management

In order to persist state about a connected wallet, you should use a variable, service \(Angular App\) or use Context \(React App\).

In this example we will use Context. Here is some of the scaffolding you will need:

{% code title="src/components/pages/wallet/keystore.js" %}
```text

import React, { useState, useContext } from 'react'
...
const Keystore = props => {
  ...
  const context = useContext(Context)
  ...
}
export default Keystore
```
{% endcode %}

### Keystore

In our file we need to import the `crypto` library, the `react-file-picker` and the Binance client we created earlier:

```text
import { crypto } from '@binance-chain/javascript-sdk'
import { FilePicker } from 'react-file-picker'
import Binance from "../../../clients/binance"
```

We then add some react hooks to set state variables, as well as the file-picker method:

```text
  const [keystore, setKeystore] = useState(null)
  const [password, setPassword] = useState(null)
  const [keystoreError, setKeystoreError] = useState(null)

// File reader methods
  var reader = new FileReader();
  
  reader.onload = () => {
    try {
      const key = JSON.parse(reader.result)
      if (!('version' in key) || !('crypto' in key)) {
        setKeystoreError("Not a valid keystore file")
      } else {
        setKeystoreError(null)
        setKeystore(key)
      }
    } catch {
      setKeystoreError("Not a valid json file")
    }
  }
  
// Connect your upload button to this
  const uploadKeystore = f => {
    reader.readAsText(f)
  }
```

This enough now to let the user click a button to upload a file, select a keystore file, do some simple validation on the file, then upload it. 

We then need to add an input field to allow the user to enter a password, then a button to unlock the wallet and persist the keystore and address into the Context. 

Importantly, we are **not** going to save the privatekey or password the user types in, we will **only** store how the wallet is connected and the public address:

```text
  // Connect your password input to this
  const onPasswordChange = e => {
    setPassword(e.target.value)
  }
  
  // Connect your unlock button to this
  const unlock = () => {
    const privateKey = crypto.getPrivateKeyFromKeyStore(keystore, password)
    const address = crypto.getAddressFromPrivateKey(privateKey, Binance.getPrefix())
    context.setContext({
      "wallet": {
        "keystore": keystore,
        "address": address,
      }
    }, () => {
      props.history.push("/")
    })
  }
```

![The Keystore UI](.gitbook/assets/image%20%284%29.png)

The html for the above pane is below. \(Note: it uses Ant Design `Row, Button, Icon, Text, Input` components\)

```text
<div>
      <Row style={{marginBottom: 10}}>
        <Text size={18}>Select your keystore file</Text>
      </Row>
      <Row style={{marginBottom: 10}}>
        <FilePicker
          onChange={f => (uploadKeystore(f))}
          onError={err => (console.error(err))}
        >
          <div>
            <Button
              style={{padding: "0px 10px", fontSize: 14}}
              bold={true}
              fill={false}>
              <Icon type="upload" /> Upload keystore file
            </Button>&nbsp;
            {keystore && !keystoreError && 
            <Icon type="check-circle" theme="twoTone" twoToneColor="#52c41a" />
            }
          </div>
        </FilePicker>
        {keystoreError &&
        <Text color='#FF4136'>{keystoreError}</Text>
        }
      </Row>
      <Row style={{marginTop: 30, marginBottom: 10}}>
        <Input.Password
          allowClear
          onChange={onPasswordChange}
          placeholder="Enter wallet password"
        />
      </Row>
      <Row style={{marginBottom: 10}}>
        <Button 
          disabled={keystore === null || password === null}
          onClick={unlock} 
          style={{float: "right"}}
          fill={true}
        >
          Unlock Wallet Now <Icon type="arrow-right" />
        </Button>
      </Row>
    </div>
```

