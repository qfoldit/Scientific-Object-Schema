# Scientific World Schema v1.1

## Object + Operation + Agent + Experiment + Workflow + Measurement + Knowledge Specification

This document extends **Scientific World Schema v1.0** (included below by
reference) to close the gaps found when checking it against a realistic
inventory of laboratory domains and the MCP servers that would need to
exchange data through it. It does not replace v1.0's core model --
`Object -> State -> Operation -> Experiment -> Workflow -> Measurement ->
Knowledge` still holds. It fills in what that model needs to actually
be implementable across many independent, domain-specific MCP servers
talking to one shared world.

**Scope note:** this is a schema design proposal built from the two
uploaded v1.0 documents plus a systematic inventory of laboratory
domains -- it is not itself a published/standardized specification, and
no external citation is claimed for the schema design itself. Where a
domain example below maps onto real, already-built tools (this
project's own `protein_design_mcp`, `bindcraft_mcp`, ZairaChem,
QuPepFold, etc.), those retain their own real citations as already
documented in that codebase's `CITATION.cff` -- this document doesn't
re-litigate those.

---

## 1. What v1.0 got right, and what was missing

v1.0 correctly identifies the five questions a scientific-world schema
must answer (what exists, what can be done, who does it, how
experiments evolve, how knowledge is produced) and a correct top-level
model. But checked against a real multi-domain, multi-MCP deployment,
five concrete gaps show up:

| # | Gap in v1.0 | Consequence if unfixed |
|---|---|---|
| 1 | `Operation Schema` lists only 6 generic verbs (`create, modify, simulate, measure, optimize, evolve`) | Can't express `synthesize`, `mutate`, `dock`, `sequence`, `calibrate`, `culture`, etc. -- most real lab actions have no verb |
| 2 | "Who performs actions" is stated as a goal, but no `Agent` object is ever defined | No way to attribute an action to a human, an AI agent, or a robot; no permissions model for a multiplayer world |
| 3 | Object taxonomy covers 9 types (Protein, DNA, RNA, Cell, Organism, Material, Robot, Planet, Ecosystem) | Missing Molecule/Ligand, Enzyme, Antibody, Plasmid, GeneticCircuit, Tissue/Organoid, Crystal, Polymer, Instrument, Sensor, Dataset, MLModel, Reaction, QuantumCircuit, DigitalTwin, Knowledge/Claim |
| 4 | `Experiment Schema` has no provenance/citation/license/uncertainty/safety fields | Every MCP would reinvent its own ad-hoc way of tracking where a result came from -- exactly the kind of gap that caused fabricated-citation risk in earlier, document-level-only citation tracking |
| 5 | A graph substrate is named elsewhere as "the foundation" (UAG) but never formally defined at the scientific-activity level in this schema | Nothing to actually implement -- "foundation" needs node/edge types, not a name. NOTE: this gap's fix (§7) originally reused the name "UAG" for a *different, broader* graph than the one that name already refers to in `claude-skills/skills/game-designer/references/uag_schema.md` -- see §7's disambiguation note for the correction. |
| 6 | `MCP Integration` gives 2 illustrative examples (Protein MCP, Materials MCP), no general contract | Every new domain MCP would integrate differently, breaking interoperability |
| 7 | Domain coverage misses robotics/lab-automation, instrumentation/metrology, astronomy/planetary, agriculture, and ML/agent-orchestration | Real deployed labs in those areas have nothing to plug into |

Sections 2-8 below close each of these, in order.

---

## 2. Laboratory Domain Inventory ("what labs exist, what can we do in them")

This is deliberately broad -- the goal is that **any** domain-specific
MCP a future contributor writes should already have a home in this
table, an Object subtype namespace, and an Operation vocabulary to
extend, rather than needing a schema change.

