# Investigating a CEO Account Takeover

> **Simulated client engagement - MyFirstHack training project.** Cloudora, its users, and all indicators in this repository are fictional. This repository documents investigation practice, not employment or a real incident.

## Scenario

Cloudora's IT team raised an impossible-travel alert after CEO Daniel Reeve appeared to sign in from Lagos at 03:12 UTC while normally based in London. I investigated the alert with KQL over Entra ID-style sign-in and audit logs, reconstructed the intrusion, checked its scope, and documented a containment plan.

## Investigation outcome

| Area | Result |
| --- | --- |
| Initial access | A low-and-slow password spray from three `102.89.x.x` addresses preceded the successful sign-ins. |
| Confirmed compromise | `daniel.reeve@cloudora.io` and `priya.nair@cloudora.io` were accessed from attacker infrastructure. |
| Persistence | The CEO account received a rogue MFA registration (`Pixel 6`) and a mail-hiding rule (`RSS Subscriptions`). |
| False positive | Omar Farah's Dubai activity matched legitimate travel, although he was still a spray target. |
| Scope | 24 additional accounts received failed spray attempts only and require precautionary resets. |

## What I investigated

1. Validated the CEO's incident-day sign-ins against the account's normal London baseline.
2. Compared the Lagos activity with Omar Farah's explainable Dubai travel.
3. Grouped failed sign-ins by source IP to identify password-spray behaviour.
4. Reviewed audit activity for persistence and email-rule tampering.
5. Scoped successful sign-ins from attacker IPs and investigated the second victim.
6. Produced a reset list and a six-hour detection query that would surface the spray before compromise.

## Evidence walkthrough

| Stage | Evidence |
| --- | --- |
| CEO incident timeline | [Screenshot 1](<Screenshot%20Result/1%20Triage%20the%20Alert.png>) |
| CEO geographic baseline | [Screenshot 2](<Screenshot%20Result/2%20Baseline%20the%20account.png>) |
| Legitimate-travel comparison | [Screenshot 3](<Screenshot%20Result/3%20Baseline%20the%20account%20for%20omar.png>) |
| Spray attribution | [Screenshot 4](<Screenshot%20Result/4%20Hunt%20for%20credential%20attacks.png>) |
| Three-night attack window | [Screenshot 5](<Screenshot%20Result/5%20Confirm%20attack%20window.png>) |
| CEO persistence activity | [Screenshot 6](<Screenshot%20Result/6%20Check%20For%20Persistence.png>) |
| Successful attacker sign-ins | [Screenshot 7](<Screenshot%20Result/7%20Check%20Anyone%20else%20compromised.png>) |
| Priya incident timeline | [Screenshot 8](<Screenshot%20Result/8%20Check%20Timeline%20of%20other%20account.png>) |
| Priya baseline | [Screenshot 9](<Screenshot%20Result/9%20Baseline%20of%20other%20account.png>) |
| Priya audit review | [Screenshot 10](<Screenshot%20Result/10%20Check%20for%20persistence%20for%20other%20account.png>) |
| Near-miss reset list | [Screenshots 11](<Screenshot%20Result/11%20Check%20which%20other%20account%20was%20sprayed%20.png>) and [12](<Screenshot%20Result/12%20Check%20which%20other%20account%20was%20sprayed%20cont.png>) |
| Night-one detection rule | [Screenshot 13](<Screenshot%20Result/13%20The%20rule%20to%20catch%20spray%20on%20the%20first%20day%20of%20attack.png>) |

## Query pack

The commented KQL is in [Queries Used](<Queries%20Used>). Load the synthetic data as `CloudoraSignIn_CL` and `CloudoraAudit_CL`. `ResultType` must be ingested as a **string**; queries intentionally compare it with values such as `"50126"` and `"0"`.

| Query | Purpose |
| --- | --- |
| [01](<Queries%20Used/01-triage-ceo-signins.kql>) | CEO sign-ins on the incident day |
| [02](<Queries%20Used/02-baseline-ceo.kql>) / [03](<Queries%20Used/03-baseline-omar.kql>) | Baseline comparison |
| [04](<Queries%20Used/04-password-spray-by-ip.kql>) / [05](<Queries%20Used/05-password-spray-window.kql>) | Password-spray attribution and timing |
| [06](<Queries%20Used/06-ceo-persistence-audit.kql>) / [07](<Queries%20Used/07-scope-attacker-successes.kql>) | Persistence and scoping |
| [08](<Queries%20Used/08-priya-timeline.kql>) - [10](<Queries%20Used/10-priya-audit-review.kql>) | Second-victim investigation |
| [11](<Queries%20Used/11-near-miss-reset-list.kql>) / [12](<Queries%20Used/12-password-spray-detection.kql>) | Reset inventory and detection engineering |

## ATT&CK techniques observed

| Tactic | Technique | Evidence |
| --- | --- | --- |
| Credential Access | T1110.003 - Password Spraying | Distributed failed credential attempts across 26 accounts |
| Initial Access | T1078 - Valid Accounts | Successful sign-ins on Daniel's and Priya's accounts |
| Persistence | T1098.005 - Device Registration | Rogue `Pixel 6` MFA registration |
| Defense Evasion | T1564.008 - Email Hiding Rules | `RSS Subscriptions` diverted finance and invoice mail |

## Deliverables

- [Full incident report](<Report/incident-report.md>)
- [KQL query pack](<Queries%20Used>)
- [Screenshot evidence](<Screenshot%20Result>)

## Responsible use

The scenario, synthetic datasets, and assignment structure come from the MyFirstHack community resource. The KQL exercises were completed for training; original investigation notes, file organization, and portfolio write-up are provided here. No source datasets or answer-key material are redistributed.
