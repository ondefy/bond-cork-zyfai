# CLAUDE.md — agent context for this repository

## What this repository is

Zyfai's workspace for integrating Cork cover, pinned to Distribution
**`phoenix/v0.5-rc.1`** with cork-cli **`v0.7.0-rc.2`**. `zyfai/` is Zyfai's own
code. `INTEGRATION.md` is the runbook — read it before doing any integration
work. `README.md` maps the pinned reference material.

Two names, two things. The **Distribution** is `phoenix/v0.5-rc.1`: the set of
component versions in the manifest. The **contract generation** the tool prints
in `data.generation` is still `phoenix/v0.4-rc.1` (primary) or
`phoenix/v0.3-rc.1` (previous), because this Distribution keeps the same
Phoenix pool manager. Do not "correct" one name to the other.

## Hard rules

1. **The manifest is the authority.**
   [`distributions/phoenix/v0.5-rc.1.json`](https://github.com/Cork-Technology/distribution/blob/main/distributions/phoenix/v0.5-rc.1.json)
   in `Cork-Technology/distribution` names every version, address, codehash and
   known issue. Where any document, code comment or memory disagrees with it,
   the manifest wins. Read living fields (`stage`, `reviewLevel`, `status`,
   `knownIssues`) fresh from the repo, never from a copy.
2. **Never write a contract address into this repository** — not in code, not
   in docs, not in comments. Read addresses live (`ch query protocol-config`
   via the cork-cli MCP) or from the manifest at the moment of use. If you find
   a hardcoded address here, that is a bug: remove it and cite the live source
   instead.
3. **Never resurrect anything from git history.** Pre-August history holds the
   hackathon package: deleted entrypoints (`CorkMarketCreator.createMarket`),
   retired adapter generations, dead addresses, and a stale `cork-operations`
   skill. None of it is a valid reference for any question about Cork.
4. **Signing stays in Zyfai's stack.** The cork-cli tooling reads state, does
   deterministic math, and builds *unsigned* artifacts; only `cork_submit`
   relays a signed payload, and nothing in it ever holds a key. Do not build or
   propose flows where the tooling signs or custodies funds.
5. **Do not edit `zyfai/` unless the task explicitly asks for it.** It is
   Zyfai's production-path code, not shared scaffolding.
6. **Name the generation you act on.** A chain hosts two active contract
   generations: `phoenix/v0.4-rc.1` (the primary) and `phoenix/v0.3-rc.1` (the
   previous one). A read of an existing pool follows the pool's own
   generation. A prepare targets the primary unless you pass `generation` (a
   label, or the alias `previous`). Every result names the generation it
   answered from in `data.generation`; check it before you act. Zyfai runs one
   `CorkForSelfAdapter` per generation: the first is bound to
   `phoenix/v0.3-rc.1`, the second to the primary. Each serves only its own
   generation. The tool refuses a wrong pairing (`adapter_binding_mismatch`);
   do not work around the refusal.
7. **Every RFQ write is signed.** cork-cli `0.7` speaks venue RFQ v2 only:
   `rfq-open` needs `kind` (`new_position` or `rollover`) and `auth`. The
   signature comes from Zyfai's Safe, in Zyfai's stack. Never move a Safe
   signature into a tool keystore.
8. **In a rollover, Zyfai is the filler.** The cPT holder signs a rollover
   order; the cST holder fills it, pays the premium and receives the new cST.
   A rollover RFQ is opened by the cPT holder. Zyfai's side is
   `rollover-fill`, which is not a ForSelf route.

## Operating Cork

Use the [`cork-integration`](./.claude/skills/cork-integration/SKILL.md) skill.
It wires the cork-cli MCP server (pinned tag `v0.7.0-rc.2`) and carries the
decision rules. The tool surface is self-documenting: start any unfamiliar task
with `cork_capabilities`, not with a guess.

## Chains

Deployed on Arbitrum One (42161) and Base (8453) at identical addresses.
**Base first** — it is the proven partner path. Never infer the chain from an
address; select it explicitly.
