# Cardano Submit API Tutorial

## Why this Guide?

- This guide is intended to show Raspberry-pi/ARM users how to use the Cardano Submit API with your own running Cardano node to watch for successful transaction submissions while using Nami wallet.

## Download and Install Cardano Submit API

Download the latest version of the Cardano node, cli, and tx-submit-api from the [Armada Alliance Github repository](https://github.com/armada-alliance/cardano-node-binaries).

```bash
wget -O 1_33_1.zip https://github.com/armada-alliance/cardano-node-binaries/blob/main/static-binaries/1_33_1.zip?raw=true
```
Unzip the contents of the zip file.

```bash
unzip 1_33_1.zip -d cardano-node-1.33.1
```
```bash
mv cardano-node-1.33.1/cardano-node/cardano-submit-api ~/.local/bin/
```

## Make a simple bash script to run the Cardano Submit API

You can use whatever text editor you would like and feel free to change the file name if you would like.

```bash
nano ~/.local/bin/tx-submit-service
```
Next, copy and paste the following into the file:

```bash
#!/bin/bash

cardano-submit-api \
  --socket-path /home/ada/pi-pool/db/socket \
  --port 8090 \
  --config /home/ada/pi-pool/files/tx-submit-mainnet-config.yaml \
  --listen-address 0.0.0.0 \
  --mainnet
```
**Before you save and exit you need to make sure you have entered the correct full-path to your Cardano node's `socket` and `tx-submit-mainnet-config.yaml file` because it will be different.**

Save and exit. Now we need to give permissions to the file so that it can be executed.

```bash
chmod +x ~/.local/bin/tx-submit-service
```


## Get the tx-submit-mainnet-config.yaml file from IOG's github repository

```bash
cd ~/pi-pool/files && wget -O tx-submit-mainnet-config.yaml https://raw.githubusercontent.com/input-output-hk/cardano-node/master/cardano-submit-api/config/tx-submit-mainnet-config.yaml
```

## Test the Cardano Submit API

Create a tmux session for the Cardano Submit API service.
```bash
tmux new-session -d -t cardano-submit-api && tmux attach -t cardano-submit-api
```
Run the Cardano Submit API service.
```bash
~/.local/bin/tx-submit-service
```
You should see the following output in your terminal:
> ![](https://raw.githubusercontent.com/armada-alliance/assets/gh-pages/cardano-submit-api.png)


## Connect the Cardano Submit API with Nami Wallet

Now you just need to connect the Cardano Submit API with Nami Wallet. Open your browser with your Nami wallet navigate to settings, select network, switch on custom node, then enter in `http://localhost:8090/api/submit/tx`.

***If you are using a local network node (i.e. a node running at home in your local network) then you need to enter `http://x.x.x.x:8090/api/submit/tx` and replace the `x.x.x.x` with the IP address of your local network node.***

[![How to Connect Cardano Node with Nami Wallet](https://res.cloudinary.com/marcomontalbano/image/upload/v1643225455/video_to_markdown/images/youtube--23SDU4dcJr0-c05b58ac6eb4c4700831b2b3070cd403.jpg)](https://youtu.be/23SDU4dcJr0 "How to Connect Cardano Node with Nami Wallet")


## Test the Cardano Submit API with Nami Wallet

Now that that is setup, let's just test this by sending some ada to ourselves using Nami Wallet. Once you get the tx submitted success notification from Nami wallet, head back to the tmux session you created earlier that is running the cardano-submit-api and look to see if your transaction was submitted. It will output the following log message:
> ![](https://raw.githubusercontent.com/armada-alliance/assets/gh-pages/tx-submit-success.png)