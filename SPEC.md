AI Execution Integrity Framework (AIEF)

Draft v0.2.5 — Public Comment Edition
Document status: Mixed normative (only sections explicitly marked “Normative” contain requirements)
Last updated: 2026-02-27
Recommended review cadence: Quarterly

Notice

This document proposes baseline control objectives and evidence expectations for AI execution integrity in systems where decisions or actions may require auditability, defensibility, or long-term traceability. It is implementation-agnostic and intended for public discussion. It does not constitute legal advice or regulatory guidance.

This is a living draft. Comments, issues, and pull requests are welcome at:https://github.com/artnames/AIEF

Why this exists now

There is no widely adopted standard that defines what verifiable AI execution means in practice. Organizations are rapidly deploying AI decision systems without shared expectations around auditability, integrity verification, or long-term defensibility. AIEF is an attempt to define practical, implementable expectations early—before regulatory pressure or high-profile incidents force a rushed response.
Executive Summary

AI systems are transitioning from advisory tools to autonomous decision-makers embedded in SaaS workflows, financial operations, government services, and enterprise automation. As impact rises, trust shifts from model capability to decision defensibility: the ability to demonstrate what occurred, under which context, and whether the evidence record has been altered.

Traditional observability (logs, traces, metrics) is necessary for operations, but often insufficient for governance and external scrutiny. In higher-impact contexts, organizations benefit from an execution artifact: a structured, tamper-evident record of a decision event that supports deterministic integrity verification.

AIEF defines:
• a shared vocabulary for AI execution integrity
• baseline control objectives and evidence expectations
• conformance levels to support incremental adoption
• non-normative examples and illustrative schemas
• a minimal verifier interoperability contract to support cross-vendor verification

AIEF does not require deterministic model outputs. It focuses on the integrity and verifiability of recorded decision artifacts. For probabilistic systems, AIEF assumes cache-as-truth semantics: the recorded output is authoritative for audit; verification checks integrity of the record rather than reproducing outputs.
Quick Start (10-Minute Read)

2.1 Recommended starting point

A practical starting point is Level 2 conformance (Artifact + Tamper-Evidence + Deterministic Verification):
Capture an execution artifact representing the decision context and outputs.
Ensure the artifact has tamper-evidence over a defined protected set of fields.
Provide a deterministic verifier that outputs PASS/FAIL and reason codes.
Store the artifact (or a redacted form) with a stable retention policy and export path.

For agentic / multi-step workflows, the practical starting point is Level 2 plus chain and dependency traceability (see Level 4).

2.2 What to prioritize first

Choose one early workflow where:
• decisions have external impact (financial, legal, user access, safety, compliance), and
• there is a clear owner for the workflow and its evidence.

Start with workflows like:
• approve/deny/escalate
• risk score thresholds
• fraud/AML flags
• account actions
• policy enforcement decisions

2.3 Common pitfalls (avoid)

• treating distributed logs as “the artifact”
• storing unverifiable records (no integrity mechanism or unclear protected set)
• confusing integrity verification with “re-run the model and match outputs”
• allowing “hash everything” to become fully opaque compliance with no reviewer access path
• silent degraded modes where missing evidence still “looks conformant”
Public Comment Invitation

This document is deliberately early and open.

We welcome feedback from builders, auditors, regulators, academics, and anyone who has encountered these challenges in practice — no contribution is too small.

Suggested contributions:
• edits to definitions or control objectives
• missing threat vectors or overlooked use cases
• conformance level adjustments by risk tier
• interoperability recommendations
• clearer test procedures and evidence expectations
• privacy-preserving integrity practices that work today
Scope and Applicability

4.1 In scope

AIEF applies to AI-driven systems producing decisions/actions that may be scrutinized, including:
• regulated SaaS workflows (eligibility, risk scoring, compliance triage)
• financial services (KYC/KYB, transaction monitoring, underwriting, disputes)
• government/public sector systems (case triage, benefits eligibility, enforcement prioritization)
• enterprise policy engines and automated approvals
• long-term archives and evidentiary systems

4.2 Out of scope

AIEF does not attempt to:
• define model safety/alignment requirements
• prove algorithmic correctness (formal verification)
• define fairness/bias standards
• replace comprehensive governance programs (except where necessary to anchor execution evidence)
• guarantee confidentiality (beyond enabling privacy-preserving integrity patterns)

4.3 Integrity vs determinism

AIEF focuses on integrity of the record. Many AI systems are probabilistic; AIEF does not require deterministic future outputs.
Execution Integrity Boundary

AIEF governs the integrity of the execution artifact, not “result accountability” in the sense of proving a decision was correct, fair, or appropriate.

