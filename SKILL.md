---
name: zanka
version: 1.4.1
description: 'Use at the start of ANY project work and read it BEFORE writing code, docs, or build tooling. Governs workflow fundamentals: every project keeps an up-to-date README headed by a centered SVG banner, title, and description; bun replaces npm/pnpm/yarn/npx everywhere; every repo''s default branch is master, never main; anything beyond a one-command dev server gets a Makefile where plain `make` does the entire golden path; projects with a build step end every session with the production build run and passing (`bun run build`); macOS apps needing Accessibility/TCC permissions get stale grants reset programmatically on every reinstall; and for services the project actively uses, tasks their MCP servers can do (Supabase migrations, Stripe ops) are done directly — auth included — instead of handed back, while servers for unused services are never probed. Triggers on new projects, README/docs work, package installs, build/install/release flows, desktop-app permission problems, and "run this migration" moments.'
---

# zanka.md

Zanka Nijiku's philosophy — plain fundamentals drilled until they outclass exotic weapons — applied to a repo: the rules below are non-negotiable, and you take this stance in **every** project, without being asked.

## 1. README fundamentals

Every project gets a README, and it stays in sync with the code — a README describing last week's behavior is a bug. Structure is fixed:

```markdown
<div align="center">

<img src="assets/<project>.svg" alt="<project>" width="330" />

# project-name

**One line saying what this is.**<br>
*Optional second, softer line.*

</div>

---

Everything from here down is normal, left-anchored markdown.
```

- **An existing banner stays.** If the README already has a banner/logo image, keep it exactly as it is — no regenerating, restyling, or "upgrading" it to the rules below unless the user explicitly asks for a new one. The rest of this section is for READMEs that don't have an image yet. A stable visual identity beats a fresh one; leaving it alone is the correct outcome, not a skipped step.
- **Banner first, always, and always an SVG.** Hand-author a clean vector logo/banner and keep it at `assets/<project>.svg`. If the user supplies a reference image, convert it to SVG — trace it (`vtracer`, `potrace`) or redraw it as clean vector art — rather than embedding the raster. A raster is acceptable only in the vanishing case where vector genuinely cannot represent the content (e.g. a required photograph), and you say so explicitly.
- **Art direction: the user's call.** If the user has described a style — in the request, the repo's existing art, or an earlier banner — follow it faithfully. With no direction given, default to minimal flat vector work: few shapes, a restrained palette, no photorealism, nothing that can't reduce to a handful of clean SVG paths. Wordmark/title text may sit inside or beside the art.
- **Sharp corners only.** Rounded corners on README images are banned. The image's outer silhouette — its frame, background card, or edge — is always a hard-edged rectangle or square: no `rx`/`ry` on the outermost shape, no border-radius, no rounded-card look. Shapes *inside* the art may curve as the style demands; the image itself never does.
- **Roll the aspect ratio — with a real RNG.** So banners don't converge on one shape, generate two random integers in **3–9** before drawing (`jot -r 2 3 9`, or `python3 -c 'import random;print(random.randint(3,9),random.randint(3,9))'`) — never "pick" the numbers yourself; model-chosen randomness clusters on the same values. Larger roll : smaller roll = width : height (equal rolls = square card), so every outcome lands between 1:1 and 3:1. Compose *for* the rolled shape instead of squeezing one layout into it — square-ish cards (display width ~300–380) suit a single tight composition with the wordmark under it; wide strips (display width ~560–680) suit a cinematic composition with the wordmark beside the art.
- **Find the reference before drawing.** In order: (1) distill the project into one visual motif; (2) check the user's existing repos (`gh repo list justin06lee --limit 100`) for related projects and prior banners — stay consistent with the family and don't reuse a reference already spent on another repo; (3) search for reference imagery in the style at hand — illustrations, film stills, covers, posters, whatever fits — that *loosely* evokes the motif; the link can be a stretch as long as it's felt (a terminal multiplexer nodding to a swordsman drawing several blades at once; a sync daemon nodding to clones working in eerie unison); (4) draw an original SVG in the chosen style riffing on it. If no direction feels right, download 2–4 candidate references to a scratch dir, `open -a Preview` them, recommend one with reasoning, and draw after the user picks.
- **Reference, never reproduce.** Downloaded references are inspiration only — never commit, trace, or embed copyrighted work in a repo. The shipped banner is always original vector art.
- **Centered header block only**: banner, title, and short description are centered; the rest of the document is left-anchored. Never center body content.
- **No badges, ever.** Badge/shield pills — version tags, deps counts, platform lists, build status, shields.io and its lookalikes — never appear in a README, in the header or anywhere else. If the information actually matters (minimum version, supported platforms), it goes in prose in the description or install section; if it doesn't, it goes nowhere.
- **Words before pictures in the body.** Explain with prose by default. Generate an inline SVG illustration only when a concept is genuinely clearer drawn than written — an architecture with many interacting parts, a visual layout, a state machine. No decorative filler images.
- **Freshness is part of every change.** When behavior, commands, flags, or structure change, the README (and any other docs) update in the same unit of work.

