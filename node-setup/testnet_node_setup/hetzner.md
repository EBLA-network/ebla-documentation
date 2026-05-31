---
description: Guide for running the EBLA Node with Docker on Hetzner
---

# Hetzner

## 1. Register to Hetzner

You can use the following link to get Euro 20 to test out Hetzner Cloud:

[https://hetzner.cloud/](https://hetzner.cloud/)

## 2. Create a Project API access token

First, create a project in the [Hetnzer Cloud Console](https://console.hetzner.cloud/) Then select the project and create an API access token in the Security menu under API Tokens tab.

Make a note of that key. We will use it in the next step.

## 3. Creating a EBLA Node

{% hint style="danger" %}
If you are running this on Windows you will first need to install Windows Subsystem for Linux. You can use [this guide](https://docs.microsoft.com/en-us/windows/wsl/install-win10).
{% endhint %}

Export your Hetzner Cloud access token to your environment so that the node creation script can access it without prompting you for it:

```bash
export HCLOUD_TOKEN=your_api_key
```

{% hint style="warning" %}
The script you fetch and run in the next step depends on the network you want to deploy your node to.   Please be sure to select between mainnet and testnet as desired.
{% endhint %}

{% tabs %}
{% tab title="Mainnet" %}
Download and run the node install script for the type of node you wish to deploy:

**Mainnet Full Node:**

<pre class="language-bash"><code class="lang-bash"><strong>bash -c "$(curl -fsSL https://raw.githubusercontent.com/EBLA-network/ebla-ops/master/scripts/one-click-Hetzner.sh)" mainnet
</strong></code></pre>

**Mainnet Light Node:**

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/EBLA-network/ebla-ops/master/scripts/one-click-Hetzner.sh)" mainnet light
```
{% endtab %}

{% tab title="Testnet" %}
Download and run the node install script for the type of node you wish to deploy:

**Testnet Full Node:**

<pre class="language-bash"><code class="lang-bash"><strong>bash -c "$(curl -fsSL https://raw.githubusercontent.com/EBLA-network/ebla-ops/master/scripts/one-click-AWS.sh)" testnet
</strong></code></pre>

**Testnet Light Node:**

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/EBLA-network/ebla-ops/master/scripts/one-click-AWS.sh)" testnet light
```
{% endtab %}
{% endtabs %}

Now you should see a new server in your Hetzner Cloud Console and you should receive an email with the login details.

_NOTE: It takes a few minutes after the droplet starts to install the EBLA Node software._

## 4. Check the logs

You can use the following commands inside the created server to check if the node is up and running:

```bash
sudo docker ps

sudo docker logs -f ebla_compose_node_1
```
