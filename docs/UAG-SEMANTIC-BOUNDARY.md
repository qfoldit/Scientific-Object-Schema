# qFoldIT Scientific Object Schema → UAG Boundary

## Purpose

Scientific Object Schema is the semantic layer for persistent scientific worlds. UAG is the engine-neutral runtime package representation produced after Mission routing and compilation.

The two layers are intentionally complementary:

```text
Scientific Object Schema
        ↓
Scientific State / semantic objects
        ↓
Mission Compiler
        ↓
UAG package
        ↓
Engine Adapter
```

## Semantic responsibility

Scientific Object Schema defines what scientific entities mean, how they evolve, what actions are valid, which actors participate and how experimental knowledge is represented.

It does not prescribe UEFN, Unity, UNIGINE or Web implementation details.

## Runtime responsibility

UAG defines the engine-neutral representation consumed by runtime adapters. It carries mission identity, runtime requirements, world nodes, relations and scientific-state references without embedding a specific engine API.

## Conformance boundary

A compiled UAG package must preserve:

- source mission identity and version;
- routing decision identity;
- required capabilities;
- scientific-state references;
- provenance of compilation;
- node identity and relation integrity.

Scientific validation remains outside this layer and is performed by the qFoldIT validation fabric.

## Result

This boundary allows the same persistent scientific world to be compiled into UEFN, Unity, UNIGINE, Web and future runtimes without changing the semantic scientific model.
