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
| 5 | UAG (Universal Assembly Graph) is named elsewhere as "the foundation" but never formally defined in this schema | Nothing to actually implement -- "foundation" needs node/edge types, not a name |
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
| 1 | Structural & molecular biology | `protein_mcp`, `nucleic_mcp` | Protein, DNA, RNA, Complex, Binder | fold, design, mutate, dock, optimize, validate | This repo's `protein_design_mcp` (RFdiffusion, Boltz-2, PyRosetta) |
| 2 | Chemoinformatics & drug discovery | `chem_mcp` | Molecule, Reaction, ADMETProfile | synthesize, screen, predict, characterize | `predict_admet_profile` / ZairaChem |
| 3 | Synthetic biology | `synbio_mcp` | Plasmid, GeneticCircuit, CRISPRGuide, Strain | assemble, transform, express, culture, evolve | -- (not yet built here) |
| 4 | Materials science | `materials_mcp` | Crystal, Polymer, Alloy, Composite | synthesize, anneal, characterize, simulate | -- (mentioned, not yet built) |
| 5 | Cell & tissue biology | `cellbio_mcp` | Cell, Tissue, Organoid | culture, differentiate, image, measure | -- |
| 6 | Ecology & environmental science | `ecology_mcp` | Ecosystem, Population, Environment | simulate, perturb, monitor, evolve | -- |
| 7 | Quantum computing & physics simulation | `quantum_mcp`, `physics_mcp` | QuantumCircuit, Field, Simulation | simulate, optimize, measure | This repo's `predict_peptide_quantum_vqe` / `predict_structure_quantum_walk` (QuPepFold / QFold-inspired) |
| 8 | Robotics & lab automation | `robotics_mcp` | Robot, Instrument, Protocol | control, calibrate, dispense, monitor | -- |
| 9 | Astronomy & planetary science | `planetary_mcp` | Planet, Orbit, Atmosphere | simulate, observe, predict | -- (Planet already an Object in v1.0, no MCP defined) |
| 10 | Agriculture & plant science | `agri_mcp` | Organism (plant), Crop, Field(site) | grow (L-system), breed, monitor, optimize | v1.0's L-system section |
| 11 | ML / AI agent orchestration | `agent_mcp` | MLModel, Agent, Dataset | train, infer, evaluate, orchestrate | This repo's own `gamedesign.py` sits adjacent to this layer (re-narrates results, doesn't train models) |
| 12 | Instrumentation & metrology | `instrument_mcp` | Sensor, Instrument, Calibration | calibrate, measure, monitor | -- |
| 13 | Binder/ligand design (external) | n/a -- external MCP | Protein, Binder | design, dock, validate | `bindcraft_mcp` (BindCraft, via ProteinMCP/MacromNex) |
| 14 | Spatial digital-twin & visualization | n/a -- substrate, not a "lab" | DigitalTwin, Scene | render, export, sync, control (live mode only) | `uag_exporter.py` (`mode: "export"`, OpenUSD -> Omniverse/NanoVer/Unreal/Unity); Unity MCP Server and UNIGINE MCPBridge Plugin (`mode: "live"`, direct in-editor control -- see §7) |
| 15 | Gamification / narrative layer | n/a -- substrate, sits on top of Knowledge | PlayableArtifact (see §4) | narrate, score | `generate_game_design` -- explicitly qFoldIT's own layer, not a lab domain itself (see NOTICE-style framing in §9) |

Rows 14-15 are intentionally **not** "labs" -- they're substrate/derived
layers that consume UAG (row 14 materializes it spatially, row 15
re-narrates it as game structure). Listing them here prevents someone
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

## 4. Object Taxonomy extension (closes gap #3)

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
mode this project already ran into: earlier work on this codebase had
to manually verify (and once, correct) fabricated-looking citations
*after the fact*, in docstrings and CITATION.cff, because there was no
place in the *data* to require a reference at authoring time. Making
`references[]` a first-class field means a domain MCP that can't cite
a real source for a claim has an explicit, visible empty array,
instead of a plausible-sounding but unverified sentence in a docstring.

---

## 7. UAG (Universal Assembly Graph) -- formal definition (closes gap #5)

UAG is the graph substrate every domain MCP writes into and every
consumer (VR world, knowledge queries, gamification layer) reads from.
It is a typed property graph:

**Node types:** `Object` (a specific State snapshot), `Operation`,
`Experiment`, `Agent`, `Measurement`, `Knowledge`.

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
  inside the target engine's editor that can both receive UAG state
  and report edits back as new `Operation`s. Two concrete,
  independently-verified implementations exist today:
  - **Unity MCP Server** (official, part of Unity AI Beta, requires
    Unity 6.0+; free/included for Pro/Enterprise/Industry, a paid
    add-on for Personal edition; does not itself consume Unity AI
    credits) -- `docs.unity3d.com/Packages/com.unity.ai.assistant.../
    unity-mcp-get-started.html`.
  - **UNIGINE MCPBridge Plugin** (free, official UNIGINE Add-On Store,
    v2.21 for SDK 2.21) -- runs its MCP server built into the Editor
    itself, no external proxy; exposes 27 tools covering primitive/
    template/asset-based node creation, transform/clone/reparent,
    materials and surface masks, component attachment, XML export/
    import, console commands, and batch operations with undo/redo --
    `store.unigine.com/en/add-on/1f1237c6-234c-6f20-8dee-b35a5bf2dc28/
    description`.
  A `mode: "live"` materialization means an `Operation` performed
  inside the engine (a designer dragging a node in the Unity/UNIGINE
  editor) can itself be logged back into UAG via `performed_by` (§5's
  `Agent` would be `kind: "human"`, operating through that engine's
  MCP) -- something `mode: "export"` cannot do.

Two materializations of the same UAG subgraph are expected to coexist
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

---

## 8. MCP Integration Contract (closes gap #6 -- generalizes v1.0's 2 examples)

Any domain MCP -- whether built in this repo or external (like
`bindcraft_mcp`) -- must satisfy this contract to participate in the
shared world:

1. **Object conformance**: every input/output must be a v1.1 Object
   type, or a domain-namespaced extension (`materials.Alloy`) declared
   in that MCP's own schema file.
2. **UAG logging**: every tool call that performs an Operation must be
   able to emit the `{agent, core_verb, domain_verb, inputs, outputs,
   experiment, timestamp}` record from §3.2 -- whether it writes
   directly to a shared graph store or returns it for a caller to log.
3. **Result envelope**: every tool result is JSON with, at minimum,
   `{"status": "ok" | "error" | "unavailable" | "not_configured",
   "data": {...}, "references": [...]}`. This is not new -- it's the
   same convention already used throughout this codebase's
   `predict_admet_profile` (`not_configured` per endpoint, never a
   fabricated score) and `predict_peptide_quantum_vqe` (`unavailable`
   if the quantum backend isn't installed, never a crash).
4. **Spatial bridge**: any tool producing 3D/spatial output should be
   exportable through a shared `materializes`-edge-compatible bridge
   (this repo's `uag_exporter.export_pdb_to_openusd` is the reference
   implementation) rather than a one-off, domain-specific exporter.
5. **Naming convention**: `<domain>_mcp` for the server, `verb_object`
   for tools (`design_binder`, `predict_structure`, `fold_protein`),
   matching this repo's existing convention.

---

## 9. How this maps onto what's already built (sanity check)

| Schema concept | Concrete implementation already in this ecosystem |
|---|---|
| Domain MCP (#1, structural biology) | `protein_design_mcp` (this repo) |
| Domain MCP (#2, chemoinformatics) | `predict_admet_profile` / ZairaChem wrapper |
| Domain MCP (#7, quantum) | `predict_peptide_quantum_vqe`, `predict_structure_quantum_walk` |
| External domain MCP (#13) | `bindcraft_mcp` (BindCraft via ProteinMCP/MacromNex) |
| Spatial materialization (§7) | `uag_exporter.py` (`export_to_openusd`, `export_pdb_to_openusd`) -- `mode: "export"` |
| Spatial materialization, live mode (§7) | Unity MCP Server (official, Unity 6.0+) and UNIGINE MCPBridge Plugin (free, official Add-On Store) -- `mode: "live"`, bidirectional in-editor control |
| Narrative materialization (§7) | `generate_game_design` |
| `references[]` field (§6) | Already the pattern this repo's own `CITATION.cff` had to be corrected into after the fact (Neil Voss / Virtual-Lab-Simulation, ZairaChem's real authors/DOI) -- v1.1 makes that a schema field, not just a document convention |

**One explicit framing note, consistent with how this ecosystem has
handled attribution so far:** `generate_game_design`'s `PlayableArtifact`
node and the "qFoldIT adds gamification to science" positioning behind
it remain qFoldIT's own product layer (per the `narrates` edge's
one-directional, non-authoritative definition in §7) -- this schema
change formalizes that boundary structurally, it doesn't change what
any upstream tool or paper claims.

---

## Appendix: v1.0 source documents (for reference)

The two uploaded documents (`README_EN.md` and
`Scientific_World_Schema_v1.0.md`) remain the base layer this document
extends; nothing above removes or contradicts their core model, JSON
examples (parametric organism, L-system), or the `Future Layers` note
(`Schema -> Runtime -> MCP -> Simulation -> VR World`) -- §7-8 above are
the concrete content for the "MCP" and "Simulation" stages of that
pipeline.