Within boundary (AIEF targets):
• completeness of the execution artifact as defined by scope
• tamper-evidence over a defined protected set
• deterministic verification of integrity
• chain integrity for multi-step workflows (when applicable)
• preservation of declared versions and context necessary for interpretation

Outside boundary (AIEF does not assert):
• correctness of the decision
• fairness, bias, or alignment properties
• the business appropriateness of the outcome
• the real-world truth of external dependencies
• good-faith issuance by the originating system (without additional measures)

Artifact authenticity vs artifact integrity (scope note)

AIEF provides mechanisms to detect modification of protected fields after issuance. It does not, by itself, prove that the artifact faithfully represents what actually occurred if the issuing system generates false-but-internally-consistent artifacts. Insider threat, false issuance, and runtime trust at issuance time are out of scope unless paired with additional measures (e.g., independent attestation, trusted execution, transparency logging, or provenance controls such as AIEF-10).

Reliance statement (Normative within AIEF)

Successful verification of an AIEF artifact supports claims that protected fields have not been modified since issuance under the declared integrity and stability schemes. It does not support claims about factual accuracy, correctness, fairness, or good-faith issuance by the originating system.
Definitions

AI Execution
A discrete decision event performed by an AI system (model or agent) that produces an output from specified inputs and execution context. An execution may represent either (a) an advisory recommendation or (b) an operative action; implementations SHOULD declare which their artifact represents.

Decision
An output of an AI execution that recommends or determines an outcome (e.g., “ALLOW”, “DENY”, “ESCALATE”, “RISK=0.73”). A decision may be advisory or operative.

Action
A business effect taken by a system or human (e.g., account blocked, transaction held, benefit granted). Actions may occur in a different system and at a different time than the AI recommendation; implementations MAY link actions to the originating execution artifact.

Execution Artifact
A structured, machine-verifiable representation of an AI execution suitable for storage and deterministic integrity verification. An execution artifact may be represented as:
• a discrete document (e.g., JSON),
• a signed or hashed event stream, or
• a verifiable data structure (e.g., Merkle tree over events),
provided it satisfies the control objectives in Section 10.

Execution Snapshot
A point-in-time representation of relevant inputs, context, parameters, versions, and outputs. (A snapshot is one form of an execution artifact.)

Evidence Pack
A companion bundle of referenced materials required for human interpretability and audit review (e.g., full prompt text, full or redacted tool responses, policy text snapshots, reviewer notes), linked from the execution artifact via hashes or evidence references. The pack is not required to be globally verifiable by itself, but SHOULD be integrity-anchored by the execution artifact.

Execution Integrity
The property that an execution artifact is complete (per scope), tamper-evident, and verifiable via deterministic rules, enabling reviewers to trust the artifact without trusting the originating application alone.

Protected Set
The subset of artifact fields that are integrity-protected and whose modification must be detectable by verification.

Stable Serialization (Stability Scheme)
A deterministic serialization procedure that produces the same byte representation for the same logical artifact, enabling consistent integrity verification over time.

Replay
The ability to reconstruct and validate the recorded execution context and artifact integrity. Replay does not necessarily require re-executing the model.

Cache-as-Truth (Core AIEF semantic)
An approach where the recorded output is treated as authoritative for audit; verification checks integrity of the recorded artifact rather than attempting to reproduce probabilistic outputs.

Verification
A deterministic procedure that checks artifact integrity (e.g., stable serialization + digest/signature verification, chain integrity), returning PASS/FAIL and reason codes.

Version Pinning
Preservation of declared identifiers and versions (model/provider, runtime/protocol, policy rules) such that historical context remains interpretable over time.

Tool Call / External Dependency Event
An interaction where an agent/workflow calls external APIs/tools/services that influence outputs.

Chain Integrity
A property of multi-step workflows ensuring steps cannot be inserted/removed/reordered without detection.

Reviewer Class
A defined role or audience (internal or external) with documented access to interpretive materials (artifact fields and/or Evidence Pack) required for audit review.

Verifier (Interoperability Class)
A deterministic verifier implementation that accepts AIEF artifacts and outputs standardized PASS/FAIL with reason codes per Section 9.
Design Principles
Artifact-first: treat important AI decisions as preserved artifacts.
Tamper-evidence: changes to protected fields must be detectable.
Deterministic verification: verification must be repeatable and rules-based.
Independent validation: verification should not require trusting the originating application alone.
Portability: artifacts should be exportable and verifiable over time.
Privacy-aware: support minimization, redaction, hashing, and access controls.
Cache-as-truth: for non-deterministic systems, the recorded output is authoritative for audit; verification validates the artifact, not model reproducibility.
Integrity Verification Semantics in Non-Deterministic Systems

