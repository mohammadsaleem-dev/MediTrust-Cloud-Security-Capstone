# MediTrust — Microsoft Cloud Security Capstone

**Paramount PCCP 2026 · Group 2 · Simulated healthcare environment**

MediTrust is a team cybersecurity capstone exploring how Microsoft cloud security services can protect a simulated healthcare organization. The project brings together identity and access management, endpoint threat protection, data security, security monitoring, automation, and incident response.

This repository documents the team's scope and the implementation evidence available for publication. Mohammad Sohail Saleem's IAM work includes tested emergency access, Conditional Access enforcement, Windows device compliance, Defender onboarding, and emergency-use monitoring.

> **Documentation status:** The detailed implementation and test results below are grounded in the IAM final report updated on 1 October 2026. The threat-protection, data-security, and broader monitoring/response sections describe assigned team workstreams; their completed configurations and results should be added from their owners' reports. This README does not claim that every team workstream has been independently verified.

## Contents

- [Project background](#project-background)
- [Objectives](#objectives)
- [Team responsibilities](#team-responsibilities)
- [Technology and control map](#technology-and-control-map)
- [IAM implementation](#iam-implementation)
- [Team workstreams](#team-workstreams)
- [Validation results](#validation-results)
- [Monitoring and alerting](#monitoring-and-alerting)
- [Emergency-access procedure](#emergency-access-procedure)
- [Limitations and follow-up](#limitations-and-follow-up)
- [Final lab state](#final-lab-state)
- [Suggested repository structure](#suggested-repository-structure)
- [Evidence and publication](#evidence-and-publication)
- [Lessons learned](#lessons-learned)

## Project background

Healthcare security requires dependable access for authorized staff, protected devices, sensitive-data safeguards, and a clear process for detecting and responding to suspicious activity. This capstone uses MediTrust as a fictional organization to explore those needs in a Microsoft cloud lab.

The environment uses pilot identities and a Windows test device. SharePoint Online serves as the protected application for the documented risk and device-compliance tests. It represents an application access scenario; it is not a deployed electronic health record system.

The project is an educational implementation. It does not establish production readiness, clinical deployment, or compliance certification.

## Objectives

1. Maintain emergency administrator access when routine administration or role activation is unavailable.
2. Evaluate identity risk and require the configured MFA authentication strength for selected risky sign-ins.
3. Use device compliance as an application access condition.
4. Restrict legacy authentication for the pilot population.
5. Integrate endpoint protection and investigate device-risk signals.
6. Monitor emergency sign-ins and relevant administrative changes.
7. Document threat protection, data security, monitoring, and response across the team's workstreams.
8. Retain evidence that distinguishes configuration, simulation, live enforcement, and unresolved outcomes.

## Team responsibilities

| Member | Assigned workstream | Contribution scope |
|---|---|---|
| Mohammad Sohail Saleem | Identity and Access Management | Emergency access, risky sign-ins, outdated clients, Conditional Access, device compliance, and IAM-specific monitoring and evidence |
| Ajay | Threat Protection | Threat-protection challenges and coordination of endpoint/security signals |
| Farouq | Monitoring, Automation, and Response | Central monitoring, automation, and incident-response challenges |
| Abdalaziz | Data Security | Data-security challenges and coordination of overlapping application/data controls |

The scope of this repository covers all four workstreams. Detailed verified outcomes currently available here concern the IAM implementation; each owner should contribute their own technical report and evidence for the remaining workstreams.

## Technology and control map

| Technology | Role in the project | Documentation status |
|---|---|---|
| Microsoft Entra ID | Pilot identities, groups, emergency identities, administrative roles, and sign-in records | Documented in IAM report |
| Entra Conditional Access | Risk-based MFA, compliant-device requirement, and legacy-authentication blocking | Simulated and live tests documented |
| Microsoft Intune | Windows enrollment and compliance evaluation | Documented and tested |
| Microsoft Defender for Endpoint | Device onboarding and benign detection/device-risk investigation | Onboarding and detection documented; risk-to-compliance outcome unresolved |
| Azure Monitor | IAM emergency sign-in and audit log alerts | Live firing documented |
| Log Analytics / KQL | Queries over sign-in and audit records | Documented in IAM report |
| Azure Monitor action groups | Email action configuration and notification testing | Sample email received; actual rule-generated email not received |
| SharePoint Online | Protected lab application for risk and device-compliance tests | Live tests documented |
| Microsoft Sentinel | Broader monitoring/response workstream | Owner documentation to be added; IAM emergency alert used Azure Monitor |
| Microsoft Purview | Data-security workstream | Owner documentation to be added |

## IAM implementation

### 1. Pilot identities and policy scope

The IAM implementation used two pilot users, a dedicated pilot security group, and a separate emergency-account group. Naming and scoping identified Group 2 resources within the shared training environment.

Pilot membership limited the documented Conditional Access tests to the intended users. The emergency group was excluded from the three Group 2 policies. These exclusions apply to the evaluated Group 2 scope; they do not guarantee protection from every future tenant-wide policy.

### 2. Emergency administrator access

**Problem:** Ordinary administrator authentication or PIM activation may be unavailable during a recovery event.

**Implementation:**

- Created two cloud-only emergency accounts using the tenant's `onmicrosoft.com` domain.
- Added both accounts to the emergency security group.
- Assigned Global Administrator directly to each account as a permanent active role.
- Registered Microsoft Authenticator device-bound passkeys on Android.
- Registered additional device-bound passkeys on an iPad for backup authentication.
- Used Temporary Access Passes during initial authentication-method registration.
- Excluded the emergency group from Group 2 risk, device-compliance, and legacy-authentication policies.

Group membership does not itself grant Global Administrator. The direct account role assignments provide the emergency administrative privilege.

**Validation:** Both accounts completed passkey sign-in and accessed the Entra Users → All users page. Backup iPad sign-ins also succeeded in fresh private sessions during a primary-phone-unavailable drill. Phone isolation was operator-reported; authentication and portal access were recorded.

The iPad credentials are additional registered authentication methods, not recovered copies of the Android credentials. Physical FIDO2 security keys and independently assigned custodians were not demonstrated.

### 3. Risk-based Conditional Access

**Problem:** Suspicious sign-in activity requires stronger access evaluation.

The `CA-02-RiskySignIn` policy targeted the pilot population and SharePoint Online, excluded the emergency group, and selected Medium and High sign-in risk. The grant required the configured MFA authentication strength, with sign-in frequency set to Every time.

**Validation:**

- What If checks matched Medium and High risk.
- A Low-risk What If check did not match the risk condition.
- A controlled live SharePoint sign-in showed policy Success during temporary enforcement.
- Authentication details recorded that MFA had been previously satisfied.

The live event demonstrated policy application and acceptance of an existing MFA claim. It did not demonstrate a fresh MFA challenge. The exact sign-in risk level was not captured in the final evidence, so the Tor browser and foreign location are not treated as proof of a specific risk classification.

### 4. Windows device enrollment and compliance

The Windows lab device was joined to Entra ID, enrolled in Intune, and associated with a pilot user. Its final documented system was Windows 11 Enterprise Evaluation 25H2.

| Compliance setting | Recorded lab baseline |
|---|---|
| Minimum OS version | `10.0.26200.0` |
| Firewall | Required |
| Antivirus | Required |
| Antispyware | Required |
| Maximum accepted Defender device risk | Low |
| BitLocker, Secure Boot, and code integrity | Not configured as requirements in the recorded policy |

This is the recorded lab baseline, not a verified production healthcare standard.

The `CA-03-RequireCompliantDevice` policy targeted pilot access to SharePoint Online, excluded emergency accounts, and required a compliant device.

**Live validation:** Managed compliant-device access succeeded. An unmanaged browser sign-in was blocked, with the policy showing Failure and the sign-in showing no managed/compliant device context.

### 5. Outdated operating-system simulation

The device's recorded build was `10.0.26200.9457`. For a controlled compliance test, the minimum OS threshold was raised to `10.0.26200.9999`, above the actual build.

Intune reported the minimum OS setting as Not compliant. Restoring the threshold to `10.0.26200.0` returned the device to Compliant.

This validates compliance failure and recovery through a threshold change. It does not represent a real operating-system upgrade or establish a production support lifecycle baseline.

### 6. Legacy-authentication blocking

The `CA-04-BlockLegacyAuth` policy targeted all resources for the pilot population, excluded emergency accounts, selected Exchange ActiveSync clients and Other clients, and specified Block access.

**Validation:**

- What If matched Other clients to the blocking policy.
- The browser negative test did not match the legacy client-app condition.
- A real SMTP AUTH LOGIN attempt returned an authentication failure.
- The corresponding Exchange Online sign-in showed enforced `CA-04`, Block access, and Failure.

The policy-level sign-in result establishes Conditional Access as a blocking cause; the SMTP error alone would not establish that. The attempt authenticated only and sent no email.

Legacy client-app conditions do not establish an arbitrary minimum browser version.

### 7. Defender onboarding and device-risk investigation

The Windows device was onboarded to Microsoft Defender for Endpoint. The sensor was running, onboarding state was established, and the device appeared in the intended Defender organization. Defender and Entra identifiers were checked during troubleshooting; the Defender organization ID and Entra tenant ID are distinct identifiers.

An authorized benign Microsoft demonstration generated `BmTestOfflineUI` detection and Low device risk.

For the integration test, the Intune maximum accepted device-risk threshold was temporarily set to Clear. The expected outcome was noncompliance for the Low-risk device. Despite saving and syncing, the device remained Compliant. The root cause was not established, and the threshold was restored to Low.

Defender onboarding and detection passed. Defender-risk-driven Intune noncompliance was attempted but not demonstrated. This is separate from the successful compliant/unmanaged-device access tests.

## Team workstreams

### Threat Protection — Ajay

This workstream addresses MediTrust's threat-protection challenges and coordinates relevant security signals with the identity and monitoring workstreams.

The owner contribution should document the actual services used, protection settings, authorized test scenarios, detections, incident analysis, response actions, and supporting screenshots. The IAM Defender onboarding and benign detection described above are documented overlap; they do not substitute for the full threat-protection report.

### Monitoring, Automation, and Response — Farouq

This workstream addresses central monitoring, automated workflows, and incident response. It connects security events to investigation and response procedures across the project.

The owner contribution should identify connected data sources, ingestion validation, queries and detection rules, any implemented automation, incident handling, and actual test outcomes. The IAM emergency alerts below were implemented in Azure Monitor; this README does not label them as Sentinel scheduled analytics rules.

### Data Security — Abdalaziz

This workstream addresses MediTrust's data-security challenges and coordinates application/data controls with the other owners.

The owner contribution should identify the actual Purview or other data controls implemented, their scope, policy settings, test data, expected and observed behavior, and evidence. Sensitivity labels, DLP policies, retention controls, or other features should be listed as completed only when the owner's report establishes their implementation.

## Validation results

| Area | Test | Observed outcome | Assessment |
|---|---|---|---|
| Emergency access | Primary passkey sign-ins for both accounts | Authentication and admin-portal access succeeded | Passed |
| Emergency access | iPad backup passkeys for both accounts | Authentication and admin-portal access succeeded | Passed |
| Emergency exclusions | Evaluated Group 2 policies | Not applied to emergency sign-ins | Passed for evaluated scope |
| Identity risk | Medium/High/Low What If checks | Medium and High matched; Low did not | Simulation passed |
| Identity risk | Live CA-02 test | Success with previously satisfied MFA | Policy application passed; fresh prompt not demonstrated |
| Device compliance | Managed compliant-device access | Live CA-03 Success | Passed |
| Device compliance | Unmanaged-browser access | Live CA-03 Failure and access block | Passed |
| OS baseline | Minimum above actual build | Minimum OS setting Not compliant | Passed |
| OS baseline | Restore original threshold | Device returned to Compliant | Passed |
| Legacy authentication | Client-app What If checks | Legacy category matched; Browser did not | Simulation passed |
| Legacy authentication | Real SMTP AUTH attempt | Enforced CA-04 Block access Failure | Passed |
| Endpoint protection | Onboarding | Active device and sensor verified | Passed |
| Endpoint protection | Benign demonstration | Detection and Low risk observed | Detection passed |
| Risk integration | Clear threshold with Low-risk device | Device remained Compliant | Expected outcome not observed |
| Emergency monitoring | Live sign-in alert | Fired, resolved, and fired again | Passed |
| Audit monitoring | Harmless emergency-group description change | Audit event ingested; alert fired | Passed for tested event |
| Notification | Action-group sample test | Sample email received | Passed |
| Notification | Actual rule-generated email | Not received | Delivery unresolved |

**Evidence interpretation:** Configuration proves a setting exists. What If predicts policy selection. Report-only evaluates without enforcement. Live results demonstrate the behavior of the tested scope at the recorded time. These are distinct levels of evidence.

## Monitoring and alerting

### Emergency sign-ins

Emergency sign-in records were queried in the Group 2 Log Analytics workspace. The control was implemented through Azure Monitor after Sentinel analytics navigation redirected to Defender workspace settings.

| Setting | Recorded configuration |
|---|---|
| Signal | Custom log search |
| Measure | Aggregated logs, table-row count |
| Query window | 30 minutes |
| Evaluation frequency | Every 5 minutes |
| Threshold | Count greater than 0 |
| Dimension split | None |
| Severity | 1 — Error |
| Action | Email action group |

The saved query matched the emergency accounts by user principal name and included successful and failed interactive sign-ins. The following is a sanitized equivalent, not an exported copy of the saved rule:

```kusto
SigninLogs
| where TimeGenerated > ago(30m)
| where UserPrincipalName in~ (
    "emergency01@example.onmicrosoft.com",
    "emergency02@example.onmicrosoft.com"
)
| project TimeGenerated, UserPrincipalName, AppDisplayName,
          ResultType, IPAddress
| order by TimeGenerated desc
```

Replace the example identities with approved lab identities before using the query. Stable UserId matching was validated as an alternative, but replacing the saved rule with it was not confirmed.

The alert fired during live testing. A recorded instance resolved and a subsequent instance fired. Action-group invocation was recorded. The aggregate configuration does not establish one alert or email per sign-in, and unusually late ingestion can fall outside the finite query window.

### Emergency-account and group changes

A separate alert searched AuditLogs for emergency-account and emergency-group IDs in TargetResources. A harmless group-description edit generated an Update group event and an alert.

TargetResources matching does not establish complete detection of actions performed by emergency accounts when their identity appears only in InitiatedBy. Broader actor/target coverage requires additional validation.

### Notification outcome

The recipient was verified and subscribed. A Log alert V2 sample test succeeded and the sample email arrived. An actual rule-generated emergency email was not received despite live alert firing and recorded action-group invocation.

The sample establishes the tested action-group email path. It does not establish successful end-to-end email delivery from the actual emergency rule.

## Emergency-access procedure

The following operational procedure accompanies the tested authentication paths. Custodian assignment and secure-storage arrangements remain recommendations unless separately implemented.

1. Record the incident or drill reason and the failure preventing ordinary administration.
2. Obtain approval where possible; record the justification for urgent use and arrange retrospective review if needed.
3. Retrieve the approved emergency authentication method through the designated custodian.
4. Use a secure workstation and a fresh private browser session; verify the intended tenant and identity.
5. Confirm the expected administrative role and perform only the recorded recovery actions.
6. Retain sign-in and audit evidence and record each change and its time.
7. Confirm ordinary administrator access is restored, then sign out and close the emergency session.
8. Review alerts, audit records, authorization, and notification delivery with the monitoring owner.
9. Re-secure credentials and address exposed methods or residual sessions as appropriate.

A second registered device provides an additional tested path. A new Temporary Access Pass still requires another authorized administrator, so it is not independent recovery when all administrators are locked out.

## Limitations and follow-up

| Item | Current limit | Follow-up |
|---|---|---|
| Live email | Actual rule email not received | Investigate message delivery and repeat a correlated end-to-end test |
| Defender risk integration | Low-risk device stayed Compliant with Clear threshold | Investigate connector/evaluation behavior and repeat with documented timing |
| MFA challenge | Existing claim satisfied live requirement | Validate a fresh challenge if required by the scenario |
| Emergency independence | Shared backup iPad and authentication technology | Establish separate custodians/storage and evaluate independent recovery methods |
| Audit coverage | Tested target-ID match | Validate role, credential, membership, and actor-based events |
| Stable rule matching | Saved rule still uses UPN matching | Confirm and document stable-ID migration |
| OS baseline | Threshold simulation only | Define supported production versions and test a genuine remediation path |
| Team evidence | Full owner reports not incorporated | Add reports and test results from the three other workstreams |

## Final lab state

At completion on 1 October 2026, the owner confirmed:

- All three Group 2 Conditional Access policies were restored to **Report-only** after controlled live enforcement tests.
- The maximum accepted Defender device risk was restored to **Low**.
- The minimum Windows version was restored to `10.0.26200.0`.
- The Windows lab device was **Compliant**.

The final state was owner-confirmed. Earlier screenshots record their own capture times and should not be relabeled as final-state screenshots. Defender risk clearing was not separately established.

## Suggested repository structure

This is a proposed layout. Only this README is supplied with this draft; the folders and their contents should be added as the team prepares them.

```text
README.md
docs/
  project-overview.md
  iam/
    implementation.md
    testing-results.md
    emergency-access-procedure.md
  threat-protection/
    implementation.md
    testing-results.md
  monitoring-response/
    implementation.md
    testing-results.md
  data-security/
    implementation.md
    testing-results.md
queries/
  emergency-signins.kql
  emergency-audit-changes.kql
evidence/
  iam/
  threat-protection/
  monitoring-response/
  data-security/
reports/
  public-team-summary.pdf
```

## Evidence and publication

For each published test, retain its purpose, prerequisites, policy state, expected outcome, observed outcome, timestamp, and relevant screenshot or log excerpt. Label simulations and sample notifications explicitly.

Useful IAM evidence includes emergency role assignments and passkey results, policy exclusions, risk What If checks, live CA results, compliance failure/restoration, Defender onboarding/detection, alert histories, and the separately labeled sample email.

Before publishing reports or screenshots, remove credentials, Temporary Access Passes, recovery information, tokens, tenant-specific identities/IDs, and unrelated shared-tenant information. Use sanitized copies and keep original evidence under the team's approved handling process. This README intentionally uses example identities in its query.

## Lessons learned

- Emergency access depends on usable authentication, administrative roles, policy treatment, monitoring, and recovery procedures together.
- A successful risk policy result can accept an existing MFA claim; it does not necessarily mean a new challenge appeared.
- Identity risk and Defender device risk are separate signals with separate policy and evaluation paths.
- Device onboarding, endpoint detection, compliance evaluation, and access enforcement each need their own evidence.
- An alert can fire while its live notification delivery remains unresolved.
- Threshold-based OS testing demonstrates compliance logic without demonstrating a real update.
- Shared-tenant scoping and cleanup are part of a controlled lab implementation.

## Credits

Developed by PCCP 2026 Group 2 during the Paramount Computer Systems program. Detailed IAM implementation and validation documented by Mohammad Sohail Saleem; other workstreams owned by Ajay, Farouq, and Abdalaziz as listed above.

**Source record:** MediTrust IAM Final Implementation and Testing Report, updated 1 October 2026. This repository is an educational team project and is not an official Microsoft product or a production healthcare deployment.