## 2. bun, always

bun is the package manager, script runner, and JS runtime of record:

- `bun install` / `bun add` / `bun remove` — never `npm install`, `pnpm add`, `yarn add`.
- `bun run <script>` — never `npm run`.
- `bunx <tool>` — never `npx`.
- `bun init`, `bun create`, `bun test`, `bun build` where applicable.
- The only lockfile is `bun.lock`. If you find `package-lock.json`, `pnpm-lock.yaml`, or `yarn.lock` in a project you're working on, migrate: run `bun install`, delete the foreign lockfiles, and tell the user you did.
- Docs, README instructions, CI configs, and Makefiles you write also say bun.
- Sole exception: a dependency or tool that concretely breaks under bun. Name the specific incompatibility, use the smallest possible fallback, and prefer returning to bun when it's fixed.

## 3. The Makefile contract

**When to create one:** skip it only for projects fully served by a single trivial command (`bun run dev`, `vercel dev`, and friends). Anything more — compiled binaries, multi-step builds, codegen, bundling, daemons/services, app packaging, permission handling — gets a `Makefile` at the repo root.

The point of the Makefile is **absorbing complexity into one word**. The user types `make` and everything happens. The public surface is at most four targets:

| Target | Contract |
|---|---|
| `make` | The entire golden path: build → install → (macOS permission reset if applicable) → restart/launch. Zero decisions, zero arguments. |
| `make build` | Produce the artifact only. |
| `make install` | Put the binary somewhere on `$PATH` (`/usr/local/bin` or `~/.local/bin` — verify the chosen dir is actually in the user's `$PATH`) so it runs globally by name from any directory, never by repo-relative path. `.app` bundles go to `/Applications`. |
| `make update` | Full refresh of a live binary/service: **stop** everything tied to the current binary (kill processes, `launchctl bootout`/unload daemons and agents), **delete** the old binary, **build** the new one, **install** it, then **restart** everything that was stopped. |

Rules:

- The default target is the golden path — a bare `make` must never print help text instead of doing the work.
- Internal helper targets are fine (and encouraged for readability), but the user never needs to know they exist. Mark targets `.PHONY`.
- Idempotent: running `make` twice in a row must be safe.
- If the project has services/daemons (launchd plists, background agents), `make` and `make update` manage their lifecycle — the user never runs `launchctl` by hand.

## 4. macOS TCC / Accessibility permissions

**The mechanics you must know:** macOS ties privacy grants (Accessibility, Input Monitoring, Screen Recording, Full Disk Access, …) to the binary's code-signing identity/hash. Replacing the binary — every rebuild of an ad-hoc-signed app — silently invalidates the grant while the *stale entry keeps sitting in System Settings looking enabled*. The new binary often won't even re-prompt until the stale entry is gone.

**So for any project that requests TCC permissions** (desktop apps, input listeners, screen tools), the Makefile handles the dance automatically inside `make` and `make update`, before shipping the new binary:

```make
BUNDLE_ID := com.example.myapp
APP       := MyApp

reset-permissions:
	-osascript -e 'quit app "System Settings"'
	-tccutil reset Accessibility $(BUNDLE_ID)
	-tccutil reset ListenEvent $(BUNDLE_ID)      # only if the app uses Input Monitoring
	-tccutil reset ScreenCapture $(BUNDLE_ID)    # only if the app uses Screen Recording
	rm -rf /Applications/$(APP).app
```

Sequence baked into `make` / `make update`:

1. Quit System Settings (it caches the TCC table; a stale open pane hides the reset).
2. `tccutil reset <Service> <bundle-id>` for each permission the app uses — this deletes the stale catalogue entry, not just the checkbox state.
3. Remove the old app/binary.
4. Build and install the fresh binary.
5. Launch it so it re-requests permissions cleanly and the user can grant them to the *new* binary immediately.

Additional discipline:

- Reset only the app's own bundle id — never bare `tccutil reset Accessibility` (that nukes every app's grants).
- If a stable Developer ID signing identity is available, use it — a stable identity keeps grants valid across rebuilds. But keep the reset flow in the Makefile regardless, for the ad-hoc/dev case.
- Never write "then go to System Settings → Privacy & Security and remove the old entry" in a README. That instruction is an admission the Makefile is incomplete.

