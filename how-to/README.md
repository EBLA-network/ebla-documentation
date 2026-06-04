---
description: Practical, copy-paste guides for people running an EBLA node.
---

# 🧭 How-To Guides

Short, task-focused guides for EBLA node operators. Each one is self-contained:
prerequisites, the exact commands to run, and how to read the output. The tools are
**read-only** — they query the chain and your node, and never send transactions.

### Available guides

* [🕵️ Check validator slashing status](check-validator-slashing.md) — detect validators that have stopped producing blocks, see how much voting power they've lost, and project when an inactive validator will be evicted from the active set.

A taste of the output — `ebla-slash-check` scanning the live validator set in one command:

```
EBLA validator health @ block 251903  (9 validators, RPC http://127.0.0.1:7777)
validator                                            stake  expect   votes  status
0xc6cF7C14CA2308AA9269b9f3Aa8C711BB85481AB         1000000    1000    1000  ok
0xb3beeF2d5696ae67611407DBC54C94a61A16D589          681091     681     681  ok
0x232efe2995Ea01e0c4a42f53E0504b6743b69f9E            6000       6       5  SLASHED (~83% of expected votes)
…
1 validator(s) being inactivity-slashed.  total eligible votes = 2347
```

_More guides will be added here over time._
