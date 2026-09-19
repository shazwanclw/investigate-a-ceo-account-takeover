# Security Incident Report

| Field | Detail |
| --- | --- |
| Report ID | CLD-IR-0001 |
| Related ticket | CLD-0001 - Suspicious login on `daniel.reeve@cloudora.io` |
| Title | Executive account takeover investigation: password spray, MFA persistence, and mail-rule tampering |
| Analyst | SOC Analyst - simulated Cloudora engagement (MyFirstHack training) |
| Report date (UTC) | 2026-08-10 |
| Severity | P1 - executive mailbox compromise during an active enterprise deal, with financial-email concealment staged |
| Status | Containment response documented for training; validation would be required in a live tenant |
| Classification | CONFIDENTIAL - Internal and client distribution only |

> **Training context:** Cloudora and all evidence in this report are fictional. Response actions below are the documented actions appropriate to this simulated scenario, not claims of access to a live environment.

## 1. Executive summary

Between 8 and 10 August 2026, an external source performed a low-volume password spray against 26 Cloudora accounts from three Lagos-based IP addresses. Early on 10 August, the activity led to successful access to the CEO account and a second employee account. On the CEO account, the intruder registered a new MFA device and created a rule intended to conceal finance and invoice-related email. The alert was raised after the CEO's normal London activity created an impossible-travel indicator. The available sign-in and audit logs show no payment or fraud transaction evidence; however, the mail-rule behaviour created material business-email-compromise risk and requires immediate containment.

## 2. Incident timeline

All timestamps are UTC. Log sources are `CloudoraSignIn_CL` and `CloudoraAudit_CL` unless otherwise noted.

| Time (UTC) | Source | Event | Evidence / notes |
| --- | --- | --- | --- |
| Aug 08, 00:00-05:00 | Sign-in | Password spray, night 1 | 44 failed sign-ins from `102.89.x.x` across about 20 accounts. |
| Aug 09, 00:00-05:00 | Sign-in | Password spray, night 2 | 36 failed sign-ins from the same infrastructure. |
| Aug 10, 00:35:48 | Sign-in | Daniel targeted | Failed sign-in from `102.89.44.17`. |
| Aug 10, 03:09:12 | Sign-in | CEO authentication failure | `50126` for Daniel from `102.89.44.17`. |
| Aug 10, 03:10:41 | Sign-in | CEO authentication failure | Second `50126` for Daniel from the same IP. |
| Aug 10, 03:12:05 | Sign-in | CEO account accessed | Success (`0`) from Lagos on Windows 10 / Chrome. |
| Aug 10, 03:14:30 | Sign-in | Outlook Web opened | Mailbox activity from `102.89.44.17`. |
| Aug 10, 03:18:44 | Audit | MFA method registered | `Pixel 6` was added to Daniel's security information. |
| Aug 10, 03:26:02 | Sign-in | Azure Portal opened | Same CEO session and attacker IP. |
| Aug 10, 03:31:09 | Audit | Mailbox rule created | `RSS Subscriptions` moves finance or invoice mail to RSS Feeds and marks it read. |
| Aug 10, 03:44:55 | Sign-in | Priya targeted | Failed authentication from `102.89.45.101`. |
| Aug 10, 03:47:18 | Sign-in | Priya account accessed | Success (`0`) from `102.89.45.101`. |
| Aug 10, 03:52:40 | Sign-in | SharePoint Online accessed | Follow-on activity using Priya's account. |
| Aug 10, 08:41:00 | Sign-in | Daniel's normal London sign-in | Usual Mac/Safari activity from `203.0.113.11`. |
| Aug 10, 08:55:00 | Ticket | Alert opened | IT administrator raised CLD-0001 for impossible travel. |
| Aug 10, approximately 10:30 | Response plan | Containment sequence | Training-scenario response actions documented in section 7. |

## 3. Findings

### Finding 1 - Daniel Reeve's account was accessed without authorization

Daniel's account successfully authenticated from `102.89.44.17` in Lagos at 03:12:05 after two invalid-password results. This device was Windows/Chrome, while the account baseline was London-based and Daniel used Mac/Safari from London at 08:41 the same day. The timing, unfamiliar device, preceding failures, and conflicting location support account takeover rather than routine travel or VPN use.

### Finding 2 - The initial-access pattern is password spraying

Three attacker IPs produced 114 `50126` failures across 26 accounts between 8 and 10 August, normally only one to three attempts per account in a night window. The distributed, low-volume pattern is consistent with password spraying and maps to T1110.003. It is materially different from a user repeatedly mistyping one password.

### Finding 3 - A rogue MFA device created persistence on the CEO account

At 03:18:44, audit activity recorded a new security-information registration from the attacker IP, with device name `Pixel 6`. A password reset alone would not be sufficient if that device remained registered, so the evidence supports a device-registration persistence finding (T1098.005).

### Finding 4 - The CEO mailbox was prepared for financial-email concealment

At 03:31:09, the attacker created `RSS Subscriptions`, which redirects finance-sender or invoice-keyword messages to the RSS Feeds folder and marks them read. This is an email-hiding rule (T1564.008) that could facilitate business email compromise. The supplied logs do not show a fraudulent transfer or outgoing fraud email.

### Finding 5 - Priya Nair was a second confirmed victim

Priya's account was successfully accessed at 03:47:18 from `102.89.45.101`, after a failed attempt at 03:44:55, and then used for SharePoint Online access. Her baseline is London activity, with no established foreign-travel pattern. The available audit records do not show attacker persistence for Priya; that absence is a log limitation, not proof that no persistence existed.

