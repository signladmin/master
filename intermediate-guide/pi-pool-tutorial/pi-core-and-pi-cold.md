---
description: Create operational keys & certificates. Create wallet & register stake pool
---

# Pi-Core/Cold

{% hint style="warning" %}
There exists a way to create your pools wallet by creating it in Yoroi and using cardano-wallet to extract the key pair from the mnemonic seed. This allows you to have a seed backup of the wallet and to also easily extract rewards or send funds elsewhere.

[https://gist.github.com/ilap/3fd57e39520c90f084d25b0ef2b96894​](https://gist.github.com/ilap/3fd57e39520c90f084d25b0ef2b96894)

Cardano-wallet will not build on arm due to dependency failure. @zwerk tried to build it for us and failed. You may want to install cardano-wallet on an offline x86 machine and go through this process. That is how I did it. You can get cardano-wallet binary below.

[https://hydra.iohk.io/build/3770189](https://hydra.iohk.io/build/3770189)
{% endhint %}

## Generate core and cold key requirements

#### Key Evolving Signature keypair

KES keys expire after 90 days and you will have to create a new pair as part of pool operations before they expire.

{% tabs %}
{% tab title="Core" %}
```bash
cd $NODE_HOME
cardano-cli node key-gen-KES \
  --verification-key-file kes.vkey \    
  --signing-key-file kes.skey​
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Cold Offline" %}
```bash
mkdir $HOME/cold-keys
cd cold-keys
cardano-cli node key-gen \
  --cold-verification-key-file node.vkey \
  --cold-signing-key-file node.skey \
  --operational-certificate-issue-counter node.counter
```
{% endtab %}
{% endtabs %}

Create variables with the number of slots per KES period from the genesis file and current tip of the chain.

{% tabs %}
{% tab title="Core" %}
```bash
slotsPerKesPeriod=$(cat $NODE_FILES/${NODE_CONFIG}-shelley-genesis.json | jq -r '.slotsPerKESPeriod')
slotNo=$(cardano-cli query tip --mainnet | jq -r '.slot')
echo slotsPerKesPeriod: ${slotsPerKesPeriod}
echo slotNo: ${slotNo}
```
{% endtab %}
{% endtabs %}

Set the startKesPeriod variable by dividing ${slotNo} / ${slotsPerKESPeriod}.

{% tabs %}
{% tab title="Core" %}
```bash
startKesPeriod=$((${slotNo} / ${slotsPerKesPeriod}))
echo startKesPeriod: ${startKesPeriod}
```
{% endtab %}
{% endtabs %}

Write down startKesPeriod value down & copy the kes.vkey to your cold offline machine.

### Generate node.cert operational certificate

Move kes.vkey to Cold offline machine.

Replace **&lt;startKesPeriod&gt;** with the value you wrote down.

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli node issue-op-cert \
  --kes-verification-key-file kes.vkey \
  --cold-signing-key-file $HOME/cold-keys/node.skey \
  --operational-certificate-issue-counter $HOME/cold-keys/node.counter
  --kes-period <startKesPeriod> \
  --out-file node.cert
```
{% endtab %}
{% endtabs %}

Copy node.cert to your Core machine and create a VRF key pair.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli node key-gen-VRF \
  --verification-key-file vrf.vkey \
  --signing-key-file vrf.skey
```
{% endtab %}
{% endtabs %}

For security purposes the vrf.skey **needs** be read only or cardano-node will not start.

{% tabs %}
{% tab title="Core" %}
```bash
chmod 400 vrf.skey
```
{% endtab %}
{% endtabs %}

Edit the cardano-service startup script by adding kes.skey, vrf.skey and node.cert to the cardano-node run command and changing the port it listens on.

{% tabs %}
{% tab title="Core" %}
```bash
nano $HOME/.local/bin/cardano-service
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}

{% endhint %}

{% tabs %}
{% tab title="Core" %}
```bash
#!/bin/bash
DIRECTORY=/home/ada/pi-pool
FILES=/home/ada/pi-pool/files
PORT=3000
HOSTADDR=0.0.0.0
TOPOLOGY=${FILES}/mainnet-topology.json
DB_PATH=${DIRECTORY}/db
SOCKET_PATH=${DIRECTORY}/db/socket
CONFIG=${FILES}/mainnet-config.json
KES=${DIRECTORY}/kes.skey
VRF=${DIRECTORY}/vrf.skey
CERT=${DIRECTORY}/node.cert
## +RTS -N4 -RTS = Multicore(4)
cardano-node run +RTS -N4 -RTS \
  --topology ${TOPOLOGY} \
  --database-path ${DB_PATH} \
  --socket-path ${SOCKET_PATH} \
  --host-addr ${HOSTADDR} \
  --port ${PORT} \
  --config ${CONFIG} \
  --shelley-kes-key ${KES} \
  --shelley-vrf-key ${VRF} \
  --shelley-operational-certifcate ${CERT}
```
{% endtab %}
{% endtabs %}

Restart your node with new directives.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-service restart
```
{% endtab %}
{% endtabs %}

## Create the pool wallet payment & staking key pairs

{% hint style="danger" %}
**Cold offlline machine.** Take the time to visualize the operations here.

1. _**Generate**_ a wallet key pair named payment. = **payment.vkey** & **payment.skey**
2. _**Generate**_ staking key pair. = **stake.vkey** & **stake.skey**
3. _**Build**_ a stake address from the newly created stake.vkey. = **stake.addr**
4. _**Build**_ a wallet address from the payment.vkey & delegate to it with stake.vkey. = **payment.addr**
5. Add funds to the wallet by sending ada to payment.addr
6. Check balance.
{% endhint %}

### Generate wallet key pair

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cd $NODE_HOME
cardano-cli address key-gen \
  --verification-key-file payment.vkey
  --signing-key-file payment.skey
```
{% endtab %}
{% endtabs %}

### Generate staking key pair

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli stake-address key-gen \
  --verification-key-file stake.vkey \
  --signing-key-file stake.skey
```
{% endtab %}
{% endtabs %}

### Build stake address

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli stake-address build \
  --stake-verification-key-file stake.vkey \
  --out-file stake.addr
  --mainnet
```
{% endtab %}
{% endtabs %}

### Build payment address

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli address build \
  --payment-verification-key-file payment.vkey \
  --stake-verification-key-file stake.vkey \
  --out-file payment.addr \
  --mainnet
```
{% endtab %}
{% endtabs %}

### Fund wallet

```text
cat payment.addr
```

Copy **payment.addr** to a thumb drive and move it to the core nodes pi-pool folder.

Add funds to the wallet. This is the only wallet the pool uses so your pledge goes here as well. There is a 2 ada staking registration fee and a 500 ada pool registration deposit that you can get back when retiring your pool.

{% hint style="warning" %}
Test the wallet by sending a small amount waiting a few minutes and querying it's balance.
{% endhint %}

{% hint style="danger" %}
Core node needs to be synced to the tip of the blockchain.
{% endhint %}

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli query utxo \
  --address $(cat payment.addr) \
  --mainnet
```
{% endtab %}
{% endtabs %}

## Register stake address 🥩

### Create a staking certificate

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli stake-address registration-certificate \
  --stake-verification-key-file stake.vkey \
  --out-file stake.cert
```
{% endtab %}
{% endtabs %}

Copy **stake.cert** to your core node's pi-pool folder.

{% tabs %}
{% tab title="Core" %}
```bash
slotNo=$(cardano-cli query tip --mainnet | jq -r '.slot')
echo slotNo: ${slotNo}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli query utxo \
  --address $(cat payment.addr) \
  --mainnet > fullUtxo.out

tail -n +3 fullUtxo.out | sort -k3 -nr > balance.out   
cat balance.out   
tx_in=""
total_balance=0

while read -r utxo; do
  in_addr=$(awk '{ print $1 }' <<< "${utxo}")
  idx=$(awk '{ print $2 }' <<< "${utxo}")
  utxo_balance=$(awk '{ print $3 }' <<< "${utxo}")
  total_balance=$((${total_balance}+${utxo_balance}))
  echo TxHash: ${in_addr}#${idx}
  echo ADA: ${utxo_balance}
  tx_in="${tx_in} --tx-in ${in_addr}#${idx}"
done < balance.out

txcnt=$(cat balance.out | wc -l)
echo Total ADA balance: ${total_balance}
echo Number of UTXOs: ${txcnt}
```
{% endtab %}
{% endtabs %}

Retrieve stakeAddressDeposit value from params.json.

{% tabs %}
{% tab title="Core" %}
```bash
stakeAddressDeposit=$(cat $NODE_HOME/params.json | jq -r '.stakeAddressDeposit')
echo stakeAddressDeposit : ${stakeAddressDeposit}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Stake address registration is 2,000,000 lovelace or 2 ada.
{% endhint %}

{% hint style="warning" %}
Take note of the invalid-hereafter input. We are taking the current slot number\(tip of the chain\) and adding 1,000 slots. If we do not issue the signed transaction before the chain reaches this slot number the tx will be invalidated. A slot is one second so you have 16.666666667 minutes to get this done. 🐌
{% endhint %}

Build tx.tmp

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli transaction build-raw \
  ${tx_in} \
  --tx-out $(cat payment.addr)+0 \
  --invalid-hereafter $(( ${slotNo} + 10000)) \
  --fee 0 \
  --certificate stake.cert \
  --out-file tx.tmp
```
{% endtab %}
{% endtabs %}

Calculate the minimal fee.

{% tabs %}
{% tab title="Core" %}
```bash
fee=$(cardano-cli transaction calculate-min-fee \
  --tx-body-file tx.tmp \
  --tx-in-count ${txcnt} \
  --tx-out-count 1 \
  --mainnet \
  --witness-count 2 \
  --byron-witness-count 0 \
  --protocol-params-file params.json | awk '{ print $1 }')
echo fee: $fee
```
{% endtab %}
{% endtabs %}

Calculate txOut.

{% tabs %}
{% tab title="Core" %}
```bash
txOut=$((${total_balance}-${stakeAddressDeposit}-${fee}))
echo Change Output: ${txOut}
```
{% endtab %}
{% endtabs %}

Build the full transaction to register your staking address.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli transaction build-raw \
  ${tx_in} \
  --tx-out $(cat payment.addr)+${txOut} \
  --invalid-hereafter $(( ${currentSlot} + 10000)) \
  --fee ${fee} \
  --certificate-file stake.cert \
  --out-file tx.raw
```
{% endtab %}
{% endtabs %}

Transfer tx.raw to your Cold offline machine and sign the transaction with the payment.skey and stake.skey.

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli transaction sign \
  --tx-body-file tx.raw \
  --signing-key-file payment.skey \
  --signing-key-file stake.skey \
  --mainnet \
  --out-file tx.signed
```
{% endtab %}
{% endtabs %}

Move the signed transaction back to the Core.

Submit the transaction to the blockchain.

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli transaction submit \
  --tx-file tx.signed \
  --mainnet
```
{% endtab %}
{% endtabs %}

## Register the pool 🏊♂

Create a poolMetaData.json file. It will contain important information about your pool. You will need to host this file somewhere online. [https://pages.github.com/](https://pages.github.com/) is a popular solution. I say host it on your Pi with NGINX.

{% embed url="https://www.coincashew.com/coins/overview-ada/guide-how-to-build-a-haskell-stakepool-node/how-to-upload-poolmetadata.json-to-github​" caption="" %}

{% tabs %}
{% tab title="Core" %}
```bash
cd $NODE_HOME
nano poolMetaData.json
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
The extendedPoolMetaData.json file is used by adapools and others to scrape information like where to find your pool logo and social media links. Unlike the poolMetaData.json this files hash is not stored in your registration certificate and can be edited without having to rehash and resubmit pool.cert.
{% endhint %}

Add the following and customize to your metadata.

{% tabs %}
{% tab title="Core" %}
```text
{
"name": "Pool Name",
"description": "Pool description, no longer than 255 characters.",
"ticker": "AARCH",
"homepage": "https://example.com/",
"extended": "https://example.com/extendedPoolMetaData.json"
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli stake-pool metadata-hash \
  --pool-metadata-file poolMetaData.json > poolMetaDataHash.txt
```
{% endtab %}
{% endtabs %}

Copy poolMetaData.json to [https://pages.github.io](https://pages.github.io) or host it yourself along with your website.

{% hint style="info" %}
Here is my poolMetaData.json & extendedPoolMetaData.json as a reference and shameless links back to my site. 😰

[https://adamantium.online/poolMetaData.json](https://adamantium.online/poolMetaData.json)

[https://adamantium.online/extendedPoolMetaData.json​](https://adamantium.online/extendedPoolMetaData.json)
{% endhint %}

{% tabs %}
{% tab title="Core" %}
```bash
minPoolCost=$(cat $NODE_HOME/params.json | jq -r .minPoolCost)
echo minPoolCost: ${minPoolCost}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
#### How to configure multiple relay nodes

**DNS based relays, 1 entry per DNS record**

```bash
    --single-host-pool-relay r1.example.com \
    --pool-relay-port 3001 \
    --single-host-pool-relay r2.example.com \
    --pool-relay-port 3002 \
```

**IP based relays, 1 entry per IP address**

```bash
    --pool-relay-ipv4 <your first relay node public IP address> \
    --pool-relay-port 3001 \
    --pool-relay-ipv4 <your second relay node public IP address> \
    --pool-relay-port 3002 \
```
{% endhint %}

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli stake-pool registration-certificate \
  --cold-verification-key-file $HOME/cold-keys/node.vkey \
  --vrf-verification-key-file vrf.vkey \
  --pool-pledge 10000000000 \
  --pool-cost 340000000 \
  --pool-margin 0.01 \
  --pool-reward-account-verification-key-file stake.vkey \
  --pool-owner-stake-verification-key-file stake.vkey \
  --mainnet \
  --single-host-pool-relay <r1.example.com> \
  --pool-relay-port 3000 \
  --metadata-url <https://example.com/poolMetaData.json>
  --metadata-hash $(cat poolMetaDataHash.txt) \
  --out-file pool.cert
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli stake-address delegation-certificate \
  --stake-verification-key-file stake.vkey \
  --cold-verification-key-file $HOME/cold-keys/node.vkey \
  --out-file deleg.cert
```
{% endtab %}
{% endtabs %}

Retrieve your stake pool id

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli stake-pool id --cold-verification-key-file $HOME/cold-keys/node.vkey --output-format hex > stakePoolId.txt
cat stakePoolId.txt
```
{% endtab %}
{% endtabs %}

Move **pool.cert**, **deleg.cert** & **stakePoolId.txt** to your online core machine.

{% tabs %}
{% tab title="Core" %}
```bash
slotNo=$(cardano-cli query tip --mainnet | jq -r '.slot')
echo slotNo: ${slotNo}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli query utxo \
  --address $(cat payment.addr) \
  --mainnet > fullUtxo.out

tail -n +3 fullUtxo.out | sort -k3 -nr > balance.out
cat balance.out    
tx_in=""
total_balance=0

while read -r utxo; do
  in_addr=$(awk '{ print $1 }' <<< "${utxo}")
  idx=$(awk '{ print $2 }' <<< "${utxo}")
  utxo_balance=$(awk '{ print $3 }' <<< "${utxo}")
  total_balance=$((${total_balance}+${utxo_balance}))
  echo TxHash: ${in_addr}#${idx}
  echo ADA: ${utxo_balance}
  tx_in="${tx_in} --tx-in ${in_addr}#${idx}"
done < balance.out
txcnt=$(cat balance.out | wc -l)
echo Total ADA balance: ${total_balance}
echo Number of UTXOs: ${txcnt}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Core" %}
```bash
stakePoolDeposit=$(cat $NODE_HOME/params.json | jq -r '.stakePoolDeposit')
echo stakePoolDeposit: ${stakePoolDeposit}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli transaction build-raw \
  ${tx_in} \
  --tx-out $(cat payment.addr)+$(( ${total_balance} - ${stakePoolDeposit}))  \
  --invalid-hereafter $(( ${slotNo} + 10000)) \
  --fee 0 \
  --certificate-file pool.cert \
  --certificate-file deleg.cert \
  --out-file tx.tmp
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Core" %}
```bash
fee=$(cardano-cli transaction calculate-min-fee \
  --tx-body-file tx.tmp \
  --tx-in-count ${txcnt} \
  --tx-out-count 1 \
  --mainnet \
  --witness-count 3 \
  --byron-witness-count 0 \
  --protocol-params-file params.json | awk '{ print $1 }')
  echo fee: ${fee}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Core" %}
```bash
txOut=$((${total_balance}-${stakePoolDeposit}-${fee}))
echo txOut: ${txOut}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli transaction build-raw \
  ${tx_in} \
  --tx-out $(cat payment.addr)+${txOut} \
  --invalid-hereafter $(( ${slotNo} + 10000)) \
  --fee ${fee} \
  --certificate-file pool.cert \
  --certificate-file deleg.cert \
  --out-file tx.raw
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Cold Offline" %}
```bash
cardano-cli transaction sign \
  --tx-body-file tx.raw \
  --signing-key-file payment.skey \
  --signing-key-file $HOME/cold-keys/node.skey \
  --signing-key-file stake.skey \
  --mainnet \
  --out-file tx.signed
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Core" %}
```bash
cardano-cli transaction submit \
  --tx-file tx.signed \
  --mainnet
```
{% endtab %}
{% endtabs %}
