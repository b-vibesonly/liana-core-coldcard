# Nunchuk Variations

The main focus of this repo was to document a Liana specific exercise - see the original README.md 

A similar workflow can be used with the additional timelock types the Nunchuk miniscript solution offers.

Below are examples of creating a correct PSBT for the Coldcard to sign, not all types or variations of wallet are covered. LLMs such as ChatGPT 5 are very good at helping with the construction and debugging of such `bitcoin-cli` commands. For the sake of privacy I personally don't recommend you use LLMs for help with real mainnet `bitcoin-cli` tasks.

## Absolute Block Height Based Locks

Note the creation of a `sendall` transaction is very similar when using the Nunchuk absolute block height based policy.

We just need to include the `locktime` and set it to the **current block height** (of signet).

```bash
bitcoin-cli -signet -named -rpcwallet=nunblocks sendall \
    recipients='["tb1qzhequ5d2cdenah4t5e4as90fhygeuflej80egh"]' \
    options='{
    "add_to_wallet": false,
    "locktime" : 272250,
    "fee_rate": 10,
    "inputs":  [
      {
        "txid": "2501630b5a94d9ba3f3ad784f18a8096373018d2584c1acfff3bbaa8025c8b84",
        "vout": 0,
        "sequence": 308
      },
      {
        "txid": "6b6661892dde2362a71e11e6279561cde0ac93a33168a035b043f2b07a58ed23",
        "vout": 0,
        "sequence": 308
      },
      {
        "txid": "37cacf6ab0966e935157d0a1ecb220a6a6cbd09f035f0825dff1a4b128595a1f",
        "vout": 0,
        "sequence": 308
      }
    ]
  }'
```
## Absolute Time Based Locks

In this case the creation of a `sendall` transaction requires us to include the `locktime` and set it to the current date and time as represented by a UNIX timestamp.
Additionally the sequence is set to a specific value of **4294967293** which is a special code that is comonnly used with an absolute locktime.
```bash
bitcoin-cli -signet -named -rpcwallet=nuntime sendall \
    recipients='["tb1qzhequ5d2cdenah4t5e4as90fhygeuflej80egh"]' \
    options='{
    "add_to_wallet": false,
    "replaceable": true,
    "fee_rate": 2,
    "locktime": 1759326300,
    "inputs": [
      { "txid": "015ce8c92829b2f4615746c4e2064891a77236ac9bc145f6a437fdef58ee0286",
        "vout": 1,
        "sequence": 4294967293
      }
    ]}'

```
