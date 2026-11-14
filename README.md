# When a Valid Login Isn't Enough: SAML Certificate Rollover & Recovery

**Cedarbridge Health System (CHS)** · **Microsoft Entra ID** · **SAML 2.0 SSO** · **X.509 signing certificates** · **Zammad** · **Certificate lifecycle** · **Trust management**

> **A user can have a valid account, satisfy MFA, and still be locked out of an application.** If Entra starts signing SAML assertions with a certificate the application does not trust, the authentication handoff breaks. This lab follows a planned certificate change from baseline and preparation to rollover, rollback, and fresh sign-in verification.

## The problem: What happens when the trust changes?

Cedarbridge Health System is a **fictional healthcare organization** using a Zammad-based Clinic Integration Support Desk. Staff use the portal to report technical issues, including clinical-system access problems. Even a short SSO interruption could delay support requests at the wrong time.

Its SAML integration relies on a signing certificate in **Microsoft Entra ID** and a matching **IdP public certificate configured in Zammad**. That dependency can be easy to overlook: replacing a certificate in the identity provider is not enough if the service provider still trusts the previous one.

The original certificate was valid until **November 2029**. There was **no live expiration incident**. I performed a **proactive rotation simulation** to answer three operational questions:

- **Can we identify the current signing certificate and prepare its replacement before cutover?**
- **Can we coordinate the trust change and prove *new* SAML sessions work?**
- **If the change breaks authentication, is there a verified way back?**

**My objective:** Rotate an Entra SAML signing certificate without losing administrative recovery access, verify application-level SSO after each change, and leave a reproducible rotation runbook.

## The solution: preserve trust, verify the handoff

```text
Microsoft Entra ID (IdP)                       Zammad (Service Provider)
  Signs SAML response                            Verifies the signature
  with Certificate A ──────────── trusted by ───> Certificate A
          │
          │  Stage Certificate B as inactive
          │  Back up A + confirm local admin access
          v
  Planned cutover: update Zammad trust to B → activate B in Entra
          │
          v
  Fresh CHS Employee SSO login → confirm Zammad + Entra sign-in
          │
          v
  Recovery drill: restore A on both sides → fresh login succeeds
          │
          v
  Final state: B active/trusted; A retained inactive for rollback
```

| Safeguard | Implementation | Verified outcome |
| --- | --- | --- |
| **Certificate inventory** | Inspected Entra active status, validity and thumbprint; reviewed Zammad's IdP certificate setting | Original signing certificate identified; baseline SSO worked |
| **Staged replacement** | Created a new Entra SAML signing certificate and left it **Inactive** | Original remained Active during preparation |
| **Change coordination** | Updated Zammad's trusted public IdP certificate and activated the matching Entra certificate | Fresh SAML login reached Jordan's Zammad account |
| **Recovery access** | Kept a separate, functioning Zammad **local admin** session and backup of the original certificate | SAML configuration remained recoverable during cutover |
| **Rollback drill** | Restored the original certificate on both sides, then tested a **new SSO login** | Original trust path worked again |
| **Final-state check** | Reapplied the replacement to Zammad, activated it in Entra and signed in again | Replacement Active, original Inactive; fresh SSO succeeded |

## Case study 1 — Preparing for the failure *before* it happens

**Scenario:** The support desk's signing certificate is due for replacement. An administrator who activates a new Entra certificate without updating Zammad risks breaking authentication, even if users and MFA policies are healthy.

I inspected the SAML certificate inventory in **Entra Enterprise Applications**, confirmed the original certificate's validity through November 2029, and checked Zammad's current **IdP Certificate** trust configuration. I then generated a second Entra certificate but kept it **Inactive** while the original continued signing assertions.

The preparation included downloading both **public certificates**, keeping the original trust value available privately for rollback, and confirming that Zammad's local administrator login still worked without CHS SSO. This was a controlled **manual** change, not an automated metadata rollover.

**Evidence:** [Zammad's original SAML trust configuration](Evidence/01-Baseline/PKI-Zammad-Trust-Baseline.png). Additional Entra certificate-inventory and staging screenshots are described in the [evidence index](Evidence/README.md).

**Lesson:** Certificate rotation is an application-trust change, not merely a new file generated in Entra.

## Case study 2 — A successful login needs a fresh session

**Change:** I placed the new public signing certificate into Zammad's IdP Certificate field and activated that same replacement certificate in Entra. Zammad uses a single configured IdP certificate, so there was a possible **brief mismatch window** during the coordinated switch; this lab does not claim zero downtime.

I opened a fresh private-browser sign-in using **CHS Employee SSO** and signed in as **Jordan Lee**, a fictional test employee. The user reached Zammad's ticket dashboard. The related Entra sign-in record showed the **CHS – Clinic Integration Support Desk** application with **Success**.

**Evidence:** [Successful Entra application sign-in following the rotation](Evidence/03-Rollover/PKI-SSO-Post-Rotation-Success.png).

**Lesson:** A green certificate status in Entra does not prove the service provider trusts it. A **fresh, application-level SSO test** is the more meaningful verification.

## Case study 3 — Proving the rollback, not just writing it down

The engineering decision I wanted to test was **recovery before retirement**. I did not delete the original certificate after activating the replacement.

I restored the original IdP certificate in Zammad, reactivated it in Entra and performed a fresh SSO login. It worked. Zammad also displayed a prompt to reload an existing session; I **signed out and signed in again** rather than treating a refresh as evidence of new SAML authentication.

After the rollback drill, I repeated the planned cutover and verified another fresh login with the **replacement Active** and the **original Inactive**. The retained inactive certificate provides a rollback option, but is **not a substitute** for periodic certificate retirement and a tested recovery procedure.

**Evidence:** See the [rollback and final-state screenshot checklist](Evidence/README.md) and the [rotation runbook](Docs/Certificate-Rotation-Runbook.md).

**Lesson:** The recovery procedure is part of the change, not an afterthought. An already-open application session is not proof that new federation requests will succeed.

## What this lab demonstrates—and what it doesn't

**Demonstrated:** SAML 2.0 IdP/service-provider trust coordination, X.509 certificate staging, planned rollover, independent SSO verification, local administrator recovery, successful rollback and restoration of the replacement certificate.

**Not demonstrated:** An actual certificate-expiration outage; certificate-authority deployment; private-key generation or escrow; automated federation-metadata refresh; signing-algorithm cryptographic inspection; continuous availability during rollover; or enterprise-wide expiry discovery. Expiration-notification *delivery* was not independently tested.

For real operations, I'd add monitored ownership/notification distribution lists, approved maintenance windows, a certificate inventory with expiry alerts, a second-person change review, automated SP metadata ingestion where supported, and post-change monitoring across affected applications.

**Runbook:** [Certificate rotation, validation and recovery](Docs/Certificate-Rotation-Runbook.md) · **Evidence:** [Screenshot index](Evidence/README.md)

*CHS and the employee accounts are simulated. The original certificate was not about to expire; this was a controlled proactive lifecycle test, not a production outage or a full PKI deployment.*
