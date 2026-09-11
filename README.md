# cloud-itonami-isic-1629: Manufacture of other products of wood; manufacture of articles of cork, straw and plaiting materials

Open Business Blueprint for **ISIC Rev.5 1629**: manufacture of other
products of wood; manufacture of articles of cork, straw and plaiting
materials — an autonomous "actor" (LLM advisor behind an independent
Governor, langgraph-clj StateGraph, append-only audit ledger) that
coordinates back-office wood/cork/straw-products-shop **plant
operations**: production-batch data logging (product-spec/unit-count/
output-quality for wooden tools/handles, cork stoppers, and wicker/
basketry items), cutting/molding/weaving-equipment maintenance
scheduling, materials-safety/equipment-safety concern flagging, and
outbound product shipment coordination.

This repository designs a forkable OSS business for wood/cork/straw-
products-shop plant operations: run by a qualified operator so a
wood/cork/straw-products shop (wooden tools/handles turned on a
cutting lathe, cork stoppers formed on a molding press, wicker/
basketry items formed on a weaving loom) keeps its own operating
records instead of renting a closed SaaS.

## What this actor does

Proposes **plant operations coordination**, not equipment operation:
- `:log-production-batch` — product batch product-spec/output-quality data logging (administrative, not an operational decision)
- `:schedule-maintenance` — cutting/molding/weaving-equipment maintenance scheduling proposal
- `:flag-safety-concern` — surface a materials-safety/equipment-safety concern (always escalates)
- `:coordinate-shipment` — outbound product shipment coordination proposal

## What this actor does NOT do

**CRITICAL SCOPE BOUNDARY — this is a safety-critical domain**
(lathe/molding-press/loom blade, pinch-point, and repetitive-motion
injury risk, and wood-dust fire/explosion hazard):

- Does NOT control cutting, molding, or weaving equipment directly
- Does NOT make plant-safety or hazard decisions (that's the plant supervisor's exclusive human authority)
- Does NOT authorize or finalize a cutting/molding/weaving-line run (human plant supervisor decides)
- ONLY proposes/coordinates operations back-office; all actuation requires explicit human approval
- Safety-concern flagging ALWAYS escalates — never auto-decided, no confidence threshold or phase below escalation

## Architecture

Classic governed-actor pattern (`woodcork.operation/build`, a langgraph-clj StateGraph):
1. **`woodcork.advisor`** (sealed intelligence node, `WoodCorkStrawAdvisor`): proposes decisions only, never commits
2. **`woodcork.governor`** (independent, `Wood, Cork & Straw Products Shop Plant Operations Governor`): validates against domain rules, re-derived from `woodcork.registry`'s pure functions and `woodcork.store`'s SSoT -- never trusts the advisor's own self-report
   - HARD invariants (always `:hold`, no override):
     - Shop/batch record must be independently verified/registered (`:verified?` AND `:registered?`) before any action is taken against it (equipment before maintenance scheduling, batch before shipment coordination)
     - The request's own `:effect` must be `:propose` (never a direct-write bypass)
     - `:op` must be in the closed four-op allowlist
     - The proposal's own `:effect` must be one of the four propose-shaped effects (no direct cutting/molding-line-equipment control)
     - Finalizing a cutting/molding/weaving-line run (`:finalize? true`) is a PERMANENT, unconditional block
     - A shipment may not push a batch's own recorded shipped unit count past its own logged production unit count (independently recomputed)
     - No double-scheduling the same maintenance record
     - No fabricated `:product-spec` value on a production-batch patch
     - No physically implausible `:output-quality-percent` value on a production-batch patch
   - ESCALATE (always human sign-off, overridable by a human):
     - `:flag-safety-concern` always escalates, regardless of confidence
     - Low-confidence proposals
3. **`woodcork.phase`** (Phase 0->3 rollout): `:schedule-maintenance`/`:flag-safety-concern`/`:coordinate-shipment` are NEVER in any phase's `:auto` set (permanent, matching the governor's own posture); only `:log-production-batch` may auto-commit at phase 3 when clean
4. **`woodcork.store`** (append-only audit ledger + SSoT): a single `MemStore` backend behind a `Store` protocol (see ns docstring for why a second Datomic-backed backend is out of scope for this build)

## Development

```bash
# Run tests (top-level deps.edn already pins langgraph+langchain local/root)
kbb -M:test

# Run tests via the workspace :dev override alias (equivalent, kept for sibling-repo parity)
kbb -M:dev:test

# Run the demo
kbb -M:dev:run

# Lint
kbb -M:lint
```

## Status

`:implemented` — `governor.cljc`/`store.cljc`/`advisor.cljc`/`registry.cljc` + `deps.edn` complete the module set; tests green, demo runnable, langgraph-clj integration verified.

## License

AGPL-3.0-or-later
