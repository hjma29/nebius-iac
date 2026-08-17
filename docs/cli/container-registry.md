---
title: Container Registry
description: Copying container images between Nebius Container Registry regions with skopeo.
---

# Container Registry

Nebius Container Registry (`cr.<region>.nebius.cloud`) is regional — an image
pulled by a workload in one region (e.g. `us-central1`) should be copied into
that region's registry first, rather than pulling cross-region on every node
boot. [`skopeo`](https://github.com/containers/skopeo) copies images
directly registry-to-registry without needing a local Docker daemon.

## Copy an image between registries

```bash
skopeo copy --all \
  docker://cr.eu-north1.nebius.cloud/nebius-benchmarks/nccl-tests:2.23.4-ubu22.04-cu12.4 \
  docker://cr.us-central1.nebius.cloud/u00btcj9x0cvd9sghc/nccl-tests:2.23.4-ubu22.04-cu12.4
```

- `--all` copies every platform variant in a multi-arch manifest list, not
  just the one matching the local host.
- The destination path (`u00btcj9x0cvd9sghc`) is the target project's
  container registry ID, not a literal repository name.

Example output (blob-copy lines are repeated many times for large layers —
condensed here):

```text
Getting image source signatures
Copying blob 7021d1b70935 done   |
Copying blob 0d6448aff889 done   |
... (many more "Copying blob <digest> done" lines, one per layer) ...
Copying blob 56dc85502937 done   |
Copying blob 0a7674e3e8fe done   |
Copying blob ec6d5f6c9ed9 done   |
Copying blob b71b637b97c5 done   |
Copying blob 47b8539d532f done   |
Copying blob fd9cc1ad8dee done   |
Copying blob 83525caeeb35 done   |
Copying blob 8e79813a7b9d done   |
Copying blob 312a542960e3 done   |
Copying blob 9da66fc24710 done   |
Copying blob eb2f93fcfbf9 done   |
Copying blob c8970155eb5e done   |
Copying blob 5e2ec5c313f6 done   |
Copying blob abc28f2bfdc1 done   |
Copying blob bc40c7e7477f done   |
Copying blob 33e31b36644f done   |
Copying blob 1df3a14347bb done   |
Copying blob 255e77d6951a done   |
Copying blob c50d151061a5 done   |
Copying blob bffb03ce0424 done   |
Copying blob ecbd8513c0a1 done   |
Copying blob 46a62b5137e1 done   |
Copying blob 092500c5d6b0 done   |
Copying blob ddb041633fc0 done   |
Copying blob 8994716854c2 done   |
Copying blob bfb47cd44e61 done   |
Copying config ad14ff8369 done   |
Writing manifest to image destination
```

## List registries in a project

```bash
nebius registry list
```

```yaml
items:
  - metadata:
      id: registry-u00btcj9x0cvd9sghc
      parent_id: project-u00rpx09pr003ap0ehc79j
      name: hj-contrainer-registry-01
      resource_version: "1"
      created_at: "2026-08-16T21:48:18.069042Z"
      updated_at: "2026-08-16T21:48:19.901293Z"
    spec:
      images_count: 1
    status:
      state: ACTIVE
      images_count: 1
      registry_fqdn: cr.us-central1.nebius.cloud
```

`registry_fqdn` is the actual host to push/pull against
(`cr.<region>.nebius.cloud`) — the `registry-...` `id` is only used for
CLI/API calls like the ones below, not for image references.

## List images in a registry

```bash
nebius registry image list --parent-id registry-u00btcj9x0cvd9sghc
```

```yaml
items:
  - id: artifact-u00pzj2rj6p99a8ws8
    name: u00btcj9x0cvd9sghc/nccl-tests
    media_type: application/vnd.docker.distribution.manifest.v2+json
    digest: sha256:ac690d7e792f9baed20c079e664fee097ec0f9b872a1ac6e4f6ecb4c9c667381
    size: "6388334302"
    status: ACTIVE
    type: MANIFEST
    created_at: "2026-08-17T00:35:34.821893Z"
    updated_at: "2026-08-17T00:35:34.821893Z"
    tags:
      - 2.23.4-ubu22.04-cu12.4
```

Confirms the `skopeo copy` above landed correctly — one `MANIFEST` artifact
tagged `2.23.4-ubu22.04-cu12.4`, matching the destination image reference.
