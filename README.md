SMURF 🦀⏳
Spatial Multidimensional Unified Runtime Framework
An experimental programming language where code is a graph, execution is a validated transition, and time is a controllable coordinate.
Project status: Concept / early research. The syntax, architecture, and CLI examples below are design proposals. They are not claims that a working compiler, runtime, or language adapters have already been implemented.

SMURF is an experimental language and compiler project bootstrapped in Rust. It explores whether the most useful capabilities of Rust, C, C++, JavaScript, TypeScript, and Python can be expressed through one coherent language, with simpler everyday syntax and explicit control when necessary.

Unlike a conventional transpiler or a bundle of wrappers, SMURF aims to define its own syntax, type system, effect model, temporal graph intermediate representation (IR), runtime, and compilation pipeline.
Why SMURF?
Modern software development often requires moving between languages and tools to accomplish a single task. Each language has valuable ideas, but also its own syntax, memory model, build system, error handling, and runtime assumptions.

SMURF asks a different question:

What if developers could describe an operation once, while a compiler mapped its types, dependencies, state changes, time, and execution strategy?

The intended experience is:
- Simple by default: concise expressions, inferred types, and minimal boilerplate.
- Explicit when needed: ownership, effects, memory layout, and execution strategy can be controlled.
- Graph-native: functions, values, dependencies, and state transitions have inspectable relationships.
- Temporal: graphs and nodes can be created, changed, removed, queried historically, and replayed.
- Persistent: committed states can be stored as content-addressed snapshots.
- Polyglot: existing libraries and APIs remain accessible through verified interoperability boundaries.
The six language influences
Ecosystem	Ideas SMURF explores
Rust	Ownership, memory safety, traits, concurrency, native performance
C	ABI interoperability, predictable data layouts, low-level control
C++	Generic programming, native integration, high-performance abstractions
JavaScript	Closures, dynamic data, events, asynchronous workflows
TypeScript	Structural typing, interfaces, unions, type inference
Python	Expressive syntax, scripting ergonomics, data and scientific workflows


Unification does not mean identical semantics. Unsafe memory, dynamic typing, exceptions, object lifetimes, and concurrency differ across ecosystems. SMURF must specify its own rules and preserve foreign-language behavior at explicit boundaries.
A first look at SMURF
The following is proposed syntax:
space Genesis {
    node source at (10, 20) {
        energy: f64 = 100.0;
    }

    node target at (11, 20) {
        energy: f64 = 0.0;
    }

    transition transfer(amount: f64) {
        require amount >= 0.0;
        require source.energy >= amount;

        source.energy -= amount;
        target.energy += amount;

        ensure source.energy + target.energy == 100.0;
    }
}
A transition is intended to be atomic: preconditions are checked, candidate changes are evaluated, and invariants are validated before state is committed. A failed transition leaves the committed state unchanged.
A simple scraper experiment
space NewsScraper {
    page = web.get("https://example.com")

    articles = page.select("article").map(article => {
        title: article.text("h2")
        url: article.link("a")
    })

    save articles to "articles.json"
}
The intended compiler would resolve the network and filesystem effects, infer types, build a dependency graph, and lower the operations into a runtime or compiled backend. Network and filesystem failures must remain observable rather than silently ignored.

A more compact scraper DSL may eventually be offered as syntax sugar over the same core language.
The temporal graph model
SMURF's proposed program model is:
\[
G = (V, E, \Sigma, \mathcal{T}, \mathcal{H})
\]
- V — program entities: functions, variables, modules, objects, and spatial nodes.
- E — dependencies, dataflow, control flow, and other relationships.
- Σ — types and runtime states.
- T — validated state transitions.
- H — event history and provenance.
A node may optionally have a logical spatial address (x, y, z, layer). Logical grid positions do not imply physical coordinates.

The graph is time-dependent:
\[
G(t) = (V(t), E(t), S(t))
\]
Nodes and edges may not have existed at an earlier time. Historical queries must distinguish present, known absent, and unknown. Deleting an active node need not erase its historical record.
Synthesized simulation time
SMURF explores a virtual clock whose rate is controlled numerically:
\[
t_{n+1} = t_n + v_n\Delta\tau
\]
Here v is a speed multiplier and Δτ is a reference-time increment. The model supports pause, fractional speed, fast-forward, and historical navigation.

For deterministic replay, canonical event ordering should use integer ticks and explicit ordering rules. Playback speed must remain separate from numerical solver timestep. Rewinding means restoring or replaying historical state—not reversing arbitrary physical processes.
Two kinds of history
SMURF intends to support bitemporal records:
- Valid time: when something occurred in the modeled domain.
- Recorded time: when SMURF learned or stored it.
This makes it possible to ask both “What do we currently believe happened at T?” and “What did we know about T as of R?”
Proposed compiler pipeline
                     .smurf source
                           |
                      Lexer / Parser
                           |
                      Typed AST
                           |
               Name + Type + Effect Checks
                           |
                  Temporal Graph IR
                           |
            Dependency / Transition Validation
                           |
                       Optimizer
                           |
               +-----------+-----------+
               |           |           |
          Interpreter    Native       WASM
                           |
                   Explicit FFI bridges
                   (C/C++, Python, JS)