| # | Domain | Representative MCP (naming convention: `<domain>_mcp`) | Primary Objects | Primary Operations (domain vocabulary, see §3) | Real-world example already in this ecosystem |
|---|---|---|---|---|---|
| 1 | Structural & molecular biology | `protein_mcp`, `nucleic_mcp` | Protein, DNA, RNA, Complex, Binder | fold, design, mutate, dock, optimize, validate | `qfoldit/Protein-Design-MCP` (RFdiffusion, Boltz-2, PyRosetta) |
| 2 | Chemoinformatics & drug discovery | `chem_mcp` | Molecule, Reaction, ADMETProfile | synthesize, screen, predict, characterize | `predict_admet_profile` / ZairaChem |
| 3 | Synthetic biology | `synbio_mcp` | Plasmid, GeneticCircuit, CRISPRGuide, Strain | assemble, transform, express, culture, evolve | -- (not yet built here) |
| 4 | Materials science | `materials_mcp` | Crystal, Polymer, Alloy, Composite | synthesize, anneal, characterize, simulate | -- (mentioned, not yet built) |
| 5 | Cell & tissue biology | `cellbio_mcp` | Cell, Tissue, Organoid | culture, differentiate, image, measure | -- |
| 6 | Ecology & environmental science | `ecology_mcp` | Ecosystem, Population, Environment | simulate, perturb, monitor, evolve | -- |
| 7 | Quantum computing & physics simulation | `quantum_mcp`, `physics_mcp` | QuantumCircuit, Field, Simulation | simulate, optimize, measure | `qfoldit/Protein-Design-MCP`'s `predict_peptide_quantum_vqe` / `predict_structure_quantum_walk` (QuPepFold / QFold-inspired) |
| 8 | Robotics & lab automation | `robotics_mcp` | Robot, Instrument, Protocol | control, calibrate, dispense, monitor | -- |
| 9 | Astronomy & planetary science | `planetary_mcp` | Planet, Orbit, Atmosphere | simulate, observe, predict | -- (Planet already an Object in v1.0, no MCP defined) |
| 10 | Agriculture & plant science | `agri_mcp` | Organism (plant), Crop, Field(site) | grow (L-system), breed, monitor, optimize | v1.0's L-system section |
| 11 | ML / AI agent orchestration | `agent_mcp` | MLModel, Agent, Dataset | train, infer, evaluate, orchestrate | `qfoldit/Protein-Design-MCP`'s own `gamedesign.py` sits adjacent to this layer (re-narrates results, doesn't train models) |
| 12 | Instrumentation & metrology | `instrument_mcp` | Sensor, Instrument, Calibration | calibrate, measure, monitor | -- |
| 13 | Binder/ligand design (external) | n/a -- external MCP | Protein, Binder | design, dock, validate | `bindcraft_mcp` (BindCraft, via ProteinMCP/MacromNex) |
| 14 | Spatial digital-twin & visualization | n/a -- substrate, not a "lab" | DigitalTwin, Scene | render, export, sync, control (live mode only) | `uag_exporter.py` (`mode: "export"`, OpenUSD -> Omniverse/NanoVer/Unreal/Unity/UNIGINE); Unity MCP Server, Unreal MCP, UNIGINE MCPBridge Plugin, and community UEFN bridges (`mode: "live"`, direct in-editor control -- each with its own client-support/official-vs-community caveats, see §7) |
| 16 | Multiplayer gamification distribution | n/a -- qFoldIT's own product layer, not a "lab" | PlayableArtifact, DigitalTwin | narrate, export, control (live mode, editor-scene only) | `generate_game_design` (narrates edge) + `uefn-fortnite-world-builder` skill (community UEFN bridge, `mode: "live"`) -- Fortnite/UEFN specifically chosen as the multiplayer distribution vehicle since, unlike single-player browser demos (`molecular-tetris`), a published Fortnite island is inherently multiplayer at scale. Scope limit: this bridge authors the editor scene only -- it does not touch Fortnite's Verse/Device gameplay layer or publish the island (see that skill's own scope note) |
| 15 | Gamification / narrative layer | n/a -- substrate, sits on top of Knowledge | PlayableArtifact (see §4) | narrate, score | `generate_game_design` -- explicitly qFoldIT's own layer, not a lab domain itself (see NOTICE-style framing in §9) |

Rows 14-16 are intentionally **not** "labs" -- they're substrate/derived
layers that consume the graph substrate (row 14 materializes it
spatially, row 15 re-narrates it as game structure, row 16 does both --
narrates *and* materializes live, specifically for multiplayer
distribution). Listing them here prevents someone
from accidentally trying to model a `render` operation as if it were a
scientific measurement.

---

## 3. Universal Operation Taxonomy (closes gap #1)

Two tiers, so any domain MCP has both a stable core to rely on and
room to extend without a schema-wide change.

### 3.1 Core verbs (domain-agnostic, always valid on any Object)

```
create · modify · delete · measure · simulate · optimize · evolve ·
compare · annotate · export · import · validate
```

### 3.2 Domain vocabularies (namespaced extensions)

Each domain MCP registers a vocabulary of additional verbs, namespaced
by domain so two domains can safely reuse a word with different
meaning (e.g. `materials.anneal` vs. a future unrelated use):

```
protein/nucleic:  fold · design · mutate · dock · express
chem:             synthesize · screen · react · purify · extract
synbio:           assemble · transform · culture · breed · cross
materials:        anneal · characterize · fabricate
robotics:         control · calibrate · dispense · pick · place
quantum/physics:  entangle · measure_state · anneal (quantum sense)
ml/agent:         train · infer · evaluate · orchestrate
instrumentation:  calibrate · monitor · sample
```

An `Operation` object always carries both its core verb (for
world-level bookkeeping) and its domain-qualified verb (for
domain-specific meaning):

```json
{
  "type": "Operation",
  "core_verb": "modify",
  "domain_verb": "protein.mutate",
  "agent": "agent://human/researcher_42",
  "inputs": ["object://protein/1abc#state3"],
  "outputs": ["object://protein/1abc#state4"],
  "experiment": "experiment://exp_2026_0711_017",
  "timestamp": "2026-07-11T20:14:00Z"
}
```

---

## 4. Object Taxonomy extension (closes gap #3) -- a.k.a. SOS (Scientific Object Schema)

A confirmed architecture reference (2026-07-14) names this taxonomy
**SOS (Scientific Object Schema)**, labeled "Data Specification" --
the canonical model of scientific objects that both SKG and SEM (§7)
are built on top of.

Added to v1.0's existing 9 types (Protein, DNA, RNA, Cell, Organism,
Material, Robot, Planet, Ecosystem):

