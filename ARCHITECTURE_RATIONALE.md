# SymBro Architecture — Design Rationale

This document complements the published SymBro architecture views with the reasoning behind selected structural decisions. The diagrams show how responsibilities are separated; this rationale explains why those boundaries exist, which alternatives were deliberately avoided, and which trade-offs follow from them.

The document grows together with the published architecture series. This first release covers Views 1–3 only.

## 1. System Context

### External AI is a permitted capability, not system authority

SymBro is designed as a privacy-first personal AI system whose orchestration and governed state remain under its own control. External AI services may provide explicitly permitted reasoning or other high-precision capabilities, but they are not modeled as architectural authorities over SymBro.

A simpler design could delegate larger parts of orchestration to an external model or provider. That would reduce local coordination logic, but it would also blur the boundary between external probabilistic capability and internal system authority.

The architecture therefore models External AI Services as capabilities selectively used by SymBro. This keeps the system boundary explicit and allows external capabilities to evolve without redefining ownership of SymBro's internal decisions or governed state.

**Trade-off:** SymBro must maintain its own orchestration responsibilities instead of delegating them wholesale to an external provider.

## 2. Architecture Overview

### Client and server are responsibility boundaries, not merely deployment locations

SymBro separates the interactive runtime from canonical knowledge and background knowledge-processing responsibilities. The client owns the interactive path: routing, context preparation, agent control, prompt compilation, and model execution control. The server owns governed memory and knowledge responsibilities, including source integration, candidate processing, retrieval, semantic index management, and canonical persistence.

It would be simpler to treat this split as a description of the machines on which components currently run. That would make the architecture depend on today's deployment topology rather than on stable responsibilities.

The architecture therefore uses **Client** and **Server** as logical responsibility boundaries. Concrete hosts and deployment choices may change without changing the architectural contract.

**Trade-off:** Communication across the boundary must use explicit contracts, and responsibilities cannot silently migrate between client and server merely because doing so is convenient in a particular deployment.

### Canonical responsibilities remain on the governed side of the boundary

Source orchestration may happen outside SymBro, and interactive execution happens on the client, but neither is allowed to become an alternative authority over canonical memory or knowledge. Canonical persistence and the lifecycle that governs information before it becomes trusted state remain server responsibilities.

This separation deliberately favors governance and traceability over the convenience of allowing every producer or consumer to update persistent state directly.

**Trade-off:** New ingestion and interaction paths require integration with the governed lifecycle rather than direct persistence shortcuts.

## 3. Interactive Runtime & Decision Flow

### Retrieval prepares context; Agent Control does not execute retrieval

An interactive request may require knowledge before SymBro can make its deterministic agent decision. A tempting implementation would let Agent Control discover that need and invoke retrieval itself. That would make Agent Control both a decision authority and an execution orchestrator for knowledge access.

SymBro instead evaluates retrieval before the deterministic Agent Control decision. When retrieval is required, structured knowledge is prepared as context and then supplied to the controlled decision pipeline.

This keeps the responsibilities distinct: retrieval obtains relevant governed knowledge; Agent Control decides whether and how generation may proceed.

**Trade-off:** The interactive runtime requires an explicit preparation stage before Agent Control rather than a single component dynamically performing every required action.

### Prompt Compilation remains deterministic

Prompt Compilation assembles the context that has already been selected and admitted by upstream responsibilities. It does not become another routing layer, retrieval engine, or policy authority.

Allowing prompt construction to make hidden decisions about retrieval, routing, or execution could make the final model input convenient to assemble, but it would also move architectural decisions into a stage that is difficult to observe and govern independently.

SymBro therefore keeps Prompt Compilation deterministic: it compiles prepared inputs according to explicit contracts rather than discovering new work while constructing the prompt.

**Trade-off:** Upstream components must provide sufficiently structured and complete inputs; Prompt Compilation cannot silently repair missing architectural decisions.

## Published Views in This Release

1. **System Context** — SymBro's system boundary, sole user, and controlled relationship with external AI capabilities.
2. **Architecture Overview** — the major client, server, source-orchestration, persistence, and external-AI responsibilities.
3. **Interactive Runtime & Decision Flow** — the controlled path from an interactive request through routing, context preparation, retrieval evaluation, Agent Control, prompt compilation, and response generation.

Further rationale will be added as the remaining architecture views are published.