
VS Code Copilot Agent Plugin Lifecycle Assurance Engine
Handover Specification & Architecture Blueprint (v1.1)
Target Audience: Autonomous Implementation Agent / VS Code Copilot Engineering Agent
Primary Invariant: For every published plugin version and declared capability, the system must prove—via reproducible, machine-verifiable evidence—the complete execution path from immutable source revision to runtime execution, or isolate the exact failure state and root cause.
1. System Overview & Problem Statement
The existing VS Code Copilot Plugins Marketplace experiences high runtime failure rates because marketplace indexing, artifact downloading, and local file placement do not guarantee capability discovery, UI binding, or successful runtime execution.
An Agent Plugin is a distributed composite of metadata, prompt configurations, tool bindings, Model Context Protocol (MCP) servers, and execution hooks. This system functions as a standalone Automated Certification & Governance Gatekeeper ("App Store Pre-Flight & Staging Gateway"). It intercepts candidate plugins, places them in a quarantined state, validates them across an isolated headless VS Code sandbox environment, scores runtime execution using multi-tier AI test oracles, and publishes only cryptographically verified artifacts.
2. Architectural Topography & System Boundaries
This system is decoupled from the core marketplace repository and operates as an independent service with its own compute resources, worker pool, and storage.
 ┌─────────────────────────────────────────────────────────────────────────────────────────┐
 │                                   CONTRIBUTOR PORTAL                                    │
 │  • Submit Repo / VSIX / Manifest   • Real-Time Simulation Trace   • Diagnostic Auto-Fix │
 └────────────────────────────────────────────┬────────────────────────────────────────────┘
                                              │
                                              ▼
 ┌─────────────────────────────────────────────────────────────────────────────────────────┐
 │                   GOVERNANCE WEB APP & DEV SERVER (Node.js / TS)                        │
 │  ┌─────────────────────────┐ ┌─────────────────────────┐ ┌───────────────────────────┐  │
 │  │    Policy Engine &      │ │   State Machine &       │ │      Evidence & Audit     │  │
 │  │   Compliance Checks     │ │   Orchestrator Worker   │ │     Cryptographic Store   │  │
 │  └───────────┬─────────────┘ └───────────┬─────────────┘ └─────────────▲─────────────┘  │
 └──────────────┼───────────────────────────┼─────────────────────────────┼────────────────┘
                │                           │                             │
                ▼                           ▼                             │ Signed Evidence[span_6](start_span)[span_6](end_span)
 ┌──────────────────────────────┐ ┌───────────────────────────────────────┴────────────────┐
 │    SECURITY / STATIC GATE    │ │         SANDBOX SIMULATION HARNESS (Docker / VM)       │
 │ • AST & Secret Scan          │ │ • Headless VS Code + Extension Host                    │
 │ • Schema & SemVer Validation │ │ • Capability Reachability Test (Slash `/`, MCP, Agent) │
 │ • Dependency Graph Check     │ │ • AI Oracle Tiers 1–4 (Deterministic to LLM Judge)    │
 └──────────────────────────────┘ └────────────────────────────────────────────────────────┘
                                              │
                                              │ (Only if RELEASE_ELIGIBLE / GO)[span_7](start_span)[span_7](end_span)
                                              ▼
 ┌─────────────────────────────────────────────────────────────────────────────────────────┐
 │                      EXISTING VS CODE COPILOT MARKETPLACE                               │
 │                   (Unquarantine, Index, & Distribute to Users)                          │
 └─────────────────────────────────────────────────────────────────────────────────────────┘

Architectural Decisions
 * Repository Strategy: Standalone repository and service. Running untrusted contributor scripts, spawning MCP child processes, and hosting headless VS Code containers requires isolated compute to eliminate security blast radius and prevent resource starvation on marketplace search APIs.
 * Integration Strategy: Asynchronous webhook quarantine gate. All plugin uploads default to STATUS: QUARANTINED until the engine returns a signed evidence.schema.json verifying release eligibility.
