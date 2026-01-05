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

**REMINDER:** This is a pre-training repo. I'm using this to learn the prerequisites before building the real project: **Veil**.

---

## Project Status

**Current Phase:** Preparation (Learning & Prototyping)  
**Last Completed Topic:** Ensuring Reproducible Builds (Rust Book)

---

## What Is Veil?

Veil is a Linux command-line tool that runs applications inside a temporary filesystem sandbox. The core principle: if something isn't explicitly allowed, it doesn't exist to the app. Not blocked—literally invisible.

**Problem:** Desktop applications on Linux typically have unrestricted access to your home directory. Running untrusted binaries, installers, or scripts requires blind trust and exposes personal data.

**Solution:** Veil enforces least-privilege filesystem access by restricting what an application can see. Only directories you whitelist are visible. System stuff like `/usr` and `/lib` are read-only. When the app exits, the sandbox unmounts and leaves no trace.

The security model is "deny by invisibility" rather than reactive blocking. If it's not allowed, the filesystem just...doesn't show it.

---

## Why This Repo Exists

Honestly? I need to learn a bunch of stuff before attempting the real thing:

- Rust (ownership model, error handling, the whole deal)
- FUSE filesystem programming
- How to map live directories without breaking things
- Logging denied accesses
- Running processes inside a sandboxed environment

The code here is gonna be rough. That's the point "learning by breaking things, not building production software".

---

## Core Design Principles

1. **Deny by default** – Nothing is accessible unless explicitly allowed
2. **Filesystem visibility equals permission** – If you can't see it, you can't access it
3. **No system-wide impact** – Changes are temporary and isolated
4. **Stable behavior for GUI applications** – No freezing or blocking on permission prompts
5. **Explicit user control** – Clear, intentional permission grants

---

## How It Works

High-level flow:

1. User launches an application using Veil
2. Veil creates a temporary sandbox directory
3. A FUSE filesystem is mounted at the sandbox location
4. System directories (`/usr`, `/lib`, `/bin`, `/etc`) are exposed as read-only
5. One user-selected directory is exposed as read-write
6. `/tmp` is exposed as read-write for temporary files
7. The application runs inside the sandbox
8. Any denied access attempts are logged
9. On exit, sandbox is removed cleanly

**Filesystem View Inside Sandbox:**
- **Read-only:** `/usr`, `/lib`, `/bin`, `/etc`
- **Read-write:** `/tmp`, user-selected directory
- **Invisible:** Everything else

The key insight: don't block unauthorized access, just don't show it in the first place.

---

## Runtime Flow Example

```
$ veil run --allow ~/Documents krita
→ Creates FUSE mount
→ Mounts system dirs (read-only)
→ Mounts ~/Documents (read-write)
→ Mounts /tmp (read-write)
→ Launches krita in the sandbox via bubblewrap
→ krita tries to access ~/Pictures → denied (invisible)
→ Logs the attempt
→ krita exits
→ Unmount sandbox, clean slate
→ Post-run summary shows denied accesses with suggestions
```

This should work for GUI apps, system-installed programs, AppImages, and random scripts.

---

## Denied Access Handling

When an application tries to access a restricted path, the operation fails safely. Veil logs the denied access and provides a clear explanation after the application exits.

**Post-Run Suggestions:** Veil analyzes the logs and suggests which directories to allow if you want to rerun the application successfully.

---

## Tech Stack

- **Language:** Rust
- **Filesystem:** FUSE via `fuser` crate for userspace filesystem handling
- **Sandboxing:** `bubblewrap` for secure process isolation
- **Concurrency:** `DashMap` for thread-safe permission storage
- **Communication:** JSON over Unix sockets using `serde`
- **CLI Parsing:** `clap` for argument parsing
- **Logging:** `log` + `env_logger` for access tracking

**Smart Engineering Approach:** Veil uses battle-tested tools and libraries to avoid reinventing complex, error-prone components.

## Implementation Strategy

1. Start with a FUSE passthrough filesystem using `fuser`
2. Add allow and deny checks for paths
3. Store permissions using `DashMap` for thread safety
4. Run applications inside `bubblewrap` for isolation
5. Log denied accesses and generate summaries
6. Keep interactive permissions optional and CLI-only

