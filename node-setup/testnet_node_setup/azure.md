# Azure

## 1. Create an Azure Account

Go to \[[http://portal.azure.com](http://portal.azure.com)] and sign up. You can use your Github credentials or create a new account.

Use \[[http://portal.azure.com](http://portal.azure.com)] to sign in. This is the hub where you can manage all your services, so it's a good idea to take a moment to familiarize yourself with the layout before proceeding.

## 2. Create a subscription

To run services in Azure, you must have a subscription. When you sign up for a new account, you get 12 months of free usage for certain services and $200 worth of credits for 30 days.

## 3. Create the node

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

<pre class="language-bash"><code class="lang-bash"><strong>bash -c "$(curl -fsSL https://raw.githubusercontent.com/EBLA-network/ebla-ops/master/scripts/one-click-Azure.sh)" mainnet
</strong></code></pre>

**Mainnet Light Node:**

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/EBLA-network/ebla-ops/master/scripts/one-click-Azure.sh)" mainnet light
```
{% endtab %}

{% tab title="Testnet" %}
Download and run the node install script for the type of node you wish to deploy:

**Testnet Full Node:**

<pre class="language-bash"><code class="lang-bash"><strong>bash -c "$(curl -fsSL https://raw.githubusercontent.com/EBLA-network/ebla-ops/master/scripts/one-click-Azure.sh)" testnet
</strong></code></pre>

**Testnet Light Node:**

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/EBLA-network/ebla-ops/master/scripts/one-click-Azure.sh)" testnet light
```
{% endtab %}
{% endtabs %}

You should get a Azure login window in your browser after running the script. Login to proceed.

The script will run for a while. When the script tells you to make a note of your nodes IP address, do so. The script will create SSH credentials for you to access your node.

Login via ssh

```bash
ssh <your-node-ip-address>
```

## 4. Check your node

```bash
sudo docker ps
```

## 5. Further tweaks

The node is running under root account. You might want to change that for easier access and security reasons. To do so, first, set the root password.

```bash
sudo passwd root
```

Now you can change into root. Make note of your current user before with whoami.

```bash
whoami
su
```

If you want to continue as is, all good. Save your root password and your done. If you'd rather move your node under your account, read on.

Change into root home and move your node to your personal home

```bash
cd /root
cd ./ebla-ops/ebla_compose
docker-compose down
cd /root
mv -R ebla-ops /home/<your-user-account>
exit
```

Once back in your user account, start the node

```bash
cd ~/ebla-ops/ebla_compose
sudo docker-compose up -d
```

You should be good to go!
