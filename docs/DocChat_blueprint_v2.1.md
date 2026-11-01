# DocChat — Guided Build Blueprint v2.1

**Owner:** Clifford Opoku-Sarkodie  
**Revision date:** 9 October 2026  
**Goal:** Learn backend engineering by building a document Q&A portfolio project, then progressively evolve it into a secure SaaS.  
**Companion:** Place `AGENTS.md` at the repository root. This blueprint defines the product, architecture, phases, and acceptance criteria; `AGENTS.md` defines how the IDE agent teaches and collaborates.

## Revision provenance

The supplied `DocChat_blueprint.txt` contains the original Phases 0–8. The earlier conversation describes a 16-phase v2.0, but that file was not supplied. This is a complete v2.1 revision based on the actual attachment and the agreed recommendations; it does not claim to preserve unseen v2.0 text. Original milestones remain in Phases 0–8, with authentication explicitly gated before remote access. Phases 9–15 define the SaaS progression.

v2.1 adds adaptive teaching, progressive assistance, learning recovery, measurable assessments, a completion matrix, an explicit agent authorization policy, and a retrieval baseline. It also corrects page provenance, tokenization assumptions, cross-store consistency, fixed retrieval thresholds, citation guarantees, upload safety, and launch sequencing.

## 1. Product scope and release gates

DocChat accepts PDF, TXT, and DOCX files, extracts and indexes their text, retrieves passages for a question, and returns an answer with traceable citations. The first client is a small Next.js web app with an installable PWA. An Expo client later reuses the API. Backend depth is the priority; frontend polish and native mobile work must not crowd out learning.

| Stage | Phases | Audience and release gate |
|---|---|---|
| Local learning MVP | 0–6 | Single user on loopback; synthetic or public documents; no public upload/chat endpoint |
| Portfolio demo | 7 | Authenticated controlled demo after security checks, or static screenshots/video without exposing the backend |
| Optional native client | 8 | One thin client using the secured API; may be explicitly deferred |
| SaaS foundation | 9–12 | PostgreSQL, durable jobs/storage, tenant isolation, persistent conversations, measurable answer quality |
| Private beta and SaaS | 13–15 | Enforced quotas, reliable operations, tested deletion, release/incident procedures, then an explicitly approved rollout |

Completing Phase 7 does not establish production SaaS readiness. A successful answer does not establish mastery. Each phase requires implementation evidence and demonstrated understanding/debugging.

## 2. Learning system

The agent selects the mode from demonstrated knowledge, rather than assuming prerequisites because a technology appears here.

| Mode | Agent behavior | Learner output |
|---|---|---|
| Teacher | Explain what, why, how, and where; use concrete analogies and short unrelated examples | Explain the concept in your own words |
| Guided practice | Give a small exercise outside the production feature; progressively reduce help | Solve and explain the exercise |
| Challenge | Provide a brief with behavior, constraints, deliverables, acceptance criteria, and test cases | Design and implement the feature |
| Code review | Review the actual diff and relevant tests; explain defects and give hints | Fix issues and justify decisions |

Before a new challenge, ask 2–4 focused prerequisite questions or one tiny practical task. If the topic is unfamiliar, return to Teacher mode. Do not turn assessment into a lengthy exam or require the learner to guess unknown concepts.

**Assistance ladder:** clarify expected/actual behavior and what was tried → conceptual hint → smaller subproblem → pseudocode or analogous example → interactive walkthrough → targeted implementation only when explicitly requested. Short illustrative snippets normally stay under 15 lines. Full feature/file solutions require explicit authorization. Showing code does not authorize writing it into files.

**Recovery:** When you cannot explain or reproduce a feature, mark mastery pending, identify the missing prerequisite, teach it again with a simpler example, and assign a fresh exercise. After two unsuccessful attempts, pause escalation and reassess the hypothesis/prerequisite; do not automatically rewrite the implementation or endlessly repeat the same hint.

**Challenge card required for every phase:** mission; prerequisites and selected mode; expected inputs/outputs; functional requirements; security/correctness constraints; deliverables; acceptance tests; conceptual questions; independent coding exercise; deliberately broken related exercise; evidence to record. Agent-provided tests may define the contract, but you must also design tests yourself and explain their purpose. Fixtures must not reveal the whole solution.