Important (read first):
If you came here expecting “replay” to mean “re-run the model and get the same answer,” stop here. That is not what AIEF means by replay. AIEF defines replay as integrity replay: the ability to reconstruct the recorded execution context and verify the artifact has not been altered. This is intentional because probabilistic model outputs and external dependencies may not be reproducible.

8.1 What replay is (and is not)

Replay in AIEF is integrity replay, not output determinism.

Replay means:
• the artifact preserves decision context at time of execution
• integrity can be validated deterministically
• reviewers can detect post-hoc changes

Replay does not require:
• re-running the model and matching tokens/output
• external dependencies returning identical responses

Example: a fraud decision depends on a sanctions list lookup. At audit time, the sanctions list may have changed. AIEF replay does not require the lookup to return the same result—only that the record of the original lookup result (or its hash/evidence reference) is present and unaltered.

8.2 Recommended pattern: cache-as-truth

For probabilistic systems, treat the recorded output as authoritative, while ensuring:
• output is preserved (or output hash preserved)
• integrity mechanism binds output to inputs + context
• verification proves the record has not been altered

8.3 Streaming and long-running executions

For streaming outputs or long-running agents, an execution artifact may be:
• a bounded event stream (token chunks, tool events, state transitions) with verifiable integrity, or
• a snapshot plus references to an integrity-protected event log,
provided the protected set and verification semantics are defined.

8.4 Optional extensions

Some systems may additionally implement:
• determinism for subcomponents (policy evaluation, thresholds)
• pinned tool outputs (store bodies or response hashes)
• deterministic replay tests for deterministic segments only
Minimal Verifier Interoperability Contract (Normative)

To support cross-vendor verification, AIEF defines a minimal verifier interoperability class.

9.0 Normative interpretation rules
A verifier MUST accept an AIEF artifact in a supported schema/protocol version, or fail deterministically.
A verifier MUST apply the declared stability scheme (stable serialization) or fail with a defined reason code.
A verifier MUST treat the protected set as authoritative for integrity validation.
A verifier MUST validate the integrity proof (digest/signature/proof) over the stable serialization bytes as defined by the stability scheme.
If a field listed in protectedFields is missing, the verifier MUST fail with protectedFieldMissing, unless omission semantics are explicitly defined by the stability scheme / protected set rules.
If both protectedSetId and protectedFields are present, protectedFields is authoritative; protectedSetId identifies the intended profile/version for audit reference.
If chain fields are absent, chainValid MUST be true (chain is not applicable).
Verification MUST be deterministic given identical artifact + verification material.

9.1 Required verification output schema

{
“result”: “PASS” | “FAIL”,
“reason”: “string | null”,
“checks”: {
“schemaSupported”: true | false,
“integrityValid”: true | false,
“protectedSetValid”: true | false,
“chainValid”: true | false
}
}

9.2 Required minimum reason codes

• integrityProofMismatch
• unsupportedSchema
• protectedFieldMissing
• chainBreakDetected
• signatureInvalid
• verificationMaterialUnavailable
• malformedArtifact
• incompleteArtifact

Verifiers MAY define additional reason codes, but MUST preserve these base semantics.

9.3 Recommended structured diagnostics (Non-Normative)

On FAIL, verifiers are RECOMMENDED to include:
• field (field/path associated with failure when applicable)
• expected (expected digest/signature/value)
• observed (observed digest/signature/value)
• notes (human-readable guidance)

9.4 Redaction and omission semantics (Normative)

If a protected field may be absent due to redaction, the artifact MUST define omission semantics via one of:
• a stable null placeholder, or
• a stable redaction envelope (e.g., { “redacted”: true, “hash”: “sha256:…” }), or
• an explicit redactionPolicyId whose rules define omission handling.

If none is declared and a protected field is missing, the verifier MUST fail with protectedFieldMissing.
Control Objectives

Each control includes Objective → Baseline requirements → Evidence → Test procedures. Controls emphasize properties over primitives: strict on outcomes, flexible on implementation.

10.0 Evidence Packs (Non-Normative)

Execution artifacts are optimized for deterministic integrity verification. In higher-impact contexts, interpretability typically requires additional materials (prompt templates, policy text at time of execution, tool response bodies, reviewer notes). AIEF uses the term Evidence Pack for this companion bundle.

Recommended pattern:
• artifact includes evidencePack reference + evidencePackHash (hash over pack manifest or archive)
• pack contains a manifest + per-item hashes
• access is governed by reviewer class; the artifact remains the anchor

AIEF-01 — Execution Artifact Completeness

Objective
Capture AI executions as verifiable artifacts representing the decision context and outputs.

