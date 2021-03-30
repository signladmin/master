---
description: >-
  After completing the Raspberry Pi setup we are now ready to download the files
  needed for the testnet.
---

# Setup a relay node on Testnet

{% hint style="danger" %}
**This tutorial is meant to get a single node syncing to the Cardano blockchain! We have skipped certain steps and security in order to make this tutorial as easy as possible - DO NOT USE this tutorial to form a mainnet stake pool. Please use our** [**intermediate guides**](../../intermediate-guide/pi-pool-tutorial/) **for the mainnet.**    
{% endhint %}

{% hint style="danger" %}
 **This tutorial is for only use with Raspberry Pi OS 64bit and is solely for educational purposes to get a cardano-node syncing to the blockchain.** 
{% endhint %}

## Summary 

1. Environment Setup
2. Downloading the binaries needed to build a Cardano node relay
3. Download configuration files from IOHK/Cardano-node
4. Edit the config settings 
5. Download a db snapshot to speed up the sync process
6. Run the basic passive relay node to connect to the testnet
7. Monitor the relay node with gLiveView  

{% hint style="success" %}
Please do not skip steps young Padawan  ![](../../.gitbook/assets/download-10-.jpeg) 
{% endhint %}

## Setting up our environment

* We must first update our OS and install needed upgrades if available.

{% hint style="info" %}
It is highly recommended to update the operating system every time you boot up and log in to your **Raspberry Pi** to prevent security vulnerabilities.
{% endhint %}

\*\*\*\*

```
# We are using the sudo prefix to run commands as non-root-user  

sudo apt update
sudo apt upgrade -y

```

* We can now reboot the Pi and let the updates take effect by running this command in a terminal.

```text
sudo reboot 
```

* Next, we need to continue with a few more tasks to make sure our  is ready.

```bash

```

| Provided By | Link to Cardano Static Build  |
| :--- | :--- |
| [Moritz Angermann \[ZW3RK\]![](../../.gitbook/assets/git.jpeg)](https://github.com/angerman)  | [https://ci.zw3rk.com/build/1753/download/1/aarch64-unknown-linux-musl-cardano-node-1.26.1.zip](https://ci.zw3rk.com/build/1753/download/1/aarch64-unknown-linux-musl-cardano-node-1.26.1.zip) |

* A static build is one in which all dependencies are included in the final build result which allows you to immediately run it on a compatible system without having to deal with building anything.
* Now we need to simply download the zip file above to our Pi's home directory and then move it to the right location so we can call on it later to start the node.

```text
# First change to the home directory
cd ~/

# Now we can download the static build 
wget https://ci.zw3rk.com/build/1753/download/1/aarch64-unknown-linux-musl-cardano-node-1.26.1.zip
```

{% hint style="info" %}
If you are unsure if the file downloaded or need the name of the folder/files, we can use the linux "ls" command to list everything in our current working directory.
{% endhint %}

* Use "unzip" command on the downloaded zip file.

```text
unzip aarch64-unknown-linux-musl-cardano-node-1.25.1.zip
```

* Next, we need to make sure the newly downloaded "cardano-node" folder and its contents are present.  

```text
ls
```

* You should see this in your home directory after running ls command:

![](../../.gitbook/assets/screen-shot-2021-03-21-at-7.29.03-pm%20%281%29.png)

* Now we need to move the cardano-node folder into our local binary directory.

```text
mv cardano-node ~/.local/bin
```

* Stay in the home directory and add a new directory/folder to download the Cardano config files and monitoring service we will be using.

{% hint style="success" %}
You can call the folder whatever you would like, but it is recommend to name it according to its use.
{% endhint %}

```text
mkdir testnet-relay
cd testnet-relay/
```

* Download the four Cardano node configuration files we need from the official [IOHK website](https://hydra.iohk.io/build/5822084/download/1/index.html) and or [documentation](https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles_AND_Connect.html). We will be using our CLI "wget" command to download the files.

```text
wget https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/testnet-config.json
wget https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/testnet-byron-genesis.json
wget https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/testnet-shelley-genesis.json
wget https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/testnet-topology.json

```

* Use the nano text editor to change a few things in our "testnet-config.json" file
* [ ] Change the **"TraceBlockFetchDecisions"** line from "**false**" to "**true**"
* [ ] Change the **"hasEKG"** to **12600**
* [ ] Change  the **"hasPrometheus"** address/port to 12700

```text
sudo nano testnet-config.json
```

{% hint style="warning" %}
Due to the size of the blockchain it may take anywhere from 25 to 40 hours to get your node fully synced.
{% endhint %}

## Download a snapshot of the blockchain to speed the sync process

{% hint style="danger" %}
Make sure you have not started your node before proceeding. Shut it down if so.
{% endhint %}

* Thankfully, we have been provided a "snapshot" of the database folder from the [\[OTG\] Star Forge Stake Pool](https://adamantium.online/). This will dramatically speed up the process.
* We will run the following commands and then begin downloading the snapshot.

```bash
# make sure you are in testnet-relay/ folder
cd testnet-relay
wget -r -np -nH -R "index.html*" -e robots=off https://db.adamantium.online/db/
```

{% hint style="info" %}
This download will take anywhere from 25 min to 2 hours depending on your internet speeds.
{% endhint %}

## Finish syncing to the blockchain 

* Now we can start the "passive" relay node to begin syncing to the blockchain.

```bash
cardano-node run \
   --topology testnet-topology.json \
   --database-path db \
   --socket-path db/socket \
   --host-addr 0.0.0.0 \
   --port 3000 \
   --config testnet-config.json 
```

## Setting up gLiveView to monitor the node during its syncing process

```bash
cd cardano-node/
curl -s -o gLiveView.sh https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/gLiveView.sh
curl -s -o env https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/env
chmod 755 gLiveView.sh
```

* Use nano to edit the env file and change the "**CNODE\_PORT**" to the port you set on your cardano-node, in our case let's change it to **3000.**

```bash
sudo nano env
```

* Finally, we can exit the nano editor and just run the gLiveView script.

```bash
cd cardano-node
touch db/clean
./gLiveView.sh
```

{% hint style="success" %}
If you want to monitor your Raspberry Pi performance you can use the following commands.
{% endhint %}

{% tabs %}
{% tab title="Get Cpu Temp" %}
```bash
vcgencmd measure_temp
```
{% endtab %}

{% tab title="Use htop for CPU and RAM Performance" %}
```bash
htop
```
{% endtab %}
{% endtabs %}

## References:

{% tabs %}
{% tab title="📚" %}
{% embed url="https://github.com/wcatz/pi-pool" %}



{% embed url="https://github.com/alessandrokonrad/Pi-Pool" %}

{% embed url="https://github.com/angerman" %}



{% embed url="https://docs.cardano.org/projects/cardano-node/en/latest/stake-pool-operations/getConfigFiles\_AND\_Connect.html" %}

{% embed url="https://cardano-community.github.io/guild-operators/\#/Scripts/gliveview" %}
{% endtab %}
{% endtabs %}