**Mastery checkpoint:** demonstrate understanding, implementation, and debugging independently. A targeted solution revealed by the agent triggers a later analogous exercise without that solution. Scores and completion dates must reflect observed evidence, never estimates of your knowledge.

**Session workflow:** identify phase and last checkpoint → assess prerequisite → teach/practice as needed → one small challenge → learner implements → diff/test review → explanation/debugging check → update progress and learning log when authorized. End with a concise next action and unresolved question. Optional phases can be marked deferred with your reason, rather than falsely completed.

## 3. Ownership and permissions

You remain the primary developer. Reading ordinary project source, explaining it, and proposing changes are allowed by default. The IDE agent must obtain your authorization before creating, editing, deleting, renaming, or executing project files; running commands/tests, installing dependencies, invoking external APIs, migrations, Git mutations, and deployment also require authorization covering those actions.

Authorization may cover a bounded batch or session: for example, “run the existing tests and inspect the diff,” or “create the progress documents for Phase 0.” Reuse that scope without repeatedly asking; stop when it no longer covers the proposed action. “Guide me through Phase 0” authorizes teaching, not automatic scaffolding. Never inspect secret contents without a clear need and authorization; never print them.

Permission to edit progress documents is separate from permission to implement application features. A request to see an implementation permits displaying the requested code, not silently saving it. Explain the precise file/command and purpose when approval is needed. Current explicit instructions from you take precedence over the default mentoring policy.

## 4. Architecture and technology choices

Ingestion: validated upload → document record → page/block extraction → deterministic chunks → embeddings → vector index → ready state. Q&A: authenticated scope → question embedding → scoped retrieval → context budget → generation → citation validation → response. SQL owns document state, authorization, and provenance; the vector index is a rebuildable search projection.

| Component | Initial choice | Evolution / constraint |
|---|---|---|
| Backend | Supported Python release + FastAPI + Pydantic settings | Keep routers thin; isolate parser/provider/index interfaces |
| SQL metadata | SQLite, foreign keys enabled | PostgreSQL with migrations in Phase 9 |
| Parsing | PyMuPDF for text PDFs; python-docx; UTF-8 TXT | Keep parser replaceable; review PyMuPDF licensing before release |
| Chunking | Deterministic text windows with size/overlap and provenance | Use model-compatible counting where available; label estimates |
| Embeddings/chat | Mistral through backend-only adapters | Select available models at implementation time; record model and dimension; no hard-coded price assumptions |
| Vector search | Persistent Chroma, explicitly configured cosine distance | Dedicated service or PostgreSQL/pgvector after evaluating workload |
| Upload storage | Private local directory | Private object storage in Phase 10 |
| Web | Supported Next.js + TypeScript; minimal Tailwind UI | Lock tested versions; avoid freezing the old Next.js 14 assumption |
| Mobile | PWA, then optional Expo/React Native | Share OpenAPI-derived types; platform-specific upload/auth adapters |
| Jobs | Bounded in-process ingestion for small local fixtures | Durable external worker and queue in Phase 10 |

Pin the versions actually verified together and retain lockfiles. Check official documentation before selecting SDK methods, model limits, package APIs, framework setup, or store submission requirements. Floating model aliases may change behavior: record provider model details with evaluation results and re-evaluate changes. A tokenizer named `tiktoken` is not automatically compatible with the selected Mistral model.

Do not introduce a RAG orchestration framework until you can explain the explicit pipeline and justify the added dependency. Do not build billing, OCR, native clients, or streaming before their prerequisites.

## 5. Suggested repository layout

```text
docchat/
  AGENTS.md
  docs/
    DocChat_blueprint_v2.1.md
    PROGRESS.md
    LEARNING_LOG.md
    DECISIONS.md
    API_CONTRACT.md
    SECURITY.md
    DEPLOYMENT.md
    RUNBOOK.md
  backend/
    app/
      main.py
      config.py
      models/
      routers/
      services/       # parser, chunker, ingestion, retrieval, answering
      adapters/       # SQL, vector index, model provider, storage
      auth/
      workers/        # introduced with durable jobs
    tests/
      fixtures/       # synthetic/public documents only
      unit/
      integration/
    evaluations/      # dataset, manifest, reproducible results
    migrations/
    data/             # ignored uploads, SQL and vector persistence
    pyproject.toml
    .env.example
    Dockerfile
  frontend/
    app/
    components/
    lib/
    public/           # icons, manifest, service worker if used
  packages/api-types/ # introduced when a second client needs it
  mobile/            # optional Phase 8
```

