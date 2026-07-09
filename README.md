# Ceph OSD Troubleshooting Guide

A practical troubleshooting guide for diagnosing Ceph OSD issues in Kubernetes and Rook-Ceph environments.

This guide focuses on common production symptoms such as:

- OSD down
- Rook-Ceph OSD pod CrashLoopBackOff
- Activate container failure
- Missing disk or BlueStore metadata
- PCIe or NVMe detection issues
- Disk and hardware escalation

---

## Troubleshooting Flow

```mermaid
flowchart TD
    A[CCD Alarm Triggered] --> B[Check Ceph Health]
    B --> C[Check OSD Tree]
    C --> D{Is OSD Down?}

    D -->|No| E[Check Deployment and Pod Status]
    D -->|Yes| F[Check OSD Pod]

    F --> G{Pod Status}
    G -->|Running| H[Check Ceph Health Detail]
    G -->|CrashLoopBackOff| I[Check Failing Container]
    G -->|Pending| J[Check Scheduling and Node Status]

    I --> K{Failing Container}
    K -->|activate| L[Check Activate Logs]
    K -->|expand-bluefs| M[Check BlueFS Logs]
    K -->|osd| N[Check OSD Logs]

    L --> O{Disk Found?}
    O -->|Yes| P[Check BlueStore Metadata]
    O -->|No| Q[Check Linux Disk Visibility]

    Q --> R[Check lsblk and blkid]
    R --> S[Check PCIe NVMe Devices]
    S --> T{Device Missing?}

    T -->|Yes| U[Escalate to Hardware or SDI Team]
    T -->|No| V[Investigate Rook/Ceph Metadata]
```

---

## Quick Triage Commands

### Ceph Health

```bash
ceph -s
ceph health detail
ceph osd tree
ceph osd status
ceph osd df tree
```

### Kubernetes Pod Check

```bash
kubectl get pods -n rook-ceph | grep osd
kubectl describe pod <pod-name> -n rook-ceph
kubectl logs <pod-name> -n rook-ceph -c activate --previous
```

### Linux Storage Check

```bash
lsblk
lsblk -f
blkid
lspci | grep -i "Non-Volatile"
dmesg | grep -Ei "nvme|pci|error|timeout|reset"
```

---

## Decision Tree: Activate Container Failure

```mermaid
flowchart TD
    A[Activate Container Fails] --> B[Check activate logs]

    B --> C{Error Message}
    C -->|no disk found with OSD ID| D[Check ceph-volume raw list]
    C -->|permission or config error| E[Check Rook Config and Secret]
    C -->|BlueStore error| F[Check BlueStore Label]

    D --> G{OSD ID visible?}
    G -->|Yes| H[Check device path mismatch]
    G -->|No| I[Check disk visibility from OS]

    I --> J[Run lsblk and blkid]
    J --> K{Disk visible?}

    K -->|Yes| L[Check BlueStore metadata]
    K -->|No| M[Check PCIe/NVMe controller]

    M --> N[Compare lspci with healthy node]
    N --> O{PCIe device missing?}

    O -->|Yes| P[Likely hardware or PCIe issue]
    O -->|No| Q[Investigate OS driver or namespace issue]
```

---

## Example Case Study

### Symptom

CCD alarm reported:

```text
Deployment rook-ceph-osd-21 in namespace rook-ceph is running less than desired replicas 0/1.
```

### Findings

- OSD 21 was down.
- The OSD pod failed during the activate init container.
- Activate logs showed:

```text
no disk found with OSD ID 21
```

- `ceph-volume raw list` detected only other OSDs.
- The expected NVMe device was not visible from the OS.
- PCIe comparison with a healthy worker showed one missing NVMe controller.

### Conclusion

The issue was not resolved by restarting the pod. The evidence pointed toward a disk, PCIe, or hardware-level detection issue.

---

## Important Notes

- Do not manually edit Rook-generated OSD deployments unless advised.
- Restarting the pod only helps if the disk and metadata are still available.
- If the activate container cannot find the OSD disk, always verify Linux disk visibility and PCIe detection.
- Compare with a healthy node whenever possible.
- Escalate to the hardware or SDI team with evidence from Ceph, Kubernetes, Linux, and PCIe checks.
