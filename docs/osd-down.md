# Ceph OSD Down

---

## Overview

This runbook describes the investigation process when a Ceph OSD enters the **down** state.

---

## Symptoms

- OSD reported as DOWN
- Deployment running 0/1
- CrashLoopBackOff
- Storage health degraded

---

## Initial Checks

### Check Ceph Health

```bash
ceph -s
ceph health detail
```

Expected

```
HEALTH_OK
```

---

### Check OSD Status

```bash
ceph osd tree
```

Example

```
host worker05

osd.1 up

osd.7 up

osd.14 up

osd.21 down
```

---

### Check Kubernetes

```bash
kubectl get pods -n rook-ceph
```

---

## Decision

If the OSD pod is **Running**

➡ Proceed to cluster investigation.

If the OSD pod is **CrashLoopBackOff**

➡ Continue with **Activate Container Failure**.
