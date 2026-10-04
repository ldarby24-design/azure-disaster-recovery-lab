# Azure Cross-Region Disaster Recovery Lab

## Overview

This project documents the design, deployment, troubleshooting, and validation of a cross-region disaster recovery environment in Microsoft Azure using Azure Site Recovery (ASR).

The lab protects an Ubuntu 24.04 Azure virtual machine running in **East US** and replicates it to **Central US**. The project was designed around explicit recovery objectives, tested with a non-disruptive test failover, and validated against observed recovery performance.

## Project Goals

- Define measurable disaster recovery objectives before implementing technology.
- Deploy and validate a production-style Azure VM and network.
- Configure Azure Site Recovery for cross-region replication.
- Build an isolated test network for DR validation.
- Troubleshoot real replication failures instead of rebuilding around them.
- Validate recovery time and recovery-point performance.
- Clean up test resources while leaving protection healthy.
- Review Azure cost after the lab.

## Recovery Objectives

| Objective | Target | Observed Result |
|---|---:|---:|
| Recovery Time Objective (RTO) | 1 hour | Test failover completed in **21 min 02 sec** |
| Recovery Point Objective (RPO) | 15 minutes | Observed replication RPO: **4 minutes** |

The observed test results met both lab objectives.

## Architecture

```mermaid
flowchart LR
    A[VM-DRApp01\nEast US / Zone 1] --> B[ASR Cache Storage]
    B --> C[Azure Site Recovery]
    C --> D[Replicated Managed Disk\nCentral US]
    C --> E[VNET-DRLab-Prod-asr\nRecovery Network]
    C --> F[VNET-DRLab-Test\nIsolated Test Network]
    F --> G[VM-DRApp01-test\nCentral US / Zone 1]
```

### Key Resources

| Resource | Purpose |
|---|---|
| `VM-DRApp01` | Source Ubuntu 24.04 workload |
| `RG-DRLab-Prod` | Production resource group |
| `VNET-DRLab-Prod` | Production virtual network |
| `SNET-App` | Production application subnet |
| `RSV-DRLab-CentralUS` | Recovery Services vault |
| `RG-DRLab-Recovery-CentralUS` | Recovery-side management resources |
| `RG-DRLab-Prod-asr` | ASR-created recovery resource group |
| `VNET-DRLab-Prod-asr` | Recovery virtual network |
| `VNET-DRLab-Test` | Isolated test-failover network |

## 1. Production Environment

The source workload was deployed as an Ubuntu 24.04 LTS VM in East US with Trusted Launch, Secure Boot, vTPM, SSH key authentication, and a private address on the application subnet.

![Production VM overview](screenshots/01-foundation/04-production-vm-overview.png)

The VM security configuration included Trusted Launch, Secure Boot, and vTPM.

![Production VM security](screenshots/01-foundation/05-production-vm-security.png)

SSH connectivity and Linux network settings were validated before configuring DR.

![SSH validation](screenshots/01-foundation/06-ssh-validation.png)

## 2. Recovery Services Vault

The first vault design was corrected after Azure identified that cross-region DR requires the recovery vault to be outside the source region. The final vault was created in **Central US** to support East US to Central US recovery.

![Recovery vault review](screenshots/02-vault/01-recovery-vault-review-central-us.png)

![Recovery vault overview](screenshots/02-vault/02-recovery-vault-overview.png)

Azure Site Recovery was then selected for Azure-to-Azure VM protection.

![Site Recovery setup](screenshots/02-vault/03-site-recovery-setup.png)

## 3. Replication Configuration

The source VM was configured to replicate from **East US** to **Central US**. Azure created recovery-side resources including a target resource group and virtual network.

![Replication settings](screenshots/03-replication/01-replication-settings.png)

A 24-hour retention policy was used, and ASR was allowed to manage mobility-service updates automatically.

![Replication policy](screenshots/03-replication/02-replication-policy-management.png)

The final review confirmed the source region, destination region, recovery resource group, recovery VNet, and replication policy.

![Replication review](screenshots/03-replication/03-replication-review.png)

## 4. Troubleshooting: Unsupported Linux Kernel

The initial replication attempt failed during **Installing Mobility Service and preparing target**.

Azure initially surfaced a generic Replication Provider error, but deeper job details revealed the actual issue: the ASR Mobility Service did not support the VM's running kernel, `7.0.0-1014-azure`.

![ASR kernel compatibility error](screenshots/04-troubleshooting/01-asr-kernel-compatibility-error.png)

The VM contained multiple Azure kernels, so the installed versions were reviewed.

![Kernel inventory](screenshots/04-troubleshooting/02-kernel-inventory.png)

