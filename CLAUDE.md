# GTM Command Center: handover for Claude Code

Owner: Marcos (marcosmcuellar@gmail.com). He has severe ADHD, so always reply with a TLDR first, then short bullets. Be kind and stay on task.

## TLDR

- One-file web app: `index.html` (~511 KB). React 19, already built and minified, with all CSS and JS inline. There is no build step and nothing to install.
- It is a GTM "command center" for one operator who runs several product lines (brands): IZTIC, OLLIN OS, OLLIN GO, NÈNÈMI, CUEPA, plus user-added ones like Chantli.
- All data lives in the browser's `localStorage`. There is no backend.
- Version 51 is in this repo (adds Export / Import data). Version 50 lives at claude.ai as a private Artifact: https://claude.ai/code/artifact/af54616c-24a7-44db-8bb8-85905e04454d
- Next goal: deploy to Vercel as a static site (see "Deploy").

## Files

- `index.html`: the whole app. It opens directly in a browser (`file://` works).
- `CLAUDE.md`: this file.

## IMPORTANT: no original source in this folder

- `index.html` is a **compiled bundle**. The original Vite/React/TypeScript source is not here.
- Versions 48–50 were **patched straight into the minified bundle** (see "Recent changes"). If someone rebuilds from the old source, those fixes are lost and must be ported over.
- When editing:
  - Use exact string replace on unique snippets. Assert the snippet occurs exactly once before replacing.
  - Never reformat or pretty-print the whole file.
  - Minified names (`Q`, `Nt`, `R`, `O`, `mv`…) are only stable inside this build. Re-grep before every edit.
  - Put new CSS as plain rules in the `<style>` block, just before the comment `/* mobile pass: opened system card */`.
- If a long-term codebase is wanted, propose rebuilding as a Vite + React + TS project. Ask Marcos first; it's a big job.

## How it works

### Routes (hash router)

`#/board` (default), `#/execution`, `#/content`, `#/targets`, `#/hunt`, `#/land` (light-themed landing page). `#/home` redirects to `#/board`.

### Main screens

- **Board**: "Independent systems. One operator."
  - This-week strip: overdue, streak, targets hit, Friday countdown.
  - **Systems grid**: one card per brand. The biggest card is the "TODAY" hero.
  - **Focus actions**: 3 tasks of 15 minutes or less.
- **System card, open state.**
  - Header row: LED, SYS-nn, name, phase, health word, chevron.
  - Phase ladder, 90-day objective, queue, latest log line.
  - **Next step** input.
  - Mission, bottleneck, milestones.
  - **Files** dropzone.
  - Open queue.
  - **Digital log**: notes plus "Ask AI".
  - Edit / Remove.
- **Add project** tile, which opens the "New product line" form.
  - One-line AI draft ("Draft with AI ✦").
  - Name, phase, health, mission, bottleneck, color, next step, objective, milestones.
  - **Files** (added in v50).
- **Execution / Content / Targets / Hunt**: task queue, content library, target accounts (174 demo targets), and the outreach "hunt" list. It includes OLLIN OS intel import (JSON/CSV).

### Data and persistence

- localStorage key `gtm-command-center` holds JSON: `{ v: 8, savedAt, data: { plan, inbox, brands, tasks, targets, content } }`.
- If `v` doesn't match, or `targets`/`tasks` are missing, the saved data is **discarded** and the demo seed loads.
  - **Bump `v` only on purpose.** Bumping it wipes every user's projects.
- Saves are debounced with requestAnimationFrame and flushed on `beforeunload`.
- The footer shows `localStorage · NN KB · saved …` and has **Reset to Demo Data**.
- In-memory store methods: `listBrands`, `addBrand`, `updateBrand`, `removeBrand`, `listTasks`, `upsertTask`, `updateTask`, `setTaskStatus`, `listTargets`, `upsertTargets`, `updateTarget`, `listContent`, `saveContent`, `setContentStatus`, `getPlan`, `updatePlan`, `listInbox`, `addInbox`, `removeInbox`.
- App-level actions (React context hook `Q()`) also include `addFiles(brandId, File[])`, `removeFile`, `setMilestones` and `toggleMilestone`.
- Brand fields: `id`, `name`, `phase`, `healthStatus` (good / attention / critical / parked), `mission`, `currentBottleneck`, `color`, `nextStep`, `objective`, `milestones[{id, title, due}]`, `files[]`, `custom`, `updatedAt`.
- Task rules: `estimatedMinutes` must be an integer from 1 to the max (15), and `priority` must be 1, 2 or 3.

### AI (two modes, picked automatically)

1. **API key**
   - The user pastes an Anthropic key in the footer.
   - It is stored in localStorage key `gtm-command-center.ai-key`, never in the file.
   - Calls go from the browser to `https://api.anthropic.com/v1/messages` with `anthropic-dangerous-direct-browser-access: true`.
   - Model `claude-sonnet-4-5`, streaming, max 600 tokens.
