# qFoldIT Platform Integration

Scientific Object Schema is the semantic layer for persistent scientific worlds. It defines scientific entities, actions, actors, experiment evolution and knowledge state above engine-specific scene implementations.

## Relationship to qFoldIT contracts

```text
Scientific Object Schema
        |
        v
Scientific State
        |
        v
UAG
        |
        +--> UEFN
        +--> Unity
        +--> UNIGINE
        +--> Web
        +--> Standalone
```

The semantic model describes what a scientific world means. UAG describes how that world is assembled for a runtime. Engine adapters realize UAG without changing the underlying scientific semantics.

## Supported conceptual domains

- protein and molecular structures;
- drug-discovery objects and experiment workflows;
- synthetic-biology systems;
- L-system and procedural biological growth;
- materials and atomistic systems;
- environmental and ecological simulations;
- AI scientist agents and experiment actions.

## Architectural rule

Scientific Object Schema remains engine-neutral and solver-neutral. Scientific computation is delegated to authoritative scientific services, while world runtimes consume canonical state and bindings.
