# Core server setup

Using Martin Lang's StakePool Operator Scripts to manage cardano-node. These scripts not only handle pool operations. They can mint NFT's and query the blockchain handling many complex transactions with ease.

https://github.com/gitmachtl/scripts

Please visit and review the configuration, scriptfiles syntax & filenames to better familiarize yourself with the workflow and capabilities of this awesome toolset.

## Online Installation

Clone the repo into your home directory. Create a folder to hold the scripts and add them to your PATH.

```bash
cd; git clone https://github.com/gitmachtl/scripts.git $HOME/stakepoolscripts
mkdir -p $HOME/stakepoolscripts/bin && cd $_
echo "export PATH=\"$PWD:\$PATH\"" >> $HOME/.adaenv
export PATH="$PWD:$PATH"; . $HOME/.adaenv
```

By now you should have chosen and synced your node on Testnet or Mainnet. There are two sets of scripts respectively. If you are on Testnet you can run a core with all the keys on it in Online mode. With Mainnet we set up an Online Core running a full node and an offline machine that runs the same version of cardano-cli as the online machine uses. The Cold machine does not run cardano-node. It is offline.

This offline or cold machine protects the nodes cold keys and the owners pledge keys. A json file with built transactions are transfered to the cold machine for signing and then moved back to the core for submission. Preventing Node and Wallet keys from ever being on a machine connected to the internet.

```bash
cd $HOME/stakepoolscripts
git fetch origin && git reset --hard origin/master
```

Confirm these scripts are in your PATH and optionally check the integrity of the scripts with git.

Git has checksums baked right in.

Everything in Git is an object. Every object has an ID. The IDs are a checksum of the content and connections. If the content or connections change, the ID is no longer valid.

For example, a commit ID is basically a checksum of...

- The contents and permissions of all files (git calls them "blobs") at the point of that commit (which have their own IDs).
- The fields of the commit like author, date, log message, etc...
- The commit IDs of the parent commits.

```bash
git fsck --full
git status
```

You will see that our new bin folder is untracked. everything else should be up to date.

Copy the latest versions of the scripts into the bin folder.

```bash
rsync -av $HOME/stakepoolscripts/cardano/${NODE_CONFIG}/* $HOME/stakepoolscripts/bin
```

Martin hosts checksums but I have found he updates them faster than his webserver can serve them resulting in those newest updated files failing. Checking your local version against github before moving them to your cold machine is a good practice to get into. All of this is dependent on the replies you get from your DNS server, security wise.

I am in the habit of pulling updates, running a check against the repo and gathering copies of any binaries needed for USB transfer to the cold machine. These would include the latest $HOME/stakepoolscripts/bin folder and a copy of the cardano-cli binary in $HOME/.local/bin

### Common.inc

Create a variable for testnet magic, Byron to Shelley epoch value and a variable to determine whether we are on mainnet or testnet. If on testnet we apend the magic value onto out CONFIG_NET variable.

```bash
echo export MAGIC=$(cat ${NODE_FILES}/${NODE_CONFIG}-shelley-genesis.json | jq -r '.networkMagic') >> ${HOME}/.adaenv; . ${HOME}/.adaenv
if [[ ${NODE_CONFIG} = 'testnet' ]]; then echo export BYRON_SHELLEY_EPOCHS=74; else echo export BYRON_SHELLEY_EPOCHS=208; fi >> ${HOME}/.adaenv
if [[ ${NODE_CONFIG} = 'testnet' ]]; then echo export CONFIG_NET='testnet-magic\ "${MAGIC}"'; else echo export CONFIG_NET=mainnet; fi >> ${HOME}/.adaenv; . ${HOME}/.adaenv
```

Copy the top portion of the 00_common.sh file into a new file named common.inc. This will hold the variable paths needed to connect these scripts to our running node.

```bash
cd stakepoolscripts/bin/
sed -n '1,69p' 00_common.sh >> common.inc
```

And edit the lines needed to get up and running. I would look in this file beforehand to get an idea of what I am changing from defaults.

```bash
sed -i common.inc \
    -e 's#socket="db-mainnet/node.socket"#socket="${NODE_HOME}/db/socket"#' \
    -e 's#genesisfile="configuration-mainnet/mainnet-shelley-genesis.json"#genesisfile="'${NODE_FILES}'/'${NODE_CONFIG}'-shelley-genesis.json"#' \
    -e 's#genesisfile_byron="configuration-mainnet/mainnet-byron-genesis.json"#genesisfile_byron="'${NODE_FILES}'/'${NODE_CONFIG}'-byron-genesis.json"#' \
    -e 's#cardanocli="./cardano-cli"#cardanocli="cardano-cli"#' \
    -e 's#cardanonode="./cardano-node"#cardanonode="cardano-node"#' \
    -e 's#bech32_bin="./bech32"#bech32_bin="bech32"#' \
    -e 's#offlineFile="./offlineTransfer.json"#offlineFile="${HOME}/usb-transfer/offlineTransfer.json"#' \
    -e 's#byronToShelleyEpochs=208#byronToShelleyEpochs='${BYRON_SHELLEY_EPOCHS}'#' \
    -e 's#magicparam="--mainnet"#magicparam="--${CONFIG_NET}"#' \
    -e 's#addrformat="--mainnet"#addrformat="--${CONFIG_NET}"#'
```

This gets us what we need to continue. Have a look in the file for more options and edits you may need to make depending on your task(like catalyst voting, minting tokens and setting up a hardware wallet).

### Test installation

Let's test we have these scripts in our PATH and test they are working.

```bash
cd; 00_common.sh
```

Should see this on testnet or similiar for mainnet. If somethine went wrong Matin presents you with a nice mushroom cloud ascii drawing and a hint as to what failed. If you are not synced to the tip of the chain it will warn you that the socket does not exist!