Baseline requirements (recommended starting point)
An artifact SHOULD include:
• executionId (unique identifier)
• issuedAt (timestamp)
• application/workflow identifier (or equivalent)
• decisionType: ADVISORY | OPERATIVE (recommended)
• declared model/provider identifier (when applicable)
• declared policy/ruleset identifier (when applicable)
• inputs OR inputHash (when redaction required)
• outputs OR outputHash (when redaction required)
• parameters relevant to decision (thresholds, temperature, etc.)
• artifact schema/protocol version
• integrity block identifying stability scheme + proof + protected set

For streaming/long-running workflows, the artifact may be a snapshot, an event stream, or a verifiable data structure that preserves equivalent information.

For aggregated decisions (batch scoring, daily adjustments), implementations SHOULD either:
(a) emit one artifact per execution plus a linking decision artifact, or
(b) emit a single artifact that documents aggregation semantics and references component executions.

Time-source note: issuedAt SHOULD be derived from a synchronized, auditable time source; if not, implementations SHOULD document time-source assumptions.

Evidence
• artifact samples (machine readable)
• schema definition
• mapping between workflow events and artifacts
• (if used) evidence pack manifest format and reference scheme

Test procedures
• sample N executions and confirm required fields are present (or explicitly marked as not captured with rationale)
• confirm artifacts correspond to real business decisions/events
• confirm identifiers are unique and traceable
• confirm at least one Reviewer Class is defined with documented access to interpretive materials sufficient to review decision basis (artifact fields and/or Evidence Pack)
• confirm hashing/redaction does not reduce the system to fully opaque compliance without a documented reviewer access path

AIEF-02 — Tamper-Evidence

Objective
Execution artifacts must possess the property of tamper-evidence: modification of protected fields is detectable via deterministic verification.

Baseline requirements
• the system defines a protected set of fields
• the artifact is verifiable such that modification of any protected field is detectable
• verification yields PASS/FAIL and reason codes when integrity cannot be established
• artifacts declare a stabilitySchemeId that specifies stable serialization rules sufficient for independent verification
• artifacts SHOULD be self-describing by including protectedFields alongside protectedSetId

Protected set governance requirement
• Implementations MUST document which meaningful fields are excluded from the protected set and why (e.g., mutable annotations, non-evidentiary operational metadata). This documentation is part of conformance evidence.

Implementation examples (Non-Normative)
Tamper-evidence may be implemented using one or more of:
• cryptographic digests over stable serialization
• digital signatures with verifiable public keys
• append-only/immutable storage plus an integrity proof
• Merkle commitments over event streams for long-running executions

Evidence
• integrity field(s) (digest/signature/proof reference)
• stability/canonicalization scheme description
• protected set definition and exclusions rationale
• verifier output samples (PASS/FAIL)

Test procedures
• mutate a protected field → verification fails
• re-serialize the same logical artifact → verification is stable
• verify in at least two environments → consistent PASS/FAIL result

AIEF-03 — Deterministic Verification Procedure

Objective
Provide deterministic verification producing PASS/FAIL with actionable reason codes.

Baseline requirements
• verifier is deterministic
• verifier is version-aware (schema/protocol versions)
• output includes PASS/FAIL and reason codes
• verifier checks the integrity mechanism and protected set semantics
• verifier behavior for missing/omitted/redacted protected fields is explicit and deterministic

Evidence
• verification tool documentation
• example PASS + FAIL outputs

Test procedures
• verify artifacts repeatedly → identical results
• verify artifacts across environments → consistent results
• confirm reason codes map to specific failure causes

AIEF-04 — Version Preservation and Context Pinning

Objective
Preserve declared context needed to interpret historical decisions.

Baseline requirements
Artifacts SHOULD preserve:
• artifactVersion/protocolVersion
• model identifier and version (when available)
• runtime version (when applicable)
• policy/ruleset version (when applicable)

Interpretability note (Non-Normative)
In higher-stakes contexts, preserving identifiers alone (e.g., “model: X”, “policy: Y”) may be insufficient if referenced materials (policy text, thresholds, prompt templates, model cards, interpretive docs) are not archived or retrievable. AIEF does not mandate archival of model weights or full policy text, but implementers should define a companion archival or retrieval policy where interpretability is required.

Minimal example: retain a snapshot of the policy document or prompt template referenced by rulesetId/version at the time of execution, stored in an internal repository and accessible to authorized reviewer classes on request for the artifact retention period.

Evidence
• artifact fields populated
• version change log (schema/protocol semantics)
• (if applicable) archival/retrieval policy for referenced interpretive materials

Test procedures
• confirm version fields are present and meaningful
• confirm historical artifacts remain interpretable after updates (per the declared archival/retrieval policy)
• confirm semantics changes trigger explicit version increments

