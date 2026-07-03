# retroachievements-layout-manager Repo State

Last reviewed: 2026-07-03

## Purpose

`retroachievements-layout-manager` is a Windows application for RetroAchievements tracking overlays and stream layout helpers.

## Current State

- Repository shape: existing codebase with XML/csproj/config/resx validation.
- Application/runtime code: .NET Framework 4.7.2 WinForms desktop app with WebView2 and ClickOnce publish settings.
- Validation: `bash scripts/validate.sh`.
- Build boundary: full compile and publish require Windows desktop tooling, not the stewardship Linux validation environment.
- Config boundary: RetroAchievements username/API key values are user settings or local-only values and must not be committed.

## Native Validation

```bash
bash scripts/validate.sh
```

## Next Slice

Classify the PackageReference dependency surface and decide whether the repo
should add a Windows build workflow or stay on structural validation only.
