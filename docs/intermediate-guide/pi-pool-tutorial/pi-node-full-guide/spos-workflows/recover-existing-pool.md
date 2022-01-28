# Recover an Existing Pool

## Pool Recovery

This guide is for when you already have an existing pool and either need to recover from a crash or want to move to using the Stakepool Operator Scripts. The easiest scenario is when you have the node.skey from your offline machine or backup. 

These steps were created following Martin's guide and are meant as a clarification.

{% embed url="https://github.com/gitmachtl/scripts/tree/master/cardano/mainnet#import-your-existing-pool-from-cli-keys-or-tutorials" %}

### Prerequisites 

Before you start, you will need the following:

- Existing node.skey file from offline
- Running and synced relay and core nodes
- Flash drive formatted from Pi-Core step mounted in Pi-Cold step


### Using Online-Core

On your core machine, we need to pull the pool data from online. This is done using the script 0x_importHelper.sh and your pool name PoolID.  Using the example from Martin of a pool named "WAKANDA FOREVER" and a poolID of "abcdefc27be2bdde3ec11b9f696cf21fad39e49097be9b0193e6dcba" we would run the following command. 


```0x_importHelper.sh wakanda abcdefc27be2bdde3ec11b9f696cf21fad39e49097be9b0193e6dcba```

Online-core:

```bash
cd $HOME/stakepoolscripts/bin
./0x_importHelper.sh <poolname> <poolid>
```

Then move the newly created offlineTransfer.json file to the usb-transfer drive

```bash
cp offlineTransfer.json $HOME/usb-transfer
```
#### Cold-Offline

Moving over to your cold machine, insert the transfer flash drive and verify it is mounted. If you placed the offlineTransfer.json file correcetly you will be able to copy it to the stakepoolscripts/bin directory.

```bash
cd $HOME/stakepoolscripts/bin
cp $HOME/usb-transfer/offlineTransfer.json $HOME/stakepoolscripts/bin
```

You will also need to move your node.skey file from where ever you have it stored into the $HOME/stakepoolscripts/bin directory.

### Unpack Node Files

