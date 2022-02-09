## Wed 09/Feb/2022

- System
  - [x] Solve kernel vs. admgpu problem
    - [x] Fight 5.4 kernel vs zfs
    - [x] Fight find compatible 5.11 kernel
- Studies
  - C64 ASM Programming
    - [ ] Machine Code Games Routines for the Commodore 64
      - [ ] 10. Indirect Routines
      - [ ] Appendix A: Utility Programs

## Tue 08/Feb/2022

- Tools
  - [x] Review Code Reading Club tools
    - [x] Review and report bug with official PDF maker
- SWE
  - Practice
    - Leetcode
      - [x] Medium 85%,  807: 1 tests, 1 submissions, 22.5'
- Studies
  - C64 ASM Programming
    - [x] Review and copy routines to project
      - [x] "Inverting"
      - [x] "Attribute Flasher"
        - [x] Fix, and add entry to blog post
      - [x] "Alternative Sprite System"

## Mon 07/Feb/2022

- SWE
  - Practice
    - Leetcode
      - [x] Medium 86%, 1302: 1 tests, 1 submissions, 4' (!)
- Studies
  - C64 ASM Programming
    - [x] Add other errata to blog
    - [x] Copy Joystick handling routine to project

## Sun 06/Feb/2022

- SWE
  - Practice
    - Leetcode
      - [x] Medium 86%, 1769: 2 tests, 1 submissions, 30'
        - +1 test with printout
        - mistake: one end cycle incorrectly considered

## Sat 05/Feb/2022

- Studies
  - C64 ASM Programming
    - [ ] Machine Code Games Routines for the Commodore 64
      - [ ] 10. Indirect Routines

## Fri 04/Feb/2022

- SWE
  - C64 ASM Programming
    - [ ] Machine Code Games Routines for the Commodore 64
      - [x] 9. Direct Routines

## Thu 03/Feb/2022

- SWE
  - Practice
    - Leetcode
      - [x] Medium 88%, 1689: 2 tests, 1 submissions, 6'
        - Wrong Ruby API called
      - [x] Medium 87%, 1828: 2 tests, 1 submissions, 13'
        - Wrong sign; tried to find fast solution (not provided!?)
      - [ ] Medium 86%, 1769: 4 tests, 1 submissions, >20'
        - Disaster: Mistaken requirements, then index issues, then slow; found fast solution towards the end
- Admin
  - [x] Review and discuss Rust Code Reading Club
- Studies
  - C64 ASM Programming
    - [ ] Machine Code Games Routines for the Commodore 64
      - [ ] 9. Direct Routines
        - [x] Line Blank (09.087)
          - Add to project
        - [ ] Keyboard/Joystick

## Wed 02/Feb/2022

- SWE
  - Practice
    - Leetcode
      - [x] Easy   43%,  203: 1 tests, 1 submissions, 15'
      - [x] Easy   43%,  367: 1 tests, 1 submissions, 13'
      - [x] Medium 88%, 1476: 2 tests, 1 submissions, 6'
        - Typo
- Tools
  - [x] Follow up file sharing bug
- Studies
  - C64 ASM Programming
    - [ ] Machine Code Games Routines for the Commodore 64
      - [ ] 9. Direct Routines
        - Add to project/blog
          - [x] 256 Byte Continuous Scroll (09.085)
            - [x] Fix bug
          - [x] Scroll Into Lower Memory (09.086)
    - [x] Review zero page addressing mode

## Tue 01/Feb/2022

- Tools
  - [x] Cosmetic and functional improvements to work computation script
  - [x] Investigate editors resource consumption
- SWE
  - Practice
    - Leetcode
      - [x] Easy 42%,   66: 2 tests, 1 submissions
        - Not considered array direction in result
      - [x] Easy 42%,   35: 1 tests, 2 submissions
        - Usual issue with `-1` referring to last array item, rather than none