```
Molecule / Ligand      -- small molecules, drug candidates
Enzyme                 -- catalytic proteins (subtype of Protein, own kinetics fields)
Antibody / Binder       -- subtype of Protein with a target reference
Plasmid                 -- vector backbone + insert
GeneticCircuit          -- composed regulatory elements
CRISPRGuide             -- guide RNA + target locus
Tissue / Organoid       -- multicellular structures
Crystal / Polymer / Alloy / Composite  -- materials science subtypes
Instrument / Sensor     -- physical measurement devices
Dataset                 -- any tabular/structured experimental data
MLModel                 -- a trained model, with its own provenance chain
Reaction                -- a chemical transformation (inputs/outputs/conditions)
QuantumCircuit / Field  -- quantum computing / physics simulation objects
DigitalTwin / Scene     -- the OpenUSD-materialized form of any Object's State
Knowledge / Claim       -- a stated, citable conclusion derived from Measurements
PlayableArtifact        -- a `generate_game_design`-style derived object (levels/achievements); explicitly NOT a scientific Object, tagged `derived_from_knowledge: true` to keep it out of scientific queries by default
```

Every Object still follows v1.0's `Object -> State` pattern -- these
are additional `type` values, not a change to the envelope.

---

## 5. Agent Schema (closes gap #2 -- NEW, not present in v1.0 at all)

```json
{
  "type": "Agent",
  "id": "agent://ai/protein_mcp_worker_7",
  "kind": "human | ai_agent | robot | hybrid",
  "capabilities": ["protein.fold", "protein.design", "measure"],
  "permissions": {
    "can_write_experiment": true,
    "can_approve_knowledge_claim": false
  },
  "affiliation": "optional org/lab name",
  "mcp_endpoint": "optional -- which MCP server this agent runs through, if ai_agent/robot",
  "identity_ref": "optional -- e.g. an ORCID for a human researcher"
}
```

