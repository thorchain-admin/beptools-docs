---
description: How to do Atomic Swaps with BEP2 tokens
---

# Swap \(BEP3\)

## Overview

[BEP3 is the ability to swap tokens](https://docs.binance.org/guides/concepts/atomic-swaps.html#deposit-htlt) on Binance Chain. It is meant to work with an Ethereum relay contract to enable BEP2&lt;&gt;ERC-20 tokens, but it also works really well to swap just BEP2 tokens.

There are three parts:

1. Seller sets up the Swap Transaction with the tokens to sell. 
2. Buyer deposits their capital \(BNB or BUSD\).
3. Seller claims the capital, the swap funds are exchanged.  

### Get the CLI tool

You must be running the latest Binance Chain CLI.

#### Testnet

Download here: [https://github.com/binance-chain/node-binary/blob/master/cli/testnet/0.6.3/mac/tbnbcli](https://github.com/binance-chain/node-binary/blob/master/cli/testnet/0.6.3/mac/tbnbcli)

#### Mainnet

Download here: [https://github.com/binance-chain/node-binary/blob/master/cli/prod/0.6.3/mac/bnbcli](https://github.com/binance-chain/node-binary/blob/master/cli/prod/0.6.3/mac/bnbcli)

{% hint style="info" %}
Download the binary off Github Large-File-Storage using the "DOWNLOAD" button. The binary will download to your downloads file
{% endhint %}

Open terminal, navigate to where the binary is, then enable full permissions:

```text
cd downloads
sudo chmod 777 tbnbcli
```

 The `bnbcli` or `tbnbcli` is now ready to run. 

### Get accounts ready

Ensure you already have the accounts added to the cli tool:

```text
./tbnbcli keys list
```

If not, then add it: [https://docs.binance.org/api-reference/cli.html](https://docs.binance.org/api-reference/cli.html)

Check the balances on your accounts first:

```text
./tbnbcli account tbnb1vh0ka0na9ded8h34vd55e9qpa4qhe5cvql9nu3 --chain-id Binance-Chain-Nile --node https://data-seed-pre-1-s3.binance.org:443 --indent
```

### Run the Transactions

#### 1\) HTLT Transaction

Seller makes the `HTLT` transaction first, specifying the buyer's address, the length of time to leave the offer up \(`height-span`\), the amount of selling tokens and the expected purchase amount.

{% hint style="info" %}
Binance Chain blocks are 300ms, so here is a cheat sheet:  
1hr: 12,000  
6hrs: 72,000  
24hours: 288,000  
48hours: 576,000
{% endhint %}

The command, and the response:

```text
./tbnbcli token HTLT --from main1 --recipient-addr tbnb1uf4zjj8079mjggv623w5fe4sz770m8nl0ms4pq  --chain-id Binance-Chain-Nile  --height-span 10000 --amount  10000000:TST-D57  --expected-income 10000000:BNB --trust-node --node http://data-seed-pre-0-s3.binance.org:80\

Random number: 3c571c15730cb20e7ff7801e2078cf6087ee1cfa03fc955ada327f70fbdff05f
Timestamp: 1587873748
Random number hash: 763d1133aac27f474963098dd3d44429b615e4ae07ace2b0a48f1c019c568a88

Password to sign with 'main1':

Committed at block 79094976 (tx hash: FB9A4BE68AE07DB4E0918309271106DB09D887CBD5C8DDD2174F7FCB4A13608A, 
response: {Code:0 Data:[59 117 104 91 139 9 249 208 252 4 236 252 209 94 103 63 184 1 188 181 59 108 255 179 101 70 61 69 58 171 188 250] 
Log:Msg 0: 
swapID: 3b75685b8b09f9d0fc04ecfcd15e673fb801bcb53b6cffb365463d453aabbcfa 
Info: GasWanted:0 GasUsed:0 
Events:...
```

Example: [https://testnet-explorer.binance.org/tx/FB9A4BE68AE07DB4E0918309271106DB09D887CBD5C8DDD2174F7FCB4A13608A](https://testnet-explorer.binance.org/tx/FB9A4BE68AE07DB4E0918309271106DB09D887CBD5C8DDD2174F7FCB4A13608A)

There are two important bits of data:

1. The `random number`
2. The `swapID`

Save the number securely and give the `swapID` to the buyer. 

#### 2\) Deposit Transaction

Beller makes the `deposit` transaction next, adding the `swapID` and the buying amount of tokens:

```text
./tbnbcli token deposit --swap-id 3b75685b8b09f9d0fc04ecfcd15e673fb801bcb53b6cffb365463d453aabbcfa  --amount 10000000:BNB --from main2 --chain-id Binance-Chain-Nile --trust-node --node http://data-seed-pre-0-s3.binance.org:80

Password to sign with 'main2':

Committed at block 79095254 (tx hash: F575E851194196F96AE6ED8286133D74ED8748871AA8934A71E06840601FCF17, 
response: {Code:0 Data:[] 
Log:Msg 0:  
Info: GasWanted:0 GasUsed:0 
Events:...
```

Example: [https://testnet-explorer.binance.org/tx/F575E851194196F96AE6ED8286133D74ED8748871AA8934A71E06840601FCF17](https://testnet-explorer.binance.org/tx/F575E851194196F96AE6ED8286133D74ED8748871AA8934A71E06840601FCF17)

#### 3\) Claim Transaction

Finally, the seller should claim the deposit and in doing so, swap tokens with the Buyer. 

```text
./tbnbcli token claim --swap-id  3b75685b8b09f9d0fc04ecfcd15e673fb801bcb53b6cffb365463d453aabbcfa  --random-number 3c571c15730cb20e7ff7801e2078cf6087ee1cfa03fc955ada327f70fbdff05f --from main1  --chain-id Binance-Chain-Nile --trust-node --node http://data-seed-pre-0-s3.binance.org:80

Password to sign with 'main1':

Committed at block 79095300 (tx hash: 12B0082D68FF2EB5746A8AB5AF8D630EB1ECE80116BC2AB1FC430CA47669DB44, response: {Code:0 Data:[] Log:Msg 0:  Info: GasWanted:0 GasUsed:0 
Events:...
```

Example:[https://testnet-explorer.binance.org/tx/12B0082D68FF2EB5746A8AB5AF8D630EB1ECE80116BC2AB1FC430CA47669DB44](https://testnet-explorer.binance.org/tx/12B0082D68FF2EB5746A8AB5AF8D630EB1ECE80116BC2AB1FC430CA47669DB44)

## Conclusion

We have used BEP3 swap tool to complete a trustless swap of BEP2 tokens on Binance Chain. 









