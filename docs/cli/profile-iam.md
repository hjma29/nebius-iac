---
title: Profiles & IAM
description: Nebius CLI profile setup and IAM (tenants, projects) commands.
---

# Profiles & IAM

## Create a profile

```bash
nebius profile create
```

Walks through browser-based (federation) login and writes the result to
`~/.nebius/config.yaml`:

```yaml
default: profile-01
profiles:
    profile-01:
        endpoint: api.nebius.cloud
        auth-type: federation
        federation-endpoint: auth.nebius.com
        parent-id: project-u00rpx09pr003ap0ehc79j
```

## Inspect the active profile

```bash
nebius profile list      # all configured profiles
nebius profile active     # name of the active profile
nebius profile current    # active profile's full config
```

There's no `nebius profile describe` — full details come from
`nebius profile current` or by reading `~/.nebius/config.yaml` directly.

## List projects under a tenant

```bash
nebius iam project list --parent-id tenant-e00n8mpcqrjw1ehff0 --format table
```

```text
┌────────────────────────────────┬───────────────────────────┬─────────────────────────────┬─────────────────────────────┬─────────────────────────────┬─────────────────┐
│ METADATA.ID                    │ METADATA.PARENT_ID        │ METADATA.NAME               │ METADATA.CREATED_AT         │ METADATA.UPDATED_AT         │ METADATA.LABELS │
├────────────────────────────────┼───────────────────────────┼─────────────────────────────┼─────────────────────────────┼─────────────────────────────┼─────────────────┤
│ project-e00wtv1zpr00acf65jwg96 │ tenant-e00n8mpcqrjw1ehff0 │ default-project-eu-north1   │ 2026-08-09T15:00:08.457281Z │ 2026-08-11T23:07:47.061255Z │ {}              │
│ project-e01eekr0pr003k07q8fe8e │ tenant-e00n8mpcqrjw1ehff0 │ default-project-eu-west1    │ 2026-08-09T15:00:08.526677Z │ 2026-08-11T23:07:59.295255Z │ {}              │
│ project-e03x00bnpr00bftv17fx45 │ tenant-e00n8mpcqrjw1ehff0 │ default-project-uk-south1   │ 2026-08-09T15:00:08.515105Z │ 2026-08-11T23:07:54.841243Z │ {}              │
│ project-i00hgpq9pr001tqbpzbb0d │ tenant-e00n8mpcqrjw1ehff0 │ default-project-me-west1    │ 2026-08-09T15:00:08.628976Z │ 2026-08-11T23:07:54.296294Z │ {}              │
│ project-u00hgr6rpr00t4x43snme7 │ tenant-e00n8mpcqrjw1ehff0 │ default-project-us-central1 │ 2026-08-09T15:00:08.720020Z │ 2026-08-11T23:07:57.163793Z │ {}              │
│ project-u00rpx09pr003ap0ehc79j │ tenant-e00n8mpcqrjw1ehff0 │ hj-tenant1-project1         │ 2026-08-11T23:08:15.303078Z │ 2026-08-11T23:08:30.305830Z │ {}              │
└────────────────────────────────┴───────────────────────────┴─────────────────────────────┴─────────────────────────────┴─────────────────────────────┴─────────────────┘
```

`--format table` is a built-in CLI output formatter (`yaml|json|jsonpath|table|text`).

### Get a single project by ID

```bash
nebius iam project get --id project-u00rpx09pr003ap0ehc79j
```

### Pick specific columns with `jsonpath`

```bash
nebius iam project list --parent-id tenant-e00n8mpcqrjw1ehff0 \
  --format 'jsonpath={range .items[*]}{.metadata.name}{"\t"}{.metadata.id}{"\n"}{end}'
```

### Get an OAuth access token

```bash
nebius iam get-access-token
```

Prints a short-lived bearer token to stdout (e.g. for use as an `Authorization: Bearer <token>`
header when calling Nebius APIs directly). **Never commit or share the actual token value** — it's
a live credential, not a static ID like a profile or project name.
