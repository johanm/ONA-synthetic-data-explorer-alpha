# ONA Synthetic Benchmark — v6

An interactive, browser-native tool for generating and visualising synthetic Organisational Network Analysis (ONA) benchmark datasets. Open `ona_benchmark_v6.html` in any modern browser. No install, no server, no dependencies beyond the single file.

---

## Origin and Credits

The scenario design, generation logic, topology families, relational layer model, industry templates, and export contracts in this tool are based on the work of:

**Silvia I. Fierăscu, PhD**
West University of Timișoara
in collaboration with the Network First Manifesto (NFM) Community

Silvia's original work is a self-contained R Markdown protocol — *Synthetic ONA Benchmark Dataset: A Parameterised Protocol for Generating Synthetic Organisational Network Benchmark Datasets, v3* — which serves as both the annotated specification and an executable R generator for the same scenarios. v6 of this HTML tool is a faithful reimplementation of that specification, with a series of structural improvements described at the end of this document.

The interactive HTML visualiser was developed by:

**Johan Myrberger**
[linkedin.com/in/myrberger](https://www.linkedin.com/in/myrberger/)

---

## Licence

Released under Creative Commons Attribution 4.0 International (CC BY 4.0). You are free to share and adapt the material for any purpose, including commercial use, provided you give appropriate credit to both the original protocol author (Silvia I. Fierăscu / NFM Community) and the HTML implementation author (Johan Myrberger), and indicate if changes were made.

Full licence text: [creativecommons.org/licenses/by/4.0](https://creativecommons.org/licenses/by/4.0)

---

## What Is ONA?

Organisational Network Analysis maps how work actually happens inside a company — who collaborates with whom, who seeks advice from whom, who trusts whom, where information silos form, where brokers carry too much load, and how change moves through the organisation. Unlike org-chart analysis, ONA operates on the informal relational structure that drives productivity, innovation, and resilience — the network that exists between the lines of the hierarchy.

Synthetic benchmark datasets serve four purposes:

- **Method benchmarking** — test and compare ONA algorithms against controlled, reproducible ground truth with known structural properties
- **Privacy-safe demonstration** — show ONA tooling and findings to clients without using real employee data
- **Scenario modelling** — explore what a merger, reorg, or AI adoption might look like structurally before collecting real survey data
- **Platform validation** — confirm that a platform produces expected outputs on well-specified inputs before going live with real data

---

## A Note on Realism and Applicability

The networks produced by this tool are synthetic by design. They are constructed from statistical models and calibrated parameters, not from observed relational data.

**Topological plausibility, not empirical accuracy.** The five topology families produce networks whose structural properties — clustering coefficients, degree distributions, hierarchy depth, hub concentration — resemble those found in real organisational networks. They do not reproduce the specific dynamics of any particular organisation or industry.

**Synthetic outcomes are benchmarks, not predictions.** The outcome variables (`engagement_score`, `burnout_risk`, `innovation_score`, and scenario-specific scores) are derived from structural position using plausible but simplified assumptions. They are appropriate for testing visualisation and algorithm behaviour — not for drawing conclusions about real people.

**Scenario realism is structural, not causal.** Each scenario is tuned to produce a network with a specific, findable structural signal relevant to the stated business question. Whether that signal accurately models the dynamics of a real merger, reorg, or culture change programme is a matter of expert judgement.

---

## Quick Start

1. Download `ona_benchmark_v6.html`
2. Open in Chrome, Firefox, Safari, or Edge (version 120+ recommended)
3. Select a scenario from the left panel
4. Adjust node count, random seeds, and active layers if desired
5. Click **Generate**

The file is fully self-contained. No internet connection is required after download.

---

## Network Topologies

v6 supports five topology families. Each scenario specifies a default topology; it can be overridden manually.

### Erdős–Rényi (`er`)

Every pair of nodes is connected independently with probability `p`, which scales with network size (`p = 0.07` for N ≤ 100, `p = 0.04` for larger networks). The resulting degree distribution is Poisson — no hubs, no clustering above chance. Used as the null-model baseline for the `RESEARCH_SM` scenario. Appropriate whenever the research question requires a known random baseline to contrast against structured topologies.

### Watts–Strogatz (`ws`)

Begins as a ring lattice where each node is connected to its `k` nearest neighbours, then rewires each edge with probability `β = 0.12`. The result is a small-world network: high local clustering (similar to a lattice) but short average path length (similar to a random graph). Degree distribution is narrow and approximately normal. Used for `DEV_S`.

### Barabási–Albert (`ba`)

Preferential attachment model. New nodes arrive sequentially and attach to `m = 5` existing nodes with probability proportional to `degree^1.3 × seniority_fitness`, where `seniority_fitness` ranges from 1.0 (individual contributor) to 3.5 (executive). The result is a scale-free degree distribution with a pronounced fat tail: a small number of high-degree hubs, a large number of peripheral low-degree nodes. An additional 15% of low-fitness peripheral edges are rewired toward high-fitness nodes post-construction to amplify the hub-and-spoke pattern. Used for `SUCCESSION_M`.

### Stochastic Block Model (`sbm`)

Nodes are partitioned into blocks by department. Within-block connection probability is higher than between-block probability, producing the clustered-with-silos structure characteristic of most organisations. v6 uses a **degree-corrected** SBM: each node receives a sociability score `θ_i` drawn from a Pareto distribution (α = 1.5) multiplied by a seniority factor (executive = 2.6×, individual = 1.0×). The connection probability between nodes `i` and `j` is `p_ij = block[bi][bj] × θ_i × θ_j`, normalised per block. This preserves block structure while producing a fat-tailed degree distribution within each block — matching the empirical finding that communication networks within departments are not Poisson. Used for `CHRO_M`, `MA_M`, and `CULTURE_M`, each with scenario-specific block matrix calibrations.

### Hierarchical Corporate (`hier_corp`)

A purpose-built topology that produces realistic corporate hierarchies overlaid with informal ties. Generated in three phases:

**Phase 1 — Formal tree.** A single global CEO node is identified (highest level + tenure). Each department's most senior node reports to the CEO. Nodes cascade downward using level-dependent span-of-control: executive nodes have 3–4 direct reports; senior managers 5–7; managers 8–10; supervisors 10–12. For `RESIZE_M`, spans are deliberately narrowed to 2–4 to simulate an over-layered organisation. Every node receives `hierarchy_depth` (distance from CEO root, 0 = CEO) and `span_of_control` (actual direct report count) attributes from the constructed tree.

**Phase 2 — Informal within-department ties.** A sparse set of additional undirected edges is added within each department. Target count: `min(√dept_size × 2, 40)`. Acceptance probability is `0.15 + 0.60 × (avgLevel / 6)` — senior pairs are more likely to connect informally (p ≈ 0.75 for two level-6 nodes) than IC pairs (p ≈ 0.25 for two level-1 nodes).

**Phase 3 — Cross-department leadership bridges.** ~8% of level-4+ nodes receive an additional cross-department informal tie, creating the integrator-hub pattern visible in real executive networks.

**Visualisation.** When the reporting layer is soloed, the visualiser switches to a hierarchy-aware radial layout. Each node is pulled to a concentric ring at `hierarchy_depth × 95px` from the centre. The CEO sits at the centre, department VPs on the first ring, and so on outward. Link strength is raised to 0.90 and charge scales with depth so the larger IC population at outer rings spreads cleanly around its ring. When any other layer combination is active, the standard force-directed layout is restored.

Used for `DEMO_M`, `REORG_M`, `AI_M`, and `RESIZE_M`.

---

## Relational Layers

Each network is multiplex: multiple relationship types (layers) are generated on the same set of nodes. Active layers are configurable per scenario.

| Layer | Type | Direction convention | What it captures |
|---|---|---|---|
| `communication` | Undirected | — | Day-to-day information exchange |
| `trust` | Undirected | — | Psychological safety and reliance |
| `collaboration` | Undirected | — | Joint work on tasks and projects |
| `innovation` | Undirected | — | Idea-sharing and creative exchange |
| `energy` | Undirected | — | Who energises or drains |
| `advice` | Directed | seeker → advisor | Who seeks expertise from whom |
| `reporting` | Directed | manager → report | Formal management line |
| `mentorship` | Directed | mentor → mentee | Developmental guidance |
| `decision_influence` | Directed | influencer → recipient | Who shapes decisions |
| `tool_interaction` | Directed | human → agent | Human-to-AI-agent workflows |

### Layer generation

All layers except `trust` and `mentorship` are derived from the base topology by degree-dependent edge retention. The keep probability for each edge scales as `base_keep × (1 + hub_boost × mean_degree_norm)`, where `mean_degree_norm` is the mean of the two endpoint degrees normalised by the maximum degree in the network. `hub_boost` varies by layer: `communication` (0.60), `collaboration` (0.50), `decision_influence` (0.55), `innovation` (0.40), `energy` (0.25). This preserves the hub structure of the base graph independently in each layer.

**Trust** is generated by an independent sparse graph model, not derived from the base topology. Acceptance probability between any pair: base 0.06, +0.28 for same department, +0.10 for same location, +0.18 for same seniority level, +0.08 for adjacent levels, −0.06 for a gap of 3+ levels. Target average degree ≈ 2.5. The independence means a peripheral communicator can have deep trust ties that don't appear in the communication layer — which matches the empirical structure of trust networks.

**Mentorship** is also generated independently. Only level-4+ nodes can be mentors; ~40% of ICs and managers receive one. 30% of mentorship ties cross department boundaries. Direction is mentor → mentee (senior → junior), consistent with the `reporting` layer convention. Average degree ≈ 1.2.

Edge weights are context-sensitive: within-department edges receive a weight drawn from Beta(4, 2) (~0.65 mean); cross-department edges from Beta(2, 4) (~0.35 mean); cross-location and cross-legacy edges are further adjusted per layer.

---

## Location Homophily Overlay

After the base topology is generated, a location homophily overlay adds edges between nodes who share a physical location. Within-department same-location pairs receive additional edges at a rate of ~7% of possible pairs. Cross-department same-location pairs receive additional edges at ~1% of possible pairs. This is applied to all topologies and is the primary structural mechanism in the `CHRO_M` location homophily finding.

---

## Scenario Catalog

### `DEMO_M` — Executive Demo: Silos & Brokers

**Default N:** 300 · **Topology:** `hier_corp` · **Layers:** communication, advice, trust

The go-to first-meeting scenario. A `tech_product` template organisation with a formal hierarchy plus two informal layers. Designed to surface three findings in sequence: departmental silos (E-I index approximately −0.25 to −0.35), key brokers connecting otherwise isolated clusters, and articulation-point fragility — the risk posed by nodes whose removal disconnects the network. Compact at 300 nodes so it renders in under a second and reads comfortably on a laptop screen. The hierarchy is visible as a radial tree; the informal communication and advice layers overlay it as shorter-range connections.

---

### `CHRO_M` — Privacy-Safe CHRO Pilot

**Default N:** 500 · **Topology:** `sbm` · **Layers:** communication, trust, collaboration, energy

For HR analytics conversations where real data cannot be used. A `professional_services` template with boosted Remote location probability (~45% of nodes). The degree-corrected SBM produces the department clustering; the location homophily overlay creates a second, cross-cutting cluster structure based on physical presence. Remote workers form a discernible collaboration cluster separate from on-site workers — the hybrid-work connectivity gap. The energy layer captures informal vitality ties. No demographic attributes are included; the findings are structural.

---

### `REORG_M` — Reorg Lab: Team Merge & Split

**Default N:** 500 · **Topology:** `hier_corp` · **Layers:** reporting, communication, collaboration, trust

Simulates an organisation mid-reorganisation. Every node carries `team_before` (original department) and `team_after` (one of 4–6 new cross-functional squads). Within-new-team informal ties are dense; collaboration and trust edges crossing the new squad boundary are thinned by 55% to represent the collaboration debt that builds up after a structural change. The reporting layer shows the new formal structure; the trust layer shows where the old structure lives on — people still trust their former colleagues, regardless of which squad they now belong to. Outcome variable: `transition_stress`.

---

### `RESEARCH_SM` — Reproducible Research Benchmark

**Default N:** 150 · **Topology:** `er` · **Layers:** communication, trust, advice, collaboration

A clean null-model baseline for method comparison. Erdős–Rényi topology produces a known Poisson degree distribution. Four independent layers — including trust and advice, which are generated by their independent generators — give researchers a multiplex baseline for algorithm development and validation. Fully reproducible: same seeds produce identical networks. Scaled to 150 nodes for fast computation and clean publication figures.

---

### `DEV_S` — Tool Builder Integration Pack

**Default N:** 120 · **Topology:** `ws` · **Layers:** communication, trust, innovation, mentorship

Compact and stable, designed for developers building ONA tooling. Watts-Strogatz small-world topology gives realistic clustering and short path lengths. Four layers exercise all three edge types: undirected (communication, trust, innovation) and directed with both conventions (mentorship: senior → junior; advice direction in trust is undirected here). Node IDs are zero-padded and stable across regeneration at the same seed. GraphML and JSON exports are well-formed and immediately loadable into igraph, NetworkX, or Gephi.

---

### `AI_M` — AI Adoption & Human-Agent Network

**Default N:** 500 · **Topology:** `hier_corp` · **Layers:** communication, advice, trust, innovation, decision_influence, tool_interaction

Models how AI tool adoption spreads through an organisation. Six non-human AI agent nodes are embedded in the network with `is_non_human: true`. High-adoption human nodes (top 30% by `ai_adoption_score`) connect to agents via directed `tool_interaction` edges. Nodes carry `ai_adoption_score`, `manager_enablement_score`, and `adoption_champion` (top 10% AI adoption) flags. The advice and decision_influence layers reveal whether champions are central or peripheral to how information and decisions actually flow — a common finding is that early AI adopters are technically central but politically peripheral. Outcome variable: `adoption_velocity`.

---

### `RESIZE_M` — Restructuring & Delayering

**Default N:** 500 · **Topology:** `hier_corp` · **Layers:** reporting, communication, advice, mentorship, trust

Exposes the cost of over-layered hierarchies. The HierCorp generator uses deliberately narrow spans: level 6 nodes have 2 direct reports; level 5 nodes have 2; level 4 nodes have 3; level 3 managers have 4. This produces 6–7 hierarchy levels in a 500-node network. Every node carries `span_of_control` (actual direct report count from the constructed tree) and `hierarchy_depth` (distance from CEO). `at_risk_role` flags nodes in Operations, Admin, Finance, and Quality at IC/manager level — the typical targets for consolidation. The gap between the formal reporting layer and the informal advice and trust layers shows where actual decision-making is happening, regardless of org chart position. Outcome variable: `overload_risk`.

---

### `MA_M` — Post-Merger Integration

**Default N:** 500 · **Topology:** `sbm` · **Layers:** communication, trust, advice, decision_influence, innovation

Two legacy organisations merged on paper but not yet in practice. Legacy assignment alternates by department: even-indexed departments form Legacy A, odd-indexed departments form Legacy B. This gives both legacy companies full functional coverage — they each had an Engineering, a Finance, a Sales, etc. — which is the realistic starting condition for most mergers. Within-legacy tie density is high; trust edges crossing the legacy boundary are thinned by an additional 40% on top of the sparse between-block SBM probability, reflecting the trust lag that persists 12–18 months post-merger. Nodes carry `legacy_company` and `integration_readiness`. The decision_influence layer frequently shows one legacy retaining disproportionate influence over decisions — a pattern that drives talent attrition from the acquired organisation.

---

### `CULTURE_M` — Culture Change & Collaboration Reset

**Default N:** 500 · **Topology:** `sbm` · **Layers:** communication, trust, energy, innovation, mentorship

For culture transformation engagements. The SBM between-block probability is set very low (0.015) — a strong departmental silo baseline. On top of this, nodes flagged as `culture_carrier` (top quartile of `change_readiness`, or belonging to strategy/product/design/engineering/client-services departments) generate additional cross-boundary `energy` ties. These ties create emergent communities of practice that cut across the formal block structure — visible when comparing the communication layer (siloed) against the energy layer (cross-cutting). The mentorship layer shows who culture carriers are developing: their sponsorship choices matter as much as their own cross-boundary ties.

---

### `SUCCESSION_M` — Succession Planning & Key-Person Risk

**Default N:** 500 · **Topology:** `ba` · **Layers:** decision_influence, advice, trust, communication, mentorship

A Barabási-Albert hub network calibrated to expose key-person risk. A small number of senior nodes concentrate betweenness and decision influence; removing the top three by betweenness visibly fragments the network. Nodes carry `key_person_risk_score` (computed from betweenness 45% + level 35% + performance 20%, calculated after network metrics are available) and `successor_readiness` (performance × change_readiness × level proximity). `is_key_person` flags nodes at or above the 80th level percentile. The mentorship layer shows who key persons are developing; the advice layer shows whether successor candidates are already in the information flow. A realistic succession finding: the most likely successors by readiness score are often not yet in the advice network of the person they would replace.

---

## Node Attributes

All nodes carry these attributes regardless of scenario.

| Attribute | Type | Notes |
|---|---|---|
| `id` | string | Zero-padded stable identifier: `p_00001` |
| `label` | string | Short display label: `P1` |
| `department` | string | From industry template |
| `seniority_level` | string | From industry template (e.g. `Senior`, `Director`) |
| `level` | int 1–6 | Numeric: 1 = IC, 6 = CEO |
| `role_bucket` | string | `individual`, `manager`, `senior_manager`, `executive` |
| `location` | string | From template: `HQ`, `Regional`, `Remote`, etc. |
| `hire_cohort` | int | Year of hire, 2017–2026 |
| `tenure_years` | float | Derived from `hire_cohort` — always internally consistent |
| `performance_score` | float 1–5 | Left-skewed normal, mean 3.2, SD 0.6 |
| `gender` | string | `female` (46%), `male` (50%), `non-binary` (4%) |
| `is_newcomer` | bool | `tenure_years < 1.0` |
| `ai_adoption_score` | float 0–1 | Pareto-ish beta, boosted for tech/strategy departments and senior roles |
| `change_readiness` | float 0–1 | Beta(3,2), slightly higher for short tenure |
| `attrition_risk` | float 0–1 | Beta(2,5), elevated for new joiners and high performers |
| `is_non_human` | bool | `true` for AI agent nodes in `AI_M` only |

Scenario-specific attributes are added on top. Key additions:

| Scenario | Extra attributes |
|---|---|
| `AI_M` | `adoption_champion`, `manager_enablement_score` |
| `MA_M` | `legacy_company`, `integration_readiness` |
| `REORG_M` | `team_before`, `team_after` |
| `RESIZE_M` | `at_risk_role`, `span_of_control`, `hierarchy_depth` |
| `SUCCESSION_M` | `is_key_person`, `key_person_risk_score`, `successor_readiness` |
| `CULTURE_M` | `culture_carrier` |

---

## Synthetic Outcome Variables

Computed after network metrics (betweenness, degree, closeness proxy) are available.

| Variable | Range | Scenarios | Primary drivers |
|---|---|---|---|
| `engagement_score` | 0–1 | All | Trust degree, closeness centrality, structural constraint |
| `innovation_score` | 0–1 | All | Communication degree, betweenness, AI adoption score |
| `burnout_risk` | 0–1 | All | Communication overload, betweenness, low trust degree |
| `adoption_velocity` | 0–1 | AI_M | AI adoption score × change readiness × network position |
| `overload_risk` | 0–1 | RESIZE_M | Burnout risk × normalised span-of-control |
| `transition_stress` | 0–1 | REORG_M | Burnout × (1 − change_readiness) × cross-squad displacement |

---

## Industry Templates

Four templates define the department mix, seniority distribution, and location split.

| ID | Sectors | Departments (sample) |
|---|---|---|
| `tech_product` | SaaS, platform, tech | Engineering, Product, Design, Data, Sales, Marketing, Finance |
| `professional_services` | Consulting, law, finance | Advisory, Client Services, Risk, Compliance, Strategy, Operations |
| `manufacturing` | Industrial, supply chain | Operations, Quality, Supply Chain, Engineering, Finance, HR, Admin |
| `healthcare` | Hospital, pharma | Clinical, Research, Operations, Compliance, Finance, IT |

---

## Export

| File | Format | Contents |
|---|---|---|
| `nodes.csv` | CSV | All node attributes including scenario-specific scores |
| `edges.csv` | CSV | All edges: `from`, `to`, `layer`, `weight`, `directed`, `same_dept`, `same_location`, `same_legacy` |
| `metrics.json` | JSON | Network-level metrics, per-department E-I index, component count, betweenness distribution |
| `platform_payload.json` | JSON | Complete portable package: nodes, edges, group metrics, outcomes, validation results |
| `network.graphml` | GraphML | Compatible with Gephi, igraph (R and Python), NetworkX, yEd |

---

## Validation

After each generation, a set of structural checks is run and displayed in the validation overlay:

- **Density 0.01–0.20** — confirms the network is neither trivially sparse nor near-complete
- **≥3 high-betweenness actors** — confirms hub or broker structure is present
- **E-I index variation (SD > 0.05)** — confirms meaningful between-group structural variation
- **Components ≤ 1% of N** — confirms the network is largely connected
- **Degree VMR > 1.5** — variance-to-mean ratio of the degree distribution confirms non-Poisson structure (expected for all topologies except `er`)

Scenario-specific checks (silo depth for `DEMO_M`, cross-legacy tie ratio for `MA_M`, tool layer presence for `AI_M`) are added on top.

---

## Browser Compatibility

Chrome 120+, Firefox 121+, Safari 17+, Edge 120+. No WebGL required. The file is fully self-contained and works offline.

---

## Changes from v3

v6 represents four generations of development from the original v3 R protocol implementation.

### v4 — Generation algorithm correctness

**BA topology.** The original inner loop recomputed the full weight sum `Σ degree^1.2` on every candidate trial, O(n²) per arriving node. With `m=3` and power 1.2, the degree distribution was nearly Poisson. v4 replaced this with a stochastic acceptance sampler (O(1) per trial), increased to `m=5`, power 1.3, seniority fitness applied during construction, 15% post-construction rewiring toward high-fitness nodes.

**HierCorp topology.** v3 built a separate tree per department with no cross-organisation CEO. `Math.min/max` on node indices was used as the edge key, losing directionality. Within-department informal ties used `0.18 × choose2(n)` — near-clique density for large departments. v4 introduced a single global CEO, level-dependent span-of-control (exec: 3–4, IC managers: 10–12), informal ties capped at `min(√dept_size × 2, 40)`, and cross-department leadership bridges for level-4+ nodes.

**SBM topology.** v3 used flat block probabilities, producing a Poisson degree distribution within blocks. v4 introduced the degree-corrected SBM with per-node Pareto sociability scores.

**Layer derivation.** v3 applied uniform keep-probability per edge, destroying the hub structure of the base graph in every derived layer. v4 made keep-probability degree-dependent with per-layer hub-boost multipliers.

**DEMO_M silo.** v3 set between-block probability to 0.005, producing two near-disconnected cliques. v4 raised this to produce a realistic E-I index of −0.25 to −0.35. (Note: DEMO_M uses `hier_corp`, not SBM — the silo comes from the organic department separation in the tree, not from block probabilities.)

**Visual design.** v4 migrated from a dark-mode neon palette to a warm parchment light-mode design. Brand wordmark uses Fraunces serif. Attribution link added.

### v5 — Scenario plausibility

**Node data consistency.** v3/v4 drew `tenure_years` from an independent lognormal distribution, uncorrelated with `hire_cohort`. A 2024 hire could have 9 years tenure. v5 derives `tenure_years` from `hire_cohort`, making the two attributes internally consistent.

**Gender encoding.** v3/v4 used binary male/female. v5 added `non-binary` (~4%).

**Independent trust and mentorship layers.** v3/v4 derived all layers by subsampling the base topology. Trust and mentorship ties could only exist between pairs already in the base communication graph. v5 introduced independent sparse generators for both. A quiet communicator can now have strong trust ties. Mentorship targets ~40% of IC/manager nodes with level-4+ mentors, 30% cross-department.

**Advice edge direction corrected.** v3/v4 directed advice edges from senior to junior. This is semantically backwards — advice edges should point from seeker (junior) toward advisor (senior). Fixed in v5.

**Scenario structural signals.** REORG_M gained `team_before`/`team_after` attributes and 55% cross-squad collaboration thinning. RESIZE_M gained narrow spans (2–4) with `span_of_control` and `hierarchy_depth` attributes. SUCCESSION_M gained betweenness-derived `key_person_risk_score` and `successor_readiness`. CULTURE_M gained a properly siloed SBM baseline (between-block 0.015) with culture-carrier energy ties providing the cross-boundary contrast. MA_M's legacy split was changed from raw index order to department-alternating, giving both legacy companies full functional coverage.

**Scenario descriptions** were rewritten to accurately reflect the structural signal each network contains.

### v6 — Silent bug fixes and visualiser correctness

Seven issues in v5 were producing structurally incorrect output or incorrect visual representation without errors:

1. **HierCorp informal tie acceptance was inverted** — the division `/ 12` made the rejection condition fire for ICs and never fire for executives, opposite of intent. Fixed with explicit `p = 0.15 + 0.60 × (avgLevel / 6)`.

2. **Location overlay loop added at most one edge per group** — a premature `break` after first success meant the overlay added negligible edges regardless of the computed target. Fixed with a proper rejection-sampling `while (found < target)` loop.

3. **Trust acceptance probabilities double-counted level checks** — `if (lvlDiff === 0)` and `if (lvlDiff <= 1)` both fired for same-level pairs (+0.23 combined vs +0.08 intended). Rewritten as mutually exclusive `if / else if` tiers.

4. **Mentorship direction was inconsistent** — the independent generator bypassed `orientEdge`, producing mentee → mentor direction while all other directed layers use `orientEdge` conventions. Fixed by applying `orientEdge` in `generateLayers`.

5. **Dead entries in LAYER_KEEP_PROB and HUB_BOOST** — `trust` and `mentorship` entries remained in both tables despite those layers no longer using edge derivation. Removed to prevent reader confusion.

6. **Dead DEMO_M block in buildSBMMatrix** — special-case code for a scenario that never calls that function. Removed.

7. **Reporting layer did not produce a hierarchical star layout** — the force simulation used identical physics for all layers. When the reporting layer was soloed, D3 saw a tree but had no instruction to lay it out as one, producing an irregular blob. A secondary issue: `hierarchy_depth` was only written to node objects for `RESIZE_M`, leaving the attribute as zero for `DEMO_M`, `REORG_M`, and `AI_M`. Fixed by introducing `applySimulationForces`, which detects reporting-only views and switches to a `forceRadial` configuration keyed to `hierarchy_depth`, with link strength raised to 0.90 and depth-scaled repulsion. `hierarchy_depth` is now written for all `hier_corp` scenarios.