### Finding 6 - Omar Farah's Dubai activity was legitimate travel, not compromise

Omar had 12 Dubai sign-ins over three days, during daytime hours, from his usual iOS device and without preceding failures. Daniel's Lagos events were a short early-morning burst from an unfamiliar Windows/Chrome device after failures. Omar remains in the reset population because he was sprayed, but he is cleared as a compromise victim.

## 4. Indicators of compromise

| Type | Value | First seen (UTC) | Context |
| --- | --- | --- | --- |
| IPv4 | `102.89.44.17` | Aug 08, approximately 00:00 | Spray source; Daniel sign-in, MFA registration, and mail-rule creation. |
| IPv4 | `102.89.44.23` | Aug 08, approximately 00:00 | Spray source. |
| IPv4 | `102.89.45.101` | Aug 08, approximately 00:00 | Spray source and Priya compromise. |
| Device / user agent | Windows 10 / Chrome 125 | Aug 08 | Attacker-associated device pattern. |
| MFA device | `Pixel 6` | Aug 10, 03:18:44 | Unauthorized registration on Daniel's account. |
| Inbox rule | `RSS Subscriptions` | Aug 10, 03:31:09 | Rule hiding finance and invoice-related mail. |

## 5. MITRE ATT&CK mapping

| Tactic | Technique ID | Technique | Evidenced by |
| --- | --- | --- | --- |
| Credential Access | T1110.003 | Brute Force: Password Spraying | Finding 2: 114 failures across 26 accounts. |
| Initial Access | T1078 | Valid Accounts | Findings 1 and 5: successful sign-ins after spray activity. |
| Persistence | T1098.005 | Account Manipulation: Device Registration | Finding 3: `Pixel 6` registration. |
| Defense Evasion | T1564.008 | Hide Artifacts: Email Hiding Rules | Finding 4: `RSS Subscriptions` rule. |

## 6. Scope

### Confirmed compromised accounts (2)

| Account | Evidence |
| --- | --- |
| `daniel.reeve@cloudora.io` | Successful attacker-IP sign-in at 03:12:05; Outlook Web and Azure Portal activity; MFA and mailbox-rule changes. |
| `priya.nair@cloudora.io` | Successful attacker-IP sign-in at 03:47:18; subsequent SharePoint Online activity. |

### Targeted but not breached (24)

`alba.vega@cloudora.io`, `amelia.frost@cloudora.io`, `aria.reid@cloudora.io`, `cole.burke@cloudora.io`, `dina.said@cloudora.io`, `emma.hayes@cloudora.io`, `ethan.wells@cloudora.io`, `freya.lynn@cloudora.io`, `gwen.muir@cloudora.io`, `isla.grant@cloudora.io`, `joel.kerr@cloudora.io`, `jude.ross@cloudora.io`, `kian.patel@cloudora.io`, `leah.stone@cloudora.io`, `lena.voss@cloudora.io`, `liam.doyle@cloudora.io`, `mira.shah@cloudora.io`, `nina.cole@cloudora.io`, `omar.farah@cloudora.io`, `rhys.owen@cloudora.io`, `ruth.dean@cloudora.io`, `ryan.boyd@cloudora.io`, `seth.lane@cloudora.io`, and `sofia.marino@cloudora.io`.

These accounts produced attacker-IP failures but no attacker-IP success. Query 11 provides the repeatable reset list.

### Investigated and cleared (1)

| Account | Assessment |
| --- | --- |
| `omar.farah@cloudora.io` | Dubai activity is consistent with travel. This account is still included above because it received failed spray attempts. |

## 7. Documented response actions for this training scenario

| Order | Action | Owner | Verification expected |
| --- | --- | --- | --- |
| 1 | Revoke active sessions and refresh tokens for Daniel and Priya. | SOC analyst / identity administrator | Confirm existing sessions are invalidated. |
| 2 | Reset credentials for both confirmed victims. | SOC analyst / identity administrator | Force reauthentication and confirm new passwords. |
| 3 | Remove `Pixel 6` from Daniel's methods; review Priya's authentication methods. | Identity administrator | Confirm no attacker-controlled method remains. |
| 4 | Delete `RSS Subscriptions`; review both mailboxes for additional rules. | Messaging administrator | Confirm the rule is absent. |
| 5 | Block the three known attacker IPs through Conditional Access or named locations. | Identity administrator | Confirm policy deployment and test block behaviour. |
| 6 | Re-run triage and scoping queries after containment. | SOC analyst | Confirm no post-containment activity from `102.89.x.x`. |

## 8. Recommendations

1. Force password resets for the 24 spray-targeted accounts, including Omar Farah.
2. Enforce MFA on all accounts and remove legacy authentication paths that can bypass it.
3. Alert on executive MFA method additions and new mailbox rules that redirect finance or invoice-related messages.
4. Deploy the six-hour password-spray rule in [Query 12](../Queries%20Used/12-password-spray-detection.kql) and tune its threshold to the tenant's normal egress patterns.
5. Review Conditional Access policy for countries where Cloudora has no business requirement, using step-up verification or a block where appropriate.
6. Brief finance staff to verify payment-detail changes through an established out-of-band contact method.

## 9. Lessons learned

The incident was surfaced by impossible travel approximately five and a half hours after the CEO account was accessed, although the password spray was visible two days earlier. A rule based on distinct accounts targeted per source IP would have surfaced the pattern on its first night. Audit-log review changed the response from a credential reset into a complete persistence cleanup. Finally, comparing the abnormal sign-in with Omar's genuine travel demonstrated why country alone is not a compromise verdict: timing, device history, failure pattern, and user context are decisive.
