<table border="0">
<tr>
<td style="border: none;">
<img src="logo.png" alt="Veil Logo" width="120" style="border-radius: 5px;">
</td>
<td style="border: none;">
<h1>veil<sub> alpha</sub></h1>
</td>
</tr>
</table>

Learning project: building prerequisites for the real Veil implementation.

---

## Project Status

**Current Phase:** Preparation (Learning & Prototyping)  
**Last Completed Topic:** Ensuring Reproducible Builds (Rust Book)

---

## What Is Veil?

Linux CLI tool for running applications in a temporary filesystem sandbox. Denies access by making unauthorized paths invisible.

Applications get:
- Read-only access to system directories (`/usr`, `/lib`, `/bin`, `/etc`)
- Read-write access to whitelisted directories and `/tmp`
- No visibility of anything else

---

## Learning Goals

- Rust fundamentals (ownership, error handling, lifetimes)
- FUSE filesystem programming
- Directory mapping and process sandboxing
- Access logging and denial tracking

---

## Design Principles

1. Deny by default
2. Filesystem visibility = permission
3. Temporary, isolated changes
4. No blocking prompts for GUI apps
5. Explicit permission grants

---

## How It Works

1. User launches an app via Veil
2. Temporary sandbox directory created
3. FUSE filesystem mounted at sandbox location
4. System directories (`/usr`, `/lib`, `/bin`, `/etc`) exposed as read-only
5. User-selected directory and `/tmp` exposed as read-write
6. Application runs inside sandbox
7. Denied access attempts logged
8. On exit, sandbox removed

**Filesystem View:**
- **Read-only:** `/usr`, `/lib`, `/bin`, `/etc`
- **Read-write:** `/tmp`, user-selected directory
- **Invisible:** Everything else

---

## Runtime Flow Example

```
$ veil run --allow ~/Documents krita
→ Creates FUSE mount, mounts system dirs (read-only)
→ Mounts ~/Documents and /tmp (read-write)
→ Launches krita via bubblewrap in sandbox
→ krita tries to access ~/Pictures → denied (invisible), logged
→ krita exits → unmount sandbox
→ Post-run summary shows denied accesses with suggestions
```

Works with GUI apps, system programs, AppImages, and scripts.

---

## Denied Access Handling

Restricted access fails safely, gets logged, and shows in post-run summary with permission suggestions.

---

## Tech Stack

- **Rust** — Core language
- **FUSE** — `fuser` crate
- **bubblewrap** — Process isolation
- **DashMap** — Thread-safe permissions
- **serde** — Unix socket communication
- **clap** — CLI parsing
- **log + env_logger** — Access tracking

## Implementation Strategy

1. Start with a FUSE passthrough filesystem using `fuser`
2. Add allow and deny checks for paths
3. Store permissions using `DashMap` for thread safety
4. Run applications inside `bubblewrap` for isolation
5. Log denied accesses and generate summaries
6. Keep interactive permissions optional and CLI-only

## MVP Goals

Target functionality:
- Rust CLI with basic stability
- Working FUSE filesystem mount
- Passthrough for user-specified directories
- System directories visible (read-only)
- Denied access logging
- Post-run summary with actionable suggestions
- `bubblewrap` integration for process isolation
- Clean mount/unmount without zombie mounts
- Process execution inside sandbox

---

## Use Cases

- Run untrusted binaries safely
- Test installers and scripts
- Isolate developer workspaces
- Prevent accidental file corruption

---

## Out of Scope

Not implementing:
- GUI
- Config files/policy engine
- Malware detection
- System-wide enforcement
- Package manager integration
- Network isolation
- Seccomp filters

---

## Optional Experiments

**Interactive permission mode:** Runtime permission prompts for CLI tools only. Disabled by default. Not supported for GUI apps (causes freezing).

---

## Learning Roadmap

1. Rust fundamentals (syntax, ownership, error handling)
2. FUSE hello-world (minimal read-only filesystem)
3. Directory passthrough (map real directories into FUSE)
4. Process execution (spawn subprocess inside sandbox)
5. Denial logging (catch and record access attempts)
6. Dynamic allow (add directories at runtime)
7. Cleanup (proper unmount, no zombie mounts)

## Learning Outcomes

- Rust ownership, borrowing, lifetimes, error handling
- FUSE userspace filesystem architecture
- Linux mount points and visibility control
- Process sandboxing and isolation
- Systems programming: spawning, signals, cleanup
- Deny-by-default security models
- Access tracking and audit logging

---

## Future Work

- GUI for permission management
- Policy presets
- Better log visualization
- Extended interactive permissions

---

## Disclaimer

No security guarantees. Pre-alpha quality. Learning exercise only.

---

## License

TBD.

---

<sub>*Logo generated with Gemini</sub>
