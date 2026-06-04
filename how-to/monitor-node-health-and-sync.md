---
description: >-
  Confirm an EBLA node is synced, peered, and participating in consensus —
  straight from the node, with copy-paste commands.
---

# 📡 Monitor node health & sync

A node is only useful when it is **synced**, **connected to peers**, and (for a
validator) **participating in consensus**. This guide gives you copy‑paste checks
to confirm all three, plus a one‑command health probe you can drop into cron.

Everything here is **read-only** — it sends no transactions and cannot change the chain.

## The three questions

1. Is my node **caught up** with the rest of the network?
2. Does it have healthy **peer connectivity**?
3. Is it **producing/finalizing** blocks (and, for a validator, voting)?

## Prerequisites

Point the tools at your node's RPC. The default RPC port is **7777**:

```bash
export EBLA_RPC=http://127.0.0.1:7777          # your local node
# export EBLA_RPC=https://rpc.testnet.eblanetwork.com/   # public testnet RPC
# export EBLA_RPC=https://rpc.eblanetwork.com/           # public mainnet RPC (after launch)
```

You need `curl` and `jq`.

> **Two kinds of RPC.** The detailed `get_node_status` call is a **test-API**
> method — it only answers if the node was started with `--rpc.enable-test-rpc`
> (the standard operator bundle enables it). The `eth_*` / `ebla_*` / `net_*`
> calls in [the public checks](#public-rpc-checks-no-test-rpc-needed) work on any
> endpoint, including public ones.

---

## The fastest check — the dashboard

Every node ships a status dashboard. Open your node's IP at port **3000**:

```
http://<your-node-ip>:3000
```

A healthy node shows **"Synced — Participating in consensus."** If you only have a
terminal, the checks below give you the same answer (and more).

---

## Test 1 — One-command health probe

`get_node_status` returns the node's own view of itself and the network. This
one-liner pulls the fields that matter:

```bash
curl -s -X POST -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"get_node_status","params":[],"id":1}' \
  "$EBLA_RPC" | jq '.result | {
    synced, syncing_seconds, peer_count, node_count,
    my_pbft_size: .pbft_size,
    best_peer_pbft_size: .network.peer_max_pbft_chain_size,
    pbft_sync_queue_size, blk_queue_size,
    dpos_node_votes, dpos_total_votes, dpos_quorum
  }'
```

Example output (a healthy, fully-synced node):

```json
{
  "synced": true,
  "syncing_seconds": 0,
  "peer_count": 8,
  "node_count": 9,
  "my_pbft_size": 251903,
  "best_peer_pbft_size": 251903,
  "pbft_sync_queue_size": 0,
  "blk_queue_size": 0,
  "dpos_node_votes": 1000,
  "dpos_total_votes": 2347,
  "dpos_quorum": 1468
}
```

How to read it:

| Field | Healthy looks like | What it means |
|---|---|---|
| `synced` | `true` | The node believes it is caught up. |
| `syncing_seconds` | `0` | Seconds spent still syncing; `0` once synced. |
| `peer_count` | `> 0` (ideally several) | Live peer connections. `0` = isolated — see [0 peers](#zero-peers). |
| `my_pbft_size` vs `best_peer_pbft_size` | **equal or within a few** | Your finalized height vs the best peer's. A large, growing gap = falling behind. |
| `pbft_sync_queue_size` / `blk_queue_size` | near `0` | Backlog waiting to be processed. Persistently large = the node can't keep up (usually disk/CPU). |
| `dpos_node_votes` | `floor(your_total_stake / 1000)` | **Validators only.** Your voting power. Lower than expected = you're being [inactivity-slashed](check-validator-slashing.md). `0` is normal for a non-validator/RPC node. |
| `dpos_total_votes` | network total | Sum of all validators' votes. |
| `dpos_quorum` | the 5/8 threshold | Votes the chain needs to finalize a block. |

The single most useful signal is **`my_pbft_size` vs `best_peer_pbft_size`**: if
they track each other block-for-block, you are keeping up. At EBLA's ~3.7 s block
time, both should advance by roughly one every few seconds.

---

## Test 2 — Public RPC checks (no test-rpc needed) <a id="public-rpc-checks-no-test-rpc-needed"></a>

These work against **any** endpoint — handy for checking a public RPC, or a node
where the test API is disabled.

**Latest block height (PBFT period):**

```bash
curl -s -X POST -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  "$EBLA_RPC" | jq -r '.result' | xargs printf "%d\n"
```

**Am I caught up?** Compare your node's height to a public reference node — if they
match (give or take a few blocks), you're synced:

```bash
mine=$(curl -s -X POST -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  "$EBLA_RPC" | jq -r '.result')
ref=$(curl -s -X POST -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  https://rpc.testnet.eblanetwork.com/ | jq -r '.result')
echo "mine=$((mine))  reference=$((ref))  behind=$(( ref - mine ))"
```

You can also eyeball the network height on the [testnet explorer](https://explorer.testnet.eblanetwork.com/).

**Chain stats** (finalized period, executed DAG blocks and transactions):

```bash
curl -s -X POST -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"ebla_getChainStats","params":[],"id":1}' \
  "$EBLA_RPC" | jq '.result'
```

```json
{ "pbft_period": 251903, "dag_blocks_executed": 12750431, "transactions_executed": 4815162 }
```

**Peer count** (standard `net_*`, no test API):

```bash
curl -s -X POST -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"net_peerCount","params":[],"id":1}' \
  "$EBLA_RPC" | jq -r '.result' | xargs printf "%d\n"
```

> **Note:** EBLA does **not** implement `web3_clientVersion` (it returns an error).
> Tools that ping it to test connectivity — e.g. `web3.py`'s `is_connected()` —
> will report a false "not connected" even though every real call works. Use
> `eth_blockNumber` or `net_listening` to probe liveness instead.

---

## Test 3 — Read the logs

The node prints a periodic status summary. Tail the container logs and look for the
`---- tl;dr ----` block:

```bash
docker compose logs -f --tail 200
```

The line you want says:

```
STATUS: GOOD. NODE SYNCED AND PARTICIPATING IN CONSENSUS
```

Messages you may see — and what they mean:

| Log message | Concern? |
|---|---|
| `STATUS: GOOD. NODE SYNCED AND PARTICIPATING IN CONSENSUS` | None — this is the goal. |
| `PARTICIPATING IN CONSENSUS BUT NO NEW FINALIZED BLOCKS` | Usually transient (network catching up). Watch the block height; if it stays flat for long, check peers. |
| `PBFT STALLED, POSSIBLY PARTITIONED` / `STUCK. NODE HAS NOT RESTARTED SYNCING` | Transient on a recovering network. Persistent = check connectivity and version. |

> The container name varies by host (`ebla_compose_node_1`, `ebla-compose-node-1`,
> …). If `docker compose logs` doesn't work from your folder, run `docker ps` to
> find the exact name, then `docker logs --tail 200 -f <name>`.

---

## Common issues

### "Synced" but no blocks produced (validators)

If the node is 100% synced but isn't producing blocks, confirm it is **registered**
and that its **total stake ≥ 5,000 EBLA** — below that threshold it stays out of
consensus. If it once produced blocks and stopped, check for
[inactivity slashing](check-validator-slashing.md).

### Sync percentage goes *down*

Normal. The node estimates progress from its peers; when it connects to a
further-ahead peer, it revises the target. Compare against the explorer for the
real network height.

### Zero peers <a id="zero-peers"></a>

`peer_count: 0` means the node can't reach the network. Common causes:

* **No connectivity** — the logs show `Number of discovered peers: 0`. Check the
  machine's outbound networking and that the p2p port (**10002**, TCP **and** UDP)
  is reachable.
* **Same-host hairpin NAT** — if this node shares a host/public IP with another
  EBLA node, it often can't reach that neighbour via the public IP. Add the
  neighbour's **internal** address as an extra bootnode:
  `--boot-nodes-append <internal-ip>:10002/<enode-pubkey>`.
* **Wrong version / corrupted state** — peers discovered but none connect usually
  means a version mismatch or a bad `state_db`; reset the node's data.

---

## Turn it into a monitor

A tiny wrapper that exits non-zero when the node is unhealthy — perfect for cron or
a `watch` heartbeat. Save it as `ebla-node-health`:

```bash
#!/usr/bin/env bash
# ebla-node-health — exit 0 if the node is synced with peers, 1 otherwise.
set -euo pipefail
RPC="${EBLA_RPC:-http://127.0.0.1:7777}"

json=$(curl -s -X POST -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"get_node_status","params":[],"id":1}' "$RPC")

synced=$(jq -r '.result.synced'        <<<"$json")
peers=$(jq  -r '.result.peer_count'    <<<"$json")
mine=$(jq   -r '.result.pbft_size'     <<<"$json")
best=$(jq   -r '.result.network.peer_max_pbft_chain_size' <<<"$json")
behind=$(( best - mine ))

echo "synced=$synced peers=$peers height=$mine best=$best behind=$behind"

# Unhealthy if: not synced, no peers, or more than 5 blocks behind the best peer.
if [[ "$synced" != "true" || "$peers" -lt 1 || "$behind" -gt 5 ]]; then
  echo "EBLA node UNHEALTHY"
  exit 1
fi
echo "EBLA node OK"
```

Run it on a schedule:

```bash
watch -n 30 'EBLA_RPC=http://127.0.0.1:7777 ./ebla-node-health'
```

Or alert from cron when it goes unhealthy:

```bash
*/5 * * * * EBLA_RPC=http://127.0.0.1:7777 /root/ebla-node-health >> /var/log/ebla-health.log 2>&1 \
  || echo "EBLA node unhealthy" | mail -s "EBLA node alert" you@example.com
```

## Summary

| Question | Tool |
|---|---|
| Synced, peered, voting? | Test 1 — `get_node_status` one-liner |
| Caught up vs a public node? | Test 2 — `eth_blockNumber` diff |
| What do the logs say? | Test 3 — `docker compose logs` → `STATUS: GOOD …` |
| Automated alerting | `ebla-node-health` in cron / `watch` |
