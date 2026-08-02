<p align="right"><b>English</b> · <a href="README.md">Русский</a></p>

# lzt-flows

A catalog of ready-made modules for the [auto-lzt](https://github.com/open-lzt/auto-lzt) automation engine.

From the bot:

```
/modules              — list available modules
/import bump-daily    — install a module by name
```

From a server with no stand set up yet:

```bash
wget -qO- https://github.com/open-lzt/open-lzt/raw/main/install-flow.sh | sudo bash
```

## Modules

Nine of them: seven graphs and two node packs.

| Module | Version | Kind | What it does |
|---|---|---|---|
| `bump-daily` | 1.1.0 | flow | bumps every lot on the account on a schedule, every 4 hours by default |
| `sniper-autobuy` | 1.0.0 | flow | autobuy in any category — the section is a parameter, not baked into the graph |
| `steam-autobuy` | 2.1.0 | flow | Steam autobuy: price ceiling, count cutoff |
| `fortnite-autobuy` | 1.0.0 | flow | the same for Fortnite |
| `riot-autobuy` | 1.0.0 | flow | the same for Valorant and League of Legends |
| `supercell-autobuy` | 1.0.0 | flow | the same for Brawl Stars, Clash of Clans, Clash Royale |
| `telegram-autobuy` | 1.0.0 | flow | the same for Telegram accounts |
| `pricing-pack` | 1.0.0 | python | pricing nodes: percent of price, round to a nice number |
| `notify-pack` | 1.0.0 | python | notification nodes: Discord and a generic webhook |

Every autobuy module ships with `dry_run` on. Turn it off once you've seen what lands in the results.

## Two kinds of module

`kind: python` **is executable code, pip-installed onto the operator's host, running with their tokens.** That's why only the repo owner publishes those. `kind: flow` is data, and anyone can publish it.

| | `kind: flow` | `kind: python` |
|---|---|---|
| What it is | a node graph | a package of Python nodes |
| Files | `module.yaml` + `flow.json` | `module.yaml` + `pyproject.toml` + the package |
| Executed | by the engine, along the graph | on the host, as ordinary code |
| Who publishes | any author listed in `authors.yml` | the repo owner only |

## Module shape

`modules/<name>/module.yaml`:

```yaml
schema_version: 1
name: steam-autobuy
version: 2.1.0
author: zlexdev
kind: flow
description: |-
  Steam account autobuy: filtered search with a price ceiling, count cutoff, buy each.
  dry_run is on by default.
requires_nodes:
  - market.search
  - logic.condition
  - logic.take
  - logic.for_each_lot
  - market.fast_buy
```

`kind` may be omitted — it defaults to `flow`. `author` must match the PR author.

`modules/<name>/flow.json` — the graph itself: parameters and nodes.

```json
{
  "name": "steam-autobuy",
  "entry_node_id": "search",
  "params": [
    {
      "key": "max_price",
      "label": "Price up to",
      "control": "number",
      "required": true,
      "default": 10,
      "minimum": 1
    }
  ],
  "nodes": [
    {
      "id": "search",
      "type": "market.search",
      "inputs": {
        "max_price": { "literal": "{{vars.max_price}}" },
        "category": { "literal": "steam" }
      },
      "edges": { "next": "found" }
    }
  ]
}
```

## index.json

Generated automatically on push to `main`. Don't touch it by hand — a PR that changes it is rejected.

```json
{
  "schema_version": 1,
  "modules": [
    { "name": "steam-autobuy", "version": "2.1.0", "sha256": "98cf4a6a…", "kind": "flow" }
  ]
}
```

`flow.json` is hashed for `kind: flow`, `pyproject.toml` for `kind: python`.

**The sha256 here is a transfer-integrity check, not a signature.** It catches a corrupted download, not a swap: whoever controls `index.json` controls the hash too.

## Contributing a module

1. One PR: add your GitHub login to `authors.yml`.
2. A separate PR: the `modules/<name>/` directory. One module per PR — never mixed with `authors.yml`.
3. CI runs the checks.

What `.github/workflows/validate.yml` enforces:

| Check | What fails the PR |
|---|---|
| no `pull_request_target` in workflows | supply-chain guard: a fork must not get secrets |
| `check_code_owner.py` | a `.py` or `pyproject.toml` from anyone but the repo owner |
| one concern per PR | `authors.yml` and `modules/` changed together |
| `index.json` untouched | hand-editing a generated file |
| `check_author.py` | manifest `author` doesn't match the PR author, or isn't in `authors.yml` |
| `lzt-flow-validate modules/<name>` | the graph doesn't compile or references an unknown node |

The validator is the same one the backend runs: installed via `pip install lzt-flow` from [auto-lzt](https://github.com/open-lzt/auto-lzt).

The real security boundary for code is not CI but `CODEOWNERS` plus branch protection. `check_code_owner.py` only surfaces a readable error earlier.

## Ecosystem

[auto-lzt](https://github.com/open-lzt/auto-lzt) — the engine · [lzt-plugins](https://github.com/open-lzt/lzt-plugins) — executable plugins · [the whole stand](https://github.com/open-lzt/open-lzt)
