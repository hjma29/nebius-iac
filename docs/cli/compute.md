---
title: Compute
description: Listing and deleting Nebius Compute instances via the CLI.
---

# Compute

`nebius compute instance` manages VM instances directly — useful for
inspecting or cleaning up stray nodes (e.g. leftover `mk8s` node-group
instances) outside of Terraform.

## List instances (compact tabular view)

Pipe `--format json` through `jq` for just the columns that matter:

```console
$ nebius compute instance list --parent-id project-u00rpx09pr003ap0ehc79j --format json \
  | jq -r '["NAME","PLATFORM","PRESET","STATUS","DISK_TYPE","DISK_GB"],
      (.items[] | [.metadata.name, .spec.resources.platform, .spec.resources.preset, .status.state,
        .spec.boot_disk.managed_disk.spec.type, .spec.boot_disk.managed_disk.spec.size_gibibytes]) | @tsv' \
  | column -t -s $'\t'
NAME                                          PLATFORM  PRESET      STATUS   DISK_TYPE    DISK_GB
mk8snodegroup-u00he1nwg6gmxkwd4e-smvmm-5hv82  cpu-d3    4vcpu-16gb  STOPPED  NETWORK_SSD  64
```

## List instances (full YAML)

```console
$ nebius compute instance list --parent-id project-u00rpx09pr003ap0ehc79j
items:
  - metadata:
      id: computeinstance-u00h5mf9q0e9rrq0va
      parent_id: project-u00rpx09pr003ap0ehc79j
      name: mk8snodegroup-u00he1nwg6gmxkwd4e-smvmm-5hv82
      resource_version: "3"
      created_at: "2026-08-11T23:56:40.601782Z"
      labels:
        mk8s-cluster-id: mk8scluster-u00aqc4ewh20nr91sx
        mk8s-node-group-id: mk8snodegroup-u00he1nwg6gmxkwd4e
    spec:
      resources:
        platform: cpu-d3
        preset: 4vcpu-16gb
      gpu_cluster: {}
```

The `mk8s-cluster-id`/`mk8s-node-group-id` labels identify instances that
belong to a managed Kubernetes node group — these are normally created and
destroyed by Terraform/the `mk8s` control plane, not deleted by hand, unless
they're stray/orphaned (e.g. left `STOPPED` after a failed teardown).

## Delete an instance

```console
$ nebius compute instance delete --id computeinstance-u00h5mf9q0e9rrq0va
waiting for operation "computeoperation-u00xzssk09htyga0zv" over resource "computeinstance-u00h5mf9q0e9rrq0va" to complete
Operation completed successfully: computeoperation-u00xzssk09htyga0zv
✓ : Deleting instance elapsed 27s
id: computeoperation-u00xzssk09htyga0zv
description: Delete Instance
created_at: "2026-08-17T01:14:40.835028Z"
finished_at: "2026-08-17T01:15:06.957016Z"
resource_id: computeinstance-u00h5mf9q0e9rrq0va
status: {}
progress_tracker:
  description: Deleting instance
  started_at: "2026-08-17T01:14:40.835028Z"
  finished_at: "2026-08-17T01:15:06.957016Z"
```
