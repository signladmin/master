---
description: >-
  After completing the setup we are now ready to download the files needed to
  the testnet
---

# Connect a Relay Node to theTestnet\(Work-inProgress\)

{% hint style="danger" %}
This tutorial is meant only to get a single node syncing to the Cardano ledger/blockchain! We have skipped certain steps to make this tutorial as easily as possible DO NOT USE this tutorial to form a Stake Pool. Please use our intermediate guides for creation of Stake Pools because we wont be using some resources needed and security measures are almost non-existent in this tutorial.  
{% endhint %}

## Summary 

1. Downloading the Binaries needed to build Cardano node relay
2. Download Configuration files from IOHK/Cardano-node
3. Edit the settings like Alessandro showed how
4. Run the basic passive relay node to connect to testnet
5. Monitor Relay Node  

## Step 1: Download Files

{% hint style="info" %}
Please do not skip steps young Padawan  ![](../../.gitbook/assets/download-10-.jpeg) 
{% endhint %}

* **We must first update our OS and install needed upgrades if available**

```
# We are using the sudo preffix to run commands as non-root-user  
$ sudo apt update
$ sudo apt upgrade -y

```

{% hint style="info" %}
 Super-powers are granted randomly so please submit an issue if you're not happy with yours.
{% endhint %}

Once you're strong enough, save the world:

{% code title="hello.sh" %}
```bash
# Ain't no code for that yet, sorry
echo 'You got to trust me on this, I saved the world'
```
{% endcode %}



