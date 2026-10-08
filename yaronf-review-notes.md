# Documentation review notes

Working notes from reviewing the local docs preview (`http://127.0.0.1:8000/`).
Each `##` heading is one comment; titles are issue-ready.

**Open-issue check:** Before keeping a comment, search open issues on [`GenAI-Security-Project/agent-control-standard`](https://github.com/GenAI-Security-Project/agent-control-standard/issues). Drop the comment if an open issue fully covers it (same ask). Related-but-narrower issues do not count as duplicates.

Checked against open issues (2026-10-08): no comment fully duplicates an open issue. Closest overlaps (kept): large-payload note ↔ [#155](https://github.com/GenAI-Security-Project/agent-control-standard/issues/155) (truncation only); Approver/ASK clarity ↔ [#175](https://github.com/GenAI-Security-Project/agent-control-standard/issues/175) / [#51](https://github.com/GenAI-Security-Project/agent-control-standard/issues/51); MODIFY/ASK on handshake ↔ [#147](https://github.com/GenAI-Security-Project/agent-control-standard/issues/147) (substitution representation, not ClientHello declaration); scope/audience ↔ [#133](https://github.com/GenAI-Security-Project/agent-control-standard/issues/133) (FAQ page, not agent-class scope).

## Top 5

Curated as we go; re-rank when a new finding outranks an entry below.

1. [Define ACS scope and target audience across agent classes](#define-acs-scope-and-target-audience-across-agent-classes) — who the standard is for is never stated.
2. [Call the HMAC baseline a MAC, and justify why not ECDSA-P256](#call-the-hmac-baseline-a-mac-and-justify-why-not-ecdsa-p256) — Core integrity floor is misnamed and under-justified.
3. [Put MODIFY (and ASK) capability on the handshake wire](#put-modify-and-ask-capability-on-the-handshake-wire) — Core negotiates capabilities, then leaves these off the wire.
4. [Clarify what control paradigms mean for implementers](#clarify-what-control-paradigms-mean-for-implementers) — IBAC/FIDES/CaMeL/AARM read like a product matrix, not optional citation styles.
5. [Separate chain-integrity forensics from chain-based policy enforcement](#separate-chain-integrity-forensics-from-chain-based-policy-enforcement) — naming chains helps audit; deep-chain policy is unproven.

---

## Add A2A hook pages to the MkDocs nav

**Where:** `mkdocs.yml` nav; source pages under `docs/spec/instrument/a2a/hooks/`

**Notes:** MkDocs build warns that these pages exist but are not in nav, so they are hard to reach from the site:

- `cancel_task_request.md`
- `get_task_push_notification_config_request.md`
- `get_task_request.md`
- `resubscribe_to_task_request.md`
- `send_message_request.md`
- `set_task_push_notification_config_request.md`
- `stream_message_request.md`

---

## Rename the Instrument "ACS specification" nav label

**Where:** `mkdocs.yml` → Specification → Instrument → `ACS specification` (`spec/instrument/specification.md`)

**Notes:** "ACS specification" sits two levels under the top-level "Specification" section, which reads as recursive and confusing. Rename the leaf (e.g. to match the page's role under Instrument: protocol/hooks contract) so the nav path is not Specification → … → ACS specification.

---

## Define ACS scope and target audience across agent classes

**Where:** Top-level docs (landing / About / Topics intro); no explicit audience statement found

**Notes:** "Agents" in ACS can mean several very different deployments: enterprise production software agents, coding agents, and personal agents of the Claw ilk. Readers cannot tell which class the standard is written for, or whether all three are in scope. Add an explicit scope and target-audience statement that names these classes and what ACS claims to cover for each.

Checked: `docs/README.md`, `docs/about.md`, and pillar intros frame ACS around enterprises ("the enterprise that welcomes them in", "Enterprises need to see…") but never enumerate agent classes or say whether coding agents or personal agents are in or out of scope.

On ASK: mandatory ASK does **not** by itself make coding agents the primary scope. Guardians MUST support ASK, but approvers MAY be human, agent, or service (§9), and §9.2 substitutes DEFER/DENY for approver-incapable clients (explicitly including headless automation, not only IDE plugins). So interactive coding agents are a natural ASK consumer, not the declared primary audience — but the tension between enterprise framing and interactive-approval gravity is another reason the scope statement is missing.

---

## Balance Inspect transparency against attacker reconnaissance

**Where:** `docs/README.md` — Trustworthy Agents blurb, Inspectable paragraph ("What are the services behind them, and which data they can access.")

**Notes:** The blurb presents that visibility as purely desirable. Too much transparency can also help an attacker map services and data access. The intro should acknowledge the tradeoff, not only the benefit.

---

## Call the HMAC baseline a MAC, and justify why not ECDSA-P256

**Where:** `docs/acs.md` ("baseline HMAC-SHA256 signature"); also `docs/spec/conformance.md`, `docs/spec/instrument/specification.md` §10 (registry correctly labels HMAC as "Symmetric MAC" but the surrounding prose still says "signature")

**Notes:** Formally HMAC-SHA256 is a MAC, not a signature. Calling the Core baseline a "signature" blurs that it cannot provide non-repudiation or bind a compromised Guardian (which the same § already admits). Separately: why is shared-secret HMAC the mandatory baseline rather than a public-key algorithm — ECDSA-P256 being the natural classical choice, already OPTIONAL in the §10.1 registry — with HMAC as an optional same-host shortcut? The adoption/simplicity rationale should be explicit if HMAC stays Core.

---

## Mention ECDSA-P256 in the ACS-Crypto overview blurb

**Where:** `docs/acs.md` — ACS-Crypto profile bullet ("HMAC-SHA256 baseline, ML-DSA-65 / SLH-DSA-128s for PQC, hybrid composites…")

**Notes:** The overview skips classical asymmetric entirely. `ECDSA-P256` is in the §10.1 registry (OPTIONAL, strongest current ecosystem support) and appears only inside hybrid composites in the blurb. A reader scanning the profile list would think ACS-Crypto is HMAC + PQC only. Name ECDSA-P256 (and optionally RSA-PSS) alongside the PQC algorithms, especially given the open question of whether P-256 should be baseline rather than HMAC.

---

## Clarify how Runtime Hooks relate to ACSInstrument

**Where:** `docs/acs.md` — Trustworthy agents are › Instrumentable › "Runtime Hooks" example (follows the earlier "Init agent with ACS" / `ACSInstrument` snippet)

**Notes:** Confusing for newcomers. After wrapping with `ACSInstrument`, this section shows hand-registered `toolCallRequest` hooks that call `guardian.evaluate` themselves. Unclear whether that manual wiring is always required, or only when you need tighter control than `ACSInstrument` already provides. Spell out the relationship (what the wrapper does by default vs when you register hooks).

Same Python snippet only branches on `DENY` and `MODIFY`. ACS dispositions also include `ALLOW`, `ASK`, and `DEFER`, and there is no catch-all path for an unexpected `action`. (Python `if`/`elif` uses `else` for that; `match`/`case` uses `case _:`.) The TypeScript tab's `switch` has the same gap.

---

## Clarify that AgBOM serializations are not required to be signed

**Where:** Inspect definition — `docs/spec/inspect/README.md` (and related CycloneDX/SPDX/SWID extension pages)

**Notes:** Signing a BOM is optional in CycloneDX. The ACS-Inspect profile should state that a signed serialization is not required for conformance. Integrity for Inspect remains the ACS envelope on `agbom/snapshot`/`changed` and the SessionContext chain.

---

## Put MODIFY (and ASK) capability on the handshake wire

**Where:** `docs/spec/instrument/specification.md` §6.5 (MODIFY-incapable clients); same pattern in §9.2 (approver-incapable clients); contrast §4 Capability Negotiation Handshake

**Notes:** §6.5 lists out-of-band ways to learn that a client cannot apply MODIFY (agent identity, `agent_id` policy, org config, "any other signal") and admits ACS does not put the declaration on the wire in v0.1. Those are all unsatisfactory once ACS-Core already requires a capability-negotiation handshake (`methods_implemented`, `profiles_supported`, etc.). Disposition-handling capability (at least MODIFY; same for ASK in §9.2) belongs in ClientHello rather than the Guardian's policy bundle.

---

## Rename the Instrument pillar to name what it does

**Where:** Pillar naming across the docs (e.g. Instrument / Trace / Inspect); `docs/spec/instrument/`, nav, conformance, intros

**Notes:** The three pillars are Instrument, Trace, and Inspect. Trace and Inspect name outcomes; Instrument names the mechanism (hooks) rather than the job (runtime permit/deny/modify of agent actions). Prefer a verb that says what it does — e.g. Govern, Police, or Control — so the triad is parallel.

---

## Include SPIFFE (and future WIMSE) in identity type examples

**Where:** `docs/concepts/identity.md`, `docs/topics/core_concepts.md`, `docs/spec/instrument/specification.md` §11 (identity descriptor `type` examples: `posix_uid`, `oauth_subject`, `cert_subject`, …)

**Notes:** The example discriminator list skips workload identity. SPIFFE IDs (and, later, WIMSE identifiers) are the natural types for agent/workload principals, and `docs/identity/standards.md` already proposes WIMSE/SPIFFE as the recommended model. Add `spiffe_id` (and a forward pointer to WIMSE) to the illustrative `type` set so the core identity text matches the identity workstream.

---

## Consider RFC 9493 Subject Identifiers instead of ad-hoc oauth_subject

**Where:** `docs/concepts/identity.md` (and related identity type examples); Identity workstream descriptor schema

**Notes:** SECEVENT published [RFC 9493](https://www.rfc-editor.org/rfc/rfc9493.html) (Subject Identifiers for Security Event Tokens): typed JSON subjects with a `format` discriminator (`iss_sub`, `email`, `uri`, `did`, `opaque`, …) and an IANA registry. `iss_sub` is a better stand-in for OAuth/OIDC principals than ACS's illustrative `oauth_subject` + bare `sub` (issuer+subject is what makes the id unique). Adopted mainly via OpenID Shared Signals / CAEP / RISC (e.g. Okta SSF); not a universal OAuth wire replacement yet. Worth aligning ACS identity descriptors with that registry (and using `uri`/`did` for SPIFFE/WIMSE) rather than growing a parallel type list.

---

## Identity "verified credential" claim has no ACS wire support (e.g. PoP)

**Where:** `docs/concepts/identity.md` ("An asserted identity is weaker than one bound by a verified credential or signature"); related trust-basis language in `docs/topics/core_concepts.md`

**Notes:** The sentence implies ACS can distinguish asserted vs credential-bound identities. On the wire, identity descriptors are just typed claims; there is no PoP / DPoP / mTLS-bound / SVID presentation field, and auth remains deployment-defined (§ identity + handshake). ACS-Crypto attests *envelopes*, not principal credentials. Identity workstream prose requires DPoP/mTLS compositionally (`docs/identity/standards.md`), but that is not expressed in the Core identity descriptor model. Either add how a Guardian learns that an identity was PoP-verified, or soften the claim so it does not read as an ACS feature.

---

## Replace leftover "ASOP" references with ACS

**Where:** `docs/topics/core_concepts.md` diagram (`docs/assets/agent_env.png` — edge labeled ASOP between Observed Agent and Guardian); `docs/spec/instrument/a2a/extend_a2a.md` ("uses ASOP as a transport"); `docs/spec/trace/OCSF/implementation_examples.md` (`ASOP Security Layer`, `unmapped.asop.*`)

**Notes:** "ASOP" is never defined in the current documentation. It reads as a stale project name (alongside historical AOS branding) left in examples and the Core Concepts environment diagram. Rename to ACS (and `unmapped.acs.*` if that namespace is kept) so newcomers are not hunting for a fourth acronym.

---

## ACS-in-action sequence: DENY is legal at every allow, and AgBOM is mis-ordered

**Where:** `docs/topics/ACS_in_action_example.md` — Sequence mermaid

**Notes:** The walkthrough correctly shows `allow` for a permit path, but a reader can take that as the only legal verdict. Per hooks, DENY is allowed at each decision shown: `agbom/snapshot` (banned component), `sessionStart` (refuse session), `userMessage` (ALLOW/DENY/MODIFY), `toolCallRequest` (full vocabulary), `toolCallResult` (ALLOW/DENY/MODIFY). Call that out (note or alternate deny branch), especially since the page later shows deny-shaped examples only for the tool-call paradigms.

Also: the diagram emits `agbom/snapshot` *before* `sessionStart`. Normative ordering is after `sessionStart` and before the first content-bearing hook (`hooks.md` / Inspect README).

---

## Specify large-payload handling: stream, truncate, compress, or refuse — with an indicator

**Where:** Wire format / Instrument — `docs/spec/instrument/specification.md` §3 (streaming deferred to v0.2), §4 `max_payload_size_bytes`; no truncation/compression fields found in schemas

**Notes:** Hook bodies (e.g. tool results, messages, skill definitions) can be very large. v0.1 does not support streaming ACS hook traffic (explicitly deferred to v0.2). There is also no defined truncation mode or wire flag that a payload was truncated — only ClientHello `max_payload_size_bytes` ("willing to send"), with no matching Guardian limit, refuse error, or `truncated`/`content_hash`-only alternative. Compression is likewise unspecified (no content-encoding negotiation on the ACS envelope/transport beyond whatever HTTP might do out of band). Spec should say whether oversized content is refused, hashed/summarized, compressed, or streamed later, and how a Guardian knows it did not see the full body.

Even in v0.1 the Guardian should be able to declare its own receive limit (e.g. on ServerHello), not only the Observed Agent's send willingness — otherwise the Guardian cannot bound memory/DoS risk on the wire.

---

## Document authorization paradigms without forcing them into Concepts

**Where:** IBAC/FIDES/CaMeL/AARM used across Instrument and Topics; no dedicated page. Concepts README altitude rule: cross-cutting → Concepts, mechanism → pillar

**Notes:** Readers need a clear definition of the named paradigms (not common OWASP terms; lightly expanded today). But authz/policy evaluation is Instrument-mechanism — single-pillar — so a Concepts page would break the altitude rule Concepts exists to enforce. Prefer an Instrument page (or a Topics explainer that points into Instrument §6.2 / §12) for paradigm → wire mapping, engines (OPA/Cedar), and "optional citation style, not required product." Keep Intent/Capability/Trust in Concepts as the horizontal invariants paradigms consume.

---

## Stop restating structured decision fields in reasoning examples

**Where:** `docs/topics/ACS_in_action_example.md` (e.g. FIDES deny `reasoning`); also other paradigm examples there. Spec: `docs/spec/instrument/specification.md` §6.1

**Notes:** Example `reasoning` strings repeat what already lives in `reason_codes`, `policy_references`, `policy_data`, and `cited_provenance_ids` (provenance ids, paradigm check, policy id/version). §6.1 defines `reasoning` as a single *human-renderable* explanation and tells UIs/meta-policies to switch on `reason_codes` rather than parse prose. Examples should show short user-facing text (or omit fluff), not a second encoding of the structured payload — otherwise readers learn to dump structure into `reasoning`.

---

## Clarify what control paradigms mean for implementers

**Where:** `docs/topics/ACS_in_action_example.md` — "What different paradigms cite"; also Instrument §6.2 (and whatever Instrument/Topics page eventually defines paradigms — not Concepts)

**Notes:** The section presents IBAC, FIDES, CaMeL, and AARM as if they were ACS features a Guardian might be expected to ship. Unclear: must a Guardian implement all, some, or none? Are they negotiated on the wire? Are they normative ACS profiles or external research architectures ACS merely accommodates?

From the rest of the docs: the wire is paradigm-neutral (`policy_data` / `reason_codes` / `policy_references`); handshake negotiates ACS profiles (e.g. `acs-provenance` for IFC-style paradigms), not paradigm names; pure IBAC needs no Provenance profile. That "optional policy style, cite what fired" story should lead the paradigms section — with links to the source papers/specs and an honest note on how complete/implementable each mapping is — rather than four peer denial examples that read like a product matrix.

---

## Show conventional authz in ACS-in-action, not only research paradigms

**Where:** `docs/topics/ACS_in_action_example.md` — "What different paradigms cite" (IBAC, FIDES, CaMeL, AARM); contrast Instrument §12.1 (OPA/Rego as v0.1 reference, Cedar as v0.2)

**Notes:** The worked example leads with research control paradigms and never shows a conventional authorization policy (allow/deny on identity, role, resource, or a plain OPA/Cedar rule). Readers can infer that ACS is for FIDES/CaMeL/AARM-style systems only. Add at least one ordinary authz example (and preferably lead with it), since the deterministic layer's starting reference is OPA/Rego and paradigms are optional citation styles, not the product.

---

## Clarify whether Approver is optional and who resolves ASK

**Where:** `docs/concepts/agents.md` (Approver); Instrument §9 / §9.2; ASK as Core MUST-support disposition

**Notes:** Unclear whether an Approver is always required when ASK is used, or optional. Two models are mixed: (1) Concepts/§9 — the Guardian consults a third-party Approver (`ask_details.approver` endpoint) while the Observed Agent pauses; (2) §9.2 — some Observed Agents "route" ASK / need "approver UX", which implies ASK is returned to the main agent for local resolution (e.g. IDE user). Spell out whether the Guardian MAY return `ask` to the Observed Agent without a designated Approver, who presents the approval UI, and when Approver may be omitted entirely (e.g. never raise ASK; or substitute DEFER/DENY only).

---

## Rewrite the opaque "trust basis, not that it arrived" sentence

**Where:** `docs/concepts/agents.md` — Trust between agents: "How much weight a given fact carries depends on its trust basis, not on the fact that it arrived."

**Notes:** The last sentence is opaque without already knowing [Trust basis](docs/concepts/trust.md). Say plainly that arrival on the wire does not make a claim true — reliance follows how the fact was produced (asserted vs framework-attached vs cryptographically attested), and point at Trust basis. Do not rely on "weight" / "trust basis" jargon in the punchline.

---

## Clarify Guardian identity vs the policy (and policy-author)

**Where:** `docs/concepts/identity.md` — "Guardian identity: which policy authority is deciding."; also `docs/topics/core_concepts.md`

**Notes:** Wording makes Guardian identity sound like "the policy." Elsewhere ACS keeps three distinct: Observed Agent, Guardian, and **policy-author** (conformance: policy-author ≠ Guardian). Spell out that Guardian identity is the deciding *runtime/party* (its own principal), not the policy document and not the policy author — and say what (if anything) carries that identity on the wire in v0.1.

---

## Fix broken lead sentence on Identity overview

**Where:** `docs/identity/overview.md` — "There are 5 unique identity challenges that this standard aims to solve, where existing use cases that traditional identity models cannot support."

**Notes:** Editorial: the clause after the comma is ungrammatical (dangling "where … that …"). Rewrite into clear prose — e.g. five challenges that traditional identity models cannot support for agent use cases — and drop filler ("unique", "aims to solve") per STYLE.md.

---

## Drop the false "build-time vs execute-time supply chain" claim

**Where:** `docs/identity/overview.md` — "The supply chain you signed off on at build time is not the supply chain that executes."

**Notes:** Wishy-washy and misleading. Tools, runtime environment, and often prompts are stable; this is not primarily a supply-chain problem. The real point is non-deterministic *control flow* / action composition at runtime (which tool, order, arguments). Rewrite without the supply-chain metaphor.

---

## Do not imply agentic identity stops the "valid token, bad outcome" failure class

**Where:** `docs/identity/overview.md` — "Agent systems are introducing a new class of failures where every token is valid, every API call is authorized, and the outcome is still a security incident." (and the Unit 42 example that follows)

**Notes:** The failure class is real, but agentic identity does not solve it — it enables better governance and forensics (who acted, under which chain/intent). Prompt injection and similar attacks still succeed against valid credentials. Say that clearly so readers do not think naming/chaining identity closes the incident class.

Also, these two sentences do not follow: "The failure occurred because the runtime could not distinguish user intent from adversarial instructions introduced during execution. This illustrates the central identity challenge of agent systems." Failure to separate user intent from injected instructions is a runtime-control / intent-enforcement problem, not (by itself) an identity challenge. Either retarget the punchline to Instrument/Intent, or explain the actual identity-shaped gap the example is meant to show.

---

## Restructure the five identity-challenges table for readability

**Where:** `docs/identity/overview.md` — "The 5 Unique Runtime Identity Challenges for Agents" table

**Notes:** Editorial: cells are long multi-paragraph "poems" (threat narrative, standards gap, desired outcome, status crammed into one row). Hard to scan in MkDocs. Prefer one short row per challenge with a link into a dedicated subsection (or definition list / cards), keeping the comparison surface tight.

---

## Stop calling the identity challenges "unique"

**Where:** `docs/identity/overview.md` — "5 Unique Runtime Identity Challenges" (heading and body); at least Over-Privilege and Token Theft Resistance are long-standing IAM problems

**Notes:** Drop the uniqueness claim. The challenges are real; they are not unique to agents.

---

## Separate chain-integrity forensics from chain-based policy enforcement

**Where:** `docs/identity/overview.md` — Chain Integrity challenge (and related identity-standards prose)

**Notes:** Naming and verifying a delegation chain is clearly useful for forensics. It is much less clear that real-life policy enforcement can be driven by long, complicated chains. Human-authored policies cannot reasonably express rules over deep chains, and there is little evidence of other authz systems that make effective runtime use of them. Split the ask: audit/reconstructability vs enforcement, and do not treat "verify the full chain at every hop" as an obvious policy input without showing how Guardians would consume it.

---

## (Title of next comment)

**Where:**

**Notes:**
