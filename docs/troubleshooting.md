# Troubleshooting Log

## 1. Recovery Services vault placed in the source region

### Symptom
Azure Site Recovery warned that the Recovery Services vault and source workload were in the same region, which did not support the intended cross-region DR design.

### Resolution
A new Recovery Services vault was created in **Central US**, while the source VM remained in **East US**.

### Lesson
Recovery-vault placement is part of the DR architecture and should be planned before enabling protection.

---

## 2. Enable replication failed with Replication Provider error

### High-level error
- Error ID: `539`
- Message: The requested action could not be performed by the Replication Provider.

### Root cause
Detailed Site Recovery job information exposed error `151141`. The ASR Mobility Service version did not support the source VM's running kernel:

```text
7.0.0-1014-azure
```

### Investigation
Installed kernels were enumerated with:

```bash
dpkg --list | grep linux-image
```

A supported 6.14 Azure kernel family was then installed.

### Remediation

```bash
sudo apt install linux-azure-6.14
```

GRUB was configured to boot the supported kernel by default, then rebuilt:

```bash
sudo update-grub
```

After reboot, the active kernel was validated:

```bash
uname -r
# 6.14.0-1017-azure
```

The failed ASR enable-replication job was restarted and completed successfully.

### Lesson
Do not stop at the top-level cloud error. Trace the failure through the detailed job stages until the underlying guest or platform issue is identified.

---

## 3. Replication warning after source VM shutdown

### Symptom
Replication health changed to warning because the source VM was powered off.

### Resolution
The source VM was powered back on. Azure Site Recovery resumed replication and performed resynchronization automatically.

### Final state
- Replication Health: Healthy
- Status: Protected
- RPO: 4 minutes
- Errors: 0
