# `homebrew-localdrop`

The Homebrew tap for [LocalDrop](https://github.com/Abhi-Subedi/LocalDrop) —
self-hosted file sharing on your own network.

```bash
brew install abhi-subedi/localdrop/localdrop
```

LocalDrop is a user-level daemon. Homebrew will not start it for you:

```bash
localdrop                     # start it (Ctrl-C to stop)
localdrop install-service     # or run at login via launchd
```

It serves `http://<your-lan-ip>:8080` and prints a one-time setup token on first
run. Files and the embedded database live in
`~/Library/Application Support/LocalDrop`.

Full configuration reference:
[docs/CONFIGURATION.md](https://github.com/Abhi-Subedi/LocalDrop/blob/main/docs/CONFIGURATION.md).

## What is in here

| Path | |
|---|---|
| `Formula/localdrop.rb` | The formula. Generated, not hand-edited. |
| `.github/workflows/sync-formula.yml` | Points the formula at the newest release. |

There is one formula and no casks. If you are on Apple Silicon, the `arm64`
tarball is selected automatically; Intel gets `x64`. Both are checksum-verified
against the release's `SHA256SUMS` before anything is unpacked.

## How the formula stays current

`sync-formula.yml` runs daily and on demand. It resolves the newest published
release, reads the macOS checksums from that release's `SHA256SUMS`, and
rewrites the formula — including the version literal and both per-architecture
digests.

Two things it deliberately does not do:

- **It does not reimplement the substitution.** It fetches
  `packaging/update-brew-formula.py` from the LocalDrop repository, so "rewrite
  the formula for release N" has exactly one implementation. The version on this
  page used to drift precisely because two copies of that logic disagreed.
- **It needs no credentials.** It pushes with this repository's own
  `GITHUB_TOKEN`. The release workflow in the parent repository can push here
  too, but only with a fine-grained PAT (`TAP_TOKEN`) that has to be created in a
  browser — and while that secret was missing, the parent repository's formula
  kept its placeholders and `brew install` did not work at all.

Run it by hand from the Actions tab, or nudge it immediately after a release
with:

```bash
gh api -X POST repos/Abhi-Subedi/homebrew-localdrop/dispatches \
  -f event_type=localdrop-released
```

A release therefore reaches `brew upgrade` within about a day. If you want it
faster, configure `TAP_REPO=Abhi-Subedi/homebrew-localdrop` and `TAP_TOKEN` on
the parent repository and the release workflow will update this formula during
the release itself.

## Verifying a formula before you trust it

```bash
brew cat localdrop            # what is actually installed
brew info --json=v2 localdrop | jq '.formulae[0].urls'
```

Or check it by hand — the digests in the formula must equal the ones the release
publishes:

```bash
curl -sSL https://github.com/Abhi-Subedi/LocalDrop/releases/download/v1.1.0/SHA256SUMS
```

## Licence

[AGPL-3.0-only](https://github.com/Abhi-Subedi/LocalDrop/blob/main/LICENSE). If
you host a modified LocalDrop as a network service you must offer your modified
source to your users. The reasoning is in
[ADR-010](https://github.com/Abhi-Subedi/LocalDrop/blob/main/docs/adr/ADR-010-license.md).