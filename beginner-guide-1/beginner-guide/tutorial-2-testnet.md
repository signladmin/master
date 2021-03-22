---
description: >-
  After completing the setup we are now ready to download the files needed to
  the testnet
---

# Connect a Relay Node to theTestnet\(Work-inProgress\)

{% hint style="danger" %}
**This tutorial is meant only to get a single node syncing to the Cardano ledger/blockchain! We have skipped certain steps to make this tutorial as easily as possible DO NOT USE this tutorial to form a Stake Pool. Please use our intermediate guides for creation of Stake Pools because we wont be using some resources needed and security measures are almost non-existent in this tutorial.**  
{% endhint %}

{% hint style="danger" %}
 **This tutorial is for only use with raspianOS-64bit and is soley for education purposes for creating a cardano-node and watch it sync to the blockchain** 
{% endhint %}

## _Summary_ 

1. Downloading the Binaries needed to build Cardano node relay
2. Download Configuration files from IOHK/Cardano-node
3. Edit the settings like Alessandro showed how
4. Run the basic passive relay node to connect to testnet
5. Monitor Relay Node with gLiveView  

{% hint style="success" %}
Please do not skip steps young Padawan  ![](../../.gitbook/assets/download-10-.jpeg) 
{% endhint %}

## Step 1: Download Files

* **We must first update our OS and install needed upgrades if available**

```
# We are using the sudo preffix to run commands as non-root-user  

$ sudo apt update
$ sudo apt upgrade -y

```

* **We can now reboot the pi and let the updates take effect by running this command in terminal**

```text
$ sudo reboot 
```



| Provided By | Link to Cardano Static Build  |
| :--- | :--- |
| [Moritz Angermann \[ZW3RK\]![](../../.gitbook/assets/git.jpeg)](https://github.com/angerman)  | [https://ci.zw3rk.com/build/719/download/1/aarch64-unknown-linux-musl-cardano-node-1.25.1.zip](https://ci.zw3rk.com/build/719/download/1/aarch64-unknown-linux-musl-cardano-node-1.25.1.zip) |

* Now we need to simply download the zip file above to our Pi's Home directory and then move it to the right location so we can call on it later to start the node.

```text
# First change to the home directory
$ cd ~/

# Now we can download the static build 
$ wget https://ci.zw3rk.com/build/719/download/1/aarch64-unknown-linux-musl-cardano-node-1.25.1.zip
```

{% hint style="info" %}
If you are unsure it the file downloaded or need the name of the folder/files we can use the linux "ls" command to list everything in our current working directory 
{% endhint %}

* Use "unzip" command on the downloaded zip file...

```text
$ unzip aarch64-unknown-linux-musl-cardano-node-1.25.1.zip
```

