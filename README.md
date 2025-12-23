<h1 align="center">sync-rs</h1>

<p align="center">
  <strong>Zero-config file sync between local and remote machines</strong>
</p>

<p align="center">
  <a href="https://github.com/JiwanChung/sync-rs/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="License"></a>
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
- **Bi-directional** — Push (default) or pull with `--pull`
- **Dry-run Preview** — See a tree-style diff before syncing
- **SSH Host Picker** — Fuzzy-select hosts from `~/.ssh/config`
- **Progress Display** — Real-time transfer progress with summary stats
- **Sensible Defaults** — Auto-excludes `.git/`, `node_modules/`, `target/`, `.DS_Store`

## Installation

```bash
cargo install --git https://github.com/JiwanChung/sync-rs
```

> Requires `rsync` and `ssh` in your PATH.

## Quick Start

```bash
# Push to remote
sync-rs ~/projects/my-app my-server

# Pull from remote
sync-rs --pull ~/projects/my-app my-server

# Preview changes first
sync-rs -d ~/projects/my-app my-server

# No host? Get a fuzzy picker
sync-rs ~/projects/my-app
```

## Usage

```
sync-rs [OPTIONS] <PATH> [HOST]
```

| Option | Description |
|--------|-------------|
| `--pull` | Pull from remote to local (default is push) |
| `-d`, `--dry-run` | Preview changes with tree diff |
| `--no-perms` | Skip permission sync (useful for macOS/Linux) |

## How It Works

1. **Path Mapping** — Translates local paths to remote equivalents relative to `~`
2. **Auto mkdir** — Creates missing parent directories on the remote
3. **SSH Multiplexing** — Reuses connections via ControlMaster for speed
4. **Delta Transfer** — Only syncs what's changed

## License

MIT
