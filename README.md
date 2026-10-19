<!-- markdownlint-disable MD007 -- Unordered list indentation -->
<!-- markdownlint-disable MD010 -- No hard tabs -->
<!-- markdownlint-disable MD033 -- No inline html -->
<!-- markdownlint-disable MD055 -- Table pipe style [Expected: leading_and_trailing; Actual: leading_only; Missing trailing pipe] -->
<!-- markdownlint-disable MD041 -- First line in a file should be a top-level heading -->
# tukfs

The file layer of tukzedo. It keeps track of what's in your files, answers questions about them for desktop apps, and later can show the system's folders in a newer layout than the old Linux one.

## Parts

- **Integrity**: A Rust library and command-line tool that own the checksum database, the `user.blake3.*` attributes left on files, and the code that checks them. [tukfs-hash](https://github.com/tukzedo/tukfs-hash) and later tools build on it.

- **Metadata service**: A small per-user daemon that indexes files, with checksums, tags, and search, so apps like Captain Nemo and tukzdesk can ask it instead of each reading the disk on their own.

- **Folder layout**: A filesystem layer that shows a modern folder tree in place of the legacy FHS, while every old path still works. It starts as a FUSE filesystem. A kernel version may follow if people like it. It can be turned off completely, and when it's on the cost should be too small to measure.

## Status

Early. There's no code yet and nothing to install. The plan is in [project/design.md](project/design.md) and the work list in [project/backlog.md](project/backlog.md).

## Related projects

- [tukfs-hash](https://github.com/tukzedo/tukfs-hash) checksums every file on a box and finds ones whose contents changed while the size and time didn't. It's the first tool under tukfs, and its database and attributes are what the library here will start from.

- [tukzedo-linux](https://github.com/tukzedo/tukzedo-linux) is the distro. The folder layout is its Phase III, and the proof of concept for it lives there for now.

- [lxqtukz](https://github.com/tukzedo/lxqtukz) is the desktop. Its apps are the main users of the metadata service.

Each part should also work on plain Debian and NixOS, without the rest of tukzedo.

## Copyright and license

> Copyright © 2026 t00mietum (CryptogID: ปʬϝღถɔ4რఠΔթะ9ƾǝu)<br>
> Licensed under [GNU LGPL v2.1 or later](https://spdx.org/licenses/LGPL-2.1-or-later.html) license. No warranty. See `license.txt`.