Create folders as needed; this layout is not an instruction for the agent to scaffold everything. Ignore secrets, generated data, real uploaded documents, build output, virtual environments, and dependency folders. Example environment files contain placeholders only.

## 6. Correctness and security invariants

### Parsing and provenance

- PDF citations use actual, one-based physical page numbers; printed page labels may be stored separately. DOCX and TXT have no reliable rendered page number: return `page: null` with paragraph/block or line ranges. Do not pretend the whole DOCX is page 1.
- Extract DOCX paragraphs and table text in document order, document unsupported elements, and test table content. TXT decoding errors, malformed/encrypted PDFs, scanned-only PDFs, and empty extracted text receive actionable errors. OCR comes later.
- A valid short document is not rejected by an arbitrary 30-character threshold. Reject empty/whitespace-only usable text; classify unsupported scanned input clearly.
- Split with explicit maximum size and guaranteed progress, including long sentences/paragraphs without punctuation. Normalize deterministically, preserve provenance and readable Unicode, and validate `0 <= overlap < chunk_size`. Token estimates must be documented and actual provider limits enforced separately.
- Each chunk has document/revision ID, index, text, source location, chunking version, embedding model/dimension, and stable deterministic identity within that revision. Record content hash; scope duplicate detection to the owner/tenant to avoid leaking another user's uploads.

### Uploads, consistency, and deletion

- Enforce streamed byte limits, supported extensions plus content/parse validation, generated storage names, and resource/time limits. Client MIME type is not proof. Bound DOCX ZIP expansion and parser work; reject traversal and archive abuse. Run heavy parsing outside the async event loop; use isolated workers for hostile uploads before SaaS.
- Local states: `processing -> ready | failed`, and `ready | failed -> deleting -> deleted`. `deleted` can be a retained tombstone until cleanup/retention policy permits purging. Later jobs add queued and stage progress. Retrieval admits only authorized `ready` revisions.
- SQL and Chroma do not share an atomic transaction. Stage/index work with idempotent IDs, publish `ready` only after validation, clean up failures, and reconcile stale processing records on startup. Catching an exception is not enough to recover a process crash.
- Mark `deleting` before cleanup so results are immediately excluded. Retry deletion of blobs/vectors and reconcile interrupted cleanup. Do not claim final deletion while external cleanup is pending. Concurrent query/index/delete tests must verify visibility.
- Never expose internal storage paths or stack traces. Record structured internal errors with request IDs; avoid logging raw document/question/answer text by default.

### Retrieval and grounded answers

- Explicitly configure cosine distance for Chroma and pass embeddings through the chosen adapter; do not accidentally invoke a default embedding model. Smaller distance means closer under that metric. Never label distance as probability or confidence.
- Query and chunks use the same compatible embedding model and preprocessing. A model/dimension change creates a new index revision; never mix spaces. Reindex without exposing half-built revisions.
- Validate the entire requested document set against ownership before search. Tenant filtering and ready-state checks apply before context construction, not merely to the displayed answer. Empty authorized scope must not fall back to a global search.
- Calibrate thresholds on a labeled evaluation set; `0.55` is not a universal cutoff. `TOP_K=5`, chunk size 400, overlap 60, and a 3000-token context target are starting experiments, not guaranteed best values. Reserve generation and prompt overhead within the chosen model's actual context limit.
- Treat questions, filenames, and document excerpts as untrusted data. System rules instruct the model to use only authorized excerpts, ignore instructions within them, cite supported factual claims, and abstain when evidence is missing. Prompts reduce risk; they do not prove grounding or prevent all injection.
- Build the source map server-side from retrieved chunk IDs. Validate citation labels and source membership; require citations for non-abstaining factual answers. Unknown/missing citations trigger one bounded regeneration or a safe failure/abstention. A valid citation label does not prove that its passage supports the claim: evaluate that separately.
- Canonical abstention text: `I couldn't find this in your documents.` Return a machine-readable status (`answered`, `not_found`, or `generation_failed`) as well as text. When no relevant context survives, return abstention without paying for a chat generation call. Provider failure must not masquerade as lack of evidence.

### Public access and data protection