Every `Operation`, `Experiment`, and `Knowledge` object carries an
`agent` reference (see §3.2's example). This is what makes "who
performs actions" actually queryable in a multiplayer world, rather
than only stated as a design goal.

---

## 6. Experiment & Provenance Schema extension (closes gap #4)

v1.0's Experiment already combines goals/objects/equipment/workflows/
results. Extended with:

```json
{
  "type": "Experiment",
  "id": "experiment://exp_2026_0711_017",
  "goal": "...",
  "hypothesis": "optional",
  "protocol_ref": "optional -- link to a reusable Workflow",
  "replicate_count": 3,
  "controls": ["object://..."],
  "agent": "agent://human/researcher_42",
  "safety_level": "BSL-1 | BSL-2 | ... | n/a",
  "license": "e.g. Apache-2.0, CC-BY-4.0, or a named commercial license (PyRosetta, etc.)",
  "references": [
    {"doi": "10.1234/...", "note": "method this experiment's workflow is based on"}
  ],
  "uncertainty": {"metric": "final_energy", "stddev": 0.03},
  "results": ["measurement://..."]
}
```

The `references[]` field is the structural fix for a real failure
mode the qFoldIT ecosystem already ran into: earlier work in `qfoldit/Protein-Design-MCP` had
to manually verify (and once, correct) fabricated-looking citations
*after the fact*, in docstrings and CITATION.cff, because there was no
place in the *data* to require a reference at authoring time. Making
`references[]` a first-class field means a domain MCP that can't cite
a real source for a claim has an explicit, visible empty array,
instead of a plausible-sounding but unverified sentence in a docstring.

---

## 7. Scientific Knowledge Graph (SKG) + Scientific Execution Model (SEM) -- formal definition (closes gap #5)

**Naming history, stated plainly (two corrections in sequence):**

1. An early draft called this graph "UAG (Universal Assembly Graph)" --
   but that name already belongs to a real, different, narrower,
   pre-existing format: the engine-neutral **3D scene graph** defined
   in `claude-skills/skills/game-designer/references/uag_schema.md`
   (node types `mesh`/`light`/`camera`/`trigger_volume`/etc., with
   `connections`/`constraints`/`interactions`, validated by
   `claude-skills/skills/game-designer/scripts/uag_validate.py`). That
   UAG is real, tested, and already in active use by six engine-adapter
   skills -- it is not renamed or altered by anything below.
2. The next draft renamed the broader graph to "SAG (Scientific
   Assembly Graph)" as a single, undifferentiated node/edge model. A
   subsequent architecture reference (`qFoldIT Components` /
   `Runtime Mapping`, confirmed 2026-07-14) established that qFoldIT's
   actual naming splits this into **two** named components instead of
   one, matching a real distinction worth keeping separate: the
   *static relational/semantic structure* (what relates to what) vs.
   the *dynamic execution/protocol layer* (what can be done, and how it
   unfolds over time). This document adopts that split:

- **SKG (Scientific Knowledge Graph)** -- "Semantic Graph." Relations
  between objects, experiments, and knowledge. Node types: `Object` (a
  specific State snapshot), `Experiment` (as a container/context),
  `Measurement`, `Knowledge`. Edge types: `part_of`, `derived_from`,
  `cites`, `measured_by`, `materializes`, `narrates` (defined below).
  This is "the world state as a graph you can query."
- **SEM (Scientific Execution Model)** -- "Executable Model."
  Protocols, workflows, constraints, and experiment telemetry. This is
  §3's Operation taxonomy (core + domain-namespaced verbs), §5's
  `Agent` schema (who can perform which operations), and §6's
  `Experiment` protocol/constraint/replicate/safety fields -- the
  *process* definition, as opposed to SKG's *state* representation.
  Every `Operation` an `Agent` performs, once executed, writes its
  `produced_by`/`derived_from` result back into the SKG as new `Object`
  state -- SEM generates events; SKG stores the resulting graph.

Also per that same reference: **SOS (Scientific Object Schema)** is the
official name for §4's Object taxonomy (the "Data Specification" layer
underneath both SKG and SEM -- what an `Object`'s `type`/`properties`/
`state`/`relationships`/`capabilities` actually look like), and **UWI
(Universal World Interface)** names a *target, not-yet-built*
abstraction discussed in §7.5 below.

The relationship between SKG/SEM and UAG remains simple and
non-competing: an SKG `Object.State`'s `materializes` edge (below)
produces a `DigitalTwin` node, and when that DigitalTwin targets a game
engine, its *content* is exactly a UAG document per the existing,
unmodified `uag_schema.md` -- UAG is the concrete payload format for
one specific SKG node type, not a competing top-level graph.

Together, SKG + SEM form the graph substrate every domain MCP writes
into and every consumer (VR world, knowledge queries, gamification
layer) reads from. Treated as one combined typed property graph for
the node/edge definitions below (most domain MCPs will implement both
halves together in practice):

**Node types:** `Object` (a specific State snapshot, SKG), `Operation`
(SEM), `Experiment` (both -- container in SKG, protocol in SEM),
`Agent` (SEM), `Measurement` (SKG), `Knowledge` (SKG).

**Edge types:**

```
produced_by   Object       -> Operation
input_of      Object       -> Operation
part_of       Object       -> Object          (composition, e.g. atom -> residue -> chain)
performed_by  Operation     -> Agent
belongs_to    Operation     -> Experiment
measured_by   Object        -> Measurement
derived_from  Knowledge     -> Measurement | Experiment
cites         Knowledge | Experiment -> (external reference, DOI/URL)
materializes  Object        -> DigitalTwin     (spatial form -- see "mode" below)
narrates      Knowledge | Experiment -> PlayableArtifact  (gamification layer, one-directional, read-only)
```

