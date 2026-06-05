---
description: >-
  Detect inactive validators and watch EBLA's inactivity-slashing in real time,
  straight from your node.
---

# 🕵️ Check validator slashing status

EBLA penalises validators that stop producing blocks. This guide gives you copy‑paste
tools to answer three questions from any synced node:

1. Is **my** validator losing voting power?
2. **Which** validators on the network are being slashed right now?
3. For a slashed validator — **how many** times has it been penalised, and **when** will it be evicted?

Everything here is **read-only** — it sends no transactions and cannot change the chain.

## How EBLA inactivity slashing works

Inactivity slashing is **automatic**. There is no command to start it: the DPoS
contract runs the check itself during block finalization, once every **10,000
blocks** (~10.3 hours at 3.7 s blocks).

* A validator that produces **no DAG block** during a 10,000-block window loses **5%** of its **voting-power factor** (100% → 95% → 90.25% → 85.74% → …). The decay is multiplicative.
* It is **voting power that decays, never the coins.** The staked balance is untouched.
* **Effective stake** = `total_stake × factor`. **Votes** = `floor(effective_stake / 1,000 EBLA)`.
* **Recovery is instant:** producing a single DAG block resets the factor to 100%.
* **Eviction:** when effective stake falls below the **5,000 EBLA** eligibility threshold, the validator is force-undelegated (kicked off the active set).

So the voting-power factor is the single source of truth: full power = healthy,
anything below 100% = currently being slashed.

