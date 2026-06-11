# FabMo Def (Definitions & Recovery)

## Project Overview
`fabmo-def` is a tiny, **data-only** git repository that lives **outside `/opt`** so it survives an OS reinstall, an engine update, or `/opt/fabmo` being wiped. It serves two roles for the FabMo Engine:

1. **First-boot auto-profile** — `fabmo-def.json` tells the engine which machine profile (or snapshot) to apply automatically on startup, so a freshly imaged device configures itself with no user interaction.
2. **Persistent recovery store** — `snapshots/` (created on demand) mirrors user-blessed "default" snapshots so config/macros can be recovered even after `/opt` is destroyed.

There is **no code, no build, and no dependencies here** — it is consumed by the engine in `/fabmo`. "def" = definition / default.

> **Sibling repos**: Part of a three-repo system. See `/fabmo/doc/system-architecture.md` for the full deployment story, and the `CLAUDE.md` files in `/fabmo` (the engine) and `/fabmo-updater` (the online updater).

## Contents
```
/fabmo-def/
├── fabmo-def.json     # Auto-profile definition (the one meaningful file)
├── README.md          # Human docs + list of valid profile names
└── snapshots/         # (created on demand) mirrors of user-default snapshots
```

## `fabmo-def.json`
Read by the engine on startup. Shape:
```json
{
  "auto_profile": {
    "enabled": true,
    "profile_name": "fabmo-profile-dt",
    "apply_once": true,
    "force_reapply": false
  },
  "created": "2025-01-15T10:30:00Z",
  "description": "Auto-configure for DT on first boot",
  "owner": "ShopBot customer"
}
```
- **`auto_profile.enabled`** — master switch for auto-application.
- **`profile_name`** — full profile id; one of `fabmo-profile-dt`, `fabmo-profile-dtmax`, `fabmo-profile-dtatc`, `fabmo-profile-handibot-2`, or `default`. (Confirm the current list against `README.md` and `/fabmo/profiles/`.)
- **`snapshot_name`** *(optional)* — restore a named snapshot instead of a profile. At least one of `profile_name` / `snapshot_name` must be present.
- **`apply_once`** — apply only on first boot; a marker prevents re-application thereafter.
- **`force_reapply`** — re-apply even if already applied.
- `created` / `description` / `owner` — metadata only.

Unknown top-level keys are preserved on write — treat the schema as extensible and don't drop fields you don't recognize.

## How the Engine Uses This Repo
All consuming logic lives in `/fabmo`, not here:
- **`/fabmo/config/profile_definition.js`** — `ProfileDefinition` singleton reads/writes `/fabmo-def/fabmo-def.json` (cached ~5s, validated, atomic writes that preserve unknown keys).
- **`/fabmo/engine.js`** — on startup checks the auto-profile; if enabled and not yet applied, applies the profile and restarts (a "double boot"). Applied state is recorded in `/opt/fabmo/config/.auto_profile_applied` (`in_progress` flag during the transition).
- **`/fabmo/snapshots.js`** — when a user marks a snapshot as the default, it is mirrored into `/fabmo-def/snapshots/<name>/` (config + macros + `snapshot_info.json`; excludes runtime-only `instance.json` and `auth_secret`). On recovery this is preferred over generic profiles.

Why outside `/opt`: `/opt/fabmo` is the mutable runtime dir and can be wiped by updates/factory reset. Keeping the definition and the blessed snapshots here makes unattended provisioning and disaster recovery possible on remote machines.

## Conventions
- **Edit by hand or via the engine, then commit.** No scripts, no `npm`. Changes are tracked through git.
- **Timestamps**: ISO 8601 (`2025-01-15T10:30:00Z`).
- **Snapshot names**: `[a-zA-Z0-9_-]{1,25}`; reserved auto prefixes `auto_pre_profile_*` and `auto_pre_restore_*` (created by the engine before profile changes / restores — don't author these manually).
- **Profile names** use the `fabmo-profile-<variant>` convention.

## Key Files Reference
- `fabmo-def.json` — the auto-profile directive read on every engine boot
- `README.md` — usage notes + authoritative list of valid profile names
- `snapshots/` — recovery mirrors of user-default snapshots (engine-managed)
- (in the engine) `/fabmo/config/profile_definition.js`, `/fabmo/snapshots.js`, `/fabmo/engine.js` — the code that reads/writes this repo