`materializes` carries a `mode` attribute distinguishing two genuinely
different integration patterns, both valid, not interchangeable:

- `mode: "export"` -- **one-directional**, snapshot -> scene. The
  `DigitalTwin` is written once from an `Object.State` and the engine
  has no path back to the graph. Reference implementation:
  `uag_exporter.py`'s `export_to_openusd` / `export_pdb_to_openusd`
  (OpenUSD -> Omniverse / NanoVer VR -- both consume OpenUSD as an
  import format, not live).
- `mode: "live"` -- **bidirectional**, an actual running MCP server
  inside the target engine's editor that can both receive SKG state
  and report edits back as new `Operation`s (an SEM concept). Four engines were checked
  directly against their own docs/store listings (2026-07-12); each
  has a genuinely different integration shape -- treat none of these
  as interchangeable with another:
  - **Unity MCP Server** (official, part of Unity AI Beta, requires
    Unity 6.0+; free/included for Pro/Enterprise/Industry, a paid
    add-on for Personal edition; does not itself consume Unity AI
    credits) -- confirmed via `docs.unity3d.com/Packages/
    com.unity.ai.assistant.../unity-mcp-get-started.html`, including
    its exact Claude-Desktop-style client config (a relay binary at
    `~/.unity/relay/`). This is the **primary** `mode: "live"` target
    for Unity in this ecosystem (`unity-experience-builder` in
    `qfoldit/skills` authors its workflows against this server's own
    tool set by default, e.g. `Unity_ManageGameObject`). A separate,
    independent **community** project, `CoplayDev/unity-mcp` ("MCP for
    Unity", MIT-licensed, explicitly not affiliated with Unity
    Technologies), serves only as a **fallback** for Unity <6.0 or
    projects without the AI Assistant package -- it has a different
    default config key (`unityMCP` vs. the official `unity-mcp`) and a
    different tool set, so a workflow authored against one does not
    run unmodified against the other.
  - **UNIGINE MCPBridge Plugin** (free, official UNIGINE Add-On Store,
    v2.21 for SDK 2.21) -- runs its MCP server built into the Editor
    itself, no external proxy; exposes 27 tools covering primitive/
    template/asset-based node creation, transform/clone/reparent,
    materials and surface masks, component attachment, XML export/
    import, console commands, and batch operations with undo/redo --
    confirmed via a screenshot of the live store listing on
    2026-07-12 (`store.unigine.com/en/add-on/1f1237c6-234c-6f20-8dee-
    b35a5bf2dc28/description`). Installation writes a project-root
    `.mcp.json` automatically -- this is **Claude Code's** project-
    scoped MCP convention, not Claude Desktop's global config, so this
    integration is used by opening the UNIGINE project directory in
    Claude Code, not by a manual `claude_desktop_config.json` entry.
  - **Unreal MCP** (official, plugin id `ModelContextProtocol`, shipped
    Experimental in Unreal Engine 5.8 by Epic Games) -- confirmed via
    `dev.epicgames.com/documentation/unreal-engine/unreal-mcp-in-
    unreal-editor`. Embeds a local HTTP MCP server in the Editor
    process; requires a sibling `AllToolsets` plugin for actual tools
    (spawn actors, materials, Slate widgets, automation tests, and
    more via Unreal's ToolsetRegistry). Epic's own documented
    generated-client-config targets are Claude Code, Cursor, VS Code,
    Gemini, Codex, and "All" -- **Claude Desktop is not among them** as
    of this check. A separate, independent, EXPERIMENTAL **community**
    project, `chongdashu/unreal-mcp` (Python-based, not affiliated
    with Epic Games), does document a Claude-Desktop-style manual
    config and can be used as a substitute for Desktop-based workflows.
  - **Omniverse -- narrower, not general-purpose**: `NVIDIA-Omniverse/
    kit-usd-agents` is real and official but is a set of *developer/
    coding-assistance* MCP servers (Kit extension search, USD API/
    class/method docs, code examples) for someone writing Omniverse
    extension code -- **not** a scene-editing bridge comparable to the
    three above (no "spawn a prim here" tool). The closest real
    equivalents to a live scene-editing bridge in the Omniverse family
    are narrower still: an NVIDIA-forum-documented Isaac Sim MCP setup
    (robotics-focused) and RTX Remix's own official MCP Server (scoped
    specifically to the RTX Remix Toolkit, not general Kit). For
    getting a molecule/protein scene into Omniverse at all, `mode:
    "export"` via `uag_exporter.py` remains the correct, general path
    -- Omniverse opens `.usda` natively; there is no verified general
    `mode: "live"` bridge for arbitrary Omniverse Kit scenes as of this
    check.
  A `mode: "live"` materialization means an `Operation` performed
  inside the engine (a designer dragging a node in the Unity/UNIGINE
  editor) can itself be logged back into SEM via `performed_by` (§5's
  `Agent` would be `kind: "human"`, operating through that engine's
  MCP) -- something `mode: "export"` cannot do.