3. End-to-End Traceability & Lifecycle Proof Graph
3.1 Immutable Identity Chain
Every operational event must carry an identity tuple traceable end-to-end:
Contributor Repo URI[span_13](start_span)[span_13](end_span)
  └── Branch Name[span_14](start_span)[span_14](end_span)
       └── Commit SHA (40-char hex)[span_15](start_span)[span_15](end_span)
            └── CI Build Run ID[span_16](start_span)[span_16](end_span)
                 └── Marketplace Entry ID[span_17](start_span)[span_17](end_span)
                      └── Plugin ID (`publisher.name`)[span_18](start_span)[span_18](end_span)
                           └── SemVer (`MAJOR.MINOR.PATCH`)[span_19](start_span)[span_19](end_span)
                                └── Artifact Checksum (SHA-256)[span_20](start_span)[span_20](end_span)
                                     └── Installed Local Directory[span_21](start_span)[span_21](end_span)
                                          └── Loaded Runtime Instance UUID[span_22](start_span)[span_22](end_span)
                                               └── Capability Unique Identifier[span_23](start_span)[span_23](end_span)

3.2 Formal Lifecycle Proof Graph
                                  GOAL[span_24](start_span)[span_24](end_span)
                                   │
                                   ▼
                          ACCEPTANCE CRITERIA[span_25](start_span)[span_25](end_span)
                                   │
                                   ▼
                               PLUGIN ID[span_26](start_span)[span_26](end_span)
                                   │
                    ┌──────────────┴──────────────┐
                    ▼                             ▼
               REPOSITORY[span_27](start_span)[span_27](end_span)            VERSION[span_28](start_span)[span_28](end_span)
                    │                             │
                    ▼                             ▼
               COMMIT SHA ─────────────────► MARKETPLACE[span_29](start_span)[span_29](end_span)
                                                  │
                                                  ▼
                                              DISCOVERY[span_30](start_span)[span_30](end_span)
                                                  │
                                                  ▼
                                               INSTALL[span_31](start_span)[span_31](end_span)
                                                  │
                                                  ▼
                                             LOCAL STATE[span_32](start_span)[span_32](end_span)
                                                  │
                                                  ▼
                                                LOAD[span_33](start_span)[span_33](end_span)
                                                  │
                                                  ▼
                                          CAPABILITY GRAPH[span_34](start_span)[span_34](end_span)
                                                  │
                            ┌─────────────────────┼─────────────────────┐
                            ▼                     ▼                     ▼
                          Agent[span_35](start_span)[span_35](end_span)       Skill[span_36](start_span)[span_36](end_span)        MCP[span_37](start_span)[span_37](end_span)
                            │                     │                     │
                            └─────────────────────┼─────────────────────┘
                                                  ▼
                                           SURFACE EXPOSURE[span_38](start_span)[span_38](end_span)
                                                  │
                                                  ▼
                                              INVOCATION[span_39](start_span)[span_39](end_span)
                                                  │
                                                  ▼
                                              EXECUTION[span_40](start_span)[span_40](end_span)
                                                  │
                                                  ▼
                                               EVIDENCE[span_41](start_span)[span_41](end_span)
                                                  │
                                                  ▼
                                              VALIDATION[span_42](start_span)[span_42](end_span)
                                                  │
                                       ┌──────────┴──────────┐
                                       ▼                     ▼
                                     PASS[span_43](start_span)[span_43](end_span)        FAIL[span_44](start_span)[span_44](end_span)
                                       │                     │
                                       ▼                     ▼
                                    RELEASE[span_45](start_span)[span_45](end_span)     DIAGNOSIS[span_46](start_span)[span_46](end_span)
                                                             │
                                                             ▼
                                                        REMEDIATION[span_47](start_span)[span_47](end_span)
                                                             │
                                                             ▼
                                                           RETEST[span_48](start_span)[span_48](end_span)

