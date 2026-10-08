# Zyfai × Cork — integration workspace

This repository is where Zyfai integrates Cork cover. Zyfai's own code lives in
[`zyfai/`](./zyfai/) and is owned by the Zyfai team. Everything else here is the
integration foundation contributed by Cork: orientation, the runbook, and agent
context — pinned to one released Cork Distribution.

> **Status** — 2026-10-08. Pinned to Distribution **`phoenix/v0.5-rc.1`**
> (stage `partner-preview`, review level `unreviewed`) and cork-cli
> **`v0.7.0-rc.2`**. Chains: Arbitrum One (42161) and Base (8453); integrate on
> **Base first**. This cut keeps the Phoenix contracts of the previous pin, so
> your pools, your cover and both your adapters keep working. It changes the
> RFQ path: cork-cli `0.7` speaks venue RFQ v2 only, and every RFQ write is
> signed. It also adds rollover RFQs and the filler side of a rollover. See
> "Moving from v0.4-rc.1" below. The earlier hackathon integration package
> that used to live here is removed — see "What happened to the hackathon
> code" below.

## The one number to track

Everything you integrate against is pinned by a single name:

**[`phoenix/v0.5-rc.1`](https://github.com/Cork-Technology/distribution/blob/main/distributions/phoenix/v0.5-rc.1.json)**
in [`Cork-Technology/distribution`](https://github.com/Cork-Technology/distribution).

One name is not the other. That name is the **Distribution**, the set of
component versions. The tool labels each **contract generation** separately,
in `data.generation`: the primary is still `phoenix/v0.4-rc.1` and the previous
one `phoenix/v0.3-rc.1`, because this Distribution keeps the same Phoenix pool
manager. Both names are correct.

That manifest names the exact component versions, per-chain addresses and
codehashes, vendored ABIs, the third-party contracts the set was built against,
what review the set carries, and its known issues. Two rules follow:

1. **The manifest is the authority.** Where any document — including the ones
   linked below — disagrees with the manifest on a version or an address, the
   manifest wins.
2. **Read living facts fresh.** `stage`, `reviewLevel`, `support`, `status` and
   `knownIssues` can change in place. Read them from the distribution repo at
   the moment you need them, never from a copy.

There are deliberately **no contract addresses anywhere in this repository**.
Hand-copied address tables are how the previous integration package went stale
while still looking authoritative. Addresses come from the manifest, or live
from the tool (`ch query protocol-config`, which now lists every generation the
chain hosts, with each block's addresses and wire).

## Start here

| You want | Read |
|---|---|
| The integration path, start to finish | [`INTEGRATION.md`](./INTEGRATION.md) — the runbook for this workspace |
| The full walkthrough with runnable commands | [Zyfai quickstart](https://github.com/Cork-Technology/cork-cli/blob/v0.7.0-rc.2/docs/zyfai-quickstart.md) (pinned to the released `cork-cli` tag; **orientation, not `v0.7.0-rc.2` evidence** — its captures date from `0.6.1-rc.1` and earlier, and its rollover appendix predates `rollover-fill`; the runbook carries the current rollover flow) |
| The CLI / MCP command reference | [`ch` reference](https://github.com/Cork-Technology/cork-cli/blob/v0.7.0-rc.2/docs/cli.md) |
| The JIT-order contract, field by field | [`jit-order-anatomy.md`](https://github.com/Cork-Technology/cork-cli/blob/v0.7.0-rc.2/docs/jit-order-anatomy.md) — chain-agnostic and address-free |
| The TypeScript SDK behind the tool | [`sdk.md`](https://github.com/Cork-Technology/cork-cli/blob/v0.7.0-rc.2/docs/sdk.md) — `@cork/core`, shipped as attested tarballs on the same release |
| The venue / indexer API | [api-phoenix.cork.tech/docs](https://api-phoenix.cork.tech/docs) |
| The receiver-forcing adapter you deploy, one per generation | [`cork-periphery`](https://github.com/Cork-Technology/cork-periphery/tree/v0.2.0-rc.1) at `v0.2.0-rc.1` (ForSelf source unchanged since `v0.1.1`; the package no longer ships deploy scripts) |
| The rollover component's frozen deployment record | [rollover `v0.2.0` release](https://github.com/Cork-Technology/rollover/releases/tag/v0.2.0) — pinned, with caveats; see below |
| What exactly is deployed, and its assurance | [The manifest](https://github.com/Cork-Technology/distribution/blob/main/distributions/phoenix/v0.5-rc.1.json) |

Agents working in this repository get their context from [`CLAUDE.md`](./CLAUDE.md)
and operate Cork through the [`cork-integration`](./.claude/skills/cork-integration/SKILL.md)
skill, which wires the `cork-cli` MCP (Model Context Protocol) server.

## Layout

```
zyfai/                            Zyfai's integration code. Zyfai-owned; Cork does not edit it.
INTEGRATION.md                    The runbook: the path, the trust boundary, the checklist.
CLAUDE.md                         Context for agents working in this repository.
.claude/skills/cork-integration/  Agent skill for operating the pinned Cork Distribution.
```

## Moving from v0.4-rc.1

This cut keeps the Phoenix contracts and the generation labels. It changes
three components. Two of them reach you.

- **Phoenix `1.4.0-rc.1`, Rollover `0.2.0` and `cork-periphery v0.2.0-rc.1`:
  unchanged.** Your pools, the cover you hold and both your adapters keep
  working. We checked your second adapter with the released `v0.7.0-rc.2`
  binary on 2026-10-08 (`ch track` mode `verify`, subject `forSelfAdapter`):
  it verifies, and its bindings name the primary generation.
- **Market Registry `0.5.0` → `0.6.0`: one contract changed.** The JIT
  adapter (`CorkLimitOrderAdapter`, `version()` `0.5.0`) now takes the share
  token's permit as one `bytes` signature, so a Safe can sign it. You fill and
  sign nothing on the JIT path, so your route does not change. One effect
  reaches you: a resting JIT order whose permit uses the old `v, r, s` form
  cannot fill through the new adapter. Its maker must sign it again. If a
  quoted order fails the fill pre-flight, tell the underwriter.
- **`cork-api` `0.4.3` → `0.4.6`.** The venue serves RFQ v2 (`/rfqs/v2`):
  signed writes, a `fixed_rate` mode, and rollover RFQs. RFQ v1 is still
  served on its own path, but the new tool does not use it.
- **`cork-cli` `0.6.0` → `0.7.0-rc.2`, breaking on the RFQ path.** From the
  pinned [CHANGELOG](https://github.com/Cork-Technology/cork-cli/blob/v0.7.0-rc.2/CHANGELOG.md)
  (sections `0.7.0-rc.2`, `0.7.0-rc.1`, `0.6.1-rc.*`):
  1. **RFQ v1 is removed.** Every RFQ read and write goes to `/rfqs/v2`. An RFQ
     opened on v1 does not show in a v2 read.
  2. **`rfq-open` requires `kind`:** `new_position` (cover for a new position)
     or `rollover`. There is no default.
  3. **Every RFQ write is signed.** `rfq-open` takes `auth`. Build the write
     with `ch prepare order rfq-write`, sign its `data.typedData` with your
     Safe in your own stack, and pass
     `--auth '{"method":"signature","signature":"0x…"}'`. The tool rebuilds the
     body, recovers the signer and checks your Safe's `isValidSignature`
     before it relays. The other method, `{"method":"apiKey"}`, needs a venue
     API key; ask Cork before you use it.
  4. **Rollover, filler side.** `ch prepare order rollover-fill` builds the
     fill of a resting rollover order. In a rollover you are the filler; see
     the runbook's "Rolling cover".
  5. **New local commands you do not need.** `ch wallet` and `ch sign` keep an
     EOA key in a local keystore. Your Safe signs in your own stack; keep it
     there.
  6. **Every `ch` command and flag your wrapper calls is unchanged.** We
     checked each one in `zyfai/src/services/cork-cli.ts` against the
     `v0.7.0-rc.2` binary on 2026-10-08. `zyfai/` needs no change for this cut.
- **The hosted MCP stays on `0.6.0`** until `0.7.0` final. It speaks RFQ v1
  only. Run the released binary (`ch`, or `ch mcp` over stdio).

The Distribution owner waived the cross-component integration suite again for
this cut (deviation D10, 2026-10-08). Read the manifest's deviations and
`knownIssues` before you trade.

## Moving from v0.3-rc.1

This cut is **breaking on every covered surface**. The previous two pins kept
every contract address; this one does not. What changed:

- **Contracts: a second generation, side by side with the first.** Phoenix
  `1.3.0-rc.1` → `1.4.0-rc.1`, Market Registry `0.3.3` → `0.5.0`, Rollover
  `0.1.0-rc.2` → `0.2.0` and the ForSelf reference adapter are **new
  deployments on both chains**. The `v0.3-rc.1` contracts are not retired,
  paused or upgraded: every pool the venue lists today lives on them, and the
  tool keeps them readable and preparable as the generation
  `phoenix/v0.3-rc.1`. Three contract facts matter to an integrator:
  1. A pool on the new Phoenix has a **10-field** market struct: the two fees
     are part of the pool id, and the fee bound is strictly below 100% instead
     of 5%.
  2. The new Market Registry speaks a **nested** wire: the JIT payload carries
     `extraData` (the old `additionalData`) and an `oracleSalt`, denominations
     are keyed by unit address, and `CorkMarketCreator` now lives in the
     registry package. The registry holds registered assets on both chains;
     read the live list with `ch query registry-assets`.
  3. Rollover `0.2.0` adds `oracleSalt` to the JIT commitment, which changes
     the signed typehash; the order data itself is unchanged, and the venue
     admits both settler generations. It is still **pinned, not proven**, and
     its routes are still outside the covered API surface.
- **`cork-periphery` `0.1.3-rc.1` → `0.2.0-rc.1`.** The ForSelf contracts —
  `CorkForSelfAdapter` included — are **unchanged since `v0.1.1`**: same
  entrypoints, same constructor `(cork, whitelistManager, lop)`, same
  selectors. The package drops `CorkMarketCreator` and ships **no deployment
  scripts**; you deploy with your own reviewed tooling. Cork's reference
  adapter for the new generation is a fresh deployment bound to the new pool
  manager, recorded in the manifest so you can diff yours against it.
- **`cork-cli` `0.4.1` → `0.6.0`, breaking.** Install the new tag; the
  release-gate evidence is in the manifest. What a demand-side integrator
  meets, from the pinned
  [CHANGELOG](https://github.com/Cork-Technology/cork-cli/blob/v0.6.0/CHANGELOG.md)
  (sections `0.6.0`, `0.5.1`, `0.5.0`):
  1. **Generations.** A chain hosts a set of contract generations, one of them
     primary (`phoenix/v0.4-rc.1` on both chains). Every chain-backed tool
     takes an optional `generation` — a label, or the aliases `primary` and
     `previous`. A read or prepare **by pool id follows the pool's own
     generation**, resolved from the chain; every result names it in
     `data.generation`. `protocol-config` lists every generation. A pool no
     pool manager knows is `pool_not_found`, not `chain_read_failed`.
  2. **Positions in one read.** `ch query account-state --account <safe>`
     with no pool id lists the account's cST/cPT positions across every
     generation, scanned from the chain in one request, each row with its
     generation and expiry. `cork_capabilities topic:"migration"` is the
     exit-and-re-enter recipe.
  3. **The book is ranked by default.** `ch query orderbook` returns fillable
     rows for the fill sender named in `filters.account`, best price first;
     rows it cannot fill ride under `excluded` with a reason. **Pass your
     adapter as the account** — it is the address that calls the LOP on your
     route. `sort: "venue"` restores the old newest-first list.
  4. **Reserved orders.** A maker can reserve an order for one fill sender
     (`allowedSender`); rows carry `exclusivity`, and a fill prepare for the
     wrong sender refuses `private_order` with no bytes.
  5. **Refusals that protect a fill.** A resting order whose extension names a
     contract the tool has not pinned is refused (`foreign_extension_target`);
     an order whose maker cannot deliver is excluded (`maker_not_ready`); every
     venue row is re-verified — extension bytes and maker signature — the way
     the fill verifies it.
  6. **Renames.** `additionalData` → `extraData` on every JIT input (the old
     name is accepted with a deprecation notice); `oracleSalt` and the two fee
     percentages join the JIT and derive inputs; `cork-defaults.v2.json` is
     the address surface this tool reads.
  7. **`0.5.0` security remediation.** Atomic bundles only (no pre-funded
     mode), every decoded leg verified against the address book, bounded HTTP
     admission, same-origin venue redirects, and `self-update` that never
     downgrades silently.
- **`cork-api` `0.4.0` → `0.4.3`.** Adds the Registry v2 interface for Market
  Registry 0.5.0. `/registry/v1` is still served for the 0.3.3 registry but is
  outside the covered surface. The tool reads registries on chain, never
  through that route, so this repository has nothing to re-point.

## What this means for your adapter

Written for the move to the new generation (`phoenix/v0.4-rc.1`, primary since
2026-09-24). It still holds under `phoenix/v0.5-rc.1`.

1. **Nothing you run today breaks.** Your deployed `CorkForSelfAdapter`, the
   cover you hold, and `buy:cover` against an existing market all live on
   `phoenix/v0.3-rc.1`. The pinned tool resolves that generation from the pool
   id and builds the same artifacts it built before. Verified against the
   released `v0.6.0` binary on 2026-09-24: your Safe's position reads across
   both generations, and the exercise artifact for it builds with the same
   shape your wrapper expects.
2. **Your adapter cannot serve a market on the new generation.** An adapter's
   binding is immutable: its `CORK()` names the `phoenix/v0.3-rc.1` pool
   manager, and the constructor requires the whitelist manager of the same
   pool manager. A fill or exercise on a `phoenix/v0.4-rc.1` pool through it
   is refused by the tool (`adapter_binding_mismatch`) before any bytes exist.
3. **To trade on the new generation, deploy a second adapter.** You did this
   on 2026-09-30 and filled your second cover through it the same day. It
   still serves the primary under `phoenix/v0.5-rc.1`. Same source
   (`cork-periphery v0.2.0-rc.1`, unchanged ForSelf code), same selectors,
   new constructor arguments: the new pool manager, the new whitelist manager,
   the same LOP — all read live from `ch query protocol-config`. Whitelist the
   new address with the same two selectors, and point the buy script at it.
   One `CORK_FOR_SELF_ADAPTER` per generation. Keep the old adapter whitelisted
   until your last `v0.3-rc.1` cover has been exercised or has expired.

## Notices and security

- **Change notice channel:** GitHub Releases on each public repository the
  manifest pins. Watch them; deprecations and removals are announced there.
- **Security contact:** security@cork.tech.

## What happened to the hackathon code

Until August 2026 this repository carried the July hackathon package: contracts
(`CorkMarketCreator`, an early LOP adapter), examples, and an agent skill with a
hardcoded address book. All of it described deployments that no longer exist and
entrypoints that were deleted upstream. It was removed rather than patched, and
remains reachable in git history — **do not integrate against anything found
there**, and do not let an agent resurrect it from history.
