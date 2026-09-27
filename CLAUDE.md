# fabmo-def — notes for Claude

The one repo that is **machine- and user-specific and never overwritten by
updates**. Installed at `/fabmo-def` on every tool. See ~/.claude/CLAUDE.md for
the platform picture; a committed copy of that overview lives here as
`FABMO-DEV-OVERVIEW.md` so it survives re-imaging a dev Pi.

## Overview
`fabmo-def` is a tiny, **data-only** git repository that lives **outside `/opt`** so it survives an OS reinstall, an engine update, or `/opt/fabmo` being wiped. It serves two roles for the FabMo Engine:

1. **First-boot auto-profile** — `fabmo-def.json` tells the engine which machine profile (or snapshot) to apply automatically on startup, so a freshly imaged device configures itself with no user interaction.
2. **Persistent recovery store** — `snapshots/` (created on demand) mirrors user-blessed "default" snapshots so config/macros can be recovered even after `/opt` is destroyed.

There is **no code, no build, and no dependencies here** — it is consumed by the engine in `/fabmo`. "def" = definition / default.

## Contents
- `fabmo-def.json` — startup definition. On a fresh `/opt/fabmo`, the engine
  boots `default`, reads `auto_profile.profile_name` here, then restarts once
  into that profile (`apply_once: true`). Valid names are the full profile
  directory names from the engine (`fabmo-profile-dt`, `-dtmax`, `-dtatc`,
  `-handibot-2`, `default`), not display names. A second reboot may be needed
  for IP-address signalling.
- `snapshots/` — user-saved snapshots of config + macros at a point in time
  (managed by the engine's `snapshots.js`). User data: never delete or
  regenerate.

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

## Conventions
- **Edit by hand or via the engine, then commit.** No scripts, no `npm`. Changes are tracked through git.
- **Timestamps**: ISO 8601 (`2025-01-15T10:30:00Z`).
- **Snapshot names**: `[a-zA-Z0-9_-]{1,25}`; reserved auto prefixes `auto_pre_profile_*` and `auto_pre_restore_*` (created by the engine before profile changes / restores — don't author these manually).
- **Profile names** use the `fabmo-profile-<variant>` convention.
- Keep this repo tiny. It is cloned onto customer machines; every file here
  ships.
- When a new profile is added to the engine, update the README's profile list
  here too.
- Changes here are rare.

## Key Files Reference
- `fabmo-def.json` — the auto-profile directive read on every engine boot
- `README.md` — usage notes + authoritative list of valid profile names
- `snapshots/` — recovery mirrors of user-default snapshots (engine-managed)
- (in the engine) `/fabmo/config/profile_definition.js`, `/fabmo/snapshots.js`, `/fabmo/engine.js` — the code that reads/writes this repo