4. 14-State Installation State Machine & Atomicity
The plugin manager and verification engine strictly execute the 14-state lifecycle machine:
DISCOVERED ──► AVAILABLE ──► DOWNLOAD_STARTED ──► DOWNLOADED ──► VALIDATED ──► INSTALLED[span_50](start_span)[span_50](end_span)
                                                                                   │
READY ◄── DISCOVERED_CAPABILITIES ◄── LOADED ◄── ENABLED ◄── REGISTERED ◄──────────┘[span_51](start_span)[span_51](end_span)
  │
  └── (On Error at Any Point) ──► FAILED ──► PARTIAL ──► ROLLBACK[span_52](start_span)[span_52](end_span)

Atomicity Invariants
 * Extract-to-Temp & Atomic Rename: All package downloads stage in .tmp/ scratch storage. Manifest and checksum validations execute prior to moving files into active execution paths.
 * Failure at 10% (Download drop): Incomplete buffers wiped; state returns to AVAILABLE.
 * Failure at 50% (Unpack corruption / Disk full): Staging directory deleted; zero registry entries written.
 * Failure at 90% (Post-install validation failure): Registry entry purged, directory deleted, active memory unmapped, state transitioned to FAILED -> ROLLBACK. Zero partially installed artifacts may remain in active plugin paths.
