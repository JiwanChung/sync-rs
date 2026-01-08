# AGENT.md: `syncz` Project Specification

## 1. Executive Summary

`syncz` is a high-performance Rust CLI designed to bridge the gap between local development and remote environments. It abstracts the complexity of `rsync` and `ssh` into a "zero-config" experience, automatically mapping paths, ensuring remote directory existence, and resolving hosts.

---

## 2. Core Functional Requirements

### 📂 Smart Pathing & Symmetry

The tool maps paths relative to the user's home directory across different environments.

* **Path Mapping:** `~/projects/xx` locally maps to `~/projects/xx` on the remote, regardless of the absolute path difference (e.g., `/Users/name` vs `/home/name`).
* **Auto-Creation:** If the destination parent directory does not exist, `syncz` will automatically create it using `--mkpath` or a remote `mkdir -p` command.
* **Trailing Slash Normalization:** The CLI automatically handles trailing slashes to ensure the **directory itself** and its **contents** are synced consistently without user micro-management.
* **File Support:** Detects if the input is a single file and adjusts the `rsync` command to avoid creating a directory shell around it.

### 🔑 Host Resolution & Connectivity

* **Discovery:** If no host is provided, parse `~/.ssh/config` and present a fuzzy-finder list.
* **Snappiness:** Utilize **SSH ControlMaster** (multiplexing) where available to reuse existing connections, reducing handshake latency for "check-then-sync" operations.

### 🔄 Sync Directions & Dry Run

* **Bidirectional (Default):** Local <-> Remote (Newer Wins via sequential `-u` passes).
* **Push (`--push`):** Local -> Remote.
* **Pull (`--pull`):** Remote -> Local.
* **Dry Run (`-d`):** Displays a tree-style diff and transfer size for all active sync directions.

---

## 3. Technical Constraints & Critical Handling

| Scenario | Strategy |
| --- | --- |
| **Missing Remote Parent** | Execute `mkdir -p` via SSH before rsync or use `--mkpath`. |
| **OS Permission Clashes** | Use `-avz` but allow `--no-perms` to avoid macOS/Linux UID/GID conflicts. |
| **Interrupted Transfers** | Use `--partial` and `--inplace` to allow resumes on large files. |
| **Bloatware/Logs** | Default ignore patterns: `.git/`, `node_modules/`, `target/`, `.DS_Store`. |

---

## 4. UI/UX Design

* **Interactive Selection:** Use `dialoguer` for host selection.
* **Visual Feedback:** Use `indicatif` for a multi-line progress bar showing current file and overall percentage.
* **Summary:** On completion, show total bytes sent, speed, and duration.

---

## 5. Implemented Features

* [x] **CLI & SSH Discovery:** `clap` based arguments and `~/.ssh/config` parsing for host selection.
* [x] **Smart Pathing:** Automatic logic for home-directory translation and remote `mkdir -p` triggers.
* [x] **Rsync Execution:** Wrapping the system `rsync` with optimized flags (`-avzP`).
* [x] **Dry Run Visualizer:** Parsing `--dry-run` output into a clean human-readable tree.
* [x] **Performance:** SSH ControlMaster (multiplexing) enabled for fast connection reuse.
* [x] **Bidirectional Sync:** Default mode uses sequential rsync passes with `--update` for a "Newer Wins" strategy.

---

## 6. Future Roadmap

### 1. Watch Mode (`--watch`)
*   **Goal:** Provide a "live sync" experience for development loops.
*   **Strategy:** Use `notify` crate to watch local filesystem events.
*   **Behavior:** Debounce rapid events and trigger a `Push` sync automatically.

### 2. Advanced Filtering
*   **Goal:** Give users granular control over what gets synced.
*   **Refinement:**
    *   `--all`: Disable default smart excludes (sync `node_modules`, `target`, etc.).
    *   `--gitignore`: Parse and respect `.gitignore` files.
    *   `--max-size <SIZE>`: Exclude large artifacts.

### 3. Safety & Conflict Handling
*   **Goal:** Prevent accidental data loss during bidirectional syncs.
*   **Strategy:**
    *   `--backup`: Enable `rsync --backup` to save overwritten files to a timestamped directory (e.g., `~/.syncz/backups`).
    *   **Conflict Reporting:** improved logging when files are updated on both sides (requires state tracking or complex heuristics).

---

### Implementation Detail: The "mkdir" Strategy

To ensure the remote path exists without slowing down the app, the CLI will execute:
`ssh <host> "mkdir -p <remote_parent_path>"`
immediately followed by the `rsync` command. If multiplexing is enabled, this happens over the same socket in milliseconds.