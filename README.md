# paureis/scoop-bucket

The [Scoop](https://scoop.sh) bucket for [cloudpin](https://github.com/paureis/cloudpin): pin cloud CLI accounts to a
repo, and stop any command (yours or your AI agent's) that would run on the wrong one.

## Install

On Windows, from cloudpin 1.2.0:

```powershell
scoop bucket add paureis https://github.com/paureis/scoop-bucket
scoop install paureis/cloudpin
```

Upgrade with `scoop update cloudpin`. cloudpin runs on Scoop's own Node.js (the `nodejs-lts` app from Scoop's main
bucket), which Scoop installs if it isn't there yet, and it always uses that one, even when another Node.js (nvm,
the nodejs.org installer) is on PATH. A global install (`scoop install -g paureis/cloudpin`) needs `nodejs-lts`
installed globally too: Scoop adds it as a global dependency, and a per-user `nodejs-lts` doesn't count.

`nodejs-lts` also puts its folders at the front of your user PATH (its manifest's `env_add_path`), so in new shells
`node` and `npm` are Scoop's (and `npm install --global` installs there), ahead of any other Node.js on your user
PATH; one on the system PATH, where the nodejs.org installer puts it, still comes first. Uninstalling `nodejs-lts`
breaks cloudpin until you run `scoop install nodejs-lts` again. Scoop's `no_junction` setting isn't supported: the
`cloudpin` shim runs `nodejs-lts\current\node.exe`, and without junctions there is no `current` folder.

Switching from npm? Run `npm uninstall --global cloudpin` first, so the old copy doesn't keep running from earlier on
PATH.

## What it installs

The manifest, `bucket/cloudpin.json`, downloads `cloudpin-X.Y.Z-windows.zip` from the matching
[GitHub Release](https://github.com/paureis/cloudpin/releases): the files of cloudpin's npm package plus their
production `node_modules`, built by cloudpin's release workflow. Its `hash` is the one listed in that release's
`SHA256SUMS`, and the zip carries the same signed build-provenance attestation as the npm tarball. The `cloudpin`
command is Scoop's exe shim running `nodejs-lts\current\node.exe` with cloudpin's `dist\cli.js`: no `.cmd` or
`.ps1` launcher, so `cmd.exe` never re-reads your arguments (`&`, `|`, `%` and `^` reach cloudpin as typed). To check
the zip yourself:

```powershell
$v = (scoop list cloudpin 6>$null | Where-Object Name -eq cloudpin).Version   # the installed version
Invoke-WebRequest "https://github.com/paureis/cloudpin/releases/download/v$v/cloudpin-$v-windows.zip" -OutFile "cloudpin-$v-windows.zip"
gh attestation verify "cloudpin-$v-windows.zip" --repo paureis/cloudpin   # built and signed by cloudpin's release workflow
(Get-FileHash "cloudpin-$v-windows.zip" -Algorithm SHA256).Hash          # equals the hash in bucket/cloudpin.json
```

Installing changes no shell profile and installs no agent hook. To be protected, run the next steps yourself (see
cloudpin's README):

```powershell
cloudpin shell-init pwsh | Out-String | Invoke-Expression   # guard your terminal: add this line to $PROFILE
cloudpin install-hook claude                                 # guard an AI agent (or codex, cursor, copilot, antigravity, gemini)
cloudpin doctor                                              # check the whole setup
```

## How versions are bumped

After each cloudpin release, `scripts/bump-installers.mjs` in [paureis/cloudpin](https://github.com/paureis/cloudpin)
reads that release's `SHA256SUMS`, checks the zip it lists, and rewrites the manifest's `version`, `url` and `hash`:

```sh
node scripts/bump-installers.mjs X.Y.Z --bucket ../scoop-bucket
```

The change is reviewed and merged by pull request. This repository's CI installs Scoop on a Windows runner, installs
the manifest, checks that `cloudpin --version` matches it, runs a pin file cloudpin must refuse and one it accepts,
checks that arguments with `&`, `%` and `^` reach cloudpin unchanged, and checks that a wrong `hash` makes the
install fail. The manifest also has `checkver` and `autoupdate`, so Scoop's own tools can follow new releases.

## Problems

Report them in [paureis/cloudpin's issues](https://github.com/paureis/cloudpin/issues).

## License

MIT, as cloudpin.
