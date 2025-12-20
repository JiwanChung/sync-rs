# sync-rs

`sync-rs` is a fast Rust CLI that wraps `rsync` and `ssh` with smart path mapping and a zero-config flow. It mirrors paths relative to your home directory, auto-creates remote parents, and supports push/pull with dry-run previews.

## Features

- Smart home-relative path mapping across machines.
- Automatic remote `mkdir -p` for missing parents.
- Push (default) and pull (`--pull`) directions.
- Dry-run tree diff with transfer size (`-d`).
- SSH host picker from `~/.ssh/config`.
- Progress display and summary stats.
- Default excludes: `.git/`, `node_modules/`, `target/`, `.DS_Store`.

## Requirements

- `rsync` and `ssh` available in your PATH.
- Rust toolchain for building from source.

## Installation

### Build from source

```bash
cargo build --release
```

The binary will be at `target/release/sync-rs`.

### Install to your PATH

```bash
cargo install --path .
```

This installs `sync-rs` into Cargo's bin directory (typically `~/.cargo/bin`).

## Usage

Push local to remote (default):

```bash
sync-rs ~/projects/my-app my-host
```

Pull remote to local:

```bash
sync-rs --pull ~/projects/my-app my-host
```

Dry run preview:

```bash
sync-rs -d ~/projects/my-app my-host
```

If no host is provided, `sync-rs` will prompt you with a fuzzy list of entries from `~/.ssh/config`.

## Flags

- `--pull`: Pull remote to local.
- `-d`, `--dry-run`: Show a tree-style diff and transfer size.
- `--no-perms`: Skip permission syncing (useful across macOS/Linux).

## Notes

- Paths are mapped relative to your home directory (`~/projects/foo` -> `~/projects/foo` on the remote).
- If the destination parent does not exist, it is created before syncing.
- SSH multiplexing is enabled via ControlMaster for snappy repeated commands.
