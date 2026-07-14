# Azure DevOps Pipelines

Azure DevOps YAML pipelines for operational automation, deployment, monitoring, and compliance across enterprise Windows and Azure environments. Distilled from pipelines used in production. Server names, paths, and service-connection names are sanitised, and secrets are referenced from variable groups rather than committed. Adapt the variables for your own Azure DevOps organisation.

The pipelines share a consistent pattern: parameterised target environments, a WhatIf preview mode for destructive operations, secrets held in variable groups, self-hosted Windows agents pinned to the target host, and pipeline artifacts published for audit.

## Pipelines

### deployment/multi-server-dotnet-update.yml
Multi-stage, gated deployment of .NET runtime (hosting bundle) updates across DEV, STAGING, and PRODUCTION. Manual trigger only. Three stages:
- **Pre-Flight:** runs the update in WhatIf mode against the target server list and publishes a validation report before any change is made.
- **Deploy:** an Azure DevOps deployment job bound to an environment resource (approval-gated for staging and production), executed only when WhatIf is disabled.
- **Post-Deploy:** re-runs the .NET vulnerability audit to confirm the installed runtime versions and publishes the verification.

Per-environment server lists keep target isolation explicit.

### compliance/atlassian-license-audit.yml
Scheduled audit of Atlassian (Jira/Confluence) licence consumption against Active Directory group membership, reporting inactive or unentitled accounts for licence reclamation.

### monitoring/ssl-certificate-monitor.yml
Scheduled SSL/TLS certificate expiry monitoring with threshold-based alerting, flagging certificates approaching expiry before they lapse.

### monitoring/vulnerability-scan.yml
Scheduled vulnerability scan of installed .NET and Java runtimes across the server estate, reporting hosts running versions with known vulnerabilities.

### tableau-ops/tableau-backup.yml
Scheduled daily Tableau Server backup with retention cleanup and email notification. Parameterised per environment, WhatIf mode, self-hosted agent pinned to the primary node, 180-minute timeout.

### tableau-ops/tableau-cleanup.yml
Scheduled daily TSM maintenance (ziplog generation and log-retention cleanup across primary and secondary nodes) with before/after disk-space checks and email alerting. Runs ahead of the backup window.

### tableau-ops/tableau-pdf-export.yml
Scheduled export of Tableau views to PDF for distribution.

## Patterns used

- **Parameterised environments:** one pipeline targets DEV/STAGING/PRODUCTION via parameters, per-environment variable groups, and per-environment server lists.
- **WhatIf / preview mode:** destructive operations support a dry-run pass that reports intended changes without applying them.
- **Approval gates:** deployment jobs bind to Azure DevOps environment resources, so staging and production require sign-off.
- **Pre- and post-validation:** deployments validate targets before running and verify state afterward.
- **Secrets via variable groups:** server names, credentials, and SMTP settings come from variable groups (ideally Key Vault-backed), never committed to source.
- **Self-hosted agents:** pinned to the target host via agent demands.
- **Artifact publishing:** each run publishes logs or reports for an audit trail.

## Prerequisites

- An Azure DevOps organisation and project
- A self-hosted Windows agent pool with access to the target servers
- Environment resources configured with approval checks for staging and production
- Variable groups (ideally Key Vault-backed) holding server names, paths, credentials, and SMTP settings
- The referenced PowerShell scripts (see the powershell-automation repository) checked out alongside these pipelines

## Note
These are reference pipelines, not a turnkey deployment. Review the parameters and variable-group references at the top of each file and point them at your own environment before running.
