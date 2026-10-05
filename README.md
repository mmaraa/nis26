# 🔐 nis26

Materials from **The integration architect's guide to hybrid identity lifecycle with Logic Apps**,
presented by [Markus Lintuala](https://linkedin.com/in/markuslintuala) at Nordic Integration Summit 2026. 🇸🇪

Native SCIM provisioning gets you about 60% of the way to a working Day-1 identity. These files are
the other 40%: a deterministic UPN resolver 🎯, the right ordering around proxyAddresses and the sync
clock ⏱️, and correlation on `employeeId` instead of run IDs 🔗.

## 🎤 Slides

- 📊 [2026NIS_MarkusLintuala_IntegrationArchitectsGuideToHybridIdentityLifecycleWithLogicApps.pdf](2026NIS_MarkusLintuala_IntegrationArchitectsGuideToHybridIdentityLifecycleWithLogicApps.pdf)
  — the full session deck: reference architecture, source-of-authority decision (Entra → AD with
  Cloud Sync vs. API-driven into AD), the four gotchas, and the demo walkthrough.

## 🧭 Proper identity creation process

- 🗺️ [IdentityProvisioningProcess.pdf](IdentityProvisioningProcess.pdf) — one-page reference of the
  pattern: HR → Logic Apps → UPN resolver → Entra ID → Cloud Sync → AD → business application,
  with the six principles (single source of authority, deterministic UPN, proxyAddresses before
  license, design for the sync clock, idempotent every step, observable by `employeeId`).

## ⚠️ Gotcha list with fixes

- 🩹 [EntraProvisioningGotchas.pdf](EntraProvisioningGotchas.pdf) — one-page table of four gotchas and
  their fixes: where SCIM stops, Entra → AD sync prerequisites (agent version, `msDS-ExternalDirectoryObjectId`,
  Governance licensing), the extra sync cycle when AD is the source, and cross-system correlation.

## ⚙️ Logic App snippets

Workflow definitions for Logic Apps Standard. Both authenticate with managed identity 🪪; all
environment-specific values are parameters with blank defaults, so fill them in before deploying.

- 📥 [logicapp-ingest.json](logicapp-ingest.json) — receives a SCIM bulk request from the HR system,
  resolves each user's UPN through the Azure Function resolver, enriches the SCIM payload with
  `userName`, work email and `mailNickname`, then forwards it to the Entra inbound provisioning
  `/bulkUpload` endpoint. Tracked properties on every action, keyed by `employeeId`.
  Parameters: `upnDomain`, `bulkUploadUrl`, `upnResolverUrl`, `upnResolverKey` (secure), `eventTag`.
- 🎟️ [logicapp-accesspackage.json](logicapp-accesspackage.json) — called by an Entitlement Management
  custom extension when an access package assignment is granted. Reads `employeeId` from Graph and
  starts an Automation runbook on a Hybrid Runbook Worker to grant access in the on-prem application.
  Parameters: `automationAccountId`, `runbookName`, `hybridWorkerGroup`, `eventTag`.
  Needs `User.Read.All` for the managed identity and Automation Job Operator on the account.

## 🔍 KQL queries

Written for a Log Analytics workspace that collects workspace-based Application Insights, the Entra
diagnostic settings (`ProvisioningLogs`, `AuditLogs`) and the Logic App / Automation diagnostics.

- 🧵 [joiner-timeline.kql](joiner-timeline.kql) — everything that happened to one employee across every
  system, correlated on `employeeId` rather than run ID. Set the `employeeId` variable at the top.
- 📈 [joiners-today.kql](joiners-today.kql) — the dashboard card: how many joiners got a UPN today, how
  many have landed in Entra, how many have landed in AD, and how many are still waiting on sync.
- 🚨 [joiner-failures-alert.kql](joiner-failures-alert.kql) — any failed joiner step in the last 15
  minutes with the fields an on-call card needs. Use as a log search alert, 5-minute frequency,
  threshold > 0.

---

`#NIS26` `#azure` `#identity`