- Keep local development bound to loopback. Before any remotely accessible data API, require backend token/session validation and owner isolation on every operation, including source downloads. CORS is a browser policy, not access control.
- For identity-provider tokens, validate signature, trusted issuer, audience, expiry, and applicable claims on the backend. For cookies, add CSRF defenses and secure cookie settings. Public endpoint design needs rate/cost/upload caps; a frontend login screen is insufficient.
- Keys stay server-side; keep secrets out of Git and clients. Do not rotate a key solely because deployment occurs; rotate on exposure or defined policy. Use private storage and authorized download links.
- Sanitize Markdown/links; do not render raw model HTML. PWA caches must exclude private API responses, uploaded files, authenticated HTML and session material. Cache only explicitly public static assets, purge user-scoped local state on logout, and show an honest offline state.
- Document the third-party AI data flow, retention/deletion policy, dependency licenses, and release limitations before accepting real user data. PyMuPDF offers AGPL and commercial licensing; choose a compliant licensing strategy or a suitable alternative before product release.

## 7. API contract: initial implementation and migration

Use `/api` consistently and document request/response schemas in OpenAPI. Request IDs and stable error codes let both clients distinguish user errors, provider outages, and retryable failures.

| Method / path | Local MVP behavior | SaaS evolution |
|---|---|---|
| `POST /api/documents` | Multipart upload; 201 only after bounded successful ingestion, with `status: ready` | 202 with document/job IDs after durable acceptance |
| `GET /api/documents` | Owner-scoped summaries/status | Pagination and tenant scope |
| `GET /api/documents/{id}` | Authorized status/details; no internal paths | Progress and retry information |
| `DELETE /api/documents/{id}` | 204 only when cleanup is complete; 202 if pending | Idempotent durable deletion and status |
| `POST /api/chat` | `{question, doc_ids?}` → `{status, answer, sources, request_id}` | Authorized conversation ID and optional streaming |
| `GET /api/health` | Minimal liveness; no secrets | Separate readiness covering required dependencies |

Omitting `doc_ids` means all of this caller's ready documents. Explicit `doc_ids: []` is a validation error. A set containing an unavailable/unauthorized ID returns a non-disclosing 404 before retrieval. No available documents is a 409 `NO_DOCUMENTS_READY`; ready documents with no answer return 200 `not_found`. Duplicate bytes within the same owner produce 409 `DUPLICATE_DOCUMENT`; a later explicit reprocess operation creates a controlled revision rather than unsafe `force=true` behavior.

Use a uniform error response such as `{error: {code, message, request_id, retryable}}`; map FastAPI validation errors into it without leaking input secrets. Suggested mapping: 413 `FILE_TOO_LARGE`; 415 `UNSUPPORTED_TYPE`; 422 `EMPTY_DOCUMENT`, `INVALID_DOCUMENT`, `OCR_REQUIRED`, or invalid query; 429 quota/rate limit; 502/503 provider/dependency failure. Async job errors live in job/document status after acceptance. Do not return HTTP success while pretending a failed operation completed.

Sources contain a server-generated numeric citation label, authorized document ID/name, chunk ID, `page` (nullable), block/line range when applicable, and excerpt. If exposing a metric, call it `cosine_distance` and explain it as a retrieval diagnostic; avoid a fake confidence percentage. Client code must not trust model-produced filenames, IDs, or locations.

## 8. The 16 phases

Every phase uses the learning system and challenge card in Section 2. Each assessment includes the listed coding/explanation/debugging tasks as well as its acceptance tests. Add small prerequisites when needed; do not let prerequisites silently become a second oversized project.

### Phase 0 — Environment, fundamentals, and scaffolding

**Learn:** Python modules/functions/type hints and exceptions as needed; HTTP basics; virtual environments; environment settings; Git; frontend/backend separation.

**Challenge:** You create the smallest repository with root `AGENTS.md`, this blueprint in `docs/`, a health-only FastAPI app, and a web starter. Add `.env.example`, ignore rules, dependency pins/locks, and test setup. Obtain an API key only when live-provider work needs it; mocks let earlier work proceed without spending money.

**Acceptance:** Both local servers start; health test passes; missing required settings fail clearly; no secret/data artifacts tracked; setup can be followed from a clean environment. Do not install the entire future stack immediately.

**Assessment:** Independently add a harmless configuration option; explain environment variables versus committed config; diagnose a wrong working directory/import or missing setting. Record commands actually run and results.

### Phase 1 — Parsing and deterministic chunking

