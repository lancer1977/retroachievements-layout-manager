# Validation

## Command

Run the repo-native validation wrapper:

```bash
bash scripts/validate.sh
```

## Notes

Validation currently checks XML/csproj/config/resx inputs with `xmllint`.

This is a structural validation path for the repository shape. It does not
compile the .NET Framework 4.7.2 WinForms app, exercise WebView2/Cef-style UI
surfaces, publish ClickOnce artifacts, or call the RetroAchievements API.

Manual Windows builds need the appropriate Visual Studio/.NET Framework desktop
tooling and user-provided RetroAchievements credentials. Keep real usernames and
API keys out of tracked files; `.env.example` documents the optional local
values used during manual runs.
