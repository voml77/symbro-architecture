# SymBro Architecture

SymBro is a privacy-first personal AI system built around deterministic orchestration, governed memory and knowledge, and controlled use of external AI capabilities.

This repository documents the architecture of SymBro and the reasoning behind selected design decisions. The SymBro source code remains private; this repository contains architecture documentation only.

## Published Architecture Views

The architecture is being published incrementally. The current release contains:

1. **System Context** — the system boundary, sole user, and controlled relationship with external AI capabilities.
2. **Architecture Overview** — the major architectural responsibilities across client, server, source orchestration, persistence, and external AI.
3. **Interactive Runtime & Decision Flow** — the controlled path of an interactive request through routing, context preparation, retrieval evaluation, agent control, prompt compilation, and response generation.
4. **Routing & Model Execution** — the governed routing path from intent correction through domain and context routing to controlled local or external model execution.
5. **Memory & Knowledge Lifecycle** — the governed path from interactions, source packages, and research results through candidate validation and lifecycle management to domain-specific canonical promotion.
6. **Knowledge Retrieval** — the read-only retrieval path from query embedding and semantic candidate discovery to canonical rehydration, validation, and structured retrieval results.
7. **Semantic Index Lifecycle** — the controlled synchronization and generation lifecycle of a rebuildable semantic locator derived from canonical knowledge.
8. **Knowledge Gap & Research** — the governed escalation path for unresolved knowledge needs from gap evaluation through permitted external research capability and back into candidate governance.

The corresponding diagrams are available in [`diagrams/`](diagrams/).

## Design Rationale

The diagrams describe the structural views. [`ARCHITECTURE_RATIONALE.md`](ARCHITECTURE_RATIONALE.md) explains selected design decisions, deliberately avoided shortcuts, and their trade-offs.

The rationale grows together with the published architecture views.

## Scope

This repository intentionally does not contain the SymBro application source code, internal Structurizr model, or implementation-specific configuration.

## License

The architecture documentation and diagrams in this repository are licensed under the [Creative Commons Attribution 4.0 International License](LICENSE).

Copyright © 2026 Vadim Ott.