# Install ego lite on Windows

Read `install.md` first; it covers upgrading an older ego lite and the onboarding step shared by every platform. This file has the Windows install steps, the commands that confirm the installation, and Windows troubleshooting.

## Run the installer

There is no install script for Windows. Have the user download the installer from https://lite.ego.app/ and run it, then open ego lite.

After the app opens, the user completes the first-run onboarding in the app (see `install.md`). Onboarding registers the `ego-browser` command on the PATH. The PATH entry only reaches processes started afterwards, so open a new terminal before checking for the command.

## After installing: confirm `ego-browser` is available

Once the user has finished onboarding, open a new terminal and confirm the command is ready. The command depends on the shell.

PowerShell:

```powershell
Get-Command ego-browser
```

cmd:

```bat
where ego-browser
```

Git Bash:

```bash
command -v ego-browser
```

Once the command exists, verify the runtime. In PowerShell or cmd, where heredoc does not exist:

```powershell
ego-browser nodejs -e "console.log('ego-browser ready')"
```

In Git Bash:

```bash
ego-browser nodejs <<'EOF'
console.log('ego-browser ready')
EOF
```

Printing `ego-browser ready` means the environment is ready. Finally confirm that the installed ego lite prints its usage guide:

```powershell
ego-browser skill
```

Then return to the user's original task, as `install.md` describes.

## Use the Windows ego-browser from WSL

This section applies when ego lite is installed on Windows but the coding agent runs inside WSL. There is no Linux build of ego-browser, so the agent drives the Windows CLI through WSL interop. This is a local bridge, not a separate installation: the Windows browser and its signed-in state are shared.

WSL interop puts Windows PATH directories on the WSL PATH, but it exposes the command as `ego-browser.exe`, so a bare `ego-browser` is still not found. Give the agent a `ego-browser` command by adding a wrapper that forwards everything to the Windows CLI:

```bash
mkdir -p ~/.local/bin
cat > ~/.local/bin/ego-browser <<'SH'
#!/bin/sh
exec ego-browser.exe "$@"
SH
chmod +x ~/.local/bin/ego-browser
```

Make sure `~/.local/bin` is on the WSL PATH, then confirm the bridge from WSL:

```bash
command -v ego-browser
ego-browser skill
ego-browser nodejs <<'EOF'
console.log('ego-browser ready')
EOF
```

If `command -v ego-browser.exe` finds nothing, WSL interop PATH sharing is disabled. Check that `/etc/wsl.conf` enables `[interop] enabled=true`, or point the wrapper at the installed EXE directly:

```bash
exe=$(ls /mnt/c/Users/*/.local/share/ego/active_version_dir/ego-browser.exe 2>/dev/null | head -1)
```

Then replace `ego-browser.exe` in the wrapper with that absolute path.

The `exec` in the wrapper keeps stdin, stdout, stderr, and the exit code intact, so heredoc scripts and `--sdk-path` invocations work unchanged. Two paths differ from a native installation:

- Absolute paths that scripts hand to the browser, such as task space workspaces and `download.saveAs()` targets, are read by the Windows-side runtime. Use Windows paths (`C:/Users/<name>/...`), not WSL paths (`/home/...`), because the Windows process cannot see the WSL filesystem.
- The `ego-browser skill` guide is the Windows guide, and its examples already use Windows path conventions.

## Troubleshooting

- **SmartScreen blocks the installer**: have the user choose "More info" and then "Run anyway" in the SmartScreen dialog.
- **Command still unavailable after onboarding**: open a new terminal first, because an existing one keeps the PATH it started with. Otherwise have the user reopen ego lite, finish onboarding, and retry.
