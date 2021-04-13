---
description: Let's make some native assets on Cardano ❤️✨
---

# Cardano Native Asset and NFT Tutorial

## Who is this guide for?

* For people who want to make NFT's
* For people who know about Cardano 

## Benefits of NFT's on Cardano

* Low transaction fees
* Native on the blockchain 

## Prerequisites

* cardano-node / cardano-cli set up on local machine
* Make sure you have a Cardano node running and fully synced to database
* Make sure node.js installed

```bash
#Copy/Paste this into your terminal if node.js is not intsalled
curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt-get install -y nodejs
```

{% hint style="warning" %}
We made this tutorial on Rasberry-Pi-ARM machines so make sure to download the **correct** node.js for your **local machine/CPU and OS**.
{% endhint %}

### Verify everything is set up properly on our machine ⚙️

```bash
#Copy/paste into terminal window
cardano-cli version; cardano-node version
```

 Your output should look like this 👇 

```bash
cardano-cli 1.26.1 - linux-aarch64 - ghc-8.10
git rev 0000000000000000000000000000000000000000
cardano-node 1.26.1 - linux-aarch64 - ghc-8.10
git rev 0000000000000000000000000000000000000000
```

#### Verify our node.js version is correct and is on v14.16.0

```bash
#Copy/paste into terminal window
node -v
```

```bash
v14.16.0
```

## Create our project directory and initial setup

```bash
#make sure our db is in our $PATH
CARDANO_NODE_SOCKET_PATH="$NODE_HOME/db/socket"

mkdir minter
cd minter 
npm init -y #creates package.json)
npm install cardanocli-js --save
```

1. **Copy the Cardano node genesis latest build number from the IOHK hydra website**
   * [https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/index.html](https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/index.html) 
2. **Create a bash shell script to Download the latest Genesis config file needed**

```bash
sudo nano fetch-config.sh
```

```bash
#NODE_BUILD_NUM may be different
NODE_BUILD_NUM=5822084
wget -N https://hydra.iohk.io/build/${NODE_BUILD_NUM}/download/1/mainnet-shelley-genesis.json
```

**Now we need to give permissions to our new script to execute then we will run our script and download the genesis files.** 

```bash
sudo chmod +x fetch-config.sh
./fetch-config.sh
```

### Next, we make our src folder/directory and then create the Cardano client.

```bash
mkdir src; cd src
sudo nano cardano.js
```

```javascript
const Cardano = require("cardanocli-js");

const cardano = new Cardano({
  network: "mainnet",
  dir: __dirname + "/../",
  shelleyGenesisPath: __dirname + "/../mainnet-shelley-genesis.json",
});

module.exports = cardano;
```

## Create a wallet

```bash
sudo nano create-wallet.js
```

```javascript
const cardano = require('./cardano')

const createWallet = (account) => {
  cardano.addressKeyGen(account);
  cardano.stakeAddressKeyGen(account);
  cardano.stakeAddressBuild(account);
  cardano.addressBuild(account);
  return cardano.wallet(account);
};

createWallet("ADAPI")
```

```bash
cd minter
node src/create-wallet.js
```

#### Verify balance wallet balance is Zero, then we fund the wallet

* **First, we need to create a get-balance.js script**

```bash
cd minter/src; sudo nano get-balance.js
```

```javascript
// create get-balance.js
const cardano = require('./cardano')

const sender = cardano.wallet("ADAPI");

console.log(
    sender.balance()
)
```

1. check the balance \(utxo\)

```text
cd ..
node src/get-balance.js
```

1. Download IPFS
2. Upload your files to IPFS

* image - ipfs://QmQqzMTavQgT4f4T5v6PWBp7XNKtoPmC9jvn12WPT3gkSE
* src - ipfs://Qmaou5UzxPmPKVVTM9GzXPrDufP55EDZCtQmpy3T64ab9N

1. Generate policy id
2. Define your meta data
3. create mint transaction

