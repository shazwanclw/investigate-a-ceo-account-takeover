# CEO Account Takeover Portfolio Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Publish an original Markdown portfolio package for the Cloudora simulated CEO account-takeover investigation.

**Architecture:** Keep the existing screenshots as immutable evidence. Add a root README that connects the scenario, evidence, query files, and report; add a commented KQL query pack grouped by investigation stage; and turn the supplied report text into a template-aligned Markdown report.

**Tech Stack:** GitHub Markdown, KQL for Azure Data Explorer / Microsoft Sentinel.

---

### Task 1: Create the README

**Files:**
- Create: `README.md`

**Step 1:** State the scenario as a MyFirstHack simulated engagement and identify fictional data.

**Step 2:** Add original sections for key findings, workflow, evidence index, query index, MITRE mapping, and responsible-use attribution.

**Step 3:** Link each of the 13 existing screenshot assets to the relevant investigation stage.

**Step 4:** Check every relative link resolves to an existing repository path.

### Task 2: Create the KQL evidence pack

**Files:**
- Create: `Queries Used/01-triage-ceo-signins.kql`
- Create: `Queries Used/02-baseline-ceo.kql`
- Create: `Queries Used/03-baseline-omar.kql`
- Create: `Queries Used/04-password-spray-by-ip.kql`
- Create: `Queries Used/05-password-spray-window.kql`
- Create: `Queries Used/06-ceo-persistence-audit.kql`
- Create: `Queries Used/07-scope-attacker-successes.kql`
- Create: `Queries Used/08-priya-timeline.kql`
- Create: `Queries Used/09-priya-baseline.kql`
- Create: `Queries Used/10-priya-audit-review.kql`
- Create: `Queries Used/11-near-miss-reset-list.kql`
- Create: `Queries Used/12-password-spray-detection.kql`

**Step 1:** Add a purpose, required table, and result-type note to each relevant file.

**Step 2:** Use supplied course queries for instructor-led stages, retaining concise comments only.

**Step 3:** Include original task solutions for the Priya investigation, near-miss reset list, and bounded six-hour spray detector.

**Step 4:** Verify every query uses `CloudoraSignIn_CL` or `CloudoraAudit_CL` and compares `ResultType` as a string.

### Task 3: Format the incident report

**Files:**
- Create: `Report/incident-report.md`

**Step 1:** Rebuild the nine supplied template sections using Markdown headings and tables.

**Step 2:** Preserve supported findings, but qualify response activities as documented training-scenario actions and limit claims to available log evidence.

**Step 3:** Include explicit scope evidence, all 24 near-miss accounts, MITRE mappings, prioritised recommendations, and lessons learned.

**Step 4:** Check every table has a header and delimiter row, and check the findings/timeline align with the stated timestamps.

### Task 4: Validate and publish

**Files:**
- Verify: `README.md`, `Queries Used/*.kql`, `Report/incident-report.md`

**Step 1:** Inspect `git diff --check` for whitespace errors.

**Step 2:** Programmatically confirm Markdown links reference real local paths and all 13 screenshots are mentioned in the README.

**Step 3:** Review Git status and commit the finished artifacts.

**Step 4:** Push `main` to `origin` and verify the remote branch commit.
