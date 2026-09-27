# FabMo Platform — cross-cutting notes (loaded in every session on this Pi) [this is a backup of `/root/.claude/CLAUDE.md

This Pi is a FabMo development machine. Everything below is true across all the
FabMo repos; each repo has its own CLAUDE.md with specifics.

## What FabMo is

Open-source CNC tool control platform from ShopBot Tools (gofabmo.org). Each
tool runs its own node.js server on a Raspberry Pi 5; users drive the tool from
a browser on any client (PC, tablet, phone). Ships on all new ShopBot tools
today (~30–40/month); ~15,000 legacy ShopBots run older Windows software that
FabMo is intended to replace eventually.

Motion is handled by G2core firmware (our fork of Synthetos g2core, C++) on a
microcontroller, talking to the engine over USB serial with JSON. The engine
streams g-code to G2 and receives ~100 ms status reports back.

## The codebases and where they live on this Pi

| Repo (github.com/FabMo/…)      | On this Pi            | Role |
|--------------------------------|-----------------------|------|
| FabMo-Engine                   | /fabmo                | The server: runtimes, dashboard, apps, profiles, G2 driver |
| FabMo-Updater                  | /fabmo-updater        | Independent agent (port 81): updates, patches, restarts, diagnostics |
| FabMo_RPi_SD_Image_Builder     | /fabmo_image_builder  | Scripts + notes to build the Pi SD image |
| fabmo-def                      | /fabmo-def            | Per-machine definition (which profile) and user snapshots; survives updates |
| FabMo-G2-Core                  | not on this Pi (yet)  | Motion firmware. Edited on the PC, compiled in Atmel Studio on a separate PC |

Runtime state — everything that changes after install — lives in **/opt/fabmo**
(config, macros, profiles as applied, jobs, logs). Backups in /opt/fabmo_backup
and other /opt/… fabmo folders. /fabmo itself is in theory never modified after
installation; on this dev Pi it is the working checkout. Never add /opt as a
whole to a session (backups are large); add /opt/fabmo when runtime context is
needed.

Restart after server-side changes: `systemctl restart fabmo` (engine),
`systemctl restart fabmo-updater` (updater). Client-side (dashboard) changes
need a hard browser reload; some need `npm run build` first (see engine notes).

## Team and workflow

Two developers share the repos (Ted and Brian, at the moment). Conventions:
- Personal-prefix branches (`th_…` for Ted) → PR into `master`.
- Functional / motion / runtime changes get looked over by the other developer
  before merge. UI-only changes are generally merged by their author.
- Work is planned in GitHub Issues (being rebuilt from the "FabMo Work"
  spreadsheet in Sept 2026). An issue that says "Ted reviews before Step 2"
  means stop and report; do not continue into the next step.

## Coding conventions (all JS repos)

- Deliberately **old-style JavaScript**: prototype-pattern "classes",
  `require`, callbacks. Promises/async are used only where they are clearly
  more practical for a specific piece (some newer code does). Do not
  modernize surrounding code as a side effect of a change; match the style of
  the file you are in. Consistency and readability over idiom.
- Prettier + ESLint are configured in FabMo-Engine; run `npm run lint` before
  a PR there.
- Callback-order and timing bugs are the classic failure mode in this
  codebase. When touching async code, state explicitly what fires in what
  order and what happens if the order flips.

## Safety — this software moves a spindle

- Anything touching motion, feedhold/quit/resume, spindle on/off, outputs,
  limits, homing, or the G2 driver (`g2.js`) needs a physical test on a real
  tool before merge. Say so in the PR; do not claim it is tested if it was only
  run headless.
- `g2.js` has deliberate state "jiggerypokery" around quit/stop/hold to work
  around firmware behavior. Do not simplify it without an explicit ask.
- Outputs (spindle, dust collector, etc.) must never come on unexpectedly at
  startup or after a file. There is an output mask/policy system for this
  (`runtime/output_policy.js`, `output_triggers.js`).

## Two languages the tools run

- **OpenSBP** (ShopBot's language; `.sbp` files, macros `macro_N.sbp`) — the
  primary language for ShopBot users; runtime in `/fabmo/runtime/opensbp`.
  Variables are case-insensitive. `$name` = persistent (stored in config tree
  `opensbp.variables`), `&name` = file-local.
- **G-code** — runtime in `/fabmo/runtime/gcode`.

## Where things are documented

- Engine: `/fabmo/README.md`, `/fabmo/doc/`, `CONFIG_VARIABLE_FEATURE.md`.
- Updater: `/fabmo-updater/README.md` (good), `patches/README.md`, `hooks/index.js`.
- Image builder: README plus the TAILSCALE-*.md and PROCEDUREdetails.txt.
- User-facing docs: gofabmo.org, opensbp.org, shopbottools.com.

## Additional Documentation

In some cases, recent work by developers has been documented in `.md` files for other 
developers, ai, and users and saved in `/fabmo/doc/`. This folder is worth checking
for features and functions recently added or underdevelopment in FabMo. At the moment,
the folder contains 3 files of interest: `canned_cuts.md` describes ongoing work on
mini-apps that are conceptualized as 'shop tools' in traditional woodworking shops (
The development work conceptualizes these apps as modular and having CNC functionality
that may be useful elsewhere); `i18nmd.md` describes an ongoing project for creating
language translation versions of the FabMo ai; and `misc_project_layout.md` contains some
additional details of FabMo organization -- in general this is covered in other docs as well.  

## Repo-specific notes (imported)
@/fabmo/CLAUDE.md
@/fabmo-updater/CLAUDE.md
@/fabmo-def/CLAUDE.md
@/fabmo_image_builder/CLAUDE.md
