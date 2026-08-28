---
layout: case-study
case_id: cs-02
short_title: "Native SAST & Reachability"
title: "Native SAST & Reachability Engine"
standfirst: "A static reachability analysis engine that answers the one question standard SCA can't: is this flagged vulnerability actually reachable?"
status: shipped
revision: "1.0"
reading_time: 8
permalink: /projects/native-sast-reachability/
description: >-
  An in-house multi-language static reachability analysis engine —
  architecture, benchmark results, and a real CVE demonstration.
summary:
  problem: "SCA flags every vulnerability in every present dependency, whether or not the code path is ever actually called."
  build: "A multi-language (Java/Python/Node.js/TypeScript) reachability engine, benchmark-validated against real CVEs."
  outcome: "100% precision/recall across a 62-fixture regression corpus; zero regression across languages."
  scope: "Owned: architecture, all 4 language integrations, benchmark methodology."
prev_url: /projects/security-enablement-platform/
prev_id: cs-01
prev_title: "Security Enablement Platform"
---

*A component of the Security Enablement Platform; the differentiator most off-the-shelf ASPM tooling in this price/maturity bracket doesn't include. Public benchmark evidence and real CVE demonstrations available in the [POC repository](https://github.com/deepakguntoju/security-enablement-platform-poc).*

<h2 id="problem"><span class="kicker">01</span>Problem</h2>

Standard software composition analysis flags a vulnerability in any dependency present in a build, regardless of whether the vulnerable code path is ever actually reachable from the application's own code. At meaningful scale this produces a large volume of findings that are technically accurate — the vulnerable component is present — but practically low-priority, because the vulnerable function is never called.

This isn't a hypothetical problem. It's the specific failure mode that erodes trust in a findings queue: a team burns triage time on a CVE that can never actually execute, gets burned enough times, and starts deprioritizing the whole SCA feed — including the findings that were genuinely exploitable.

<h2 id="architecture"><span class="kicker">02</span>Architecture</h2>

Every supported language implements the same five-piece contract, dispatched through one language registry:

1. **Parser** — produces a normalized file/method model (imports, classes, method bodies, invocations, local variables, field annotations) from the language's real AST, not a regex approximation
2. **Entry-point discovery** — framework-specific heuristics first (Spring annotations, Flask/FastAPI routes, NestJS decorators, Express route registration), with a generic fallback
3. **Dependency resolver** — parses the language's manifest format (`pom.xml`, `requirements.txt`, `package-lock.json`) to resolve installed package versions
4. **Call-graph builder** — same-file, cross-file, and instance-method resolution; parameter/return/alias propagation; sanitizer detection; dependency-node synthesis for unresolved third-party calls
5. **Analyzer** — orchestrates the above, assembles the evidence model, and classifies the result: `REACHABLE`, `REACHABLE_SANITIZED`, `DIRECTLY_REFERENCED`, `DEPENDENCY_PRESENT_ONLY`, `NO_PATH_FOUND`, `INCONCLUSIVE`

A generic core — bounded BFS path search, confidence scoring, dependency-injection-aware edge resolution — is shared across every language, so adding language N+1 means writing the five language-specific pieces, not rebuilding the reasoning engine underneath them.

<h2 id="implementation"><span class="kicker">03</span>Implementation</h2>

**Multi-language, not single-language.** Java, Python, Node.js/JavaScript (Express), and TypeScript (NestJS backend with constructor-injection-aware call resolution) are all supported through the same contract. The NestJS support specifically required a dependency-injection resolution model — recognizing `private readonly userService: UserService` in a constructor and resolving `this.userService.findOne()` as a DI-mediated call — the same architectural shape Java's `@Autowired` field-annotation detection already handled, ported forward for TypeScript's constructor-injection pattern.

**Benchmark-driven, not rule-count-driven.** The engine wasn't declared "done" for a language until it had its own evidence: true/false positive and negative rates, cross-file and instance-method resolution counts, parameter/return propagation counts, sanitizer detection, cycle detection, and — critically — real-world validation against actual CVEs, not just synthetic fixtures. The full regression corpus is 62 fixtures (30 Java + 32 TypeScript), and the engine holds **100% precision and 100% recall** across it.

**A hybrid AI-assisted SAST architecture is designed** (implementation in progress, not shipped) that keeps this same discipline: deterministic analysis remains the system of record for whether a finding exists and where; local AI is used strictly as a contextual-reasoning layer on top of candidates the deterministic engine has already identified and constrained — never as the sole detector, never with unvalidated output entering the correlation pipeline.

<div class="scope-block">
  <div class="scope-row"><span class="scope-label">OWNED</span><span class="scope-val">Engine architecture, five-piece language contract, all 4 language integrations, benchmark methodology</span></div>
  <div class="scope-row"><span class="scope-label">INFLUENCED</span><span class="scope-val">Hybrid AI-assisted SAST architecture (design complete, build in progress)</span></div>
</div>

<h2 id="security-workflow"><span class="kicker">04</span>Security Workflow</h2>

