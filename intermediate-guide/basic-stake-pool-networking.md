---
description: >-
  This section will help with understanding the relay and block producer
  topology.
---

# Basic Stake Pool Networking \(Work-in-Progress\)

## Assumptions

For the sake of this tutorial we are assuming that the Raspberry Pi nodes you are running are in your home and connected to either your ISP's internet router or a separate switch connected to your ISP's internet router. Your nodes should have a firewall configured with as few ports open as possible and with your firewall rules as specific as possible. Furthermore, your ISP's internet router should also have firewall settings configured. If you are not familiar with them, leaving the firewall defaults from your ISP are generally okay.

If you have a block producer running, at a minimum it's firewall rules should have it's node port available only to your configured relay IPs and then the port you use for ssh. If you want to monitor your block producer metrics using Grafana, you'll also need to provide access to the Grafana port. Same thing if you want to monitor your relays.

We are not network experts here. This is only provided as a point of general understanding of how the node topology and network interact. For more advanced network discussions, feel free to use the NASEC discord channel.

## What are the topology files

Stuff

```
$ command line stuff
```

{% hint style="info" %}
 Bring focus to this informational stuff.
{% endhint %}

More stuff

{% code title="stuff.sh" %}
```bash
# Comment for the codez
echo 'the codez'
```
{% endcode %}



