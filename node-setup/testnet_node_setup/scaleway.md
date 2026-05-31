---
description: Guide for running the EBLA Node with Docker on Scaleway
---

# Scaleway

## 1. Register to Scaleway

You can use the following link:

[https://www.scaleway.com/en/docs/create-your-scaleway-account/](https://www.scaleway.com/en/docs/create-your-scaleway-account/)

_NOTE: We will use default project to create EBLA nodes._

## 2. Create API keys on default project

You can create api keys on this page:

[Scaleway Credentials Page](https://console.scaleway.com/project/credentials)

The document for generating api keys: [https://www.scaleway.com/en/docs/generate-api-keys](https://www.scaleway.com/en/docs/generate-api-keys)

Make a note of that key. We will use it in the next step.

## 3. Creating a EBLA Node

{% hint style="danger" %}
If you are running this on Windows you will first need to install Windows Subsystem for Linux. You can use [this guide](https://docs.microsoft.com/en-us/windows/wsl/install-win10).
{% endhint %}

{% hint style="warning" %}
The script you fetch and run in the next step depends on the network you want to deploy your node to.   Please be sure to select between mainnet and testnet as desired.
{% endhint %}

{% tabs %}
{% tab title="Mainnet" %}
Download and run the node install script for the type of node you wish to deploy:

**Mainnet Full Node:**

<pre class="language-bash"><code class="lang-bash"><strong>bash -c "$(curl -fsSL https://raw.githubusercontent.com/EBLA-network/ebla-ops/master/scripts/one-click-Scaleway.sh)" mainnet
</strong></code></pre>

**Mainnet Light Node:**

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/EBLA-network/ebla-ops/master/scripts/one-click-Scaleway.sh)" mainnet light
```
{% endtab %}

{% tab title="Testnet" %}
Download and run the node install script for the type of node you wish to deploy:

**Testnet Full Node:**

<pre class="language-bash"><code class="lang-bash"><strong>bash -c "$(curl -fsSL https://raw.githubusercontent.com/EBLA-network/ebla-ops/master/scripts/one-click-Scaleway.sh)" testnet
</strong></code></pre>

**Testnet Light Node:**

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/EBLA-network/ebla-ops/master/scripts/one-click-Scaleway.sh)" testnet light
```
{% endtab %}
{% endtabs %}

Look

Now you should see a new droplet in your Scaleway account.

_NOTE: It takes a few minutes after the droplet starts to install the EBLA Node software._

## 4. Connect to your instance

Where is the SSH key?

* If you have added a SSH public key to your default project via Scaleway console page, the script will use it.
*   If you haven't set a public key, the script will generate a new SSH key. You can find the new public key and private key in this directory: `~/.ssh/`.

    ```
    ls -al .ssh/ | grep "ebla"
    ```

Next, you can login your instance.

## 5. Check the logs

You can use the following commands to check if the node is up and running:

```bash
sudo docker ps

sudo docker logs -f ebla_compose_node_1
```
