# Activate Container Failure

---

## Symptoms

```
Back-off restarting failed container activate
```

---

## Investigation

### Describe Pod

```bash
kubectl describe pod <pod-name> -n rook-ceph
```

---

### Read Logs

```bash
kubectl logs <pod-name> \
-c activate \
--previous
```

---

## Common Errors

### Error

```
no disk found with OSD ID
```

Possible causes

- Missing disk
- Missing BlueStore metadata
- Wrong device path
- Hardware issue

---

## Next Step

Proceed to

➡ Linux Storage Investigation
