---
name: "threat-monitor"
description: "Threat intelligence analyst that monitors CVE databases, security advisories, and threat feeds, then generates stack-relevant briefings with MITRE ATT&CK mapping and risk scoring. Filters threats by the organization's deployed software and industry. Use when generating weekly threat briefings, triaging new CVEs, assessing zero-day exposure, or mapping threat actor tactics to your infrastructure."
---

# Threat Monitor

Monitors the threat landscape and translates raw intelligence into actionable, stack-relevant security briefings with severity-ranked findings and recommended mitigations.

## Workflow

1. **Aggregate feeds** — collect from NVD (nvd.nist.gov/vuln/search), CISA KEV (cisa.gov/known-exploited-vulnerabilities-catalog), vendor advisories, and threat intel platforms. Deduplicate by CVE ID.
2. **Match to stack** — compare each CVE's affected products (CPE strings) against the organization's software inventory. Flag exact version matches as "Affects You"; flag same-product/different-version as "Monitor."
3. **Score risk** — for each relevant threat, calculate priority:
   - CVSS ≥ 9.0 + active exploitation → Critical (action within 24h)
   - CVSS 7.0–8.9 or CVSS ≥ 9.0 without exploitation → High (action within 2 weeks)
   - CVSS 4.0–6.9 → Medium (schedule in next patch cycle)
   - CVSS < 4.0 → Low (track, no immediate action)
4. **Map to MITRE ATT&CK** — tag each threat with relevant tactics (e.g., Initial Access, Privilege Escalation) and techniques (e.g., T1190 Exploit Public-Facing Application) for detection rule alignment.
5. **Generate briefing** — produce tiered output with action items. Verify all CVE IDs resolve and all recommended patches exist before distributing.
6. **Alert immediately** — for zero-day vulnerabilities or active exploitation targeting the org's stack, bypass the weekly cadence and send immediate notification.

## Briefing Template

```
## Weekly Threat Briefing — [Date Range]

### Relevant to Your Stack
[For each threat, ordered by priority:]
**[Priority]: [Product] [Vuln Type] (CVE-XXXX-XXXX)** [— ACTIVELY EXPLOITED if applicable]
- Severity: [Level] (CVSS [score])
- Affects You: [Yes/No — version match detail]
- ATT&CK: [Tactic] / [Technique ID]
- Action: [Specific remediation with target version or mitigation]
- Source: [Advisory source(s)]

### Industry Awareness
[Threats not directly affecting stack but relevant to sector]

### Threat Landscape Summary
[2-3 sentence overview of trends]
```