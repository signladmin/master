---
description: Pi-Node to Pi-Relay
---

# Pi-Relay

To turn Pi-Node into a passive relay we have to.

1. Configure hostname.
2. Configure static IP.
3. Configure port for cardano-node.
4. Configure port forwarding on router.
5. Update port in env file.
6. Enable cron job.
7. Wait for service onboarding\(4 hours\).
8. Pull in new list of peers.
9. Prune list of best peers.

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

Reboot.

## Network

### Static IP

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

```bash
sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
```

```bash
network: {config: disabled}
```

```text
sudo netplan apply
```

## Configure service port

```bash
nano /home/ada/.local/bin/cardano-service
```

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
cardano-service restart
cardano-service status
```

## Forward port on router

{% hint style="danger" %}
Do not forward a port to your Core machine it only connects to the relay\(s\) on your LAN
{% endhint %}

Log into your router and forward port 3001 to your relay nodes LAN IPv4 address port 3001. Second relay forward port 3002 to LAN IPv4 address for relay 2 to port 3002.

## Topology Updater

## 

