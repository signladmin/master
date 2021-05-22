---
description: Quick bootstrap a synced configured node in a hour!
---

# Pi-Node \(quick start\)

{% hint style="info" %}
It will take about 15 minutes to download the chain and another 45 to sync to the tip. You will not be able to do much until your node has synced with the tip of the block chain.

It can take anywhere from 5 to 50 minutes to sync after a reboot depending how the node was shut down or restarted. Check if process is running with htop. If it is use gLiveView.sh or go for walk. It will sync and the socket will be created.

It is best to just leave it running.
{% endhint %}

## Quick Start

1. Download and flash the [Pi-Node.img.gz](https://db.adamantium.online/Pi-Node.img.gz).
2. SSH into server.

   **ssh ada@&lt;pi-node private IPv4&gt;**  
   default credentials = **ada:lovelace**  
   Set new password and log back in.

3. Enter the pi-pool folder. **cd /home/ada/pi-pool**
4. Download database snapshot. **wget -r -np -nH -R "index.html\*" -e robots=off** [**https://db.adamantium.online/db/**](https://db.adamantium.online/db/)\*\*\*\*
5. Enable & start the cardano-service. **cardano-service enable** **cardano-service start**
6. Enable & start the cardano-monitor **cardano-monitor enable** **cardano-monitor start**
7. Confirm services are running. **cardano-service status** **cardano-monitor status**
8. gLiveView.sh. **cd $NODE\_HOME/scripts** **./gLiveView.sh**
9. Grafana. Enter your Node's IPv4 address in your browser. default credentials = **admin:admin** 

{% hint style="info" %}
The following guide builds out the image, use it as a reference and please feel free to ask for clarification in our Telegram channel. [https://t.me/armada\_alli](https://t.me/armada_alli)
{% endhint %}