AIEF-05 — Independent Validation Capability

Objective
Enable verification without relying solely on trust in the originating application.

Baseline requirements
• verification should be feasible using the artifact + documented method
• verification should not require privileged access to the originating database
• if signatures are used, verification material (e.g., public keys) should be obtainable via documented means

Multi-tenant note (Non-Normative)
In SaaS, providers may expose verification by offering downloadable artifacts + open verifier, a verification API implementing Section 9 semantics, and/or export bundles enabling offline verification by customers or auditors.

Evidence
• offline verification instructions
• key distribution documentation (if applicable)

Test procedures
• verify artifact outside originating environment
• confirm verification works without internal DB access

AIEF-06 — External Dependency Evidence (Tool Calls)

Objective
Preserve evidence of external dependencies where they influence decisions.

Baseline requirements
By default, implementations SHOULD record all tool calls associated with an execution, including at least:
• tool identifier
• timestamp
• outputHash (or evidence reference)

If an implementation excludes specific tool calls, it MUST provide explicit justification and document exclusion criteria.

Operational definition (Non-Normative, used for exclusion justification)
A tool call is material if its output is referenced in the decision output, influences a computed score/threshold, or could plausibly change the decision outcome if substituted.

Implementations SHOULD exclude tool calls only when they can justify that the calls are non-material under this definition.

Evidence
• tool call section in artifacts
• evidence pack references (if used)

Test procedures
• sample executions with tool calls and confirm evidence presence
• confirm exclusions (if any) are documented and justified
• confirm privacy controls do not eliminate all dependency traceability
• simulate missing tool evidence → verifier fails deterministically with verificationMaterialUnavailable or incompleteArtifact (as appropriate)

AIEF-07 — Multi-Step Chain Integrity (Agent Runs)

Objective
For multi-step workflows, preserve ordering and prevent undetected insertion/removal/reordering.

Baseline requirements
• steps should link to prior steps via cryptographic linkage (e.g., prevStepHash) or equivalent integrity structure
• run summary should enumerate step hashes (or equivalent proof structure)
• chain verification should fail on deletion/insertion/reorder

Evidence
• step artifacts + run summary
• chain verification output

Test procedures
• remove/reorder steps → chain verification fails
• confirm step ordering is derivable from artifact/proof structure

Human-in-the-loop note (Non-Normative)
Many regulated workflows include human review steps between AI recommendations and final actions. AIEF does not require HITL representation, but implementers may model HITL events as chain steps (e.g., “humanApprove”, “humanOverride”). If a human override changes or supersedes the AI recommendation, that event SHOULD be represented as a chain step capturing the final action and its relationship to the superseded AI output.

AIEF-08 — Retention, Portability, Offline Verification

Objective
Ensure artifacts remain accessible and verifiable over time.

Baseline requirements
• retention policy defined for artifacts and evidence packs
• artifacts should be exportable in a portable format
• offline verification procedure should be documented

Degraded mode semantics (Non-Normative)
If evidence cannot be captured (outage, logging failure, verification material unavailable), artifacts SHOULD be explicitly flagged as partial and verifiers SHOULD fail deterministically with a specific reason code rather than silently passing.

Evidence
• sample exports
• retention policy
• offline verification instructions

Test procedures
• export → verify offline
• verify historical artifacts after system updates

Artifact discovery note (Non-Normative)
AIEF specifies integrity, retention, and portability properties, but does not mandate indexing or lookup semantics (e.g., finding artifacts by subject/date/decision ID). Artifact discovery is application-layer functionality and should be designed to meet domain audit needs.

AIEF-09 — Privacy, Minimization, Redaction Controls

Objective
Support privacy and minimization without breaking integrity guarantees.

Baseline requirements
• redaction/hashing policy documented
• redacted artifacts remain verifiable for retained protected fields
• hashed fields have defined semantics (what bytes are hashed, using which stable serialization)

Protected set interaction (clarifying rule)
For a field to remain in the protected set after redaction, its representation in the artifact MUST remain stable and verifiable (e.g., replace sensitive value with a digest field that remains protected). If a field is removed entirely, it must be removed from protectedFields or omission semantics must be explicitly defined and verifier behavior documented.

Evidence
• redaction policy
• sample redacted artifacts + verification outputs

Test procedures
• confirm redaction/hashing policy exists and is applied consistently
• produce a redacted artifact according to policy → run verifier → confirm deterministic PASS for retained protected fields
• confirm verifier behavior is deterministic and documented for omitted/redacted fields (PASS with defined omission semantics, or FAIL with protectedFieldMissing / incompleteArtifact)
• confirm hash semantics are precisely defined (what bytes are hashed, using which stable serialization)