5. Scope Precedence, Environments & Component Graphs
5.1 Scope Precedence Hierarchy
When conflicting components share an identifier, resolve strictly in priority order:
 * Repository Scope: .github/copilot-plugins/*
 * Workspace Scope: .vscode/settings.json
 * Profile Scope: Active VS Code Profile Customizations
 * User Scope: ~/.copilot/plugins/* or global settings
 * System Scope: Managed enterprise policies
 * Plugin Default: Manifest base definitions
5.2 Component Dependency Graph
Plugin (Container & Manifest)[span_66](start_span)[span_66](end_span)
 ├── Agent (Prompt orchestration + execution policy)[span_67](start_span)[span_67](end_span)
 │    ├── Skill (Specialized instructions & task procedures)[span_68](start_span)[span_68](end_span)
 │    ├── MCP Server (Model Context Protocol endpoints)[span_69](start_span)[span_69](end_span)
 │    └── Tool (Discrete computational functions)[span_70](start_span)[span_70](end_span)
 ├── Skill (Standalone prompt extensions)[span_71](start_span)[span_71](end_span)
 ├── Command (Slash commands bound to chat surfaces)[span_72](start_span)[span_72](end_span)
 ├── Prompt (Reusable parameterized `.prompt.md` files)[span_73](start_span)[span_73](end_span)
 ├── Hook (Lifecycle triggers: `onInstall`, `preInvoke`, `postInvoke`)[span_74](start_span)[span_74](end_span)
 └── Rule (Static validation and guardrail definitions)[span_75](start_span)[span_75](end_span)

5.3 Capability Reachability & Negative Invariants
 * Reachability Invariant: Declared -> Installed -> Loaded -> Discovered -> Reachable -> Invocable -> Executable -> Successful.
 * Negative Test Cases:
   * Skills marked user-invocable: false must never appear in Chat / command completions.
   * Skills marked disable-model-invocation: true must never be injected into model tool-definition buffers.
   * Malformed frontmatter in SKILL.md must trigger immediate loader errors and cleanly isolate or block plugin registration.
6. Multi-Tier AI Test Oracle & Evidence Model
Generative non-deterministic agent outputs are verified through a four-tiered oracle hierarchy:
Tier 1: Deterministic Assertions[span_81](start_span)[span_81](end_span)
  ├── JSON/YAML syntax validation[span_82](start_span)[span_82](end_span)
  ├── File system path checks[span_83](start_span)[span_83](end_span)
  └── Tool parameter schema matching[span_84](start_span)[span_84](end_span)

Tier 2: Semantic Assertions[span_85](start_span)[span_85](end_span)
  ├── Cosine similarity of response embeddings vs. ground truth (> 0.88)[span_86](start_span)[span_86](end_span)
  └── Required key phrase / entity containment[span_87](start_span)[span_87](end_span)

Tier 3: Probabilistic Assertions[span_88](start_span)[span_88](end_span)
  ├── Pass rate across N=5 repeated execution runs (>= 80%)[span_89](start_span)[span_89](end_span)
  └── Tool selection frequency distribution within acceptable tolerance[span_90](start_span)[span_90](end_span)

Tier 4: LLM-as-a-Judge Evaluation[span_91](start_span)[span_91](end_span)
  ├── Multi-turn evaluation rubrics: Correctness, Tone, Safety (Threshold >= 4/5)[span_92](start_span)[span_92](end_span)
  └── Reasoning chain validation against ACCEPTANCE.md criteria[span_93](start_span)[span_93](end_span)

Cryptographic Evidence Schema (evidence.schema.json)
{
  "$schema": "https://agent-plugins.org/schemas/1.1.0/evidence.schema.json",
  "testRunId": "run-89234-a8f1",
  "correlationId": "commit-f39b1a:ci-4412:pkg-1.2.0",
  "timestamp": "2026-08-29T10:15:30Z",
  "identity": {
    "repository": "https://dev.azure.com/org/project/_git/plugin-repo",
    "commitSha": "f39b1a09d3b7642134812a67e584f275e7a90b41",
    "pluginId": "enterprise.devops-agent",
    "version": "1.2.0"
  },
  "environment": {
    "os": "Linux x86_64",
    "vscodeVersion": "1.98.0",
    "target": "DevContainer"
  },
  "assertionResults": [
    {
      "capabilityId": "code-review",
      "surface": "agent-selector",
      "reachabilityState": "EXECUTABLE",
      "oracleTier": "Tier4_LLMJudge",
      "score": 4.8,
      "verdict": "PASS",
      "telemetryTrace": "trace-uuid-9901-abcd"
    }
  ],
  "finalVerdict": "GO"
}
```[span_94](start_span)[span_94](end_span)

---

## 7. Security, Resilience & Regression Baselines

* **Supply-Chain Integrity:** Cryptographically enforce `Signed Commit SHA -> Attested Build -> Registry SHA-256 -> Download Checksum -> Local Filesystem Hash`[span_95](start_span)[span_95](end_span).
* **MCP & Hook Sandboxing:** MCP servers spawned over stdio/SSE must execute as unprivileged child processes isolated from parent environment secrets[span_96](start_span)[span_96](end_span). Secrets matching `/(KEY|TOKEN|SECRET|PASSWORD|AUTH|CREDENTIAL)/i` must be redacted from logs[span_97](start_span)[span_97](end_span).
* **Concurrent Update Safety:** When an update installs while an agent executes, active executions maintain a read-lock on the prior version directory[span_98](start_span)[span_98](end_span). New installations populate an isolated version directory and bind to subsequent invocations[span_99](start_span)[span_99](end_span).
* **Differential Regression Baseline:** Candidate versions (`vN+1`) are diffed against baseline snapshots (`vN`)[span_100](start_span)[span_100](end_span). If capability counts drop, latencies spike unexpectedly, or permissions expand without declaration, flag `REGRESSION_FAIL`[span_101](start_span)[span_101](end_span).
* **Self-Healing Rule:** Diagnostic agents may auto-repair schemas, file paths, and syntax errors, but **are strictly prohibited from weakening acceptance criteria or assertion thresholds to force a PASS**[span_102](start_span)[span_102](end_span).

---

## 8. Marketplace Integration Contract

### Ingestion Webhook (Marketplace -> Assurance Engine)
```http
POST /api/v1/certify-candidate
Content-Type: application/json
X-Hub-Signature-256: sha256=<hmac_signature>

{
  "pluginId": "enterprise.devops-agent",
  "version": "1.2.0",
  "artifactUrl": "https://marketplace-storage.internal/quarantine/devops-agent-1.2.0.vsix",
  "artifactChecksum": "9f83c6d...",
  "sourceRepo": "https://dev.azure.com/org/project/_git/plugin-repo",
  "commitSha": "f39b1a09d3b7642134812a67e584f275e7a90b41",
  "targetEnvironments": ["Desktop", "DevContainer", "RemoteSSH"]
}
```[span_103](start_span)[span_103](end_span)

### Callback Webhook (Assurance Engine -> Marketplace)
```http
POST /api/v1/certification-callback
Content-Type: application/json
X-Hub-Signature-256: sha256=<hmac_signature>

{
  "pluginId": "enterprise.devops-agent",
  "version": "1.2.0",
  "verdict": "GO",
  "evidenceReportUrl": "https://assurance-storage.internal/reports/run-89234-a8f1.json",
  "evidenceSignature": "ecdsa-sig-3045...",
  "unquarantine": true
}
```[span_104](start_span)[span_104](end_span)

---

## 9. Failure Taxonomy

All test and runtime failures must map to this ontology[span_105](start_span)[span_105](end_span):

```text
ERR_LIFECYCLE_TAXONOMY[span_106](start_span)[span_106](end_span)
 ├── ERR_SOURCE (ERR_SRC_MANIFEST_INVALID, ERR_SRC_CIRCULAR_DEPENDENCY, ERR_SRC_FILE_NOT_FOUND)[span_107](start_span)[span_107](end_span)
 ├── ERR_MARKETPLACE (ERR_MKT_SYNC_TIMEOUT, ERR_MKT_HASH_MISMATCH, ERR_MKT_VERSION_COLLISION)[span_108](start_span)[span_108](end_span)
 ├── ERR_INSTALL (ERR_INS_ATOMIC_RENAME_FAILED, ERR_INS_DISK_FULL, ERR_INS_CORRUPT_ARCHIVE)[span_109](start_span)[span_109](end_span)
 ├── ERR_SECURITY (ERR_SEC_PROVENANCE_FAILED, ERR_SEC_UNAUTHORIZED_ENV_ACCESS, ERR_SEC_PROMPT_INJECTION)[span_110](start_span)[span_110](end_span)
 ├── ERR_SURFACE (ERR_SURF_SLASH_COMMAND_UNREGISTERED, ERR_SURF_AGENT_PICKER_MISSING, ERR_SURF_SCOPE_COLLISION)[span_111](start_span)[span_111](end_span)
 └── ERR_RUNTIME (ERR_RUN_MCP_PROCESS_CRASH, ERR_RUN_ORACLE_TIER_FAIL, ERR_RUN_TIMEOUT)[span_112](start_span)[span_112](end_span)

10. Implementation Sequence for the Next Engineer / Agent
Execute implementation strictly in the following order:
 * Phase 1: Schemas & Static Verification
   * Define plugin.schema.json, capabilities.manifest.yaml, and evidence.schema.json.
   * Implement AST manifest parsers and acyclic dependency graph validators.
 * Phase 2: Headless VS Code Sandbox Harness
   * Build Dockerized test runner with headless VS Code and Extension Host drivers.
   * Implement simulated installation, atomic file extraction, and network fault injection (10%, 50%, 90%).
 * Phase 3: Surface & Reachability Verification
   * Implement RPC inspectors to verify that skills, slash commands, agents, and MCP servers physically register on target VS Code UI surfaces.
 * Phase 4: AI Oracle & Regression Engine
   * Build Tier 1–4 assertion runners and baseline snapshot diffing.
   * Integrate cryptographic signing for generated evidence records.
 * Phase 5: Web API & Marketplace Quarantine Hooks
   * Implement /api/v1/certify-candidate and /api/v1/certification-callback endpoints with HMAC authentication.
   * Build the web management dashboard displaying live Reachability DAGs and self-healing diagnostic diffs.