Two materializations of the same SKG subgraph are expected to coexist
without conflict:

- **Spatial materialization**: `Object.State` (atom coordinates, a PDB,
  a crystal lattice) -> `DigitalTwin` via a `materializes` edge ->
  rendered as an OpenUSD scene (`mode: "export"`, e.g.
  `uag_exporter.py`) or driven live inside an engine's own editor
  (`mode: "live"`, e.g. Unity MCP Server or UNIGINE MCPBridge).
- **Narrative materialization**: `Knowledge`/`Experiment` ->
  `PlayableArtifact` via a `narrates` edge -> a game design document
  (`generate_game_design` is a concrete implementation of this edge).
  This edge is explicitly one-directional and non-authoritative: a
  `PlayableArtifact` never feeds back into scientific `Knowledge`.
  (Note the asymmetry with `materializes`: `narrates` has no `mode:
  "live"` equivalent by design -- gamification output should never
  write back into scientific Knowledge, whereas an engine's own
  editor legitimately can write back new Operations.)

This is also where "Object + Operation + Result = New State" (v1.0's
closing line) becomes precise: a `Result` is an `Operation`'s output
`Object.State`, and the graph edge recording that is `produced_by`.

### 7.5. UWI (Universal World Interface) -- translation layer built; live cross-engine invocation still isn't

A confirmed architecture reference (2026-07-14) names a sixth
component, **UWI (Universal World Interface)**, explicitly labeled
"Target Specification" (as opposed to UAG's "Runtime DSL," which is
labeled as an *existing* format) -- described as a unified abstract API
for interacting with game engines, simulators, and labs, via gRPC/
Protobuf, with a proposed method set like `SpawnObject`, `ApplyGraph`,
`GetState`, `StreamEvents`.

**Updated status:** `qfoldit/Protein-Design-MCP`'s `src/protein_design_mcp/uwi.py`
now implements a real translation layer for `SpawnObject`, `ApplyGraph`,
and `GetState` (exposed as the `uwi_spawn_object` / `uwi_apply_graph`
MCP tools), plus an honest `StreamEvents` finding (below). Two things
are true at once, and both need to be said precisely:

1. **What's real:** given a generic call (e.g. "spawn a mesh at
   position X on engine Y"), `uwi.py` returns the *exact* tool name and
   arguments to invoke on that engine's real, already-connected MCP
   server -- with an explicit `confidence` field (`verified` /
   `best_effort` / `unsupported`) so a caller knows how much to trust
   each mapping rather than getting a uniform-looking answer that
   hides which engines were actually checked at the API-signature
   level (only UEFN's `spawn_actor`/`get_project_info`/`get_level_info`
   are `verified`; Unity/Unreal/UNIGINE mappings are `best_effort` --
   the *capability* was confirmed via each engine's own docs, but not
   the exact tool/parameter name, so a caller should confirm via that
   engine's own `tools/list` before relying on it). `uwi_apply_graph`
   further decomposes a whole UAG document into an ordered sequence of
   these translations (parents before children), plus an honest,
   per-engine report of which UAG `connections`/`constraints`/
   `interactions` categories have no dedicated tool at all (mirroring
   each engine skill's own SKILL.md gap notes, e.g. UEFN's Verse/Device
   layer being entirely out of scope).
2. **What's still not real:** this translation layer does **not**
   itself hold a live connection to Unity/Unreal/UNIGINE/UEFN's MCP
   servers, and does not invoke their tools. The calling agent still
   has to take the returned `{tool_name, arguments}` spec and make the
   actual call on that engine's own connected MCP server -- because an
   MCP server has no built-in way to reach into a *different* MCP
   server's tool-call channel without also acting as that other
   server's client. A true "one call does it live on any engine"
   proxy would mean this process holding simultaneous client
   connections to four other MCP servers -- a distinct, larger,
   not-yet-attempted project. `uwi.py`'s own module docstring states
   this scope limit explicitly, so it isn't rediscovered as a surprise
   later.

**`StreamEvents` -- an honest negative result, not a placeholder:**
no evidence was found, across any of the four engines' own docs/repos
checked in this pass, of an event-streaming tool. `uwi_stream_events`
returns `confidence: "unsupported"` for all four rather than guessing
a plausible-sounding tool name -- this is a genuine finding (nothing
to translate to), not unfinished work.

A `mode: "live"` `materializes` edge (§7 above) today still always
means "one specific engine's bespoke tool calls, now with a known
translation available for SpawnObject/GetState/ApplyGraph" -- not "an
autonomous UWI call that reaches the engine by itself."

### Runtime Reality Check (vs. the `qFoldIT Components` / `Runtime Mapping` reference)

That reference's Runtime Mapping table shows a checkmark for every
component (SOS/SKG/SEM/UAG/UWI/MCP) across 11 runtimes. This document
only asserts what was independently verified; treat unchecked rows as
"not yet independently confirmed," not "confirmed absent." The UWI
column below reflects `uwi.py`'s translation layer (§7.5) specifically
-- 🟡 means "SpawnObject/GetState translation implemented, confidence
noted; StreamEvents honestly unsupported," not "fully live."

| Runtime | SOS/SKG/SEM | UAG (`mode: "export"`, via `uag_exporter.py`) | UAG (`mode: "live"` engine bridge) | UWI (`uwi.py` translation layer) |
|---|---|---|---|---|
| Unity | Applies generically (any domain MCP) | N/A (not an OpenUSD target) | ✅ Official Unity MCP Server (verified); ⚠️ community CoplayDev/unity-mcp fallback | 🟡 SpawnObject/GetState: `best_effort` (tool name not API-verified, confirm via `tools/list`); StreamEvents: `unsupported` |
| Unreal Engine | Applies generically | ✅ opens `.usda` | ✅ Official Unreal MCP, UE 5.8 (verified) -- Claude Code/Cursor/VSC/Gemini/Codex only | 🟡 SpawnObject/GetState: `best_effort` (`tool_name` intentionally `None`, confirm via `tools/list`); StreamEvents: `unsupported` |
| UNIGINE | Applies generically | ✅ opens `.usda` | ✅ Official MCPBridge Plugin (verified) -- Claude Code via project `.mcp.json` | 🟡 SpawnObject/GetState: `best_effort` (`tool_name` intentionally `None`, confirm via `tools/list`); StreamEvents: `unsupported` |
| NVIDIA Omniverse | Applies generically | ✅ opens `.usda` natively | ⚠️ `kit-usd-agents` is coding-assistance only (verified) -- no general live scene-editing MCP found | N/A -- no scene-control MCP exists to translate to; `uwi.py` has no Omniverse adapter |
| Fortnite/UEFN | Applies generically | ✅ opens `.usda` (Fortnite/UEFN's underlying Unreal base supports USD import) | ✅ Community bridges only (verified) -- no Epic-published UEFN MCP exists today | ✅ SpawnObject/GetState: `verified` (confirmed against KirChuvakov/uefn-mcp-server's own tool list); StreamEvents: `unsupported` |
| Blender | Applies generically | ✅ opens `.usda` (native USD I/O) | **Not independently verified in this pass** -- Blender has a large addon ecosystem; a live MCP bridge plausibly exists but wasn't checked here | Not implemented in `uwi.py` |
| Labster, LabVIEW, ROS2, NI DAQ, Physical Sensors | Applies generically (SOS/SKG/SEM don't require a 3D scene) | N/A or unclear (some of these have no natural "3D scene" concept -- matches the source diagram's own UAG column showing "--" for LabVIEW/ROS2/NI DAQ/Physical Sensors) | **Not independently verified in this pass** | Not implemented in `uwi.py` -- no natural "spawn object" concept for most of these |

---

## 8. MCP Integration Contract (closes gap #6 -- generalizes v1.0's 2 examples)

Any domain MCP -- whether built in `qfoldit/Protein-Design-MCP` or external (like
`bindcraft_mcp`) -- must satisfy this contract to participate in the
shared world:

1. **Object conformance**: every input/output must be a v1.1 Object
   type, or a domain-namespaced extension (`materials.Alloy`) declared
   in that MCP's own schema file.
2. **SEM logging**: every tool call that performs an Operation must be
   able to emit the `{agent, core_verb, domain_verb, inputs, outputs,
   experiment, timestamp}` record from §3.2 -- whether it writes
   directly to a shared graph store or returns it for a caller to log.
3. **Result envelope**: every tool result is JSON with, at minimum,
   `{"status": "ok" | "error" | "unavailable" | "not_configured",
   "data": {...}, "references": [...]}`. This is not new -- it's the
   same convention already used throughout `qfoldit/Protein-Design-MCP`'s
   `predict_admet_profile` (`not_configured` per endpoint, never a
   fabricated score) and `predict_peptide_quantum_vqe` (`unavailable`
   if the quantum backend isn't installed, never a crash).
4. **Spatial bridge**: any tool producing 3D/spatial output should be
   exportable through a shared `materializes`-edge-compatible bridge
   (`qfoldit/Protein-Design-MCP`'s `uag_exporter.export_pdb_to_openusd` is the reference
   implementation) rather than a one-off, domain-specific exporter.
5. **Naming convention**: `<domain>_mcp` for the server, `verb_object`
   for tools (`design_binder`, `predict_structure`, `fold_protein`),
   matching `qfoldit/Protein-Design-MCP`'s existing convention.

---

## 9. How this maps onto what's already built (sanity check)

| Schema concept | Concrete implementation already in this ecosystem |
|---|---|
| Domain MCP (#1, structural biology) | `protein_design_mcp` (`qfoldit/Protein-Design-MCP`) |
| Domain MCP (#2, chemoinformatics) | `predict_admet_profile` / ZairaChem wrapper |
| Domain MCP (#7, quantum) | `predict_peptide_quantum_vqe`, `predict_structure_quantum_walk` |
| External domain MCP (#13) | `bindcraft_mcp` (BindCraft via ProteinMCP/MacromNex) |
| Spatial materialization (§7) | `uag_exporter.py` (`export_to_openusd`, `export_pdb_to_openusd`) -- `mode: "export"` |
| Spatial materialization, live mode (§7) | Unity MCP Server (official, Unity 6.0+, Claude-Desktop-documented), Unreal MCP (official, UE 5.8, Claude Code/Cursor/VSC/Gemini/Codex only -- not Claude Desktop), UNIGINE MCPBridge Plugin (official, auto-configures Claude Code's project `.mcp.json`), UEFN community bridges (KirChuvakov/uefn-mcp-server, MIT; quangdang46/uefn-verse-mcp, AGPL -- neither affiliated with Epic Games) -- `mode: "live"`, bidirectional in-editor control, each with a different client-support surface |
| UWI (§7.5) | `qfoldit/Protein-Design-MCP`'s `src/protein_design_mcp/uwi.py` (`uwi_spawn_object`, `uwi_apply_graph`, `uwi_get_state`, `uwi_stream_events` MCP tools) -- a real translation layer (verified for UEFN, best_effort for Unity/Unreal/UNIGINE, honestly `unsupported` for StreamEvents everywhere). Still not a live cross-engine invocation system -- the calling agent makes the actual call on the target engine's own connected MCP server using the returned spec. |
| Multiplayer gamification distribution (§2 row 16) | `generate_game_design` + `uefn-fortnite-world-builder` skill -- see that row's own scope-limit note (authors the editor scene, doesn't publish/wire gameplay) |
| Spatial materialization, coding-assistance (not live scene control) | `NVIDIA-Omniverse/kit-usd-agents` -- official, but answers API/extension questions rather than editing a running scene; not a `materializes` implementation in the same sense as the row above |
| Narrative materialization (§7) | `generate_game_design` |
| `references[]` field (§6) | Already the pattern `qfoldit/Protein-Design-MCP`'s own `CITATION.cff` had to be corrected into after the fact (Neil Voss / Virtual-Lab-Simulation, ZairaChem's real authors/DOI) -- v1.1 makes that a schema field, not just a document convention |

**One explicit framing note, consistent with how this ecosystem has
handled attribution so far:** `generate_game_design`'s `PlayableArtifact`
node and the "qFoldIT adds gamification to science" positioning behind
it remain qFoldIT's own product layer (per the `narrates` edge's
one-directional, non-authoritative definition in §7) -- this schema
change formalizes that boundary structurally, it doesn't change what
any upstream tool or paper claims.

---

## Appendix: v1.0 source documents (for reference)

The two original v1.0 documents this extends are archived in this
same repository at `archive/v1.0/README_EN.md` and
`archive/v1.0/Scientific_World_Schema_v1.0.md`. Nothing above removes
or contradicts their core model, JSON examples (parametric organism,
L-system), or the `Future Layers` note (`Schema -> Runtime -> MCP ->
Simulation -> VR World`) -- §7-8 above are the concrete content for the
"MCP" and "Simulation" stages of that pipeline.