- Studies
  - Ruby
    - [x] Search coincise way of safely addressing arrays respect to negative indexes
  - C64 ASM Programming
    - [ ] Machine Code Games Routines for the Commodore 64
      - [ ] 9. Direct Routines
        - Add to project/blog
          - [x] Delays (09.076)
          - [x] Fundamental Bomb Update (09.080)
            - [x] Fix bug, and add loop
          - [x] Hail of Barbs (09.081)
            - [x] Fix bug

## Mon 31/Jan/2022

- SWE
  - Practice
    - Leetcode
      - [x] Easy 42%, 1232: 3 tests, 3 submissions
        - Didn't consider one case + trickery due to FP inaccuracy
- Studies
  - C64 ASM Programming
    - [ ] Machine Code Games Routines for the Commodore 64
      - [ ] 9. Direct Routines

## Sun 30/Jan/2022

- Writings
  - Comment(s) on the apt lock post
    - [x] Investigate and propose possible/likely solution
  - Comment on the Linux kernel customization
    - [x] Review and reply
- Studies
  - Rust
    - [x] Review and extend `MaybeUninit` usage pattern
- Studies
  - C64 ASM Programming
    - [x] Research idiomatic Kick Assembler code
    - [ ] Machine Code Games Routines for the Commodore 64
      - [ ] 9. Direct Routines
        - [x] Fix bug
        - [x] Extend the small memory fill routine
          - [x] Optimize it
        - [x] Publish the routine errata and optimization on the blog
        - [x] Optimize the memory copy routine
        - [x] Investigate CLC+BCC <> JMP

## Sat 29/Jan/2022

- SWE
  - Practice
    - Leetcode
      - [x] Easy 42%,  263: 1 tests, 2 submissions
        - Mistakes with API
      - [x] Easy 42%,  263: 1 tests, 1 submissions
        - Alternative (faster) approach
      - [x] Easy 42%,  111: 2 tests, 1 submissions
        - Misunderstood the problem
      - [x] Easy 42%, 1736: disaster
        - Disaster; tried to find clever solution; the edge cases are multiple
- Tools
  - [x] Work accounting: Fix day type computation for `ye`sterday
- Writings
  - [x] Update "Machine Code Games Routines for the Commodore 64" Errata
    - Add new erratum, and comment routine
- Studies
  - C64 ASM Programming
    - [ ] Machine Code Games Routines for the Commodore 64
      - [x] Copy notes
      - [x] 6. Writing Your Own Routines
      - [x] 7. Testing and Debugging
      - [x] 8. Connections: Stringing the Game Together

## Fri 28/Jan/2022

- Tools
  - VSC
    - [x] Make Sort selection key binding conditional to selection
- Open source
  - VSCode Kick Assembler Studio extension
    - [x] Diagnose/workaround debugger bug
      - [x] Fight the bug
      - [x] Fight extension configurations
      - [x] Fight building the package
    - [x] Open upstream issue
    - [x] Create fork with workaround
- SWE
  - Practice
    - Leetcode
      - [x] Easy 41%, 1784: Confusing description
      - [x] Easy 41%,  645: 3 tests, 1 submissions
        - Inconsistency in the data stored; wrong storage referenced
      - [x] Easy 42%,  205: 1 tests, 1 submissions

## Thu 27/Jan/2022

- Tools
  - [x] Separate playground projects
- SWE
  - Practice
    - Leetcode
      - [x] Easy 40%,  278: 1 test, 3 submissions
        - Not considered that bad=1 is an edge case independently of n
- Studies
  - C64 ASM Programming
    - [x] Create book listings repository
    - [ ] Machine Code Games Routines for the Commodore 64
      - [ ] 6. Writing Your Own Routines
        - [x] Fight converting first routine
          - [x] Study Kick Assembler basics
          - [x] Try alternative debugger
          - [x] Review Kick Assembler Studio bug

## Wed 26/Jan/2022

- Tools
  - [x] Fight compile VICE
- Open source
  - [x] Contribute to official 5.13 amdgpu incompatibility report
- SWE
  - Practice
    - Leetcode
      - [x] Easy 40%,  290: 2 tests, 2 submissions
        - Assumed a constraint that there wasn't
      - [x] Easy 40%,  219: 1 tests, 1 submissions
