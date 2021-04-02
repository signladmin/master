# Topology Updater

## Updating your topology files

### Create topologyUpdater.sh

```bash
nano $HOME/.local/bin/topologyUpdater.sh
```

{% hint style="warning" %}

{% endhint %}

```bash
#!/bin/bash
# shellcheck disable=SC2086,SC2034
# shellcheck source=/dev/null

PARENT="$(dirname $0)"
[[ -f "${PARENT}"/env ]] && . "${PARENT}"/env offline

######################################
# User Variables - Change as desired #
######################################

CNODE_HOSTNAME="CHANGE ME"  # (Optional) Must resolve to the IP you are requesting from
CNODE_VALENCY=1             # (Optional) for multi-IP hostnames
MAX_PEERS=15                # Maximum number of peers to return on successful fetch (note that a single peer may include valency of up to 3)
#CUSTOM_PEERS="None"        # *Additional* custom peers to (IP,port[,valency]) to add to your target topology.json
                            # eg: "10.0.0.1,3001|10.0.0.2,3002|relays.mydomain.com,3003,3"
#BATCH_AUTO_UPDATE=N        # Set to Y to automatically update the script if a new version is available without user interaction

######################################
# Do NOT modify code below           #
######################################

PARENT="$(dirname $0)"
[[ -f "${PARENT}"/.env_branch ]] && BRANCH="$(cat ${PARENT}/.env_branch)" || BRANCH="master"

usage() {
  cat <<-EOF
		Usage: $(basename "$0") [-b <branch name>] [-f] [-p]
		Topology Updater - Build topology with community pools
		-f    Disable fetch of a fresh topology file
		-p    Disable node alive push to Topology Updater API
		-b    Use alternate branch to check for updates - only for testing/development (Default: master)
		
		EOF
  exit 1
}

TU_FETCH='Y'
TU_PUSH='Y'

while getopts :fpb: opt; do
  case ${opt} in
    f ) TU_FETCH='N' ;;
    p ) TU_PUSH='N' ;;
    b ) BRANCH=${OPTARG}; echo "${BRANCH}" > "${PARENT}"/.env_branch ;;
    \? ) usage ;;
  esac
done
shift $((OPTIND -1))

[[ -z "${BATCH_AUTO_UPDATE}" ]] && BATCH_AUTO_UPDATE=N

# Check if update is available
URL="https://raw.githubusercontent.com/cardano-community/guild-operators/${BRANCH}/scripts/cnode-helper-scripts"
if curl -s -f -m 10 -o "${PARENT}"/topologyUpdater.sh.tmp ${URL}/topologyUpdater.sh && curl -s -f -m 10 -o "${PARENT}"/env.tmp ${URL}/env && [[ -f "${PARENT}"/topologyUpdater.sh.tmp && -f "${PARENT}"/env.tmp ]]; then
  if [[ -f "${PARENT}"/env ]]; then
    if [[ $(grep "_HOME=" "${PARENT}"/env) =~ ^#?([^[:space:]]+)_HOME ]]; then
      vname=$(tr '[:upper:]' '[:lower:]' <<< "${BASH_REMATCH[1]}")
    else
      echo -e "\nFailed to get cnode instance name from env file, aborting!\n"
      rm -f "${PARENT}"/topologyUpdater.sh.tmp
      rm -f "${PARENT}"/env.tmp
      exit 1
    fi
    sed -e "s@/opt/cardano/[c]node@/opt/cardano/${vname}@g" -e "s@[C]NODE_HOME@${BASH_REMATCH[1]}_HOME@g" -i "${PARENT}"/topologyUpdater.sh.tmp -i "${PARENT}"/env.tmp
    TU_TEMPL=$(awk '/^# Do NOT modify/,0' "${PARENT}"/topologyUpdater.sh)
    TU_TEMPL2=$(awk '/^# Do NOT modify/,0' "${PARENT}"/topologyUpdater.sh.tmp)
    ENV_TEMPL=$(awk '/^# Do NOT modify/,0' "${PARENT}"/env)
    ENV_TEMPL2=$(awk '/^# Do NOT modify/,0' "${PARENT}"/env.tmp)
    if [[ "$(echo ${TU_TEMPL} | sha256sum)" != "$(echo ${TU_TEMPL2} | sha256sum)" || "$(echo ${ENV_TEMPL} | sha256sum)" != "$(echo ${ENV_TEMPL2} | sha256sum)" ]]; then
      . "${PARENT}"/env offline &>/dev/null # source in offline mode and ignore errors to get some common functions, sourced at a later point again
      if [[ ${BATCH_AUTO_UPDATE} = 'Y' ]] || { [[ -t 1 ]] && getAnswer "\nA new version is available, do you want to upgrade?"; }; then
        cp "${PARENT}"/topologyUpdater.sh "${PARENT}/topologyUpdater.sh_bkp$(date +%s)"
        cp "${PARENT}"/env "${PARENT}/env_bkp$(date +%s)"
        TU_STATIC=$(awk '/#!/{x=1}/^# Do NOT modify/{exit} x' "${PARENT}"/topologyUpdater.sh)
        ENV_STATIC=$(awk '/#!/{x=1}/^# Do NOT modify/{exit} x' "${PARENT}"/env)
        printf '%s\n%s\n' "$TU_STATIC" "$TU_TEMPL2" > "${PARENT}"/topologyUpdater.sh.tmp
        printf '%s\n%s\n' "$ENV_STATIC" "$ENV_TEMPL2" > "${PARENT}"/env.tmp
        {
          mv -f "${PARENT}"/topologyUpdater.sh.tmp "${PARENT}"/topologyUpdater.sh && \
          mv -f "${PARENT}"/env.tmp "${PARENT}"/env && \
          chmod 755 "${PARENT}"/topologyUpdater.sh "${PARENT}"/env && \
          echo -e "\nUpdate applied successfully, please run topologyUpdater again!\n" && \
          exit 0; 
        } || {
          echo -e "\n${FG_RED}Update failed!${NC}\n\nplease install topologyUpdater.sh & env with prereqs.sh or manually download from GitHub" && \
          rm -f "${PARENT}"/topologyUpdater.sh.tmp && \
          rm -f "${PARENT}"/env.tmp && \
          exit 1;
        }
      fi
    fi
  else
    mv "${PARENT}"/env.tmp "${PARENT}"/env
    rm -f "${PARENT}"/topologyUpdater.sh.tmp
    echo -e "\nCommon env file downloaded: ${PARENT}/env"
    echo -e "This is a mandatory prerequisite, please set variables accordingly in User Variables section in the env file and restart topologyUpdater.sh\n"
    exit 0
  fi
fi
rm -f "${PARENT}"/topologyUpdater.sh.tmp
rm -f "${PARENT}"/env.tmp

if [[ ! -f "${PARENT}"/env ]]; then
  echo -e "\nCommon env file missing: ${PARENT}/env"
  echo -e "This is a mandatory prerequisite, please install with prereqs.sh or manually download from GitHub\n"
  exit 1
fi

# source common env variables in case it was updated and run in offline mode, even for TU_PUSH mode as this will be cought by failed EKG query
if ! . "${PARENT}"/env offline; then exit 1; fi

# Check if old style CUSTOM_PEERS with colon separator is used, if so convert to use commas
if [[ -n ${CUSTOM_PEERS} && ${CUSTOM_PEERS} != *","* ]]; then
  CUSTOM_PEERS=${CUSTOM_PEERS//[:]/,}
fi

if [[ ${TU_PUSH} = "Y" ]]; then
  fail_cnt=0
  while ! blockNo=$(curl -s -f -m ${EKG_TIMEOUT} -H 'Accept: application/json' "http://${EKG_HOST}:${EKG_PORT}/" 2>/dev/null | jq -er '.cardano.node.metrics.blockNum.int.val //0' ); do
    ((fail_cnt++))
    [[ ${fail_cnt} -eq 5 ]] && echo "5 consecutive EKG queries failed, aborting!"
    echo "(${fail_cnt}/5) Failed to grab blockNum from node EKG metrics, sleeping for 30s before retrying... (ctrl-c to exit)"
    sleep 30
  done
fi

if [[ -n ${CNODE_HOSTNAME} && "${CNODE_HOSTNAME}" != "CHANGE ME" ]]; then
  T_HOSTNAME="&hostname=${CNODE_HOSTNAME}"
else
  T_HOSTNAME=''
fi

if [[ ${TU_PUSH} = "Y" ]]; then
  if [[ ${IP_VERSION} = "4" || ${IP_VERSION} = "mix" ]]; then
    curl -s -f -4 "https://api.clio.one/htopology/v1/?port=${CNODE_PORT}&blockNo=${blockNo}&valency=${CNODE_VALENCY}&magic=${NWMAGIC}${T_HOSTNAME}" | tee -a "${LOG_DIR}"/topologyUpdater_lastresult.json
  fi
  if [[ ${IP_VERSION} = "6" || ${IP_VERSION} = "mix" ]]; then
    curl -s -f -6 "https://api.clio.one/htopology/v1/?port=${CNODE_PORT}&blockNo=${blockNo}&valency=${CNODE_VALENCY}&magic=${NWMAGIC}${T_HOSTNAME}" | tee -a "${LOG_DIR}"/topologyUpdater_lastresult.json
  fi
fi
if [[ ${TU_FETCH} = "Y" ]]; then
  if [[ ${IP_VERSION} = "4" || ${IP_VERSION} = "mix" ]]; then
    curl -s -f -4 -o "${TOPOLOGY}".tmp "https://api.clio.one/htopology/v1/fetch/?max=${MAX_PEERS}&magic=${NWMAGIC}&ipv=${IP_VERSION}"
  else
    curl -s -f -6 -o "${TOPOLOGY}".tmp "https://api.clio.one/htopology/v1/fetch/?max=${MAX_PEERS}&magic=${NWMAGIC}&ipv=${IP_VERSION}"
  fi
  if [[ -n "${CUSTOM_PEERS}" ]]; then
    topo="$(cat "${TOPOLOGY}".tmp)"
    IFS='|' read -ra cpeers <<< "${CUSTOM_PEERS}"
    for cpeer in "${cpeers[@]}"; do
      IFS=',' read -ra cpeer_attr <<< "${cpeer}"
      case ${#cpeer_attr[@]} in
        2) addr="${cpeer_attr[0]}"
           port=${cpeer_attr[1]}
           valency=1 ;;
        3) addr="${cpeer_attr[0]}"
           port=${cpeer_attr[1]}
           valency=${cpeer_attr[2]} ;;
        *) echo "ERROR: Invalid Custom Peer definition '${cpeer}'. Please double check CUSTOM_PEERS definition"
           exit 1 ;;
      esac
      if ! isValidIPv4 "${addr}" || ! isValidIPv6 "${addr}"; then echo "ERROR: Invalid IPv4 or IPv6 address '${addr}'. Please double check CUSTOM_PEERS definition"; exit 1
      elif ! isNumber ${port}; then echo "ERROR: Invalid port number '${port}'. Please double check CUSTOM_PEERS definition"; exit 1
      elif ! isNumber ${valency}; then echo "ERROR: Invalid valency number '${valency}'. Please double check CUSTOM_PEERS definition"; exit 1; fi
      topo=$(jq '.Producers += [{"addr": $addr, "port": $port|tonumber, "valency": $valency|tonumber}]' --arg addr "${addr}" --arg port ${port} --arg valency ${valency} <<< "${topo}")
    done
    echo "${topo}" | jq -r . >/dev/null 2>&1 && echo "${topo}" > "${TOPOLOGY}".tmp
  fi
  mv "${TOPOLOGY}".tmp "${TOPOLOGY}"
fi
exit 0
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