```javascript
const fs = require("fs");
const cardano = require("./cardano");

// 1. Get the wallet
// 2. Define mint script
// 3. Create POLICY_ID
// 4. Define ASSET_NAME
// 5. Create ASSET_ID
// 6. Define metadata
// 7. Define transaction
// 8. Build transaction
// 9. Sign transaction
// 10. Submit transaction

const buildTransaction = (tx) => {
  const raw = cardano.transactionBuildRaw(tx);
  const fee = cardano.transactionCalculateMinFee({
    ...tx,
    txBody: raw,
  });
  tx.txOut[0].amount.lovelace -= fee;
  return cardano.transactionBuildRaw({ ...tx, fee });
};

const signTransaction = (wallet, tx, script) => {
  return cardano.transactionSign({
    signingKeys: [wallet.payment.skey, wallet.payment.skey],
    scriptFile: script,
    txBody: tx,
  });
};

const wallet = cardano.wallet("ADAPI");

const mintScript = {
  keyHash: cardano.addressKeyHash(wallet.name),
  type: "sig",
};

const POLICY_ID = cardano.transactionPolicyid(mintScript);
const ASSET_NAME = "BerrySpaceGreen";
const ASSET_ID = POLICY_ID + "." + ASSET_NAME;

const metadata = {
  721: {
    [POLICY_ID]: {
      [ASSET_NAME]: {
        name: ASSET_NAME,
        image: "ipfs://QmQqzMTavQgT4f4T5v6PWBp7XNKtoPmC9jvn12WPT3gkSE",
        description: "Super Fancy Berry Space Green NFT",
        type: "image/png",
        src: "ipfs://Qmaou5UzxPmPKVVTM9GzXPrDufP55EDZCtQmpy3T64ab9N",
        authors: ["PIADA", "SBLYR"],
      },
    },
  },
};

const tx = {
  txIn: wallet.balance().utxo,
  txOut: [
    {
      address: wallet.paymentAddr,
      amount: { ...wallet.balance().amount, [ASSET_ID]: 1 },
    },
  ],
  mint: [{ action: "mint", amount: 1, token: ASSET_ID }],
  metadata,
  witnessCount: 2,
};

const raw = buildTransaction(tx);
const signed = signTransaction(wallet, raw, mintScript);
const txHash = cardano.transactionSubmit(signed);
console.log(txHash);
```

1. Run the minting script, then wait a few moments to check the balance \(utxo\)

```text
cd ..
node src/mint-asset.js
```

1. send your nft back to your wallet
   * Create anew script to send nft to wallet

```text
const cardano = require("./cardano");

// 1. get the wallet
// 2. define the transaction
// 3. build the transaction
// 4. calculate the fee
// 5. pay the fee by subtracting it from the sender utxo
// 6. build the final transaction
// 7. sign the transaction
// 8. submit the transaction

const sender = cardano.wallet("ADAPI");

console.log(
  "Balance of Sender wallet: " +
    cardano.toAda(sender.balance().amount.lovelace) +
    " ADA"
);

const receiver =
  "addr1qym6pxg9q4ussr96c9e6xjdf2ajjdmwyjknwculadjya488pqap23lgmrz38glvuz8qlzdxyarygwgu3knznwhnrq92q0t2dv0";

const txInfo = {
  txIn: cardano.queryUtxo(sender.paymentAddr),
  txOut: [
    {
      address: sender.paymentAddr,
      amount: {
        lovelace: sender.balance().amount.lovelace - cardano.toLovelace(1.5),
      },
    },
    {
      address: receiver,
      amount: {
        lovelace: cardano.toLovelace(1.5),
        "ad9c09fa0a62ee42fb9555ef7d7d58e782fa74687a23b62caf3a8025.BerrySpaceGreen": 1,
      },
    },
  ],
};

const raw = cardano.transactionBuildRaw(txInfo);

const fee = cardano.transactionCalculateMinFee({
  ...txInfo,
  txBody: raw,
  witnessCount: 1,
});

//pay the fee by subtracting it from the sender utxo
txInfo.txOut[0].amount.lovelace -= fee;

//create final transaction
const tx = cardano.transactionBuildRaw({ ...txInfo, fee });

//sign the transaction
const txSigned = cardano.transactionSign({
  txBody: tx,
  signingKeys: [sender.payment.skey],
});

//subm transaction
const txHash = cardano.transactionSubmit(txSigned);
console.log("TxHash: " + txHash);
```

1. view your nft in your wallet
2. View your asset on cardanoassets.com
3. View your asset on pool.pm \(see the actual picture\)
4. Show the original minting metadata
5. open the src ipfs to prove that it work

