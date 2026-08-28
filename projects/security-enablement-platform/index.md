---
layout: case-study
case_id: cs-01
short_title: "Security Platform (ASPM)"
title: "Security Enablement Platform"
standfirst: "A reusable ASPM / security-orchestration platform — 11 planes of security signal correlated into one findings workflow."
status: shipped
revision: "1.0"
reading_time: 9
permalink: /projects/security-enablement-platform/
description: >-
  A reusable ASPM / security-orchestration platform — architecture, real
  governance evidence, and results.
summary:
  problem: "Noisy, uncorrelated findings across 8+ scanner types, no consistent risk posture, security gates that get routed around."
  build: "An 11-plane ASPM platform: correlation layer, dynamic onboarding, dual-path SBOM, severity-threshold gating."
  outcome: "4 projects shifted from blocker/high/critical to predominantly low/medium severity, no release-SLA exceptions."
  scope: "Owned: architecture, correlation layer, onboarding workflow, gate policy design."
next_url: /projects/native-sast-reachability/
next_id: cs-02
next_title: "Native SAST & Reachability Engine"
---

*Private implementation; public architecture, ADRs, and real evidence available in the [POC repository](https://github.com/deepakguntoju/security-enablement-platform-poc).*

<h2 id="problem"><span class="kicker">01</span>Problem</h2>

Most security programs at scale hit the same wall from three directions at once:

- **Noise erodes trust.** Standard SCA flags every vulnerability in every present dependency, whether or not the application ever calls the vulnerable code path. Left unaddressed, teams start ignoring the findings queue.
- **No single answer to "what's our risk posture."** Findings arrive from 8+ independent scanner types — SAST, secrets, SCA, container scanning, cloud posture, identity exposure, IaC, DAST, mobile, runtime — each with its own severity model and its own re-scan behavior, which produces duplicate findings across runs with no consistent way to compare them.
- **Security-as-blocker undermines velocity.** A gate that fails a build over a low-priority finding gets routed around by engineering teams — and a security control that gets routed around is a worse outcome than a slightly looser gate that stays trusted and enabled.

<div class="callout">
<span class="kicker">Constraints</span>
<p>Keep scanning authority with specialist engines. Make one central layer authoritative for the finding <em>workflow</em> only — never the scanning itself. No single scanner or pipeline tool may become a long-term data store for findings.</p>
</div>

<h2 id="architecture"><span class="kicker">02</span>Architecture</h2>

**11 planes**, each a distinct engine or capability, all correlating into one hub:

1. CI/CD security gate — SAST, secret scanning, SCA, container scanning, SBOM generation, policy evaluation, signing, evidence publication
2. Software composition service — component inventory, continuous vuln/license analysis, scheduled reconciliation
3. **Normalized finding model + correlation layer** — consolidates overlapping findings from multiple scanners into one canonical finding while preserving per-scanner provenance and raw evidence
4. Cloud posture + identity exposure (CSPM/CIEM)
5. Infrastructure-as-code scanning
6. Dynamic application security testing (DAST)
7. Mobile security testing
8. Runtime/workload behavioral detection
9. Central ASPM / findings hub — ingestion, dedup, correlation, severity model, ownership, SLA tracking, ticketing, reporting
10. **Dynamic application onboarding** — repository discovery → project configuration → pipeline provisioning → branch selection → security-configuration generation → pipeline execution → evidence ingestion, as one repeatable workflow rather than a manual integration per application
11. Multi-application security posture view — CI/CD evidence, findings, supply-chain intelligence, remediation state, and coverage across every application in one cross-portfolio view

Every architectural choice behind these planes is documented as a formal ADR — the alternatives considered and why they were rejected, not just the decision made. Seven ADRs are public.

<h2 id="implementation"><span class="kicker">03</span>Implementation</h2>

Three pieces of this platform are worth calling out specifically, because they're where the real engineering happened, not just the integration:

**The correlation layer.** Deduplication that just hides duplicate text isn't correlation — it loses the ability to trace a finding back to what actually flagged it. The correlation layer keys on a stable fingerprint (target + rule/check + location), not raw finding text, so repeated scans of unchanged code don't create duplicate open findings, while every canonical finding still preserves which scanner(s) raised it, their raw evidence, and severity/CVSS context.

**Dynamic application onboarding.** Adding a new application to the platform is a first-class, repeatable workflow — discover the repo, confirm configuration, stage it, promote it — not a bespoke integration project every time. This surfaced a real production defect: branch propagation didn't behave consistently across re-onboarding, new branches triggered unwanted multibranch pipeline fan-out, and project configuration didn't reliably survive a container/runtime restart. The root cause was treating the pipeline orchestrator's own containerized runtime state as if it were durable configuration storage. The fix was an architectural one: redesign persistence around a dedicated project-configuration store owned by the platform itself, with the pipeline orchestrator treated purely as disposable execution. This is documented as ADR 0006 — a real bug-driven redesign, not a greenfield decision.

**Dual-path SBOM upload.** A single upload path creates a specific failure mode depending on which path is chosen — pipeline-push-only means silent failures go unnoticed; scheduled-pull-only means a visibility delay between when a vulnerable component ships and when it's reflected in the inventory. The platform uses both: pipeline push for immediacy, independent scheduled reconciliation for resilience. Documented as ADR 0003.

<div class="scope-block">
  <div class="scope-row"><span class="scope-label">OWNED</span><span class="scope-val">Platform architecture, correlation layer, onboarding workflow, gate policy design, ADRs</span></div>
  <div class="scope-row"><span class="scope-label">INFLUENCED</span><span class="scope-val">SBOM/supply-chain tooling selection (Dependency-Track, DefectDojo), CI/CD pipeline conventions</span></div>
  <div class="scope-row"><span class="scope-label">ADVISED</span><span class="scope-val">Cloud posture and runtime detection rollout across other teams' pipelines</span></div>
</div>

<h2 id="security-workflow"><span class="kicker">04</span>Security Workflow</h2>

How a finding actually moves through the platform, end to end:

<div class="table-wrap">
<table>
<thead><tr><th>Control</th><th>Where it acts</th><th>What happens on failure</th></tr></thead>
<tbody>
<tr><td>Severity-threshold gate</td><td>Build time, before merge/deploy</td><td>Build blocked only above the defined severity/exploitability bar; everything else routes to the findings hub for tracked, SLA-bound remediation — never silently dropped</td></tr>
<tr><td>Correlation &amp; dedup</td><td>Ingestion, before a finding is stored</td><td>Duplicate scans of unchanged code produce zero new open findings; canonical finding keeps every scanner's raw evidence</td></tr>
<tr><td>Governance override</td><td>Post-gate, human-initiated</td><td>An authorized override is recorded as an auditable event (who, when, why) — not a silent bypass</td></tr>
<tr><td>OWASP ASVS mapping</td><td>Per build, continuous</td><td>Every build's control coverage is scored against a real compliance standard, not a custom checklist</td></tr>
</tbody>
</table>
</div>

<h2 id="evidence"><span class="kicker">05</span>Evidence</h2>

This is the platform actually operating, not a design document — real screenshots from a running instance, redacted only of internal project names and internal product branding.

<figure class="evidence">
  <img src="https://raw.githubusercontent.com/deepakguntoju/security-enablement-platform-poc/main/docs/screenshots/findings-hub-and-compliance/01-dashboard-security-posture.png" alt="Findings hub dashboard showing severity distribution and gate decisions" loading="lazy">
  <figcaption>E-1 — Findings hub dashboard: real portfolio data, 44 critical / 344 high / 377 medium / 163 low findings across 5 projects / 10 services (sanitized recreation of internal project names only).</figcaption>
</figure>

<figure class="evidence">
  <img src="https://raw.githubusercontent.com/deepakguntoju/security-enablement-platform-poc/main/docs/screenshots/findings-hub-and-compliance/03-build-detail-pipeline-gate.png" alt="Build detail page showing a blocked security gate decision and risk enrichment" loading="lazy">
  <figcaption>E-2 — Build detail: full pipeline stage breakdown, a real BLOCKED gate decision (SECRETS: BLOCK vs. VULNERABILITY: WARN), EPSS/KEV risk enrichment, SBOM sync status.</figcaption>
</figure>

<figure class="evidence">
  <img src="https://raw.githubusercontent.com/deepakguntoju/security-enablement-platform-poc/main/docs/screenshots/findings-hub-and-compliance/05-governance-audit-trail.png" alt="Governance audit trail showing real security gate override events" loading="lazy">
  <figcaption>E-3 — Governance audit trail: real <code>SECURITY_GATE_OVERRIDE_SET</code> / <code>SECURITY_GATE_OVERRIDE_CLEARED</code> events with timestamps and identities — the risk-acceptance workflow actually in use.</figcaption>
</figure>

<figure class="evidence">
  <img src="https://raw.githubusercontent.com/deepakguntoju/security-enablement-platform-poc/main/docs/screenshots/findings-hub-and-compliance/04-compliance-owasp-asvs.png" alt="OWASP ASVS compliance mapping showing 127 controls and their status" loading="lazy">
  <figcaption>E-4 — OWASP ASVS v4.0.1 compliance mapping for a real build: 127 controls, evidenced/passed/failed/at-risk/not-assessed breakdown.</figcaption>
</figure>

<div class="callout">
<span class="kicker">Evidence &amp; redaction note</span>
<p>Screenshots are unedited captures of a running instance except for black-box redaction of internal project names and one internal product wordmark — no data was fabricated or recreated. A real, unedited pipeline gate report against the public Flask repository (secret scan, SAST, SCA, SBOM, a genuine policy-override decision) is also available: <a href="https://github.com/deepakguntoju/security-enablement-platform-poc/blob/main/docs/reports/security_report_flask-app_build8.html">view the report ↗</a>.</p>
</div>

<h2 id="results"><span class="kicker">06</span>Results</h2>

<div class="metric-strip">
  <div class="metric"><span class="value">4</span><span class="label">projects shifted to low/medium severity within &lt;1 year</span></div>
  <div class="metric"><span class="value">0</span><span class="label">release-SLA exceptions required</span></div>
  <div class="metric"><span class="value">11</span><span class="label">correlated planes in one findings workflow</span></div>
  <div class="metric"><span class="value">127</span><span class="label">OWASP ASVS controls mapped per build</span></div>
</div>

> "Across four supported projects, the mix of open findings shifted materially away from blocker/high/critical severities toward low/medium within under a year of onboarding each project — achieved without requiring release-SLA exceptions or blocking scheduled releases."

Achieved by gating releases only on findings that meet a defined severity/exploitability bar (ADR 0005), not on every finding — pairing that discipline with the reachability engine (see [CS-02](/projects/native-sast-reachability/)) to keep the gated set genuinely high-signal.

CI/CD security maturity: multiple pipelines at or approaching an advanced DevSecOps Security Maturity Model (DSOMM) level, with maturity work continuing across the rest of the portfolio — stated deliberately as "approaching," not "implemented L5," because that's what's actually true.

<h2 id="lessons"><span class="kicker">07</span>Lessons Learned</h2>

- The project-configuration-store redesign (ADR 0006) is the most instructive piece of this whole platform: don't let "where a build runs" and "where configuration is durably stored" become the same system, because when they are, restarting the wrong container silently erases configuration nobody meant to lose.
- Centralizing the *workflow* while deliberately not centralizing the *scanning* is what lets each specialist engine keep doing what it's good at, while still producing one coherent answer to "who owns this, how bad is it, and is it fixed."
- A severity-threshold gate only works if the underlying findings are trustworthy — this is the direct link to the reachability engine in the companion case study: without it, the gate would either be too strict (blocking on noise) or too loose (missing signal in the noise).
- Status, honestly: implemented — CI/CD gate, findings hub, cloud posture, DAST, mobile, runtime, dynamic onboarding. In progress — reachability language expansion, broader DSOMM maturity. Future — threat-intel enrichment, SIEM/XDR correlation, AI-assisted triage.

<div class="callout">
<span class="kicker">Further reading</span>
<p><a href="https://github.com/deepakguntoju/security-enablement-platform-poc/blob/main/docs/ARCHITECTURE.pdf">Full architecture PDF ↗</a> · <a href="https://github.com/deepakguntoju/security-enablement-platform-poc/tree/main/docs/adr">All 7 ADRs ↗</a> · <a href="https://github.com/deepakguntoju/security-enablement-platform-poc/blob/main/docs/METRICS.md">Metrics &amp; evidence index ↗</a></p>
</div>
