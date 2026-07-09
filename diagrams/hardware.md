# Hardware Investigation

```mermaid
flowchart TD

A[Missing NVMe]

--> B[Compare Healthy Worker]

--> C[Compare PCI Devices]

--> D{PCI Missing?}

D -->|Yes| E[Hardware]

D -->|No| F[OS Investigation]

E --> G[iDRAC]

G --> H[SDI]

H --> I[Vendor]
```
