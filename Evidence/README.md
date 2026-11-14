# Evidence Index — PKI Certificate Lifecycle for IAM SSO

| Folder | Screenshot | What it establishes | Evidence limitation |
| --- | --- | --- | --- |
| `01-Baseline` | `PKI-Certificate-Baseline.png` | Original Entra signing certificate Active, expiration November 2029, Sign SAML assertion / SHA-256, notification addresses configured (redacted) | Thumbprint is intentionally redacted |
| `01-Baseline` | `PKI-Zammad-Trust-Baseline.png` | Working SAML configuration with IdP Certificate field populated and SSL verification enabled | Certificate body and URLs redacted; not independent proof of matching thumbprint |
| `01-Baseline` | `PKI-SSO-Baseline-Success.png` | Successful authentication step and timestamp before rotation | The crop does not show user or application, so it is supplementary baseline evidence |
| `02-Preparation` | `PKI-Certificate-Staged-Replacement.png` | Original A Active; replacement B Inactive | Thumbprints redacted |
| `03-Rollover` | `PKI-Certificate-Rollover-Activated.png` | Replacement B Active; original A Inactive | Does not independently prove SP trusts B |
| `03-Rollover` | `PKI-SSO-Post-Rotation-Success.png` | Jordan successful sign-in for CHS – Clinic Integration Support Desk following rotation | Timestamp not in this crop; use with activated-cert evidence |
| `03-Rollover` | `PKI-Zammad-Post-Rotation-Access.png` | Test employee reaches Zammad ticket dashboard | Demo ticket number and title visible—publish only if this is fictional lab ticket data |
| `04-Rollback-Recovery` | `PKI-Rollback-Original-Restored.png` | Original A Active again, B Inactive | Certificate status proves rollback configuration, not fresh SSO alone |
| `04-Rollback-Recovery` | `PKI-Certificate-Final-State.png` | Replacement B Active, A Inactive; notifications show successful activation operations | No dedicated final login result in this image |
| `04-Rollback-Recovery` | `PKI-Zammad-Config-Reload-Notice.png` | Zammad prompts for page reload after a configuration change | **Not evidence of fresh authentication**. Renamed from `PKI-SSO-Final-Verification.png` to avoid a misleading claim |