**Learn:** file formats, context managers, tokens/characters, overlap, provenance, bounded algorithms.

**Challenge:** Build parser and chunker interfaces returning typed source blocks/chunks. Support text PDFs, DOCX paragraphs/tables, and UTF-8 TXT under the invariants above. Save synthetic fixtures you are allowed to share.

**Acceptance:** Tests cover tiny/empty/invalid/scanned input, DOCX tables, Unicode, long uninterrupted text, overlap validation, size enforcement, stable IDs, and correct locations. Every loop makes progress and identical input/config produces identical output.

**Assessment:** Implement a new small TXT fixture test yourself; explain why DOCX has no physical page number; diagnose an infinite chunking loop or a source-location offset bug.

### Phase 2 — Embeddings, SQL, and recoverable ingestion

**Learn:** vectors, cosine distance, SQL constraints, transactions, idempotency, compensating cleanup.

**Challenge:** Create SQL document/chunk records, persistent Chroma configuration, a provider adapter, and bounded local ingestion. Store uploads with generated names/content hashes. Record processing state before external work. Enforce foreign keys and deterministic revision/chunk IDs; SQL owns metadata and ready visibility.

**Acceptance:** Mock success, transient 429/timeout, permanent provider errors, failure halfway through indexing, process restart, and duplicate requests. Bounded backoff with jitter honors retry guidance and stops on non-retryable errors. Batch size observes provider input/token limits. Interrupted processing and orphaned vectors can be reconciled; no failed revision is query-visible.

**Assessment:** Add a uniqueness/foreign-key test; explain why SQL rollback cannot roll back Chroma; diagnose a restart that leaves partially indexed data. Perform one opt-in live smoke test later using public data and a cost cap.

### Phase 3 — Retrieval, generation, and the evaluation baseline

**Learn:** retrieval versus generation, top-k, context budgets, abstention, citation membership versus support.

**Challenge:** Build scoped retrieval, context packing, prompt construction, model adapter, citation validation, and structured results. Do not require an untested prompt verbatim; version it and compare its behavior.

**Baseline dataset:** Before tuning, assemble at least 30 questions across at least five small public/synthetic documents. Include at least 15 answerable cases, eight unanswerable cases, and seven adversarial/ambiguous cases. Label answerability, expected passages/locations, acceptable factual answer, and whether context contains injection. Ensure multi-document and boundary-spanning examples are present. Split by document into development and held-out evaluation; use development for thresholds and leave held-out cases untuned. Do not expose expected labels to the answering pipeline.

**Measures:** For answerable questions, report passage hit rate@k (at least one expected passage) and recall@k (fraction of expected passages retrieved). Report unanswerable abstention rate, false abstention on answerable cases, invalid-citation count, and manually judged citation support. Record failures by case ID, dataset hash, models, metric, chunk settings, threshold, prompt version, latency and provider usage. Run mocked CI contract tests separately from explicitly authorized live evaluations.

**Initial project gates:** development hit rate@5 at least 80%; unanswerable abstention at least 80%; zero unknown citation IDs; held-out results reported separately, without hiding failures. These are learning targets on a small dataset, not production accuracy claims. Adversarial cases report source leakage and unsupported claims explicitly. No unauthorized context may reach the model.

**Acceptance:** Baseline runner is reproducible; zero-surviving-context avoids the chat API; budgets account for prompt/question/output; malformed citations have a bounded fallback; outages differ from `not_found`.

**Assessment:** Add and label a fresh question without adjusting the threshold to it; explain retrieval failure versus unsupported generation; diagnose inverted cosine ordering or a model mismatch.

### Phase 4 — REST API and document lifecycle

**Learn:** request validation, multipart streaming, HTTP semantics, error handling, blocking versus async work.

**Challenge:** Expose Section 7's routes with integration tests using mocked providers. Validate upload resource limits, requested IDs, error envelopes, and deletion lifecycle. Keep parser/embedding/index calls out of event-loop blocking paths.

**Acceptance:** OpenAPI matches actual responses; malicious filenames cannot affect storage paths; oversized streamed input is stopped; explicit empty document scope is rejected; partial deletion stays invisible and can retry; no stack traces leak. Endpoint contracts work without live AI calls in CI.

**Assessment:** Design an error-case integration test; explain 201 versus 202; debug ghost search results after deletion or event-loop starvation.

