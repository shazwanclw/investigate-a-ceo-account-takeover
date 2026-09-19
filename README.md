# CEO Account Takeover Investigation

## Case overview

Cloudora's IT administrator opened a P1 ticket after CEO Daniel Reeve appeared to sign in from Lagos at 03:12 UTC while normally working from London. The account was being used days before an important client deal, so the investigation needed to establish whether the alert was a real takeover, identify what the intruder changed, and determine whether the activity reached other users.

The investigation followed the evidence from an impossible-travel alert to a three-night password spray, two confirmed account compromises, MFA persistence, and a mailbox rule designed to hide finance-related email.

## Results at a glance

| Question | Investigation result |
| --- | --- |
| Was Daniel's Lagos sign-in legitimate? | No. It followed failed password attempts, used a new Windows/Chrome device, and conflicted with Daniel's normal London Mac/Safari activity. |
| How did the intruder gain access? | Password spraying: 114 invalid-password events across 26 accounts from three `102.89.x.x` IP addresses. |
| What changed after access? | A `Pixel 6` MFA device was registered and the `RSS Subscriptions` mailbox rule hid finance and invoice messages. |
| Was there another victim? | Yes. `priya.nair@cloudora.io` was accessed from the attacker IP range and then used to access SharePoint Online. |
| Was every foreign sign-in malicious? | No. Omar Farah's Dubai events matched a normal travel pattern, although he was still targeted by the spray. |

## Investigation path

```text
Impossible-travel alert
        |
        v
CEO timeline and baseline
        |
        v
Password-spray attribution
        |
        v
Persistence review
        |
        v
Scope second victim and near-misses
        |
        v
Containment plan and detection rule
```

## Evidence gallery

### 1. CEO alert triage

The incident-day timeline shows the failed sign-ins, successful Lagos session, follow-on app access, and Daniel's later London sign-in.

![CEO incident-day timeline](<Screenshot%20Result/01%20Triage%20the%20Alert.png>)

### 2. Baseline the CEO account

Daniel's regular history is London-based; the Lagos event is an outlier tied to the attack window.

![Daniel location baseline](<Screenshot%20Result/02%20Baseline%20the%20account.png>)

### 3. Compare the false positive

Omar's Dubai activity provides the comparison case: repeated daytime activity on a familiar device with no failed-password lead-in.

![Omar travel baseline](<Screenshot%20Result/03%20Baseline%20the%20account%20for%20omar.png>)

### 4. Identify the password spray

Failed password activity grouped by IP reveals the many-account pattern behind the successful compromises.

![Password spray by IP](<Screenshot%20Result/04%20Hunt%20for%20credential%20attacks.png>)

### 5. Confirm the attack window

The attacker infrastructure is active over three consecutive nights before the successful sign-ins.

![Attack window by day](<Screenshot%20Result/05%20Confirm%20attack%20window.png>)

### 6. Check CEO persistence

Audit activity identifies the MFA registration and email-rule change made after the CEO account was accessed.

![CEO persistence audit events](<Screenshot%20Result/06%20Check%20For%20Persistence.png>)

### 7. Scope successful attacker activity

Filtering attacker-IP events to successes identifies every account that was accessed.

![Successful attacker sign-ins](<Screenshot%20Result/07%20Check%20Anyone%20else%20compromised.png>)

### 8. Investigate the second victim

Priya's attacker-associated activity shows failed attempts across the spray window, then a successful sign-in and SharePoint access.

![Priya timeline](<Screenshot%20Result/08%20Check%20Timeline%20of%20other%20account.png>)

![Priya baseline](<Screenshot%20Result/09%20Baseline%20of%20other%20account.png>)

![Priya audit review](<Screenshot%20Result/10%20Check%20for%20persistence%20for%20other%20account.png>)

### 9. Produce the near-miss reset list

This query separates accounts with attacker-IP failures only from the two accounts with successful attacker-IP activity.

![Near-miss reset list, part 1](<Screenshot%20Result/11%20Check%20which%20other%20account%20was%20sprayed%20.png>)

![Near-miss reset list, part 2](<Screenshot%20Result/12%20Check%20which%20other%20account%20was%20sprayed%20cont.png>)

