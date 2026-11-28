# Install the QFlex runtime

QFlex 0.1.16 is an experimental, source-free runtime preview. Each archive holds compiled JavaScript, package manifests, the unpacked Chrome extension, and the production dependencies QFlex runs with; it does not include TypeScript source for QFlex, source maps, build scripts, tests, or fixture servers. This release is a prerelease and has not been verified on macOS or Linux.

## Requirements

Use Node.js 26.7.0. pnpm is no longer needed to install a release. The runtime requires an existing Chrome installation; the extension declares Chrome 125 as its minimum version.

## Download and verify

Download one archive and `SHA256SUMS` from the [v0.1.16 prerelease](https://github.com/whackur/qflex/releases/tag/v0.1.16). The direct assets are [ZIP](https://github.com/whackur/qflex/releases/download/v0.1.16/qflex-0.1.16-runtime.zip), [tar.gz](https://github.com/whackur/qflex/releases/download/v0.1.16/qflex-0.1.16-runtime.tar.gz), and [SHA256SUMS](https://github.com/whackur/qflex/releases/download/v0.1.16/SHA256SUMS). Keep the files in the same directory while checking the checksum.

On Windows PowerShell, verify the ZIP by comparing its digest with the matching `SHA256SUMS` line:

```powershell
$actual = (Get-FileHash .\qflex-0.1.16-runtime.zip -Algorithm SHA256).Hash.ToLowerInvariant()
$expectedLine = Get-Content .\SHA256SUMS | Where-Object { $_ -match '  qflex-0.1.16-runtime\.zip$' }
$expected = ($expectedLine -split '\s+')[0]
if ($actual -ne $expected) { throw 'SHA-256 mismatch' }
```

On macOS, run:

```bash
actual=$(shasum -a 256 ./qflex-0.1.16-runtime.tar.gz | awk '{print $1}')
expected=$(awk '$2 == "qflex-0.1.16-runtime.tar.gz" { print $1 }' SHA256SUMS)
test "$actual" = "$expected"
```

On Linux, run:

```bash
actual=$(sha256sum ./qflex-0.1.16-runtime.tar.gz | awk '{print $1}')
expected=$(awk '$2 == "qflex-0.1.16-runtime.tar.gz" { print $1 }' SHA256SUMS)
test "$actual" = "$expected"
```

## Extract and install

The archive contains a `qflex-0.1.16-runtime` directory with regular files and directories only. The directory links that Node.js needs to find dependencies are listed in `qflex-links.json`, and the installer creates them. Extract into a new, empty directory, then run the installer from that directory with Node.js.

Windows PowerShell:

```powershell
$extractBase = Join-Path $env:TEMP 'qflex-0.1.16'
New-Item -ItemType Directory -Path $extractBase | Out-Null
Expand-Archive .\qflex-0.1.16-runtime.zip -DestinationPath $extractBase
node (Join-Path $extractBase 'qflex-0.1.16-runtime\qflex-install.mjs')
```

macOS or Linux with bash or zsh:

```bash
extract_base="$(mktemp -d "${TMPDIR:-/tmp}/qflex-0.1.16.XXXXXX")"
tar -xzf ./qflex-0.1.16-runtime.tar.gz -C "$extract_base"
node "$extract_base/qflex-0.1.16-runtime/qflex-install.mjs"
```

`qflex-install.mjs` uses only Node.js itself. It creates the listed links inside the extracted directory and then runs `qflex update install --from` on it, which copies the runtime into the managed install at `~/.qflex/app/0.1.16`, makes it the active version through the `~/.qflex/current` link, and prints the MCP client registration and next steps as JSON. A first install also creates the extension directory `~/.qflex/extension-relay/extension`, fills it with this version's extension, and records it; every later update synchronises that directory, so Chrome always loads the extension from there. The installer exits with the same code as that command; a non-zero code means the install did not complete. The registration uses the stable `~/.qflex/current` paths, which keep working when a later version is activated, so register clients with those paths rather than with the extracted directory.

Installs made before this version (with the earlier extract-and-`pnpm install` instructions) are not managed installs and are never updated. Reinstall once with these steps, point your MCP client at the new paths, load the extension from `~/.qflex/extension-relay/extension`, and prepare and register the native host again from `~/.qflex/current`.

`third-party-licenses.json` at the runtime root lists each bundled dependency with its name, version, and declared license; each dependency keeps its own license files in its package directory.

## How QFlex runs

A QFlex install runs as separate Node.js processes, all started from `~/.qflex/current`:

- **MCP server** (`packages/mcp/dist/stdio.js`): each MCP client session, such as one Claude Code or Codex session, starts its own server over stdio. It starts nothing else when it starts; it forwards browser tool calls to the Broker.
- **Broker** (`packages/broker/dist/bin.js`): the only process that controls Chrome and owns sessions, tabs, observations, actions, and their records. When the client registration sets `QFLEX_BROKER_ENTRY`, as in the setup below, the first tool call that needs the browser starts a Broker if none is running. The Broker outlives the MCP server that started it and is shared by every MCP server, from any client, that uses the same profile and data directory, so those sessions share the profile's cookies and storage. After 30 minutes with no session or request it exits, and the next browser tool call starts it again.
- **Native host** (`packages/extension-host`): Chrome starts it for the extension relay; it connects the QFlex extension in your Chrome profile to the Broker.
- **CLI** (`packages/cli/dist/bin.js`): runs one command and exits; see [Run the QFlex CLI](#run-the-qflex-cli).

The Broker reads the runtime configuration, `~/.qflex/config.yaml`, and `~/.qflex/credentials.yaml` once when it starts. After changing any of them, run `qflex broker stop`; the next browser tool call starts a Broker with the new settings. `broker stop` refuses while browser work is live, and that work may belong to another agent sharing the Broker, so finish or close those sessions and run it again.

## Run the QFlex CLI

The installer does not put a `qflex` command on your `PATH`. The CLI entry is `~/.qflex/current/packages/cli/dist/bin.js`, and you can always run it as `node ~/.qflex/current/packages/cli/dist/bin.js <command>` (PowerShell: `node (Join-Path $HOME '.qflex\current\packages\cli\dist\bin.js') <command>`). The commands in this guide are written as `qflex <command>`; either create the launcher below once or substitute the full `node` form.

The launcher runs the CLI through the stable `current` link, so it keeps working after every update and never needs to be recreated. It starts whichever `node` is first on `PATH`, which must satisfy the Node.js requirement above.

macOS or Linux with bash or zsh, using `~/.local/bin`:

```bash
mkdir -p "$HOME/.local/bin"
cat > "$HOME/.local/bin/qflex" <<'EOF'
#!/bin/sh
exec node "$HOME/.qflex/current/packages/cli/dist/bin.js" "$@"
EOF
chmod 755 "$HOME/.local/bin/qflex"
```

Many Linux distributions already include `~/.local/bin` on `PATH`; macOS does not. If `command -v qflex` prints nothing in a new terminal, add `export PATH="$HOME/.local/bin:$PATH"` to `~/.zshrc` (zsh, the macOS default) or `~/.bashrc` (bash), then open a new terminal.

Windows PowerShell, creating `qflex.cmd` under `%LOCALAPPDATA%\QFlex\bin` and adding that directory to your user `Path`:

```powershell
$bin = Join-Path $env:LOCALAPPDATA 'QFlex\bin'
New-Item -ItemType Directory -Force -Path $bin | Out-Null
Set-Content -LiteralPath (Join-Path $bin 'qflex.cmd') -Encoding ascii -Value '@echo off', 'node "%USERPROFILE%\.qflex\current\packages\cli\dist\bin.js" %*'
$userPath = [Environment]::GetEnvironmentVariable('Path', 'User')
if (("$userPath" -split ';') -notcontains $bin) {
  [Environment]::SetEnvironmentVariable('Path', (("$userPath".TrimEnd(';'), $bin) -join ';').TrimStart(';'), 'User')
}
```

Open a new terminal so the updated `Path` applies; `qflex` then works in PowerShell and Command Prompt. `cmd.exe` re-parses arguments passed through `qflex.cmd`, so for arguments containing quotes or JSON, such as `qflex qa run --task-input`, use the full `node` form instead.

Check the launcher with `qflex update status`, which reads local state only and prints a JSON report. `qflex --help` lists every command. The commands for operating an install are:

| Command | Purpose |
| --- | --- |
| `qflex doctor [--json]` | Read-only diagnosis of the resolved configuration and Broker status; it does not start Chrome or call models. |
| `qflex broker stop` | Ask the project's Broker to shut down gracefully; it refuses while work is live and never starts a Broker. |
| `qflex update status` | Show the local update state without network access. |
| `qflex update check` | Ask GitHub for the newest public release. |
| `qflex update run` | Check and stage a verified newer version according to the update mode, at most once a day, without switching `current`. |
| `qflex update apply` | Activate a staged version now, with every MCP client closed. |
| `qflex update rollback` | Return to the version the last update replaced. |

`qflex broker status` reports only a Broker named by an explicit `QFLEX_BROKER_ENDPOINT`; to inspect the Broker your MCP clients use, run `qflex doctor`. The remaining commands (`qa`, `navigation benchmark`, and `calibration collect`) are for QFlex development and evaluation; `qflex --help` describes them.

## Automatic updates

A managed install updates itself by default. About a minute after an MCP server of the install starts, QFlex checks the public releases (at most once a day), downloads the newer `tar.gz`, verifies it against `SHA256SUMS`, extracts and smoke-tests it, and stages it under `~/.qflex/app/<version>`. An archive that fails verification is never staged. The next time an MCP client starts the QFlex MCP server, the server activates the staged version before it serves and then serves the new version. Startup activation defers optional cleanup of older version directories. A later eligible update run may remove only versions not protected by verified current, rollback, staged, process, and launcher evidence; when persisted activation state or any protection is uncertain, QFlex keeps the additional versions.

Activation is skipped, changing nothing, and tried again at the next start when another QFlex process uses the install (another MCP client's server, a native host, a CLI command, or a Broker with live work; an idle Broker is stopped gracefully), when the process list cannot be read, or when the registered native-host launcher names one installed version (see below). After an update Chrome still runs the previous extension build, which keeps working with the new version; QFlex records an automatic reload attempt before contacting the Broker. A verified reload clears the pending mark and a confirmed no-effect result allows another attempt on a later run. Interrupted, unverified, or older pending attempts require manual inspection and appear in `qflex doctor` as `update.extension`.

The mode is `apply` (default: check, stage, and activate at MCP start), `check` (only record the newest release), or `off` (no automatic check or activation). Set it in the user config file `~/.qflex/config.yaml`:

```yaml
version: 1
mcp:
  autoUpdate: off
```

or with the `QFLEX_AUTO_UPDATE` environment variable (`apply`, `check`, or `off`), which takes precedence over the config file; set it in the MCP server's environment. Any other value stops the updater instead of falling back. To inspect or trigger it by hand, use the `qflex update` commands listed in [Run the QFlex CLI](#run-the-qflex-cli); each prints a JSON report.

### Apply an update by hand

Automatic activation needs no action beyond restarting your MCP clients. To activate a staged version yourself, or when `qflex doctor` or `qflex update status` shows that activation keeps being skipped:

1. Exit every MCP client session that uses QFlex, in every client (each Claude Code and Codex session, for example), so that no QFlex MCP server is running.
2. Run `qflex broker stop`. Exit code 0 means the Broker stopped or was not running; 1 means it refused because browser work is still live.
3. Run `qflex update apply` and read `outcome` in its JSON line. `activated` means the new version is active; `not_staged` means there is no newer staged version (run `qflex update run` to stage one).
4. On `blocked`, the `reason` names what kept the running version: `version_in_use` (a QFlex MCP server, native host, or CLI of the install is still running), `broker_running` or `broker_refused` (a Broker still has live work), `process_listing_unsure` (the process list could not be read), or `launcher_names_version` (see below). Remove that cause and run the command again.

Start your MCP clients again afterwards; their MCP servers then run the activated version.

A native host registered from an earlier version runs that version directly. Because an older native host can reject a newer Broker and extension, updates are not activated while it is registered (`launcher_names_version`). Run `node ~/.qflex/current/packages/extension-host/dist/bin.js register` once; it rewrites only the launcher so it follows `~/.qflex/current`, and leaves the host configuration and secret as they are.

### Recover an interrupted Windows activation

Windows activates `current` by replacing a directory junction. If QFlex or its host process is terminated after removing the old junction but before moving the new one into place, `current` is missing and one `.current-<pid>` junction may remain. Close every QFlex MCP client and confirm no QFlex process still uses this installation before recovery. The command below moves only a single valid junction whose target is an installed version with its install manifest and MCP entry; it stops when any check is uncertain. For a nondefault `--root`, set `$root` to that install root.

```powershell
$root = Join-Path $HOME '.qflex'
$rootItem = Get-Item -LiteralPath $root -Force -ErrorAction Stop
if (-not $rootItem.PSIsContainer -or ($rootItem.Attributes -band [IO.FileAttributes]::ReparsePoint)) {
  throw 'the install root is not a regular directory'
}
$current = Join-Path $root 'current'
if ($null -ne (Get-Item -LiteralPath $current -Force -ErrorAction SilentlyContinue)) {
  throw 'current already exists; inspect the installation before recovering'
}
$pending = @(Get-ChildItem -LiteralPath $root -Directory -Force -Filter '.current-*')
if ($pending.Count -ne 1) { throw 'expected exactly one pending current junction' }
$candidate = $pending[0]
if (-not ($candidate.Attributes -band [IO.FileAttributes]::ReparsePoint)) {
  throw 'the pending current entry is not a junction'
}
$target = & node -e 'process.stdout.write(require("node:fs").readlinkSync(process.argv[1]))' $candidate.FullName
if ($LASTEXITCODE -ne 0 -or [string]::IsNullOrWhiteSpace([string]$target)) {
  throw 'could not read the pending junction target'
}
$target = [IO.Path]::GetFullPath([string]$target)
$app = [IO.Path]::GetFullPath((Join-Path $root 'app')).TrimEnd('\')
$appPrefix = $app + '\'
$targetParent = [IO.Path]::GetDirectoryName($target.TrimEnd('\'))
$appItem = Get-Item -LiteralPath $app -Force -ErrorAction Stop
if (-not $target.StartsWith($appPrefix, [StringComparison]::OrdinalIgnoreCase) -or
    -not [String]::Equals($targetParent, $app, [StringComparison]::OrdinalIgnoreCase)) {
  throw 'the pending junction target is outside the app version directories'
}
if (-not $appItem.PSIsContainer -or ($appItem.Attributes -band [IO.FileAttributes]::ReparsePoint)) {
  throw 'the install app directory is not a regular directory'
}
$targetItem = Get-Item -LiteralPath $target -Force -ErrorAction Stop
if (-not $targetItem.PSIsContainer -or ($targetItem.Attributes -band [IO.FileAttributes]::ReparsePoint)) {
  throw 'the pending junction target is not a regular directory'
}
$manifest = Get-Content -LiteralPath (Join-Path $target 'qflex-install.json') -Raw | ConvertFrom-Json
if ($manifest.version -ne (Split-Path $target -Leaf) -or
    -not (Test-Path -LiteralPath (Join-Path $target 'packages\mcp\dist\stdio.js') -PathType Leaf)) {
  throw 'the pending junction does not name a valid installed version'
}
Move-Item -LiteralPath $candidate.FullName -Destination $current
Write-Output "Recovered current -> $target"
Resolve-Path -LiteralPath $current
& node (Join-Path $current 'packages\cli\dist\bin.js') update status
if ($LASTEXITCODE -ne 0) { throw 'update status failed after recovery' }
```

If the check stops, leave the version directories and pending entry in place for manual inspection. A termination after the junction replacement but before state recording can leave the new version active without a recorded rollback target.

## Recommended setup: extension relay

The extension relay connects QFlex to a Chrome profile that you select and keep open. QFlex does not launch or close that Chrome instance. Use a profile dedicated to QFlex or intentionally selected for it; sessions on the same profile share its cookies and storage.

The commands below use `$runtime` for the stable install path: `$HOME/.qflex/current` (PowerShell: `$runtime = Join-Path $HOME '.qflex\current'`; bash or zsh: `runtime="$HOME/.qflex/current"`).

1. In the Chrome profile you selected, open `chrome://extensions`, enable Developer mode, choose **Load unpacked**, and select the extension directory the installer printed, `~/.qflex/extension-relay/extension` (PowerShell: `Join-Path $HOME '.qflex\extension-relay\extension'`). Do not load it from `$runtime`, which changes with every update. Open the QFlex popup, expand **Diagnostics**, and copy the **Profile** instance ID.
2. Prepare and register the native host. Replace `PROFILE_INSTANCE_ID` with the value from the extension popup. These commands create protected per-user host files and register the host with Chrome for your OS user; do not edit the generated secret or host files.

   Windows PowerShell:

   ```powershell
   $extensionHost = Join-Path $runtime 'packages\extension-host\dist\bin.js'
   node $extensionHost prepare --profile-instance-id PROFILE_INSTANCE_ID
   node $extensionHost register
   ```

   macOS or Linux with bash or zsh:

   ```bash
   node "$runtime/packages/extension-host/dist/bin.js" prepare --profile-instance-id PROFILE_INSTANCE_ID
   node "$runtime/packages/extension-host/dist/bin.js" register
   ```

3. Create a runtime configuration file outside the install directory. The extension host's default configuration path is `%LOCALAPPDATA%\QFlex\native-host\host-config.json` on Windows, `~/Library/Application Support/QFlex/native-host/host-config.json` on macOS, and `${XDG_DATA_HOME:-~/.local/share}/QFlex/native-host/host-config.json` on Linux. Use the absolute path for your OS and merge this `browser` object into an existing runtime file instead of replacing other settings:

   ```json
   {
     "schemaVersion": 1,
     "browser": {
       "backend": "extension-relay",
       "hostConfigPath": "/absolute/path/to/native-host/host-config.json"
     }
   }
   ```

   For Windows JSON paths, use forward slashes, such as `C:/Users/you/AppData/Local/QFlex/native-host/host-config.json`.
4. Register the MCP server with the registration that `qflex-install.mjs` printed, and set `QFLEX_CONFIG_PATH` to that runtime configuration. For Claude Code, with the stable paths:

   Windows PowerShell:

   ```powershell
   $node = (Get-Command node).Source
   $brokerEntry = Join-Path $runtime 'packages\broker\dist\bin.js'
   $mcpEntry = Join-Path $runtime 'packages\mcp\dist\stdio.js'
   $runtimeConfig = 'C:\absolute\path\to\runtime.json'
   claude mcp add --scope user qflex -e "QFLEX_CONFIG_PATH=$runtimeConfig" -e "QFLEX_BROKER_ENTRY=$brokerEntry" -- $node $mcpEntry
   ```

   macOS or Linux with bash or zsh:

   ```bash
   node_path="$(command -v node)"
   broker_entry="$runtime/packages/broker/dist/bin.js"
   mcp_entry="$runtime/packages/mcp/dist/stdio.js"
   runtime_config="$HOME/.qflex/runtime.json"
   claude mcp add --scope user qflex -e "QFLEX_CONFIG_PATH=$runtime_config" -e "QFLEX_BROKER_ENTRY=$broker_entry" -- "$node_path" "$mcp_entry"
   ```

   For Codex, use the same absolute paths in `~/.codex/config.toml` under `[mcp_servers.qflex]` as `command`, `args`, and `env.QFLEX_CONFIG_PATH` / `env.QFLEX_BROKER_ENTRY`. Set `QFLEX_PROJECT_DIR` to the project directory if your MCP client does not set a project working directory for the server.

Tool discovery only starts the MCP server. The first valid browser call opens a QFlex-owned session and waits for the extension relay; listing tools alone does not verify the relay connection. The extension popup status is also not a connection check.

## Local Chrome fallback

To use the local backend, set the runtime configuration to `{"schemaVersion":1,"browser":{"backend":"local"}}` and register the same MCP and Broker entries. The local backend launches Chrome with a QFlex-managed profile; it does not use the profile selected for the extension relay. Core browser operations do not require the extension on this backend. Keep `browser.backend` omitted only when using the backwards-compatible local default.

## Verify the setup

Run the read-only doctor command with the same environment as the MCP server to inspect its resolved configuration and Broker status:

```powershell
qflex doctor --broker-entry "$runtime\packages\broker\dist\bin.js"
```

```bash
qflex doctor --broker-entry "$runtime/packages/broker/dist/bin.js"
```

The doctor does not start Chrome or call models. To verify browser access, use your MCP client to make a browser call after the extension and native host are installed, or after choosing the local backend. Tool listing by itself does not verify browser access.

QFlex makes no internal model calls without an explicit runtime model configuration. Optional internal model providers belong in the Broker's runtime configuration and `~/.qflex/credentials.yaml`, never in the MCP client configuration. A connected agent can call the QFlex `settings` tool with kind `get` and follow its `setup.nextSteps`, which name each required setting and whether the running Broker must be restarted, without reading credential values.
