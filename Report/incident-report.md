# Security Incident Report

| Field | Detail |
| --- | --- |
| Report ID | CLD-IR-0001 |
| Related ticket | CLD-0001 (Suspicious login on `daniel.reeve@cloudora.io`) |
| Report title | Investigation into Account Takeover, MFA Tampering, and Email Rule Setup |
| Analyst | SOC Analyst |
| Date of report (UTC) | 2026-08-10 |
| Incident severity | P1 (The CEO's account got breached right before a major client deal, and an inbox rule was planted) |
| Status | Contained (attacker kicked out, extra accounts checked) |
| Classification | CONFIDENTIAL - Internal and client distribution only |

## 1. Executive summary

Over three nights, an attacker tested common passwords against 26 company accounts using IP addresses from Nigeria. On the morning of August 10, they cracked the passwords for both the CEO (Daniel Reeve) and an employee (Priya Nair). On the CEO's account, the attacker added their own phone as an MFA method and set up a rule to hide invoice emails. The team spotted the login when Daniel signed in from London a few hours later. We ended all active sessions, changed passwords, deleted the attacker's MFA device and email rule, and blocked the IPs. No company funds were lost, and we flagged the other 24 accounts that were targeted so they can reset their passwords too.

## 2. Incident timeline

All times are in UTC. Data comes from sign-in logs (`CloudoraSignIn_CL`) and audit logs (`CloudoraAudit_CL`).

| Time (UTC) | Source | Event | Evidence / notes |
| --- | --- | --- | --- |
| Aug 08, 00:00-05:00 | Sign-in | Night 1 password spray: 44 failed logins across ~20 accounts | Failed login error 50126 from `102.89.x.x` addresses. |
| Aug 09, 00:00-05:00 | Sign-in | Night 2 password spray: 36 failed logins | Same IPs, trying a couple of times per account. |
| Aug 10, 00:35:48 | Sign-in | First failed attempt against `daniel.reeve` | IP `102.89.44.17`. |
| Aug 10, 03:09:12 | Sign-in | Failed login on CEO account | Error 50126, IP `102.89.44.17`. |
| Aug 10, 03:10:41 | Sign-in | Failed login on CEO account | Error 50126, IP `102.89.44.17`. |
| Aug 10, 03:12:05 | Sign-in | CEO account compromised | Successful login (code 0) from `102.89.44.17` on Windows 10 / Chrome. |
| Aug 10, 03:14:30 | Sign-in | Attacker opens Outlook Web | Mailbox opened from `102.89.44.17`. |
| Aug 10, 03:18:44 | Audit | Attacker adds MFA device | Phone named Pixel 6 registered on CEO account. |
| Aug 10, 03:26:02 | Sign-in | Attacker opens Azure Portal | Session from `102.89.44.17`. |
| Aug 10, 03:31:09 | Audit | Inbox rule created | Rule RSS Subscriptions moves invoice and finance emails to RSS folder. |
| Aug 10, 03:44:55 | Sign-in | Failed login on Priya Nair's account | Error 50126 from `102.89.45.101`. |
| Aug 10, 03:47:18 | Sign-in | Priya Nair account compromised | Successful login (code 0) from `102.89.45.101`. |
| Aug 10, 03:52:40 | Sign-in | Attacker accesses SharePoint Online | Used Priya's account from `102.89.45.101`. |
| Aug 10, 08:41:00 | Sign-in | Daniel signs in from London | Regular Mac/Safari login from `203.0.113.11`. |
| Aug 10, 08:55:00 | Ticket | Alert raised by IT admin | Ticket CLD-0001 opened for impossible travel. |
| Aug 10, ~10:30 | Response | Response steps completed | Sessions killed, passwords reset, MFA/rules cleaned up. |

## 3. Findings

### Finding 1: The CEO's account had an unauthorized login, not normal travel

**Fact:** Daniel Reeve's account was logged into by someone else from Lagos, Nigeria.

**Evidence:** The account logged in at 03:12 UTC from `102.89.44.17` using Windows 10 and Chrome. Daniel normally signs in from London on a Mac using Safari, and he signed in from London as usual at 08:41 UTC.

**Why it matters:** Flying between Lagos and London in under six hours is not possible. Seeing two wrong password tries right before the login also proves it was an attacker trying passwords, not a VPN or travel glitch.

### Finding 2: Entry was gained through a slow password spray

**Fact:** The attacker did not use phishing; they guessed passwords across many users over three days.

**Evidence:** In the sign-in logs, three IPs from Lagos (`102.89.44.17`, `102.89.44.23`, `102.89.45.101`) made 114 failed attempts across 26 different accounts between midnight and 5 AM. Each account was only tried 1 to 3 times per night.

**Why it matters:** Spreading attempts out this way avoids locking user accounts, so nobody notices until a password works.

### Finding 3: The attacker added their own MFA device to keep access

**Fact:** The attacker linked their own phone to Daniel's account.

**Evidence:** At 03:18 UTC, audit logs show a new security method was added from `102.89.44.17`, adding a phone called Pixel 6.

**Why it matters:** If we only changed Daniel's password, the attacker could still use their phone to approve MFA prompts and get right back in.

### Finding 4: An inbox rule was set up to steal and hide financial emails

**Fact:** The attacker tampered with Daniel's mailbox settings.

**Evidence:** At 03:31 UTC, an inbox rule called RSS Subscriptions was made. It directs any mail with the word "invoice" or sent from `finance@cloudora.io` to the RSS Feeds folder and marks it as read.

**Why it matters:** This hides billing and invoice emails from Daniel so the attacker can talk to finance or clients secretly and commit payment fraud.

### Finding 5: Priya Nair's account was also breached

**Fact:** Daniel was not the only victim.

**Evidence:** The same IP pool got into `priya.nair@cloudora.io` at 03:47 UTC, and opened SharePoint five minutes later.

**Why it matters:** Looking past the first alert showed that another staff member was breached and had internal files opened.

### Finding 6: Omar Farah's logins were legitimate travel

**Fact:** Omar's logins from Dubai were genuine.

**Evidence:** Omar had 12 logins from Dubai between August 8 and 10. They were all during normal daytime hours, had zero password errors, and used his regular iPhone.

**Why it matters:** This shows how to tell the difference between real travel and a hack. Even so, because the attacker also guessed his password during the spray, he still needs a password reset.

## 4. Indicators of compromise (IOCs)

| Type | Value | First seen (UTC) | Context |
| --- | --- | --- | --- |
| IPv4 | `102.89.44.17` | Aug 08, 00:00 | Attacker IP used for spray, CEO login, MFA setup, and mail rule. |
| IPv4 | `102.89.44.23` | Aug 08, 00:00 | Attacker IP used for password spray. |
| IPv4 | `102.89.45.101` | Aug 08, 00:00 | Attacker IP used for spray and Priya Nair breach. |
| User Agent / OS | Windows 10 / Chrome 125 | Aug 08 | Device fingerprint used by the attacker. |
| MFA Device | Pixel 6 | Aug 10, 03:18:44 | Rogue phone added to Daniel's account. |
| Mailbox Rule | RSS Subscriptions | Aug 10, 03:31:09 | Rule created to hide invoice emails. |

## 5. MITRE ATT&CK mapping

| Tactic | Technique ID | Technique name | Evidenced by |
| --- | --- | --- | --- |
| Credential Access | T1110.003 | Brute Force: Password Spraying | Finding 2 (114 failed attempts across 26 users). |
| Initial Access | T1078 | Valid Accounts | Findings 1 and 5 (successful logins on Daniel and Priya). |
| Persistence | T1098.005 | Account Manipulation: Device Registration | Finding 3 (Pixel 6 registered for MFA). |
| Defense Evasion | T1564.008 | Hide Artifacts: Email Hiding Rules | Finding 4 (RSS Subscriptions inbox rule). |

## 6. Scope

### Accounts confirmed compromised (2)

`daniel.reeve@cloudora.io` and `priya.nair@cloudora.io` (both had successful logins from the attacker IPs on August 10).

### Accounts targeted but not breached (24)

These accounts were sprayed by the attacker but never had a successful login. All need their passwords changed:

`alba.vega@cloudora.io`, `amelia.frost@cloudora.io`, `aria.reid@cloudora.io`, `cole.burke@cloudora.io`, `dina.said@cloudora.io`, `emma.hayes@cloudora.io`, `ethan.wells@cloudora.io`, `freya.lynn@cloudora.io`, `gwen.muir@cloudora.io`, `isla.grant@cloudora.io`, `joel.kerr@cloudora.io`, `jude.ross@cloudora.io`, `kian.patel@cloudora.io`, `leah.stone@cloudora.io`, `lena.voss@cloudora.io`, `liam.doyle@cloudora.io`, `mira.shah@cloudora.io`, `nina.cole@cloudora.io`, `omar.farah@cloudora.io`, `rhys.owen@cloudora.io`, `ruth.dean@cloudora.io`, `ryan.boyd@cloudora.io`, `seth.lane@cloudora.io`, `sofia.marino@cloudora.io`.

### Accounts investigated and cleared (1)

`omar.farah@cloudora.io` (checked because of logins from Dubai, but confirmed as benign business travel).

## 7. Actions taken

Carried out on August 10, 2026:

| Time (UTC) | Action | Performed by | Verified how |
| --- | --- | --- | --- |
| 10:15 | Revoked sessions and tokens for Daniel and Priya | SOC Analyst | Confirmed open sessions were dropped in Entra ID. |
| 10:20 | Reset passwords for both accounts | SOC Analyst | Verified passwords were changed and required at next sign-in. |
| 10:25 | Removed the Pixel 6 MFA device from Daniel's profile | SOC Analyst | Checked user authentication methods; only Daniel's real device remained. |
| 10:30 | Deleted the RSS Subscriptions inbox rule | SOC Analyst | Checked mailbox rules in Exchange to confirm it was gone. |
| 10:35 | Blocked the three attacker IPs in Conditional Access | SOC Analyst | Verified IPs added to the block list. |
| 10:45 | Re-checked sign-in and audit logs | SOC Analyst | Ran queries to ensure no traffic came from those IPs after containment. |

## 8. Recommendations

1. **Reset targeted accounts:** Make all 24 targeted staff members change their passwords right away.
2. **Warn the billing team:** Tell the finance team to call and verbally verify any bank detail changes, especially while closing the current deal.
3. **Turn on MFA everywhere:** Ensure multi-factor authentication is required on every account and disable legacy protocols that can skip it.
4. **Alert on suspicious changes:** Create automated alerts whenever an executive adds a new MFA device or makes a rule moving invoice emails.
5. **Set up spray detection:** Create a SIEM detection rule to spot when an IP fails passwords across several users.

## 9. Lessons learned

**Catching sprays early:** The attack was discovered 5.5 hours after the CEO's account fell because an IT admin saw Daniel login from two distant places. If we had a spray detection rule running, we could have blocked the attacker on night one.

**Checking audit logs matters:** A password reset alone would have left the attacker's Pixel 6 phone and the hiding email rule active. Looking at audit logs is essential to clean up an account completely.

**Context over simple alerts:** Omar's Dubai sign-ins looked suspicious on paper, but looking at his device and normal hours showed it was just travel, saving us from a false alarm.