### Phase 5 — Minimal web client

**Learn:** only needed TypeScript, React state, HTTP clients, safe rendering, accessible forms.

**Challenge:** Build library/upload, chat/composer, and source panels. Centralize API calls and typed errors. Distinguish upload byte progress from ingestion status; disable selection/chat for non-ready documents. Provide retry, empty, loading, and failure states without issuing duplicate paid requests.

**Acceptance:** End-to-end upload/question/citation flow works at phone and desktop widths; source chips map to server sources; keyboard navigation and labels work; network failure exits pending state; Markdown/links are safe. No provider key reaches client bundles.

**Assessment:** Build one error-state component yourself; explain request flow; debug a stale response overwriting a newer selected document. Keep styling scope small.

### Phase 6 — PWA and honest offline behavior

**Learn:** manifests, service-worker scope, cache rules, offline constraints.

**Challenge:** Add manifest/icons and a deliberately minimal service worker if needed. Serve it under a valid public URL. Cache only allowlisted public static assets, never private data or API responses. Network-dependent features show a clear offline state.

**Acceptance:** Verify current browser installability requirements and physical-device behavior where available; document untested platforms. Test offline startup, failed chat, update behavior, and logout cache cleanup. Do not rely on an obsolete “Lighthouse PWA pass” as the sole gate.

**Assessment:** Implement a safe offline notice; explain cache-first versus network-only; diagnose a stale/private response cached by an overly broad rule.

### Phase 7 — Portfolio security, tests, CI, and controlled deployment

**Learn:** authentication versus authorization, threat modeling, CI, containers, secrets and deployment.

**Challenge:** Add backend-enforced authenticated owner scope and two-account isolation tests before remote exposure. Add request/upload/rate and provider-cost limits, sanitized errors, dependency/license review, production settings and persistent storage. CI runs unit/integration tests with mocks; one E2E happy path plus failure/authorization cases validates the client.

**Acceptance:** Caller A cannot list, query, delete, or obtain sources for B's documents; mixed-owner `doc_ids` fail before retrieval; forged/expired tokens fail. Tests pass, persistence survives restart, production smoke test is explicitly authorized, and documentation explains limitations and data flow. Remote hosting needs private/authenticated access at both frontend and API; private frontend hosting alone does not protect the API.

**Assessment:** Write a cross-owner test yourself; trace backend token validation; diagnose a protected UI with an unprotected API. If auth/deployment work is deferred, deliver screenshots/video and keep the backend local.

### Phase 8 — Optional native mobile client

**Learn:** platform rendering, upload adapters, mobile authentication, secure token storage, maintenance costs.

**Challenge:** Build a small Expo client only after the secured API works. Reuse shared/generated API types; use platform-specific file/auth handling. Implement library, chat and source display, safe areas, and keyboard behavior. PWA remains the first mobile experience.

**Acceptance:** Device test against a secured backend; interrupted upload/chat has a recoverable state; logout clears private state; no keys bundled. App-store release is separate and requires checking current requirements/costs and explicit authorization.

**Assessment:** Implement one screen interaction; explain shared contract versus platform-specific transport; debug an incorrect multipart upload or token storage issue. You may explicitly defer this phase to prioritize backend SaaS work; record it as deferred.

### Phase 9 — PostgreSQL and schema migrations

**Learn:** relational modeling, migrations, constraints, transaction boundaries, concurrency and connection pools.

**Challenge:** Migrate local SQL state to PostgreSQL with a tested export/import path. Introduce explicit tenant/owner relationships, document revisions, ingestion jobs and audit metadata where needed. Keep embeddings rebuildable from durable text/provenance and model metadata.

**Acceptance:** Clean-install and upgrade migrations run on disposable databases; constraints prevent inconsistent references; representative migrated records match hashes/counts/locations; rollback/recovery is documented and tested without using production data. Document ORM/SQL choice in decisions.

**Assessment:** Add a small migration with a constraint; explain index versus uniqueness; debug a lost update or wrong transaction boundary.

### Phase 10 — Durable background ingestion and object storage

**Learn:** at-least-once delivery, retries, idempotency, durable acceptance, worker isolation and storage permissions.

**Challenge:** Introduce a durable queue/worker and private object storage. Use a transactional outbox or equivalent durable recovery so accepted SQL jobs cannot disappear between DB commit and queue publication. API returns 202 only after durable acceptance. Jobs track queued/processing/ready/failed with bounded attempts and operator recovery. Reject unsafe uploads before expensive work.