- Studies
  - C64 ASM Programming
    - [ ] Machine Code Games Routines for the Commodore 64
      - [x] 5. Coding the Program
      - [ ] 6. Writing Your Own Routines

## Tue 25/Jan/2022

- SWE
  - Practice
    - Leetcode
      - [x] Easy 38%, 1037: 1 tests, 2 submissions
      - [x] Easy 38%,  434: 1 tests, 1 submissions
      - [x] Easy 38%,  680: 2 tests, 1 submissions
      - [x] Easy 39%, 1668: 1 tests, 2 submissions
- Tools/Open source
  - C64 ASM Programming
    - [x] Review again all the VSC extensions
    - [x] Open a few issues
      - [x] Investigate workaround for resource not loaded
    - [x] Investigate ASM hello world (launcher)
    - [x] Finalize VSC C64 ASM extension setup

## Mon 24/Jan/2022

- Open source
  - `vscode-open-in-github` plugin
     - [x] Diagnose and open issue about Markdown links not generated correctly
- SWE
  - Practice
    - Leetcode
      - [x] Easy 35%,  925: 2 tests, 1 submission, fancy logic
      - [x] Easy 35%, 1805: 1 tests, 1 submission
      - [x] Easy 36%, 1346: 3 tests (1 debug), 1 submission
      - [ ] Easy 36%,   28: 1 tests, 2 submissions (2*slow)
      - [x] Easy 36%,   58: 2 tests, 2 submissions
      - [x] Easy 36%,   69: 2 tests, 3 submissions (slow -> fancy logic)
        - Research Babylonian method
      - [ ] Easy 38%,  507: insane
- Open source
  - Telegram
    - [x] Follow up bug report

## Sun 23/Jan/2022

- SWE
  - Practice
    - Leetcode
      - [x] Easy 31%: 3 tests, 1 submission
      - [x] Easy 32%: 1 tests, 2 submissions
      - [x] Easy 33%: Solution looked up
      - [x] Easy 33%: 2 tests, 2 submissions

## Sat 22/Jan/2022

- System
  - [x] Fight keyboard layout icon not showing
- Tools
  - [x] Remote desktop switch script: Detect output port name
- SWE
  - Practice
    - [x] Leetcode: Attempt to solve in one submission, easy problems: 70%:4 60%:3 50%:4 30%:1
    - [x] Leetcode: Solve in one submission, medium problems: 37%:1

## Fri 21/Jan/2022

- Tools/Linux
  - [x] Find how to navigate zsh history
- Studies
  - JITs
    - [x] TenderJIT stream
- SWE
  - Practice
    - [x] Leetcode: Solve in one submission, easy problems: 10

## Thu 20/Jan/2022

- SWE
  - Practice
    - [x] Leetcode: A few easy problems

## Wed 19/Jan/2022

- SWE
  - Practice
    - [x] Leetcode: Several easy problems
- Studies
  - C64 ASM Programming
    - [ ] Setup vs64 environment

## Tue 18/Jan/2022

- Tools
  - [x] Try Coderpad
- SWE
  - Practice
    - [x] Solve problems: https://howigotjob.com/interview-questions/coderpad-interview-questions
    - [x] Leetcode: a few easy problems
- Studies
  - C64 ASM Programming
    - [ ] Machine Code Games Routines for the Commodore 64
      - ~~3. The Birth of a Game~~
      - [x] 4. Program Design
      - [ ] 5. Coding the Program

## Mon 17/Jan/2022

- Projects
  - QEMU-pinning
    - [x] Create wiki page with user-provided script
    - [x] Update README with reference to script, and add new details about project status
- Tools
  - [x] Follow up Windows VM setup, including ebooks conversion setup
- Studies
  - C64 ASM Programming
    - [x] Purchase/convert new book
    - [ ] Machine Code Games Routines for the Commodore 64
      - [x] Copy notes
        - [x] Research found errata
      - [x] First look at multiplication routines
- Writings
  - [x] "Machine Code Games Routines For The Commodore 64" Errata (WIP) article

## Cycle notes

