# Sweeping a Liana Miniscript Wallet Using Bitcoin Core and a Coldcard

This exercise was inspired by the Liana team's recovery instructions:  
https://github.com/wizardsardine/liana/blob/master/doc/RECOVER.md

---

# Assumptions

The year is 2125. You’ve just woken up from your cryogenic freeze.  

You’re hungry and need a snack after your 100-year sleep — but how are you going to pay for it?  

Luckily, the world is on a Bitcoin standard. You still have your old Liana wallet miniscript descriptor, a dusty old Coldcard, and a cryptosteel seed for one of the recovery paths in your pocket. Unluckily, the Liana team has long since retired to Mars, and there isn’t an installable version of the wallet left. Bitcoin Core v100 has just been released, and it still supports miniscript descriptors as it did in 2025.

* **If you are ever in this hypothetical situation, you should find a professional Bitcoin recovery expert.** This set of steps is for educational purposes only, to give you confidence that you could, potentially, do this yourself. The goal is to learn something new and have fun doing it.
* You understand the basics of setting up a Liana miniscript wallet, running your own node, using a Coldcard, and working with `bitcoin-cli`. If you don’t, I recommend learning from the BTC Sessions site.
* One of your active recovery paths only requires a single signature.
* You are running **Bitcoin Core v29.0+**, which makes importing Liana descriptors easier.
* You know how to run **Bitcoin Core in "signet" mode**, and you have some "signet" test coins in an address on the Liana wallet you are restoring. Testing one of the Liana releases when the team asks for volunteers is a good way to get some signet coins if you don't have any yet.
* You have installed the latest **"Edge" (experimental) firmware on your Coldcard**, which supports miniscript wallets.
* You have already initialized the Coldcard with the recovery path seed.
* You have your **Coldcard in testnet4 mode**. (`Advanced/Tools -> Danger Zone -> Testnet Mode -> Testnet4`)

Ok, let’s go.

---

## 1. Load Descriptor into Coldcard
First, import the **raw Miniscript descriptor** into the Coldcard:  
* Save the raw descriptor in a text file on an SD card (I used a `.txt` extension, which Coldcard accepted).  
* Import the descriptor into the Coldcard (`Settings -> Miniscript -> Import`).  

The raw descriptor should look something like this (and we will use this descriptor in this exercise):

```
wsh(or_i(and_v(v:pkh([b56686c1/48'/1'/0'/2']tpubDEM3trD2yFgHKhmTEJQwj7E36cZiukJ5YNWSXBaJwWkiwt2pKgZ3bKwWQHy561FW7QGG8JQvXPcxdnFq9TSA62mBPsas7ZyiB6RbWrtB6ps/<2;3>/*),older(2)),or_i(and_v(v:pkh([c2bb89a7/48'/1'/0'/2']tpubDEkGHwx49AQfVEDr9hPNTn88MCZkHnBeVRQgY2jg8oeaHzMZexmsEi77jvpn6cjVVSWuhSWTpSE2hcB4BfWBP3QjiDtzTSPyQNHB239veqQ/<2;3>/*),older(1)),and_v(v:pk([c2bb89a7/48'/1'/0'/2']tpubDEkGHwx49AQfVEDr9hPNTn88MCZkHnBeVRQgY2jg8oeaHzMZexmsEi77jvpn6cjVVSWuhSWTpSE2hcB4BfWBP3QjiDtzTSPyQNHB239veqQ/<0;1>/*),pk([b56686c1/48'/1'/0'/2']tpubDEM3trD2yFgHKhmTEJQwj7E36cZiukJ5YNWSXBaJwWkiwt2pKgZ3bKwWQHy561FW7QGG8JQvXPcxdnFq9TSA62mBPsas7ZyiB6RbWrtB6ps/<0;1>/*)))))#qxexzqcw
```

For reference, this is the policy enforced by the descriptor:

> **2 signatures** out of 2 by **Hot Signer** and **Coldcard** can always spend this wallet's funds (Primary path).  
> **1 signature** by **Coldcard** can spend coins inactive for **1 block (~10m)** (Recovery path #1).  
> **1 signature** by **Hot Signer** can spend coins inactive for **2 blocks (~20m)** (Recovery path #2).  

---

## 2. Create Wallet with Liana & Rescan
* Create a wallet in Bitcoin Core:  

```bash
bitcoin-cli -signet createwallet liana_recovery true
```

* Next, load the descriptor into the Bitcoin Core wallet:  

```bash
bitcoin-cli -signet -rpcwallet=liana_recovery importdescriptors "[{\"desc\":\"wsh(or_i(and_v(v:pkh([b56686c1/48'/1'/0'/2']tpubDEM3trD2yFgHKhmTEJQwj7E36cZiukJ5YNWSXBaJwWkiwt2pKgZ3bKwWQHy561FW7QGG8JQvXPcxdnFq9TSA62mBPsas7ZyiB6RbWrtB6ps/<2;3>/*),older(2)),or_i(and_v(v:pkh([c2bb89a7/48'/1'/0'/2']tpubDEkGHwx49AQfVEDr9hPNTn88MCZkHnBeVRQgY2jg8oeaHzMZexmsEi77jvpn6cjVVSWuhSWTpSE2hcB4BfWBP3QjiDtzTSPyQNHB239veqQ/<2;3>/*),older(1)),and_v(v:pk([c2bb89a7/48'/1'/0'/2']tpubDEkGHwx49AQfVEDr9hPNTn88MCZkHnBeVRQgY2jg8oeaHzMZexmsEi77jvpn6cjVVSWuhSWTpSE2hcB4BfWBP3QjiDtzTSPyQNHB239veqQ/<0;1>/*),pk([b56686c1/48'/1'/0'/2']tpubDEM3trD2yFgHKhmTEJQwj7E36cZiukJ5YNWSXBaJwWkiwt2pKgZ3bKwWQHy561FW7QGG8JQvXPcxdnFq9TSA62mBPsas7ZyiB6RbWrtB6ps/<0;1>/*)))))#qxexzqcw\", 
\"range\": [0,10000], 
\"timestamp\": 1756684800, 
\"active\": true}]"
```

*Note: The timestamp above (Sept 1, 2025) is the start date/time for Bitcoin Core to rescan the blockchain for transactions related to the wallet. You need to set this timestamp before the first transaction for the wallet. The further back you set it, the longer the rescan will take.*  

---

## 3. Get a List of UTXOs
Now we need a list of UTXOs from the wallet. Soon we’ll sweep them all to a new address.  

When constructing a PSBT for the Coldcard, we must include how many blocks have passed since each UTXO was confirmed. This ensures the Coldcard knows the recovery path conditions are met (or not). In the command below we will convert the number of confirmations for each UTXO into a 'sequence' value which will let the Coldcard determine that it can sign the recovery path which in this example was enabled after a single block passed.

Run:

```bash
bitcoin-cli -signet -rpcwallet=liana_recovery listunspent | jq '[.[] | {txid, vout, sequence: .confirmations}]'
```

Example output (2 UTXOs):

```
[
  {
    "txid": "015ce8c92829b2f4615746c4e2064891a77236ac9bc145f6a437fdef58ee0286",
    "vout": 0,
    "sequence": 20
  },
  {
    "txid": "670623bcc8d88159588efaf6a78fd75fbc349945f625645df667bb37b8ad1afd",
    "vout": 1,
    "sequence": 20
  }
]
```

---

## 4. Create a `sendall` Transaction
Now we’ll generate a PSBT that sweeps all funds to:  
`tb1qzhequ5d2cdenah4t5e4as90fhygeuflej80egh`.

Include the inputs and sequence values from the previous step, and set a fee rate (10 sats/vB in this example):

```bash
bitcoin-cli -signet -named -rpcwallet=liana_recovery sendall \
  recipients='["tb1qzhequ5d2cdenah4t5e4as90fhygeuflej80egh"]' \
  options='{
    "add_to_wallet": false,
    "fee_rate": 10,
    "inputs": [
      {
        "txid": "015ce8c92829b2f4615746c4e2064891a77236ac9bc145f6a437fdef58ee0286",
        "vout": 0,
        "sequence": 20
      },
      {
        "txid": "670623bcc8d88159588efaf6a78fd75fbc349945f625645df667bb37b8ad1afd",
        "vout": 1,
        "sequence": 20
      }
    ]
  }'
```

This will output a PSBT:

```
{
  "psbt":"cHNidP8BA...gAAAAIACAACAAgAAAAMAAAAAAA==",
  "complete": false
}
```

Save the PSBT (everything inside the double quotes) to an SD card file, e.g., `recover-all.psbt`. Sign it with the Coldcard. 

Almost done!

---

## 5. Broadcast the Signed PSBT
The Coldcard will create a file such as `recover-all-part.psbt`. Copy the signed PSBT data from the file.  

Finalize the PSBT with Bitcoin Core:

```bash
bitcoin-cli -signet -rpcwallet=liana_recovery finalizepsbt cHNidP...qxoaAAA
```

If successful, you’ll see a transaction hex and `"complete": true`:

```
{
  "hex": "02...0000",
  "complete": true
}
```

Finally, broadcast the hex data:

```bash
bitcoin-cli -signet -rpcwallet=liana_recovery sendrawtransaction 02...0000
```

You should get back a transaction ID indicating success, time for that well-deserved snack! 🍫

---

## Finally...
If I needed to unlock a recovery path that required more than one signature, I’d likely use multiple Coldcards (one per seed) instead of trying to use the HWI interface. I would pass the PSBT between devices, signing it on each, and then transmit the final file. This is my personal preference, as I have no experience with using HWI/connecting hardware signers with it.
