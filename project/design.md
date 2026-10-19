<!-- markdownlint-disable MD007 -- Unordered list indentation -->
<!-- markdownlint-disable MD010 -- No hard tabs -->
<!-- markdownlint-disable MD033 -- No inline html -->
<!-- markdownlint-disable MD055 -- Table pipe style [Expected: leading_and_trailing; Actual: leading_only; Missing trailing pipe] -->
<!-- markdownlint-disable MD041 -- First line in a file should be a top-level heading -->
# Design

Draft. Nothing here is built yet. Goes with [backlog.md](backlog.md).

## Summary

tukfs is the file layer for tukzedo. It has 3 parts that share one Rust codebase:

- An integrity library and command-line tool.

- A per-user metadata daemon for desktop apps.

- A filesystem layer that remaps the legacy FHS into a newer folder layout.

All of them need to work with tukzedo-linux and lxqtukz, and on plain Debian and NixOS too.

## Integrity

- A Rust library crate, plus a CLI on top of it. It owns 3 things that are now only in tukfs-hash:
	- The database. `files` has one row per distinct content, and `paths` one row per name, pointing at a `files` row. It's the same split nemo-anywhere's cache uses.
	- The attribute format. `user.blake3.b64u`, `.bytes`, `.date`, `.mtime` and `.inode`, with names shared with nemo-anywhere so each can use what the other left.
	- The checksum logic. Reading with blake3, when a stored value can be trusted, and the rules for writing attributes back.

- tukfs-hash becomes a user of the library, and so does a later watch daemon. Moving the code out of tukfs-hash is the first real step.

- The database stays SQLite in one file, for the same reasons tukfs-hash gives: one writer keeps up with the disk, and a single file is easy to copy or snapshot.

## Metadata service

- A user-level daemon that indexes files for desktop apps such as Captain Nemo and tukzdesk. It answers with checksums, tags, and search results, so each app doesn't walk the disk and hash files on its own.

- It runs as the user, one per user, and reads the same database format as the integrity part.

- It's started on demand, and exits when idle.

### Transport

Proposed, and waiting on an OK before any code depends on it.

- A Unix domain socket, ideally speaking varlink.
	- systemd socket activation starts the daemon on first use.
	- `SO_PEERCRED` gives the uid of each client, so access is per user without any token scheme.
	- Large replies, like a folder's worth of checksums, stream over the socket without a size cap or a broker in the middle.
	- varlink has a small, typed interface file and a plain JSON wire format. Rust has crates for it, and `varlinkctl` can poke it from a shell.

- D-Bus only later, and only as a thin layer for small desktop signals such as "this file's tags changed".

- Other options looked at:

	| Option           | Why not first
	| :---             | :---
	| D-Bus            | Slow for bulk replies, and needs a session bus running. A headless box or a root shell may not have one.
	| FIFO             | One way only, and the daemon can't tell clients apart or answer each one.
	| Plain socket API | Would work, but varlink gives a typed interface and tooling for about the same cost.

## Folder layout

This is Phase III in tukzedo-linux's README.

- Shows a modern folder tree in place of the legacy FHS. The FHS is still there, but hidden from directory listings.

- Only listings are remapped. Any path typed or opened by name works, old or new, so every existing tool keeps working. If one doesn't, that's a bug.
	- ZFS's `.zfs` folder works the same way. It's missing from listings but there when named.

- Each user can switch views at runtime without logging out. New users see the new layout by default, and `root` sees the FHS.

- FUSE first. tukzedo-linux has a short C proof of concept in `source/poc_remap-fhs.c`, which only sketches the callbacks. The real version is Rust, on a FUSE crate.

- A kernel stacked filesystem later, if users like it. That part may need C, and would borrow from OverlayFS.

- It must be possible to turn it off completely, so nothing sits between programs and the real filesystem. When it's on, the overhead must be close to unmeasurable in controlled tests. Every change to it gets a benchmark against the bare filesystem.

## Language

- Rust for all 3 parts.

- C or C++ only where a part needs it. The kernel module is the one case known now.

- This matches tukzedo's other apps, which lead with Rust, and tukfs-hash, which is already Rust.

## Open questions

- Does tukfs-hash move into this repo as a crate in one workspace, or stay its own repo and depend on the library?

- The transport above needs an OK.

- Can `poc_remap-fhs.c` move here from tukzedo-linux?

- License. This repo is LGPL-2.1-or-later like tukzclip and lxqtukz, but tukfs-hash is GPL-2.0, and the proof of concept is GPL-2.0-or-later. A kernel module would need to be GPL-2.0 compatible.

- Where the daemon's index lives. It could share the integrity database or keep its own file with tags and search data beside it.

- What the new folder layout looks like, and where its map is configured.
