---
description: Guide for running the EBLA Node with Docker on Vultr
---

# Vultr

## 1. Register to Vultr

You can use the following link to get $100 to test out their platform:

[https://www.vultr.com/](https://www.vultr.com/)

## 2. Create a Personal access token

You can find a personal access token on this page:

[Vultr API KEY](https://my.vultr.com/settings/#settingsapi)

Make a note of that key. We will use it in the next step.

## 3. Creating a EBLA Node

{% hint style="danger" %}
If you are running this on Windows you will first need to install Windows Subsystem for Linux. You can use [this guide](https://docs.microsoft.com/en-us/windows/wsl/install-win10).
{% endhint %}

Export your Vultr access token to your environment so that the node creation script can access it without prompting you for it:

```bash
export VULTR_API_KEY=your_api_key
```

{% hint style="warning" %}
The script you fetch and run in the next step depends on the network you want to deploy your node to.   Please be sure to select between mainnet and testnet as desired.
{% endhint %}

{% tabs %}
{% tab title="Mainnet" %}
Download and run the node install script for the type of node you wish to deploy:

**Mainnet Full Node:**

<pre class="language-bash"><code class="lang-bash"><strong>bash -c "$(curl -fsSL https://raw.githubusercontent.com/EBLA-network/ebla-ops/master/scripts/one-click-Vultr.sh)" mainnet
</strong></code></pre>

**Mainnet Light Node:**

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/EBLA-network/ebla-ops/master/scripts/one-click-Vultr.sh)" mainnet light
```
{% endtab %}

{% tab title="Testnet" %}
Download and run the node install script for the type of node you wish to deploy:

**Testnet Full Node:**

<pre class="language-bash"><code class="lang-bash"><strong>bash -c "$(curl -fsSL https://raw.githubusercontent.com/EBLA-network/ebla-ops/master/scripts/one-click-Vultr.sh)" testnet
</strong></code></pre>

**Testnet Light Node:**

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/EBLA-network/ebla-ops/master/scripts/one-click-Vultr.sh)" testnet light
```
{% endtab %}
{% endtabs %}

Now you should see a new droplet in your Vultr account and you should receive an email with the login details.

_NOTE: It takes a few minutes after the droplet starts to install the EBLA Node software._

## 4. Check the logs

You can use the following commands to check if the node is up and running:

```bash
sudo docker ps

sudo docker logs -f ebla_compose_node_1
```
