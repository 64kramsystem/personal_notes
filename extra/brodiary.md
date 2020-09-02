## Wed Sep/02

- Extra
  - [x] Switch to alternative Android sync service, after Dropbox disaster

## Tue Sep/01

- Emulation/Rust
  - Scheduler
    - [x] Implementation of a fast enough scheduler (insane)
    - [ ] Readings
      - [ ] https://matklad.github.io/2020/01/02/spinlocks-considered-harmful.html

## Mon Aug/31

- Emulation/Rust
  - Scheduler
    - [ ] Think scheduler
    - [x] Playground: improve, reorganize, and review
    - [x] Experiments with async
    - [x] Cleanup the async experiments, merge, and conclude
  - [ ] Theory
    - [ ] Study [Emulation of Nintendo Game Boy](https://git.io/JUtp6)
  - Misc (Project)
    - [x] Review and merge playground
    - [x] Rename the CHIP-8 component to system

## Sun Aug/30

- Extra
  - [x] Add missing testing resources to awesome_rust
    - [x] Fix PR
  - [x] Move `brodiary.md` to `personal_notes`
  - [x] Discuss issue with rust-analyzer

- Emulation/Misc
  - [x] Discord
    - [x] Fight setup discord
    - [x] Ask question about idling
    - [x] Threading discussion
  - [x] Investigate Atari 2600 emu documentation/find alternative
    - Alternatives
      - Space invaders, Pacman, Gameboy, GBC, Sega master system and Sega Genesis
      - Apple-II
      - NES
      - GB/NES
      - SMS
  - [x] Reorganize resources

- Emulation/Rust
  - [x] Inbetween tasks/cleanups
    - [x] Improve naming
      - [x] Think names
      - [x] Rename
      - [x] Update `README.md`
    - [x] Integrate playground!
    - [x] Move CHIP-8 references to wiki
    - [x] All libraries: uniform error handling: either `unwrap()` or return `Result<(), Box<dyn Error>>`
    - [x] Fight rust-analyzer nested project
      - [x] participate discussion: https://github.com/rust-analyzer/rust-analyzer/issues/3265
  - [x] Scheduler/Concurrency theory
    - [x] Decide what next #1
    - [x] Investigate idling without sleep
      - concepts: HLT, HPET, TSC
      - pause concept
        - https://stackoverflow.com/questions/12894078/what-is-the-purpose-of-the-pause-instruction-in-x86
        - https://stackoverflow.com/questions/7086220/what-does-rep-nop-mean-in-x86-assembly-is-it-the-same-as-the-pause-instru
    - [x] Experiment further with sleep/sync implementation
      - Conclusion: Atomic data types are very fast; follow up on building the machine, and think about communication when it's time.
  - Game Boy
    - [ ] Theory
      - [x] Review what to study, and dump to offline format
      - [ ] Study [Emulation of Nintendo Game Boy](https://git.io/JUtp6)

## Sat Aug/29

- Emulation/Rust
  - [ ] Scheduler/Concurrency theory
    - [x] Research designs/performance #1
    - [x] Ask how to trivially write filler operations
    - [x] Find concurrency resources
    - [ ] Reads
      - [x] https://rust-lang.github.io/async-book

## Fri Aug/28

- Emulation/Rust
  - Emulators design/concurrency theory
    - [x] https://arstechnica.com/gaming/2011/08/accuracy-takes-power-one-mans-3ghz-quest-to-build-a-perfect-snes-emulator/
    - [x] https://byuu.net/design/schedulers/
    - [x] https://forums.nesdev.com/viewtopic.php?f=3&t=8999
    - [x] https://news.ycombinator.com/item?id=3987660
  - [ ] Scheduler/Concurrency theory
    - [x] Research designs/performance #1
    - [x] Interesting APIs (take note): Barrier/Condvar
    - [ ] Reads
      - [x] https://blogs.msmvps.com/peterritchie/2007/04/26/thread-sleep-is-a-sign-of-a-poorly-designed-program
      - [x] Review sleep in higan scheduler
  - [ ] Atari 2600
    - [ ] Study "Making Games for the Atari 2600"
      - [ ] Few chapters
  - [ ] Scheduler/Concurrency theory
    - [ ] Test async approach
      - [ ] Implement async simple scheduler
        - [x] First attempt, with sleep

- Extra
  - Method chaining challenge from colleague

## Thu Aug/27

- Emulation/Rust
  - Inbetween tasks/cleanups
    - [x] Plan next
      - [x] Machine (or WASM) -> Atari 2600
      - [x] Books -> Making Games for the Atari 2600
    - [x] Check if can contribute to https://github.com/tobiasvl/awesome-chip-8
  - [ ] Atari 2600
    - [ ] Study "Making Games for the Atari 2600"
      - [ ] Chapter 01

- Extra
  - [ ] Add missing testing resources to awesome_rust

- Rust/study
  - [x] Study testing chapter in Rust reference
    - [x] Fight find testing frameworks, and review (-> ruspec)
  - [ ] Find how to get Rust review
    - [x] General find
      - https://www.quora.com/Where-can-I-find-experts-to-code-review-my-GitHub-code

- ZFS
  - [x] Review and reply encryption feature request

## Wed Aug/26

- Emulation/Rust
  - [x] CHIP-8
    - [x] Add documentation where missing
    - [x] write README
    - [x] release

- Blog
  - Test Inkscape for diagrams

- Extra
  - Visual Studio Code launch configuration
    - Make Rust (extensions) handle a project inside another project
  - Update/structure personal notes about sdl2/rust sound
  - Follow up RSpec issue investigation
  - Forward WxWidgets technical explanation

## Tue Aug/25

- Emulation/Rust
  - CHIP-8
    - [x] Sound interface
      - [x] Rip out previous (misunderstood) beep logic
      - [x] Debug audio timer
      - [x] Improve max speed logic
      - [x] Test
    - [x] Review overall interfaces structure/organization
      - [x] Review
      - [x] Merge commits
      - [x] Rename DEVICE_FREQUENCY to AUDIO_DEVICE_FREQUENCY
      - [x] Review branch history warnings/errors
      - [x] Split `interfaces` into modules: `audio`, `video`, `logging`, `events`
      - [x] Review imports (curly braces etc.)

- Extra
  - Visual Studio Code launch configuration
    - [x] Think/setup best option for a playground package
    - [x] Configure multiple launch configurations
    - [x] Fight launch specific target keyboard shortcut

## Mon Aug/24

- Emulation/Rust
  - CHIP-8
    - [x] Final cleanups
      - [x] remove NullLogger, and use `Option<Logger>`
      - [x] don't draw screen twice!
    - [ ] Sound interface
      - [x] Think
      - [x] Implement
      - [x] Wire

## Sat Aug/22

- Emulation/Rust
  - CHIP-8
    - [x] Allow screen resize
      - [x] Make window `resizable()`; respond to event in `poll_event()`, and use that in `init()`
      - [x] Manually center canvas
    - [ ] Final cleanups
      - [x] Experiment: Pixel tuple struct

## Fri Aug/21

- Emulation/Rust
  - CHIP-8
    - [x] Implement new algorithm for capping
    - [x] Massive speedup via storing pixel in RGB format
    - [ ] Final cleanups
      - [x] move `set_keys` after `emulate_cycle`

## Thu Aug/20

- Emulation/Rust
  - CHIP-8
    - [x] Add "max speed" mode
    - [x] Refactorings/cleanups
      - [x] replace sdl_context with event_pump
      - [x] split modules instead of using `include!`
      - [x] (failed attempt) EventCode: embed boolean in the KeyCode enums
      - [x] (failed idea): `sdl_to_io_frontend_keycode`: make it enum method (!)
      - [x] Merge wait_keypress and poll_event
      - [x] Pass Quit reference around, rather than returning the value
      - [x] Review `target_texture` purpose
    - [x] fix (game) bugs
        - [x] flightrunner: `PS` doesn't show in `OOPS` anymore
            - backup and remove capping logic
        - [x] wait_keypress doesn't map
        - [x] tombstontipp: closing the window doesn't have any effect
          - wait event steals events from poll events
          - [x] think solution
    - [x] fix refactoring bug
      - [x] tombstone keys are not correct

## Wed Aug/19

- Extra
  - [x] Blog
    - [x] Check out diagram help applications
  - [x] Emulation
    - [x] (Fight) write tool for interactively linting a branch's history

- Emulation/Rust
  - CHIP-8
    - [ ] Refactorings/cleanups
      - [x] Clean errors and (unreported) warnings in history
      - [x] Complete: Review default implementations
    - [x] UI/Frontend improvements
      - [x] implement audio interface
        - [x] implement chip8 beep and test (instruction: `LD ST`)
          - [x] implement speed factor
    - [ ] fix (game bugs)
        - [ ] tombstontipp: closing the window doesn't have any effect
          - wait event steals events from poll events
          - additionally, wait_keypress discards key released events!
        - [ ] flightrunner: `PS` doesn't show in `OOPS` anymore
          - due to screen capping
          - [x] think solution

## Tue Aug/18

- Emulation/Rust
  - CHIP-8
    - [ ] UI/Frontend improvements
      - [x] Allow keys remapping
        - [x] test
      - [x] (Correctly) implement window maximizing and scaling
        - Fix unnecessary clear screen inefficiency
      - [ ] implement audio interface
        - [x] Mad find how to generate beep
        - [x] Fight ceil() function
        - [x] Find pretentious alternative formula
        - [ ] implement chip8 beep
    - [x] Refactor IoInterface: remove draw_pixel
    - [x] Fight Texture/Creator lifetime

## Mon Aug/17

- Emulation/Rust
  - CHIP-8
    - [x] libchip8: Refactoring: Use tuple unpacking for decoding instructions
    - [ ] UI/Frontend improvements
      - [x] Allow user to close the window (and halt the emulator)
        - [x] improve naming
      - [x] Cap display updates to 60 Hz
      - [ ] Allow keys remapping
        - [x] implement
        - [ ] test
    - [ ] Test ROMs/fix bugs
      - [x] flightrunner (crashes after a few runs) -> confirmed rom bug

## Sun Aug/16

- Emulation/Rust
  - CHIP-8
    - [x] Review and hack in video scaling
    - [x] libchip8: Simplify draw sprite routine, and add wrap around
    - [x] Test ROMs/fix bugs (round 1)
      - [x] octojam5title
      - [x] mondrian
      - [x] flightrunner
      - [x] br8kout
      - [x] horseyJump
      - [x] octopaint (unsupported: XO-CHIP)
      - [x] nokiatemplate (runs, although not double checked)
      - [x] ghostescape (runs, but not double checked)
      - [x] 1dcell
      - [x] octojam4title
      - [x] octojam1title
      - [x] sweetcopter (unsupported: Super-CHIP 1.1)
      - [x] octojam3title
    - [ ] UI/Frontend improvements
      - [x] Speed investigation
      - [ ] Allow user to close the window (and halt the emulator)
        - [x] base implementation
    - Quick look at wasm, and source code of related project

## Sat Aug/15

- Extra
  - Scripting
    - Create `brody`
  - Open source
    - Submit PR with shebang addition to CHIP-8 ROMs disassembler project

- Emulation/Rust
  - CHIP-8 base (SDL)
    - [x] implement SDL video/key interface
    - implement remaining chip8 stuff
      - [x] wire `IoFrontend`
      - [x] draw_graphics
      - [x] set_keys
      - [x] instruction 0xFX0A (wait keypress)
      - [x] Think/plan testing/debugging
      - [x] Implement variable screen resolution, and hires mode
      - [x] Fix CALL
      - [x] Implement debug mode
        - fight dynamic traits
        - fight clap
      - [x] Implement debug logs (ASM symbols: http://devernay.free.fr/hacks/chip8/C8TECH10.HTM)
      - [ ] Test via step-in into easy roms, in size order

## Fri Aug/14

- Extra
  - QEMU: build 5.1

- Emulation/Rust
  - CHIP-8 base (SDL)
    - [x] review SDL
      - [x] draw pixels: https://stackoverflow.com/q/20579658
      - [x] (method discarded): draw textures (pixel-based): https://dzone.com/articles/sdl2-pixel-drawing
    - [x] think/implement video/key interface
    - [ ] implement SDL video/key interface

## Thu Aug/13

- Emulation/Rust
  - [ ] review SDL
    - [x] fight event pump
    - [x] fight compiler red herring
    - [x] write improved base program

## Wed Aug/12

- Emulation/Rust
  - CHIP-8
    - implement clock
      - [x] timers
  - [ ] review SDL
    - [x] check guides
      - https://lazyfoo.net/tutorials/SDL
    - [ ] review Rust Programming By Example

- Extra
  - [x] Upgrade Libreoffice; test boot time and check how to improve it

## Tue Aug/11

- Emulation/Rust
  - CHIP-8
    - implement clock
      - [x] research Rust time handling
      - [x] main loop

## Mon Aug/10

- Emulation/Rust
  - [ ] CHIP-8 implementation
    - Base: http://www.multigesture.net/articles/how-to-write-an-emulator-chip-8-interpreter
      - [x] implement all the opcodes (except 0xFX0A)
    - [Gameloop](https://dewitters.com/dewitters-gameloop) + clock speed
      - [ ] implement clock (~500 Hz; can set no instantiation) + timers

## Sun Aug/09

- Rust: Study
  - Hands-On Data Structures and Algorithms in Rust
    - [ ] Section 2
      - Merge sort (alternative implementations)

## Fri Aug/07

?

## Thu Aug/06

- Emulation/Rust
  - [ ] CHIP-8 implementation

## Wed Aug/05

- Emulation/Rust
  - [ ] Start CHIP-8 implementation
    - Refence: http://www.multigesture.net/articles/how-to-write-an-emulator-chip-8-interpreter

- Rust
  - Practice
    - Experiment/fight with some ownership concepts (using Clap)

- Extra
  - Find/Review Gitlens extension

## Tue Aug/04

- Rust: Study
  - The Rust Programming Language (completed)
    - [x] Copy #19: Advanced features

- Blog
  - [x] Add Rust book to bookshelf

- Extra
  - [x] Jekyll/Bash scripting

- Emulation
  - Big learning material review and planning
  - [ ] http://www.multigesture.net/articles/how-to-write-an-emulator-chip-8-interpreter
    - [x] http://en.wikipedia.org/wiki/CHIP-8#Virtual_machine_description

## Mon Aug/03

- Rust: Study
  - The Rust Programming Language
    - [x] Copy #15: Smart pointers
    - [x] Copy #16: Concurrency
    - [x] Copy #17: OO Programming
    - [x] Copy #18: Patterns

## Sun Aug/02

- Rust: Study
  - The Rust Programming Language
    - [x] Copy #12: Project (until skipped)
    - [x] Copy #13: Iterators
    - [ ] Copy #15: Smart pointers
  - Hands-On Data Structures and Algorithms in Rust
    - [x] Section 1 (skipped large part)
  - [x] Fight method chaining on iterators

- Extra
  - Udemy
    - Handle courses/accounts
  - Blog
    - Add AWS course to bookshelf

## Fri Jul/31

- Rust: Study
  - The Rust Programming Language
    - [x] Copy notes #10: Generics, lifetimes

- QEMU-pinning
  - [x] Port patch to 5.1.0-rc2

- Rust: Practice
  - [Ferrous systems teaching exercises](https://ferrous-systems.github.io/teaching-material)
    - [x] [Multithreaded mailbox](https://ferrous-systems.github.io/teaching-material/assignments/multithreaded-mailbox.html)
    - Closing; not interested in the other two exercises
    - Review/copy exercise interesting APIs (if any)

## Thu Jul/30

- Rust: Practice
    - [x] [Connected mailbox](https://ferrous-systems.github.io/teaching-material/assignments/connected-mailbox.html)

## Wed Jul/29

- Rust: Practice
  - [Ferrous systems teaching exercises](https://ferrous-systems.github.io/teaching-material)
    - [ ] [Connected mailbox](https://ferrous-systems.github.io/teaching-material/assignments/connected-mailbox.html)
- Rust: Study
  - The Rust Programming Language
    - [ ] Copy notes #10: Generics, lifetimes

## Tue Jul/28

- Rust
  - Practice
    - [Ferrous systems teaching exercises](https://ferrous-systems.github.io/teaching-material)
      - [x] [Redisish protocol parser](https://ferrous-systems.github.io/teaching-material/assignments/redisish.html)
        - Move to independent repository
        - Review full solution, and convert to library
      - [x] [TCP server](https://ferrous-systems.github.io/teaching-material/assignments/tcp-echo-server.html)
      - [x] [TCP client](https://ferrous-systems.github.io/teaching-material/assignments/tcp-client.html)
  - Study
    - [x] Copy notes #7: Packaging

## Mon Jul/27

- Rust
  - Practice
    - Rustlings
      - One exercise
      - Closing (~60 exercises)
    - Ferrous systems teaching exercises
      - Redisish protocol parser: implementation and tests
        - todo: review full solution, and convert to library

- Work: Fight Ubuntu keyserver

## Sun Jul/26

- Work: AWS
  - Ultimate AWS Certified Cloud Practitioner - 2020
    - [x] Completed remaining sections
    - [x] Completed test exam

## Sat Jul/25

- Work: AWS
  - Ultimate AWS Certified Cloud Practitioner - 2020
    - [x] Sections: 20

## Fri Jul/24

- Work: AWS
  - Ultimate AWS Certified Cloud Practitioner - 2020
    - [x] Sections: 12, 13, 15, 16

## Thu Jul/23

- Work: AWS
  - Ultimate AWS Certified Cloud Practitioner - 2020
    - [x] Section 08-09, 17, 19, 21, 10-11
    - [x] Register and setup certification account

## Wed Jul/22

- Work: AWS
  - Ultimate AWS Certified Cloud Practitioner - 2020
    - [x] Sections 05-07

- Rust
  - Practice
    - Rustlings: couple of exercises

- Extra
  - File VSC search/replace bug

## Tue Jul/21

- Rust
  - Study
    - The Rust Programming Language **top**
      - [x] Complete advanced chapter

- Work: AWS
  - Ultimate AWS Certified Cloud Practitioner - 2020
    - [x] Sections 01-04

- Extra
  - openscripts
    - `encode_to_m4a`: Add support for (file) symlinks

## Mon Jul/20

- Rust
  - Study
    - The Rust Programming Language **top**
      - [x] Study concurrency chapter
      - Start advanced chapter

- Extra
  - Reset Dropbox shares

## Sun Jul/19

- Extra
  - Scripting

## Sat Jul/18

- Blog: Add Jekyll support for MySQL filtered RSS (2)

- Try Dropbox alternative (https://is.gd/rirV2L)
  - [x] sync: no linux
  - [x] pCloud: problem with docs editing on android
  - [x] zoho docs: no android
  - [x] box.com (10gb): no linux
  - [x] cloudup: invite only
  - [x] spideroak: no free
  - [x] mediafire
  - [x] workzone: no free
  - [x] nextcloud: trash
  - [x] cloudme: poor android app
  - [x] sugarsync: no free
  - [x] teamdrive.com (2gb): terrible android app
  - [x] mega.nz

## Fri Jul/17

- Rust
  - Study
    - The Rust Programming Language
      - Completed reading; skipped some chapters; must copy some
  - Practice
    - Ferrous systems: https://ferrous-systems.github.io/teaching-material
      - [x] [Files, match and Results](https://ferrous-systems.github.io/teaching-material/assignments/result-option-assignment.html)

- Blog: Add MySQL filtered RSS
  - [x] investigate/fix clean jekyll warnings
  - [x] sync gem versions with github pages
  - [x] investigate feasibility (see https://github.com/jekyll/jekyll-feed)
  - [x] fight display icon

- Extra
  - Find temporary Dropbox alternative (-> pCloud) for desktop/tablet

## Thu Jul/16

- Rust
  - Study
    - The Rust Programming Language
      - Follow up
  - Scripting
    - Few updates to `compute_hours`
  - Practice
    - Ferrous systems: https://ferrous-systems.github.io/teaching-material
      - FizzBuzz

- Extra
  - Rust
    - Fight nightly strip binaries

## Wed Jul/15

- ZFS
  - Review fork (limit: Wed/15)

- Rust
  - Study
    - The Rust Programming Language
      - Follow up

- Spreadbase
  - Review and merge PR (limit: Fri/17)

- Extra
  - Find out usable books to use for Rust game development

## Tue Jul/14

- Rust
  - Study
    - The Rust Programming Language
      - Partial #12: Project
      - Start #13: Iterators/closures
  - Practice
    - Rustlings

## Mon Jul/13

- Rust
  - Study
    - The Rust Programming Language
      - Complete #7: Packaging

## Sun Jul/12

- Emulation
  - Coursera: Build a Modern Computer from First Principles: From Nand to Tetris (I)
    - Project #6: Complete -> Course completed

## Sat Jul/11

- Emulation
  - Coursera: Build a Modern Computer from First Principles: From Nand to Tetris (I)
    - Project #6: WIP

- Extra
  - Review Rust study/practice plans

## Fri Jul/10

- Emulation
  - Coursera: Build a Modern Computer from First Principles: From Nand to Tetris (I)
    - Study #6

## Thu Jul/09

- Emulation
  - Coursera: Build a Modern Computer from First Principles: From Nand to Tetris (I)
    - Complete #5
    - Project #5

## Wed Jul/08

- Emulation
  - Coursera: Build a Modern Computer from First Principles: From Nand to Tetris (I)
    - Study #4
    - Project #4
    - Start #5

- ZFS installer
  - Review and discuss proposed feature(s)

## Tue Jul/07

- Emulation
  - Coursera: Build a Modern Computer from First Principles: From Nand to Tetris (I)
    - Project #3

- Extra
  - Replace defective RAM module
  - Review/ask further hardware books

## Mon Jul/06

- Extra
  - New phone
    - Signal test
    - Investigate O/S and rooting

## Sun Jul/05

- Emulation
  - Coursera: Build a Modern Computer from First Principles: From Nand to Tetris (I)
    - Complete Project #2
      - Fight issue with `ALU-nostat.hdl`, and minor other
    - Study #3

- Extra
  - Some Perl

## Sat Jul/04

- Emulation
  - Coursera: Build a Modern Computer from First Principles: From Nand to Tetris (I)
    - Complete project #1
    - Study #2

- Extra
  - Solve dependency issues LLVM repository and i386 packages
  - Try Firefox containers

## Fri Jul/03

- Emulation
  - Coursera: Build a Modern Computer from First Principles: From Nand to Tetris (I)
    - Study #1
    - Start project #1

- Extra
  - Review and reply blog comment
  - Coursera whole setup for studying

## Thu Jul/02

- Rust
  - Practice
    - Rustlings
  - The Rust Programming Language
    - Complete #10 (Generic types, traits, and lifetimes)

- Emulation
  - Computer Architecture: A Quantitative Approach
    - Close #1
    - Start #2
  - Extra: find new book, current is not fit for the purpose

## Wed Jul/01

- Rust
  - The Rust Programming Language
    - Complete #9 (error handling)
    - Start #10

## Tue Jun/30

- Rust
  - The Rust Programming Language
    - Follow up #9 (error handling)

- ZFS
  - Add Linux Mint 20 support

- Extra
  - Port QEMU patch to 4.2.1

## Sun Jun/28

- Emulation
  - Computer Architecture: A Quantitative Approach

## Sat Jun/27

- Rust
  - fork, clean, and submit Rust programming by example source code

## Fri Jun/26

- Rust
  - Rust programming by example
    - Follow up #3

## Mon Jun/22

- Rust
  - Scripting
    - Completed hours compute script

## Sun Jun/21

- Rust
  - Rust programming by example
    - Complete #1, #2
  - Scripting
    - Write working hours compute script
      - Review argument parsing in Rust
      - Fight borrow checker

- Extra
  - Test vivaldi (https://help.vivaldi.com/article/manual-setup-vivaldi-linux-repositories)

## Mon Jun/15

- Extra
  - Blog
    - Completed article on WFs/CTEs; took also few days during last week

## Sun Jun/07

- Rust
  - Study #8
  - Start #9

- Rust/Golang
  - Review/plan books and practice

- Extra
  - Think future article about apt/snaps/ppas
  - Read index dives article

## Sat Jun/06

- Golang
  - Powerful Command-Line...
    - Complete copy and review #4
    - Copy and review #5, #6

- Extra
  - Scripting: Fix tomatoes script

## Fri Jun/05

- Extra
  - Blog
    - Whole day research on user-defined variables problem

## Tue Jun/02

- Golang
  - Powerful Command-Line...: Complete the remaining chapters -> ?

## Fri May/29

- Golang
  - Read article on (XKCD) CLI application
  - Powerful Command-Line...: (Finish) study #4

## Thu May/28

- Golang
  - Powerful Command-Line Applications in Go: Copy and review #3
  - Exercism: One exercise
    - Fight parallelism and multiple arrays

- Extra
  - ZFS
    - Diagnose reported problem, and create minimal test case
    - Improve disks wiping, by using the zpool functionality
    - Ask ML resizing safety
    - Review zvols, and reply on Launchpad
  - delete_in_batches
    - review, and send analysis to project

## Wed May/27

- Golang
  - Powerful Command-Line Applications in Go: Copy and review #2
  - Research FlagSet

- Extra
  - Mirror and create PPA for [ppastats](https://fosspost.org/tutorials/how-to-get-the-download-stats-of-any-ubuntu-ppa)
  - ZFS: Close issue, with PR `Warn about "-O devices=off" already set in the tweaks dialog`
  - ZFS: Investigate pool corruption issue and propose solution
  - Desktop: Correct MIME types association

## Tue May/26

- Extra
  - PPA: Push scripts to new repository
  - Blog: Complete and release PPA article
  - PPA: Minor tweaks to published scripts

## Mon May/25

- Extra
  - PPA: Complete scripting infrastructure

## Sun May/24

- Golang
  - Review gccgo (+alias)
  - Study Study: Powerful Command-Line Applications in Go, until Chapter 3

## Sat May/23

- Golang
  - Re-read chapter 16
  - Study chapter 19

- Extra
  - Browser scripting
    - Rewritten logic; hopefully final version.
  - Books
    - Fight Dropbox not syncing tablet book(s)
  - Fight tablet SMB connection
  - Fight display sleep on laptop
  - Write article on display sleep

## Fri May/22

- Extra
  - PPA
    - Expand and prepare tooling for internal production usage
    - Add cowbuild option

## Thu May/21

- Rust
  - Review/copy notes chapter 6/beginning 8
  - Rustlings

- Extra
  - PPA
    - Update script
    - Review and reorganize documentation
  - Scripting
    - Write daily notes template script

## Wed May/20

- Rust
  - Review/copy notes chapter 5

- Extra
  - Exercism discussion

## Tue May/19

- Extra
  - (Mad) PPA Ruby packaging

## Mon May/18

- Extra
  - ZFS
    - Reply email
  - (Mad) PPA Ruby packaging

## Sat May/16

- Rust
  - Review notes chapter 4

- Extra
  - System
    - Correct xbacklight support on laptop
    - Finally solve issue with backlight

## Fri May/15

- Rust
  - Several Rustlings exercises

- Extra
  - ZFS installer
    - Review and fix issue

## Thu May/14

- Rust
  - Several exercism exercises
  - Review resources for following up practice

- Extra
  - Fix/extract repositories sync script

## Wed May/13

- Rust
  - One Exercism exercise (~1h)

- Extra
  - Update exercism management script (~30')
    - Workaround submission "playgrounded" scripts
    - Automatically patch Rust test file(s)

## Tue May/12

- Rust
  - Resume studies

- Extra
  - Update exercism management script (~1.5h)
    - Fight bash inherit_errexit

## Mon May/11

- Rust
  - Resume studies

- Extra
  - Scripts
    - `notify_travis_builds`: Make compatible with Ruby 2.7
  - Geet
    - First Ruby 2.7 upgrade work
    - Add Ruby 2.7 and HEAD to build
  - Bash
    - Nightmare bash RC files

## Sun May/10

- Extra
  - Review/copy Git notes (`log`)

## Sun May/09

- Extra
  - System
    - Solve permanently screen turning off spontaneously after issuing `xset dpms force off` 

## Sat May/08

- Extra
  - System
    - Update desktop to Ubuntu 20.04
    - Handle VFIO on 20.04
    - Update vga-passthrought project to 20.04

## Fri May/07

- Extra
  - System
    - Move to new password manager
    - Fight NxServer issue; report

## Wed May/06

- Extra
  - ZFS
    - Complete shortcut script for fast O/S installation
  - QEMU-pinning
    - Test v5.0.0 patch
  - Firefox scripting
    - Add support for slack:// protocol

## Tue May/05

- Extra
  - ZFS
    - Prepare animated demo
    - Make raid type user selectable
    - Reorganize pools creation/custom script execution
    - Add script for fast O/S installation via partition snapshot

## Mon May/04

- Extra
  - Dconf/Gsettings review
  - ZFS installer
    - Manage RAIDZ contribution
  - Find strategy for capturing video and encoding it, with skips

## Sat May/02

- Extra
  - Minor shell scripting

## Thu Apr/30

- Extra
  - Finalize Firefox launcher script
  - 20.04/laptop
    - First look of S3 client alternatives
    - Fight video crashing
    - Fight set default/preferred application

## Wed Apr/29

- Extra
  - Upgrade laptop to Ubuntu 20.04
  - Write super-annoying Firefox launcher script

## Tue Apr/28

- Extra
  - ZFS installer
    - Review PR, issue
    - Cool mkfifo solution for not logging confidential information
    - Debug desktop env not logged
    - Add new logging
    - Release new installer version, and reply issue
  - QEMU pinning
    - Updated patch; required updates (needs testing)

## Mon Apr/27

- Extra
  - Ruby (notes/review): ampersand prefix operator
  - Date filtering extension to internal Trello management tool
  - System: move passwords to new password manager
  - System: big update email client

## Sun Apr/26

- Extra
  - ZFS installer
    - Improve UX (and code) for the passphrase configuration
    - Considerable fight against Ubiquity 20.04
    - Insane amount of overall work, including final support for Ubuntu Desktop/Server 20.04
- Copy some linux notes; experiment with resizing/partitions

## Sat Apr/25

- Extra
  - ZFS installer: lots of work, mostly related to Ubuntu 20.04

## Fri Apr/24

- Extra
  - Ubuntu 20.04 preparation, part 2
  - ZFS installer
    - Close 1 issue (confidential information in the log)
    - Add extra logging
      - Fight variables in sudo env
    - Fix shellcheck warnings
    - PRs management/review
  - Copy some Linux system notes, reviewing Apt stuff

## Thu Apr/23

- Mentoring
  - 1 iteration

## Tue Apr/21

- Rust
  - Study chapters 5, 6

- Mentoring
  - 3 iterations
  - 1 solution

## Mon Apr/20

- Golang
  - Experiment/copy notes remaining chapter 16 (Concurrency)

- Mentoring
  - 9 solutions

- Extra (≈1h)
  - Several reviews on the Goby project
  - Review clever regex for finding sequences

## Sun Apr/19

- Golang
  - Re-read chapter 16
  - Study chapter 17
  - Copy notes chapter 8, 17; half chapter 16

- Extra
  - Tweaks to broser script + test multiple windows matching behavior with xdotool

## Fri Apr/17

- Golang
  - Review and copy notes chapters 7, 9
  - Study chapter 8, 9

- Extra
  - Ruby: review process execution strategies

## Thu Apr/16

- Golang study
  - Study chapters 5, 6, 7
    - Copy notes chapters 5, 6
  - 2 exercises exercism

- Mentoring
  - 2 iterations
  - two exercism discussions

- Extra
  - Bash: exact filenames cycling, with complex text, and handling inside function
  - Review Golang exercise website options
    - Register gophercises

## Wed Apr/15

- Golang study
  - Copied/reviewed chapter 4

- Mentoring
  - Discussion about ratings
  - 6 iterations

- Extra
  - checked out the Zeus source code for signals/restart support
  - tested spring in explicit mode

## Tue Apr/14

- Extra
  - MySQL `REGEXP_SUBSTR()` and review regex capabilities
  - Bash: for/while + IFS (fight while)
  - Check out Golang GUI libraries/book

- Mentoring
  - Around 15 iterations (≈1h)

## Mon Apr/13

- Golang study
  - Complete chapters 2,3
    - Merged previous other book notes
  - Exercises
    - Bubble sort

- Extra
  - Preparation installation Ubuntu 20.04 (#1)

## Sun Apr/12

- Rust study
  - Complete chapter 4
    - Copy notes, including previous chapters
  - Fight exercism exercise

- Mentoring
  - Around a dozen exercism iterations
  - Week total: 35

- Extra
  - Big refactoring and extension of the Exercism script

## Sat Apr/11

- Rust study
  - New APIs, from exercises
  - 1 exercism exercise
  - Follow up chapter 3, start chapter 4

- Mentoring
  - 3/4 Bash iterations

- Extra
  - Expand Exercism script: open and submit

## Fri Apr/10

- Rust study
  - Little study
  - Exercise: fight `&str` vs. `to_string()`

## Thu Apr/09

- Rust study
  - Copy book notes
  - Follow up chapter 3
  - 1 exercism exercise

- Extra
  - ZFS installer: increase boot pool size to 768M

- Mentoring
  - ≈5 Ruby iterations
  - 1 Bash iteration

## Wed Apr/08

- Rust study
  - put book concepts into small program
  - 1 exercism exercise

- General notes
  - Flush remaining pending notes (Bash/Linux tools)

- Extra
  - Unf#ck Slack email links
  - Blog post about Unf#cking Slack email links (part 1)
    - Minor review of Bash regexes

## Tue Apr/07

- Mentoring
  - 7/8 Ruby iterations

- General notes
  - copy/review some Bash subjects, mainly process substitution
  - copy minor Linux tools/Ruby subjects

## Mon Apr/06

- General notes
  - copy/reorganize several Linux subjects

- Rust study
  - followed up chapter 2

- Mentoring
  - Review four Ruby solutions

- Extra
  - Make browser scripts put Firefox window in front when launching from Thunderbird
  - Add custom browser desktop application, and associate it

## Sun Apr/05

- Exercises: Golang
  - Many exercism exercises
    - Spent considerable time on a regex-based exercise 
    - Spent time on byte-level strings (/slices) handling

- Mentoring
  - Review several (Ruby) solutions

- Extra
  - Discussions on Exercism Slack channels

## Sat Apr/04

- Mentoring
  - Review several solutions

- Extra
  - Emergency potential malware on Chromium
    - Move from Chromium to Firefox (lots of work)
  - VSC snippets

## Fri Apr/03

- Study
  - Golang: several codewars kata
  - Golang: three exercism

- Mentoring
  - Several solutions (with an interesting concept found in one)

## Thu Apr/02

- Study
  - Golang: 3 codewars kata

- https://mast.queensu.ca/~peter/inprocess/sumofcubes.pdf
  - `sqrt(sum) = (n² + n) / 2`
- https://www.tiger-algebra.com/drill/n~2_n-200=0
  - `An2+Bn+C = 0 -> (-B ± sqrt(B²-4AC)) / 2A`

## Wed Apr/01

- Extra
  - think plan Exercism; review Codewars

- Waste
  - 15' wiki lino banfi/franco e ciccio

## Tue Mar/31

- Mentoring
  - Five solutions/iterations

- Study
  - Copy notes: Golang
  - Copy notes: Rust

- Extra
  - Bash: one exercise (left track)
  - Copy notes: generic
    - tested and reorganized Linux parallel tools

## Mon Mar/30

- Mentoring
  - Some solutions
  - Long reply on glob expansion
  - Difficult regex on one

- Study
  - Golang: few exercises
    - One took a very long time

- Extra
  - Bash: one exercise
    - Review substring functionalities
  - Bash: research glob expansion
  - Try vscode golang debug

- Waste
  - Many minor distractions over the day
  - HN 30' reply

## Sun Mar/29

- Study
  - Golang: a couple of exercisms
- Mentoring
  - Several solutions, including one complex and long-standing
- Extra
  - Written script for managing exercism
  - Bash: a couple of exercisms

## Sat Mar/28

- Study
  - Golang: 1 chapter
  - Rust: one exercism
- Mentoring
  - A complex and long-standing solution

## Fri Mar/27

- Scripting
  - Trello/notify_travis_builds
- Exercism
  - A few solutions
  - Started a snippets file with common concepts/replies
- Notes
  - Copied/moved a few Bash ones
- Studies
  - First chapter golang
  - One exercism golang

- Extra/waste
  - Install golang 1.14
  - Review Codewars with Stan
  - Additional exercism session
  - Update libreoffice