AIEF-10 — Provenance and Attribution (Optional)

Objective
Preserve provenance signals to support evidentiary defensibility by binding artifacts to identifiable authority or runtime context.

Baseline requirements
Where implemented, artifacts SHOULD preserve at least one of:
• signer identifier
• runtime identity
• key identifier (kid)
• delegation context (if applicable)
• trusted timestamp or issuance context

If signatures are used:
• verification material (e.g., public keys) should be obtainable via documented means
• key rotation and key identifiers are recommended
• algorithm identifier (alg) is recommended for cryptographic agility

Evidence
• signer/runtime metadata
• key distribution documentation
• sample verified signed artifacts

Test procedures
• verify signature validity and key ID resolution
• mutate signature/proof → verification fails
• rotate keys → old artifacts remain verifiable where policy requires

Coverage note
Some threats (e.g., key compromise and stronger provenance/authority claims) are not fully mitigated by Levels 1–4 in deployments that require authenticity assurances beyond “unchanged since issuance.” Where provenance, signer authority, or non-repudiation are required, AIEF-10 is strongly recommended.
Security Considerations and Threat Model

AIEF controls aim to reduce the risk of undetected manipulation of execution evidence.

11.1 Threat examples
• post-hoc tampering of inputs/outputs
• selective deletion of unfavorable artifacts
• step insertion/removal/reordering in agent chains
• tool output replacement while preserving logs
• key compromise (if signatures used)
• process replay (resubmitting old artifacts as new)
• silent degraded capture modes

11.2 Mitigation examples (Non-Prescriptive)
• tamper-evident integrity mechanisms bound to protected fields
• append-only/immutable storage semantics where appropriate
• chain integrity links for multi-step workflows
• time-bound identifiers and anti-replay checks at ingestion (where applicable)
• key rotation and key identifiers (if signatures used)
• explicit reason codes and audit trail for verification failures

11.3 Side-channels and leakage (privacy note)
AIEF addresses integrity, not confidentiality. Organizations should consider:
• sensitive prompt/context leakage
• data leakage through stored artifacts
• tool call leakage and access controls

Mitigations include minimization, redaction, hashing, authorization, and controlled release.
Threat → Control Mapping (Non-Normative)

This mapping helps auditors and implementers understand how AIEF controls relate to threats.

Threat | Primary controls | Secondary controls | Typical evidence
Post-hoc tampering of protected fields | AIEF-02, AIEF-03 | AIEF-05 | proof mismatch on mutation + verifier FAIL
Unclear protected set semantics | AIEF-02, AIEF-03 | AIEF-09 | protected set definition + exclusions rationale
Selective deletion of artifacts | AIEF-08 | AIEF-10 (optional) | retention/export + missing artifact detection practices
Step insertion/removal/reorder | AIEF-07 | AIEF-03 | chainBreakDetected on reorder/removal
Tool output replacement | AIEF-06 | AIEF-02, AIEF-07 | tool hashes + FAIL on substitution
Key compromise (signature systems) | AIEF-10 (optional) | AIEF-02 | key IDs, rotation policy, signatureInvalid behaviors
Process replay (resubmit as new) | AIEF-01, AIEF-04 | ingestion controls | uniqueness + issuedAt + workflow constraints
Loss of interpretability over time | AIEF-04, AIEF-08 | Evidence Pack | archived policy/prompt snapshots + access path
Privacy controls breaking verification | AIEF-09, AIEF-03 | AIEF-02 | deterministic behavior on redaction/omission
False-but-consistent artifact issuance | Out of scope alone | AIEF-10 + external measures | independent attestation / provenance
Silent degraded capture | AIEF-03, AIEF-08 | AIEF-06 | incompleteArtifact reason codes + partial flags
Conformance Levels (Normative within AIEF)

Conformance levels are cumulative. An organization asserting Level N MUST be prepared to provide evidence for all controls in Level N and below.

Level 1 — Artifact Capture (basic traceability)
Requires: AIEF-01

Level 2 — Tamper-Evidence + Deterministic Verification (baseline for regulated workflows)
Requires: AIEF-01 + AIEF-02 + AIEF-03

Level 3 — Portability + Independent Validation (external audit readiness)
Requires: all prior level controls + AIEF-05 + AIEF-08

Level 4 — Agent Chain + Dependency Traceability (agentic/tool-driven systems)
Requires: all prior level controls + AIEF-06 + AIEF-07
Note: Not all systems require Level 4; select conformance based on impact and architecture.

AIEF-10 (Provenance & Attribution) is optional in v0.2.5 but strongly recommended for high-assurance issuance contexts.
Risk-Tier Guidance (Non-Normative)

