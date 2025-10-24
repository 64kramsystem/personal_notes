## Thu 23/Oct/2025

- Scripts
  - `encode_swing_videos`: Change target height when a video is less than the default (720p)

## Mon 20/Oct/2025

- Openscripts
  - `xi`: Add support for passing a string (via args)

## Sun 19/Oct/2025

- Openscripts
  - `encode_to_h265`
    - [x] Add rotation handling
    - [x] Improve scale factor
- Scripts
  - `add_swing`: Full `to` functionality

## Wed 27/Aug/2025

- Scripts
  - [x] Fix FFmpeg build gcc/g++ (version) use
  - [x] Build FFmpeg using mold
  - [x] Purchases script
    - [x] Refactor
    - [x] Add date functionality

## Wed 20/Aug/2025

- Scripts
  - [x] Modernize comics adder

## Tue 19/Aug/2025

- Scripts
  - [x] Update calls to `download_ubuntu_packages` (now a binary)
  - [x] Stab at creating a library for adder scripts
  - [x] Create game adder

## Sat 09/Aug/2025

- Scripts
  - [x] Add `--english` to `whisperc`

## Thu 07/Aug/2025

- Internal scripts
  - [x] Swing videos encoder: Add output dir functionality

## Wed 06/Aug/2025

- lpim
  - [x] Final fix to locking issue under WSL/Windows

## Mon 04/Aug/2025

- Openscripts
  - [x] Kernel-related scripts
    - [x] Fix issue when comparing versions with and without ongoing release
    - [x] KernelVersion: Add support for optional kernel type to UNAME_VERSION_REGEX

## Thu 31/Jul/2025

- Openscripts
  - [x] Add WSL support to `mk_invoice`

## Fri 18/Jul/2025

- Schedule manager
  - [x] Make interpolations more flexible
  - [x] Add past days interpolation

## Sat 12/Jul/2025

- Openscripts: `encode_to_h265`
  - [x] Move input files to a temp dir before encoding
  - [x] Remove "force" functionality, and clean output files in case of premature exit instead
  - [x] Support custom extension
- Internal scripts
  - [x] Simplifications based on `encode_to_h265` updates

## Fri 11/Jul/2025

- [x] Internal Email client API: Fix Windows process detection
- Openscripts
  - [ ] Convert `download_ubuntu_packages` to Crystal

## Wed 18/Jun/2025

- Schedule manager
  - [x] Remover: Improve an error message
  - [x] Remover: Add UT to document bug

## Fri 13/Jun/2025

- [x] add_comic: Handle evaluations starting with `-`
- [x] add_brogramming: Open file with `code -a` instead of `xo`

## Wed 28/May/2025

- [x] Build FFmpeg with latest AV1 encoders
- [ ] FFmpeg+AV1 encoders optimization

## Tue 27/May/2025

- Openscripts
  - MySQL tools
    - [x] mystart: Minor refactorings
    - [x] mystop: Refactoring
    - [x] mystop: Improve process killing logic
    - [x] mystart/mystop: Directly use the MySQL configuration to gather the directories

## Mon 26/May/2025

- Schedule manager
  - [x] Reworker: Add UT for case when the insertion point is not found
  - [x] Replanner: Implement pending UT
- Personal scripts
  - [x] `add_brogramming`: Don't add day entry, if it already exists
- System installer
  - [x] Send local FFmpeg on a diet
  - [x] Optimize FFmpeg
  - [x] Move built FFmpeg to local binaries
  - [x] Add helper to prepare a git repo with the latest tag checked out
  - [ ] Build and integrate into ffmpeg: libaom and SVT-AV1
    - https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu#libaom
    - https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu#libsvtav1
- Openscripts
  - [x] `git_merge_file_commits`: Add non-interactive mode

## Fri 23/May/2025

- Packaging
  - [x] (Failed) attempt to build `libaom3` deb package on Noble, from the Questing official Launchpad project
  - [x] Build libaom from source
  - [x] Build SVT-AV1 from source
- Personal scripts
  - [ ] Integrate libadom + SVT-AV1 in ffmpeg build

## Wed 21/May/2025

- Personal scripts
  - [x] Upgrade system script: Fix Linux Python programs installation

## Tue 13/May/2025

- Personal scripts
  - Personal repos sync
    - [x] Improve reporting
    - [x] (Hopefully) fix the issue with hanging sync processes
- Schedule manager
  - [x] Reworker: Simplified LPIM_GENERATOR string interpolation
  - [x] Reworker: Add day of week to lpim generated string

## Sun 27/Apr/2025

- Schedule manager
  - [x] Fix build failure
- Personal scripts
  - [x] `encode_swing_video`: Ensure that the temp file is deleted (if left over)

## Fri 25/Apr/2025

- Personal scripts
  - [x] Write script to add an entry to the gym file
    - Wasted almost 2 hours on regexes

## Fri 25/Apr/2025

- Openscripts
  - `texerak`: Add output to JPEG functionality

## Thu 24/Apr/2025

- Openscripts
  - [x] `texerak`: Improve temporary files handling

## Wed 23/Apr/2025

- Schedule manager
  - [x] Replanner: Print moved lines in :skips_only mode
  - [x] Reworker: Simplify the approach to the lpim command insertion
- Project: QEMU-pinning
  - [x] Update patch for v10.0.0

## Sat 22/Mar/2025

- Openscripts
  - [x] `encode_to_h265`: Add procs mapping for 16 cores
  - [x] `encode_to_h265`: Make segment start/end optional

## Fri 28/Feb/2025

- Openscripts
  - [x] `encode_to_m4a`: Trash files (when requested) instead of deleting them

## Wed 26/Feb/2025

- Openscripts
  - [x] Add `add_repo_key`

## Wed 22/Jan/2025

- Openscripts
  - [x] `encode_to_h265`: Make segment start/end optional

## Wed 15/Jan/2025

- Openscripts
  - [x] `encode_to_h265`: Add support for overwriting output files
  - [x] `encode_to_h265`: Add cut support
  - [x] `encode_to_h265`: Remove "append extension" functionality
  - [x] `encode_to_h265`: Simplify output files naming

## Thu 09/Jan/2025

- VMWare Workstation kernel compatibility patches
  - [x] Port to v17.6.2 (but double check)