Good cycle, although spent a lot of time on the iPad setup; also done a non-trivial amount of secondary stuff. Started systematic codebase reading for the first time; started systematic JIT studies.

- Unscheduled
  - Tools
    - Extend internal script
    - Update other internal script
      - Add less digits support
      - Add general inconsistencies support
    - Find how to remove all tags from audio files
    - create_vsc_project: Support operating on an existing project
  - Blog/Admin: Disable Disqus ads
  - Studies/Ruby: Study `un/pack`, and rewrite noted conversions
  - Openscripts: manage_bt: Always try to disable the BT device, not only when it was enabled
  - Reverse Engineering: Big review C64 emulation/assembly resources
  - Admin: Test Ipad pdf editing/viewing

## Sun 16/Jan/2022

- Admin
  - [x] Complete Ipad configuration
- Tools
  - [x] PDF pages removal
  - [x] Try many iOS PDF annotators
- Studies
  - C64 ASM Programming
    - [ ] Machine Code Games Routines for the Commodore 64
      - [x] 1. Introduction
      - [x] 2. Further 6510 Theory
  - JITs
    - [ ] PyPy architecture (from AOSA vol.2 book)

## Sat 15/Jan/2022

- Admin
  - [ ] Complete Ipad review and configuration
- SWE
  - Read codebases
    - [ ] [Zinc64](https://github.com/binaryfields/zinc64)
      - [ ] Some other types
- Studies
  - Rust
    - [x] `Cell` vs `RefCell`
  - C64 ASM Programming
    - [x] Review Run magazine
    - [x] Fight find ASM game-oriented programming books

## Fri 14/Jan/2022

- Tools
  - [x] create_vsc_project: Support operating on an existing project
- SWE
  - Read codebases
    - [ ] [Zinc64](https://github.com/binaryfields/zinc64)
      - First look
        - [x] Review and remove `ChipFactory`
        - [x] Trace `C64` components diagram
- Admin
  - [x] Test Ipad pdf editing/viewing
- Tools
  - [x] Fight Mermaid support for bidirectional issues
    - [x] Open issue
  - [x] Find alternative

## Thu 13/Jan/2022

- Reverse Engineering
  - [x] Post mentoring request
  - [x] Big review C64 emulation/assembly resources

## Wed 12/Jan/2022

- Low-level Programming
  - The Art of 64-Bit Assembly, Volume 1 (notes)
    - [x] Review and report another erratum
    - [x] Copy notes Chapter 14

## Tue 11/Jan/2022

- Studies
  - Ruby
    - [x] Study `un/pack`, and rewrite noted conversions
- Reverse engineering
  - [x] Write on codementor.io
- Projects
  - Openscripts
    - [x] manage_bt: Always try to disable the BT device, not only when it was enabled
- Low-level Programming
  - [ ] The Art of 64-Bit Assembly, Volume 1 (notes)
    - [x] Copy notes Chapter 12

## Mon 10/Jan/2022

- Tools
  - [x] Fix/update internal script
  - [ ] Review Ruby/VSC type checkers
    - [x] Review Sorbet/VSC extension
      - [x] Investigate Sorbet possible bug
    - [x] Review RBS/Typeprof-IDE VSC extension
  - [x] Extend internal script
  - [x] Update other internal script
    - [x] Add less digits support
    - [x] Add general inconsistencies support
  - [x] Find how to remove all tags from audio files
- Blog/Admin
  - [x] Disable Disqus ads
- Projects
  - Openscripts
    - [x] gitio: Add "short" functionality (protocol stripping)
      - [x] Minor cleanups
- Low-level Programming
  - [ ] The Art of 64-Bit Assembly, Volume 1 (notes)
    - [x] Copy notes Chapter 7
    - [x] Copy notes Chapter 10
    - [x] Copy notes Chapter 11

## Cycle

Skipped a day, but overall, completed the cycle.

- Skipped
  - TJ issue investigation (solved)

## Sun 09/Jan/2022

- Projects
  - TenderJIT
    - [x] Review new fixes PR
  - Openscripts
    - [x] Add `manage_bt` new script
- SWE
  - Career
    - [x] Readings
- Low-level Programming
  - [ ] The Art of 64-Bit Assembly, Volume 1 (notes)
    - [ ] Copy notes Chapter 7

## Sat 08/Jan/2022

- Databases
  - [x] Read [MySQL UUID performance](https://www.percona.com/blog/2019/11/22/uuids-are-popular-but-bad-for-performance-lets-discuss)

## Fri 07/Jan/2022

- SWE
  - Career
    - [ ] Readings
- Low-level Programming
  - [ ] The Art of 64-Bit Assembly, Volume 1 (notes)
    - [x] Copy notes Chapter 6
    - [ ] Copy notes Chapter 7
      - [x] Think simplified version of TJ register saving trampoline

## Thu 06/Jan/2022

- Tools
  - [x] Update file sharing start script to use double resume
- JITs/Interpreters
  - [x] Read [Ruby AOT Compiler announcement](https://sorbet.org/blog/2021/07/30/open-sourcing-sorbet-compiler)
  - [x] CPython Internals
    - [x] Read again The Evaluation Loop

## Wed 05/Jan/2022

- Tools
  - Work computation scripts: Holidays and new weekend handling
    - [x] Review available holiday gems
    - Set weekends/holidays as off by default (via `holidays` gem)
      - [x] Calendar addition script
      - [x] Hours computation script
  - [x] Yearly archival script
    - [x] Add daily
    - [x] Add Email client archival of all folders
      - [x] Investigate files and formats
- SWE
  - Career
    - [ ] Readings
- JITs/Interpreters
  - [ ] CPython Internals
    - [x] Parallelism and Concurrency
    - [x] Objects and Types
    - [x] The Standard Library
    - ~~The Test Suite~~
    - [x] Debugging
    - [x] Benchmarking, Profiling, and Tracing
    - ~~Next Steps~~
    - ~~Appendix: Introduction to C for Python Programmers~~
- Low-level Programming
  - [ ] The Art of 64-Bit Assembly, Volume 1 (notes)
    - [ ] Copy notes Chapter 6

## Tue 04/Jan/2022

- JITs/Interpreters
  - [ ] CPython Internals
    - [ ] Parallelism and Concurrency

## Mon 03/Jan/2022

- Projects
  - Openscripts
    - [x] convert_cb_archive_to_pdf: Fix output filename (remove generic `cb*` extension)
  - Schedule manager
    - [x] Replanner: Add support for `day+` (day plus one week)
  - Geet
    - [x] Allow skipping the selection menu for a given parameter (e.g. reviewers)
  - Blog
    - [x] add_new_book.sh: Update parameter value for skipping reviewers
- Low-level Programming
  - The Art of 64-Bit Assembly, Volume 1
    - [x] Report erratum on "SIMD String Instructions" chapter
    - [x] Add article to blog with links
- Other
  - [x] Publish MS laptop issues
- Admin
  - [x] Cycle planning

## Cycle

- Unscheduled
  - TenderJIT issue investigations
  - ASM book remaining chapters
  - Start CPython Internals
- Postponed
  - Large project: TJ/JITs study
  - Practice: Competitive programmer's core skill

## Sun 02/Jan/2022

- JITs/Interpreters
  - [ ] CPython Internals
    - [x] Memory Management

## Sat 01/Jan/2022

- Admin
  - Systems
    - [x] Workaround Btrfs bug on background scrubbing
- Tools
  - [x] Create script for archival previous year files, including brojournal
- Projects
  - Personal notes
    - [x] Restructure repository
  - TenderJIT
- JITs/Interpreters
  - [ ] CPython Internals
    - ~~Introduction~~
    - ~~Getting the CPython Source Code~~
    - ~~Setting Up Your Development Environment~~
    - [x] Compiling CPython (part)
    - ~~The Python Language and Grammar~~
    - ~~Configuration and Input~~
    - ~~Lexing and Parsing With Syntax Trees~~
    - [x] The Compiler (part)
    - [x] The Evaluation Loop (part)
    - [ ] Memory Management
