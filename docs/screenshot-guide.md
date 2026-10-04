# Screenshot Guide

The repository uses only the strongest screenshots from the original lab capture set. The final README intentionally avoids showing every intermediate screen.

## Foundation
- `01-resource-group.png` — Production resource group
- `02-production-vnet.png` — Production VNet configuration
- `03-vm-deployment-success.png` — Successful source VM deployment
- `04-production-vm-overview.png` — Source VM overview
- `05-production-vm-security.png` — Trusted Launch, Secure Boot, vTPM
- `06-ssh-validation.png` — Linux/SSH validation

## Recovery vault
- `01-recovery-vault-review-central-us.png` — Corrected cross-region vault design
- `02-recovery-vault-overview.png` — Final Recovery Services vault
- `03-site-recovery-setup.png` — Azure VM Site Recovery workflow

## Replication
- `01-replication-settings.png` — Target region/network configuration
- `02-replication-policy-management.png` — Policy and ASR update management
- `03-replication-review.png` — Final source/target configuration review
- `04-enable-replication-success.png` — Successful enable-replication job
- `05-replication-health-4min-rpo.png` — Healthy protection with 4-minute RPO

## Troubleshooting
- `01-asr-kernel-compatibility-error.png` — Root-cause kernel compatibility error
- `02-kernel-inventory.png` — Installed kernel inventory
- `03-supported-kernel-package-check.png` — Package availability check
- `04-grub-default-6-14.png` — GRUB configured for supported kernel
- `05-running-kernel-6-14.png` — Active supported kernel after reboot

## Test failover
- `01-test-failover-in-progress.png` — DR test in progress
- `02-test-failover-succeeded-21min.png` — Successful test failover and measured duration
- `03-recovered-test-vm-central-us.png` — Recovered test VM running in Central US
- `04-recovered-vm-security-disk.png` — Recovered security/disk configuration
- `05-test-failover-cleanup.png` — Test cleanup documentation

## Validation
- `01-final-dr-validation.png` — Final healthy protected state
- `02-cross-region-infrastructure-view.png` — Azure Site Recovery infrastructure view

## Cost
- `01-cost-analysis-1-14.png` — Cost Management review

## Before publishing
Review screenshots for personally identifying information. Azure subscription IDs are not passwords, but they are unnecessary in a public portfolio. Crop or redact email addresses, subscription IDs, and any other account-specific details before publishing the repository publicly.
