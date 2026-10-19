<!-- markdownlint-disable MD007 -- Unordered list indentation -->
<!-- markdownlint-disable MD010 -- No hard tabs -->
<!-- markdownlint-disable MD033 -- No inline html -->
<!-- markdownlint-disable MD055 -- Table pipe style [Expected: leading_and_trailing; Actual: leading_only; Missing trailing pipe] -->
<!-- markdownlint-disable MD041 -- First line in a file should be a top-level heading -->
# Requirements

This is a product backlog just for pre-v1.0.0 release. After that, bugs, features, and enhancements will be managed in Github Issues.

<!-- TOC ignore:true -->
## Table of contents

<!-- TOC -->

- [Issues](#issues)
- [Template](#template)

<!-- /TOC -->

## Issues

Sorted per the notes in [Template](#template).

- Should tukfs-hash move into this repo as a crate, or stay separate?
	- ID: 2026101915280876
	- Type: Task
	- Status: Waiting for answers
	- Priority|Severity: High
	- Opened: 20261019-152808
	- Opened by: t00mietum
	- Related IDs: 2026101915280874
	- Requirements:
		- Pick one:
			- Move tukfs-hash into this repo, as one crate in a Cargo workspace beside the library.
			- Keep its own repo, and have it depend on the library.
		- Either way it keeps its name, its CLI, and its database format.
	- Note: tukfs-hash is GPL-2.0 and this repo is LGPL-2.1-or-later, so moving it also needs the license question answered (2026101915280884).

- Confirm the transport for the metadata service
	- ID: 2026101915280878
	- Type: Task
	- Status: Waiting for answers
	- Priority|Severity: High
	- Opened: 20261019-152808
	- Opened by: t00mietum
	- Related IDs: 2026101915280880
	- Requirements:
		- OK or change the proposal in `design.md` under Transport: a Unix domain socket speaking varlink, systemd socket activation, and `SO_PEERCRED` for per-user access.
		- D-Bus only later, as a thin layer for small desktop signals.
		- Once it's settled, move it from proposed to a decision in `design.md`.

- Move `poc_remap-fhs.c` here from tukzedo-linux
	- ID: 2026101915280882
	- Type: Task
	- Status: Waiting for answers
	- Priority|Severity: Avg
	- Opened: 20261019-152808
	- Opened by: t00mietum
	- Related IDs: 2026101915280886
	- Requirements:
		- Needs an OK first.
		- Move the file with its history notes, and point tukzedo-linux's Phase III section at this repo.
		- tukzedo-linux's backlog gets its own matching item when this goes ahead.

- License for tukfs and the parts it takes in
	- ID: 2026101915280884
	- Type: Task
	- Status: Waiting for answers
	- Priority|Severity: Avg
	- Opened: 20261019-152808
	- Opened by: t00mietum
	- Requirements:
		- This repo starts as LGPL-2.1-or-later, like tukzclip and lxqtukz.
		- tukfs-hash is GPL-2.0 and the FHS proof of concept is GPL-2.0-or-later. Pick what each becomes if it moves here.
		- A kernel module later has to be GPL-2.0 compatible.

- Shared library crate from tukfs-hash's database and attribute format
	- ID: 2026101915280874
	- Type: Feature
	- Status: Queued
	- Priority|Severity: High
	- Opened: 20261019-152808
	- Opened by: t00mietum
	- Related IDs: 2026101915280876
	- Requirements:
		- A Rust library crate, plus a small CLI on top of it.
		- It owns the database: the `files` and `paths` split, `runs`, and `events`, the same as tukfs-hash's current schema.
		- It owns the `user.blake3.*` attribute names and format shared with nemo-anywhere, and the rules for when a stored checksum can be trusted.
		- It owns the checksum code: blake3, and blake2b for rmlint's attributes.
		- tukfs-hash's tests still pass once it uses the library. Its database files open with no change.

- Metadata daemon MVP
	- ID: 2026101915280880
	- Type: Feature
	- Status: Queued
	- Priority|Severity: Avg
	- Opened: 20261019-152808
	- Opened by: t00mietum
	- Prereq IDs: 2026101915280874, 2026101915280878
	- Requirements:
		- Runs as the user, started on demand by a systemd user socket unit, and exits when idle.
		- Answers checksum lookups for a path or a folder, from the database or by reading the file.
		- Only answers clients with the same uid.
		- Tags and search come after this item, in their own items.

- Port the FHS remap proof of concept to Rust on FUSE
	- ID: 2026101915280886
	- Type: Feature
	- Status: Queued
	- Priority|Severity: Avg
	- Opened: 20261019-152808
	- Opened by: t00mietum
	- Related IDs: 2026101915280882
	- Requirements:
		- Remaps directory listings only. Any path opened by name works, old or new.
		- A per-user switch between the new layout and the FHS, at runtime.
		- Can be turned off completely, leaving nothing between programs and the real filesystem.
		- A benchmark against the bare filesystem, run on every change. The overhead needs to be close to unmeasurable.
		- Tested on a scratch folder first, then in a test VM, never over a live system path.
	- Note: the C proof of concept only sketches the callbacks, and doesn't build as is. Its `lookup` isn't a member of libfuse's high-level `fuse_operations`.

**Stop here for a next release**

- Kernel stacked filesystem for the folder layout
	- ID: 2026101915280888
	- Type: Feature
	- Status: Deferred
	- Priority|Severity: Low
	- Opened: 20261019-152808
	- Opened by: t00mietum
	- Prereq IDs: 2026101915280886
	- Requirements:
		- Only if test users like the FUSE version.
		- A loadable kernel module that stacks on the real filesystem, borrowing from OverlayFS.
		- Same rules as the FUSE version: fully off when disabled, and overhead close to unmeasurable.

## Template

- Notes:

	- Only use rows that you actually need or expect will be filled in. Always fill in the title, ID, Type, Status, Opened and Created by.

	- The ID is the local time to the hundredth of a second. Opened is when it was written down, which may differ. (Use a keyboard macro and possibly something like project 'zuid' to generate.)

	- Status values meaning: Testing means the fix is in and checks are running or still to run. Waiting on signoff means automated testing passed. Moot means something else changed that made it irrelevant. Canceled means it still applies but was decided against. Waiting for testing means the fix is in and waits on a long CI run or an outside test host. Can't reproduce means a real attempt to reproduce it failed.
		- An item with nothing left for a person to judge skips Waiting on signoff and goes straight to Done. `Clean up the backlog` in the directives says when.

	- As issues are worked, and statuses change, place them in correct sorting order within the list:
		- First by status: Waiting for answers, Waiting on signoff, Testing, Waiting for testing, Can't reproduce, Stalled, Started, Queued, Done, Deferred, Canceled, Moot
		- Then by severity|priority: Critical, High, Avg, Low
		- Then by type: Bugs, [not bugs together]

	- Rows marked [Bug] are for bugs only, and rows marked [Feature] for features and enhancements. Priority and Severity share one row and one scale. Priority is for a Feature or Enhancement, and Severity for a Bug. Children are not nested. They sit at the top level and point back with Parent ID.

Template:

- Title
	- ID: YYYYmmDDHHMMSSNN
	- Type: [Bug|Feature|Enhancement|Task]
	- Status: [Queued|Waiting for answers|Waiting on signoff|Waiting for testing|Started|Testing|Stalled|Can't reproduce|Moot|Canceled|Deferred|Done]
	- Needs local test suite run?:
	- Needs external testing:
	- Priority [Feature|Enhancement] | Severity [Bug]: [Critical|High|Avg|Low]
	- Opened:
	- Opened by:
	- Assigned to:
	- Parent ID:
	- Prereq IDs:
	- Related IDs:
	- Target OS:
	- Test environment:
	- Version and build:
	- Requirements  [Feature]:
		- Hierarchical bulleted list.
	- Steps to reproduce [Bug]:
		- …
	- Incorrect behavior [Bug]:
	- Expected behavior [Bug]:
	- Reproduced [Bug]: [No, or when, where and how]
	- Possible cause [Bug]:
	- Actual cause [Bug]:
		- …
	- Estimated effort: [High|Avg|Low]
	- Actual effort: [High|Avg|Low]
	- Progress log:
		- …
	- Decisions:
		- …
	- Actual fix [Bug]:
	- Branch:
	- Commit:
	- Test case: [Reason not applicable, or CI test case #]
	- Acceptance signoff:
	- Superseded by ID:
	- Closed:
