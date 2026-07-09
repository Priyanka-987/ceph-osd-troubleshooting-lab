# Linux Storage Investigation

```mermaid
flowchart TD

A[Disk Investigation]

--> B[lsblk]

--> C[blkid]

--> D[ceph-volume raw list]

--> E[lspci]

--> F[dmesg]

--> G{iDRAC}

--> H[Hardware Team]
```
