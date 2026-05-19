# Codex installation

This repository can be installed in Codex through the Codex marketplace system. The marketplace manifest is:

```text
.agents/plugins/marketplace.json
```

The plugin manifest is:

```text
.codex-plugin/plugin.json
```

The plugin manifest points Codex to the shared `skills/` directory.

## Which environment should I configure?

Use the Codex environment you actually run:

| Environment | Configure from | Codex home |
|---|---|---|
| Windows Codex App | Windows App UI or Windows PowerShell | `C:\Users\<you>\.codex` |
| Windows Codex CLI | Windows PowerShell | `C:\Users\<you>\.codex` |
| WSL Codex CLI | WSL shell | `/home/<you>/.codex` |
| Linux/macOS Codex CLI | normal shell | `~/.codex` |

Windows and WSL do not automatically share plugin marketplace state.

## Install in Codex App on Windows

Open the Codex App plugin page, choose **Add marketplace**, and enter:

```text
Source: https://github.com/likefallwind/courseskills.git
Git ref: main
Sparse paths: .agents/plugins
```

After the marketplace appears, select the `courseskills` marketplace source, search for `courseskills`, and install it.

Do **not** use `.codex-plugin/` as the marketplace sparse path. `.codex-plugin/plugin.json` is the plugin manifest. Codex App's **Add marketplace** flow discovers marketplace manifests from `.agents/plugins/marketplace.json`.

### Fix a stale failed marketplace add

If the App keeps showing `Failed to add marketplace` after a previous failed attempt, remove the stale marketplace from the Windows Codex CLI and then add it again:

```powershell
codex.cmd plugin marketplace remove courseskills
codex.cmd plugin marketplace add "https://github.com/likefallwind/courseskills.git" --ref main --sparse ".agents/plugins"
```

Restart the Codex App after the CLI add succeeds.

## Install with Codex CLI on Windows PowerShell

Install the Windows Codex CLI if needed:

```powershell
npm.cmd install -g @openai/codex
```

Use `npm.cmd` / `codex.cmd` if PowerShell blocks `npm.ps1` or `codex.ps1` with an execution-policy error.

Then add and install the plugin:

```powershell
codex.cmd plugin marketplace remove courseskills
codex.cmd plugin marketplace add "https://github.com/likefallwind/courseskills.git" --ref main --sparse ".agents/plugins"
codex.cmd plugin list --source courseskills
codex.cmd plugin install courseskills --source courseskills
```

This configures the Windows Codex home, normally under `C:\Users\<you>\.codex`.

## Install with Codex CLI on WSL, Linux, or macOS

Use this when you run Codex from WSL/Linux/macOS shells:

```bash
codex plugin marketplace remove courseskills || true
codex plugin marketplace add \
  'https://github.com/likefallwind/courseskills.git' \
  --ref main \
  --sparse .agents/plugins

codex plugin list --source courseskills
codex plugin install courseskills --source courseskills
```

On WSL, this configures the Linux home directory, usually `~/.codex` inside WSL. It will not fix or update the Windows Codex App unless that App is explicitly running against the same WSL-side configuration.

## Expected plugin layout

After installation, Codex loads a plugin root that contains:

```text
courseskills/
├── .codex-plugin/plugin.json
└── skills/
```

## Local/private marketplace option

If you prefer a purely local/private marketplace, clone this repo and register a local source path such as `./plugins/courseskills` in your own `~/.agents/plugins/marketplace.json`.
