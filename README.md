<h1 align="center">syncz</h1>

<p align="center">
  <strong>Zero-config file sync between local and remote machines</strong>
</p>

<p align="center">
  <a href="https://github.com/JiwanChung/syncz/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="License"></a>
  <a href="https://crates.io/crates/syncz"><img src="https://img.shields.io/crates/v/syncz.svg" alt="Crates.io"></a>
  <a href="https://www.rust-lang.org/"><img src="https://img.shields.io/badge/rust-1.70%2B-orange.svg" alt="Rust"></a>
</p>

<p align="center">
  A fast Rust CLI that wraps <code>rsync</code> and <code>ssh</code> with smart path mapping.<br>
  Mirror paths relative to your home directory with a single command.
</p>

---

## Features

- **Smart Path Mapping** — `~/projects/foo` syncs to `~/projects/foo` on remote, regardless of different home paths
- **Zero Config** — Just run it. No config files needed
- **Bi-directional (Default)** — Automatically syncs both ways (Newer Wins)
- **Watch Mode** — Use `--watch` to automatically sync on file changes
- **Advanced Filtering** — Sync everything with `--all`, or respect `--gitignore`
- **Safety** — Protect overwritten files with `--backup`
- **SSH Host Picker** — Fuzzy-select hosts from `~/.ssh/config`
- **Sensible Defaults** — Auto-excludes `.git/`, `node_modules/`, `target/`, `.DS_Store`

## Installation

Install via [crates.io](https://crates.io/crates/syncz):

```bash
cargo install syncz
```

> Requires `rsync` and `ssh` in your PATH.

## Quick Start

```bash
# Sync current directory with the last used host (Zero-Arg Sync!)
syncz

# Sync current directory with a specific host
syncz my-server

# Watch for changes and push automatically
syncz -w

# Sync everything (including node_modules, etc.)
syncz -a

# Respect .gitignore and backup overwrites
syncz -gb
```

## Usage

```
syncz [OPTIONS] [PATH] [HOST]
```

| Option | Description |
|--------|-------------|
| `[PATH]` | Local path to sync. Defaults to current directory (`.`) |
| `[HOST]` | SSH host. Defaults to last used host |
| `--push` | Push from local to remote (disables bidirectional) |
| `--pull` | Pull from remote to local (disables bidirectional) |
| `-w`, `--watch` | Watch for local changes and sync (Push mode) |
| `-a`, `--all` | Disable default smart excludes and size limits |
| `-l`, `--large` | Allow large files (>10MB) |
| `-g`, `--gitignore` | Respect `.gitignore` file |
| `--max-size <S>` | Exclude files larger than SIZE |
| `-b`, `--backup` | Backup updated files to `.syncz-backups` |
| `-d`, `--dry-run` | Preview changes with tree diff |
| `--no-perms` | Skip permission sync (useful for macOS/Linux) |

## How It Works

1. **Path Mapping** — Translates local paths to remote equivalents relative to `~`
2. **Auto mkdir** — Creates missing parent directories on the remote
3. **SSH Multiplexing** — Reuses connections via ControlMaster for speed
4. **Delta Transfer** — Only syncs what's changed
5. **Persistence** — Remembers the last successful host in `~/.syncz_state` for one-word syncing

## License

MIT
