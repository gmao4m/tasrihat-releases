# Tasrihat PRO — releases

This public repository holds **only** the published installer (as GitHub Release assets) and the
update **manifest** (`version.json`). **No source code lives here.**

The desktop app checks `version.json` over HTTPS (no token) and offers an update when a strictly
newer version is published.

## `version.json` (the client-facing switch)

```json
{
  "version": "1.0.2",
  "installerUrl": "https://github.com/gmao4m/tasrihat-releases/releases/download/v1.0.2/TasrihatPRO-Setup.exe",
  "sha256": "<64 hex — the installer's SHA-256; the client verifies it before installing>",
  "fileSizeBytes": 58720256,
  "minimumSupportedVersion": "1.0.0",
  "releaseNotesAr": "• …",
  "publishedAt": "2026-09-07T16:00:00Z",
  "previousVersion": "1.0.1"
}
```

**`version.json` is the only thing that reaches clients.** A release can exist without any client
being offered it until `version.json` points to it. This is deliberate: the release pipeline
verifies the asset (and runs an automated clean-machine gate) *before* switching `version.json`.

- `sha256` — mandatory. The client downloads the installer, verifies this hash, and refuses to
  install on a mismatch (a tampered/partial download never reaches the installer).
- `minimumSupportedVersion` — informational floor (a manifest whose own version is below it is
  rejected as malformed). It does not force clients today.
- `previousVersion` — the version served *before* this one, so rollback needs no lookup.

The client's manifest URL (raw file on `main`):
`https://raw.githubusercontent.com/gmao4m/tasrihat-releases/main/version.json`

## Rollback (instant, no release deleted)

Reverting `version.json` to the previous version stops a rollout immediately — the installers stay
published; only the client-facing switch moves back. Run from the app's source repo:

```
powershell -ExecutionPolicy Bypass -File release\rollback.ps1
```

It restores `version.json` to its previous committed revision (byte-for-byte the previous manifest)
and pushes. `previousVersion` tells you which version that is without diffing.

## Releasing (from the source repo — see docs/auto-update.md)

Bump `Product` in `compta/AppVersionInfo.cs`, commit, push. The pipeline (`release/publish.ps1`)
builds, runs the completeness + Win7 + test + parity + **automated cleanroom** gates, creates the
release with Arabic notes, verifies the public asset, and only then writes `version.json` here.