> **Important — these events are not in `eth_getLogs`.** The `InactivityPenalty`,
> `ValidatorEvicted` and `VotingPowerRecovered` events are emitted during block
> finalization, not inside a transaction, so they never enter the receipt/log index
> that `eth_getLogs` reads. Querying logs for them returns nothing. Read the
> voting-power factor from contract storage instead — that is exactly what
> [Test 3](#test-3-exact-penalty-count-factor-and-eviction-eta) does.

## Prerequisites

The DPoS precompile lives at the same address on every EBLA network:

```
0x00000000000000000000000000000000000000fe
```

Point the tools at an RPC endpoint — your own node, or a public one:

```bash
export EBLA_RPC=http://127.0.0.1:7777          # your local node
# export EBLA_RPC=https://rpc.testnet.eblanetwork.com/   # public testnet RPC
# export EBLA_RPC=https://rpc.eblanetwork.com/           # public mainnet RPC
```

You need:

* `curl` and `jq` for the quick check.
* `python3` + `web3` for the validator-wide scripts. Install once into a virtualenv:

```bash
python3 -m venv ~/ebla-venv
~/ebla-venv/bin/pip install -q web3
```

> **web3.py note:** EBLA answers `web3_clientVersion` with an error, so
> `w3.is_connected()` returns `False` even though every real call works. The
> scripts below deliberately do **not** call `is_connected()` — don't add it.

---

## Test 1 — Is my own node healthy?

The fastest check. `get_node_status` reports your node's own view, including its
current voting power (`dpos_node_votes`). It requires the node to be started with
`--rpc.enable-test-rpc` (your own node), and returns its fields under `.result`:

```bash
curl -s -X POST -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","method":"get_node_status","params":[],"id":1}' \
  "$EBLA_RPC" | jq '.result | {synced, pbft_size, dpos_node_votes, dpos_total_votes, dpos_quorum, peer_count}'
```

Read it like this:

| Field | Healthy looks like |
|---|---|
| `synced` | `true` |
| `dpos_node_votes` | equals `floor(your_total_stake / 1000)` — if it's **lower**, your node is being slashed |
| `dpos_total_votes` | sum of all validators' votes (sanity-cross-check for Test 2) |
| `dpos_quorum` | the 5/8 threshold the chain needs to finalize |
| `peer_count` | greater than 0 |

If `dpos_node_votes` has dropped below what your stake should give you, your node
has stopped producing DAG blocks — jump to Test 3 for the exact penalty count.

---

## Test 2 — Scan every validator for slashing

This lists every registered validator and flags the ones whose live eligible-vote
count is below what their stake should provide — i.e. the ones losing power.

Save it as `ebla-slash-check`:

```python
#!/usr/bin/env python3
"""
ebla-slash-check — list every registered EBLA validator and flag the ones whose
voting power is being inactivity-slashed (they have stopped producing DAG blocks).
Compares on-chain eligible votes against the votes their stake should give them;
a shortfall means the validator is losing power.

Usage:  EBLA_RPC=http://127.0.0.1:7777 python3 ebla-slash-check
"""
import os
import sys
from web3 import Web3

RPC = os.environ.get("EBLA_RPC", "http://localhost:7777")
w3 = Web3(Web3.HTTPProvider(RPC))
DPOS = Web3.to_checksum_address("0x00000000000000000000000000000000000000fe")
ONE, STEP, ELIG = 10**18, 1000 * 10**18, 5000 * 10**18

ABI = [
    {"name": "getValidators", "stateMutability": "view", "type": "function",
     "inputs": [{"name": "batch", "type": "uint32"}],
     "outputs": [
         {"name": "validators", "type": "tuple[]", "components": [
             {"name": "account", "type": "address"},
             {"name": "info", "type": "tuple", "components": [
                 {"name": "total_stake", "type": "uint256"},
                 {"name": "commission_reward", "type": "uint256"},
                 {"name": "commission", "type": "uint16"},
                 {"name": "last_commission_change", "type": "uint64"},
                 {"name": "undelegations_count", "type": "uint16"},
                 {"name": "owner", "type": "address"},
                 {"name": "description", "type": "string"},
                 {"name": "endpoint", "type": "string"}]}]},
         {"name": "end", "type": "bool"}]},
    {"name": "getValidatorEligibleVotesCount", "stateMutability": "view", "type": "function",
     "inputs": [{"name": "validator", "type": "address"}],
     "outputs": [{"name": "", "type": "uint64"}]},
]

c = w3.eth.contract(address=DPOS, abi=ABI)
head = w3.eth.block_number

# enumerate every validator (paginated until end == True)
validators, batch = [], 0
while True:
    res = c.functions.getValidators(batch).call()
    validators += res[0]
    if res[1]:
        break
    batch += 1

print(f"EBLA validator health @ block {head}  ({len(validators)} validators, RPC {RPC})")
print(f"{'validator':44}{'stake':>14}{'expect':>8}{'votes':>8}  status")

flagged, total_votes = 0, 0
for val in validators:
    account, stake = val[0], val[1][0]
    expected = (stake // STEP) if stake >= ELIG else 0      # votes the stake should give
    votes = c.functions.getValidatorEligibleVotesCount(account).call()  # votes after slashing
    total_votes += votes
    if votes < expected:
        flagged += 1
        status = f"SLASHED (~{round(100 * votes / expected)}% of expected votes)"
    elif expected:
        status = "ok"
    elif stake == 0:
        status = "EVICTED (stake 0)"
    else:
        status = "below 5,000 EBLA threshold"
    print(f"{account:44}{stake // ONE:>14}{expected:>8}{votes:>8}  {status}")

print(f"\n{flagged} validator(s) being inactivity-slashed.  total eligible votes = {total_votes}")
sys.exit(1 if flagged else 0)
```

Run it:

```bash
EBLA_RPC=http://127.0.0.1:7777 ~/ebla-venv/bin/python3 ebla-slash-check
```

Example output:

```
EBLA validator health @ block 251903  (9 validators, RPC http://127.0.0.1:7777)
validator                                            stake  expect   votes  status
0xc6cF7C14CA2308AA9269b9f3Aa8C711BB85481AB         1000000    1000    1000  ok
0xb3beeF2d5696ae67611407DBC54C94a61A16D589          681091     681     681  ok
0x232efe2995Ea01e0c4a42f53E0504b6743b69f9E            6000       6       5  SLASHED (~83% of expected votes)
...

1 validator(s) being inactivity-slashed.  total eligible votes = 2347
```

How to read it:

* **`ok`** — full voting power, producing blocks normally.
* **`SLASHED`** — eligible votes are below the stake-implied amount, so the validator has stopped producing DAG blocks and is decaying. The `%` is a coarse hint; for the **exact** factor and penalty count, run Test 3.
* **`below 5,000 EBLA threshold`** — staked under the minimum, not consensus-eligible at all.
* **`EVICTED (stake 0)`** — was force-undelegated after dropping below the threshold; stake is now 0 and it's out of the active set.
* The exit code is `1` if anything is flagged, `0` if all healthy — convenient for cron/monitoring.
* Cross-check: the printed `total eligible votes` should match `dpos_total_votes` from Test 1.

> The `%` shown here is `votes ÷ expected_votes` with integer rounding, which is
> lossy for small validators (e.g. a 6,000-stake validator at 95% factor still shows
> "5 of 6 votes" = 83%). The real power factor is read precisely in Test 3.

---

## Test 3 — Exact penalty count, factor and eviction ETA

For a single validator, this reads the **voting-power factor directly from contract
storage** (the reliable source, since the events aren't in `eth_getLogs`), works out
exactly how many −5% penalties produced it, and — if the validator stays offline —
projects the block and time at which it will be evicted.

Save it as `ebla-validator-eviction`:

```python
#!/usr/bin/env python3
"""
ebla-validator-eviction — for ONE validator: how many -5% inactivity penalties it
has taken, its current voting-power factor, and (if it stays offline) the block +
ETA at which it will be force-evicted.

Reads the factor straight from DPoS precompile storage at keccak256(0x09 || addr),
because the InactivityPenalty events are emitted at block finalization and are NOT
returned by eth_getLogs.

Usage:  EBLA_RPC=http://127.0.0.1:7777 python3 ebla-validator-eviction <validator_address>
"""
import os
import sys
from web3 import Web3

RPC = os.environ.get("EBLA_RPC", "http://localhost:7777")
if len(sys.argv) < 2:
    sys.exit("usage: ebla-validator-eviction <validator_address>")

w3 = Web3(Web3.HTTPProvider(RPC))
DPOS = Web3.to_checksum_address("0x00000000000000000000000000000000000000fe")
v = Web3.to_checksum_address(sys.argv[1])

EPOCH = 10_000          # inactivity epoch length, in blocks
BLOCK_MS = 3_700        # ~3.7 s target block time
ELIG = 5_000            # consensus-eligibility threshold, in EBLA
SCALE = 10_000          # basis-points scale: 100% = 10000
FIELD_FACTOR = b"\x09"  # DPoS storage field id for the voting-power factor

# getValidator -> the 8-field ValidatorBasicInfo tuple; total_stake is index 0
ABI = [{"name": "getValidator", "stateMutability": "view", "type": "function",
        "inputs": [{"name": "validator", "type": "address"}],
        "outputs": [{"name": "info", "type": "tuple", "components": [
            {"name": "total_stake", "type": "uint256"},
            {"name": "commission_reward", "type": "uint256"},
            {"name": "commission", "type": "uint16"},
            {"name": "last_commission_change", "type": "uint64"},
            {"name": "undelegations_count", "type": "uint16"},
            {"name": "owner", "type": "address"},
            {"name": "description", "type": "string"},
            {"name": "endpoint", "type": "string"}]}]}]

c = w3.eth.contract(address=DPOS, abi=ABI)
stake = c.functions.getValidator(v).call()[0] // 10**18
head = w3.eth.block_number

# an evicted validator has stake 0 and no meaningful factor — report and stop
if stake == 0:
    print(f"validator        {v}")
    print(f"head block       {head}")
    print("stake            0 EBLA")
    print("status           EVICTED — force-undelegated, removed from the active set")
    sys.exit(0)

# voting-power factor: stored at keccak256(0x09 || address) as (factor + 1);
# a stored 0 means "never set" => default 100% (SCALE).
slot = Web3.keccak(FIELD_FACTOR + bytes.fromhex(v[2:]))
stored = int.from_bytes(w3.eth.get_storage_at(DPOS, slot), "big")
factor = (stored - 1) if stored else SCALE

# how many -5% penalties produced this factor? walk the decay down to it.
f, taken = SCALE, 0
while f > factor:
    f = f * 95 // 100
    taken += 1
clean = (f == factor)

effective = stake * factor // SCALE
print(f"validator        {v}")
print(f"stake            {stake} EBLA")
print(f"head block       {head}")
print(f"voting factor    {factor} bps ({factor / 100:.2f}% power)" + ("" if clean else "  (!) not a clean -5% match"))
print(f"penalties taken  {taken}")
print(f"effective stake  {effective} EBLA  (evicted below {ELIG})")

if factor == SCALE:
    print("status           HEALTHY — full voting power, not being slashed")
    sys.exit(0)
if effective < ELIG:
    print("status           BELOW THRESHOLD — eviction in progress / complete")
    sys.exit(0)

# project forward: keep applying -5% until effective stake drops below the threshold
f, more = factor, 0
while stake * f // SCALE >= ELIG:
    f = f * 95 // 100
    more += 1
next_boundary = ((head // EPOCH) + 1) * EPOCH          # penalties only fire at multiples of 10,000
evict_block = next_boundary + (more - 1) * EPOCH
away = evict_block - head
print(f"status           SLASHED — {more} more missed epoch(s) until eviction")
print(f"next -5% at block {next_boundary}")
print(f"EVICTED at block {evict_block}  "
      f"(~{away} blocks, ~{away * BLOCK_MS / 3_600_000:.1f} h / ~{away * BLOCK_MS / 86_400_000:.2f} d) if it stays offline")
print("recovery         producing ONE DAG block resets the factor to 100% and cancels eviction")
```

Run it on a flagged validator from Test 2:

```bash
EBLA_RPC=http://127.0.0.1:7777 ~/ebla-venv/bin/python3 ebla-validator-eviction 0x232efe2995Ea01e0c4a42f53E0504b6743b69f9E
```

Example output:

```
validator        0x232efe2995Ea01e0c4a42f53E0504b6743b69f9E
stake            6000 EBLA
head block       252400
voting factor    9500 bps (95.00% power)
penalties taken  1
effective stake  5700 EBLA  (evicted below 5000)
status           SLASHED — 3 more missed epoch(s) until eviction
next -5% at block 260000
EVICTED at block 280000  (~27600 blocks, ~28.4 h / ~1.18 d) if it stays offline
recovery         producing ONE DAG block resets the factor to 100% and cancels eviction
```

This tells you precisely: the validator has been slashed **once** (factor 95%), and
unless it comes back online it will be evicted at **block 280,000**, about **28 hours**
away.

### Reading the factor

The factor decays in clean −5% steps. For any validator, eviction happens the moment
`effective stake = stake × factor` drops below 5,000 EBLA:

| Penalties | Factor | Example: 6,000-EBLA validator | Status |
|---|---|---|---|
| 0 | 100.00% | 6,000 effective | healthy |
| 1 | 95.00% | 5,700 effective | slashed |
| 2 | 90.25% | 5,415 effective | slashed |
| 3 | 85.74% | 5,144 effective | slashed |
| 4 | 81.45% | 4,887 effective | **evicted** |

A validator with more stake survives more missed epochs before crossing the 5,000
floor; the script computes the exact eviction block for whatever stake it reads.

---

## Turn it into a monitor

Both scripts are safe to run on a schedule. For a 5-minute heartbeat:

```bash
watch -n 300 'EBLA_RPC=http://127.0.0.1:7777 ~/ebla-venv/bin/python3 ebla-slash-check'
```

Or, because `ebla-slash-check` exits non-zero when any validator is flagged, drive an
alert from cron:

```bash
*/10 * * * * EBLA_RPC=http://127.0.0.1:7777 /root/ebla-venv/bin/python3 /root/ebla-slash-check >> /var/log/ebla-slash.log 2>&1 || echo "EBLA: a validator is being slashed" | mail -s "EBLA slash alert" you@example.com
```

> **Tip:** if pasting a script into a terminal heredoc mangles the Python
> indentation, save it with an editor (`nano ebla-slash-check`) instead, or transfer
> the file with `scp`.

## Summary

| Question | Tool |
|---|---|
| Is *my* node losing power? | Test 1 — `get_node_status` → `dpos_node_votes` |
| *Which* validators are slashed? | Test 2 — `ebla-slash-check` |
| *How many* penalties / *when* evicted? | Test 3 — `ebla-validator-eviction` |
| Slash status of the **whole fleet** in one go | [A to Z](#a-to-z--full-copy-paste-setup-all-nodes-slash-report) — `ebla-slash-all` |

---

## A to Z — full copy-paste setup (all-nodes slash report)

The fastest path on a **fresh node**: no editor, no manual file creation — paste these
three blocks into the CLI in order. They install and run `ebla-slash-all`, the
eviction-aware all-fleet report that reads each validator's **exact** voting-power
factor from storage and labels evicted (stake 0) validators cleanly. The script is
delivered base64-encoded so it pastes into any terminal without mangling Python
indentation.

### STEP 1 — one-time setup (Python + web3)

```bash
apt-get update -qq && apt-get install -y -qq python3 python3-venv python3-pip
python3 -m venv ~/ebla-venv
~/ebla-venv/bin/pip install -q --upgrade pip web3
~/ebla-venv/bin/python3 -c 'import web3; print("web3 OK", web3.__version__)'
```

Expect a final line like `web3 OK 7.16.0`. Run STEP 1 only once per node.

### STEP 2 — install the all-nodes slash report

```bash
echo 'IyEvdXNyL2Jpbi9lbnYgcHl0aG9uMwojIGVibGEtc2xhc2gtYWxsIOKAlCBleGFjdCBpbmFjdGl2aXR5LXNsYXNoIHN0YXR1cyBmb3IgRVZFUlkgcmVnaXN0ZXJlZCB2YWxpZGF0b3IuCiMgUmVhZC1vbmx5LiBIYW5kbGVzIGV2aWN0ZWQgKHN0YWtlIDApIHZhbGlkYXRvcnMgY2xlYW5seS4KaW1wb3J0IG9zLCBzeXMKZnJvbSB3ZWIzIGltcG9ydCBXZWIzCgpSUEMgPSBvcy5lbnZpcm9uLmdldCgiRUJMQV9SUEMiLCAiaHR0cDovL2xvY2FsaG9zdDo3Nzc3IikKdzMgPSBXZWIzKFdlYjMuSFRUUFByb3ZpZGVyKFJQQykpCkRQT1MgPSBXZWIzLnRvX2NoZWNrc3VtX2FkZHJlc3MoIjB4MDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDBmZSIpCk9ORSwgU1RFUCwgRUxJRywgU0NBTEUgPSAxMCoqMTgsIDEwMDAsIDUwMDAsIDEwMDAwCkZJRUxEID0gYiJceDA5IiAgIyBEUG9TIHN0b3JhZ2UgZmllbGQgaWQgZm9yIHRoZSB2b3RpbmctcG93ZXIgZmFjdG9yCgpBQkkgPSBbCiB7Im5hbWUiOiAiZ2V0VmFsaWRhdG9ycyIsICJzdGF0ZU11dGFiaWxpdHkiOiAidmlldyIsICJ0eXBlIjogImZ1bmN0aW9uIiwKICAiaW5wdXRzIjogW3sibmFtZSI6ICJiYXRjaCIsICJ0eXBlIjogInVpbnQzMiJ9XSwKICAib3V0cHV0cyI6IFsKICAgIHsibmFtZSI6ICJ2YWxpZGF0b3JzIiwgInR5cGUiOiAidHVwbGVbXSIsICJjb21wb25lbnRzIjogWwogICAgICB7Im5hbWUiOiAiYWNjb3VudCIsICJ0eXBlIjogImFkZHJlc3MifSwKICAgICAgeyJuYW1lIjogImluZm8iLCAidHlwZSI6ICJ0dXBsZSIsICJjb21wb25lbnRzIjogWwogICAgICAgIHsibmFtZSI6ICJ0b3RhbF9zdGFrZSIsICJ0eXBlIjogInVpbnQyNTYifSwKICAgICAgICB7Im5hbWUiOiAiY29tbWlzc2lvbl9yZXdhcmQiLCAidHlwZSI6ICJ1aW50MjU2In0sCiAgICAgICAgeyJuYW1lIjogImNvbW1pc3Npb24iLCAidHlwZSI6ICJ1aW50MTYifSwKICAgICAgICB7Im5hbWUiOiAibGFzdF9jb21taXNzaW9uX2NoYW5nZSIsICJ0eXBlIjogInVpbnQ2NCJ9LAogICAgICAgIHsibmFtZSI6ICJ1bmRlbGVnYXRpb25zX2NvdW50IiwgInR5cGUiOiAidWludDE2In0sCiAgICAgICAgeyJuYW1lIjogIm93bmVyIiwgInR5cGUiOiAiYWRkcmVzcyJ9LAogICAgICAgIHsibmFtZSI6ICJkZXNjcmlwdGlvbiIsICJ0eXBlIjogInN0cmluZyJ9LAogICAgICAgIHsibmFtZSI6ICJlbmRwb2ludCIsICJ0eXBlIjogInN0cmluZyJ9XX1dfSwKICAgIHsibmFtZSI6ICJlbmQiLCAidHlwZSI6ICJib29sIn1dfSwKIHsibmFtZSI6ICJnZXRWYWxpZGF0b3JFbGlnaWJsZVZvdGVzQ291bnQiLCAic3RhdGVNdXRhYmlsaXR5IjogInZpZXciLCAidHlwZSI6ICJmdW5jdGlvbiIsCiAgImlucHV0cyI6IFt7Im5hbWUiOiAidmFsaWRhdG9yIiwgInR5cGUiOiAiYWRkcmVzcyJ9XSwKICAib3V0cHV0cyI6IFt7Im5hbWUiOiAiIiwgInR5cGUiOiAidWludDY0In1dfSwKXQoKZGVmIHBlbmFsdGllc19mb3IoZmFjdG9yKToKICAgICIiIkhvdyBtYW55IC01JSBzdGVwcyBmcm9tIDEwMCUgcmVhY2ggYGZhY3RvcmA7IE5vbmUgaWYgbm90IGEgY2xlYW4gZGVjYXkgdmFsdWUuIiIiCiAgICBpZiBmYWN0b3IgPj0gU0NBTEU6CiAgICAgICAgcmV0dXJuIDAKICAgIGYsIG4gPSBTQ0FMRSwgMAogICAgd2hpbGUgZiA+IGZhY3RvcjoKICAgICAgICBmID0gZiAqIDk1IC8vIDEwMAogICAgICAgIG4gKz0gMQogICAgICAgIGlmIGYgPT0gMDoKICAgICAgICAgICAgcmV0dXJuIE5vbmUKICAgIHJldHVybiBuIGlmIGYgPT0gZmFjdG9yIGVsc2UgTm9uZQoKYyA9IHczLmV0aC5jb250cmFjdChhZGRyZXNzPURQT1MsIGFiaT1BQkkpCmhlYWQgPSB3My5ldGguYmxvY2tfbnVtYmVyCgp2YWxzLCBiYXRjaCA9IFtdLCAwCndoaWxlIFRydWU6CiAgICByID0gYy5mdW5jdGlvbnMuZ2V0VmFsaWRhdG9ycyhiYXRjaCkuY2FsbCgpCiAgICB2YWxzICs9IHJbMF0KICAgIGlmIHJbMV06CiAgICAgICAgYnJlYWsKICAgIGJhdGNoICs9IDEKCnByaW50KGYiRUJMQSBzbGFzaCByZXBvcnQgQCBibG9jayB7aGVhZH0gICh7bGVuKHZhbHMpfSB2YWxpZGF0b3JzLCBSUEMge1JQQ30pIikKcHJpbnQoZiJ7J3ZhbGlkYXRvcic6NDJ9IHsnc3Rha2UnOj4xMX0geydwb3dlcic6Pjd9IHsncGVuJzo+NH0geyd2b3Rlcyc6PjZ9ICBzdGF0dXMiKQpzbGFzaGVkID0gZXZpY3RlZCA9IDAKZm9yIHYgaW4gdmFsczoKICAgIGFjY3QgPSB2WzBdCiAgICBzdGFrZSA9IHZbMV1bMF0gLy8gT05FCiAgICBpZiBzdGFrZSA9PSAwOgogICAgICAgIGV2aWN0ZWQgKz0gMQogICAgICAgIHByaW50KGYie2FjY3Q6NDJ9IHtzdGFrZTo+MTF9IHsnLSc6Pjd9IHsnLSc6PjR9IHswOj42fSAgRVZJQ1RFRCAoc3Rha2UgMCkiKQogICAgICAgIGNvbnRpbnVlCiAgICBzdG9yZWQgPSBpbnQuZnJvbV9ieXRlcyh3My5ldGguZ2V0X3N0b3JhZ2VfYXQoRFBPUywgV2ViMy5rZWNjYWsoRklFTEQgKyBieXRlcy5mcm9taGV4KGFjY3RbMjpdKSkpLCAiYmlnIikKICAgIGZhY3RvciA9IChzdG9yZWQgLSAxKSBpZiBzdG9yZWQgZWxzZSBTQ0FMRQogICAgcGVuID0gcGVuYWx0aWVzX2ZvcihmYWN0b3IpCiAgICB2b3RlcyA9IGMuZnVuY3Rpb25zLmdldFZhbGlkYXRvckVsaWdpYmxlVm90ZXNDb3VudChhY2N0KS5jYWxsKCkKICAgIGVmZiA9IHN0YWtlICogZmFjdG9yIC8vIFNDQUxFCiAgICBpZiBlZmYgPCBFTElHOgogICAgICAgIHN0YXR1cyA9ICJJTkVMSUdJQkxFIgogICAgZWxpZiBmYWN0b3IgPCBTQ0FMRToKICAgICAgICBzdGF0dXMgPSAiU0xBU0hFRCIKICAgICAgICBzbGFzaGVkICs9IDEKICAgIGVsc2U6CiAgICAgICAgc3RhdHVzID0gIm9rIgogICAgcGVuX3MgPSBzdHIocGVuKSBpZiBwZW4gaXMgbm90IE5vbmUgZWxzZSAiPyIKICAgIHByaW50KGYie2FjY3Q6NDJ9IHtzdGFrZTo+MTF9IHtmYWN0b3IvMTAwOj42LjFmfSUge3Blbl9zOj40fSB7dm90ZXM6PjZ9ICB7c3RhdHVzfSIpCgpwcmludChmIlxue3NsYXNoZWR9IHZhbGlkYXRvcihzKSBiZWluZyBzbGFzaGVkLCB7ZXZpY3RlZH0gZXZpY3RlZCAoc3Rha2UgMCkuIikKc3lzLmV4aXQoMSBpZiAoc2xhc2hlZCBvciBldmljdGVkKSBlbHNlIDApCg==' | base64 -d > ~/ebla-slash-all
```

### STEP 3 — run it

```bash
EBLA_RPC=http://127.0.0.1:7777 ~/ebla-venv/bin/python3 ~/ebla-slash-all
```

> If this node isn't fully synced yet, point at one that is, e.g.
> `EBLA_RPC=https://rpc.testnet.eblanetwork.com/`. The report reads **global** DPoS
> state, so every node returns the same fleet.

Example output (an evicted validator shown cleanly):

```
EBLA slash report @ block 280631  (9 validators, RPC http://127.0.0.1:7777)
validator                                        stake   power  pen  votes  status
0xc6cF7C14CA2308AA9269b9f3Aa8C711BB85481AB     1000000  100.0%    0   1000  ok
0xb3beeF2d5696ae67611407DBC54C94a61A16D589      681091  100.0%    0    681  ok
0x232efe2995Ea01e0c4a42f53E0504b6743b69f9E           0       -    -      0  EVICTED (stake 0)
0x6234827B87e5F8313e3d16A95A5b3A9FC9a85521      511990  100.0%    0    511  ok
...

0 validator(s) being slashed, 1 evicted (stake 0).
```

| Column | Meaning |
|---|---|
| `power` | exact voting-power factor from storage — `100.0%` healthy, lower = slashed |
| `pen` | number of −5% penalties taken (`95% → 1`, `90.25% → 2`, …) |
| `votes` | live on-chain eligible votes |
| `status` | `ok` / `SLASHED` / `INELIGIBLE` / `EVICTED (stake 0)` |

Exit code is `1` if any validator is slashed or evicted, `0` if the whole fleet is
healthy — drop it straight into `watch` or cron.
