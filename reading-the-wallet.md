---
description: How to read the wallet and display a list of balances.
---

# Reading the Wallet

## Balances

We should have a mechanism to read the balance of the address. We can do this by querying the context, then passing the stored address back to our Binance client to return the balances. 

We then need to parse the balances and load them into a local array

```text
  // Get the context which contains the wallet address
  const context = useContext(Context)
  const [balances, setBalances] = useState(null)
  
  const getBalances = () => {
    if (context.wallet && context.wallet.address) {    
      Binance.getBalances(context.wallet.address)
        .then((response) => {        
          const b = (response || []).map((bal) => (
            {
              "icon": "coin-bep",                  // Custom Icon
              "ticker": bal.symbol,
              "free": parseFloat(bal.free),
              "frozen": parseFloat(bal.frozen),
              "locked": parseFloat(bal.locked),
            }
          ))
          setBalances([...b])
           
        })
        .catch((error) => {
           
        })
    }
  }
```

We use some React hooks and a mapping to display a row of Coins. Note, we use some AntDesign `Row, Col` components, and our own `Coin` component. 

```text
 const [selectedCoin, setSelectedCoin] = useState(null)
 
 ...
 
     {(balances || []).map((coin) => (
          <Row key={coin.ticker} style={coinRowStyle}>
            <Col xs={24} sm={24} md={12} lg={8} xl={6}>
              <Coin {...coin} onClick={setSelectedCoin} border={selectedCoin === coin.ticker}/>
            </Col>
          </Row>
        ))
     }
```

This is our custom `Coin` method, which also uses the `Center, Icon` AntDesign components. 

{% code title="/src/Components.js" %}
```text
const Coin = ({onClick, icon, ticker, free, border}) => {
  let styles = {width: "100%", paddingLeft: 30, cursor: "pointer", padding: 5}
  if (border) {
    styles.border = "1px solid #F0B90B"
    styles.borderRadius = 6
  }
  return (
    <Center>
      <div style={styles} onClick={() => { if (onClick) { onClick(ticker) } }}>
        <Icon icon={icon} />
        <span style={{margin: "0px 10px"}}>
          {ticker}
        </span>
        <span style={{marginLeft: 10, float: 'right'}}>
          {AmounttoString(free)}
        </span>
      </div>
    </Center>
  )
}
Coin.defaultProps = {
  border: false,
}
```
{% endcode %}

The final UI should look like:

![Our custom row of Coins, with Icon, Balance and a Selection Picker. ](.gitbook/assets/image%20%285%29.png)



