# AGENTS.md

## Purpose

This repository is a Windows desktop tracker for RetroAchievements layout data and overlays.

## Workflow

- Read `README.md`, `repo-state.md`, and `docs/index.md` before changing files.
- Keep secrets, API keys, and private tokens out of repository notes.
- Keep validation updates aligned to tracker/build health.

## Validation

Run the repo-native validation wrapper:

```bash
bash scripts/validate.sh
```

The wrapper validates XML/csproj/config/resx files.