**Acceptance:** Worker/queue crash, duplicate delivery, missing blob, provider timeout and failed cleanup tests show recoverability; no duplicate ready revisions or unauthorized visibility. Parser limits cover time/memory/archive expansion; signed download links follow authorization and expire. Client progress follows API status, not a fictitious percentage.

**Assessment:** Implement a retry/idempotency test; explain why an in-process background task is not durable; debug a crash between SQL commit and queue publication.

### Phase 11 — SaaS tenant isolation and authorization hardening

**Learn:** tenant context, membership, least privilege, threat models and defense in depth.

**Challenge:** Extend Phase 7 ownership to an explicit tenant model. Start with one personal workspace per user; defer collaboration unless required. Derive scope from verified identity/membership, never trust a request's tenant ID alone. Apply scope to SQL, vector retrieval, jobs, caches, object access and conversation/source reads.

**Acceptance:** Two-tenant malicious tests cover every endpoint and worker path, guessed IDs, mixed filters, cache keys and delayed jobs. No unauthorized passage reaches the model. If row-level security is adopted, test its configuration and transaction context; it complements application checks.

**Assessment:** Build a negative authorization test; explain identity versus tenant membership; diagnose a correctly scoped SQL lookup followed by a global vector query.

### Phase 12 — Conversations, streaming, and evaluated RAG improvements

**Learn:** persistence, contextual follow-up queries, streaming lifecycle and measurement-driven optimization.

**Challenge:** Store conversations/messages with tenant scope and stable request/idempotency keys. Treat chat history as untrusted; it can help resolve the question but is not source evidence. Add SSE only after non-streaming validation is reliable. Buffer factual answer content until citation validation, or use an explicit unverified-draft protocol; never display unvalidated draft text as a final grounded answer.

**Acceptance:** Disconnect/retry does not double-charge or duplicate completed messages; terminal stream events distinguish completion/failure; all conversation reads are authorized. Evaluate reranking/hybrid search before keeping them using development data and fresh held-out cases; report baseline changes, cost and latency. OCR is an optional extension with provenance and quality tests.

**Assessment:** Implement one persisted follow-up test; explain evidence versus history; debug a half-stream saved as a completed answer. Keep model/prompt revisions reproducible.

### Phase 13 — Quotas, usage accounting, and optional billing

**Learn:** atomic limits, concurrency, accounting ledgers, external event idempotency.

**Challenge:** Add per-tenant upload/storage/request/token or monetary budgets and a global provider spending ceiling. Reserve/check usage before work, reconcile actual usage, and define retry/refund policy. Start with free private-beta tiers; add a payment provider only after quotas work.

**Acceptance:** Concurrent requests cannot bypass limits; failed/duplicate jobs and requests reconcile reservations without double-counting. If billing is included, verify webhook signatures, deduplicate event IDs, handle out-of-order events, and test in sandbox. Never trust client-supplied plan or payment status.

**Assessment:** Write a concurrent quota test; explain reservation versus settlement; debug duplicated webhook entitlements. No live charges without explicit authorization.

### Phase 14 — Reliability, observability, and data lifecycle

**Learn:** structured logs, metrics/traces, timeouts, service objectives, backup/restore and retention.

**Challenge:** Add request/job IDs, stage metrics, resource saturation, provider costs and sanitized alerts. Define measured latency/error objectives for your workload. Implement SQL/object backups, rebuildable vector indexes, tested restores, deletion retries and documented backup retention. Include load tests and provider/worker fault exercises.

**Acceptance:** Restore a disposable environment from backups and verify document access/citations; simulate dependency outage without unbounded retries or spending. Deletion prevents access immediately, completes active-store cleanup, and documents when backups expire. No raw content/tokens in default telemetry. Establish runbook recovery steps and explain observed capacity limits.

**Assessment:** Perform a restore exercise yourself; explain liveness versus readiness; debug an accumulating worker backlog or provider timeout cascade.

### Phase 15 — Private beta, launch assessment, and portfolio evidence

**Learn:** release gates, user feedback, operational ownership, product versus technical readiness.

**Challenge:** Prepare a limited beta with invitation/access controls, support/incident workflow, retention and third-party processing disclosure, resource budgets, and clear supported-file limitations. Write architecture/API/evaluation docs, a portfolio case study and reproducible demo instructions. Deploy only under explicit release authorization.

