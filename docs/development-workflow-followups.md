# PlxNative development workflow: follow-ups

**Recorded:** 2026-10-22  
**Status:** Investigation backlog for the fork; **none of these workflow improvements is implemented by this document.**  
**Goal:** Reduce wasted implementation work, make verification trustworthy, and shorten the path from an issue to an upstream-ready PR. Keep changes small and reuse the project's existing tools.

## Priorities

### 1. Check upstream and the installed version before fixing anything
**Value:** Very high · **Effort:** Low

**Observed:** Work on the player HUD Back behavior reached testing and [upstream PR #902](https://github.com/GLinnik21/plx-native/pull/902) before it became clear that the ordinary Back/HUD behavior had already been fixed upstream and was working in nightly. The PR closed without merging. [Upstream issue #811](https://github.com/GLinnik21/plx-native/issues/811) concerns a *separate* Magic Remote pointer/HUD conflict and remains open as of this review.

**Explore:** Add a lightweight, read-only *preflight* to the agent's existing investigation process: fetch upstream main and recent commits; check relevant issues, PRs and nightly changes; identify the installed app version/commit and reproduce there; determine whether the bug persists on the newest relevant build. Distinguish stable-release lag from an actual unfixed regression. Do this before creating a branch or writing code.

**Done when:** Each proposed fix starts with an evidence-backed "still reproducible / already fixed / uncertain" verdict. Avoid an extra framework or lengthy checklist.

### 2. Get a trustworthy CI baseline
**Value:** Very high · **Effort:** Medium

**Observed:** [Fork CI run 39651468536](https://github.com/infectiousstupidity/plx-native/actions/runs/39651468536) had passing ARM cross-build/artifact assertions, lint/type checks, and harness tests. Failures were:
- `plx_media`: `media_key_mode_never_reuses_a_kept_alive_connection` and `media_seeks_and_reopens_in_key_mode_each_do_their_own_handshake` in **both** default and hostsim jobs.
- `ci/test_disk_preflight.py`: `test_a_checkout_outside_the_named_roots_is_not_planned_under_an_override` expected exit code 2, got 1.

Both fork and upstream CI specify `ubuntu-latest`. A runner-image change (24.04 to 26.04) is a **hypothesis**, not a confirmed root cause. The title PR also reported failures on unchanged fork `main`; confirm with comparable baseline runs rather than treating that assertion as proof.

**Explore:** Reproduce on a clean current fork `main` and the relevant upstream commit; compare Ubuntu 24.04 versus 26.04; investigate libcurl/TLS and the disk-test assumptions; fix the actual incompatibility or deliberately pin a runner only if the evidence supports it. Keep this work separate from UI feature changes.

**Done when:** A normal small PR has a reliable green baseline, or clearly identified pre-existing failures that cannot be confused with its changes.

### 3. Actually run and inspect UI verification
**Value:** High · **Effort:** Medium

**Observed:** `AGENTS.md` already requires screenshot inspection and documents `.agents/skills/ui-sim/SKILL.md` and `which-tier`. The risk is inconsistent execution, **not** missing documentation. [Title PR #914](https://github.com/GLinnik21/plx-native/pull/914) merged upstream after LG C1 testing, but the PR description did not include simulator capture evidence.

**Explore:** Make the existing agent workflow produce and inspect focused before/after 1920×1080 captures for relevant UI states (long text, focus, playing/paused, overlays) and include their provenance in the handoff. Reuse existing scene triggers and tooling; do **not** add an indiscriminate snapshot suite.

**Limit:** Windows WSLg simulation cannot show live playback HUD states. Use the macOS clocksink/simvideo player scene or physical TV for those pixels. Native text rasterization, video-plane composition and remote behavior require TV checks; a simulator screenshot is not TV proof.

**Done when:** UI handoffs say exactly which states were captured, visually inspected, and left for device testing.

### 4. Reduce friction in LG C1 deployment and feedback
**Value:** High · **Effort:** Low–medium

**Observed:** Manual build/install/launch/testing, including Dev Manager problems, made iteration slower. Existing TV config, SSH/deploy, `tv-lock`, `tv-session` and separate debug installs already address much of this.

**Explore:** First audit the existing documented commands, then add **only if missing** a single convenient, safe flow to cross-build, install the debug flavor beside stable, launch it, collect logs and report the exact commit installed. Respect the TV lock; do not mutate the user's Plex account or silently drive the remote. Leave physical visual/remote checks to the user when needed.

**Done when:** An agent can hand off an exact testable build and log bundle without manual Dev Manager repetition.

### 5. Make small, upstream-ready PRs the default
**Value:** Medium · **Effort:** Low

**Observed:** [PR #914](https://github.com/GLinnik21/plx-native/pull/914) is a useful pattern: three changed files, reuse of existing text-fit/marquee primitives, focused tests and LG C1 verification; it was merged. The Back effort shows why earlier upstream checks matter.

**Explore:** Start from current upstream `main`; keep feature diffs narrow and follow `CONTRIBUTING.md`, `AGENTS.md` and subsystem guidance. Run the cheapest valid test tier while iterating, then exact-tree gates and device checks as appropriate. Prepare a short PR explaining behavior, touched scope, tests/captures, and remaining limitations. Keep fork-only agent/process changes out of upstream feature PRs. No extra PR-generation framework unless repeated work justifies one.

**Done when:** The reviewable code diff contains only the intended feature/fix, with concise evidence and no unrelated formatting, CI or documentation churn.

### 6. Prevent author-email exposure before pushing
**Value:** High safety value · **Effort:** Low

**Observed:** Recent repository-wide email-history cleanup makes preventive checks preferable to another public history rewrite.

**Explore:** Verify local/global Git author and committer identity and any DCO sign-off configuration. Reuse the repository's existing outbound/secret safeguards where possible; add a small *pre-push* check for unintended author/committer/sign-off email addresses in new commits. Do not place real private emails or credentials in this public document, fixtures, logs, or examples.

**Done when:** Incorrect identity is rejected **before** publication and the remediation stays local.

## Suggested order for a future session

1. **Upstream preflight** (fastest way to avoid entire unnecessary fixes).
2. **CI baseline** (stop unrelated red jobs blocking confidence).
3. **Verification execution**, not another set of screenshot rules.
4. **LG C1 loop**, reusing existing deployment facilities.
5. **Small-PR defaults** and **Git identity guard** (can be handled independently).

Before implementing any item, re-read the relevant current code and skills: repository behavior may have changed since **2026-10-22**. Check for existing implementations before adding new scripts, hooks, docs, or abstractions. This file is a reminder and investigation queue, **not** a mandate to implement all six items.
