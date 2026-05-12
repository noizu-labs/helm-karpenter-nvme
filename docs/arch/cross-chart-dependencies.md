# Cross-Chart Dependencies

## Volume Group Contract

The LVM volume group name is the integration point between three independent Helm charts:

| Chart | Value | Must Match |
|-------|-------|------------|
| `karpenter-nvme` | `.volumeGroup` (default `tank`) | VG created by userData |
| `helm-openebs-lvm` | `.storageClass.volumeGroup` | VG OpenEBS provisions into |
| `helm-storage-consolidator` | `.storageClass` | StorageClass referencing the VG |

All three must use the same volume group name. A mismatch causes PVCs to pend indefinitely — OpenEBS cannot find the VG that userData created under a different name.

## Dependency Direction

```
karpenter-nvme (creates VG on node)
       |
       v
helm-openebs-lvm (provisions PVs from VG)
       |
       v
helm-storage-consolidator (manages StorageClass)
       |
       v
Workload PVCs
```

These charts are deployed independently (no Helm dependency declarations). Coordination is by convention — the `volumeGroup` value is the contract.

## Karpenter Controller

This chart creates CRD instances (`EC2NodeClass`, `NodePool`) that require Karpenter v1.0+ to be running in the cluster. The chart does not declare a Helm dependency on the Karpenter chart — it assumes the controller and CRDs are pre-installed.

## Infrastructure Requirements

- Subnets must be tagged `karpenter.sh/discovery: <clusterName>` for EC2NodeClass subnet selection.
- A security group must be tagged `Name: <securityGroupTag>` for node networking.
- An IAM role (`nodeRole`) must exist with EC2/EKS node policies.
