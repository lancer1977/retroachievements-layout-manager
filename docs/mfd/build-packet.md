---
title: Mapped-Frontier Build Packet - repo-native build/validation packet
status: draft
owner: lancer1977
issue: https://github.com/lancer1977/retroachievements-layout-manager/issues/5
created: 2026-11-01
updated: 2026-11-01
---

# Mapped-Frontier Build Packet: repo-native build/validation packet

## Intent

Make the actually-enforced build, validation, and merge boundary for
`retroachievements-layout-manager` explicit and durable, so future PRs (for
example the open dependency-refresh work in #3) have a single documented
source of truth for what CI checks exist, what branch protection requires,
and where there are gaps between the two. This is a docs-only Phase 0 packet:
it records current state and open decisions; it does not change code,
workflows, secrets, settings, or branch protection.

Non-goals: adding a `dev` branch, adding new CI jobs, enabling branch
protection, compiling or publishing the WinForms app, calling the live
RetroAchievements API, or rotating/handling any credential value.

## Acceptance criteria

- [ ] AC1: `docs/mfd/build-packet.md` documents every template section with
  real repo evidence (or an explicit "Not applicable" + reason), mirrors the
  actual CI workflow and branch-protection state, and passes
  `validate_mfd_packet.py`.
- [ ] AC2: `.mfd/project.yaml` exists at schema_version 2 and its
  `merge_policy` matches the packet's "Per-project merge policy" section
  exactly (no drift).
- [ ] AC3: A GitHub tracking issue exists for this packet and the PR that adds
  these files links it with `Closes #<issue>`.
- [ ] AC4: No code, workflow, secret, setting, or branch-protection rule is
  modified by this change; only `docs/mfd/build-packet.md` and
  `.mfd/project.yaml` are added/changed.

## Current behavior

- The repository is a Windows desktop (.NET Framework 4.7.2 WinForms,
  WebView2, ClickOnce) tracker/overlay tool for RetroAchievements
  (`README.md`, `repo-state.md`).
- `docs/index.md` and `docs/validation.md` already document a structural
  validation path: `bash scripts/validate.sh`, which runs `xmllint --noout`
  over `*.xml`, `*.csproj`, `*.config`, `*.resx` files up to depth 3
  (`scripts/validate.sh`). This does not compile the app, exercise
  WebView2/Cef UI, publish ClickOnce artifacts, or call the RetroAchievements
  API (`docs/validation.md`).
- CI is a single GitHub Actions workflow, `.github/workflows/ci.yml`: job
  `validate`, triggered on every `pull_request` and on `push` to `main`/
  `master`, which installs `libxml2-utils` and runs `scripts/validate.sh`.
  There is no unit, integration, automation/e2e, or smoke test job anywhere
  in `.github/workflows/`.
- Branch protection on `master` (the repo's default branch) returns HTTP 404
  ("Branch not protected") from
  `gh api repos/lancer1977/retroachievements-layout-manager/branches/master/protection`
  as of 2026-11-01 — there are no required status checks configured at all.
  The `validate` CI job runs on every PR, but nothing currently blocks a
  merge if it fails.
- There is no `dev` branch in this repository; `master` is the only
  integration/protected branch, confirmed via
  `gh repo view --json defaultBranchRef` returning `master`.
- No `.mfd/` or `docs/mfd/` packet existed before this change (first MFD
  packet for this repo).
- Repo label set (`gh label list`) is the GitHub default set: `bug`,
  `documentation`, `duplicate`, `enhancement`, `good first issue`,
  `help wanted`, `invalid`, `question`, `wontfix`. There is no `decision`
  label in this repository.
- One open issue exists: #3, "Refresh compatible NuGet references across
  owned projects" — https://github.com/lancer1977/retroachievements-layout-manager/issues/3,
  recorded as no-blocker, ready to do.

## Behavior map

1. A contributor opens a PR against `master`. GitHub Actions runs the
   `validate` job (XML/csproj/config/resx structural check via `xmllint`).
   Because `master` has no required status checks, a failing `validate` job
   does not block merge today — it is advisory only.
2. A contributor pushes directly to `main` or `master`. The same `validate`
   job runs post-merge; again nothing blocks the push if it fails, since
   there is no branch protection.
3. Failure/recovery: if `scripts/validate.sh` fails (malformed XML/csproj/
   config/resx), the CI run shows red on the PR, but a maintainer can still
   merge manually. There is no automated gate today; this is a documented
   gap, not a defect this packet fixes.
4. Boundary: full compilation, WebView2/ClickOnce publish, and live
   RetroAchievements API calls require Windows desktop tooling and user
   credentials that are out of scope for the Linux CI runner and for this
   docs-only packet (`docs/index.md` boundary note).

## UI proof

Not applicable: this packet makes no code or UI change. It is a docs-only
build/validation packet. The existing UI is documented with screenshots in
`README.md`; no new or changed surface is introduced here.

## Technical contract

- No API, event, data, or auth surface changes. This packet only adds two
  documentation/config files: `docs/mfd/build-packet.md` and
  `.mfd/project.yaml`.
- Ownership: `lancer1977` is the repository and product owner.
- Dependencies: none added. (Separately, #3 proposes a NuGet reference
  refresh — tracked there, not in this packet's scope.)
- Operations: no deployment, no secret access, no runner/infrastructure
  change. The repository has no deploy pipeline; it produces a manually
  built/published Windows ClickOnce artifact outside of CI.
- Secrets: the RetroAchievements username and Web API key are user-supplied
  local settings (`docs/validation.md`, `.env.example` where present) and
  must never be committed or referenced by value. Where a secret path must be
  named in future work, it is referenced as `infisical:/retro/retroachievements-layout-manager/<NAME>`
  only.

## Verification map

| Requirement | Scenario | Validation | Expected evidence |
| --- | --- | --- | --- |
| AC1 | Packet written with all template sections filled | `python3 validate_mfd_packet.py docs/mfd/build-packet.md` | `OK: valid MFD packet: docs/mfd/build-packet.md` |
| AC2 | `.mfd/project.yaml` merge_policy mirrors packet | Manual diff of packet "Per-project merge policy" section vs `.mfd/project.yaml` | Both list identical required checks, branches, and test tiers |
| AC3 | Tracking issue exists and PR links it | `gh issue list` / PR body contains `Closes #<issue>` | Issue URL recorded in this packet's `issue:` front matter |
| AC4 | Only docs files changed | `git diff --stat` on the PR | Only `docs/mfd/build-packet.md` and `.mfd/project.yaml` listed |

## Per-project merge policy

<!-- Mirrors `.mfd/project.yaml` `merge_policy` for this repository. -->

- Required checks: None (branch protection on `master` returns 404 — no
  required status checks configured as of 2026-11-01)
- Integration branch: `master` (no `dev` branch exists in this repository)
- Protected branch: `master`

| Test tier | Command | Required | CI job name |
| --- | --- | --- | --- |
| Unit | GAP: none in CI | No | N/A |
| Integration | GAP: none in CI | No | N/A |
| Automation | `bash scripts/validate.sh` (XML/csproj/config/resx structural check) | No | `validate` (workflow `CI`, `.github/workflows/ci.yml`) |
| Smoke | GAP: none in CI | No | N/A |

## Delivery map

| Order | GitHub issue / card | Requirement / scenario | Dependencies | Validation |
| --- | --- | --- | --- | --- |
| 1 | https://github.com/lancer1977/retroachievements-layout-manager/issues/3 | Refresh compatible NuGet references across owned projects | None (no blocker, ready to do) | Existing `validate` CI job + manual Windows build check (out of scope for this packet) |
| 2 | This packet's tracking issue (step 5) | AC1-AC4 above | None | `validate_mfd_packet.py` + manual diff |

### Rollout and rollback

No rollout: this is a documentation-only change with no runtime, deployment,
or operational effect. Rollback is a plain `git revert` of the commit adding
`docs/mfd/build-packet.md` and `.mfd/project.yaml`; nothing else depends on
these files existing.

### Blockers

| Blocker | Owner | Unblock condition |
| --- | --- | --- |
| None identified for this docs-only packet | lancer1977 | N/A |

### Future requirements

- Secrets: None for this packet. Future work touching the RetroAchievements
  API key must reference it only as `infisical:/retro/retroachievements-layout-manager/<NAME>`,
  never as an inline value.
- Infrastructure: None for this packet.
- Hardware: None for this packet. Future full-build validation would need a
  Windows desktop toolchain (Visual Studio/.NET Framework 4.7.2), which does
  not exist in the current Linux CI runner.

### Open decisions

| Decision | Options | Recommendation |
| --- | --- | --- |
| Whether to add real test tiers (unit/integration/automation/smoke beyond structural validation) | (A) Leave structural-only validation as-is; (B) add unit/integration tests for non-UI logic plus a Windows-hosted smoke build | Recommend (A) for now — the app is WinForms/WebView2-bound and a Windows build runner is a separate infrastructure decision outside this docs-only packet's scope; revisit if/when a Windows runner becomes available |
| Whether to enable branch protection with `validate` as a required check on `master` | (A) Leave `master` unprotected (current state); (B) add branch protection requiring the `validate` status check | Recommend (B) as a low-cost follow-up — `validate` already runs on every PR, so requiring it costs nothing but currently does not block a failing merge; this is a settings change outside this docs-only packet's scope and needs owner approval before being made |
| Whether to introduce a `dev` integration branch | (A) Keep `master` as the only branch (current state); (B) add `dev` per the portfolio-wide default | Recommend (A) — this repo has no CI distinction that would benefit from a `dev` buffer today; only revisit if the portfolio-wide `dev` convention is explicitly extended here |
| `escalate_labels` reference a `decision` label that does not exist in this repo | (A) Use only existing labels (`bug`); (B) create a `decision` label | Recommend (A) for this packet (docs-only, no label creation); flag label creation as a separate small follow-up if the owner wants the portfolio-wide label convention here |

## Approval

- Product owner: lancer1977
- Decision: Pending
- Date: Pending
- Approved commit or immutable revision: Pending
- Approval evidence: Pending
- Approval brief: Pending (to be posted on the tracking issue after this
  packet is committed)

## Deviations

| Date | Requirement / boundary | Decision | Approver | Follow-up |
| --- | --- | --- | --- | --- |
| None | None | None | None | None |

## Release controls and closeout evidence

- Target branch: `master`
- Pull request: Pending
- Required reviewers: Pending
- CI and validation evidence: Pending

### Deployment contract

- Deployment applicability: Not applicable: this is a docs-only packet that
  adds `docs/mfd/build-packet.md` and `.mfd/project.yaml`; no deployable
  artifact is produced or changed.
- Target environments: Not applicable: no deployment is in scope.
- Deployment authority (separate from approval): Not applicable: no
  deployment is in scope.
- Source commit: Pending (recorded once this packet's commit is pushed).
- Immutable artifact / digest: Not applicable: no artifact is produced.
- Rollout record: Not applicable: no rollout is in scope.
- Authenticated smoke proof: Not applicable: no live/authenticated surface is
  touched by this packet.
- Rollback target: Revert the commit adding these two files.

### Release closeout record

- Live verification result: Not applicable: no live/runtime change is
  authorized by this packet.
- Deferred drift / follow-up issue: See "Open decisions" above (branch
  protection, test-tier expansion, `dev` branch, `decision` label) — each is a
  deferred follow-up requiring separate owner approval, not resolved here.