AIEF conformance levels do not prescribe risk categorization. However, implementers frequently need guidance on “how much AIEF is appropriate” for a given decision system. Tier assessment SHOULD be applied at the system or workflow (use case) level, not at the individual model level.

AIEF recommends a simple overlay lens based on two factors:
• decision reversibility (ability to reverse or appeal)
• population impact (breadth and severity of harm)

A non-normative sketch:

Tier | Decision Reversibility | Population Impact | Suggested AIEF Target
Tier A | High | Low | Level 1–2
Tier B | High | High | Level 2–3
Tier C | Low | Low | Level 1–2 (irrevocable but limited blast radius)
Tier D | Low | High | Level 3–4 (+ AIEF-10 recommended)

Note: When reversibility is low, the audit burden shifts toward evidence quality; however, low population impact can still justify a Level 1–2 target.

Illustrative mapping examples:
• Credit underwriting workflow → typically Tier C or D → target Level 3–4 (+ AIEF-10 recommended for Tier D)
• Internal support triage recommendation → typically Tier A or B → target Level 1–2 (or Level 2–3 if high volume/high stakes)
Evidence Checklist (Audit-Oriented) (Non-Normative)

Organizations asserting conformance should be able to produce:
• artifact samples (raw + redacted)
• schema/protocol version history
• description of integrity method and protected set
• protected set exclusions rationale
• verifier outputs (PASS and FAIL)
• export bundles + offline verification procedure
• retention and access control policy (artifacts + evidence packs)
• tool call evidence references (if applicable)
• chain integrity evidence (if applicable)
• provenance/attribution evidence (if AIEF-10 implemented)
• documented roles and responsibilities for artifact generation, verifier ownership, and protected-set policy changes (change management)
Interoperability With Observability Pipelines (Non-Normative)

AIEF does not require a separate pipeline. Execution artifacts may be:
• stored alongside existing logs/traces
• attached as structured payloads to distributed tracing spans (e.g., OpenTelemetry)
• emitted as events into existing SIEM/observability platforms (e.g., Splunk, Datadog)
provided integrity properties remain verifiable and downstream processing does not mutate protected fields without detection.
Alignment With Existing Frameworks (Non-Normative)

AIEF is designed to complement broader governance regimes by focusing on execution-artifact integrity.

• NIST AI RMF (voluntary risk management framework) — AIEF can support documentation/traceability practices within governance functions.
• ISO/IEC 42001 (AI management systems) — AIEF can support technical evidence expectations within management systems.
• EU AI Act record-keeping (high-risk systems) — AIEF’s artifact approach may complement record-keeping by adding tamper-evidence and portable verification.

This section is informational and does not claim compliance with external standards.
Known Open Questions (Request for Input)

We explicitly want input on:
Conformance granularity: Are Levels 1–4 sufficient, or should conformance vary by risk category with different required controls?
Privacy-preserving practice: Which techniques are most practical today—redaction, hashing, secure enclaves, ZK proofs, or others?
Interoperability scope: Section 9 defines a minimal verifier contract; should AIEF also define a shared minimal artifact container/schema (beyond illustrative Appendix A) to enable plug-and-play cross-vendor tooling?
Threat model completeness: What critical threat vectors or real-world failure modes are missing?
Tooling boundaries: What minimum evidence is acceptable for external dependencies (full payload vs hash vs reference), and how should that vary by domain?
Profiles: Should AIEF publish domain profiles (lending, healthcare triage, public sector eligibility) that map controls to common regulatory expectations?
Governance and Change Management (Non-Normative)

AIEF recommends quarterly review of:
• field completeness
• stability scheme and versioning rules
• verification stability
• privacy/redaction controls
• protected set policy and exclusions rationale

Where artifacts predate integrity mechanisms, they should be clearly labeled as legacy/non-verifiable (or verifiable under limited conditions). Semantics changes should trigger explicit version increments.

Appendix A — Illustrative Artifact Schema (Non-Normative)

This schema is illustrative; implementations may factor integrity proofs or evidence into external stores provided the artifact contains sufficient references for deterministic verification per Section 9.

