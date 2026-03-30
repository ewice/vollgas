# Installing Vollgas for Codex

Enable vollgas skills in Codex via native skill discovery. Just clone and symlink.

## Prerequisites

- Git

## Installation

1. **Clone the vollgas repository:**
   ```bash
   git clone https://github.com/obra/vollgas.git ~/.codex/vollgas
   ```

2. **Create the skills symlink:**
   ```bash
   mkdir -p ~/.agents/skills
   ln -s ~/.codex/vollgas/skills ~/.agents/skills/vollgas
   ```

   **Windows (PowerShell):**
   ```powershell
   New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
   cmd /c mklink /J "$env:USERPROFILE\.agents\skills\vollgas" "$env:USERPROFILE\.codex\vollgas\skills"
   ```

3. **Restart Codex** (quit and relaunch the CLI) to discover the skills.

## Migrating from old bootstrap

If you installed vollgas before native skill discovery, you need to:

1. **Update the repo:**
   ```bash
   cd ~/.codex/vollgas && git pull
   ```

2. **Create the skills symlink** (step 2 above) — this is the new discovery mechanism.

3. **Remove the old bootstrap block** from `~/.codex/AGENTS.md` — any block referencing `vollgas-codex bootstrap` is no longer needed.

4. **Restart Codex.**

## Verify

```bash
ls -la ~/.agents/skills/vollgas
```

You should see a symlink (or junction on Windows) pointing to your vollgas skills directory.

## Updating

```bash
cd ~/.codex/vollgas && git pull
```

Skills update instantly through the symlink.

## Uninstalling

```bash
rm ~/.agents/skills/vollgas
```

Optionally delete the clone: `rm -rf ~/.codex/vollgas`.