2. **Artifact**: only when running inside claude.ai. It uses `window.claude.use("sample")`.

With neither mode, notes and Next step still work. The AI buttons just explain how to add a key.

### Files

- **Inside claude.ai**: stored in the Artifact asset store via `window.claude.use("assets")`. Max 20 MB each. Word, Excel and PowerPoint are rejected, so export them to PDF.
- **Anywhere else (Vercel, `file://`)**: stored inline in localStorage. Max 1.5 MB each, and larger files are refused with a message. Watch localStorage's roughly 5 MB total quota.

## Deploy to Vercel (pending)

- Static site with no framework and no build command. Output = the folder containing `index.html`.
- From this folder:
  1. `npx vercel` (link or create the project, e.g. `gtm-command-center`)
  2. `npx vercel deploy --prod`
- Marcos's Vercel team: `marcosmcuellar-3433's projects` (slug `marcosmcuellar-3433s-projects`).
- Tell Marcos plainly:
  - **Data won't carry over.** The claude.ai copy and the Vercel copy are different sites, so each has its own localStorage. His projects (e.g. Chantli) must be re-added, or moved over with a small export/import feature. That feature is a good next task.
  - On Vercel, AI needs the API key, and files are limited to 1.5 MB.
  - A public URL means anyone with the link can open the app. Data stays in each visitor's own browser, though. Offer Vercel password protection or deployment protection if he wants it private.

## Recent changes (patched into the bundle)

- **v48, mobile pass for the opened system card** (`@media (max-width:700px)` block).
  - Full-width Next step at 16px, so iOS doesn't zoom.
  - Header on one row, objective and log visible, bigger tap targets.
- **v49, desktop header fix for the opened card.**
  - Bug: `.sys-card .sys-head` used 3 grid columns, but the open header has 4 children. The LED floated in the middle, the name was pushed right, and ATTENTION spilled off the card.
  - Fix: `.sys-card.open .sys-head{grid-template-columns:14px minmax(0,1fr) auto 12px}`.
  - Added `padding-right:110px` on the hero card so the TODAY tag doesn't cover the health word.
- **v50, Files on the Add project form.**
  - In component `Md` (the project form): pending files are held in state `xF`.
  - The dropzone UI is wrapped in `.pf-files` and only shows when adding, not editing.
  - On submit: `addBrand(...)`, then `addFiles(newBrand.id, xF)`. That also writes an "Added N files" log line.
  - Upload errors after creating the project are currently swallowed. **TODO:** surface them.
- **v51, Export / Import data** (footer, next to Reset).
  - Component `Bkx` in the footer (`Ev`). Storage object `wn` gained `exportData()` and `importData(obj)`.
  - Export downloads `gtm-command-center-backup-YYYY-MM-DD.json` (same shape as the localStorage value: `{v, savedAt, data}`).
  - Import checks the file (must have `data.targets`/`data.tasks` arrays and `v === 8`), asks to confirm, writes localStorage, clears the pending save, then reloads.
  - Bad files show "⚠ Import failed: …" in the footer.
  - Note: inside claude.ai the sandbox may block downloads; it works on Vercel / `file://`.

- **v52, readability pass.**
  - Every CSS `font-size` from 9px to 14px went up about 1.5–2px (9→11, 10→12, 11→13, 12→13.5, 13→14.5, 14→15). Larger sizes are unchanged.
  - Contrast pass (Marcos: "darker, not bigger"). There is no zoom.
  - Light mode: cards (`--slab`, `--slab-raised`) are pure white `#ffffff`, and `--slab-deep` is `#f7f8fa`.
  - Light mode text: `--text #0a0a0a`, `--muted #1f2937`, `--dim #374151`, `--line #cbd2dc`.
  - Light mode `.liquid` glass is forced to solid white with no blur (rule before the mobile-pass comment).
  - Dark mode: `--muted #d1d5db`, `--dim #b4bac4`.

## Ideas / backlog (confirm with Marcos before building)

1. ~~Export/import all data as JSON~~ (done in v51).
2. Show file-upload errors from the Add project form.
3. Optional sync backend (e.g. Vercel KV or Supabase) so data follows him across devices.
4. Rebuild as a real source project (Vite + React + TS) so future edits aren't bundle patches.

## How to test

- Use Playwright with headless Chromium: open `file://…/index.html` at 1280px and 390px.
- Click `.sys-add button` to add a project. Click `button[aria-label="Expand <Name>"]` to open a card.
- Check for console errors, take screenshots, and look at them.
- After any change, re-test:
  - Add project, with and without files.
  - Open and close a card.
  - Mobile width (no sideways scroll).
  - Reload: data persists.