## MVP Goals

What I'm trying to get working:

- Basic Rust CLI that doesn't crash immediately
- FUSE filesystem that actually mounts
- Passthrough for user-specified directories
- System directories visible (read-only)
- Logging when something gets denied
- Post-run summary with actionable suggestions
- Integration with `bubblewrap` for process isolation
- Clean mount/unmount without leaving zombie mounts
- Actually execute a process inside the sandbox

These are aspirational. Stuff might not work, might be half-baked, might get scrapped.

---

## Use Cases

- **Running untrusted binaries safely** – Test sketchy downloads without risk
- **Testing installers and scripts** – See what they actually try to access
- **Developer workspace isolation** – Keep project dependencies contained
- **Preventing accidental file corruption** – Limit blast radius of buggy tools

---

## Out of Scope

Not even attempting:

- GUI (terminal only)
- Policy engine or config files
- Malware detection or antivirus scanning
- Any kind of absolute security guarantees
- System-wide enforcement or persistent policies
- Integration with apt/dnf/pacman
- Replacing Flatpak/Snap
- Network isolation
- Seccomp filters
- Modifying file permissions

To be clear: this is filesystem visibility control. It's not antivirus. It's not a bulletproof sandbox. **Veil does not perform malware detection, antivirus scanning, or system-wide enforcement.**

---

## Optional Experiments

**Interactive permission mode (CLI only):** Veil includes an experimental interactive permission mode for command-line tools. This mode allows blocking file access and asking the user for permission at runtime.

Example: "Allow access to ~/whatever? [y/n]"

**Important limitations:**
- This mode is **disabled by default**
- This mode is **not supported for GUI applications**
- GUI apps freeze if you block them waiting for input, and nobody wants a frozen window
- GUI apps need their permissions configured upfront

This is an optional experiment and may not make it into the final implementation.

---

## Learning Roadmap

Rough plan of attack:

1. Get comfortable with Rust (syntax, ownership, error handling)
2. FUSE hello-world (mount a minimal read-only filesystem)
3. Directory passthrough (map real directories into the FUSE mount)
4. Process execution (spawn a subprocess inside the sandbox)
5. Denial logging (catch and record filesystem access attempts)
6. Dynamic allow (add directories while the app is running)
7. Cleanup (unmount properly, no zombie mounts)

## What You'll Learn

If I actually finish this thing, I should understand:

- **Rust fundamentals** — ownership, borrowing, lifetimes, error handling (the hard parts)
- **FUSE architecture** — how userspace filesystems actually work
- **Linux filesystem semantics** — mount points, visibility control, read-only mounts
- **Process sandboxing** — filesystem isolation, least privilege
- **Systems programming** — spawning processes, handling signals, cleaning up properly
- **Security models** — deny-by-default thinking, attack surface reduction
- **Logging** — tracking access, auditing denials

Basically all the prerequisite knowledge needed before attempting the real implementation.

---

## Future Work

Once the MVP is working, potential enhancements include:

- **Graphical interface** – Desktop app for easier permission management
- **Policy presets** – Pre-configured profiles for common use cases
- **Improved visualization of logs** – Better UI for understanding access patterns
- **Extended interactive permissions** – More sophisticated runtime permission handling for CLI tools

---

## Why This Fits FOSS Hack

Veil is fully open source, avoids proprietary APIs, uses standard Linux primitives correctly, and demonstrates disciplined systems engineering. It leverages battle-tested FOSS tools like FUSE and bubblewrap rather than reinventing critical security components.

---

## Disclaimer

**Security:** This provides zero security guarantees. Don't use it as a replacement for SELinux, AppArmor, or actual containers. Definitely don't rely on it to stop malware.

**Stability:** Pre-alpha quality. Might crash. Might corrupt data. Might leave ghost mounts. Don't run this on anything important.

**Purpose:** This is a learning exercise, not a product. Not intended for actual use.

---

## License

TBD.

---

<sub>*Logo generated with Gemini</sub>