A supported 6.14 Azure kernel family was identified and installed. GRUB was then configured to boot `6.14.0-1017-azure` by default.

![GRUB default](screenshots/04-troubleshooting/04-grub-default-6-14.png)

After rebooting, the running kernel was verified:

```bash
uname -r
# 6.14.0-1017-azure
```

![Verified supported kernel](screenshots/04-troubleshooting/05-running-kernel-6-14.png)

The failed replication job was restarted after the kernel change and completed successfully.

![Replication successful](screenshots/03-replication/04-enable-replication-success.png)

### Troubleshooting takeaway

A generic cloud-platform error is not always the root cause. The successful resolution required moving from the high-level ASR error to the detailed protection job, identifying the exact guest-kernel compatibility problem, correcting the OS boot configuration, and retrying the original workflow.

## 5. Replication Health

After protection was enabled, the replicated item reported:

- Replication health: **Healthy**
- Status: **Protected**
- RPO: **4 minutes**
- Configuration issues: **None**
- Agent status: **Healthy**

![Healthy replication](screenshots/03-replication/05-replication-health-4min-rpo.png)

The observed 4-minute RPO was comfortably inside the lab target of 15 minutes.

## 6. Test Failover

A separate test VNet was used to avoid interfering with the production or actual recovery network.

The test failover was launched using the latest processed recovery point for the lowest RTO.

![Test failover in progress](screenshots/05-test-failover/01-test-failover-in-progress.png)

The test failover completed successfully in **21 minutes and 2 seconds**, meeting the 1-hour RTO target.

![Test failover succeeded](screenshots/05-test-failover/02-test-failover-succeeded-21min.png)

## 7. Recovery Validation

Azure created `VM-DRApp01-test` in **Central US** on the isolated test network. The recovered VM was running and received private IP `10.20.1.4`.

![Recovered test VM](screenshots/05-test-failover/03-recovered-test-vm-central-us.png)

The recovered system also retained Trusted Launch, Secure Boot, vTPM, and the replicated OS-disk configuration.

![Recovered VM security and disk](screenshots/05-test-failover/04-recovered-vm-security-disk.png)

## 8. Test Cleanup and Final State

After validation, the test failover was formally cleaned up and the temporary recovery VM was deleted.

![Test failover cleanup](screenshots/05-test-failover/05-test-failover-cleanup.png)

The protected workload returned to a healthy steady state:

- Replication Health: **Healthy**
- Status: **Protected**
- RPO: **4 minutes**
- Last successful test failover recorded
- Errors: **0**

![Final DR validation](screenshots/06-validation/01-final-dr-validation.png)

Azure's infrastructure view shows the East US source VM, cache storage, Azure Site Recovery, and managed recovery disk in Central US.

![Cross-region infrastructure](screenshots/06-validation/02-cross-region-infrastructure-view.png)

## 9. Cost Management

After completing the lab, the source VM was deallocated and Azure Cost Management was reviewed.

Total accumulated cost at the time of review was **$1.14**.

![Azure cost analysis](screenshots/07-cost/01-cost-analysis-1-14.png)

This reinforced an important cloud-operations practice: compute should be deallocated when not needed, while retained disks, networking, replication, and storage resources may continue to generate cost.

## Skills Demonstrated

- Microsoft Azure
- Azure Site Recovery
- Recovery Services Vaults
- Disaster Recovery Planning
- RTO and RPO
- Azure Virtual Machines
- Azure VNets and Subnets
- Cross-region replication
- Test failover and cleanup
- Linux administration
- SSH
- Linux kernel management
- GRUB configuration
- Azure troubleshooting
- Cost Management
- Cloud security controls
- Technical documentation

## Key Lessons Learned

1. **Define RTO and RPO before selecting DR technology.** Recovery design should be driven by business requirements.
2. **Test DR instead of assuming replication equals recoverability.** A healthy replica is not enough; the recovery workload must actually boot and operate.
3. **Isolate test failovers.** A dedicated test network reduces the chance of conflicts with production services.
4. **Read past generic error messages.** The root cause of the ASR failure was only visible in the detailed job error.
5. **Guest OS compatibility matters.** Cloud recovery services still depend on supported operating-system kernels and agents.
6. **DR testing should include cleanup and steady-state verification.** The process is not complete until temporary recovery resources are removed and protection returns to healthy status.
7. **Cost management is part of cloud engineering.** Resources should be deallocated or removed when no longer required.

## Result

The lab successfully demonstrated cross-region disaster recovery for an Azure Ubuntu VM from **East US to Central US**. The final environment achieved an observed **4-minute RPO** and a **21-minute test-failover time**, satisfying the project's target **15-minute RPO** and **1-hour RTO**.
