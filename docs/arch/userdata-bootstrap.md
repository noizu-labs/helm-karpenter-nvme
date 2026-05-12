# userData Bootstrap

## Sequence

The EC2NodeClass userData is a multipart MIME document with two parts:

1. **NodeConfig** (`application/node.eks.aws`) — EKS node bootstrap configuration (minimal; default kubelet settings).
2. **Shell script** (`text/x-shellscript`) — LVM setup executed once at instance launch.

## Shell Script Steps

1. `dnf install -y -q lvm2` — installs LVM2 from AL2023 repos.
2. `modprobe dm_thin_pool` + persistence via `/etc/modules-load.d/lvm-thin.conf` — loads the thin provisioning kernel module required by OpenEBS.
3. Iterates `/sys/block/nvme*n1`, reads `device/model`, and collects devices whose model contains `Instance Storage`.
4. Runs `pvcreate -f` on each discovered device.
5. Runs `vgcreate <volumeGroup>` spanning all devices.
6. Exits cleanly (exit 0) if no NVMe instance store devices are found — the node still joins the cluster but has no local storage VG.

## Design Notes

- Device discovery uses sysfs model strings, not device name patterns (`/dev/nvme*`), because EBS volumes also appear as NVMe devices on Nitro instances.
- No thin pool is created in userData. OpenEBS LVM LocalPV manages thin pool lifecycle per-PVC when `thinProvision: "yes"` is set in the StorageClass. Pre-creating a thin pool causes OpenEBS to misreport available capacity.
- The script is idempotent in practice: Karpenter provisions fresh instances, so the script always runs on a clean disk state.
