# ONA Synthetic Benchmark — Changes from v5 to v6

v6 is a correctness release. The scenario design, topology families, layer model, node attributes, and export contracts are unchanged from v5. All changes are internal to the generation pipeline, correcting six silent bugs that were causing structurally incorrect output without producing any visible errors.

---

## Bug fixes

### 1. HierCorp informal tie acceptance logic was inverted

**v5 problem:** Within-department informal ties were accepted using `infRng() > avgLv * 1.8`, where `avgLv = (level_i + level_j) / 12`. For two level-6 executives, `avgLv = 1.0`, so the rejection condition was `rng > 1.8` — which is never true, since `rng ∈ [0, 1)`. Senior pairs were accepted unconditionally. For two level-1 ICs, `avgLv = 0.167`, so the rejection fires when `rng > 0.30`, rejecting ~70% of pairs. The intent was seniors more likely to connect; the actual behaviour was the opposite: ICs were heavily filtered while executives were never filtered.

**v6 fix:** Acceptance probability is now explicit: `p_accept = 0.15 + 0.60 × (avgLevel / 6)`. Two level-6 executives: p = 0.75. Two level-1 ICs: p = 0.25. The relationship is monotonically correct and the formula is readable.

---

### 2. Location overlay loop added at most one edge per department per location

**v5 problem:** The same-department location overlay loop ran up to `target × 3` iterations but `break`ed on the first successful edge insertion. Regardless of how many edges the target calculation specified, at most one edge was ever added per department per location. The overlay was therefore almost entirely decorative — it contributed negligible structural signal to the CHRO_M location homophily finding.

**v6 fix:** The loop now runs a proper rejection-sampling `while (found < target)` structure with a `found` counter, matching the pattern used by all other edge-generation loops in the file. The cross-department location overlay had the same bug and is also fixed.

---

### 3. Trust layer acceptance probabilities double-counted level checks

**v5 problem:** The acceptance probability computation used two independent `if` statements for level difference: `if (lvlDiff === 0) p += 0.15` and `if (lvlDiff <= 1) p += 0.08`. Because these were not `else if`, a pair with `lvlDiff === 0` received both bonuses (+0.23 total), while a pair with `lvlDiff === 1` received only the second (+0.08). The level-0 case was unintentionally double-weighted.

**v6 fix:** Rewritten as mutually exclusive tiers: `if (lvlDiff === 0) +0.18; else if (lvlDiff === 1) +0.08; else if (lvlDiff >= 3) −0.06`. Each pair falls into exactly one tier. Base probability also adjusted from 0.10 to 0.06 to keep aggregate density consistent after removing the double-count.

---

### 4. Mentorship direction was inconsistent with the reporting layer convention

**v5 problem:** The independent mentorship generator pushed edges as `{from: mentee, to: mentor}` (junior to senior), with a comment saying "direction: mentee → mentor". But `orientEdge` for the `mentorship` layer swaps edges where `fromLevel < toLevel`, producing senior → junior in the output. The independent generator bypassed `orientEdge` entirely, so the mentorship layer had the opposite direction to what `orientEdge` would have produced — and the opposite direction to the `reporting` layer, which correctly uses `orientEdge` (manager → report = senior → junior).

**v6 fix:** The mentorship path in `generateLayers` now applies `orientEdge` after generating raw edges, exactly as the trust layer does. This ensures mentor → mentee direction (senior → junior) is consistent with the `reporting` and `decision_influence` conventions. The raw generator still produces `{from: mentee, to: mentor}` as input; `orientEdge` flips it.

---

### 5. Dead entries in LAYER_KEEP_PROB and HUB_BOOST

**v5 problem:** `trust: 0.54` and `mentorship: 0.25` remained in `LAYER_KEEP_PROB`, and `trust: 0.30` and `mentorship: 0.20` remained in `HUB_BOOST`. Since v5 introduced independent generators for these two layers, both lookup tables are never consulted for them. The dead entries misled readers into thinking trust and mentorship were still derived from the base topology.

**v6 fix:** Entries removed from both tables. Comments added to both tables noting that trust and mentorship use independent generators.

---

### 6. Dead DEMO_M block in buildSBMMatrix

**v5 problem:** `buildSBMMatrix` contained a special case for `DEMO_M` that set `block[0][1] = 0.02`. But `DEMO_M` uses `hier_corp` topology, not `sbm` — `buildSBMMatrix` is never called for it. The code created a false impression that the DEMO_M silo was calibrated at the SBM level when the actual silo signal comes from the organic department separation in the hierarchical tree.

**v6 fix:** The dead DEMO_M block removed. A comment added to the function noting which scenarios it is actually called for.

---

### 7. Reporting layer did not produce a hierarchical star layout

**v5 problem:** The force simulation used identical physics for all layers and all scenarios — `forceLink(distance=42/88)` and `forceManyBody(−72)`. When the reporting layer was soloed, D3 saw a pure tree graph but had no instruction to lay it out as one. The CEO ended up wherever spring forces and repulsion happened to push it. The result was an irregular blob indistinguishable from any other sparse graph, with no visible radial star pattern.

A secondary issue: `hierarchy_depth` (the distance of each node from the CEO root) was only written to node objects for `RESIZE_M`. For `DEMO_M`, `REORG_M`, and `AI_M` — which also use `hier_corp` — the attribute was zero for every node, so even a radial layout would have had nothing to anchor to.

**v6 fix:** A new `applySimulationForces` function detects when the only active layer is `reporting` and switches to a hierarchy-aware force configuration:

- `forceRadial` pulls each node to a concentric ring at radius `hierarchy_depth × 95px`. The CEO sits at the centre (r = 0), department VPs at r = 95, senior managers at r = 190, and so on out to r = 540 for deepest ICs.
- Link strength is raised from 0.35 to 0.90 so parent–child bonds hold the tree shape firmly.
- Charge strength scales with depth (`−30 − depth × 8`) so the larger IC population at outer rings spreads evenly around its ring without collapsing inward.
- Before the simulation reheats, nodes are pre-positioned onto their target rings at evenly-spaced angles. This prevents the simulation from having to unwind from a tangled force-directed blob and makes convergence fast.
- When switching back to any other layer combination, all radial forces are removed and standard force-directed physics is restored.

`hierarchy_depth` is now written to every node produced by the `hier_corp` generator, regardless of scenario.

---

## What did not change

- All scenario IDs, titles, descriptions, and default parameters
- All node attributes and their generation logic
- Topology generators (BA, ER, WS, SBM, HierCorp)
- Layer model, edge weight logic, and orientation conventions
- Export format for nodes CSV, edges CSV, GraphML, metrics JSON, and payload JSON
- Visual design and UI
- Validation checks
