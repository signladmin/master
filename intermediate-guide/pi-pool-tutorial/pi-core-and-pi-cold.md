---
description: Create operational keys & certificates. Register Stake Pool
---

# Pi-Core/Cold

## Generate core and cold key requirements

#### Key Evolving Signature keypair

KES keys expire after 90 days and you will have to create a new pair as part of pool operations before they expire.

{% tabs %}
{% tab title="Core" %}
```text
cd $NODE_HOME
cardano-cli node key-gen-KES \
  --verification-key-file kes.vkey \    
  --signing-key-file kes.skey​
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Cold Offline" %}
```text
mkdir $HOME/cold-keys
cardano-cli node key-gen \
  --cold-verification-key-file node.vkey \
  --cold-signing-key-file node.skey \
  --operational-certificate-issue-counter node.counter
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Plain Text" %}
```text

```
{% endtab %}
{% endtabs %}