### 10. Detection engineering

The final rule counts distinct accounts targeted by each IP in six-hour windows, allowing the spray to be detected on its first night.

![Six-hour spray detector](<Screenshot%20Result/13%20The%20rule%20to%20catch%20spray%20on%20the%20first%20day%20of%20attack.png>)

## KQL query pack

All KQL is in [Queries Used](<Queries%20Used>). The tables must be named `CloudoraSignIn_CL` and `CloudoraAudit_CL`. During Azure Data Explorer ingestion, set `TimeGenerated` to `datetime` and `ResultType` to **string**; otherwise comparisons such as `ResultType == "50126"` will not return the expected failed-sign-in rows.

| File | Analysis purpose |
| --- | --- |
| [01 - CEO triage](<Queries%20Used/01-triage-ceo-signins.kql>) | Read Daniel's complete incident-day sequence. |
| [02 - CEO baseline](<Queries%20Used/02-baseline-ceo.kql>) | Compare the Lagos event against normal locations. |
| [03 - Omar baseline](<Queries%20Used/03-baseline-omar.kql>) | Validate the legitimate-travel comparison. |
| [04 - Spray by IP](<Queries%20Used/04-password-spray-by-ip.kql>) | Find source IPs targeting many accounts. |
| [05 - Spray window](<Queries%20Used/05-password-spray-window.kql>) | Confirm the three-night pattern. |
| [06 - CEO persistence](<Queries%20Used/06-ceo-persistence-audit.kql>) | Review attacker-originated audit events. |
| [07 - Successful access](<Queries%20Used/07-scope-attacker-successes.kql>) | Scope compromised accounts. |
| [08 - Priya timeline](<Queries%20Used/08-priya-timeline.kql>) | Investigate the second victim's activity. |
| [09 - Priya baseline](<Queries%20Used/09-priya-baseline.kql>) | Compare Priya's normal and attacker-associated activity. |
| [10 - Priya audit review](<Queries%20Used/10-priya-audit-review.kql>) | Check available persistence evidence for Priya. |
| [11 - Near-miss reset list](<Queries%20Used/11-near-miss-reset-list.kql>) | List sprayed accounts without a success. |
| [12 - Spray detection](<Queries%20Used/12-password-spray-detection.kql>) | Detect the many-accounts-per-IP pattern. |

## MITRE ATT&CK mapping

| Tactic | Technique | Observed evidence |
| --- | --- | --- |
| Credential Access | [T1110.003 - Password Spraying](https://attack.mitre.org/techniques/T1110/003/) | 114 failed attempts across 26 accounts from three IPs. |
| Initial Access | [T1078 - Valid Accounts](https://attack.mitre.org/techniques/T1078/) | Successful attacker sign-ins to Daniel and Priya. |
| Persistence | [T1098.005 - Device Registration](https://attack.mitre.org/techniques/T1098/005/) | `Pixel 6` registered as a security method. |
| Defense Evasion | [T1564.008 - Email Hiding Rules](https://attack.mitre.org/techniques/T1564/008/) | `RSS Subscriptions` moved finance and invoice mail away from the inbox. |

## Tools and skills practiced

- KQL in Azure Data Explorer, transferable to Microsoft Sentinel
- Entra ID sign-in and audit-log analysis
- Account-takeover triage and user-baseline comparison
- Password-spray hunting and detection-rule design
- Scope assessment, containment planning, and incident reporting
- MITRE ATT&CK mapping

## Repository contents

```text
.
|-- Queries Used/       # 12 KQL investigation queries
|-- Report/             # formatted incident report
|-- Screenshot Result/  # screenshots from the investigation run
`-- README.md           # scenario, evidence, and findings
```

## Deliverables

- [Incident report (PDF)](<Report/incident-report.pdf>)
- [Incident report (Markdown)](<Report/incident-report.md>)
- [Query pack](<Queries%20Used>)
- [Evidence screenshots](<Screenshot%20Result>)

## Attribution and responsible use

This project is based on the Cloudora CLD-0001 learning scenario from the MyFirstHack community. The training material supplied the scenario, data, report template, and guided exercises. This repository contains my investigation outputs and portfolio presentation only; it does not redistribute source data or answer-key files.