## 5. MCP servers: do, don't delegate

**Scope first: an MCP server only exists for you once the project actually uses its service.** A connected server is not an invitation to go looking for work. Before touching one, confirm the project genuinely integrates that service — its SDK in the dependencies (`@supabase/supabase-js`, `stripe`, …), env vars or config pointing at it, a migrations directory, real API calls in the code. A frontend with no backend yet, or an app that simply doesn't use Supabase/Stripe, means those servers stay completely untouched — no "just in case" probing, no session-start state checks, no suggesting the service.

For services the project **does** use, the direction flips — do, don't delegate. Before telling the user *any* action item, check whether the service's MCP server can perform it. If it can, perform it:

- Pending Supabase migration → apply it with the Supabase MCP tools yourself. Never say "there's a migration you need to run."
- Stripe configuration, refunds, lookups → the Stripe MCP tools.
- Same for any other service the project uses: the presence of its tools is the instruction to use them.

**Authentication:** if a server is on the MCP list but not authenticated, that is not a reason to skip or defer. Initiate the auth flow immediately, tell the user exactly what to do to complete it ("a browser window/URL is ready — click Authorize"), and then finish the original task in the same session. Silently ignoring an unauthenticated server or pushing the task "for later" is a failure.

**The only stand-downs**, each stated with its specific reason at the moment it applies:

- Genuinely destructive or irreversible operations against production (dropping tables/data, mass refunds, deleting resources) — prepare everything, then present the single confirmation step.
- Tools that require explicit cost confirmation (e.g. creating paid infrastructure) — surface the cost and confirm first.
- The user explicitly reserved the action for themselves.

## 6. master, not main

The default branch of every repo is `master`:

- New repos: `git init -b master`. Never `-b main`, and never trust the machine's `init.defaultBranch` — pass `-b master` explicitly.
- Existing local repos sitting on `main`: rename with `git branch -m main master` as part of your work there, and mention that you did.
- If the repo has a remote where `main` is already published, the local rename still happens, but changing the remote (pushing `master`, moving the default branch, deleting remote `main`) affects collaborators and CI — confirm with the user before touching it.
- Anything you write that names the default branch — CI workflows, scripts, docs, badge URLs — says `master`.
- When another skill, tool, or habit defaults to `main` (e.g. a `git init -b main` reflex), this rule wins.

## 7. The build gate

If the project has a build step — a website that must compile to deploy, a bundled app, anything where a failed build means a failed deploy — the work is not done until that build passes locally, at the end of the session:

- After the **last** change, run the real production build: `bun run build`, `make build`, or whatever the deploy pipeline actually runs. The dev server proves nothing — dev mode tolerates what production builds reject (type errors, unused imports, strict lint, missing build-time env).
- A failing build is yours to fix **now**, in the same session. Never end a turn leaving the user to discover the breakage as a failed deploy and a redeploy cycle.
- Ran the build mid-session? It doesn't count — anything edited afterwards invalidates it. The gate is the build passing *after the final edit*.
- If the build genuinely can't run locally (requires secrets or infrastructure you don't have), say so explicitly — never imply it was verified when it wasn't.

## End-of-task drill

Run through this every time you finish work in a project:

1. README exists, is current, and follows the layout — centered SVG banner + title + description, everything else left-anchored?
2. All package operations, scripts, and docs go through bun? No foreign lockfiles left?
3. If the project outgrew a one-command dev server: Makefile present, and a bare `make` does the whole golden path?
4. `make install` puts the binary on `$PATH` (runnable by name from anywhere)? `make update` does stop → delete → build → install → restart?
5. For TCC-permissioned apps: the permission reset is baked into `make`/`make update`, not documented as manual steps?
6. Zero user-facing action items that a connected MCP server could have executed, no server skipped for being unauthenticated — and no server touched for a service the project doesn't use?
7. Default branch is `master`, with no stray `main` left behind?
8. If the project has a build step: did the production build (`bun run build` / `make build`) pass *after* the final change?
