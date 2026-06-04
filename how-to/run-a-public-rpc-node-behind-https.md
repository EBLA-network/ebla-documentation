---
description: >-
  Serve a public, HTTPS JSON-RPC endpoint for your dApp or community — backed by
  your own EBLA node and a Caddy reverse proxy that handles TLS and CORS.
---

# 🌐 Run a public RPC node behind HTTPS

The dev-team RPC endpoints ([rpc.testnet.eblanetwork.com](https://rpc.testnet.eblanetwork.com/),
and `rpc.eblanetwork.com` after mainnet launch) are provided as‑is and free of
charge. If you run a dApp, an exchange integration, or you just want a private
endpoint you control, the better long‑term answer is to run **your own RPC node**.

This guide stands up an RPC node and puts it behind **Caddy**, which terminates
**HTTPS** (auto Let's Encrypt certificates), adds the right **CORS** headers for
browser dApps, and keeps the node itself off the public internet.

> **Testnet today.** EBLA mainnet is not live yet (target launch ~July 2026), so
> the examples target **testnet**. Swap the network name and hostnames for mainnet
> once it's live.

## Architecture

```
 Browser / dApp ──HTTPS:443──▶  Caddy reverse proxy  ──HTTP:7777──▶  EBLA node
                                (TLS + CORS, public)       (bound to localhost only)
```

The node's RPC port (**7777**) is **never** exposed directly. Only Caddy listens on
the public internet (80/443).

---

## Step 1 — Deploy an RPC node

An RPC node is a normal EBLA node that **serves RPC but never produces blocks** — it
carries no validator wallet, so it has no stake. It peers, syncs, and answers
queries. The `docker-compose.light.yml` bundle is purpose-built for this.

```bash
mkdir -p ~/ebla-rpc && cd ~/ebla-rpc
wget https://raw.githubusercontent.com/EBLA-network/ebla-ops/ebla-stable/ebla_compose/docker-compose.light.yml
mv docker-compose.light.yml docker-compose.yml
```

**Bind the RPC port to localhost.** Edit `docker-compose.yml` so the node's RPC
maps to `127.0.0.1:7777` instead of `0.0.0.0`, so only the reverse proxy (same
host) can reach it:

```yaml
    ports:
      - "127.0.0.1:7777:7777"   # RPC — localhost only, fronted by Caddy
      - "10002:10002"           # p2p — must stay public (TCP)
      - "10002:10002/udp"       # p2p — must stay public (UDP)
```

> **Do NOT enable `--rpc.enable-test-rpc` on a public node.** The test API exposes
> operational methods (`get_node_status`, `get_account_address`, `send_coin_transaction`,
> …) that have no place on a public endpoint. The light bundle does not enable it
> by default — leave it that way. The standard `eth_*` / `ebla_*` / `net_*` methods
> are all your users need.

Start it and watch it sync:

```bash
docker compose up -d
docker compose logs -f
```

Wait until the logs report the node is synced before serving traffic. To skip the
full historical sync, you can [sync from a snapshot](../node-setup/syncing-from-snapshot.md).

> **Same-host peering note.** If this node shares a public IP with another EBLA node
> you run, it may not reach that neighbour via the public IP (hairpin NAT). Add the
> neighbour's **internal** address as a bootnode on the `eblad` line:
> `--boot-nodes-append <internal-ip>:10002/<enode-pubkey>`.

Confirm the node answers locally before exposing it:

```bash
curl -s -X POST -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  http://127.0.0.1:7777 | jq -r '.result'
```

---

## Step 2 — Point DNS at the server

Create an **A record** for your RPC hostname pointing at the server's public IP,
e.g.:

```
rpc.yourdomain.com   A   203.0.113.10
```

Verify it resolves from outside before requesting a certificate (Let's Encrypt
fails without it):

```bash
dig +short rpc.yourdomain.com      # → 203.0.113.10
```

Open **ports 80 and 443** to the internet in your firewall/security group. Keep
**7777 closed** to the outside.

---

## Step 3 — Install Caddy

```bash
sudo apt update
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' \
  | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' \
  | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update && sudo apt install -y caddy
caddy version
```

---

## Step 4 — Configure the reverse proxy

Write `/etc/caddy/Caddyfile`. This terminates TLS, proxies to the node on
`127.0.0.1:7777`, reflects CORS only for your own dApp origins, and answers
preflight requests:

```caddyfile
{
    email admin@yourdomain.com    # Let's Encrypt account / expiry warnings
}

rpc.yourdomain.com {
    reverse_proxy 127.0.0.1:7777

    # Reflect Allow-Origin only for your own front-end origins.
    # Server-to-server clients (backends, scripts) ignore CORS entirely.
    @cors header_regexp Origin ^https://(www\.)?yourdomain\.com$
    header @cors Access-Control-Allow-Origin "{http.request.header.Origin}"

    header {
        Access-Control-Allow-Methods "POST, OPTIONS"
        Access-Control-Allow-Headers "Content-Type"
        Vary Origin
        Strict-Transport-Security "max-age=31536000"
        X-Content-Type-Options "nosniff"
        -Server
    }

    # Answer CORS preflight requests
    @options method OPTIONS
    respond @options 204

    log {
        output file /var/log/caddy/rpc.log {
            roll_size 100mb
            roll_keep 5
        }
        format json
    }
}
```

> **Open vs. locked-down CORS.** Reflecting only your origin keeps random websites
> from driving your endpoint from visitors' browsers. If you genuinely want an
> open public RPC any site can call, replace the `@cors`/reflect lines with a
> single `Access-Control-Allow-Origin "*"` — but then definitely add rate
> limiting (below).

Validate and (re)start:

```bash
sudo mkdir -p /var/log/caddy && sudo chown -R caddy:caddy /var/log/caddy
caddy validate --config /etc/caddy/Caddyfile     # expect: "Valid configuration"
sudo systemctl reload caddy
```

Caddy fetches the certificate automatically on the first request (usually within
~60 s). Watch for it:

```bash
journalctl -u caddy --no-pager --since "2 minutes ago" \
  | grep -E 'certificate obtained|challenge failed'
```

---

## Step 5 — Test it end-to-end

From a **different** machine (not the server itself — hairpin NAT can give a false
"refused"):

```bash
curl -s -X POST -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  https://rpc.yourdomain.com/ | jq -r '.result'
```

A hex block height means TLS, the proxy, and the node are all working. Point your
wallet or dApp at `https://rpc.yourdomain.com/` with chain ID **60187** (testnet) /
**60186** (mainnet). See [connection details](../wallet/eblas-network-connection-details.md).

---

## Security & operations checklist

* **Node stays private** — RPC bound to `127.0.0.1:7777`; only 80/443 are public.
* **Test API off** — never `--rpc.enable-test-rpc` on a public node.
* **CORS scoped** — reflect only your origins unless you intend a fully open RPC.
* **Rate-limit abuse** — a public endpoint will get hammered. Put rate limiting in
  front (Caddy's `rate_limit` plugin, a Cloudflare proxy, or `nginx limit_req`),
  and consider blocking the heaviest methods (`eth_getLogs` with wide ranges,
  `debug_*`, `trace_*`).
* **Run more than one** — for reliability, run two RPC nodes behind the proxy and
  `reverse_proxy` to both (Caddy load-balances and health-checks them).
* **Watch health** — keep an eye on sync and peers with
  [Monitor node health & sync](monitor-node-health-and-sync.md); a silently
  desynced RPC node serves stale data.
* **Keep it updated** — pull the latest `ghcr.io/ebla-network/ebla-node:ebla-stable`
  image on each [node upgrade](../node-setup/upgrade-a-node/).
