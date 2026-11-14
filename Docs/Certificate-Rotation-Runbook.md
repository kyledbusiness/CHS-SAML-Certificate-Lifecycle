# Certificate Rotation Runbook — Entra SAML SSO ↔ Zammad

**Scope:** Microsoft Entra Enterprise Application `CHS – Clinic Integration Support Desk` and Zammad **Authentication via SAML**. This runbook reflects a supervised lab exercise, not an approved production change procedure.

## 1. Preconditions / change control

- Record the Entra application name, change owner, planned window, certificate expiry, and rollback decision point. Obtain change approval for a production environment.
- Ensure an authorized Entra application administrator is available, plus a **working local Zammad administrator account** independent of SAML.
- Validate baseline SSO using a **new private-browser session** (for this lab, the test user was Jordan Lee).
- In **Entra ID → Enterprise applications → [Zammad enterprise app] → Single sign-on → SAML Certificates**, record the **Active** certificate, expiry and thumbprint; verify notification contacts.
- In **Zammad → Settings → Security → Third-party Applications → Authentication via SAML**, confirm the **IdP Certificate** field is populated. The deprecated **fingerprint** field should not also be used as the trust input.
- Download the original public signing certificate and privately preserve the current Zammad IdP certificate text. Save only in access-controlled storage, outside a public repository. Do **not** export private signing keys, tokens, or tenant-specific login URLs to GitHub.

**Evidence:** `PKI-Certificate-Baseline.png`, `PKI-Zammad-Trust-Baseline.png`, `PKI-SSO-Baseline-Success.png` (where captured).

## 2. Prepare an inactive replacement

1. In the Entra SAML Signing Certificate editor, choose **New Certificate** and **Save**.
2. Verify **Certificate A = Active**, **Certificate B = Inactive**. Record validity dates and thumbprints.
3. Download **Certificate B (Base64/public .cer)**; view the complete PEM text (`BEGIN CERTIFICATE` through `END CERTIFICATE`). Ensure no extra characters or lost lines.
4. Confirm you can still authenticate to Zammad as a local administrator without SAML. Do not close that session during the change.
5. Confirm who receives certificate-expiration notifications. Notification configuration is not the same as proof an alert email was delivered.

**Evidence:** `PKI-Certificate-Staged-Replacement.png`, `PKI-Certificate-Expiration-Notifications.png` if captured.

## 3. Coordinated cutover — A to B

**Risk:** Because the lab Zammad configuration accepts one IdP certificate, new SAML logins may briefly fail while Entra and Zammad have mismatched certificate settings. Use a maintenance window in production; this lab did **not** establish zero-downtime rotation.

1. Keep the local Zammad admin session open.
2. Replace Zammad's **IdP Certificate** with the complete public **Certificate B**; save. Keep the fingerprint field unused and SSL verification enabled.
3. In Entra SAML Certificates, set **Certificate B = Active**. Verify **Certificate A = Inactive**. Do not delete A.
4. Launch a **fresh private browser**. Use **CHS Employee SSO** to sign in as the authorized test employee.
5. Verify the application dashboard loads. Correlate the time and user with Entra **Sign-in logs**; check the correct Zammad enterprise application and **Success** status.
6. If the sign-in fails, **stop**. Don't keep randomly activating/deleting certificates; use Section 4.

**Evidence:** `PKI-Certificate-Rollover-Activated.png`, `PKI-SSO-Post-Rotation-Success.png`, optional Zammad dashboard.

## 4. Rollback — B to A

1. Keep or regain **local Zammad administrator access**; do not rely on SAML for recovery.
2. Restore the original public IdP **Certificate A** in Zammad and save.
3. Make **Certificate A = Active** in Entra. Check B becomes Inactive.
4. Sign out of the test application and initiate a **new** SSO login from a private browser. An existing Zammad session or page reload does **not** prove a new SAML assertion succeeded.
5. Confirm Entra shows a successful Zammad application sign-in. Record the observed outcome and any errors.

**Evidence:** `PKI-Rollback-Original-Restored.png` and fresh authentication proof, if captured.

## 5. Return to intended final state — A to B

1. Copy Certificate B into the Zammad IdP Certificate field again; save.
2. Activate B in Entra; confirm **B Active / A Inactive**.
3. Perform a new SSO login and check successful application access plus Entra sign-in result.
4. Retain A as an inactive rollback option until a separate retirement decision, with appropriate change control. Store public certificate backups securely and avoid public upload of tenant-specific configuration.

**Evidence:** `PKI-Certificate-Final-State.png`, `PKI-SSO-Final-Verification.png` if captured.

## 6. Troubleshooting and failure response

| Symptom | Likely question / next action |
| --- | --- |
| Microsoft authentication succeeds but Zammad rejects SSO | Does Zammad's configured public IdP certificate match the signing certificate Active in Entra? Restore matching trust. |
| Existing session works but fresh SSO fails | The app session may be cached. Test a new private-browser sign-in; don't infer certificate health from a refresh. |
| Zammad local admin unavailable | **Stop before rollover.** Recover an approved non-SSO administrator path first. |
| Entra shows success but Zammad rejects or loops | Inspect the application-side SAML error, ACS/Entity ID and time synchronization; correlation may require application logs. |
| No log event immediately visible | Check tenant/application scope, interactive sign-ins and time range; avoid treating filter inconsistency as an authentication failure. |

## 7. Scope, security, and cleanup

- The certificate downloaded from Entra is **public X.509 material**, not the private key. It should still be handled as operational configuration and may identify the tenant/application.
- Don't publish real IdP SSO/logout URLs, private-key material, session IDs, administrator credentials or unredacted screenshots.
- Entra administrative authority and Zammad local admin access should follow least-privilege practices.
- The test verified **application sign-in**, not a continuously monitored zero-downtime rotation.
- Retain the original inactive certificate only as long as the agreed rollback window requires; planned retirement and inventory updates would be separate change steps.

## References

- Microsoft Learn: [Manage federation certificates for SAML single sign-on](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/tutorial-manage-certificates-for-federated-single-sign-on)
- Zammad Admin Docs: [SAML authentication](https://admin-docs.zammad.org/en/latest/settings/security/third-party/saml.html)
