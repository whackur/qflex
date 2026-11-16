# Install the QFlex runtime

QFlex 0.1.7 is an experimental, source-free runtime preview. It includes compiled JavaScript, package manifests, the frozen pnpm lockfile, and the unpacked Chrome extension; it does not include TypeScript source, source maps, build scripts, tests, or fixture servers. This release is a prerelease and has not been verified on macOS or Linux.

## Requirements

Use Node.js 26.7.0 and pnpm 12.11.2. The runtime requires an existing Chrome installation; the extension declares Chrome 125 as its minimum version. Keep the extracted directory at a stable absolute path because the MCP client, Broker, and native host refer to files there.

## Download and verify

Download one archive and `SHA256SUMS` from the [v0.1.7 prerelease](https://github.com/whackur/qflex/releases/tag/v0.1.7). The direct assets are [ZIP](https://github.com/whackur/qflex/releases/download/v0.1.7/qflex-0.1.7-runtime.zip), [tar.gz](https://github.com/whackur/qflex/releases/download/v0.1.7/qflex-0.1.7-runtime.tar.gz), and [SHA256SUMS](https://github.com/whackur/qflex/releases/download/v0.1.7/SHA256SUMS). Keep the files in the same directory while checking the checksum.

On Windows PowerShell, verify the ZIP by comparing its digest with the matching `SHA256SUMS` line:

```powershell
$actual = (Get-FileHash .\qflex-0.1.7-runtime.zip -Algorithm SHA256).Hash.ToLowerInvariant()
$expectedLine = Get-Content .\SHA256SUMS | Where-Object { $_ -match '  qflex-0.1.7-runtime\.zip$' }
$expected = ($expectedLine -split '\s+')[0]
if ($actual -ne $expected) { throw 'SHA-256 mismatch' }
```

On macOS, run:

```bash
actual=$(shasum -a 256 ./qflex-0.1.7-runtime.tar.gz | awk '{print $1}')
expected=$(awk '$2 == "qflex-0.1.7-runtime.tar.gz" { print $1 }' SHA256SUMS)
test "$actual" = "$expected"
```

On Linux, run:

```bash
actual=$(sha256sum ./qflex-0.1.7-runtime.tar.gz | awk '{print $1}')
expected=$(awk '$2 == "qflex-0.1.7-runtime.tar.gz" { print $1 }' SHA256SUMS)
test "$actual" = "$expected"
```

## Extract and install production dependencies

The archive contains a `qflex-0.1.7-runtime` directory. Choose a permanent, user-owned install location and do not move it after configuring the native host or MCP client.

Windows PowerShell:

```powershell
$installBase = Join-Path $HOME '.qflex\runtime\0.1.7'
New-Item -ItemType Directory -Force -Path $installBase | Out-Null
Expand-Archive .\qflex-0.1.7-runtime.zip -DestinationPath $installBase
$runtime = Join-Path $installBase 'qflex-0.1.7-runtime'
Push-Location $runtime
pnpm install --prod --frozen-lockfile --ignore-scripts
Pop-Location
```

macOS or Linux with bash or zsh:

```bash
install_base="$HOME/.qflex/runtime/0.1.7"
mkdir -p "$install_base"
tar -xzf ./qflex-0.1.7-runtime.tar.gz -C "$install_base"
runtime="$install_base/qflex-0.1.7-runtime"
cd "$runtime"
pnpm install --prod --frozen-lockfile --ignore-scripts
```

The install resolves production dependencies from the frozen lockfile. The runtime archive has no build or local-install command; register its absolute MCP and Broker paths directly with your MCP client.

## Recommended setup: extension relay

The extension relay connects QFlex to a Chrome profile that you select and keep open. QFlex does not launch or close that Chrome instance. Use a profile dedicated to QFlex or intentionally selected for it; sessions on the same profile share its cookies and storage.

1. In the Chrome profile you selected, open `chrome://extensions`, enable Developer mode, choose **Load unpacked**, and select `$runtime/packages/extension/dist` (PowerShell: `$runtime\packages\extension\dist`). Open the QFlex popup, expand **Diagnostics**, and copy the **Profile** instance ID.
2. Prepare and register the native host from the extracted runtime. Replace `PROFILE_INSTANCE_ID` with the value from the extension popup. These commands create protected per-user host files and register the host with Chrome for your OS user; do not edit the generated secret or host files.

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

3. Create a runtime configuration file outside the extracted directory. The extension host's default configuration path is `%LOCALAPPDATA%\QFlex\native-host\host-config.json` on Windows, `~/Library/Application Support/QFlex/native-host/host-config.json` on macOS, and `${XDG_DATA_HOME:-~/.local/share}/QFlex/native-host/host-config.json` on Linux. Use the absolute path for your OS and merge this `browser` object into an existing runtime file instead of replacing other settings:

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
4. Register the MCP server with absolute paths and set `QFLEX_CONFIG_PATH` to that runtime configuration and `QFLEX_BROKER_ENTRY` to the extracted Broker entry. For Claude Code, the commands are:

   Windows PowerShell:

   ```powershell
   $node = (Get-Command node).Source
   $brokerEntry = Join-Path $runtime 'packages\broker\dist\bin.js'
   $mcpEntry = Join-Path $runtime 'packages\mcp\dist\stdio.js'
   $runtimeConfig = 'C:\absolute\path\to\runtime-0.1.7.json'
   claude mcp add --scope user qflex -e "QFLEX_CONFIG_PATH=$runtimeConfig" -e "QFLEX_BROKER_ENTRY=$brokerEntry" -- $node $mcpEntry
   ```

   macOS or Linux with bash or zsh:

   ```bash
   node_path="$(command -v node)"
   broker_entry="$runtime/packages/broker/dist/bin.js"
   mcp_entry="$runtime/packages/mcp/dist/stdio.js"
   runtime_config="$HOME/.qflex/runtime-0.1.7.json"
   claude mcp add --scope user qflex -e "QFLEX_CONFIG_PATH=$runtime_config" -e "QFLEX_BROKER_ENTRY=$broker_entry" -- "$node_path" "$mcp_entry"
   ```

   For Codex, use the same absolute paths in `~/.codex/config.toml` under `[mcp_servers.qflex]` as `command`, `args`, and `env.QFLEX_CONFIG_PATH` / `env.QFLEX_BROKER_ENTRY`. Set `QFLEX_PROJECT_DIR` to the project directory if your MCP client does not set a project working directory for the server.

Tool discovery only starts the MCP server. The first valid browser call opens a QFlex-owned session and waits for the extension relay; listing tools alone does not verify the relay connection. The extension popup status is also not a connection check.

## Local Chrome fallback

To use the local backend, set the runtime configuration to `{"schemaVersion":1,"browser":{"backend":"local"}}` and register the same MCP and Broker entries. The local backend launches Chrome with a QFlex-managed profile; it does not use the profile selected for the extension relay. Core browser operations do not require the extension on this backend. Keep `browser.backend` omitted only when using the backwards-compatible local default.

## Verify the setup

Run the read-only doctor command with the same environment as the MCP server to inspect its resolved configuration and Broker status:

```powershell
node "$runtime\packages\cli\dist\bin.js" doctor --broker-entry "$runtime\packages\broker\dist\bin.js"
```

```bash
node "$runtime/packages/cli/dist/bin.js" doctor --broker-entry "$runtime/packages/broker/dist/bin.js"
```

The doctor does not start Chrome or call models. To verify browser access, use your MCP client to make a browser call after the extension and native host are installed, or after choosing the local backend. Tool listing by itself does not verify browser access.

QFlex makes no internal model calls without an explicit runtime model configuration. Configure any optional provider only in the Broker runtime and credentials setup described by the project documentation; do not put secrets in this MCP client configuration.
