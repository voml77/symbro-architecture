# SymBro Architecture — Design Rationale

This document complements the published SymBro architecture views with the reasoning behind selected structural decisions. The diagrams show how responsibilities are separated; this rationale explains why those boundaries exist, which alternatives were deliberately avoided, and which trade-offs follow from them.

The document grows together with the published architecture series. This release covers all eight published architecture views.

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

## 4. Routing & Model Execution

### Routing authority and model execution control remain separate

SymBro separates the decision about how a request should be routed from the control of where and under which permitted capability it is ultimately executed. Intent correction, embedding-based routing, domain routing, and context routing progressively prepare the route; Model Execution Control receives that result and governs downstream execution without becoming another routing authority.

A simpler design could combine routing and execution control in a single component. That would reduce the number of explicit boundaries, but it would also create a second place where routing decisions could be reinterpreted or silently overridden.

The architecture therefore treats the route produced by Context Routing as an input to Model Execution Control rather than as an invitation to route the request again. Model Execution Control governs execution against permitted local or external capabilities while preserving the authority of the upstream routing path.

**Trade-off:** Routing and execution require an explicit contract between responsibilities, and execution control cannot silently compensate for weak or incomplete routing decisions.

### External routing fallback does not become a second routing authority

SymBro may use an external routing fallback when local deterministic and semantic mechanisms cannot resolve an intent with sufficient confidence. The fallback provides a controlled routing result back into the same routing path; it does not own downstream execution and does not establish a parallel orchestration path.

Allowing an external model to route and execute a request directly would simplify unresolved cases, but it would also bypass the responsibility boundaries that keep external probabilistic capability subordinate to SymBro's orchestration.

**Trade-off:** Unresolved routing cases must return through the governed routing flow before execution can proceed.

## 5. Memory & Knowledge Lifecycle

### A candidate is not canonical state

Information entering SymBro from an interaction, a normalized source package, or a governed research result does not become trusted Memory or Knowledge merely because it has been extracted or classified. The Librarian produces typed candidates, which must pass validation and the candidate lifecycle before they can be considered for domain-specific promotion.

A simpler design could persist extracted information directly into active Memory or Knowledge. That would shorten the ingestion path, but it would collapse extraction, validation, review, and activation into a single probabilistic step.

SymBro therefore keeps candidate state explicitly separate from canonical state. Candidate Validation establishes structural and domain admissibility; Candidate Lifecycle governs review states such as verification, rejection, merge, or supersession; Domain Promotion determines whether a verified candidate may enter a canonical domain under that domain's policy.

**Trade-off:** Knowledge acquisition requires more lifecycle stages and explicit state transitions, but probabilistic extraction cannot silently become trusted system state.

### Promotion is domain-specific rather than a universal activation step

Personal Memory, Knowledge, and Signal do not share identical admission semantics. Personal Memory requires explicit user opt-in, Knowledge is promoted under its activation policy, and Signal remains inactive by default.

A universal activation mechanism would be simpler to implement, but it would erase meaningful differences between personal memory, reusable knowledge, and weaker interest or context signals.

The architecture therefore places Domain Promotion after the shared candidate lifecycle and applies domain-specific governance only at the point where verified candidates may become canonical state.

**Trade-off:** Each canonical domain requires its own promotion semantics instead of relying on one generic activation rule.

## 6. Knowledge Retrieval

### Semantic search locates candidates; canonical persistence decides what is valid

SymBro deliberately separates semantic discovery from canonical authority. The semantic index is used to locate potentially relevant Knowledge Artifacts, while canonical persistence remains responsible for their authoritative content and current eligibility.

A simpler RAG design could treat the vector database as both search mechanism and knowledge store. That would reduce retrieval steps, but it would also make derived index state authoritative and allow stale or otherwise ineligible indexed content to influence generated answers.

Knowledge Retrieval therefore uses the semantic index only to identify candidate artifact references. Retrieved candidates are rehydrated from canonical persistence and validated against the current canonical state before structured retrieval results are returned downstream.

This principle can be summarized as:

**Chroma locates. SQLite decides.**

**Trade-off:** Retrieval requires an additional canonical rehydration and validation step instead of returning vector-search results directly, but semantic index state cannot silently override canonical knowledge state.

### Retrieval remains read-only

