# Project Architecture Summary

## Overview

Helm chart producing Karpenter EC2NodeClass and NodePool CRDs for EC2 instances with NVMe instance store drives. A userData script bootstraps LVM from discovered NVMe devices for consumption by OpenEBS LVM LocalPV.

## Core Components

- **EC2NodeClass** (`nvme`) — AMI, subnet/SG selectors, root EBS config, userData bootstrap script.
- **NodePool** (`storage`) — Scheduling constraints: capacity type (spot/on-demand), architecture, minimum NVMe size, resource limits, disruption policy.
- **Helpers** — Shared Helm labels applied to all resources.

## userData Bootstrap

Multipart MIME userData installs LVM2, loads dm_thin_pool, discovers NVMe instance store devices by sysfs model string, and assembles them into a volume group. No thin pool is pre-created — OpenEBS handles that dynamically.

## Cross-Chart Dependencies

Volume group name (`tank` default) must match across karpenter-nvme, helm-openebs-lvm, and helm-storage-consolidator. Coordination is by convention, not Helm dependency.

## CI/CD

PR lint via `helm lint` + `helm template`. Tag-triggered release packages and pushes to `oci://ghcr.io/noizu-labs`.

## Key Decisions

- LVM over raw block for thin provisioning flexibility.
- No thin pool in userData to avoid OpenEBS capacity misreporting.
- Spot-first capacity ordering for cost optimization.
- sysfs model string filtering to distinguish instance store from EBS NVMe devices.
