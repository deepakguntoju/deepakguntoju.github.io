---
layout: page
title: About
permalink: /about/
description: About Deepak Guntoju — background in AppSec, DevSecOps, and cloud security.
---

<div class="docmeta">ABOUT</div>

# About

I'm a security engineer with 5+ years of experience spanning application security, DevSecOps, and cloud security — currently a Senior Cloud Security Engineer at the AI-enabled product division of a global enterprise technology & digital-transformation services company, based in Hyderabad, India.

My path started in hands-on AppSec: four years at Cognizant running SAST (Checkmarx) and DAST (OWASP ZAP) programs, secure code review, IAM deployment (Okta, Delinea), and SIEM operations (Splunk) across large-scale enterprise applications. That foundation — where vulnerabilities actually come from, how remediation really gets tracked, what makes a finding worth a developer's time — is the part of this work that doesn't show up in an architecture diagram but shapes every decision above it.

The last year has been a shift from *running* security tooling to *building the systems that run it*. I designed and built a reusable security-orchestration (ASPM) platform that consolidates CI/CD security, SCA, secrets detection, container security, and supply-chain intelligence into one correlated findings workflow — architected as a repeatable platform capability, not a project-specific pipeline. Inside that platform, I built a native static reachability analysis engine because standard SCA tooling's biggest practical failure is noise: flagging a vulnerability in a dependency that's present but never actually called.

That platform work is private (I don't publish employer code, architecture secrets, or customer data), but the architecture, the design decisions, and a set of real evidence — a governance audit trail with actual gate-override events, an OWASP ASVS compliance mapping, a real unedited pipeline report — are public in a sanitized proof-of-concept repository. See the [case studies](/projects/) for the detail, or the [repo itself](https://github.com/deepakguntoju/security-enablement-platform-poc) for the primary source.

## How I think about this work

Two ideas run through everything I build:

**Security should be an enabler, not a blocker.** A gate that fails a build over a low-priority finding gets routed around, and a security program that gets routed around has already lost. The platform I built gates only on a defined severity/exploitability threshold — everything else is tracked, SLA-bound, and visible, not silently dropped or blindly blocking a release.

**Repeatable systems beat one-off engineering.** The difference between "I ran a scan" and "I built the thing that makes scanning trustworthy at scale" is the difference this whole body of work is organized around — normalized findings instead of five inconsistent severity models, a reachability engine instead of a growing pile of false positives, a dynamic onboarding workflow instead of a new manual integration for every application.

## Capability areas

<div class="table-wrap">
<table>
<thead><tr><th>Area</th><th>Depth</th><th>Evidence</th></tr></thead>
<tbody>
<tr><td>Application Security &amp; Secure SDLC</td><td>Deep</td><td><a href="/resume/">4 years hands-on, Cognizant</a></td></tr>
<tr><td>DevSecOps / CI/CD Security</td><td>Deep</td><td><a href="/projects/security-enablement-platform/">Security Enablement Platform</a></td></tr>
<tr><td>Security Platform Engineering / ASPM</td><td>Deep</td><td><a href="/projects/security-enablement-platform/">Security Enablement Platform</a></td></tr>
<tr><td>Static Analysis / Reachability Research</td><td>Deep</td><td><a href="/projects/native-sast-reachability/">Native SAST &amp; Reachability Engine</a></td></tr>
<tr><td>Cloud Security (AWS, GCP)</td><td>Core</td><td><a href="/resume/">Resume</a></td></tr>
</tbody>
</table>
</div>

## Elsewhere

- [Resume](/resume/) (PDF)
- [GitHub](https://github.com/deepakguntoju)
- [LinkedIn](https://linkedin.com/in/deepak-guntoju-6694421b2)
- [Public POC repository](https://github.com/deepakguntoju/security-enablement-platform-poc)
