# CEO Account Takeover Portfolio Design

## Purpose

Turn the completed Cloudora investigation screenshots and findings into an original, portfolio-ready GitHub repository. The work is explicitly labelled as a simulated MyFirstHack training engagement using fictional data.

## Decisions

- Use the training-scenario framing throughout. Containment entries are documented response decisions, not claims of live-tenant administration.
- Keep the existing screenshot assets unchanged and link to them from the README as the evidence trail.
- Write original explanatory prose and organize the repository around the investigation workflow, rather than reproducing the reference repository's text or layout.
- Store one commented KQL file for each distinct investigation task. The two screenshots showing the near-miss output share one query file.
- Render the supplied incident-report content as Markdown with real tables for metadata, timeline, indicators, MITRE mapping, and response actions.

## Target Structure

```text
README.md
Queries Used/
  01-triage-ceo-signins.kql
  02-baseline-ceo.kql
  03-baseline-omar.kql
  04-password-spray-by-ip.kql
  05-password-spray-window.kql
  06-ceo-persistence-audit.kql
  07-scope-attacker-successes.kql
  08-priya-timeline.kql
  09-priya-baseline.kql
  10-priya-audit-review.kql
  11-near-miss-reset-list.kql
  12-password-spray-detection.kql
Report/
  incident-report.md
Screenshot Result/
```

## Content Design

The README will lead with the scenario, findings, and analyst workflow, then link to the query pack, report, and screenshots. The report will follow the provided template's nine sections. It will distinguish observed evidence from limitations of the supplied logs and will not state that money was not lost; it will state that the available logs contain no observed fraud or payment activity.

## Validation

Review the Markdown structure, confirm every screenshot is represented in the README evidence index, check KQL table names and result-type strings, and inspect the Git diff before pushing.
