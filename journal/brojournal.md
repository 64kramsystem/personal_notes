## Mon 28/Oct/2024

- Project: PM-Spotlight
  - [x] Make Windows-compatible

## Sat 17/Aug/2024

- Btrfs studies
  - [x] Review and expand basics
  - [x] Massive fight with snapshots

## Tue 04/Jun/2024

- Project: Openscripts
  - `build_kernel`
    - [x] build_kernel: Check destination directory
    - [x] build_kernel: Fix bug in find_latest_kernel_version()
    - [x] build_kernel: Add support for RCs

## Wed 29/May/2024

- Project: Openscripts
  - `upgrade_bluez_from_source`
    - [x] Always uninstall the current version
    - [x] Update only if the current version is not the latest.
    - [x] Attempt to perform a shallow repository clone
    - [x] Cosmetic cleanups
    - [x] Cache sudo

## Mon 29/Apr/2024

- Project: Openscripts
  - [x] `build_kernel`: Fix install_kernel_packages() not installing the libc package

## Wed 24/Apr/2024

- Project: QEMU-pinning
  - [x] Update patch for v8.2.3
  - [x] Update patch for v9.0.0

## Tue 16/Apr/2024

- Project: Openscripts
  - [x] Add `upgrade_bluez_from_source` script

## Mon 15/Apr/2024

- Project: Openscripts
  - [x] Add `winetemplate` script

## Wed 27/Mar/2024

- Project: PM Spotlight
  - Emoji search
    - [x] Add blowing kiss emoji
    - [x] search: Sort emoji entries
    - [x] search: Add/expand icon pattern comments
    - [x] search: Remove character `!` from `relaxed` key

## Thu 21/Mar/2024

- Project: Schedule manager
  - [x] Copy fixed time when explicit

## Sat 16/Mar/2024

- Project: Openscripts
  - [x] `build_kernel`: Fix logic for config download

## Fri 15/Mar/2024

- Project: QEMU-pinning
  - [x] Update patch for v8.2.2
- Project: Schedule manager
  - [x] Drop Ruby 3.0 support
  - [x] Don't apply interpolations on :skip
  - [x] Add Ruby 3.3 support
  - [x] Add Ruby HEAD (3.4) support
- Project: PM Spotlight
  - [x] File search: Allow ampersand (`&`)
  - [x] Minor refactoring + cosmetic fix

## Wed 13/Mar/2024

- Writings
  - [x] "Upgrading the Bluetooth (Bluez) stack on Ubuntu" quick article

## Mon 11/Mar/2024

- Project: Openscripts
  - [x] `build_kernel`: Fix print commits diff for new minor version

## Wed 28/Feb/2024

- Scripts
  - Rsync mirroring script
    - [x] Add `--no-delete` support
  - Swing content sync script
    - [x] Add `to` support
- Project: Openscripts
  - `build_kernel`
    - [x] Print commits diff with previous version
    - [x] Download ubuntu module for config, if available
    - [x] Add support for reverting a commit

## Tue 27/Feb/2024

- Scripts
  - Music handling scripts
    - [x] Handle symlinks
    - [x] Add check for reference file MIME type
    - [x] Investigate current player bug
    - [x] Replace player
- Project: Openscript
  - `control_music_player`
    - [x] Send player not found error message to stderr
    - [x] Add support for Audacious
    - [x] Cosmetic improvements

## Wed 21/Feb/2024

- Project: openscripts
  - [x] Add `tag_mp3s_bpm`
  - [x] Desnapped: Add arbitrary operations
    - [x] Also: Minor cosmetic fixes
- Sysadmin
  - [x] Investigate exit status codes for `systemctl status`
  - [x] Replace remote desktop solution in installation script
  - [x] Make email client restart synchronous

## Tue 20/Feb/2024

- Sysadmin
  - [x] Replace remote desktop solution (setup on server)

## Tue 09/Jan/2024

- Project: openscripts
  - KernelVersion improvements
    - [x] 3952fbf kernel_version.rb: Cosmetic changes/improvements
    - [x] 888b8bd Rename KernelVersion.parse_version to .parse_uname_version
    - [x] 1e5caf9 Add KernelVersion#parse_tag_version
    - [x] 20c607c KernelVersion#find_latest: Use and return KernelVersion instance(s)
      - Effect: clean_kernel_packages now recognizes equality in all uname <> tag versions

## Wed 03/Jan/2024

- [ ] Sysadmin: Change remote desktop program
  - [x] Test and compare
  - [x] Update server script
  - [x] Also: write client script
  - [ ] Update system install script
    - [x] DE hotkey
- Project: Spreadbase
  - [x] Gemspec: Cosmetic (whitespace) improvement
  - [x] Gemspec: Relax `bigdecimal` requirement
  - [x] Move `rexml` gem requirement to the Gemspec (and constrain it)
  - [x] Release new gem version

## Tue 02/Jan/2024

- Project: Spreadbase
  - [x] CI: Test pushes only on master
  - [x] Add explicit `bigdecimal` dependency to gemspec
  - [x] CI: Add new Rubies (head, 3.3)
  - [x] Convert all options handling to keyword arguments
  - [x] Update Required Ruby version to 3.0
  - [x] CI: Disable fail fast
  - [x] Update CI actions
  - [x] Upgrade RSpec to 3.12.0
