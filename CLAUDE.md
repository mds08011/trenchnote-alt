# CLAUDE.md — Standing Orders for TrenchNote

## What this project is

TrenchNote is a lightweight, open-source (AGPLv3) equipment and material tracking tool for construction field logistics. It answers three questions: What is this thing? Where is it? Who moved it? It runs on a Raspberry Pi in a construction trailer and is used by field crews scanning QR codes with their phone cameras.

## Who maintains this

The maintainer is a construction Project Engineer, not a professional software developer. He is skilled with HTML/CSS and learning backend concepts. Therefore:

- Explain jargon on first use in all docs and in your session explanations.
- Prefer boring, readable, well-commented code over clever code.
- When proposing a design choice, briefly state the alternative you rejected and why.
- Never assume familiarity with developer tooling conventions — spell out commands.

## Hard constraints — never violate these

1. **No-install, no-app.** The field-facing experience must work in a plain mobile browser via QR scan. No app store, no login for basic scan-and-move actions, no more than two taps for common tasks.
2. **Low-bandwidth and low-performance.** Pages stay under ~50KB where feasible. Vendored JS only — no CDN calls, no external fonts, no analytics. Must be usable on a cheap Android phone with a weak signal.
3. **No build step.** Plain HTML, CSS, vanilla JS/Alpine.js. If a change would require npm, webpack, or a compile step, stop and flag it to me instead.
4. **Single-binary simplicity.** PocketBase + SQLite. Backup must remain "copy one folder." Any dependency that complicates self-hosting on a Pi or $5 VPS needs my explicit approval.
5. **Stock is derived from the movement ledger, never stored.** The movements table is append-only. Do not add stored quantity/location fields that can drift from the ledger.
6. **Monetize convenience, never capability.** Never gate a core field feature behind anything. All features in this repo are free and AGPL.

## Documentation standing orders

Docs are updated in the same session as the code they describe — never deferred.

- `README.md` — what it is + quickstart. Keep it under one screen of scrolling. Verify the clone URL and commands actually work.
- `docs/ARCHITECTURE.md` — how the pieces fit together, written for the maintainer two years from now. Update whenever structure changes.
- `docs/adr/` — one short ADR (context, decision, alternatives rejected, consequences) for any decision that would be hard to reverse: schema changes, auth model, sync strategy, dependency additions, URL/tag format changes (printed labels make these permanent!).
- `docs/USER_GUIDE.md` — written for foremen and crews. Plain language, screenshots welcome, no jargon.
- `docs/DEPLOY.md` — Pi/VPS setup, backup, and restore steps that have actually been tested.

At the end of every working session, before wrapping up:
1. Update any doc affected by the session's changes.
2. Tell me if any decision made today deserves an ADR, and draft it if so.
3. Give me a one-paragraph plain-English summary of what changed and why.

## Coding standards

- Comment the "why," not the "what."
- Small commits with plain-English messages describing user-visible impact.
- Migrations for every schema change — never hand-edit the production database.
- Anything touching the tag URL format or QR content is a breaking change to physical printed labels. Flag it loudly and require my confirmation.
- Security notes stay honest: if something is not production-safe yet, say so in the docs rather than hiding it.
- Never add Co-Authored-By lines, "Generated with Claude Code", or any Claude attribution to commits, commit messages, or PR descriptions.

## Roadmap context (for your awareness, not action)

Near-term: auth model ADR (field writes without accounts vs. locked-down rules), fix repo naming/clone URL, tested backup script with integrity check.
Mid-term: offline-first PWA with service worker + sync queue (its own project phase, needs ADR first), Add to Home Screen.
Long-term: managed hosting tier, hardware kit (Pi + pre-flashed SD + label sheet). None of these justify violating the hard constraints above.
