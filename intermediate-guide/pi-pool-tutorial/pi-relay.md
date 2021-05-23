---
description: Pi-Node to Pi-Relay
---

# Pi-Relay

To turn Pi-Node into a active relay we have to.

1. Configure hostname.
2. Configure static IP.
3. Configure port for cardano-service.
4. Configure port forwarding on router.
5. Update port in env file.
6. Enable cron job.
7. Configure both topology scripts.
8. Wait for service onboarding\(4 hours\).
9. Pull in new list of peers.
10. Prune list of best peers.
11. Restart cardano-service

## Hostname

To set a fully qualified domain name \(FQDN\) for our relay edit /etc/hostname & /etc/hosts.

```text
sudo nano /etc/hostname
```

```text
r1.example.com
```

```text
sudo nano /etc/hosts
```

```bash
127.0.0.1 localhost
127.0.1.1 r1.example.com r1

# The following lines are desirable for IPv6 capable hosts
::1 ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts

## optional entries overide online dns settings
192.168.1.150 c1.example.com
#192.168.1.151 r1.example.com
192.168.1.152 r2.example.com

```

## Network

### Static IP

Open 50-cloud-init.yaml and replace the contents of the file with below.

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

```yaml
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.1.151/24
      gateway4: 192.168.1.1
      nameservers:
# Home router IP & QUAD9 https://quad9.net/
          addresses: [192.168.1.1, 9.9.9.9, 149.112.112.112]
```

Create a file named **99-disable-network-config.cfg** to disable cloud -init.

```bash
sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
```

Add the following, save and exit.

```bash
network: {config: disabled}
```

Apply your changes.

```text
sudo netplan apply
```

## Configure service port

Open the cardano service file and change the port it listens on.

```bash
nano /home/ada/.local/bin/cardano-service
```

Save and exit. **ctrl+x then y**.

```bash
#!/bin/bash
DIRECTORY=/home/ada/pi-pool
FILES=/home/ada/pi-pool/files
PORT=3001
HOSTADDR=0.0.0.0
TOPOLOGY=${FILES}/mainnet-topology.json
DB_PATH=${DIRECTORY}/db
SOCKET_PATH=${DIRECTORY}/db/socket
CONFIG=${FILES}/mainnet-config.json
## +RTS -N4 -RTS = Multicore(4)
cardano-node run \
  --topology ${TOPOLOGY} \
  --database-path ${DB_PATH} \
  --socket-path ${SOCKET_PATH} \
  --host-addr ${HOSTADDR} \
  --port ${PORT} \
  --config ${CONFIG}

```

```bash
cardano-service enable
```

Enable cardano-service at boot.

Reboot.

## Forward port on router

{% hint style="danger" %}
Do not forward a port to your Core machine it only connects to the relay\(s\) on your LAN
{% endhint %}

Log into your router and forward port 3001 to your relay nodes LAN IPv4 address port 3001. Second relay forward port 3002 to LAN IPv4 address for relay 2 to port 3002.

## Topology Updater

```bash
cd $NODE_HOME/scripts
```

Configure the script to match your environment.

{% hint style="warning" %}
If you are using IPv4 leave CNODE\_HOSTNAME the way it is. The service will pick up your public IP address on it's own. I repeat only change the port to 3001. For DNS change only the variable value. Do not edit "CHANGE ME" further down in the file.
{% endhint %}

{% code title="/home/ada/pi-pool/scripts/topologyUpdater.sh" %}
```bash
nano topologyUpdater.sh
```
{% endcode %}

Run the updater once to confirm it is working.

{% code title="/home/ada/pi-pool/scripts/topologyUpdater.sh" %}
```bash
./topologyUpdater.sh
```
{% endcode %}

Should look similar to this.

> `{ "resultcode": "201", "datetime":"2021-05-20 10:13:40", "clientIp": "1.2.3.4", "iptype": 4, "msg": "nice to meet you" }`

Enable the cron job by removing the \# character from crontab.

```bash
crontab -e
```

```bash
33 * * * * /home/ada/pi-pool/scripts/topologyUpdater.sh
```

Save and exit.

### Pull in your list of relays

Wait four hours or so and run the relay-topology\_pull.sh

Open relay-topology\_pull.sh and configure it for your environment.

{% code title="/home/ada/pi-pool/scripts/relay-topology\_pull.sh" %}
```bash
#!/bin/bash
BLOCKPRODUCING_IP=<core nodes private IPv4 address>
BLOCKPRODUCING_PORT=3000
curl -4 -s -o /home/ada/pi-pool/files/mainnet-topology.json "https://api.clio.one/htopology/v1/fetch/?max=15&customPeers=${BLOCKPRODUCING_IP}:${BLOCKPRODUCING_PORT}:1|relays-new.cardano-mainnet.iohk.io:3001:2"
```
{% endcode %}

Save and exit.

After four hours of on boarding your information will start to be available to other relays in the network. topologyUpdater.sh will create a list in $NODE\_HOME/logs. relay-topology\_pull.sh will add that list to your mainnet-topology file.

{% code title="/home/ada/pi-pool/scripts/relay-topology\_pull.sh" %}
```bash
./relay-topology_pull.sh
```
{% endcode %}