**You will need to have a fully synced node to continue.**

```bash
Version-Info: cli 1.33.0 / node 1.33.0      Scripts-Mode: online        Testnet-Magic: 1097911063
```

# Configure your USB transfer stick

Grab a USB stick and set it up with an ext4 partition owned by $USER that we can transfer between our two machines.

Create the mountpoint & set default ACL for files and folders with umask.

```bash
cd; mkdir $HOME/usb-transfer; umask 022 $HOME/usb-transfer
```

Attach the external drive into one of USB2 ports and list all drives with fdisk. Some drive adaptors eat a lot of power and you do not want to risk another USB device eating too much power on the USB3 bus.

```bash
sudo fdisk -l
```

Example output:

```bash
Disk /dev/sdb: 57.66 GiB, 61907927040 bytes, 120913920 sectors
Disk model: Cruzer
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: EECA81B9-3683-4A59-BC63-02EEDC04FD21
```

In my case it is /dev/sdb. Yours may be /dev/sdc, /dev/sdd or so on. /dev/sda is usually the system drive. **<span style="color:red">Do not format your system drive by accident</span>**.

## Create an new GUID Partition Table (GPT)

**<span style="color:red">This will wipe the disk</span>**

```bash
sudo gdisk /dev/sdb
```

Type ? to list options

```
Command (? for help): ?
b   back up GPT data to a file
c   change a partition's name
d   delete a partition
i   show detailed information on a partition
l   list known partition types
n   add a new partition
o   create a new empty GUID partition table (GPT)
p   print the partition table
q   quit without saving changes
r   recovery and transformation options (experts only)
s   sort partitions
t   change a partition's type code
v   verify disk
w   write table to disk and exit
x   extra functionality (experts only)
?   print this menu
```

1. Enter **o** for new GPT and agree to wipe the drive.
2. Enter **n** to add a new partition and accept defaults to create a partition that spans the entire disk.
3. Enter **w** and 'Y' to write changes to disk and exit gdisk.

Your new partition can be found at /dev/sdb1, the first partition on sdb.

### Optionaly Check the drive for bad blocks (takes a couple of hours)

```bash
badblocks -c 10240 -s -w -t random -v /dev/sdb
```

## Format the partition as ext4

We still need to create a new ext4 filesystem on the partition.

```bash
sudo mkfs.ext4 /dev/sdb1
```

Example output:

```bash
mke2fs 1.46.3 (27-Jul-2021)
Creating filesystem with 15113979 4k blocks and 3784704 inodes
Filesystem UUID: c2a8f8c7-3e7a-40f2-8dac-c2b16ab07f37
Superblock backups stored on blocks:
    32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
    4096000, 7962624, 11239424

Allocating group tables: done
Writing inode tables: done
Creating journal (65536 blocks): done
Writing superblocks and filesystem accounting information: done
```

## Mount the drive at boot

We want this drive to always be available to our backup job. Since it will be holding sensitive data we will mount it in a way where only root and the user cardano-node runs as can access.

Run blkid and pipe it through awk to get the UUID of the filesystem we just created.

```bash
sudo blkid /dev/sdb1 | awk -F'"' '{print $2}'
```

Example output:

```bash
c2a8f8c7-3e7a-40f2-8dac-c2b16ab07f37
```

For me the UUID=c2a8f8c7-3e7a-40f2-8dac-c2b16ab07f37

Add a mount entry to the bottom of fstab adding your UUID and the full system path to you backup folder.

```bash
sudo nano /etc/fstab
```

```bash
UUID=c2a8f8c7-3e7a-40f2-8dac-c2b16ab07f37 /home/ada/usb-transfer auto nosuid,nodev,nofail 0 1
```

> nofail allows the server to boot if the drive is not inserted.

### Test your drive mounts

Mount the drive and confirm it mounted by locating the lost+found folder. If it is not present then your drive is not mounted.

```bash
sudo mount usb-transfer; ls $HOME/usb-transfer
```

Take ownership of the filesystem.

```bash
sudo chown -R $USER:$USER $HOME/usb-transfer
```
Now your stick will automount if it is left in the core machine and is rebooted. We will repeat these steps on the offline cold machine. When you plug it into a running server just issue the same mount/check command.

```bash
sudo mount usb-transfer; ls $HOME/usb-transfer
```
We already set the location of our USB mount in the SPOS common.inc file. We can again test our installation by creating a new offlineTransfer.json file which we need for continuing in offline mode on our Cold machine.

```bash
01_workOffline.sh new
```
Lets copy the files we need on the offline machine to the USB stick for transfer.

Create an rsync-exclude.txt file so we can rip through and grab everything we need and skip the rest.

```bash
cd; nano exclude-list.txt
```
Add the following.

```bash
.bash_history
.bash_logout
.bashrc
.cache
.config
.local/bin/cardano-node
.local/bin/cardano-service
.profile
.selected_editor
.ssh
.sudo_as_admin_successful
.wget-hsts
git
tmp
pi-pool/db
pi-pool/scripts
pi-pool/logs
usb-transfer
exclude-list.txt
```
If your drive is over 20gb you can remove the pi-pool/db entry but you should shut down cardano-node first. This will give you a copy of the chain that can be transfered to other machines to save first sync time.

Backup the files and folders to the USB stick.

```bash
rsync -av --exclude-from="exclude-list.txt" /home/ada /home/ada/usb-transfer
```
```bash
cd; sudo umount usb-transfer
```

# Set up your cold machine.

For the cold machine I would use 64bit Raspberry Pi OS(Raspbian) with a desktop on a Raspi-400. It allows for multiple windows, copy and paste and another way to see your keys. It will help you start figuring out the different keys and what they are used for.

add link to cold page