<div class="table-wrap">
<table>
<thead><tr><th>Stage</th><th>What happens</th></tr></thead>
<tbody>
<tr><td>Trivy SCA scan</td><td>Flags the dependency and its known CVE — identical output regardless of reachability</td></tr>
<tr><td>Reachability analysis</td><td>Parses application code, builds the call graph, determines whether the vulnerable symbol is actually invocable from an entry point</td></tr>
<tr><td>Verdict</td><td><code>REACHABLE</code> (full call chain attached) or <code>DEPENDENCY_PRESENT_ONLY</code> (demoted, reasoning kept visible — never silently dropped)</td></tr>
<tr><td>Gate integration</td><td>Feeds directly into the Security Enablement Platform's severity-threshold gate (see <a href="/projects/security-enablement-platform/#security-workflow">CS-01</a>)</td></tr>
</tbody>
</table>
</div>

<h2 id="evidence"><span class="kicker">05</span>Evidence</h2>

The clearest demonstration of what this engine is for is a single before/after pair, run through the actual portal — not a mockup: **CVE-2022-42889 (Text4Shell)**, the same critical vulnerability, in two structurally identical fixture applications.

<figure class="evidence">
  <img src="https://raw.githubusercontent.com/deepakguntoju/security-enablement-platform-poc/main/docs/screenshots/reachability/00-trivy-component-scan.png" alt="Trivy's raw component/CVE scan, showing no reachability distinction" loading="lazy">
  <figcaption>E-1 — Trivy's raw scan: both fixtures produce the identical finding, same CVE, same severity. This is the noise the engine exists to cut through.</figcaption>
</figure>

<figure class="evidence">
  <img src="https://raw.githubusercontent.com/deepakguntoju/security-enablement-platform-poc/main/docs/screenshots/reachability/01-reachable-finding-full-call-chain.png" alt="Text4Shell CVE shown as REACHABLE with a full call chain" loading="lazy">
  <figcaption>E-2 — Fixture A: application code actually calls <code>StringSubstitutor.replace()</code>. Verdict: <code>REACHABLE</code>, confidence 1.0, full call chain <code>WebController.handleRequest → TemplateService.render → TemplateEngine.evaluate → StringSubstitutor.replace</code>.</figcaption>
</figure>

<figure class="evidence">
  <img src="https://raw.githubusercontent.com/deepakguntoju/security-enablement-platform-poc/main/docs/screenshots/reachability/02-not-reachable-finding-demoted.png" alt="Text4Shell CVE on a safe sibling app, demoted to DEPENDENCY_PRESENT_ONLY" loading="lazy">
  <figcaption>E-3 — Fixture B: same CVE, same dependency version, but application code calls a safe alternative (<code>WordUtils.capitalize</code>) instead. Verdict: <code>DEPENDENCY_PRESENT_ONLY</code>, confidence 0.85 — demoted, not dropped.</figcaption>
</figure>

<div class="callout">
<span class="kicker">Evidence &amp; method note</span>
<p>A third real example is demonstrated end-to-end: CVE-2020-14343 (PyYAML), distinguishing <code>yaml.safe_load()</code> (safe) from <code>yaml.load()</code> (vulnerable) usage. All three were run through the actual scan-to-verdict pipeline against real, if minimal, fixture applications — not synthetic unit tests in isolation.</p>
</div>

<h2 id="results"><span class="kicker">06</span>Results</h2>

<div class="metric-strip">
  <div class="metric"><span class="value">100%</span><span class="label">precision, 62-fixture regression corpus</span></div>
  <div class="metric"><span class="value">100%</span><span class="label">recall, same corpus</span></div>
  <div class="metric"><span class="value">4</span><span class="label">languages: Java, Python, Node.js, TypeScript</span></div>
  <div class="metric"><span class="value">0</span><span class="label">regressions across languages when adding a new one</span></div>
</div>

- **100% precision / 100% recall** across a 62-fixture regression corpus (30 Java + 32 TypeScript), independently reproducible via the repo's own test runner
- **Zero regression** confirmed across every language each time a new one was added — Java, Python, and Node.js results verified identical after the TypeScript integration landed
- **Real CVE validation**, not just synthetic fixtures — Text4Shell and PyYAML demonstrated end-to-end through the actual scan-to-verdict pipeline, both positive and negative cases, both via direct API and through the live portal UI

<h2 id="lessons"><span class="kicker">07</span>Lessons Learned</h2>

- The benchmark methodology is the actual lesson here, more than any individual result: a reachability engine's credibility rests entirely on precision, because a single false "reachable" verdict on a benign dependency destroys trust faster than the noise problem it was built to solve.
- Every language's support was held to the same bar before being called done — true/false positive and negative rates, path explainability, real CVE validation — the same standard the rest of the platform's claims are held to.
- The DI-aware NestJS work is a smaller but concrete lesson in reuse: recognizing that constructor injection and Java's field-annotation injection are the *same underlying problem* meant porting an existing, proven design forward instead of re-deriving it from scratch.
- Status, honestly: this is genuinely a component "in progress," not finished — language coverage is expanding, and the hybrid AI-assisted layer has architecture but not yet its own benchmark evidence.

<div class="callout">
<span class="kicker">Further reading</span>
<p><a href="https://github.com/deepakguntoju/security-enablement-platform-poc/blob/main/reachability-engine/README.md">Reachability engine README ↗</a> · <a href="https://github.com/deepakguntoju/security-enablement-platform-poc/tree/main/docs/screenshots/reachability">Live screenshot evidence ↗</a> · <a href="https://github.com/deepakguntoju/security-enablement-platform-poc/blob/main/docs/adr/0004-reachability-analysis-to-cut-false-positives.md">ADR 0004 — why build this at all ↗</a></p>
</div>