Implementation strategy: start with a small reference interpreter to define and test language semantics. Native and WebAssembly compilation follow only after the core is stable.
Databank integration
SMURF is designed to use existing per-language databanks and development infrastructure as a knowledge and interoperability layer.

Possible indexed records include:
- Language and version
- Packages, modules, and symbols
- Type signatures and API metadata
- Source locations and dependency edges
- Documentation references
- Provenance, licenses, and content hashes
These indexes may support symbol discovery, API mapping, diagnostics, binding generation, and migration assistance. They do not replace deterministic compiler checks. Retrieved metadata and AI-generated transformations must be validated before they affect compilation or execution.

Integration status: Existing databanks, repository locations, and schemas have not been inspected or verified as part of this README. No specific integration is claimed to work yet.
Persistence and replay
SMURF aims to record committed state changes as ordered events and produce immutable, content-addressed snapshots.
Initial state -> Event 1 -> Event 2 -> Event 3
     |             |          |          |
   Root 0        Root 1     Root 2     Root 3
A Merkle root is an integrity identifier, not a reversible compression of the underlying data. Reconstructing a snapshot requires retaining its content chunks or sufficient event history.

Target capabilities include:
- Transactional state updates and rollback
- Deterministic replay of recorded inputs
- Historical graph queries
- Branching and counterfactual simulations
- Incremental dependency invalidation
- Canonical serialization and snapshot integrity checks
Planned repository layout
smurf/
├── Cargo.toml
├── crates/
│   ├── smurf-lexer/
│   ├── smurf-parser/
│   ├── smurf-ast/
│   ├── smurf-types/
│   ├── smurf-grid/
│   ├── smurf-ir/
│   ├── smurf-temporal/
│   ├── smurf-runtime/
│   ├── smurf-store/
│   ├── smurf-compiler/
│   ├── smurf-databanks/
│   ├── smurf-interpreter/
│   └── smurf-cli/
├── adapters/
│   ├── rust/
│   ├── c/
│   ├── cpp/
│   ├── javascript/
│   ├── typescript/
│   └── python/
├── examples/
├── tests/
└── docs/
This is an architectural target, not an assertion that these directories exist.
Proposed CLI
smurf check examples/energy_transfer.smurf
smurf run examples/energy_transfer.smurf
smurf trace examples/energy_transfer.smurf
smurf graph examples/energy_transfer.smurf
smurf replay examples/energy_transfer.smurf
smurf databanks inspect
These commands are planned and are not yet available. Installation and usage instructions will be added when an executable implementation exists.
First executable experiment
The first milestone is deliberately narrow: implement a working parser and interpreter for a two-node energy-transfer program.

Acceptance criteria:
1. Parse a .smurf file.
2. Construct a typed graph.
3. Create nodes at logical grid coordinates.
4. Execute a valid transfer.
5. Reject insufficient-energy transfers.
6. Preserve committed state after a failed transfer.
7. Validate an energy-conservation invariant.
8. Record deterministically ordered events.
9. Produce a reproducible snapshot hash.
10. Replay the event log and reproduce the same graph state.
The scraper experiment should follow using a local HTML fixture first, then carefully controlled network access.
Roadmap
- [ ] Phase 0: Inventory existing repositories, databanks, and reusable tooling.
- [ ] Phase 1: Define grammar, AST, type rules, and graph IR.
- [ ] Phase 2: Implement transactional reference interpreter and tests.
- [ ] Phase 3: Add temporal events, snapshots, and replay.
- [ ] Phase 4: Implement scraper fixture and typed effects.
- [ ] Phase 5: Add verified language adapters and databank indexing.
- [ ] Phase 6: Add native and WebAssembly backends.
- [ ] Phase 7: Add incremental compilation and visual temporal debugger.
- [ ] Phase 8: Explore self-hosting SMURF compiler components.
Design principles
1. One coherent language, not six syntaxes pasted together.
2. Progressive complexity: simple defaults, explicit control when necessary.
3. Correctness before optimization: no silent semantic changes.
4. Existence is temporal: absence, presence, and unknown are distinct.
5. State changes are validated: failed transactions do not partially commit.
6. History is inspectable: provenance and replay are first-class concepts.
7. Interop is explicit: foreign-language boundaries preserve relevant semantics.
8. Claims are testable: no feature is considered implemented until it runs and passes tests.
Contributing
SMURF is an experimental research project. Useful early contributions include language-design feedback, grammar proposals, Rust compiler infrastructure, deterministic runtime tests, and examples comparing SMURF syntax against existing languages.

Before implementing large subsystems, document the intended semantics and create a minimal reproducible test. Contributions should not claim functionality that has not been built or verified.
License
Not selected yet. Add a LICENSE file before publicly inviting reuse or contributions. Until then, do not assume the repository is open source merely because its source is visible.