{
“artifactVersion”: “aief.artifact.v1”,
“executionId”: “exec_123”,
“issuedAt”: “2026-02-26T10:00:00Z”,
“applicationId”: “app_xyz”,
“workflowId”: “policy-decision”,
“decisionType”: “ADVISORY”,
“provider”: “anthropic”,
“model”: “claude-xyz”,
“parameters”: { “temperature”: 0.2, “threshold”: 0.7 },
“inputs”: { “caseIdHash”: “sha256:…” },
“outputs”: { “decision”: “ALLOW”, “rationaleHash”: “sha256:…” },
“policy”: { “rulesetId”: “rules_v3” },
“runtime”: { “runtimeVersion”: “1.2.0” },
“toolCalls”: [
{ “toolId”: “watchlist_lookup”, “at”: “2026-02-26T10:00:01Z”, “outputHash”: “sha256:…” }
],
“evidencePack”: {
“ref”: “s3://bucket/evidence/exec_123.zip”,
“hash”: “sha256:…”
},
“integrity”: {
“stabilitySchemeId”: “aief.stable.v1”,
“proof”: { “type”: “digest”, “value”: “sha256:…” },
“protectedSetId”: “aief.protected.v1”,
“protectedFields”: [
“artifactVersion”,
“executionId”,
“issuedAt”,
“applicationId”,
“workflowId”,
“decisionType”,
“provider”,
“model”,
“parameters”,
“inputs”,
“outputs”,
“policy”,
“runtime”,
“toolCalls”,
“chain”,
“evidencePack”
]
},
“chain”: { “runId”: “run_abc”, “stepIndex”: 3, “prevStepHash”: “sha256:…” }
}

Appendix B — Illustrative Integrity Mechanisms (Non-Normative)

AIEF does not mandate specific primitives. Examples include:
• collision-resistant hashes (e.g., SHA-256)
• digital signatures (e.g., Ed25519) with key rotation and key IDs
• append-only storage + external attestation (optional)
• Merkle commitments for event streams

Examples only; implementations may vary.

Appendix C — Example Verification Outputs (Non-Normative)

PASS
{
“result”: “PASS”,
“reason”: null,
“checks”: {
“schemaSupported”: true,
“integrityValid”: true,
“protectedSetValid”: true,
“chainValid”: true
}
}

FAIL
{
“result”: “FAIL”,
“reason”: “integrityProofMismatch”,
“checks”: {
“schemaSupported”: true,
“integrityValid”: false,
“protectedSetValid”: true,
“chainValid”: true
},
“field”: “outputs.decision”,
“expected”: “sha256:…”,
“observed”: “sha256:…”
}

Appendix D — Fintech Example: Certified Loan Decision (Non-Normative)

Scenario
A lender uses AI to recommend approve/deny based on application data and policy thresholds.

Artifact should preserve
• applicant data hash (not raw PII)
• policy/ruleset version (and retrieval/archival access path)
• model ID/version (when available)
• decision output + key factors (hashed if sensitive)
• issuance time + executionId
• tamper-evidence binding all protected fields
• tool call hashes or evidence references for material dependencies (e.g., sanctions list, bureau pull)

Audit questions enabled
• “Was this decision altered after issuance?”
• “Which policy version and threshold applied?”
• “Can we validate integrity offline months later?”
• “Which external data influenced the score (at least via hashes/references)?”

Conformance target: Tier-dependent (see Section 14). Example: a Tier D underwriting workflow would typically target Level 3–4 (+ AIEF-10 recommended).

Appendix E — Agent Example: Multi-Step Fraud Investigation (Non-Normative)

Scenario
An agent workflow:
fetch account history
fetch watchlist matches
compute risk score
decide block/escalate
human override (optional)

Required properties
• each step artifact has tamper-evidence
• step N references prior step via verifiable linkage
• run summary lists step proofs/hashes
• tool call outputs preserved as hashes or evidence references
• human overrides recorded as chain steps when they change/supersede the AI recommendation

Audit questions enabled
• “Can a step be removed without detection?” (should be NO)
• “Can we prove which external data influenced the score?” (at least via hashes/references)
• “Was the final action aligned with or divergent from the AI recommendation?”

Conformance target: Level 4 (Tier-dependent).

Appendix F — Change Log (Non-Normative)

v0.2.5 — 2026-02-27
• Section 5: Added Reliance Statement to clarify what verification does and does not prove.
• Section 6: Added Evidence Pack and Reviewer Class definitions; clarified execution vs decision vs action semantics.
• Section 7: Formalized cache-as-truth as a design principle.
• Section 9: Added normative interpretation rules, redaction/omission semantics, chainValid semantics, and incompleteArtifact reason code; elevated structured diagnostics to recommended.
• Section 10: Formalized Evidence Pack concept; strengthened protected set governance (exclusions rationale); clarified AIEF-06 materiality as exclusion justification; added aggregation guidance; added time-source note; strengthened HITL note.
• Section 13/14: Removed duplication by consolidating conformance model into a single normative section.
• Section 14: Fixed risk-tier table logic (Tier C target) and added workflow-level tier note + examples.
• Section 15: Added roles/responsibilities evidence item for audit readiness.

