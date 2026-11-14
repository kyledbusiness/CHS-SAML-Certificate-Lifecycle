# PKI Operations Runbook — Entra SAML Signing-Certificate Lifecycle

**System:** Cedarbridge Health System (simulated) · Entra enterprise application `CHS – Clinic Integration Support Desk` · Zammad SAML SSO.  
**Change type:** Supervised, proactive X.509 signing-certificate rollover / trust update.  
**Scope:** **PKI controls supporting IAM federation**, not a certificate-authority deployment or automated renewal system.

## Control objective and ownership

Keep **Entra's active private signing key / public X.509 certificate** aligned with the **IdP public certificate trusted by Zammad**. The IAM application owner is accountable for inventory, expiry notification ownership, approved cutover, SSO validation, rollback, and retirement of obsolete trust material. Entra manages its certificate's private key; the lab operator downloaded only the public certificate.

## 1. Change preflight / certificate inventory

- Identify the Entra application, change owner, SP owner, change window, on-call escalation, and recovery decision point. Obtain production authorization before any change.
- Inspect **Entra ID → Enterprise applications → Zammad → Single sign-on → SAML Signing Certificate**. Record **Active/Inactive status**, expiry, thumbprint and **Sign SAML assertion / SHA-256** settings.
- Verify **Notification Email Addresses** include monitored owners. Entra's built-in reminders are described at **60, 30 and 7 days before expiry**; configuration is not proof that an email was delivered.
- In Zammad, verify **Settings → Security → Third-party Applications → Authentication via SAML → IdP Certificate** is populated and SSL Verification = yes. Use the certificate rather than the deprecated SHA-1 fingerprint field.
- Perform **fresh private-browser CHS Employee SSO login** as an approved test identity. Verify application access and matching Entra sign-in event.
- Confirm **independent local Zammad administrator login** and an authorized Entra administrator, both available during rollover.
- Privately back up original public certificate and existing Zammad trust text. Avoid putting real SSO URLs, certificate backups, tenant IDs, keys, or secrets in GitHub.

## 2. Stage replacement B without changing live trust

1. Generate **new X.509 signing certificate B** in Entra; **Save**.
2. Ensure **A = Active, B = Inactive**; record expiration and thumbprint.
3. Download B's **Base64 public certificate** and inspect the entire PEM text (`BEGIN CERTIFICATE` through `END CERTIFICATE`). Do not export signing private keys.
4. Confirm rollback copy of A and non-SSO Zammad administrator access again.

## 3. Coordinated trust rotation A → B

> **Risk:** Zammad's lab configuration has one IdP public certificate field. There may be a short mismatch window. This lab did **not** demonstrate zero downtime.

1. With local Zammad admin open, replace the **IdP Certificate** field with **B's public certificate**, save. Keep deprecated fingerprint field unused.
2. In Entra, **Make B Active**, verify **A Inactive**. Don't delete A.
3. Open a **new private browser session**. Initiate CHS Employee SSO and confirm successful access to Zammad as test employee Jordan.
4. Correlate with the **CHS – Clinic Integration Support Desk** Entra sign-in event; confirm **Success**. An existing Zammad page or session refresh is not a valid new-login test.
5. If validation fails, **stop and use rollback** rather than continuing ad-hoc certificate changes.

## 4. Rollback B → A

1. Restore **A's public certificate** in Zammad and save using independent local admin access.
2. Make **A Active** in Entra; verify B Inactive.
3. Sign out of Zammad, open a new private browser session, and test CHS Employee SSO.
4. Verify successful entry into the application and, where available, the related Entra sign-in event.
5. Document the result. Zammad's **Config has changed / reload** notice alone does *not* prove new SAML authentication.

## 5. Return to intended final state A → B

1. Update Zammad to trust public **B** and save.
2. Reactivate **B** in Entra; confirm **B Active, A Inactive**.
3. Repeat a fresh SSO login and application check. The test was successful; the supplied screenshot set does **not** contain a dedicated final successful sign-in-log event.
4. Preserve A as an Inactive rollback option for the lab. Schedule eventual certificate retirement separately under change control.

## 6. Failure response and operational improvements

| Symptom / risk | IAM–PKI response |
| --- | --- |
| Entra login works, Zammad rejects SSO | Verify Zammad's trusted public certificate corresponds to **Active** Entra signer. Restore last known-good pair. |
| Existing Zammad page works after rollover | Force **new SAML authentication**; a cached application session isn't evidence. |
| Local administrator recovery unavailable | Stop before cutover. Establish approved non-SSO recovery route first. |
| Certificate expiry not monitored | Confirm owner, notification recipients, certificate register and alert escalation. Verify alert delivery in production. |
| Multiple SPs depend on one IdP signing certificate | Inventory all SP dependencies; coordinate per-app trust updates, staged certs and maintenance windows. |

**Production next steps:** CA/PKI standards alignment, approval and separation of duties, enterprise expiry inventory, alert verification, automated metadata ingestion where available, change windows and incident escalation, and documented certificate retirement.

## Public evidence and limitations

See [Evidence index](../Evidence/README.md). Preserve only redacted screenshots in the repository. This exercise did not include private-key escrow, HSM management, CA deployment, automated rollover, a live outage, a measured zero-downtime change, or tests of expiration-email delivery.

### References

- [Microsoft Entra: Manage federation certificates](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/tutorial-manage-certificates-for-federated-single-sign-on)
- [Microsoft Entra: Certificate signing options](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/certificate-signing-options)
- [Zammad: SAML authentication settings](https://admin-docs.zammad.org/en/latest/settings/security/third-party/saml.html)