**Acceptance:** All required earlier rows pass or have an explicitly justified deferral; auth/isolation, deletion, cost caps and restore gates cannot be deferred for SaaS. Fresh security/contract checks and representative held-out evaluation results are reported with limitations. Smoke and rollback procedures are reproducible. Dependency licensing strategy is recorded. Expand access only after beta evidence supports it.

**Assessment:** Independently trace upload-to-answer and failure recovery; diagnose a staged cross-component incident; explain three tradeoffs and what you would change at larger scale. No guarantee of correctness follows from an agent saying “approved.”

## 9. Project-wide completion matrix

Copy this into `docs/PROGRESS.md` when authorized. `—` = pending, `P` = passed with linked evidence, `D` = explicitly deferred with reason, `B` = blocked. Unit/integration tests may be supplemented by manual platform or recovery checks where appropriate; record which evidence exists.

| Phase | Implementation | Tests / checks | Understanding | Independent build | Debugging | Documentation | Git commit | Gate |
|---|---|---|---|---|---|---|---|---|
| 0 Environment | — | — | — | — | — | — | — | — |
| 1 Parse/chunk | — | — | — | — | — | — | — | — |
| 2 Embed/store | — | — | — | — | — | — | — | — |
| 3 Retrieve/evaluate | — | — | — | — | — | — | — | — |
| 4 API | — | — | — | — | — | — | — | — |
| 5 Web | — | — | — | — | — | — | — | — |
| 6 PWA | — | — | — | — | — | — | — | — |
| 7 Secure portfolio | — | — | — | — | — | — | — | — |
| 8 Native, optional | — | — | — | — | — | — | — | — |
| 9 PostgreSQL | — | — | — | — | — | — | — | — |
| 10 Durable ingestion | — | — | — | — | — | — | — | — |
| 11 SaaS isolation | — | — | — | — | — | — | — | — |
| 12 Conversations/RAG | — | — | — | — | — | — | — | — |
| 13 Usage/billing | — | — | — | — | — | — | — | — |
| 14 Operations | — | — | — | — | — | — | — | — |
| 15 Launch | — | — | — | — | — | — | — | — |

Every passed cell links a test run, diff/commit, short explanation, exercise result, or document. Git commits are made by you or explicitly authorized; the agent never fabricates hashes. A phase gate passes only after all required evidence exists and you confirm the checkpoint. If you choose to move ahead without mastery, record the gap and remediation plan rather than relabeling it passed.

Learning log entry: date; phase/mode; concept learned in your own words; exercise; failure/hypothesis/test/result; assistance level; evidence; remaining gap; next action. Decisions log entry: problem; chosen option; alternatives; tradeoffs; evidence; revisit trigger.

## 10. Definition of done and starting prompt

A feature is complete when its acceptance criteria and meaningful tests pass, failure/security paths are checked, documentation matches behavior, and the actual diff is reviewed. It is mastered when you can explain, independently implement an analogous task, and debug a related failure. Deployment readiness additionally needs the release gate for the current audience. An agent review is useful evidence, not the final authority; tests, official documentation, reproducible behavior and your understanding determine progress.

After placing the companion files, begin with:

> Read AGENTS.md and docs/DocChat_blueprint_v2.1.md. Guide me through Phase 0 in learning mode. Assess my prerequisites first. Teach anything unfamiliar, then give me one small challenge. I will write the code. Propose any file changes or commands before acting, and reuse only the authorization I explicitly give you.

## 11. Official reference points

Checked on 9 October 2026 for the narrow corrections below. Recheck version-sensitive behavior during implementation. The learning gates and numeric evaluation targets above are project design choices, not claims from these sources.

- Chroma collection configuration and distance semantics: https://cookbook.chromadb.dev/core/collections/ and Python API reference: https://docs.trychroma.com/reference/python
- Mistral embeddings API and tokenization references: https://docs.mistral.ai/api/endpoint/embeddings and https://mistralai.github.io/mistral-common/usage/tokenizers/
- PyMuPDF licensing choices: https://pymupdf.io/licensing and https://pymupdf.readthedocs.io/en/latest/faq/index.html

Use official framework/provider documentation for each implementation task and record the exact tested versions in the repository. No application has been implemented or technically validated by producing this blueprint.