Knowledge Retrieval evaluates and returns currently valid Knowledge Artifacts. It does not repair stale index entries, mutate canonical state, or perform lifecycle transitions while serving a retrieval request.

Combining retrieval and repair could make individual requests appear self-healing, but it would introduce write-side effects into a read path and make retrieval behavior harder to reason about.

Repair and synchronization therefore remain separate responsibilities.

**Trade-off:** Stale or invalid index state may require independent maintenance rather than being repaired inline during a user request.

## 7. Semantic Index Lifecycle

### The semantic index is derived and rebuildable state

The Semantic Index is not a second knowledge authority. It is a derived locator built from eligible canonical Knowledge Artifacts and can be recreated from canonical persistence.

Treating the index as independently authoritative would simplify some vector-centric implementations, but it would create two competing sources of truth and make synchronization failures architecturally ambiguous.

Semantic Index Management therefore derives index operations from canonical state and applies idempotent UPSERT, DELETE, or NO-OP decisions. Embeddings are produced for eligible artifacts as part of this controlled synchronization path.

**Trade-off:** Index management requires explicit synchronization and compatibility rules, but loss or corruption of the semantic index does not imply loss of canonical knowledge.

### Rebuilds use generations instead of replacing the active index in place

A full semantic-index rebuild must not require the currently active index to disappear while a replacement is being constructed.

Rebuilding the active generation in place would be simpler, but a failed or incomplete rebuild could leave retrieval without a valid semantic locator.

SymBro therefore builds a shadow generation from eligible canonical state, validates it, activates it atomically, and only then retires the previous generation. The shadow generation is derived index state rather than a separate software component.

**Trade-off:** Generation management introduces additional lifecycle state and storage during rebuilds, but the active index remains available until its replacement has been successfully validated.

## 8. Knowledge Gap & Research

### Retrieval and research are different responsibilities

An unresolved knowledge need does not cause Knowledge Retrieval to become a research mechanism. Retrieval answers a bounded question: what currently governed Knowledge Artifacts are available inside SymBro? Research addresses a different question: what should happen when the required knowledge is not available there?

A simpler design could let retrieval fall through directly to an external model or web-capable service whenever no useful result is found. That would blur the boundary between trusted internal knowledge and newly acquired external information.

SymBro therefore evaluates unresolved knowledge needs separately. Knowledge Gap Evaluation can request governed research execution through Model Execution Control, which may use permitted external AI capabilities. The resulting research output returns through the Librarian and the normal candidate-governance lifecycle rather than entering canonical Knowledge directly.

**Trade-off:** Newly researched information is not immediately available as trusted canonical knowledge; it must first pass through the same governance boundaries as other candidate information.

### External capability cannot write canonical knowledge directly

External AI services provide capabilities, not authority over SymBro's internal knowledge state.

Allowing research output to be written directly into canonical persistence would shorten the research path, but it would allow probabilistic external output to bypass validation, candidate lifecycle management, and domain promotion.

Research results therefore re-enter SymBro through the Librarian. From that point onward they are treated as governed candidate information and must earn canonical status through the existing lifecycle.

**Trade-off:** Research acquisition requires an additional governance path before information becomes reusable canonical Knowledge, but external capability cannot silently become internal truth.

## Published Views in This Release

1. **System Context** — SymBro's system boundary, sole user, and controlled relationship with external AI capabilities.
2. **Architecture Overview** — the major client, server, source-orchestration, persistence, and external-AI responsibilities.
3. **Interactive Runtime & Decision Flow** — the controlled path from an interactive request through routing, context preparation, retrieval evaluation, Agent Control, prompt compilation, and response generation.
4. **Routing & Model Execution** — the governed routing path from intent correction through domain and context routing to controlled local or external model execution.
5. **Memory & Knowledge Lifecycle** — the governed path from interactions, source packages, and research results through candidate validation and lifecycle management to domain-specific canonical promotion.
6. **Knowledge Retrieval** — the read-only retrieval path from query embedding and semantic candidate discovery to canonical rehydration, validation, and structured retrieval results.
7. **Semantic Index Lifecycle** — the controlled synchronization and generation lifecycle of a rebuildable semantic locator derived from canonical knowledge.
8. **Knowledge Gap & Research** — the governed escalation path for unresolved knowledge needs from gap evaluation through permitted external research capability and back into candidate governance.