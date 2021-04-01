# Topology Updater

## Updating your topology files

### Create topologyUpdater.sh

```bash
nano $HOME/.local/bin/topologyUpdater.sh
```

{% hint style="warning" %}
Change CNODE\_PORT to match the port your relay is running on.
{% endhint %}

```bash
#!/bin/bash
# shellcheck disable=SC2086,SC2034
 
USERNAME=ada
CNODE_PORT=3002 # must match relay port in /home/ada/.local/bin/cardano-service
CNODE_HOSTNAME=""  # optional. must resolve to the public IP you are requesting from
CNODE_BIN="/home/ada/.local/bin"
CNODE_HOME=/home/ada/pi-pool
CNODE_LOG_DIR="${CNODE_HOME}/logs"
GENESIS_JSON="${CNODE_HOME}/files/mainnet-shelley-genesis.json"
NETWORKID=$(jq -r .networkId $GENESIS_JSON)
CNODE_VALENCY=1   # optional for multi-IP hostnames
NWMAGIC=$(jq -r .networkMagic < $GENESIS_JSON)
[[ "${NETWORKID}" = "Mainnet" ]] && HASH_IDENTIFIER="--mainnet" || HASH_IDENTIFIER="--testnet-magic ${NWMAGIC}"
[[ "${NWMAGIC}" = "764824073" ]] && NETWORK_IDENTIFIER="--mainnet" || NETWORK_IDENTIFIER="--testnet-magic ${NWMAGIC}"
 
export PATH="${CNODE_BIN}:${PATH}"
export CARDANO_NODE_SOCKET_PATH="${CNODE_HOME}/db/socket"
 
blockNo=$(/home/ada/.local/bin/cardano-cli query tip ${NETWORK_IDENTIFIER} | jq -r .blockNo )
 
# Note:
# if you run your node in IPv4/IPv6 dual stack network configuration and want announced the
# IPv4 address only please add the -4 parameter to the curl command below  (curl -4 -s ...)
if [ "${CNODE_HOSTNAME}" != "CHANGE ME" ]; then
  T_HOSTNAME="&hostname=${CNODE_HOSTNAME}"
else
  T_HOSTNAME=''
fi

if [ ! -d ${CNODE_LOG_DIR} ]; then
  mkdir -p ${CNODE_LOG_DIR};
fi
 
curl -4 -s "https://api.clio.one/htopology/v1/?port=${CNODE_PORT}&blockNo=${blockNo}&valency=${CNODE_VALENCY}&magic=${NWMAGIC}${T_HOSTNAME}" | tee -a $CNODE_LOG_DIR/topologyUpdater_lastresult.json
```

Make it executable & run it once.

```bash
cd $HOME/.local/bin
chmod +x $HOME/.local/bin/topologyUpdater.sh
./topologyUpdater.sh
```

If successful it will print.

> `{ "resultcode": "201", "datetime":"2021-03-29 01:23:45", "clientIp": "1.2.3.4", "iptype": 4, "msg": "nice to meet you" }`

Schedule topology updater to run once every hour. Open cron and choose nano as your editor if it asks.

```bash
crontab -e
```

Paste the following in, save & exit.

```bash
33 * * * * ${HOME}/.local/bin/topologyUpdater.sh
```

{% hint style="success" %}
After four hours and four updates, your node IP will be registered in the topology fetch list.
{% endhint %}

### Relay topology pull

{% hint style="danger" %}
Complete this section after **four hours** when your relay node IP is properly registered.
{% endhint %}

Create `relay-topology_pull.sh` script which fetches your relay node peers and updates your topology file.

```bash
nano $HOME/.local/bin/relay-topology_pull.sh
```

Add following & add your block producing nodes ip. Save & exit.

```bash
#!/bin/bash
BLOCKPRODUCING_IP=<BLOCK PRODUCERS PRIVATE IP ADDRESS>
BLOCKPRODUCING_PORT=3000
curl -4 -s -o $NODE_FILES/${NODE_CONFIG}-topology.json "https://api.clio.one/htopology/v1/fetch/?max=15&customPeers=\${BLOCKPRODUCING_IP}:\${BLOCKPRODUCING_PORT}:1|relays-new.cardano-mainnet.iohk.io:3001:2"
```

Make executable.

```bash
chmod +x $HOME/.local/bin/relay-topology_pull.sh
```

After waiting at least 4 hours.

```bash
cd $HOME/.local/bin
./relay-topology_pull.sh
```

You should see your core node & IOHK's relays first followed by the peers that were propagated. The service does a good job of prioritizing peers. You can see the distance to the peer. In nano use **ctrl+k** to cut out whole lines of peers over 6,000 miles away. That is a good start.

{% hint style="info" %}
Don't forget to remove the comma from the last peer in your list or your relay won't start up.
{% endhint %}

```bash
cd $NODE_FILES
nano mainnet-topology.json
```

Restart the cardano-service for changes to take effect & check that it is running. Start up gLiveView and wait for it to sync up. Then press 'p' to show your connected peers.

```bash
cardano-service restart
cardano-service status
gLiveView.sh
```

{% hint style="warning" %}
Many operators disable icmp ping so you are bound to see some peers in as ---. Focus on out, the ones you are connecting to. This is my \#2 relay connected to ten relays. I will cut out anything over 100ms or so.
{% endhint %}

![](../../../.gitbook/assets/glive-relay-peers.png)

{% hint style="info" %}
I usually have a 3 terminals open. glive, my mainnet-topology.json file and a prompt. I use ping to resolve dns names in the topo file to locate them in glive. Changes go into affect after node restart.
{% endhint %}

Once you have the list the way you want it you can restart cardano-service. Periodically pull in new peers just be warned it will over write your mainnet-topology.json.

```bash
cardano-service restart
```

