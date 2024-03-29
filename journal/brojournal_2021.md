## Fri 31/Dec/2021

- Projects
  - Spreadbase
    - [x] Design and implement Rubyzip 3.0 compatibility
      - [x] Relase gem
  - TenderJIT
    - [x] Try to reproduce more stably and isolate the [segfault/conditions](https://github.com/tenderlove/tenderjit/issues/110)
      - [x] Write discussion
    - [x] Discuss possible other bug found
- Low-level Programming
  - [ ] The Art of 64-Bit Assembly, Volume 1
    - [x] 12: Bit Manipulation
    - [x] ~~13: Macros and the MASM Compile-Time Language~~
    - [x] 14: The String Instructions
      - [x] Read [CMOV vs. branching Torvalds discussion](https://yarchive.net/comp/linux/cmov.html)
    - [x] ~~15: Managing Complex Projects~~
    - [x] ~~16: Stand-Alone Assembly Language Programs~~
- SWE
  - [ ] Career readings

## Thu 30/Dec/2021

- Projects
  - Openscripts
    - [x] convert_cb_archive_use_trash: On deletion, use trash if available
- Low-level Programming
  - The Art of 64-Bit Assembly, Volume 1
    - [ ] 12: Bit Manipulation

## Wed 29/Dec/2021

- Low-level Programming
  - The Art of 64-Bit Assembly, Volume 1
    - [x] 7: Low-Level Control Structures
    - [x] 8: Advanced Arithmetic (part)
    - [x] ~~9: Numeric Conversion~~
    - [x] 10: Table Lookups
    - [x] 11: SIMD Instructions (part)

## Tue 28/Dec/2021

- Projects
  - Spreadbase
    - [x] Discuss Rubyzip 3.0 PR; prepare and propose code
- SWE
  - [ ] Career readings
- Low-level Programming
  - The Art of 64-Bit Assembly, Volume 1
    - [x] Copy notes Chapter 4
      - [x] Fight/research nested structures
    - [x] ~~Copy notes Chapter 5~~
    - [ ] Copy notes Chapter 6

## Mon 27/Dec/2021

- Admin
  - [x] Cycle planning
  - [x] Search PDF annotation apps (Android/iOS)
- Projects
  - geet
    - [x] MergePr: Push before merging
  - libvfio-user-rs
    - [x] Email maintainer
  - VSC grammars extension
    - [x] Split into two extensions
    - [x] Cleanus documentation(s)
  - Spreadbase
    - [x] Review, investigate and discuss Rubyzip 3.0 PR
- Tools
  - Personal notes syncing script
    - [x] Refactor and improve the commandline options
    - [x] Minor refactoring to commit logic
- Low-level Programming
  - The Art of 64-Bit Assembly, Volume 1
    - [ ] Copy notes Chapter 4

## Cycle

Messy week, due to holidays.

## Sun 26/Dec/2021

- Projects
  - Schedule manager
    - [x] Extend debug mode to Replanner class
    - [x] Replanner: Fix for multi-digit months/years intervals
  - Openscripts
    - [x] convert_cb_archive_to_pdf: Fix permissions and rebase
- Admin
  - [ ] New year planning
  - [ ] Cycle planning
- SWE
  - [ ] Career readings
- Low-level Programming
  - [ ] The Art of 64-Bit Assembly, Volume 1
    - [x] 6: Arithmetic

## Sat 25/Dec/2021

- Low-level Programming
  - [ ] The Art of 64-Bit Assembly, Volume 1
    - [ ] 6: Arithmetic

## Fri 24/Dec/2021

- Projects
  - Openscripts
    - [x] Add cbz/zip support to converter
      - [x] Some refactorings
      - [x] Research epic text padding strategy
    - [x] convert_cb_archive_to_pdf: Add support for deleting the source file
- Writings
  - [x] Article: Generically padding strings in shell scripts
- Tools
  - VSC
    - [x] Test issue with Ruby indentation

## Thu 23/Dec/2021

- Projects
  - TenderJIT
    - [x] Restrict temp vars instantiation to the block form

## Wed 22/Dec/2021

- Projects
  - TenderJIT
    - [x] Review PR
- Low-level Programming
  - [ ] The Art of 64-Bit Assembly, Volume 1
    - [ ] 6: Arithmetic
      - Fight misunderstanding of the overflow flag

## Tue 21/Dec/2021

- Tools
  - Mermaid-JS
    - [x] Review workarounds for connecting a function to a class in a class diagram
      - [x] Open [issue](https://github.com/mermaid-js/mermaid/issues/2581)
- Low-level Programming
  - [ ] The Art of 64-Bit Assembly, Volume 1
    - [x] 5: Procedures
- Tools
  - Firefox
    - [x] Handle containerized sites that redirect (https://www.thechiefmeat.com/guides/containers.html)
      - Disable redirect https://support.mozilla.org/en-US/questions/1327430 (`accessibility.blockautorefresh = false`)
  - IntelliJ IDEs
    - [x] First look at CLion and RubyMine

## Mon 20/Dec/2021

- Low-level Programming
  - [ ] The Art of 64-Bit Assembly, Volume 1
    - [ ] 5: Procedures
- Projects
  - Schedule manager
    - [x] Interpolations: Modify the planned line, not the current
  - libvfio-user-rs
    - [ ] Study source code
  - TenderJIT/VSC
    - [x] Brutal fight making the Ruby project play with VSC
- Bash
  - [x] Read https://johannes.truschnigg.info/writing/2021-12_colodebug
- Tools
  - Music play script
    - [x] Display name also for dot paths
    - [x] Add track randomization option
- Admin
  - [x] Cycle planning

## Cycle

Some work on TJ; started the libvfio work, which is going to take a large part of the time.

Unscheduled: Only a significant task, on tooling, which took a couple of hours.

## Sun 19/Dec/2021

- Projects
  - Schedule manager
    - [x] Add interpolations: tokens that apply functions to the line
  - libvfio-user-rs
    - [ ] Study [[PATCH v9] introduce vfio-user protocol specification](https://patchew.org/QEMU/20210614104608.212276-1-thanos.makatos@nutanix.com)
    - [x] First look at the project structure
- SWE
  - [Sebastian Lague Channel](https://www.youtube.com/c/SebastianLague/videos)
    - [x] Copy pending notes
    - [x] A* Pathfinding (E07 - smooth weights)
    - [ ] A* Pathfinding (E08 - path smoothing 1/2)
- Low-level Programming
  - [ ] The Art of 64-Bit Assembly, Volume 1
    - [ ] 5: Procedures

## Sat 18/Dec/2021

- SWE
  - [Sebastian Lague Channel](https://www.youtube.com/c/SebastianLague/videos)
    - [x] A* Pathfinding (E05 - units)
- Tools
  - Repositories syncing script
    - [x] Improve parallel execution
      - [x] Fight shell handling
  - Ripping tool
    - [x] Restructure and improve
- Low-level Programming
  - [ ] The Art of 64-Bit Assembly, Volume 1
    - [x] 4: Constants, Variables, and Data Types
- Projects
  - libvfio-user-rs
    - [ ] Study [[PATCH v9] introduce vfio-user protocol specification](https://patchew.org/QEMU/20210614104608.212276-1-thanos.makatos@nutanix.com)

## Fri 17/Dec/2021

- SWE
  - [Sebastian Lague Channel](https://www.youtube.com/c/SebastianLague/videos)
    - [x] [A* Pathfinding (E04: heap optimization)](https://www.youtube.com/watch?v=3Dw5d7PlcTM)
    - [x] A* Pathfinding (E06 - weights)
  - [x] The Programmer's Brain
    - [x] 7. Misconceptions: Bugs in thinking
    - [ ] 8. ~~How to get better at naming things~~
    - [ ] 9. ~~Avoiding bad code and cognitive load: Two frameworks~~
    - [x] 10. Getting better at solving complex problems
    - [ ] ~~Part 4. On collaborating on code~~
- Projects
  - TenderJIT
    - [x] Watch today's live stream
  - libvfio-user-rs
    - [ ] Study [[PATCH v9] introduce vfio-user protocol specification](https://patchew.org/QEMU/20210614104608.212276-1-thanos.makatos@nutanix.com)

## Thu 16/Dec/2021

- SWE
  - [Sebastian Lague Channel](https://www.youtube.com/c/SebastianLague/videos)
    - [ ] [A* Pathfinding (E04: heap optimization)](https://www.youtube.com/watch?v=3Dw5d7PlcTM)
- Projects
  - TenderJIT
    - [x] Review, investigate and discuss [crash issue](https://github.com/tenderlove/tenderjit/issues/110)

## Wed 15/Dec/2021

- Projects
  - TenderJIT
    - [ ] Investigate [test suite segfault](https://github.com/tenderlove/tenderjit/issues/110)
      - [x] Make `rake debug` run the whole test suite
      - [x] Discuss issue
    - [x] Organize notes
- SWE
  - [Sebastian Lague Channel](https://www.youtube.com/c/SebastianLague/videos)
    - [x] [A* Pathfinding (E03: algorithm implementation)](https://www.youtube.com/watch?v=mZfyt03LDH4)
    - [ ] [A* Pathfinding (E04: heap optimization)](https://www.youtube.com/watch?v=3Dw5d7PlcTM)
- Projects
  - libvfio-user-rs
    - [x] `README.md`
    - [ ] Study [vfio-user protocol](https://lists.gnu.org/archive/html/qemu-devel/2020-11/msg02458.html)

## Tue 14/Dec/2021

- Tools
  - Visual Studio Code
    - [x] Search method copy functionality
- Projects
  - TenderJIT
    - [x] Rebase/solve conflicts for last PR
    - [x] Investigate/report master failure
  - libvfio-user-rs
    - [x] Setup chat
    - [x] Write RedHat employee
      - [x] Start planning
  - First look at the project
    - [ ] `README.md`
  - [x] Configure VSC support
- SWE
  - [ ] [Sebastian Lague Webcasts](https://www.youtube.com/c/SebastianLague/videos)
    - [x] [A* Pathfinding (E01: algorithm explanation)](https://www.youtube.com/watch?v=-L-WgKMFuhE)
      - [x] Review and store notes
  - The Programmer's Brain
    - [x] 6. Getting better at solving programming problems

## Mon 13/Dec/2021

- Tools
  - [x] Research shortcuts for some MATE/VSC actions
- Admin
  - [x] Plan cycle
- Low-level Programming
- [ ] The Art of 64-Bit Assembly, Volume 1
  - [x] Copy errata
- Projects
  - TenderJIT
    - [x] Watch TenderJIT Webcasts
      - [x] Episode 05
    - [x] A few cleanups
    - [x] Runtime: Add autoalign support to Ruby func calling
    - [x] Investigate issues on [Runtime: Save temp var registers before calling cfuncs](https://github.com/tenderlove/tenderjit/pull/101)
      - [x] Fix issue with `toregexp`
      - [x] Brutal fight with the segfault
- SWE
  - [ ] [Sebastian Lague Webcasts](https://www.youtube.com/c/SebastianLague/videos)
    - [ ] [A* Pathfinding (E01: algorithm explanation)](https://www.youtube.com/watch?v=-L-WgKMFuhE)

## Sun 12/Dec/2021

- SWE
  - [x] Review recurse.com
  - [x] Review new books to buy
- QEMU/Rust
  - [x] Write RedHat employee
    - [x] Brutal fight with Matrix (verification)

## Sat 11/Dec/2021

- Projects
  - TenderJIT
    - [ ] Watch TenderJIT Webcasts
      - [ ] Episode 05

## Fri 10/Dec/2021

- Low-level Programming
- [ ] The Art of 64-Bit Assembly, Volume 1
  - [ ] 4: Constants, Variables, and Data Types

## Thu 09/Dec/2021

- Projects
  - TenderJIT
    - [ ] Watch TenderJIT Webcasts
      - [x] Episode 04

## Wed 08/Dec/2021

- Projects
  - TenderJIT
    - [ ] Watch TenderJIT Webcasts
      - [x] Episode 03
      - [ ] Episode 04
    - [x] Make TenderJIT allocated memory configurable
    - [ ] [Runtime: Save scratch registers before calling cfuncs](https://github.com/tenderlove/tenderjit/pull/101)
      - [x] Discuss tempvars issue

## Tue 07/Dec/2021

- Tools
  - Visual Studio Code
    - [x] Find out easiest way/extensions for comparison
- Low-level Programming
- [ ] The Art of 64-Bit Assembly, Volume 1
  - [x] 2: Computer Data Representation and Operations
    - [x] Notes
  - [x] 3: Memory Access and Organization
- Projects
  - TenderJIT
    - [x] Search (other) interpreter internals books
    - [ ] [Runtime: Save scratch registers before calling cfuncs](https://github.com/tenderlove/tenderjit/pull/101)
      - [x] Fight with architecture, to find the fitting design
      - [ ] Fight with register allocation issue
  - Schedule manager
    - [x] Compute work hour times

## Mon 06/Dec/2021

- Projects
  - TenderJIT
    - [ ] Watch TenderJIT Webcasts
      - [ ] Episode 03
- Projects
  - QEMU/libvfio-user
    - [x] Write about `libvfio-user` project
- Low-level Programming
  - [ ] The Art of 64-Bit Assembly, Volume 1
    - [ ] 2: Computer Data Representation and Operations
      - [x] Study
        - [x] Fight verify erratum
        - [x] Fight Search official errata
          - https://randallhyde.com/Forum/viewtopic.php?f=27&t=66
      - [x] Exercises

## Cycle and notes

Resurrected from the break :) Nice open source production.

Lessons: must be 100% sure of a problem, and have a very reproducible test case, before attacking it; wasted time due to working on the wrong-ish and nondeterministic problem with Solargraph.

## Sun 05/Dec/2021

- Projects
  - Openscripts
    - [x] Add script to convert .CBR to .PDF
- SWE
  - [x] Read https://docseuss.medium.com/look-what-you-made-me-do-a-lot-of-people-have-asked-me-to-make-nft-games-and-i-wont-because-i-m-29c7cfdbbb79
  - [x] Read https://docseuss.medium.com/look-what-i-made-for-you-apparently-while-i-am-not-a-dumbass-many-cryptobros-are-and-i-am-going-f6468c877acc
- Projects/SWE
  - Solargraph
    - [x] Study source code
    - [x] Experiment to make go to definition faster

## Sat 04/Dec/2021

- Open Source
  - ASM Code Lens
    - [x] Research and discuss Markdown preview rendering
- Low-level Programming
  - [ ] The Art of 64-Bit Assembly, Volume 1
    - [x] 1: Hello, World of Assembly Language
    - [ ] 2: Computer Data Representation and Operations
- SWE
  - The Programmer's Brain
    - [x] Copy remaining exercises from past chapters
    - [x] 5. Reaching a deeper understanding of code
    - [ ] 6. Getting better at solving programming problems
  - [x] Read and analyze a randomization algorithm in Fish Fight

## Fri 03/Dec/2021

- Open Source
  - ASM Code Lens
    - [x] Add support for ASM in Markdown code fences (https://github.com/maziac/asm-code-lens/issues/63)
- Projects
  - UASM-linux
    - [x] Update to v2.51, with Linux fixes
      - [x] Linux Compatibility: Redefine _getcwd as getcwd
      - [x] Clang compatibility: Cast `uint_8*` to `char*` for pointer arithmetic
      - [x] Linux compatibility: Find the executable filename (equivalent of Windows' `_pgmptr`)
      - [x] Linux compatiblity: Redefine _MAX_PATH as PATH_MAX
      - [x] Makefile (Linux): Compile in C99 std, by removing `-ansi`
    - [x] Update to v2.53 (no fixes required)
    - [x] Add announcement post on the blog
- Projects
  - TenderJIT
    - [x] Watch today's live stream
- Low-level Programming
- [ ] The Art of 64-Bit Assembly, Volume 1
  - [ ] 1: Hello, World of Assembly Language

## Thu 02/Dec/2021

- Tools
  - [x] Search clipboard manager
- Open Source
  - Clipit
    - [x] Investigate source code for "offline mode" feature
      - [x] Explain on existing GitHub issue
  - UASM
    - [x] Investigate MASM alternatives on Linux
    - [x] Create UASM fork, and fix it to work on modern Linux systems
- Low-level Programming
  - [ ] The Art of 64-Bit Assembly, Volume 1
    - [ ] 1: Hello, World of Assembly Language

## Wed 01/Dec/2021

- Open Source
  - Schedule Manager
    - [x] Check for `replan` token before moving the current date section
- Admin
  - System
    - [x] Find clean strategy for loading `$HOME/.profile` on non-login shells
  - Planning
    - [x] Refine cycle schedule (especially practice)

## Tue 30/Nov/2021

- Projects
  - TenderJIT
    - [x] Review Ruby jokes code compilation
- SWE
  - The Programmer's Brain
    - [ ] 5. Reaching a deeper understanding of code

## Cycle changes

A weak week; took a break-ish in the second half. Sunk a whole evening into Ruby Solargraph hell (although it lead to better VSC Ruby editing).

- Unscheduled
  - Blog: Review and reply [Custom kernel build comment](https://tinyurl.com/yzn8w7g2)
  - Tools
    - Brogramming transcriber: Add splitting of an entry, when there is a delimiter (`: `)
    - Connections handler: Refactoring and minor improvement
  - Open Source: Schedule manager: Add work task comment option
  - Writings: Article: Beginning x64 Assembly Programming Errata
  - Ruby: Investigate Solargraph YARD methods documentation
- Postponed
  - (dropped) TenderJit: Implement unit test for unaligned failing call_cfunc

## Sun 28/Nov/2021

- SWE
  - The Programmer's Brain
  - [ ] 5. Reaching a deeper understanding of code

## Sat 27/Nov/2021

- SWE
  - The Programmer's Brain
  - [x] 4. How to read complex code
  - [ ] 5. Reaching a deeper understanding of code

## Fri 26/Nov/2021

- SWE
  - The Programmer's Brain
  - [ ] 4. How to read complex code

## Thu 25/Nov/2021

- Ruby
  - [x] Open `vscode-solargraph` issue about performance inconsistency
- SWE
  - The Programmer's Brain
    - [x] 3. How to learn programming syntax quickly

## Wed 24/Nov/2021

- Projects
  - TenderJIT
    - [ ] Watch TenderJIT Webcasts
      - [x] Episode 02
      - [ ] Episode 03
- Ruby
  - Investigate Solargraph YARD methods documentation
    - [x] Try on TenderJIT
    - [x] Brutal fight with vscode-solargraph performance investigation

## Tue 23/Nov/2021

- Open source
  - Schedule manager
    - [x] Add work task comment option
  - Mydumper
    - [x] Discuss the issue workaround options
- Projects
  - TenderJIT
    - [ ] Watch TenderJIT Webcasts
      - [ ] Episode 02
    - [x] Prepare for testing optcarrot with TJ
- SWE
  - The Programmer's Brain
    - [ ] 3. How to learn programming syntax quickly
      - [x] Try flash card programs
      - [x] Start storing some cards
      - [x] Report Anki bug
        - [x] Follow-up with developer: test on alternate client
        - [x] Report on forum

## Mon 22/Nov/2021

- Open source
  - ZFS installer
    - [x] README: Add project status section, and extend stability section
- Projects
  - TenderJIT
    - [ ] Watch TenderJIT Webcasts
      - [x] Episode 01
      - [ ] Episode 02
    - [x] Discuss [Ruby version support](https://github.com/tenderlove/tenderjit/discussions/98)
- Planning
  - [x] Arrange long term view prospects
- SWE
  - The Programmer's Brain
    - [ ] 3. How to learn programming syntax quickly

## Sun 21/Nov/2021

- Projects
  - TenderJIT
    - [ ] Watch TenderJIT Webcasts
      - [ ] Episode 01
    - [x] Extend Runtime test suite functionality (and add supporting changes)
      - [x] Brutal fight with 64-bit addressing niggles
      - [x] Investigate unexpected build failure not in dev
- Tools
  - Connections handler
    - [x] Merge two scripts into one
    - [x] Handle other service, and pause before disconnecting
- Planning
  - [x] Large review, update and reorganization of the brogramming pools
- Open source
  - MyDumper
    - [x] Discuss FK checks bug
      - [x] Test alternate procedures, and provide super simple one

## Sat 20/Nov/2021

- Writings
  - [x] Add another erratum to the "Beginning x64 Assembly Programming Errata" article
- Projects
  - TenderJIT
  - [ ] Watch TenderJIT Webcasts
    - [ ] Episode 01
- Planning
  - [ ] Large review, update and reorganization of the brogramming pools

## Fri 19/Nov/2021

- Tools
  - [x] Brogramming transcriber
    - [x] Add splitting of an entry, when there is a delimiter (`: `)
- SWE
  - The Programmer's Brain
    - [x] 2. Speed reading for code
- Gamedev
  - [x] Review Carmack's post [Wine <> Native ports](https://www.reddit.com/r/linux/comments/17x0sh/comment/c89sfto)
- Projects
  - ZFS installer
    - Issue: [Redundant 'EFI' does not work](https://github.com/64kramsystem/zfs-installer/discussions/249)
      - [x] Try to reproduce
      - [x] Discuss
  - TenderJIT
    - [x] Fight download streams
- Writings
  - [x] Article: Beginning x64 Assembly Programming Errata

## Thu 18/Nov/2021

- Low-level programming
  - Beginning x64 Assembly Programming
    - [x] Copy notes

## Wed 17/Nov/2021

- Projects
  - ZFS installer
    - [x] Review [support request](https://github.com/64kramsystem/zfs-installer/discussions/251)
      - [x] Extend related message (and release new version)
    - [x] Follow up on EFI issue
- Blog
  - Custom Linux Kernel compilation
    - [x] Follow up discussion on modern tooling
- Low-level programming
  - Beginning x64 Assembly Programming
    - [ ] Copy notes
      - [x] Investigate Yet Another Error

## Tue 16/Nov/2021

- Projects
  - ZFS installer
    - [x] Review Issue [No suitable disks have been found | KVM | Guest](https://github.com/64kramsystem/zfs-installer/issues/247)
- Blog
  - [x] Review and apply [Bash comment](https://tinyurl.com/yegwnyxk)
  - [x] Review and reply [Custom kernel build comment](https://tinyurl.com/yzn8w7g2)
- Tools
  - Visual Studio Code (Extensions)
    - [x] Add ASM/TF code block support 
      - Fight hacking Code Fences extension
    - [x] Setup and publish extension
- Low-level programming
  - Beginning x64 Assembly Programming
    - [ ] Copy notes
- SWE
  - The Programmer's Brain
    - [ ] 2. Speed reading for code

## Mon 15/Nov/2021

- Low-level programming
  - Beginning x64 Assembly Programming
    - [ ] Copy notes
      - [x] Investigate another book errata
- Open Source
  - Mydumper
    - [x] Discuss negative regex issue

## Cycle notes

- Unscheduled
  - Schedule Manager: A full session spent on refactoring/extending/fixing

## Sun 14/Nov/2021

- Projects
  - Schedule manager
    - [x] Allow any prefix
- Admin
  - [x] Plan cycle
- Open Source
  - Mydumper
    - [x] Myloader: Investigate exact conditions, and prepare/anonymize simplest dataset for issue reproduction
  - Schedule manager
    - [x] Fix: numerical next prefix was not included when rewriting a full update
- Tools
  - [x] Add script to manage remote desktop operations
- System
  - [x] Check modern 13"+ detachable laptops
    - [ ] Check Surface series [Linux project compatibity](https://github.com/linux-surface/linux-surface)

## Sat 13/Nov/2021

- Low-level programming
  - Beginning x64 Assembly Programming
    - [ ] Copy notes
- Projects
  - Fisk
    - [x] Prepare BSD for testing
    - [x] Try and report issue
    - [x] Brutal fight with BSD variants

## Fri 12/Nov/2021

- Projects
  - TenderJIT
    - [ ] Try install BSD distro(s)
      - [x] Find other distro
- Open Source
  - Mydumper
    - [x] Brutal fight with Myloader bug investigation
- Sysadmin
  - [x] Check Terraform Up and Running 2nd ed.

## Thu 11/Nov/2021

- Projects
  - Fisk
    - [x] Investigate for Jeremy a useful UT, and discuss it

## Wed 10/Nov/2021

- Linux
  - [x] Review ZFS/DB write safety
    - https://lists.launchpad.net/maria-discuss/msg05205.html
- Projects
  - Openscripts
    - `.git_maintain_branches`
      - [x] Exit with error if the configfile is not present (note that it's important)
      - [x] Don't push protected branches
  - Schedule manager
    - [x] Implement automatic work end times
- SWE
  - The Programmer's Brain
    - [x] 1. Decoding your confusion while coding
    - [ ] 2. Speed reading for code
- Open source
  - Resque
    - [x] Discuss future development (and scheduler status)
- Low-level programming
  - Beginning x64 Assembly Programming
    - [ ] Copy notes

## Tue 09/Nov/2021

- Ruby
  - [x] Review type checking
- Projects: Schedule manager
  - [x] Add support for skip on updates
    - [x] Made massive refactorings, part of which were required
  - [x] Refactored and extended Replanner test suite
  - [x] Fix (old) bug with updates showing the prefix on indented lines

## Mon 08/Nov/2021

- Open source
  - Mydumper
    - [x] Follow up investigation on [myloader drop table issue](https://github.com/maxbube/mydumper/issues/469)
  - Openscripts: `git_maintain_branches`
    - [x] Refactor
    - [x] Sync with upstream, if present
- Low-level programming
  - [x] Review and buy The Art of 64-bit Assembly Vol.1
- Tools
  - VSC
    - [x] Investigate and report [Ruby indentation bug](https://github.com/microsoft/vscode/issues/136708)

## Sun 07/Nov/2021

- Low-level programming
  - [x] Beginning x64 Assembly Programming
    - [x] Chapter 38: Performance Optimization
    - [x] Chapter 39: Hello, Windows World
    - [x] Chapter 40: Using the Windows API
    - [x] Chapter 41: Functions in Windows
    - Skipped last two Windows chapters
    - [x] Collect, verify and publish question about seeming errata
      - [x] Brief look at NASM code, to find out why `SAL` is converted to `SHL`
- Open source
  - ps_mem
    - [x] Investigate and open PR: Convert shebang to python3
- Projects: TenderJIT/Fisk
  - [x] Brutal fight with finding SEGV case
    - [x] Brutal fight with stack alignment interpretation
  - [ ] Brutal fight with TJ's alignment
    - The runtime start unaligned (!!)
  - [x] Find issue with alignment UT failing
    - `RegistersSavingBuffer` misaligns the stack

## Sat 06/Nov/2021

- Tools
  - Editors
    - [x] VSC: Find out how to go to beginning of current block
- Low-level programming
  - [ ] Beginning x64 Assembly Programming
    - [x] Chapter 28: SSE Alignment
    - [x] Chapter 29: SSE Packed Integers
    - [x] Chapter 35: AVX
    - [x] Chapter 36: AVX Matrix Operations (part)
    - Skipped some SSE/AVX chapters

## Fri 05/Nov/2021

- Projects: Fisk
  - [x] Open discussion: Add support for `mov [reg1 + K * reg2]` addressing
  - [x] Open discussion: Add support for writing binary values in `Fisk#asm`
- Projects: TenderJIT
  - [ ] Implement unit test for unaligned failing call_cfunc
    - [x] Cause SIGSEGV, and try to trap it
    - [x] Find way to write a UT (not implemented)
- Low-level programming
  - [x] x64 ASM: [Stack alignment](https://stackoverflow.com/a/52300101)
  - [x] Cause/recover SIGSEGV
  - Beginning x64 Assembly Programming
    - [x] Chapter 26: SIMD
    - [x] Chapter 27: Watch Your MXCSR

## Thu 04/Nov/2021

- Low-level programming
  - [ ] Beginning x64 Assembly Programming
    - [x] Chapter 24: Strings
    - [x] Chapter 25: Got Some ID?

## Wed 03/Nov/2021

- Projects: TenderJIT/Fisk
  - [x] Implement registers saving buffer (integration)
    - [x] Handle RSP value adjustment
    - [x] Minor cleanups
    - [x] Implement integration
      - [x] Various fights
      - [x] Update `Runtime.inc` test

## Tue 02/Nov/2021

- Open source
  - Mydumper
    - [x] Try negative regex option, from `regex` branch
    - [x] Open issue "Warning `Couldn't acquire global lock` when dumping"
    - [x] Open issue "With --source-db, the source database is still created"
    - [x] Open issue "Error Thread 3 issue restoring [...]: Table 'tablename' already exists while restoring tables"
- Projects
  - ZFS installer
    - [x] Reply to issues
    - [x] Attempt add KDE Neon support
  - TenderJIT/Fisk
    - LLDB
      - [x] Fight disassemble previous instructions/relative address
    - [x] Implement registers saving buffer (to be released)
      - [x] Fight a lot of stuff
- SWE
  - [x] Review and buy The Programmer's Brain
- Misc
  - [x] Plan week

## Mon 01/Nov/2021

- Projects: TenderJIT/Fisk
  - [x] Implement intern instruction (shallow): Apply feedback
  - [x] Refinements to make_warnings_errors, and two cleanups
    - [x] Require `make_warnings_errors` earlier
    - [x] make_warnings_errors: Ensure that $VERBOSE is set
    - [x] make_warnings_errors: Undefine :warn
    - [x] ISEQCompiler: Remove two unused variables
  - [x] Implement `cfunc_call` automatic alignment
    - [x] Fisk: Implement size() for Register and Temp

## Sun 31/Oct/2021

- Projects: TenderJIT/Fisk
  - [x] Implement `intern` instruction (shallow)
  - [ ] Implement `newrraykwsplat` instruction
    - [x] Dig down the rabbit hole

## Sat 30/Oct/2021

- Low-level programming: Beginning x64 Assembly Programming
  - [x] Chapter 23: Inline Assembly
  - [ ] Chapter 24: Strings
    - [x] Fight bug with code/text
- Projects: TenderJIT/Fisk
  - [ ] Study Fisk
    - Temp registers
    - Immediate operands

## Fri 29/Oct/2021

- SWE
  - [x] Search code reading clubs
- Projects: TenderJIT
  - [x] Runtime: Split division() into div() and mod() #85
  - [x] `topn`: Add UT

## Thu 28/Oct/2021

- Projects
  - TenderJIT
    - [x] Review Nikita's `swap` implementation
      - [x] Test bug
      - [x] Research x64 swap strategies speed
  - Schedule manager
    - [x] Detection of a new type of invalid work line format

## Wed 27/Oct/2021

- Projects: TenderJIT
  - [x] Add Rake (lldb) debug task
    - [x] Brutal fight with Bundler
  - [x] Rakefile: Don't generate the reverse test suite
  - [x] Implement opt_mod instruction
  - [x] Testing: Make warnings into errors
  - [x] Comple list and open issue "Easy/ish missing instructions"
  - [x] Correctly implement `swap`
    - [x] Brutal fight with finding an alternate code generation
    - [x] Write summary
    - [x] Brutal fight with the implementation

## Tue 26/Oct/2021

- Projects
  - Geet
    - [x] Backup summary before operations
  - TenderJIT
    - [x] Implement `opt_succ` instruction
      - [x] Fight pretty much everything
      - [x] Implement `Runtime#inc`
    - [x] Implemtn `opt_div` instruction
      - [x] Implement `Runtime#div`
      - [ ] Try to find how to execute the JIT buffer
  - Schedule manager
    - [x] Update full with interval: keep the `U` flag
- Low-level programming
  - [x] Investigate presumed errata on Beginning x64 Assembly Programming

## Cycle

Full attention on TenderJIT, but slow week.

## Mon 25/Oct/2021

- Scripts
  - [x] Fight rewriting of file with unsupported unicode chars
- System
  - [x] Merge Windows virtual machines
- Projects
  - QEMU-pinning
    - [x] Follow up discussion with user
    - [x] Update README with new status
    - [x] Update master: disable vcpu check
      - [x] Verify (also against v6.1.0-rc2 patch)
    - [x] Reorganize branches for v6.1
  - TenderJIT/Fisk
    - [x] Reorganize the (big) pool
  - Fisk
    - [x] Clean up build warning
    - [x] Implement the optimizer
    - [x] Implement no-op `MOV` optimization

## Sun 24/Oct/2021

- Projects
  - TenderJIT
    - [x] Quick review recent PRs
    - [x] Implement `newrange`
      - [x] Fight attempt to implement a deeper version
    - [x] Implement `splatarray`
    - [ ] Cleanup warnings
      - Fight disable warnings for some UTs
    - [x] Discuss potential IR implementation
      - [x] First look at Rhizome design

## Sat 23/Oct/2021

- Low-level programming
  - [ ] Beginning x64 Assembly Programming
    - [x] Chapter 20: File I/O
    - [x] Chapter 21: Command Line
    - [x] Chapter 22: From C to Assembler
    - [ ] Chapter 23: Inline Assembly

## Fri 22/Oct/2021

- Projects
  - TenderJIT
    - [x] Implenent `toregexp`
      - [x] Fight storing a value across cfunc calls
      - [x] First review at disassembling via LLDB
    - [x] Help Nikita with PR
      - [x] Disassemble and find issue with the test
      - [x] Fight find simpler `swap` implementation

## Thu 21/Oct/2021

- Low-level programming
  - [ ] Beginning x64 Assembly Programming
    - [x] Chapter 16: Bit Operations
    - [x] Chapter 17: Bit Manipulations
    - [x] Chapter 18: Macros
    - [x] Chapter 19: Console I/O
- Projects
  - TenderJIT
    - [x] Performance check design: review and implement two variations

## Wed 20/Oct/2021

- Projects
  - TenderJIT
    - [x] Implement performance check
- Low-level programming
  - [ ] Beginning x64 Assembly Programming
    - [x] Chapter 13: Stack Alignment and Stack Frame
    - [x] Chapter 14: External Functions
    - [x] Chapter 15: Calling Conventions

## Tue 19/Oct/2021

- Projects
  - TenderJIT
    - [x] Review Fisk, and design performance linter
    - [x] [Search why loop is slower than dec cx/jnz](https://stackoverflow.com/q/35742570)

## Mon 18/Oct/2021

- Low-level programming
  - [ ] Beginning x64 Assembly Programming
    - [x] Chapter 10: The Stack
    - [x] Chapter 11: Floating-Point Arithmetic
    - [x] Chapter 12: Functions
- Projects
  - TenderJIT
    - [x] Further investigation on `tostring instruction fails on complex examples`
    - [x] Improve generated ASM debug output

## Cycle

Full immersion in TenderJIT/ASM.

Negligible unscheduled stuff.

## Sun 17/Oct/2021

- Projects
  - TenderJIT
    - [x] Fight try to debug `tostring instruction fails on complex examples`
- Low-level programming
  - [ ] Beginning x64 Assembly Programming
    - [x] Chapter 9: Integer Arithmetic
    - [ ] Copy notes
  - [x] Find/install updated SASM

## Sat 16/Oct/2021

- Tools
  - [x] Fix notes sync script
- Low-level programming
  - [ ] Beginning x64 Assembly Programming
    - [x] Chapter 7: Jumping and Looping
    - [x] Chapter 8: Memory
- Projects
  - TenderJIT
    - [x] Quick review discussions

## Fri 15/Oct/2021

- Open source
  - MySQLTuner
    - [x] Diagnose log buffer size bug
    - [x] Open issue and PR
    - [x] Open second (related) issue
- Projects
  - TenderJIT
    - [x] Discuss no-op `mov`s handling
      - [x] Test NASM behavior
      - [x] Quick look at (other) ASM optimizations
      - [x] Write summary
  - QEMU-pinning
    - [x] Discuss v6.1 vCPU issue
- Low-level programming
  - [ ] Beginning x64 Assembly Programming
    - [x] Chapter 5: Assembly Is Based on Logic
    - [x] Chapter 6: Data Display Debugger
    - [ ] Chapter 7: Jumping and Looping

## Thu 14/Oct/2021

- Low-level programming
  - [ ] Beginning x64 Assembly Programming
    - [x] Chapter 3: Program Analysis with a Debugger: GDB
    - [x] Chapter 4: Your Next Program: Alive and Kicking!
- Projects
  - TenderJIT
    - [x] Remove unnecessary `Fisk::Imm64` wraps
    - [x] Review how to disassemble generated code
      - [x] Fight `StringIO` vs. `JITBuffer` heisenstuff (!!)
    - [x] Test `rt.return` crash again

## Wed 13/Oct/2021

- Low-level programming
  - [x] C Calling convention (Stack alignment): https://staffwww.fullcoll.edu/aclifton/cs241/lecture-stack-c-functions.html: Stack alignment
- Projects
  - TenderJIT
    - [x] Instruction handlers modernization (first batch)
    - [x] Instruction handlers modernization (second batch)
    - [x] Implement `tostring` instruction

## Tue 12/Oct/2021

- Open Source
  - [x] Search Hacktoberfest Rust projects
    - [x] Fight how to unsubscribe from Hacktoberfest
- Projects
  - TenderJIT
    - [ ] Study new `concatarray`, and investigate improvements
    - [x] Fight investigate no-op `mov`s
    - [x] Review potential new instructions to implement
- Low-level programming
  - [x] Study x64 [C Calling convention PDF](https://aaronbloomfield.github.io/pdr/book/x86-64bit-ccc-chapter.pdf)
    - [x] Other reading: https://staffwww.fullcoll.edu/aclifton/cs241/lecture-stack-c-functions.html (conventions only)

## Mon 11/Oct/2021

- Low-level programming
  - [x] Fight prepare RISC-V book
  - [ ] Beginning x64 Assembly Programming
    - [x] Chapter 1: Your First Program
    - [x] Chapter 2: Binary Numbers, Hexadecimal Numbers, and Registers
    - [ ] Chapter 3: Program Analysis with a Debugger: GDB
      - [x] Search debugger GUI (gdbgui)
- SWE
  - [x] Fight find women in tech organizations
    - [x] Girls Who Code
    - [x] Contact Code Curious leader
- Projects
  - ZFS installer
    - [x] Make the `libzfs?linux` package installation generic
- Tools
  - [x] Journal transfer script: New section skipping logic
  - [x] Daily script: Open file only if specified (by new option)

## Cycle changes

Going well. Only minor unscheduled stuff. Lots of TenderJIT stuff, and completed the damn Rust book.

- Unscheduled
  - Blog: `add_new_book.sh`: Don't prompt for reviewers on PR creation
  - Macroquad: Investigate open issue High CPU usage (#257), and propose solution
  - Tools: Visual Studio Code: Fight find out how to exclude files per-project

## Sun 10/Oct/2021

- Rust
  - [x] Copy Programming Rust (massive) remaining notes
- Open Source
  - Macroquad
    - [x] Investigate open issue High CPU usage (#257), and propose solution
- Low-level programming
- [ ] Search and review next ASM book(s)
  - [x] Search x64/RISC-V books

## Sat 09/Oct/2021

- Projects
  - TenderJIT
    - [x] First review tenderlove's `concatarray` update
  - ZFS installer
    - [x] Investigate [21.10 grub-install issue](https://github.com/64kramsystem/zfs-installer/issues/241)
    - [x] Check if it's possible to make the `libzfs?linux` package installation generic

## Fri 08/Oct/2021

- Projects
  - TenderJIT
    - [x] Fight implement newhash
      - [x] Discuss helping other contributor
    - [x] Review default Rake task
      - [x] Investigate history and open issue
    - [x] Review running direct test
    - [x] Tentative `checkkeyword` implementation
      - [x] Open PR with test suite only
    - [x] Rake test task: Add support for test suite prefix and unit test name
    - [x] Brutal fight to implement concatarray
  - ZFS installer
    - [x] Review and discuss [remote unlock feature request](https://github.com/64kramsystem/zfs-installer/discussions/240)

## Thu 07/Oct/2021

- Projects
  - TenderJIT
    - [x] Ruby fingerprinting
      - [x] Implement and open PR
    - [x] Discuss `newhash`
      - [x] Investigate which are the conditions under which `duphash` is used instead
      - [x] Investigate Ruby internal hash creation
- Tools
  - Visual Studio Code
    - [x] Fight find out how to exclude files per-project

## Wed 06/Oct/2021

- Admin
- Projects
  - TenderJIT
    - [ ] Ruby fingerprinting
      - [x] Review RbConfig source
      - [x] Discuss new solution
    - [x] Investigate/fix `Fisk error when running the test suite, on a Clang-compiled Ruby`
      - [x] Add smoke test suite for `Runtime`
- Rust
- Tools
  - Invoicing script
    - [x] Refactor
    - [x] Add dry run functionality
    - [x] Automatically compute net days

## Tue 05/Oct/2021

- Projects
  - Blog
    - [x] `add_new_book.sh`: Don't prompt for reviewers on PR creation
  - QEMU-pinning
    - [x] Fix 6.1 breakage issue: https://github.com/64kramsystem/qemu-pinning/issues/35
  - TenderJIT
    - [x] Fight attempt to implement new instruction
    - [x] Fight understand project
    - [x] Review Fisk
      - [x] Fight debug symbols
        - [x] Open PR with indications in the README
    - [ ] Review symbols [issue](https://github.com/tenderlove/tenderjit/issues/24) and [PR](https://github.com/tenderlove/tenderjit/pull/29)
    - [ ] Investigate Clang code todos
      - [ ] Investigate unexpected failures with Fisk
        - [x] Open issue
        - [x] Discuss and review issue
    - [ ] Review Ruby fingerprinting
      - [x] Open issue

## Mon 04/Oct/2021

- Open Source
  - SubSync
    - [x] Report encoding detection issue
- Admin
  - SWE
    - [x] Buy new books for the month
      - Fight "Computer Systems: A Programmer's Perspective" editions
  - QEMU/Rust
    - [x] Join Slack channel
- Projects
  - TenderJIT
    - [x] Review/discuss issues opened
      - [x] Investigate potential RubyGems issue
- Rust
  - Programming Rust (2nd ed.)
    - [x] Chapter 17: Strings and Text
    - [x] Chapter 19: Concurrency
    - [x] Chapter 23: Foreign Functions
    - Skipped chapters 20, 21, 22

## Cycle changes

Overall, TenderJIT took most of the time.

- Swapped
  - Recompilation Project <> TenderJIT+QEMU
- Unscheduled
  - (Various projects)
    - Discussions
  - Fish Fight
    - Mentor Zac/Review PRs
  - ZFS installer
    - Review/discuss feature request dropbear boot
  - Ruby/Email
    - Fight manual composition and sending of emails (with attachments)
  - SWE
    - Review/Join Hacktoberfest
  - QEMU/Rust
    - Fight Patchew
  - Rust
    - Programming Rust (2nd ed.): Several chapters
  - Openscripts
    - `git_purge_empty_branches`: Add support for generic development branch
  - Schedule manager
    - Add update whole line (`-U`) (a few refactorings, including two significant ones)

## Sun 03/Oct/2021

- Projects
  - Schedule manager
    - [x] Workaround Timecop 0.8.x bug
    - [x] Refactor Replanner
    - [ ] Mega-refactoring to ReplanCodec
    - [x] Add update whole line (`-U`)
    - [x] Replanner: On full update, consider recurring the events with weekday but no interval
      - [x] Refactoring: Simplify line update logic when there is no interval
- Rust
  - Programming Rust (2nd ed.)
    - [ ] Chapter 17: Strings and Text

## Sat 02/Oct/2021

- Rust
  - Programming Rust (2nd ed.)
  - [x] Chapter 18: Input and Output
- Projects
  - Openscripts
    - [x] `git_purge_empty_branches`: Add support for generic development branch
- MySQL
  - [x] Fulltext performance investigation

## Fri 01/Oct/2021

- Projects
  - TenderJIT
    - [x] Review new PR
    - [x] Open issue about cleanly generating bytecode
    - [x] Brutal fight with attempt to implement `concatarray`
    - [x] Attempt to add `splatarray` UTs
      - [x] Implement `newarray` case for empty array (fixes bug)
      - [x] Add UT for empty array splatting
- Ruby/Email
  - [x] Fight manual composition and sending of emails (with attachments)
- MySQL
  - [ ] Fulltext performance investigation
  - [x] Discuss documentation issue/bug

## Thu 30/Sep/2021

- Open source
  - Fish Fight
    - [x] Review reason for crash in Zac's PR modification
  - ZFS installer
    - [x] Review/discuss opened issue about Dropbear boot
  - QEMU/Rust
    - [ ] Review all projects, and plan
  - TenderJIT
    - [x] Prepare for webcast
    - [x] Hexdevs Webcast
- MySQL
  - [ ] Fulltext performance investigation

## Wed 29/Sep/2021

- Open source
  - Fish Fight
    - [x] Review Zac's new PR
    - [x] Second review Zac's PR
  - RG3D
    - [x] Discuss RG3D proposals
  - QEMU/Rust
    - [x] Review libvfio-user Rust port, and join Outreachy
    - [x] Fight Patchew
  - [x] Review/Join Hacktoberfest
- Rust
  - Programming Rust (2nd ed.)
    - [x] Chapter 16. Collections

## Tue 28/Sep/2021

- Open source
  - Fish Fight
    - [x] Review Zac's PR
    - [x] Configuration file discussions
  - Awesome compilers
    - [x] Create PR with new tutorial
  - Schedule manager
    - [x] Retemplater: Automatically fill missing brackets
  - QEMU/Rust
    - [x] Collect current QEMU/Rust project references
- Interpreters/Low-level programming
  - [ ] Review resources for learning JIT programming, and plan
- SWE
  - [x] Check SWE meetups
- Admin
  - [x] Flush staging/Plan cycle
- Tools
  - [x] Notes syncer: add force previous functionality
  - [x] Daily script: add comment options functionality

## Cycle changes

- Unscheduled
  - Projects: Schedule manager: Minor parser refactoring
  - CS/Interpreters: Ruby Under a Microscope
  - ZFS installer: Handle reported issue
  - Fish Fight: Help beginner
- Swapped
  - Recompilation Project <> TenderJIT+QEMU

## Mon 27/Sep/2021

- Projects
  - Fish Fight
    - [x] Help beginner about Galleon weapon
- CS/Interpreters
  - [x] Ruby Under a Microscope
    - [x] Chapter 7: The Hash Table: The Workhorse of Ruby Internals
    - Skipped chapters 8/9
    - [x] Chapter 10: JRuby: Ruby on the JVM
    - [x] Chapter 11: Rubinius: Ruby Implemented with Ruby
    - [x] Chapter 12: Garbage Collection in MRI, JRuby, and Rubinius
      - Second part skipped
- Open source
  - TenderJIT
    - [x] Implement test suites
      - [x] `leave`
      - [x] `jump`
      - [x] `leave`
      - [x] `opt_aset`
    - [x] Search learning resources
    - [x] Fix testing warnings
- Rust
  - Programming Rust (2nd ed.)
  - [ ] Chapter 17: Strings and Text

## Sun 26/Sep/2021

- Open source
  - TenderJIT
    - [x] Implement branchif test suite
    - [x] Merge branchunless test suite files
    - [x] Issue: Handle Gemfile.lock
    - [x] Find out how the JIT gets the iseq, and think if testing could be based on it
      - [x] Find out if Ruby has APIs for creating an iseq
    - [ ] Try to understand branchif/branchunless
    - [ ] Implement test suites
      - [x] `branchnil`
      - [x] `intern`
      - [x] `nop`
- CS/Interpreters
  - [ ] Ruby Under a Microscope
    - [x] Chapter 6: Method Lookup and Constant Lookup

## Sat 25/Sep/2021

- CS/Interpreters
  - [ ] Ruby Under a Microscope
    - [x] Chapter 2: Compilation
    - [x] Chapter 3: How Ruby Executes Your Code
    - [x] Chapter 4: Control Structures and Method Dispatch
    - [x] Chapter 5: Objects and Classes
- Projects
  - ZFS installer
    - [ ] Ubuntu Server 21.04 user issue
      - [x] Test on VM
      - [x] Discuss
- Open source
  - TenderJIT
    - [ ] Review project
      - [x] Check branchif/branchunless: Ruby source/JIT

## Fri 24/Sep/2021

- Linux
  - [x] Different attempt to correctly decode textfile encoding
- Open source
  - MySQL
    - [x] Diagnose and report `ROW_COUNT()` bug
  - TenderJIT
    - [ ] Review and understand project
      - [x] Review README example
    - Related studies
      - [x] YJIT Talk: https://www.youtube.com/watch?v=PBVLf3yfMs8
- CS/Interpreters
  - [ ] Ruby Under a Microscope
    - [x] Chapter 1: Tokenization and Parsing
    - [ ] Chapter 2: Compilation

## Thu 23/Sep/2021

- Open source
  - TenderJIT
    - [ ] Review and understand project
    - [x] Check helper books
- Rust
  - [ ] Programming Rust (2nd ed.)
    - [x] Copy pending notes (p.381+)

## Wed 22/Sep/2021

- Projects/Emulation
  - [ ] Recompilation project
    - [x] Collect/store posts: https://make-a-demo-tool-in-rust.github.io and https://andrewkelley.me/post/jamulator.html
- Blog
  - [x] Update Disqus plan, and restore blog comments
- Rust
  - [ ] Programming Rust (2nd ed.)
    - [ ] Copy pending notes

## Tue 21/Sep/2021

- Projects/Emulation
  - [ ] Recompilation project
    - [x] Project preparation (import CHIP-8)
      - [x] Check roms licensing, and include in the project
      - [x] Read and test: https://matklad.github.io/2021/08/22/large-rust-workspaces.html
      - [x] Import project
        - [x] Minor cleanups
        - [x] Major simplifications of the source project
      - [x] Import and extend wiki
    - [x] Study: [Building a simple JIT in Rust](https://www.jonathanturner.org/building-a-simple-jit-in-rust)
      - [x] Refactor and experiment

## Mon 20/Sep/2021

- SWE/Career
  - [x] Check Shopify open source projects
- Planning
  - [x] Plan current cycle
- SWE
  - [x] Review/Pick SWE casts: https://twitter.com/wycats/status/1414002735771373568
- Rust
  - [x] What is `Cargo.toml` `default-members`?
  - [ ] Programming Rust (2nd ed.)
    - [x] Chapter 15: Iterators
- Projects
  - Schedule manager
    - [x] Add weekday support to "next"
      - [x] Fight Timecop bug
      - [x] Brutal fight with lexical ambiguity
    - [x] Minor parser refactoring
- Projects/Emulation
  - [ ] Recompilation project
    - [ ] Project preparation
      - [x] Review general information on compilation types

## Cycle notes

Completed the new plan.

- Not completed
  - Programming Rust (planned chapters)

## Sun 19/Sep/2021

- Writings
  - [x] Rust: Godbolt with any crate in VSC
    - [x] Experiment with Cargo crates structure
    - [x] Send link to Rust newsletter
- Emulation
  - Planning
    - [x] Review QEMU/C64 emulation resources
- Planning
  - [x] Complete the new plan, with details
- Rust
  - [ ] Programming Rust (2nd ed.)
    - [ ] Chapter 15: Iterators

## Sat 18/Sep/2021

- Rust/Low-level programming
  - [x] Refine/complete investigation on the disassembly procedures
- Writings
  - [ ] Rust: Godbolt with any crate in VSC

## Fri 17/Sep/2021

- Rust/Low-level programming
  - Practice/Tutoring
    - [x] Rust low-level inspection/profiling
  - [x] Experiment with analyzing the output of Rust asm tools
    - [x] Epic fight to find a visual approach (-> write article)

## Thu 16/Sep/2021

- Tools
  - VSC project creation tool
    - [x] Make VSC exclusion entries dynamic
    - [x] Some refactorings
    - [x] Add Rust workspace functionality
      - [x] Prepare `launch.json`
      - [x] Prepare `.cargo/config.toml`
    - [x] Add Rust independent project functionality
- Emulation
  - [ ] Review next emulation project

## Wed 15/Sep/2021

- Rust/Low-level programming
  - [ ] Optimization of random -1./+1
    - [x] Prepare project
- Planning
  - [x] Precisely plan mid term
    - [x] Further clean pool and remove current tasks
    - [x] Think new cycle format and topics
- Tools
  - Invoicing script
    - [x] Investigate/diagnose email client bug
  - VSC
    - [x] Fight multiproject
- Rust/Gamedev
  - [x] Check amdgpu compatibility with Bevy/other game engines (`https://github.com/bevyengine/bevy/discussions/2813`)

## Cycle notes

Radically reviewed and revamped the plan; dropped the current topics.

## Tue 14/Sep/2021

- Rust
  - Tutoring
    - [x] Chris introduction
    - [x] Review Zero to production (https://www.zero2prod.com/assets/sample_zero2prod.pdf)
    - [x] Collect and send project information/questions
- Rust/Low-level
  - [x] Read: https://medium.com/journey-to-rust/viewing-assembly-for-rust-function-d4870baad941
- Planning
  - [x] Clean subjects pool

## Mon 13/Sep/2021

- Admin
  - [x] Plan cycle
- Projects
  - Schedule manager
    - [x] Add lpim half/off day option
  - [ ] Port Code the Classics
    - [ ] Port Bunner to Tetra
      - [x] Base structure, including static intro screen

## Cycle changes

Small burnout 🤯 Also had comparatively little time, and the optimization research has been "one of those".

- Not completed
  - Rust: Programming Rust (2nd ed.)
    - Chapter 15: Iterators
    - Chapter 16. Collections
    - Chapter 17: Strings and Text
  - SWE/Rust: Optimization of random -1./+1
- Unscheduled
  - SWE/Rust: Tutoring
    - Find/contact candidates
  - Projects/Schedule manager
    - Some improvements/supporting code for the following
    - Relister: Allow missing dates (sections)
    - Relister: Filter out non-replan events
  - Rust/Gamedev
    - Test Arguably Better Breakout issue, and report it
  - Tools
    - Review Mermaid for class diagrams

## Sun 12/Sep/2021

- Rust/Low-level programming
  - [ ] Optimization of random -1./+1
    - [x] Post tutoring request
    - [x] Post question about specific case
- Rust
  - [ ] Programming Rust (2nd ed.)
    - [x] Copy notes chapter 13
    - [x] Copy notes chapter 14
- Tools
  - [x] Review Mermaid for class diagrams
- Rust/Gamedev
  - [x] Test Arguably Better Breakout issue, and report it
    - [x] Try OpenGL renderer backend
- Projects
  - Port Code the Classics
    - Port Infinite Bunner to Tetra
      - [x] Create architecture diagram

## Sat 11/Sep/2021

- Projects
  - Schedule manager
    - [x] Add debug mode
    - [x] Some improvements and supporting code for the following
    - [x] Relister: Allow missing dates (sections)
    - [x] Relister: Filter out non-replan events
- SWE/Rust
  - [ ] Tutoring
    - [x] Find/contact candidates

## Fri 10/Sep/2021

- Rust
  - [ ] Programming Rust (2nd ed.)
    - [x] Chapter 13: Utility Traits
    - [x] Chapter 14: Closures

## Thu 09/Sep/2021

- Projects
  - Port Code the Classics
    - Port Infinite Bunner to Tetra
      - [x] Accurate reading of the source code

## Wed 08/Sep/2021

- Rust
  - [ ] Programming Rust (2nd ed.)
    - [ ] Chapter 13: Utility Traits

## Tue 07/Sep/2021

- Rust/Low-level programming
  - [ ] Optimization of random -1./+1
    - [x] Benchmark different versions of random -1./+1.
    - [x] Review disassembly
    - [ ] Fight figure out how C version makes conditional version faster

## Mon 06/Sep/2021

- Projects
  - Port Code the Classics
    - Port Infinite Bunner to Tetra
      - [ ] Accurate reading of the source code
- Rust
  - [ ] Programming Rust (2nd ed.)
    - [ ] Chapter 13: Utility Traits

## Cycle changes

RACC took most of the week (more than expected); one day vga-passthrough, and a misc day.

- Unscheduled
  - zfs-installer
    - Add Debian 11 support (request)
  - geet
    - Make GitClient#cherry use the configurable main branch
  - schedule manager
    - RACC work has been more complex than expected
- Not completed
  - Rust/Gamedev: 6 total chapters of the two books
  - nv-switch article
  - Port the classics follow-up
  - SWE: reads

## Sun 05/Sep/2021

- Tools
  - [x] (Internal) Music management tool: Misc updates/cleanups
- Projects
  - Fish Fight
    - [x] Review site; plan future participation
- Admin
  - [x] End of cycle admin
    - [x] Merge temp resources into pool
    - [x] Buy new book
      - [x] Fight Packt horrible site
    - [x] Plan new cycle/projects

## Sat 04/Sep/2021

- Projects
  - geet
    - [x] Make GitClient#cherry use the configurable main branch
  - Fish Fight
    - [x] Move assets for old weapons (added by @64kramsystem) to temporary assets directory
  - Schedule manager
    - [x] Convert codec to RACC
      - [x] Tweak parser: return tokens instead of true/false, to make the manager token-oblivious
      - [x] Make period optional
      - [x] Wire parser
- Rust
  - [x] Search intermediate/advanced resources
    - [x] Review Jon Gjengset channel
- Ruby/CS
  - RACC parsing
    - [x] Simplify ReplanParser by using optional productions
    - [x] Review LL(*) parsers

## Fri 03/Sep/2021

- Ruby/CS
  - RACC parsing
    - [x] Complete parser for the schedule manager
- Projects
  - Schedule manager
    - [ ] Wire parser
  - Fish Fight
    - [x] Review new project, and discuss assets licensing

## Thu 02/Sep/2021

- Ruby/CS
  - RACC parsing
    - [x] Read: https://practicingruby.com/articles/parsing-json-the-hard-way

## Wed 01/Sep/2021

- Admin
  - [x] Review current major projects and plans
- Tools
  - Schedule manager
    - [x] Add `update` mode (`u`): ask user to update the current line
      - [x] Found funny workaround
- Projects
  - zfs-installer
    - [x] Discuss/think feature request: Install ZFS on systems with existing O/Ss
  - vga-passthrough
    - [x] Update audio passthrough section, about stability
    - [x] Create forwarder project for previous GitHub account
- Writings
  - [ ] nv-switch article
- Ruby/CS
  - RACC parsing
    - [x] Study RACC parser basics
    - [x] Build OptionsParser for the schedule manager
    - [ ] Encode series of optional values

## Tue 31/Aug/2021

- Linux
  - `date`: Study relative months arithmetic handling
- Tools
  - Invoicing script: Fix date handling
- System
  - [x] Follow up `amdgpu` bug
- Admin
  - [x] Update Github account overview
- Tools
  - [x] Add libreoffice date macro
  - Schedule manager
    - [x] Update replanner ugly message
    - [x] Remove debug messages
    - [ ] Add `update` mode (`u`): ask user to update the current line
      - [x] Fight prefill
      - [ ] Fight workaround to send a character to the terminal

## Mon 30/Aug/2021

- System
  - [x] Report amdgpu bug
  - [x] Report nvidia driver package bug
- Projects
  - zfs-installer
    - [x] Add Debian 11 support (request)
  - Drunken tomatoes
    - [x] Investigate new format for genres table
  - nv-switch
    - [x] Preliminary work: Make (almost all, as required) the hardcoded values parameters
      - [x] Large refactoring/extension required
    - [x] Add configuration management (https://github.com/64kramsystem/nv-switch/issues/2)
  - vga-passthrough
    - [x] Measure cards consumption
    - [x] Try 6600 XT passthrough, with update
      - [x] Test `nouveau`
      - [x] Check if `nvidia` supports `/dev/dri/by-path`, which includes bus id (and `/dev/dri/card*` reference)
      - [x] Test QEMU workarounds, alone and in combination
      - [x] Benchmark (vfio vs. top rankings)
- Hardware
  - Read: https://www.gamersnexus.net/guides/1229-anatomy-of-a-motherboard-what-is-a-vrm-mosfet

## Cycle changes

- Unscheduled
  - Ruby: Review RACC
  - Projects/Schedule Manager: Convert codec to RACC #1
  - Admin: Check licensing MIT, GPL, BSD
  - System+Writings+Projects: Improve VFIO card consumption (madness)
  - System: Madness 6600 XT + Linux reinstall + Windows install
  - Tools: Switch file sharing #1
  - Projects: `nv-switch`
  - Projects: qemu-pinning: Patch updates
  - Tools: Benchmark zstd
- Pushed
  - Projects/Schedule Manager
    Projects/Port Code the Classics
- Not completed
  - Rust: Programming Rust (2 chapters)

## Sun 29/Aug/2021

- System
  - [x] Full test 3d game on window
  - [x] Fight bug `lm-sensors` <> `amdgpu`
  - [x] Update Windows installation procedure/script
  - [x] Snapshot
  - [x] Configure GRUB
    - [x] Find workaround - adding a manual entry
- Rust
  - [ ] Programming Rust (2nd ed.)
    - [ ] Chapter 13: Utility Traits
- Writings
  - [ ] nv-switch article
    - [x] Find amdgpu counterpart to nvidia's device files `/dev/nvidia*`

## Sat 28/Aug/2021

- Projects
  - zfs-installer
    - [x] Update all repository links
- System
  - [x] Fight copy files
  - [x] Reinstall Linux
  - [x] Multiple fights with Windows installation
- Tools
  - [x] Benchmarks zstd

## Fri 27/Aug/2021

- Projects
  - nv-switch
    - [x] Fix switch bug
    - [x] Add support for partially unbound GPU devices
    - [x] Don't switch device driver if the new one is the same (https://github.com/64kramsystem/nv-switch/issues/4)
    - [x] Investigate `drivers_probe` interaction in the reference script (https://github.com/64kramsystem/nv-switch/issues/5)
- System
  - [x] Fight 6600 XT passthrough
    - [x] Study driver [infos](https://wiki.archlinux.org/title/AMDGPU)
    - [x] Update tools/settings
  - [x] Fight 6600 XT Linux native (!)

## Thu 26/Aug/2021

- Admin
  - [x] Fight find and buy 6600 XT
- System
  - [x] Cut new card heatsink and swap with old one
    - [x] Test power draw/speed
- Projects
  - nv-switch
    - [x] Implement install command  (https://github.com/64kramsystem/nv-switch/issues/1)
    - [x] Check processes before swapping out the nvidia driver (https://github.com/64kramsystem/nv-switch/issues/3)
- Tools
  - [x] Follow up Libreoffice GTK3 issue
    - [x] Test Libreoffice official packages (https://bugs.documentfoundation.org/show_bug.cgi?id=144033)

## Wed 25/Aug/2021

- Gamedev
  - [ ] Game Programming Algorithms and Techniques
    - [x] Chapter 06: Sound
- Projects
  - qemu-pinning
    - [x] Rebase patch to v6.0 stable, and replace master #34
      - [x] Think/drop `-changes` branch
- Rust/Gamedev
  - Macroquad
    - [x] Brief check at why `platformer` physics don't work
    - [x] Replace `physics-platformer` notes with example (`platformer.rs`)
- Admin
  - [x] Figth find 6600 XT

## Tue 24/Aug/2021

- Rust/Gamedev
  - Macroquad
    - [x] Refactor `platformer` example in order to better understand APIs (no functional changes)
      - [x] Investigate solid <> solid collision (study source)
- Gamedev
  - [ ] Game Programming Algorithms and Techniques
    - [x] Chapter 05: Input
    - [ ] Chapter 06: Sound
- Tools
  - [x] Follow up Libreoffice GTK3 issue
    - Reference: https://ask.libreoffice.org/t/calc-and-writer-ver-7-0-are-extremely-slow/58522

## Mon 23/Aug/2021

- Tools
  - [x] Investigate and report Libreoffice GTK3 issue
- Rust/Gamedev
  - Bevy
    - [x] Investigate relationship between `FixedTimestep` and states
  - Macroquad
    - [x] Experiment tilesets/properties with Tiled
    - [x] `platformer` example: Implement jumpthrough and descent (+open PR)
      - Fight existing issue with inconsistent height
- Projects
  - Improve VFIO card consumption
    - [x] Prepare all issues

## Sun 22/Aug/2021

- Tools
  - [x] Test open source alternative file sharing solution
    - [x] Preliminary investigation on how to command it from a root context
- Gamedev
  - [ ] Game Programming Algorithms and Techniques
    - [x] Chapter 03: Linear Algebra for Games (part)
    - [x] Chapter 04: 3D Graphics
- Projects
  - nv-switch
    - [x] Create project and publish first version

## Sat 21/Aug/2021

- System
  - Improve VFIO card consumption
    - [x] Write switch script
      - [x] Verify what `vfio-pci-bind.sh` does
      - [x] Fight Perl readline
    - [x] Investigate processes
    - [x] Investigate/apply permissions change strategy
      - [x] Attempt udev approach
      - [x] Brutal fight with card/services misbehavior

## Fri 20/Aug/2021

- System
  - Improve VFIO card consumption
    - [x] Verify unbinding without `nvidia-persistenced` running
    - [x] Study `nvidia-smi`

## Thu 19/Aug/2021

- Gamedev
  - [ ] Game Programming Algorithms and Techniques
    - [ ] Chapter 03: Linear Algebra for Games
- System
  - Improve VFIO card consumption
    - [x] Verify if any process runs on the second card, with `AutoAddGPU = false`
    - [x] Try `nvidia-smi` with different scenarios
    - [x] Try persistence mode
      - Found possible solution

## Wed 18/Aug/2021

- System
  - Improve VFIO card consumption
    - [x] Brutal fight with investigation
- Rust
  - [ ] Programming Rust (2nd ed.)
    - [x] Chapter 12: Operator Overloading

## Tue 17/Aug/2021

- Admin
  - [x] Check licensing MIT, GPL, BSD
  - [x] Create placeholder for former `qemu-pinning` repo
- Tools
  - [x] Fight bug with PDF reader
- Projects
  - Schedule manager
    - [x] Failed attempt to simplify the codec
- Ruby/CS
  - [x] Review RACC
- Rust
  - [ ] Programming Rust (2nd ed.)
    - [x] Copy notes; review traits
    - [x] Fight rustfmt VSC swallowed error
- Rust/Gamedev
  - Macroquad
    - [x] Study `physics-platformer` (todo: example)

## Mon 16/Aug/2021

- Rust/Gamedev
  - [ ] Study Macroquad platformer libraries
    - [x] Reorganize notes
    - [x] Study `tiled`
- Rust
  - [x] Reorganize Rust notes
- Projects
  - capistrano-sidekiq
    - [x] Add systemd version findings to README
    - [x] Add wiki page with Syslog-NG example

## Cycle changes

Half-length cycle.

- Unscheduled
  - Rust/Gamedev: Yet another review at game engines, after MQ shortcomings
  - Admin: Github account+blog move
  - Projects: `capistrano-sidekiq` issue report

## Sun 15/Aug/2021

- Admin
  - System
    - [x] Change hosting provider
    - [x] Rename Github account, and update projects (blog/zfs-installer)
  - Tools
    - Remote desktop issue
      - [x] Send logs
- Rust/Gamedev
  - [x] Study Tetra
    - [x] Study [examples](https://tetra.seventeencups.net/examples)
  - [ ] Study Macroquad platformer libraries
    - [x] First look at crates (source)

## Sat 14/Aug/2021

- Projects
  - Fish Fight
    - [x] Review and discuss clippy PR
- Rust
  - [ ] Programming Rust (2nd ed.)
    - [x] Chapter 11: Traits and Generics

## Fri 13/Aug/2021

- Projects
  - capistrano-sidekiq
    - [x] Investigate and open issue with the systemd unit
  - Fish Fight
    - [x] Review and discuss fmt/clippy PR, and investigate warning details
- Rust/Gamedev
  - Bevy
    - [x] Review FixedTimeStep source code
- Admin
  - Github account+blog move
    - [x] Gather general information on renaming account
    - [x] Plan
- Rust
  - [ ] Programming Rust (2nd ed.)
    - [x] Chapter 09: Structs
    - [x] Chapter 10: Enums and Patterns

## Thu 12/Aug/2021

- Rust
  - [ ] Programming Rust (2nd ed.)
    - [ ] Chapter 09: Structs

## Wed 11/Aug/2021

- Rust/Gamedev
  - [ ] Study Tetra
    - [ ] Study [examples](https://tetra.seventeencups.net/examples)
- Projects
  - [ ] Port Code the Classics
    - [ ] Port Infinite Bunner
      - [ ] Read chapter and code

## Tue 10/Aug/2021

- Rust/Gamedev
  - [x] Discuss z-ordering in Rust engines
  - [x] Review Tetra/Legion integration
    - [x] Make an "EC" version
  - [x] Read Legion/Specs ECS performance article: https://csherratt.github.io/blog/posts/specs-and-legion
  - [ ] Study Tetra
    - [x] Pong
- Projects
  - [ ] Port Code the Classics
    - [ ] Port Infinite Bunner
      - [ ] Read chapter and code

## Mon 09/Aug/2021

- Admin/tools
  - [ ] Remote desktop issue
    - [x] Try suggestion
  - [x] Write script to dump browser question bookmarks to CSV
  - [x] Add personal links to github profile
- Studies/Rust
  - [ ] Programming Rust (2nd ed.)
    - [x] Chapter 07: Error Handling
    - [x] Chapter 08: Crates and Modules
  - [x] Optimization problem
    - [x] Read all articles
      - https://renato.athaydes.com/posts/revisiting-prechelt-paper-comparing-languages.html
      - https://renato.athaydes.com/posts/how-to-write-slow-rust-code.html
      - https://renato.athaydes.com/posts/how-to-write-slow-rust-code-part-2.html
- Projects
  - [ ] Port Code the Classics
    - [x] Investigate Cavern music starting late
    - Engines (Rust)
      - [x] Review Z-ordering/other engines
      - [x] Review MacroQuad high-CPU issue and open bug
    - Engines (Ruby; all abandoned and broken)
      - [Metro](https://github.com/burtlo/metro)
      - [Chipmunk](https://github.com/chipmunk-rb/chipmunk): Physics engine
      - [Chingu](https://github.com/ippa/chingu)
      - [Rubygame](https://github.com/rubygame/rubygame)
      - [Gamebox](https://github.com/shawn42/gamebox): Simple Gosu wrapper

## Cycle changes

- Swapped
  - Projects
    - Fish fight <> Port Code the Classics (last days)
- Unscheduled
  - Projects
    - QEMU-pinning
      - Update patch to v6.1
    - ZFS
      - User issue discussion
      - Request: Add DNS ping test at the beginning
    - Awesome Rust
      - Rebase/Update RG3D game engine addition
  - Studies
    - Gamedev
      - Check out CS50's Introduction to Game Development content
      - Game Programming Algorithms and Techniques
      - Review Ruby 2D libraries
      - Study Gosu
    - Linux: Review disk SMART values
    - Ruby: Find out how to speedup gems installation
    - Blog: Article: Speeding up Ruby gems compilation (installation)
    - Rust
      - Programming Rust (2nd ed.)
      - Optimization problem
  - Admin
    - Tools: Programming journal manager: Add source file check; minor refactoring
    - Tools: Remote desktop issue investigation

## Sun 08/Aug/2021

- Studies/Rust
  - [x] Enum numerical casts
  - [ ] Optimization problem
    - [x] First review of the problem
    - [x] First review of the int data types and crates

## Sat 07/Aug/2021

- Studies/Rust
  - [ ] Programming Rust (2nd ed.)
    - [ ] Chapter 07: Error Handling

## Fri 06/Aug/2021

- Projects
  - [ ] Port Code the Classics
    - [x] Configuation/tooling updates related to project launch configurations
  - QEMU-pinning
    - [x] Update patch to v6.1.0-rc2
- Studies/Rust
  - [ ] Programming Rust (2nd ed.)
    - [x] Chapter 06: Expressions
  - [ ] Experiment with Prechelt phone number Rust benchmark
    - [x] Fight bash built-in time command

## Thu 05/Aug/2021

- Projects
  - Fish Fight
    - [x] Review and bisect control bug
  - [ ] Port Code the Classics
    - [ ] Implement state machine
      - [ ] Fight MacroQuad's `StateMachine` API
- Studies/Rust
  - [ ] Programming Rust (2nd ed.)
    - [ ] Chapter 06: Expressions
- Admin/Tools
  - [x] Follow up remote desktop issue investigation
- Studies/Gamedev
  - [x] First look at Tiled
  - [x] Review state of Rust game engines

## Wed 04/Aug/2021

- Admin/Other
  - [x] Select and buy books
- Admin/Tools
  - [x] Programming journal manager: Add source file check; minor refactoring
- Studies/Gamedev
  - [x] Study Gosu for prototyping
  - [x] Write a simple 1/2D scroller with Gosu
    - [x] Fight GitHub patch formatting
  - [ ] Game Programming Algorithms and Techniques
    - [ ] Chapter 03: Linear Algebra for Games

## Tue 03/Aug/2021

- Admin/Tools
  - [x] Upgrade system Clang version
    - [x] Change the symlinks strategy
- Studies/Rust
  - [ ] Programming Rust (2nd ed.)
    - [x] Chapter 04: Ownership and moves (quick read)
    - [x] Chapter 05: References
  - [x] Experiment with Prechelt phone number Rust benchmark
- Studies/Gamedev
  - [ ] Game Programming Algorithms and Techniques
    - [x] Chapter 02: 2D Graphics
  - [x] Check out Ruby game frameworks for prototyping
    - [x] Brief test Gosu
      - Metro adds extra APIs and convenient tools
    - [x] Check Ruby2D

## Mon 02/Aug/2021

- Studies/Rust
  - [ ] Programming Rust (2nd ed.)
    - [x] Chapter 03: Fundamental types
    - [ ] Chapter 04: Ownership and moves
  - [x] Investigate published Rust microbenchmark
- Studies/Gamedev
  - [ ] Game Programming Algorithms and Techniques
    - [x] Chapter 01: Game Programming Overview
    - [ ] Chapter 02: 2D Graphics
  - Macroquad
    - [x] Further look at StateMachine
- Projects
  - Fish Fight
    - [x] Add Cannonball to erupted items
      - [x] Make several refactorings

## Sun 01/Aug/2021

- Projects
  - Fish Fight
    - [x] Position discussion
    - [x] Open contract discussion
    - [x] Hitbox fixes due to recent changes
      - [x] Galleon: Clean and correct hitbox
      - [x] Cannon: Clean and correct hitbox
      - [x] Shark (rain): Clean and correct hitbox
    - Volcano: Add other weapons
      - [x] EruptingVolcano: Generalize item type spawning logic
    - [x] Prepare section for the Rust Gamedev newsletter
- Writings
  - Blog
    - [x] Article: Speeding up Ruby gems compilation (installation)
- Studies
  - Ruby
    - [x] Find out how to speedup gems installation
  - Computer Architecture
    - [x] Write Great Code, Volume 1: Understanding the Machine
      - [x] Chapter 14
      - [x] Chapter 15
  - Gamedev
    - Game Programming Algorithms and Techniques
      - [ ] Chapter 01

## Sat 31/Jul/2021

- Admin
  - [x] Handle DNS
  - Systems
    - [x] Review disk SMART values
- Studies/Computer Architecture
  - [ ] Write Great Code, Volume 1: Understanding the Machine
    - [x] Chapter 13
- Projects
  - Fish Fight
    - [ ] Minor cleanups
      - [x] Jellyfish: Clean and correct hitbox
      - [x] Curse: Clean and correct hitbox
      - [x] Curse: Implement throw
      - [x] Review alternate approach to shark rain positions

## Fri 30/Jul/2021

- Studies/Gamedev
  - [x] Check out CS50's Introduction to Game Development content
- Projects
  - QEMU-pinning
    - [x] [Update patch to v6.1](https://github.com/saveriomiroddi/qemu-pinning/issues/32)
  - Schedule manager
    - [x] Fail fast on invalid work lines
  - ZFS
    - [x] Follow up with user [issue](https://github.com/saveriomiroddi/zfs-installer/issues/229#issuecomment-887760469)
    - [x] Request: Add DNS ping test at the beginning
  - Fish Fight
    - [x] [Volcano env. weapon](https://github.com/fishfight/fish2/issues/48) MVP
      - [x] Items throw logic, with stub item
      - [x] Implement erupted item logic
      - [x] Rebase and update
  - Awesome Rust
    - [x] Rebase/Update RG3D game engine addition
- Admin/Tools
  - [x] Follow up remote desktop issue investigation
- Studies/Computer Architecture
  - [ ] Write Great Code, Volume 1: Understanding the Machine
    - [x] Chapter 11
    - [x] Chapter 12
    - [ ] Chapter 13

## Thu 29/Jul/2021

- Studies/Computer Architecture
  - [ ] Write Great Code, Volume 1: Understanding the Machine
    - [ ] Chapter 11
- Projects
  - Fish Fight
    - [ ] [Volcano env. weapon](https://github.com/fishfight/fish2/issues/48)
      - [x] Implement eruption emersion/submersion
      - [x] Items throw logic, with stub item
        - [x] Fight calculations

## Wed 28/Jul/2021

- Projects
  - Fish Fight
    - [x] Write environmental weapons document
    - [x] Review physics docs
    - [x] Fight broken project Cargo config
    - [ ] [Volcano env. weapon](https://github.com/fishfight/fish2/issues/48)
      - [x] Add Volcano (icon, not erupting)
      - [x] Instantiation, with base spawn data logic
- Admin/tools
  - [x] Remote desktop tool
    - [x] Follow up bug report

## Tue 27/Jul/2021

- Projects
  - ZFS
    - [x] Review and reply to new issue
  - OpenScripts
    - [x] Add script to rename variables/constants with composite names
  - Fish Fight
    - [x] Shark Rain
      - [x] Fight think efficient algorithm for random non-superposing distribution with equal space probability
- Studies/Rust+Gamedev
  - Macroquad
    - [x] Study Entity-Component-oriented APIs
- Studies/Rust
  - [x] Review `max`/`max_by` and related
- Admin/tools
  - [x] Review GitHub useful permissions for projects
- Studies/Computer Architecture
  - [ ] Write Great Code, Volume 1: Understanding the Machine
    - [x] Chapter 9
    - [x] Chapter 10 (not entirely)

## Mon 26/Jul/2021

- Projects
  - Geet
    - [x] Add upstream support to `pr open`
      - Final solution
  - Fish Fight
    - [x] Clean warnings
    - [x] Galleon
      - [x] Fight item not falling
      - [x] Fight thrown logic
  - ZFS
    - [x] Investigate issue feedback, and reply (https://github.com/saveriomiroddi/zfs-installer/issues/228)

## Sun 25/Jul/2021

- Projects
  - Fish Fight
    - [x] Jellyfish
      - [x] Implement gravity/flapping for FlappyJellyfish (without borders check…
      - [x] FlappyJellyfish: Terminate when going outside the borders
      - [x] FlappyJellyfish: Implement full states
        - [x] Brutal fight with throwing logic
      - [x] FlappyJellyfish: Don't spawn if colliding with a solid
      - [x] FlappyJellyfish: Collide with solid tiles (not wood)
      - [x] Adjust FlappyJellyfish fire tapping detection to new global input logic
      - [x] Final testing

### Cycle changes

- Swapped
  - Studies/SWE: Codewars <> Fish Fight
  - Rust/Gamedev: Better Breakout experiments <> Fish Fight
  - Rust/Gamedev: Port Code the Classics <> Fish Fight
- Pushed
  - FT/Materialization article
- Unscheduled
  - Admin/Tools: Test Win 10 to go + Clonezilla backup/restore from Ubuntu
  - Studies/Bash: Bash: Review different splitting patterns using `IFS`
  - Projects/Geet
    - Add support for draft PRs creation
    - Add upstream support
  - Projects/Schedule manager
    - Add support for `work -HH:MM -Nh` (hours in integer format)
    - Rescue errors, and print message, then exit (+convert one instance)
    - Minor refactorings
    - Add time section separators when adding a new date section
    - Add templating
  - Studies/Computer Architecture: Write Great Code, Volume 1: Understanding the Machine
  - Rust/Gamedev: Create Arguably Better Breakout separate repository
  - Rust/Gamedev: Setup Windows Rust env
  - Writing/Article: Compiling the CVE-2021-22555 exploit Proof of concept
  - Projects: QEMU research paper

## Sat 24/Jul/2021

- Projects
  - Fish Fight
    - [ ] Jellyfish
      - [x] Implement FlappyJellyfish control, without jumps and with basic state switch
        - [x] Brutal fight nodes
        - [x] Brutal fight input
          - [x] Fix bug
        - [x] Fight states
          - [x] Switch to remote control to simple boolean flag

## Fri 23/Jul/2021

- Projects
  - QEMU research paper
    - [x] Review presentation
      - [x] Read/fix typos, research topics
      - [x] Add photo
  - Fish Fight
    - [x] Cannon refinements
      - [x] Update assets/explosion
      - [x] Add bounciness
    - [x] Curse
      - [x] Make single-use
      - [x] Chase enemy instead of going straight towards them
      - [x] Minor cleanups/one fix
    - [x] Jellyfish
      - [x] Find/prepare basic assets
      - [x] Basic implementation
      - [x] Think design
        - [x] Failed attempt
        - [x] Successful design

## Thu 22/Jul/2021

- Projects
  - Fish Fight
    - [x] Implement Curse weapon
      - [x] Fight issue with throw (-> extra logic in unexpected location)

## Wed 21/Jul/2021

- Projects
  - Geet
    - [x] PR creation: Rename `upstream` branchs references/APIs to `remote`
    - [x] Other minor refactorings
    - [ ] Add upstream support to `pr open`
      - [x] Fight misbehaving/ignored `head` in GitHub PR API
  - Fish Fight
    - [x] Review new weapon specs

## Tue 20/Jul/2021

- Projects
  - Fish Fight
    - [x] Fix master gamepad compilation error
    - [x] Apply review notes Cannon
      - [x] Cannon: Allow self-pwning
      - [x] Add blowback
    - [x] Implement Parrot

## Mon 19/Jul/2021

- Projects
  - Fish Fight
    - [x] Complete cannon
    - [x] Improve Linux controllers detection
      - [x] Fight find simplest path handling patterns

## Sun 18/Jul/2021

- Admin/tools
  - [x] Report in depth issues with PDF tool
- Projects
  - Fish Fight
    - [x] Draw artwork
      - [x] Experiment and buy software
        - [x] Fight registration
        - [x] Report bug
    - [ ] Implement weapon
      - [ ] Big bomb
        - [x] Draw assets
        - [x] Implement base
        - [x] Implement grace time
        - [x] Add feature to debug hitboxes
        - [x] Fight animations

## Sat 17/Jul/2021

- Studies/Computer Architecture
  - [ ] Write Great Code, Volume 1: Understanding the Machine
    - [x] Chapter 7
    - [x] Chapter 8
    - [ ] Chapter 9

## Fri 16/Jul/2021

- Rust/Gamedev
  - Bevy
    - [x] Follow up with reported bug
      - [x] Prepare a Windows 10 USB installation
- Studies/Rust
  - [x] Review RefMut BCK issue
- Studies/Git
  - [x] Fetching GitHub PRs; write an alias
- Projects
  - Fish Fight
    - [x] Review PR
    - [x] Investigate/discuss Macroquad CPU occupation
    - [ ] Study grenade code
    - [ ] Think working bomb

## Thu 15/Jul/2021

- Writings
  - [x] Article: Compiling the CVE-2021-22555 exploit Proof of concept
- Projects
  - Fish Fight
    - [x] Test animation program(s)
    - [x] Preliminary discussions about the game
    - [x] Study jump code
    - [x] Investigate descend bug
    - [x] Review and remove redundant file
  - Geet
    - [x] Relax GitClient::UPSTREAM_BRANCH_REGEX
  - Bevy
    - [x] Follow up with reported bug
- Admin/Tools
  - Programming journal headers copy script
    - [x] Try multiple pattern match
    - [x] Add escaping
- Studies/Computer Architecture
  - [ ] Write Great Code, Volume 1: Understanding the Machine
    - [ ] Chapter 7

## Wed 14/Jul/2021

- Admin/tools
  - [x] Discord email notification madness
  - [x] Follow up discussion Fish Fight beta testing
- Rust/Gamedev
  - [ ] Experiment with Better Breakout code
    - [ ] Add states: Intro/Game Over
      - [x] Discuss/workaround timing issue with better breakout (known problem)
    - [x] Create separate repository
    - [x] Setup Windows Rust env
- Studies/SWE
  - Codewars
    - [x] 1 kata
- Studies/Computer Architecture
  - [ ] Write Great Code, Volume 1: Understanding the Machine
    - [x] Chapter 6
- Projects
  - Fish Fight
    - [x] Preliminary source code review
    - [x] Fix bug with error/stack trace printed when a joystick is not connected

## Tue 13/Jul/2021

- Admin/tools
  - [x] Write PDF manager support; buy pro version
- Studies/Computer Architecture
  - [ ] Write Great Code, Volume 1: Understanding the Machine
    - [x] Chapter 4
    - [x] Chapter 5
    - [ ] Chapter 6
- Rust/Gamedev
  - [x] Merge Bevy book + cookbook notes
  - [ ] Experiment with Better Breakout code
    - [ ] Add states: Intro/Game Over
      - [x] Epic general fight, including bug
      - [ ] Diagnose excessively slow state change
- Open source
  - Bevy Cheatbook
    - [x] Open issue about mistake in background color page
  - Bevy
    - [x] Discuss, created minimal test case, and report Bevy segfault bug

## Mon 12/Jul/2021

- Studies/Computer Architecture
  - [ ] Write Great Code, Volume 1: Understanding the Machine
    - [x] Chapter 3
- Studies/SWE
  - Codewars
    - [x] Review a few katas
- Rust/Gamedev
  - [ ] Experiment with Better Breakout code
    - [x] Study the codebase
    - [x] Restructure it
    - [x] Try remove FixedTimeStep, and use variable timestep

## Sun 11/Jul/2021

- Studies/Rust+Systems programming
  - [x] Rust in Action
    - [x] Chapter 12
    - [x] Copy notes/investigate topics
- Projects
  - Schedule manager
    - [x] Minor refactorings
    - [x] Add time section separators when adding a new date section
    - [x] Add templating
- Studies/SWE
  - Codewars
    - [x] 3 katas
- Studies/Computer Architecture
  - [ ] Write Great Code, Volume 1: Understanding the Machine
    - [x] Chapter 2

## Sat 10/Jul/2021

- Studies/Rust+Systems programming
  - [ ] Rust in Action
    - [x] Chapter 10
    - [x] Chapter 11

## Fri 09/Jul/2021

- Projects
  - Geet
    - [x] Add support for draft PRs creation
  - Schedule manager
    - [x] Add support for `work -HH:MM -Nh` (hours in integer format)
    - [x] Rescue errors, and print message, then exit (+convert one instance)
- Studies/Rust+Systems programming
  - [ ] Rust in Action
    - [x] Copy notes/investigate topics
    - [x] Chapter 9
    - [ ] Chapter 10

## Thu 08/Jul/2021

- Studies/Rust+Systems programming
  - [ ] Rust in Action
    - [x] Chapter 4 (peek)
    - [x] Chapter 5
    - [x] Chapter 6
    - [x] Chapter 7
    - [x] Chapter 8

## Wed 07/Jul/2021

- Open source/ZFS
  - [x] Support complex custom vdev configurations (e.g. multiple mirrors) #223
- Studies/Bash
  - [x] Review different splitting patterns using `IFS`
- Admin/Planning
  - [x] Plan next cycle
- Studies/Rust+Sytems programming
  - [ ] Rust in Action
    - [x] Peek at chapters 1 to 3
    - [ ] Chapter 4

## Tue 06/Jul/2021

- Admin/Tools
  - [x] Test Win 10 to go + Clonezilla backup/restore from Ubuntu
  - [x] Scripting: `create_vsc_project`
- Open source
  - ZFS
    - [x] Further improvement to disk name displayed in the pre-Subiquity dialog
    - [ ] Support complex custom vdev configurations (e.g. multiple mirrors) #223

## Mon 05/Jul/2021

- Studies/SWE
  - Competitive Programmer's Core Skills
    - [ ] Week 5
      - [ ] Videos/quizzes
  - Codewars
    - [x] 3 katas
- Open source/ZFS
  - [x] Close issue: Improve the description (device reference) in the Subiquity launch dialog
    - Fight find out where the prefixes come from
- Admin/Planning+Other
  - [ ] Plan next cycle

### Cycle changes

- Swapped
  - Programming from the ground up <> Competitive Programmer's Core Skills
- Unscheduled
  - Review Bevy Game Jam
  - Review Godot Wild Jam
  - Think jam participation
  - Codewars: katas
  - Personal repositories sync script: Add personal notes overwrite functionality
  - Architecture of consoles project: Review/open issue about presumed error in the PS1 article
  - Complete madness on open source issues
    - Investigate fonts issue, and open issue on fonts project
    - Rust-analyzer: Fight/open two issues with files exclusion
- Not completed
  - Port Code the Classics

## Sun 04/Jul/2021

- Studies/SWE
  - Competitive Programmer's Core Skills
    - [ ] Week 4
      - [ ] Assignment
        - [ ] Problem 5
  - Codewars
    - [x] Several katas

## Sat 03/Jul/2021

- Studies/SWE
  - Competitive Programmer's Core Skills
    - [ ] Week 4
      - [ ] Assignment
        - [x] Problem 2
        - [ ] Fight Problem 3
        - [x] Problem 4

## Fri 02/Jul/2021

- Studies/SWE
  - Competitive Programmer's Core Skills
    - [ ] Week 5
      - [ ] Videos/quizzes

## Thu 01/Jul/2021

- Projects/Open source
  - tw-emoji-color-font
    - [x] Test on stock install, and follow up discussion
  - Rust-analyzer
    - [x] Provide test case
- Studies/SWE
  - Competitive Programmer's Core Skills
    - [x] Week 3
      - [x] Assignment
        - [ ] Review binary knapsack problem solution
    - [ ] Week 4
      - [ ] Videos/quizzes
      - [ ] Assignment
        - [x] Problem 1

## Wed 30/Jun/2021

- Projects/Open source
  - Complete madness on open source issues
    - [x] Investigate fonts issue, and open issue on fonts project
      - [x] Further investigation and report
    - Rust-analyzer
      - [x] Fight/open two issues with files exclusion
        - [x] Further investigation and report
- Studies/SWE
  - Codewars
    - [x] One kata
- Studies/Comparch
  - Architecture of consoles project
    - [x] Review/open issue about presumed error in the PS1 article
- Admin/tools
  - Personal repositories sync script
    - [x] Add personal notes overwrite functionality
- Rust/Gamedev
  - Port Code the Classics
    - [x] Refactoring/extension of `add_new_project.sh`

## Tue 29/Jun/2021

- Studies/SWE
  - Competitive Programmer's Core Skills
    - [ ] Week 3
      - [x] Videos/quizzes
      - [x] Assignment
        - Fight binary knapsack problem
  - Codewars
    - [x] A few katas

## Mon 28/Jun/2021

- Studies/SWE
  - Competitive Programmer's Core Skills
    - [x] Week 2
      - [x] Assignment

## Sun 27/Jun/2021

- Studies/SWE
  - Competitive Programmer's Core Skills
    - [ ] Week 2
      - [x] Videos/readings
      - [x] Practice quiz
      - [ ] Assignment
        - [x] Problem 1

## Sat 26/Jun/2021

- Studies/SWE
  - Competitive Programmer's Core Skills
    - [ ] Week 2
      - [ ] Videos/readings

## Fri 25/Jun/2021

- Studies/SWE
  - Competitive Programmer's Core Skills
    - [ ] Week 1
      - [x] Programming assignment
        - Fight misinterpreted problem
    - [ ] Week 2
      - [ ] Videos/readings

## Thu 24/Jun/2021

- Rust/Gamedev
  - [x] Review [Godot Wild Jam](https://itch.io/jam/godot-wild-jam-33)
  - [ ] Think jam participation
  - Port Code the Classics
    - [ ] Cavern
      - [ ] Redesign with Macroquad APIs
        - [ ] Review Fish Fight source code/APIs
          - `main.rs`, `player.rs`

## Wed 23/Jun/2021

- Studies/SWE
  - Competitive Programmer's Core Skills
    - [ ] Week 1
      - [x] Videos/readings
      - [x] Practice quiz
      - [ ] Programming assignment

## Tue 22/Jun/2021

- Rust/Gamedev
  - Port Code the Classics
    - [ ] Cavern
      - [x] Port to Macroquad (naive)
  - [x] Investigate and report ggez issue with key repetition test
  - [x] Review Bevy Game Jam
- Studies/Rust
  - [x] Investigate From trait implementation for boxed serde `ErrorKind` type

## Mon 21/Jun/2021

- Admin/Planning+Other
  - [x] Plan next cycle
    - [x] Basic research SWE books/courses
  - [x] Split brogramming3 into bad/discarded
- Admin/Tools
  - [x] Network connection script: Don't connect if already connected to the given network
  - [x] Music playback script: Add support for multiple album locations
- Projects/Open source
  - OpenScripts
    - [x] Create Bash script templating script
- Admin/Blog
  - [x] Remove `small` tag (retag articles to `quick`)
- Studies/Rust
  - [ ] Investigate From trait implementation for boxed serde `ErrorKind` type
- Studies/Systems programming
  - [ ] Programming from the ground up
    - [x] 1. Introduction
    - [x] 2. Computer Architecture
- Rust/Gamedev
  - Port Code the Classics
    - [ ] Cavern
      - [ ] Port to Macroquad (naive)

### Cycle changes

- Discarded
  - Hands-On Data Structures and Algorithms in Rust
    - 6.42. Combining It All into a Simple CLI Game
    - 6.43. Introduction to Specs
- Unscheduled
  - Schedule manager
    - Refactoring: Create launcher class (Replan)
    - Add Reworker test suite
  - Geet
    - De-hardcode the `master` string (constant was already existing)
  - Blog
    - Disable Disqus + looks at alternatives
  - Admin/Tools
    - Improvements to blog's `add_new_book.sh` script
  - Admin/Systems
    - Fight mistaken malware on blog (-> Disqus)
    - Fight investigate system slowdown (-> Firefox)
    - Reinstall desktop O/S
      - Fight ZFS
      - Updates to installation script
    - Fight investigate suspend/resume (-> Ethernet driver bug)
    - Profile and investigate slow Nvidia Linux games performance
  - ZFS
    - Massive work
  - Rust/Gamedev
    - Investigate and discuss low responsiveness issue (-> framerate; game loop design)
    - Article: https://medium.com/@tglaiel/how-to-make-your-game-run-at-60fps-24c61210fe75
    - Port Code the Classics
      - Yet Another Research on Rust game engines
      - Scripting updates

## Sun 20/Jun/2021

- Rust/Gamedev
  - [ ] Port Code the Classics
    - [ ] Cavern
      - [x] Design
      - [ ] Port to Macroquad (naive)

## Sat 19/Jun/2021

- Rust/Gamedev
  - [ ] Port Code the Classics
    - [ ] Cavern
      - [ ] Port to Macroquad (naive)

## Fri 18/Jun/2021

- Admin/System
  - [x] Profile and investigate slow Nvidia Linux games performance
- Rust/Gamedev
  - [ ] Port Code the Classics
    - [x] Review Macroquad APIs
    - [x] Yet Another Research on Rust game engines
    - [x] Scripting updates
    - [ ] Cavern
      - [ ] Port to Macroquad (naive)
  - [x] Test port
  - [x] Follow up issue with bevy cheatbook

## Thu 17/Jun/2021

- Admin/System
  - [x] Fight system not suspending (-> new ethernet driver version)
- Projects/Open source
  - [x] QEMU research paper
    - [x] Address remaning review issues

## Wed 16/Jun/2021

- Rust/Gamedev
  - [x] Investigate and discuss low responsiveness issue (-> framerate; game loop design)
  - [ ] Article: https://medium.com/@tglaiel/how-to-make-your-game-run-at-60fps-24c61210fe75
    - [ ] Check out similar articles
- MySQL
  - [x] FT indexes administration deep dive (theory and first tests)

## Tue 15/Jun/2021

- Rust/Gamedev
  - Study Macroquad
    - [ ] [Making an online multiplayer game in Rust with Nakama](https://heroiclabs.com/blog/tutorials/rust-fishgame)
    - [x] Investigate low responsiveness (-> Linux)
- Admin/Other
    - [x] Fight Linux OpenGL drivers
    - [x] Compare performance with other machine
      - [x] Research and buy new card

## Mon 14/Jun/2021

- Projects/Open source
  - ZFS
    - [x] Replicate 0.5 issues on a VM
      - [x] Create bpool dataset(s), in order to avoid GRUB warning
    - [x] Backport non-procedural changes from master into `0.3_backports`
    - [x] Backport: Update pool creation features to updated procedure
    - [x] Backport: Fix /boot mounts mess (part 1: timeout)
    - [x] Backport: Fix /boot mounts mess (part 2: /boot dependency)
    - [x] Backport: Use UUID rather than PARTUUID for boot partitions in the fstab
      - [x] Fix EFI mounts (from 1 onwards) filesystem UUID
    - [x] Master+Backport: Refactoring: Move fstab preparation into separate step
    - [x] Backport: Generalize fix filesystem mount ordering
    - [x] Backport: Improve old update_zed_cache_Debian(), following the new procedure
    - [x] Backport: Datasets support
      - [x] 8f75a078aa Add support for datasets
      - [x] 198afb8762 Allow dataset creation options customization via env variable
      - [x] 4fd8ee1c8a Complete KUbuntu support
      - [x] d7518ad12b Don't set the permissions on rpool directories
      - [x] Test
    - [x] All clean
      - [x] Test Ubuntu Server on backport
      - [x] Test Debian on backport
      - [x] Test bpool dataset(s) on 0.3_backports
      - [x] Test new procedure
        - [x] Run and review errors (without bpool datasets)
        - [x] Apply the backport fixes, where applicable
        - [x] Try bpool datasets
        - [x] Test if Ubuntu Server works
        - [x] Test if Debian works
    - [x] Merge backport, as separate file
  - [x] Backport: Implement bpool datasets, with Debian required GRUB change
  - [x] Backport: Set fstab dump `0`
  - [x] Add Debian support for new procedure and drop backport
  - [x] Fix build

## Sun 13/Jun/2021

- Admin/Other
  - [x] Reinstall desktop O/S
    - [x] Update all the scripts
      - [x] Multifunction: investigate current configuration and old one
    - [x] Fight Linux boot desktop (-> nvidia drivers)
    - [x] Fight ZFS/grub

- Projects/Open source
  - ZFS
    - [x] Handle Ubiquity now emptying `/run`
    - [x] Don't set the permissions on rpool directories
    - [x] Backport the majority (but simpler) of commits to 0.3
    - [x] Officially close project

## Sat 12/Jun/2021

- Admin/Other
  - [ ] Reinstall desktop O/S
    - [x] Update all the scripts
    - [ ] Fight Linux boot desktop (nvidia drivers)

## Fri 11/Jun/2021

- Rust/CS
  - [x] Hands-On Data Structures and Algorithms in Rust
    - [ ] Section 7: Storage
      - [x] 46. Creating a Blob Store
      - [x] Quiz 7: Test your knowledge
- Admin/Tools
  - [x] Improvements to blog's `add_new_book.sh` script
- Admin/System
  - [x] Fight system not suspending

## Thu 10/Jun/2021

- Rust/CS
  - Hands-On Data Structures and Algorithms in Rust
    - [ ] Section 7: Storage
      - [ ] 46. Creating a Blob Store

## Wed 09/Jun/2021

- Rust/CS
  - Hands-On Data Structures and Algorithms in Rust
    - [x] Section 6: Entity component systems
      - [x] 40. Creating Data Stores
      - [x] 41. Building ECS Systems
      - [x] ~~42. Combining It All into a Simple CLI Game~~
      - [x] ~~43. Introduction to Specs~~
      - [x] Quiz 6: Test your knowledge
    - [ ] Section 7: Storage
      - [x] 44. Creating a Blob Data File
      - [x] 45. Converting Any Data Size to Byte String
- Admin/Systems
  - [x] Fight investigate system slowdown (-> Firefox)
- Admin/Blog
  - [x] Disable Disqus
    - [x] Second look at alternatives

## Tue 08/Jun/2021

- Geet
  - [x] Issue: Remove irrelevant `base` param
  - [x] Merge PR: Switch to main branch (instead of master) after the merge
  - [x] PR creation: Use main branch (as default) instead of hardcoded `master`
- Rust/Assembler
  - [x] Investigate compiled ASM of different ggez function versions, and report on ggez
- Rust/CS
  - Hands-On Data Structures and Algorithms in Rust
    - [ ] Section 6: Entity component systems
      - [ ] 40. Creating Data Stores

## Mon 07/Jun/2021

- Admin
  - Planning/Other
    - [x] Plan next cycle
    - [x] Fight mistaken malware on blog (-> Disqus)
  - Tools
    - [x] Git aliases: Support non-master branch
- Projects/Open source
  - Openscripts
    - purge_trash
      - [x] Continue in case of removal error
      - [x] Support GUI run for runtime errors notification
  - Schedule manager
    - [x] Move all the skip replans also in regular mode
    - [x] Replace skip_only with nomove functionality
    - [x] Refactoring: Create launcher class (Replan)
    - [x] Add Reworker test suite
    - [x] Reworker: Add HH:MM -H.MMh format (and reverse)
  - Geet
    - [x] De-hardcode the `master` string (constant was already existing)
    - [ ] Add support for non-`master` development branch
      - [x] Add internal support for non-`master` development branches
- Writings
  - [x] Article: Conveniently handling non-master development/default branches in Git(/Hub)

### Cycle changes

- Discarded
  - Bevy Cheat Book: Correct all examples
  - geet
    - Restructure `pr create -A`
- Unscheduled
  - Schedule Manager: Fix non-replan skip event decoding
  - ggez
    - PR: Snake example: Simplify non-negative modulo computation, by using rem_euclid() API
    - Open discussion about improving code based on maintainer comment
      - Fight implementation
    - PR: Snake example: Use the library's timer rather than manual time tracking
  - Rust/Gamedev
    - Deeper review at Code the Classics API requirements, and Macroquad
    - Yet another look at game engines (activity)
    - Port Code the Classics
      - Write script for adding and preparing new projects
  - ZFS
    - Fix reported issue: Openzfs ppa crashes Ubuntu server 20.04 installer
    - Fix new issue introduced by 20.04.2
    - Disable the PPA on UbuntuServer
    - Complete reported issue v0.4x breaks update-grub
  - Admin
    - Tools
      - Research and tweak plugin for printing code (to pdf)
  - Rust
    - Review supertraits and OO-related concepts
  - Gamedev
    - Talk: Exploring the Tech and Design of Noita
- Not completed
  - Rust/CS
    - Hands-On Data Structures and Algorithms in Rust
      - [ ] Section 6: Entity component systems
        - [ ] 40. Creating Data Stores
        - [ ] 41. Building ECS Systems

## Sun 06/Jun/2021

- Projects/Open source
  - ZFS
    - [x] Complete issue [v0.4x breaks update-grub](https://github.com/saveriomiroddi/zfs-installer/issues/199)
- Rust
  - [x] Review supertraits and OO-related concepts
- Rust/Gamedev
  - [x] Yet another look at game engines (activity)
  - Port Code the Classics
    - [ ] Cavern
      - [ ] Read/understand code
- Rust/CS
  - Hands-On Data Structures and Algorithms in Rust
    - [x] Section 5: Hash maps
      - [x] 36. Testing and Improving Our HashMap
      - [x] Quiz 5: Test your knowledge
  - [ ] Section 6: Entity component systems
    - [x] 38. Understand What an ECS Is and How It Differs from Traditional Structures
    - [x] 39. Creating an ID Generator
- Gamedev
  - [x] Talk: [Exploring the Tech and Design of Noita](https://www.youtube.com/watch?v=prXuyMCgbTc)
- Admin
  - Tools
    - [x] Research and tweak plugin for printing code (to pdf)

## Sat 05/Jun/2021

- Rust/Gamedev
  - Port Code the Classics
    - [x] Boing
      - [x] Think improvements to original design
    - [x] Write script for adding and preparing new projects
    - [ ] Cavern
      - [ ] Read/understand code
- Projects/Open source
  - ZFS
    - [x] Fix: [Openzfs Ppa crashes Ubuntu server 20.04 installer](https://github.com/saveriomiroddi/zfs-installer/issues/200)
    - [x] Fix new issue introduced by 20.04.2
    - [x] Disable the PPA on UbuntuServer
    - [ ] Fix: [v0.4x breaks update-grub](https://github.com/saveriomiroddi/zfs-installer/issues/199)

## Fri 04/Jun/2021

- Rust/Gamedev
  - Port Code the Classics
    - [ ] Boing
      - [x] Port
      - [x] Fight send showcase
      - [x] Unfork repo

## Thu 03/Jun/2021

- Rust/Gamedev
  - Port Code the Classics
    - [x] Design: Control functions
    - [ ] Port

## Wed 02/Jun/2021

- Rust/Gamedev
  - Port Code the Classics
    - [ ] Boing
      - [ ] Design: Fight control functions

## Tue 01/Jun/2021

- Rust/Gamedev
  - Port Code the Classics
    - [ ] Boing
      - [ ] Port
        - [x] Fight missing ggez dependency
        - [x] Fight closure/function passing, item type issue

## Mon 31/May/2021

- Rust/Gamedev
  - Port Code the Classics
    - [x] Deeper review of the API requirements
    - [ ] Boing
      - [ ] Port
  - [x] Deeper review of Macroquad

## Sun 30/May/2021

- Projects/Open source
  - ruby-packer
    - [x] Discussion maintenance
- Rust/Gamedev
  - Prepare for porting Code the Classics: Readings
    - [x] ggez API examples
      - [x] `sounds.rs`
      - [x] `files.rs`
      - [x] `canvas_subframe.rs`
      - [x] `hello_canvas.rs`

## Sat 29/May/2021

- Rust/Gamedev
  - Prepare for porting Code the Classics: Readings
    - [ ] ggez API examples
      - [x] `graphics_settings.rs`
      - [x] `input_test.rs`
- Admin
  - Planning/Other
    - [x] Review/buy books for next cycles

## Fri 28/May/2021

- Projects/Open source
  - Geet
    - [x] Workaround CLI array arguments causing an error
  - Blog
    - [x] `add_new_book.sh`: Use the cover filename as book name
- Rust
  - [ ] Investigate compiled ASM of different ggez function versions
- Rust/Gamedev
  - Prepare for porting Code the Classics: Readings
    - [ ] ggez API examples
      - [x] `transforms.rs`
      - [ ] `graphics_settings.rs`

## Thu 27/May/2021

- Projects/Open source
  - Geet
    - [ ] Review `pr create -A`
    - [ ] Review case sensitiveness
- Studies
  - Rust
    - [ ] Investigate compiled ASM of different ggez function versions

## Wed 26/May/2021

- Projects/Open source
  - ggez
    - [x] PR: Snake example: Simplify non-negative modulo computation, by using rem_euclid() API
    - [x] Open discussion about improving code based on maintainer comment
      - [x] Fight implementation
      - [x] Write detailed explanation
    - [x] PR: Snake example: Use the library's timer rather than manual time tracking
- Rust/Gamedev
  - Prepare for porting Code the Classics: Readings
    - [x] ggez game examples
    - [ ] ggez API examples
      - [x] `bunnymark.rs`
      - [x] `spritebatch.rs`
      - [x] `meshbatch.rs`
      - [x] `imageview.rs`

## Tue 25/May/2021

- Projects/Open source
  - Schedule manager
    - [x] Fix non-replan skip event decoding
- Rust/Gamedev
  - Prepare for porting Code the Classics: Readings
    - [ ] ggez game examples
      - [x] Fight `timer::check_update_time()`
      - [x] Fight mistaken interpretation of unary sign

## Mon 24/May/2021

- Rust/Gamedev
  - Prepare for porting Code the Classics: Readings
    - [x] Yet another round of research and discussion
    - [ ] ggez game examples
  - Port Code the Classics
    - [ ] Boing
      - [x] Read/understand code

## Sun 23/May/2021

- Projects/Open source
  - Intelleggere
    - [x] Prepare DDL Zan PR
      - [x] Addition DL 19 maggio 2020, n. 34, convertito, con modificazioni, dalla legge 17 luglio 2020, n. 77
      - [x] DL changes
        - [x] Application Art.9 in differential form
    - [x] Final cleanup repository
    - [x] Close project and add explanation
- Admin
  - Planning/Other
    - [x] Plan cycle
- Rust/Gamedev
  - Prepare for porting Code the Classics: Readings
    - [ ] Code the Classics book
      - [ ] Boing

### Cycle changes

- Discarded
  - The ZX Spectrum ULA - How to Design a Microcomputer
  - AWS Scripts
    - Implement 300k limit
    - Final testing
  - Intelleggere
    - Create blog project
    - Write post
- Unscheduled
  - Schedule manager
    - Minor updates/improvements
    - Refactor regex (tokens) handling
    - Add basic test suite
    - Add new format for work definition (`-MM -HH:MM`)
    - Fix skip mode: Don't move the current day
  - Hotplugger
    - Add executable bit and open PR
    - Test deprecated options format on 5.0, and open PR
  - Admin
    - Restructure home network
  - QEMU script
    - Make performance governor setting optional
    - Make QEMU killer optional
  - Code the Classics
    - PR: Provide license
  - Bevy Cheatbook
    - PR: Fix `assets-ready.rs` example
    - PR: Add group assets loading tracking example
    - PR: `AssetServer#get_group_load_state`: Convert the iterator to borrowed version
    - Discuss on Discord/Issue async assets loading
    - Discuss on Discord manual but idiomatic state handling
  - Awesome Bevy
    - PR: Add Flappy Bevy game
  - Rust/Gamedev
    - Review simple 2D engines
  - Recipeek
    - Upgrade to Rails 4.2
- Swapped
  - Bevy beginner issues <> Convert Code the classics book games to Rust/Bevy
  - Hands-On Data Structures and Algorithms in Rust: Chapter 4 <> 5
  - Snake tutorial <> Better Breakout PR
- Pushed
  - Convert Code the classics book games to Rust/Bevy

## Sat 22/May/2021

- Projects/Open source
  - Recipeek
    - [x] Upgrade to Rails 4.2
  - Bevy
    - [x] Discuss on Discord manual but idiomatic state handling
    - [x] Discuss on issue the proposed asset loading solution
  - Intelleggere
    - [ ] Prepare DDL Zan PR
      - [x] Addition DL 9 luglio 2003, n. 215
      - [ ] DL changes
        - [x] Application Art.8 in differential form
- Admin
  - [x] Restructure home network

## Fri 21/May/2021

- Projects/Open source
  - Bevy
    - [ ] PR: Improve Better Breakout assets loading
      - [x] Discuss on Discord/GitHub about the issue
      - It seems it can't be currently accomplished
- Studies
  - Rust/Gamedev
    - [x] Go through all engines again (Bevy is suitable only for very simple stuff)

## Thu 20/May/2021

- Projects/Open source
  - Bevy Cheatbook
    - [x] Apply feedback: merge examples, and simplify logic
  - Bevy
    - [ ] PR: Improve Better Breakout assets loading
      - [ ] Fight find synchronous strategy
- Studies
  - Rust/Gamedev
    - [ ] Go through actual games
      - [ ] Bevy examples: Better Breakout

## Wed 19/May/2021

- Projects/Open source
  - Bevy cheatbook
    - [x] Discuss best format for example
  - Bevy
    - [x] Discuss Better Breakout systems dependency tree and assets loading
- Studies
  - Rust/Gamedev
    - [ ] Go through actual games
      - [ ] Bevy examples: Better Breakout

## Tue 18/May/2021

- Projects/Open source
  - Intelleggere
    - [ ] Prepare DDL Zan PR
      - [ ] Addition DL 9 luglio 2003, n. 215
      - [ ] DL changes
        - [x] Application Art.5 in differential form
  - Code the Classics
    - [x] Provide license (PR)
  - Bevy Cheatbook
    - [x] PR: Fix `assets-ready.rs` example
    - [x] PR: Add group assets loading tracking example
  - Bevy
    - [x] PR: `AssetServer#get_group_load_state`: Convert the iterator to borrowed version
  - Awesome Bevy
    - [x] PR: Add Flappy Bevy game
- Studies
  - Rust/Gamedev
    - [ ] Go through all the tutorials/books
      - [x] Unofficial Bevy Cheat Book
        - [x] 3. Bevy Features
        - [x] 4. Common Pitfalls
        - [x] ~~5. Advanced Patterns~~
        - [x] 6. Bevy Cookbook
    - [x] Review [YouTube tutorials](https://duckduckgo.com/?q=bevy+rust&k5=1&iar=videos&iax=videos&ia=videos)
  - Rust/CS
    - Hands-On Data Structures and Algorithms in Rust
      - [ ] Section 5: Hash maps
        - [x] 35. Finishing the HashMap

## Mon 17/May/2021

- Projects/Open source
  - Intelleggere
    - [x] Restructure project, primarily to allow straight DDL reading
    - [ ] Prepare DDL Zan PR
      - [x] Extra-code additions
        - [x] Art. 1, definitions
        - [x] Art. 4, exceptions
        - [x] Art. 7, new festivity
        - [x] Art. 10, stats
      - [x] Addition DL Mancino (1993/04/26, n.122)
  - Bevy
- Studies
  - Rust/Gamedev
    - [ ] Go through all the tutorials/books
      - [ ] Unofficial Bevy Cheat Book
        - [x] 2. Bevy Programming
          - [x] 2.2: Next Steps
          - [x] 2.3: Advanced
    - [x] Review Wireframe magazines/books
  - Rust/CS
    - Hands-On Data Structures and Algorithms in Rust
      - [ ] Section 5: Hash maps
        - [x] 33. Building HashMap from Scratch
        - [x] 34. Building a Bucket List for the HashMap to Use
          - [ ] Review `?Sized` trait
          - [ ] Review `Borrow` trait

## Sun 16/May/2021

- Studies
  - Rust/Gamedev
    - [ ] Go through all the tutorials/books
      - [ ] Unofficial Bevy Cheat Book
        - [ ] 2. Bevy Programming
          - [x] 2.1: Basics
          - [ ] 2.2: Next Steps
- Admin
  - Tools
    - QEMU script
      - Make performance governor setting optional 
      - Make QEMU killer optional

## Sat 15/May/2021

- Studies
  - Rust
    - [x] Refresh some subjects
  - Rust/Gamedev
    - [ ] Go through all the tutorials/books
      - [ ] Unofficial Bevy Cheat Book
        - [ ] 2. Bevy Programming
          - [ ] 2.1: Basics

## Fri 14/May/2021

- Studies
  - Rust/Gamedev
    - [ ] Go through all the tutorials/books
      - [ ] Bevy examples: Breakout
        - Check also the improved version
    - [ ] Unofficial Bevy Cheat Book
      - [ ] 2. Bevy Programming
        - [ ] 2.1: Basics
        - [x] ~~7. Bevy on Different Platforms~~

## Thu 13/May/2021

- Studies
  - Rust/Gamedev
    - [x] Review Rust 2D engines
  - Rust/CS
    - Hands-On Data Structures and Algorithms in Rust
      - [ ] Section 4: Graphs
        - [ ] 29. Finding the Shortest Path
        - [ ] 30. A Greedy Solution to the Travelling Salesman
- Tools
  - Schedule manager
    - [x] Fix skip mode: Don't move the current day

## Wed 12/May/2021

- Projects/Open source
  - Intelleggere
  - [ ] Prepare DDL Zan PR
    - [x] Understand legislative process paper trail
      - [x] Follow the DL trail with new instructions
- Studies
  - Rust/Gamedev
    - [ ] Go through all the tutorials/books
      - [x] ~~Chess game in Rust using Bevy~~
        - [x] Fight assets load/path issue
      - [x] Review Path/PathBuf concepts
      - [x] Bevy book
  - Rust/CS
    - Hands-On Data Structures and Algorithms in Rust
      - [ ] Section 4: Graphs
        - [x] 27. Filling the Graph
        - [x] 28. Route Structure

## Tue 11/May/2021

- Projects/Open source
  - Intelleggere
    - [x] Create project repository
    - [ ] Prepare DDL Zan PR
      - [x] Add base articles for the bill (Part 1)
        - [x] 604-bis
        - [x] 604-ter
        - [x] 90-quater
      - [x] Prepare diff Bill articles (Part 1: direct articles)
        - [x] Art.2
        - [x] Art.3
        - [x] Art.6
      - [ ] Understand legislative process
        - [x] Find forums for question
        - [x] Write down exact questions
  - Hotplugger
    - [x] Add executable bit and open PR
    - [x] Test deprecated options format on 5.0, and open PR
- Admin
  - Tools
    - [x] QEMU VFIO: Test USB ports passthrough: https://github.com/darkguy2008/hotplugger
      - Temporarily disable powerdown functionality
- Studies
  - Rust/Gamedev
    - [x] Review [Bevy learning material](https://github.com/bevyengine/awesome-bevy#learning)
    - [ ] Go through all the tutorials/books
      - [ ] [Chess game in Rust using Bevy](https://caballerocoll.com/blog/bevy-chess-tutorial)

## Mon 10/May/2021

- Admin
  - Tools
    - Schedule manager
      - [x] Add support for half/off days
      - [x] Open source the project
        - [x] Add repo support stuff
- Studies
  - Comparch
    - [x] ~~The ZX Spectrum ULA - How to Design a Microcomputer~~
      - [ ] ~~13. Video Memory Access~~
      - [ ] ~~14. Video Control Clocks~~
  - Gamedev/SWE
    - [x] Game Programming Patterns
      - [x] VI. Optimization Patterns
        - [x] 19. Object Pool
        - [x] 20. Spatial Partition
      - [x] IV. Behavioral Patterns
        - [x] ~~11. Bytecode~~ (standard VM stuff)
        - [x] 12. Subclass Sandbox
        - [x] 13. Type Object

## Sun 09/May/2021

- Admin
  - Tools
    - [x] Schedule manager
      - [x] Minor updates/improvements
      - [x] Refactor regex (tokens) handling
      - [x] Add basic test suite
      - [x] Add new format for work definition (`-MM -HH:MM`)
- Studies
  - Comparch
    - [ ] The ZX Spectrum ULA - How to Design a Microcomputer
      - [x] 9. The Video Display
      - [x] ~~10. The Internal Clocks~~
      - [x] ~~11. Video Synchronisation~~
      - [x] 12. Generating The Display
  - Gamedev/SWE
    - [ ] Game Programming Patterns
      - [ ] VI. Optimization Patterns
        - [x] 17. Data Locality
        - [x] 18. Dirty Flag
        - [ ] 19. Object Pool

## Sat 08/May/2021

- Admin
  - Planning/Other
    - [x] Brogramming pools/Archive restructuring
    - [x] Next cycle planning
- Studies
  - Comparch
    - [ ] The ZX Spectrum ULA - How to Design a Microcomputer
      - [x] 4. ~~Semi-Custom Devices~~
      - [x] 5. ~~The Ferranti ULA~~
      - [x] 6. ~~Sinclair and the ULA~~
      - [x] 7. The ZX Spectrum Overview
      - [x] 8. The Memory Map
      - [ ] 9. The Video Display

## Fri 07/May/2021

- Writings
  - Blog replies
    - [x] Review and reply to Cherry pick MR comment
- Admin
  - [ ] Notes: move old ones/write pending
    - [ ] Game Programming Patterns
      - [x] III. Sequencing Patterns
        - [x] 8. Double Buffer
          - [x] Fight understanding the intended purpose
      - [x] V. Decoupling Patterns
        - [x] 14. Component
        - [x] 15. Event Queue
        - [x] 16. Service Locator
- Studies
  - Comparch
    - [ ] The ZX Spectrum ULA - How to Design a Microcomputer
      - [x] 1. Introduction
      - [x] 2. Integrated Circuits
      - [x] 3. The Standard Microcomputer

## Thu 06/May/2021

- Admin
  - [ ] Notes: move old ones/write pending
    - [ ] Game Programming Patterns
      - [x] II. Design Patterns Revisited
        - [x] 5. Prototype
        - [x] 6. Singleton
        - [x] 7. State
- Studies
  - Gamedev/SWE
    - [ ] Game Programming Patterns
      - [ ] VI. Optimization Patterns
        - [ ] 17. Data Locality

## Wed 05/May/2021

- Projects/Open source
  - AWS Scripts
    - [x] Large refactoring
    - [x] Open two issues and send fix PR to upstream project
- Writings
  - Blog
    - [x] Review and reply apt lock post comment
- Admin
  - [ ] Notes: move old ones/write pending
    - [ ] Game Programming Patterns
      - [ ] II. Design Patterns Revisited
        - [x] 4. Observer
- Studies
  - Gamedev/SWE
    - [ ] Game Programming Patterns
      - [x] III. Sequencing Patterns
        - [x] 8. Double Buffer
  - Linux/tools
    - [x] jq: More basics (array handling)

## Tue 04/May/2021

- Projects/Open source
  - AWS Scripts
    - [ ] Large refactoring
- Admin
  - [ ] Notes: move old ones/write pending
    - [ ] Game Programming Patterns
      - [ ] II. Design Patterns Revisited
        - [x] 3. Flyweight
- Studies
  - Linux/tools
    - [x] jq: Basics
  - Gamedev/SWE
    - [ ] Game Programming Patterns
      - [x] III. Sequencing Patterns
        - [x] 9. Game Loop
        - [x] 10. Update Method

## Mon 03/May/2021

- Admin
  - [ ] Notes: move old ones/write pending
    - [ ] Game Programming Patterns
      - [ ] II. Design Patterns Revisited
        - [x] Command
- Gamedev/SWE
  - [ ] Game Programming Patterns
    - [ ] III. Sequencing Patterns
      - [ ] 2. Game Loop

## Sun 02/May/2021

- Projects/Open source
  - QEMU-pinning
    - [x] Release 6.0.0
  - Amethyst
    - [x] PR: Implement "spin-sleep" strategy for the frame rate limiter sleep
    - [x] Review and close issue in the spin-sleep project
- Admin
  - Blog
    - [x] Add script for adding a new book to the bookshelf
  - Tools
    - Schedule manager
      - [x] Add skip mode with `replan` removal
  - Planning/Other
    - [x] Select book to study in order to understand QEMU
  - [ ] Notes: move old ones/write pending
    - [ ] Game Programming Patterns
      - [ ] II. Design Patterns Revisited
        - [x] Command
- Studies
  - Rust/Gamedev
    - [x] Read/evaluate game engines and interesting projects with easy issues
      - [x] Piston
      - [x] ggez
      - [x] A/B street
      - [x] Veloren

## Sat 01/May/2021

- Projects/Open source
  - QEMU-pinning
    - [x] Release 6.0.0
- Admin
  - Planning/Other
    - [x] Select book to study in order to understand QEMU

## Fri 30/Apr/2021

- Projects/Open source
  - ZFS
    - [x] Fix endless loops in internal invoke() calls
    - [x] Fix differences between distros in add-apt-repository options
- Admin
  - Tools
    - Schedule manager
      - [x] Add work computation (1: without end time detection)

## Thu 29/Apr/2021

- Writings
  - Blog
    - [x] Update Ruby `tk` gem build procedure
      - [x] Review native gems compiling
- Admin
  - Blog
    - [x] Update bio
    - [x] Add ZFS installer release announcement
  - System
    - [x] Fight inspecting Windows registry changes

## Thu 29/Apr/2021

- Projects/Open source
  - ZFS
    - [x] Fix installation issue with Ubuntu standard (Closes #191)
      - [x] Refactorings: functions splitting
      - [x] Implement
    - [ ] Add KUbuntu support (Closes #192)
      - [x] Refactoring: group base/common packages installation to a prior step
      - [x] Cleanup: when running `add-apt-repository`, add `-n` is an update is executed later
      - [x] Implement two required changes

## Tue 27/Apr/2021

- Projects/Open source
  - ZFS
    - [x] Add script functions hotswapping
      - [x] Switch to new `invoke()`
      - [x] Restructure parameter
      - [x] Automatically invoke `print_step_info_header()`
      - [x] Implement hotswap
    - [x] Slice 2: allow the user splitting directories into separate datasets (Closes #137)
      - [x] Write
      - [x] Test
      - [x] Add support for user parameter
      - [x] Test again

## Mon 26/Apr/2021

- ZFS
  - Reference: git.io/JODLd
  - [x] Merge Debian-specific GRUB configuration with regular one
  - [x] Test Debian, and deprecated it
  - [x] Enable GCM encryption, and test if supported
  - [x] Save all the disks in the log (Closes #75)
  - [ ] Slice 2: allow the user splitting directories into separate datasets (Closes #137)

## Sun 25/Apr/2021

- Projects/Open source
  - ZFS
    - [x] First slice: remove boot pool import hack  (Closes #140)
      - [x] Test on all major O/Ss
        - [x] Ubuntu Desktop
        - [x] Ubunt Server
        - [x] Debian - investigate issue
        - [x] Brutal fight Debian build problem
    - [x] Sort disks in selection dialog (Closes #10)
    - [x] Memory check update: 3.5 GB + always shows it if ram is below the threshold
    - [x] Minor improvements
      - [x] Clean pools raid type option, using a Bash array
      - [x] Cosmetic: Use `\K` on `perl.*\$1`/`perl.*\$\{1`
    - [x] Print internal variables as ZFS_* exports
    - [x] Simplify pool passphrase storage handling

## Sat 24/Apr/2021

- Projects/Open source
  - vga-passthough
    - QEMU pass usb host port
      - [x] Fight research
- Studies
  - Bash
    - [x] Fight best option(s) for multichar array join
- Projects/Open source
  - ZFS
    - [x] Move build to GH Actions
    - [x] Review all issues, and update if necessary
    - [x] Make jonathonf PPA user-selectable
      - [x] Review OpenZFS 2.0 benefits
      - [x] Test on VM, then desktop
      - [x] Implement
      - [x] Test
    - [x] Cosmetic cleanups
    - [x] Fight passwordless ssh login
    - [x] Fight multiple VBox machines tree issues
    - [x] Check memory requirement
    - [ ] First slice: remove boot pool import hack  (Closes #140)
      - [x] Review of the whole procedure
      - [x] Attempt at updating the whole functionality
      - [x] Fight first slice
        - [x] Apply several refactorings
      - [x] First round of testing

## Fri 23/Apr/2021

- Admin
  - [ ] Notes: move old ones/write pending
    - [x] mysql.odt (11 pag.)
    - [x] ruby.odt (27 pag.)
    - [x] Reorganize zfs notes
  - Tools
    - [x] qemu.sh: Add fetch QEMU repo
- Writings
  - Blog
    - [x] Kernel building: Add solution for certificate error
- Admin
  - Planning/Other
    - [x] Review/plan next two weeks
    - [x] Search computer architecture books
  - Tools
    - [x] QEMU script: Fix QEMU killer

## Thu 22/Apr/2021

- Projects/Open source
  - QEMU-pinning
    - [x] Use `$SUDO_USER`, when present
  - SSHKit::Sudo::Next
    - [x] Release gem
- Admin
  - Planning/Other
    - [x] Copy temporary notes

## Mon 19/Apr/2021

- Projects/Open source
  - ZFS
    - [x] Review and release Mint v20.1 support (contribution)
  - Openscripts
    - [x] bedtime
      - [x] Improve dates computation
      - [x] Add timer unit names; fix Zenity improper call
      - [x] Add checks for times not before now
- Admin
  - Tools
    - [x] Virtual machines handling script
      - [x] Refactor
      - [x] Make pattern optional, and improve listing
      - [x] Make headless mode optional
  - [ ] Notes: move old ones/write pending
    - [x] linux.odt (7 pag.)
    - [x] mysql.odt (22 pag.)
- Studies
  - Bash
    - [x] Improve `getopt`  invalid options handling
  - Linux
    - [x] Systemd: Service/Timer overrides
    - [x] Transient timers stop
    - [x] date: arithmetic; fight substraction with date operand

## Sun 18/Apr/2021

- Admin
  - Tools
    - [x] Notes syncing: update tooling to sync only at the end of the day
  - [ ] Notes: move old ones/write pending
    - [ ] linux.odt (9 pag.)

- Studies
  - Bash
    - [x] Handle invalid options with `getopt`

## Sat 17/Apr/2021

- Projects/Open source
  - [x] QEMU research paper
    - [x] Last review whole research

## Thu 15/Apr/2021

- Projects/Open source
  - Openscripts
    - [x] bedtime: Add warning before shutdown

## Wed 14/Apr/2021

- Projects/Open source
  - vga-passthrough
    - [x] 3_BASIC_SETUP: Update QEMU `readonly` argument to the exact format
  - qemu-pinning
    - [x] Revert "Build script: Add option to disable GUI modules"
    - [x] Build script: Disable docs building
    - [x] Bundle the debug logic into build option `--debug`
    - [x] Build script: Enable AIO in QEMU build
    - [x] Check if QEMU's configure supports `prefix`
    - [x] Update README
    - [x] Sync branches (master/5.2-changes/6.0/riscv_cpus)
    - [x] Fix symlinks patch permissions
    - [x] Make symlinks patch configurable
    - [x] Make riscv cpus increase patch configurable
    - [x] Simplify branches structure
    - [x] Other updates to README.md

- Admin
  - Tools
    - Schedule manager
      - Add half/off days functionality
        - [x] Addition to scheduler
        - [x] Computation

- Studies
  - Bash
    - [x] Bloody fight with killing sudoed background processes

## Tue 13/Apr/2021

- Projects/Open source
  - QEMU research paper
    - [x] Double check, and write section about QEMU performance with large number of vCPUs

- Admin
  - System
    - [x] Add new SSD
  - Tools
    - Schedule manager
      - [x] Add move skipped events
## Sun 11/Apr/2021


- Projects/Open source
  - QEMU research paper
    - [x] In-depth analysis of high-vCPUs slowness
      - [x] Scripting improvements
        - [x] Execute perf record only once per threads group
        - [x] Implement fixed threads number, in addition to interval
          - [x] Preliminary refactoring: simplify the threads number customization
        - [x] Add perf record pattern matching
          - Crucial, otherwise, the measurements are very (more or less significantly) perturbed by the benchmark preparation
          - [x] Brutal fight bash background subshells
      - [x] Get usable call stacks
        - [x] Check if useful, and added by QEMU make: `-fno-omit-frame-pointer`
        - [x] Compile opts: https://github.com/OP-TEE/optee_os/issues/2158
        - [x] Fight QEMU compile options
      - [x] Find and run best suite
        - [x] `cpu-cycles`
        - [x] Fight pkill
      - [x] Analyze results, and review possible improvements
        - [x] http://kvmonz.blogspot.com/p/knowledge-disk-performance-hints-tips.html
          - https://vmsplice.net/~stefan/stefanha-kvm-forum-2014.pdf
        - [x] https://www.ovirt.org/develop/release-management/features/virt/iothreads-support.html
        - [x] https://github.com/qemu/qemu/blob/master/docs/devel/multiple-iothreads.txt
        - [x] http://blog.vmsplice.net/2013/03/new-in-qemu-14-high-performance-virtio.html
          - https://wiki.qemu.org/Features/VirtioIoeventfd
      - [x] Open issue with summary

## Sat 10/Apr/2021

- Projects/Open source
  - Bevy
    - [x] Fix issue about required C++ package
      - [x] Test on different O/Ss, and discuss
      - [x] Open PR

## Fri 09/Apr/2021

- Projects/Open source
  - QEMU-pinning
    - [x] Update patch for 6.0
  - QEMU research paper
    - [x] Preview test on QEMU 6.0
    - [x] Full 10-run batches
      - [x] isolcpu (missing)
      - [x] basic
    - [x] review results

## Thu 08/Apr/2021

- Projects/Open source
  - QEMU research paper
    - [x] Disable SSH host added warning
    - [x] Implement perf stats
      - [x] Test it (+minor fixes)
    - [x] First review perf record/results
      - [x] Find best suited benchmark (-> bodytrack 16,128)
      - [x] Analysis of perf record
        - [x] Find differences between parameters
        - [x] Basic search futex informations
      - [x] Adjust options
    - [x] Write isolcpus results on the paper
        - [x] Check missing results, and run
          - Double check and store
        - [x] Minor scripts reorganization
          - [x] Add a couple of new small scripts
        - [x] Aggregate data
          - [x] plot_diagram.sh: `--dir` functionality, and a couple of other improvements
          - [x] Add compare_each_bench.sh
        - [x] Analyze results
      - [x] Write piece
        - [x] Re-read paper
        - [x] Write summary and discussion
    - [ ] Full 10-run batches
      - [x] isolcpu
      - [ ] basic

## Wed 07/Apr/2021

- Projects/Open source
  - QEMU research paper
    - [ ] Write isolcpus results on the paper
      - [x] Rerun all the isolcpu tests
    - [x] Study perf more in depth
      - [x] https://easyperf.net/blog/2019/10/05/Performance-Analysis-Of-MT-apps
      - [x] https://easyperf.net/blog/2019/10/12/MT-Perf-Analysis-part2
      - [x] Check if can run stat + record at the same time
        - https://stackoverflow.com/questions/10321073/can-perf-stat-results-be-generated-from-a-perf-data-file
      - [x] Review other stats
        - [x] Review http://www.brendangregg.com/perf.html#Examples
        - [x] Add new ones
          - [x] Search difference is usage between context-switches and sched:sched_switch
        - [x] Other readings
    - [ ] Implement perf stats and record
      - [x] Significant refactoring in order to accommodate the feature
      - [x] Implement it

## Tue 06/Apr/2021

- Projects/Open source
  - PPA Packaging
    - [x] Fix data directory incorrect value
    - [x] Improve help/comments related to the data directory role

- Projects/Open source
  - QEMU-pinning
    - [x] Print thread names and PIDs on start
    - [x] Add option to skip configure step on build
  - QEMU research paper
    - [x] Check why dedup failed
      - [x] Add script for running the benchmark programs used in the paper
    - [x] Add --min/--max runs
    - [x] Run perf on paper tests
      - [x] Handle failures
        - [x] Fight ferret instability
        - [x] Fight freqmine failure
          - [x] Add image check on shutdown
    - [x] Remove benchmark output from `run_benchmark.log`
    - [x] Perf improvements
      - [x] Check out the slowdown
      - [x] Fight dump threads
        - [x] Find Linux way of printing the TID/PID
        - [x] Investigate QEMU source code
          - [x] Implement threads PIDs printing
        - [x] Implement vCPU PIDs logging
    - [x] run_paper_benchmarks.sh: Make QEMU boot script an argument
    - [ ] Write isolcpus results on the paper
      - [ ] Rerun all the isolcpu tests

## Mon 05/Apr/2021

- Projects/Open source
  - QEMU research paper
    - [x] Benchmarking: Update isolated processors logic
    - [x] Benchmarking: Benchmark: Execute all runs in a single SSH session
    - [x] Remove warmup functionality
    - [x] Benchmarking: Enable SMT by default
    - [x] Test performance: isolcpus vs not
    - [x] Investigate irregular benchmark times D\*CK F\*RK
    - [x] blackscholes
    - [x] water_nsquared
    - [x] Further benchmark: ocean_cp
    - [x] Log benchmark output
    - [x] Make each cycle stand out more
    - [x] Add perf support
    - [x] Read: https://easyperf.net/blog/2019/10/05/Performance-Analysis-Of-MT-apps

## Sun 04/Apr/2021

- Projects/Open source
  - QEMU research paper
    - [ ] Learn perf
      - [ ] Context switches/migrations
        - [ ] https://eli.thegreenplace.net/2018/measuring-context-switching-and-memory-overheads-for-linux-threads
    - [x] run_benchmark: Add warmup runs (and invert threads/run cycle)
    - [x] Plan tests (first slice)

## Sat 03/Apr/2021

- Projects/Open source
  - QEMU research paper
    - [ ] Study/prepare perf
      - [x] Review QEMU threads
        - [x] Clean display of a process tree
      - [x] [Perf case with unexpected results](https://linux.kernel.narkive.com/o9Y3HOOa/scheduler-perf-stat-question-about-cpu-migrations)
      - [x] [Perf basics](http://www.brendangregg.com/perf.html)
        - [x] Find out how to count per-thread/process (not aggregate)
          - [x] Isolate the non-vcpu process(es)

## Fri 02/Apr/2021

- Studies
  - Gamedev/SWE
    - [ ] Game Programming Patterns
      - [x] II. Design Patterns Revisited
        - [x] 6. State

## Thu 01/Apr/2021

- Studies
  - Gamedev/SWE
    - [ ] Game Programming Patterns
      - [ ] II. Design Patterns Revisited
        - [ ] 6. State

## Wed 31/Mar/2021

- Projects/Open source
  - Capistrano/Netssh
    - [x] [Open issue](git.io/JYzPU) and propose implementation for ineffective `export`
  - QEMU research paper
    - [x] Remove pigz support
    - [x] Restore environment on key
      - [x] Prepare components
      - [x] Run test benchmark(s)
      - [x] Add `libsdl2-image-2.0-0` QEMU runtime dependency
      - [x] benchmark_apis: Add conditional to SMT reset hook

- Studies
  - Gamedev/SWE
    - [ ] Game Programming Patterns
      - [ ] II. Design Patterns Revisited
        - [ ] 6. State

## Tue 30/Mar/2021

- Projects/Open source
  - `sshkit-sudo`
    - [ ] Epic refactoring and improvement

## Mon 29/Mar/2021

- Projects/Open source
  - QEMU research paper
    - [x] Plan current objectives/requirements, and reorganize notes
    - [x] Lock cores speed during benchmark execution
      - [x] Find speed: 3.94 GHz
      - [x] Fight lock the speed
        - [x] Try various tools
        - [x] Try kernel 5.11
      - [x] Update the benchmarking tools
    - [x] Restore environment on key
      - [x] Update system
      - [x] Preset (3950x) ethernet driver
      - [x] Must rebuild kernel
        - [x] Fight compilation failure due to cert
      - [x] Fight custom host kernel

## Sun 28/Mar/2021

- Projects/Open source
  - Amethyst
    - [x] Additional work on [frame limiter](https://github.com/amethyst/amethyst/issues/2083)
      - [x] Search cpu monitoring crates
        - [x] Test `sysstat`
      - [x] Implement other two frame limiting strategies
        - [x] Benchmark them
      - [x] Measure (wall) power consumption
  - Bevy
    - Discuss [scheduler delay](https://github.com/bevyengine/bevy/issues/1501)
      - [x] Review Bevy source code
  - QEMU research paper
    - [x] Build text file
    - [x] Read research several times
    - [x] First researches
    - [x] Make first conclusion on the paper and write summary

- Studies
  - Rust
    - [x] Review `slotmap` crate
    - [x] Review `anymap` crate
  - Rust/Gamedev
    - [x] Preliminary review other Rust game engines

## Sat 27/Mar/2021

- Projects/Open source
  - Amethyst
    - [ ] Amethyst book
      - [ ] Pong tutorial
        - [x] 4.3: Movement
    - [x] Work on [frame limiter](https://github.com/amethyst/amethyst/issues/2083)
      - [x] Write
      - [x] Test on pong_6
      - [x] Discuss Linux slowness on Discord
      - [x] Test on other game(s)
        - [x] Evoli
        - [x] Fight compile dwarf_seeks_fortune
          - [x] Discuss
      - [x] Publish findings

## Fri 26/Mar/2021

- Projects/Open source
  - Amethyst
    - [ ] Amethyst book
      - [x] 3.4. World
      - [ ] 3.5. System
      - [ ] Pong tutorial
        - [x] 4.1
        - [x] 4.2
  - Ruby
    - [x] Presumably redundant constant `HEAP_PAGE_BITMAP_PLANES` investigation
      - [x] Write post author

## Thu 25/Mar/2021

- Projects/Open source
  - Amethyst
    - [ ] Amethyst book
      - [x] 3.1. State
      - [x] 3.2. Entity and Component
      - [x] 3.3. Resource

## Wed 24/Mar/2021

- Admin
  - Tools
    - [x] Schedule manager: Fix bug with current day editing

## Tue 23/Mar/2021

- Projects/Open source
    - Amethyst
      - [ ] Amethyst book

- Admin
  - Tools
    - [x] Schedule manager: Listing: print header comments

## Mon 22/Mar/2021

- Writings
  - VGA Passthrough
    - [x] Audio passthrough: Add the necessary new_id step, and update references to stability

## Sun 21/Mar/2021

- Projects/Open source
  - terraform-archive
    - [x] Review the new proposal and discussion, follow up and close the discussion

## Sat 20/Mar/2021

- Admin
  - Tools
    - Schedule manager
      - [x] Regex + related logic refactorings
      - [x] Add fixed timestamp support

- Projects/Open source
  - Openscripts
    - [x] bedtime: Add poweroff-only mode

- Studies
  - Gamedev/SWE
    - [ ] Game Programming Patterns
      - [ ] II. Design Patterns Revisited
        - [x] 3. Flyweight
        - [x] 4. Observer
        - [x] 5. Prototype
        - [x] 6. Singleton

## Fri 19/Mar/2021

- Studies
  - Gamedev/SWE
    - [ ] Game Programming Patterns
      - [x] I. Introduction
        - [x] 1. Architecture, Performance, and Games
      - [ ] II. Design Patterns Revisited
        - [x] 1. Architecture, Performance, and Games
        - [x] 2. Command

## Thu 18/Mar/2021

- Studies
  - Gamedev/SWE
    - [ ] Game Programming Patterns
      - [ ] I. Introduction
        - [ ] 1. Architecture, Performance, and Games

## Wed 17/Mar/2021

- Studies
  - Gamedev/SWE
    - [ ] Game Programming Patterns
      - [x] V. Decoupling patterns
        - [x] Chapter 15
        - [x] Chapter 16
    - Catherine West talk/discussions
      - [x] [Catherine West talk](https://www.youtube.com/watch?v=aKLntZcp27M)
      - [x] (15 minutes) [Jonathan Blow analysis CW talk](https://www.youtube.com/watch?v=4t1K66dMhWk)
      - [x] [Jonathan Blow HN Discussion](https://news.ycombinator.com/item?id=17993594)

- Admin
  - [ ] Notes: move old ones/write pending
    - [x] text_processing.odt (2)
      - Rearrange Perl

## Tue 16/Mar/2021

- Projects/Open source
  - Nextcloud
    - [x] Upload reported bug (provide logs)

- Admin
  - Notes
    - [x] Flush small notes

- Writing
  - Blog
    - [x] Updates to the kernel build article

## Mon 15/Mar/2021

- Projects/Open source
  - Nextcloud
    - [x] Open issue with desktop client conflicts
  - Drunken tomatoes
    - [x] Set size as fixed

- Studies
  - Gamedev
    - [ ] Game Programming Patterns
      - [ ] V. Decoupling patterns
        - [x] Chapter 14
  - Rust/Gamedev
    - [x] Review Building JavaScript Games [license](https://github.com/Apress/building-javascript-games)
      - Not permissive
    - Amethyst
      - [ ] Work on [frame limiter](https://github.com/amethyst/amethyst/issues/2083)
        - [x] First review project
          - [x] Old-review Duration concepts
          - [x] First look frame limiters and `arc_ball_camera`
          - [x] Try to hack in simulated input
        - [x] Follow up discussion about `spin-sleep` performance
      - [ ] Studies
        - [x] Generate the book for master

- Admin
  - Tools
    - planning script
      - [x] Check if there is a duplication bug
      - [x] Minor constants refactoring
      - [x] Add convenient debug mode
  - Notes
    - [ ] Flush small notes

## Sun 14/Mar/2021

- Projects/Open source
  - Openscripts
    - [x] gitio: Strip protocol, and minor refactorings
  - Rust/Gamedev
    - [ ] Search beginner issues on Rust gamedev mailing list
    - luminance
      - [x] Open PR: update deprecated calls `DynamicImage#to_rgb()`
    - Amethyst
      - [ ] Discuss/work on: https://github.com/amethyst/amethyst/issues/2083
        - [x] Research
          - [x] Discussion documentation spin loop/yield now: https://github.com/rust-lang/rust/issues/55418
          - [x] https://doc.rust-lang.org/std/thread/fn.yield_now.html
          - [x] https://man7.org/linux/man-pages/man3/pthread_yield.3.html
          - [x] https://stackoverflow.com/questions/21760566/yielding-from-linux-kernel
        - [x] Review: https://github.com/alexheretic/spin-sleep/blob/master/src/lib.rs
          - [x] Open issue to discuss the technical implementation
        - [x] Review: https://github.com/All8Up/test_limiter
        - [x] Review Cryengine: git.io/Jqw0i

- Studies
  - Javascript/Gamedev
    - [x] Building JavaScript Games
      - [x] Chapter 26
      - [x] Chapter 27
      - [x] Chapter 28
      - [x] Chapter 29

## Sat 13/Mar/2021

- Studies
  - Javascript/Rust/Gamedev
    - [ ] Building JavaScript Games (theory)
      - [x] Chapter 25
  - Other
    - [x] Review odd Perl behavior with Windows line ending

- Admin
  - Tools
    - [x] Solve fixed width problem
      - [x] Upgrade local SQLite
  - Notes
    - [ ] Flush small notes

## Fri 12/Mar/2021

- Writings
  - Blog: VGA Passthrough
    - [x] Add new motherboard data
    - [x] Add audio passthrough section

- Admin
  - Tools: Travis builds notifier
    - [x] Simplify purging logic
    - [x] Refactor: Convert instance variable @notify_travis_builds to local
    - [x] Permanently store last notified builds
  - Tools: Schedule manager
    - [x] Minor cleanups
    - [x] Add skipped event functionality
    - [x] Add list mode

- Studies
  - Rust
    - [x] Review Rust gamedev resources and meetings
      - [x] Report rg3d website bug

## Thu 11/Mar/2021

- Projects/Open source
  - Rails
    - [x] Investigate, open issue w/fix for ActiveRecord issue

## Mon 08/Mar/2021

- Projects/Open source
  - Libreoffice
    - [x] Test and close presumed bug with locale
  - Openscripts/vga-passthrough
    - QEMU script
      - [x] Large refactoring of the QEMU script, in preparation for publishing
      - [x] Implement audio passthrough

## Sun 07/Mar/2021

- Admin
  - Tools
    - [x] Change file sharing provider

## Sat 06/Mar/2021

- Projects/Open source
  - Awesome-Rust
    - [x] Add/update Game engine entries

## Fri 05/Mar/2021

- Projects/Open source
  - Spreadbase
    - [x] Review GH Actions
    - [x] Merge PR
    - [x] README updates
    - [x] Check and drop Ruby 2.3 support
  - Openscripts
    - [x] `ship_gem`: Make compatible with Ruby 3.0
    - [x] `prettify`: Minor updates
      - [x] Add YAML support
      - [x] Fix: the backup is done before checking the extension is supported

- Studies
  - Javascript/Rust/Gamedev
    - [ ] Building JavaScript Games (theory)
      - [x] Chapter 24

## Thu 04/Mar/2021

- Studies
  - Javascript/Rust/Gamedev
    - [ ] Building JavaScript Games (theory)
      - [x] Chapter 23

- Projects/Open source
  - Spreadbase
    - [x] Review/merge PRs
    - [ ] Review GH Actions

## Wed 03/Mar/2021

- Projects/Open source
  - Spreadbase
    - [ ] Review all issues opened
      - [x] Verification of `rexml` gem issue on Ruby 3.0
      - [x] Review opened PR
  - ZFS
    - [x] Review issue about (likely) nameserver problem

## Tue 02/Mar/2021

- Projects/Open source
  - Geet
    - [x] `repo open --upstream`

## Mon 01/Mar/2021

- Studies
  - Javascript/Rust/Gamedev
    - [ ] Building JavaScript Games (theory)
      - [x] Chapter 22

- Projects/Open source
  - Geet
    - [x] Implement upstream repository addition (`repo add_upstream`)

## Sun 28/Feb/2021

- Studies
  - Javascript/Rust/Gamedev
    - [x] Snake Game With Rust, JavaScript, and WebAssembly (without rewrite)
      - [x] Chapter 8: Theory + practice
      - [x] Chapter 9: Theory + practice
      - [x] Chapter 10: Theory + practice
      - [x] Review all code, and copy to notes
      - [x] Merge notes from the PragProg book and this course
    - [ ] Building JavaScript Games (theory)
      - [x] Chapter 17
      - [x] Chapter 18
      - [x] Chapter 19
      - [x] Chapter 20
      - [x] Chapter 21

- Projects/Open source
  - [x] Review and evaluate https://github.com/coding-horror/basic-computer-games

- Projects/Open source
  - Geet
    - [x] Implement repository opening functionality (`repo open`)
      - Refactorings
        - [x] Simplify (and improve) regexes
        - [x] Rename REMOTE_ORIGIN_REGEX to REMOTE_URL_REGEX
        - [x] Use the default repository instead of specifying `origin`
        - [x] Make :name an optional (keyword) parameter

## Sat 27/Feb/2021

- Studies
  - Javascript/Rust/Gamedev
    - [ ] Building JavaScript Games (theory)
      - [x] Chapter 16
    - [ ] Snake Game With Rust, JavaScript, and WebAssembly (without rewrite)
      - [x] Chapter 7: Theory + practice


## Fri 26/Feb/2021

- Studies
  - Javascript/Rust/Gamedev
    - [ ] Building JavaScript Games (theory)
      - [x] Chapter 15
    - [ ] Snake Game With Rust, JavaScript, and WebAssembly (without rewrite)
      - [x] Chapter 5
        - [x] Practice
      - [x] Chapter 6: Theory + practice
        - [x] Think alternative design

- Writings
  - [x] Article: Rust lulz: Implementing (floating point) approximate equality via traits

- Admin
  - Blog
    - [x] Sync gem versions with current ones on Github Pages

## Thu 25/Feb/2021

- Studies
  - Javascript/Gamedev
    - [ ] Building JavaScript Games (theory)
      - [x] Chapter 14
    - [ ] Snake Game With Rust, JavaScript, and WebAssembly
      - [ ] Chapter 5
        - [x] Theory

## Wed 24/Feb/2021

- Studies
  - Javascript/Rust/Gamedev
    - [ ] Snake Game With Rust, JavaScript, and WebAssembly
      - [x] Chapter 4: Theory + practice
      - [ ] Chapter 5: Theory + practice

## Tue 23/Feb/2021

- Studies
  - Javascript/Gamedev
    - [ ] Building JavaScript Games (theory)
      - [x] Chapter 9
      - [x] Chapter 10
      - [x] Chapter 11
      - [x] Chapter 12
      - [x] Chapter 13
  - Rust/CS
    - Hands-On Data Structures and Algorithms in Rust
      - [ ] Section 4: Graphs
        - [x] 26. Review theory; practice
  - Javascript/Rust/Gamedev
    - [ ] Snake Game With Rust, JavaScript, and WebAssembly
      - [x] Chapter 1: Intro
      - [x] Chapter 2: Theory + practice
      - [x] Chapter 3: Theory + practice
        - Fight favicon/constructor/source map errors

## Mon 22/Feb/2021

- Projects/Open source
  - Libemuls
    - [x] Update README; move architectural sections to the wiki

- Studies
  - Javascript/Gamedev
    - [ ] Building JavaScript Games (theory)
      - [x] Chapter 7
      - [x] Chapter 8

## Sun 21/Feb/2021

- Projects/Open source
  - Ruby packer
    - [x] Investigate open issue related to macro

- Studies
  - Javascript/WASM
    - Building JavaScript Games
      - [ ] Chapter 7
        - [ ] Theory

## Sat 20/Feb/2021

- Admin
  - Tools
    - [x] Review if can handle personal_notes with less commits
  - Planning
    - [x] Next project(s) planning
  - Other
    - [x] Review Code the classics assets and licensing
      - License: BSD-2 (https://github.com/Wireframe-Magazine/Code-the-Classics/issues/3)
      - License consequences: https://gamedev.stackexchange.com/a/98962

- Studies
  - Gamedev
    - [ ] Bevy intro

## Thu 18/Feb/2021

- Studies
  - Rust/CS
    - Hands-On Data Structures and Algorithms in Rust
      - [ ] Section 6: Entity component systems
        - [ ] Theory
      - [ ] Section 7: Storage
        - [ ] Theory

## Wed 17/Feb/2021

- Projects/Open source
  - Ruby packer
    - [x] Solve all compilation issues
      - [x] Fix build, due to Rubocop error
      - [x] Fix Ruby 2.7 options warning during compilation
      - [x] Fix Rubygems outdated certificate
      - [x] Make the gdbm subproject build on GCC 10

- Tools
  - [x] Improve `personal_notes` push script
  - [x] Improve day hours computation script
  - [x] Fix month hours hours computation script
  - [x] Investigate Lightning event timestamps storage format

- Studies
  - Rust
    - Hands-On Data Structures and Algorithms in Rust
      - [ ] Section 4: Graphs
        - [ ] Theory
      - [ ] Section 5: Hash maps
        - [ ] Theory
      - [ ] Section 6: Entity component systems
        - [ ] Theory

- Admin
  - Notes
    - [x] Review and archive: `rails.odt`

## Tue 16/Feb/2021

- Projects/Open source
  - QEMU research paper
    - [x] Delete the Fedora completed image, if not completed
    - [x] Reorganize improvements, and create an issue
    - [x] Add README
    - [x] Switch from `parallel` to `xargs`
  - Ruby packer
    - [ ] Solve compilation issue (https://git.io/Jt1FJ)
      - [x] Review and send possible patch

- Studies
  - Rust
    - [x] Review StackOverflow question

- Admin/tools
  - System
    - [x] Change password policy
  - Planning
    - [ ] Next project(s) planning
      - [x] Rearrange/archive current pool

## Mon 15/Feb/2021

- Projects/Open source
  - QEMU research paper
    - [x] setup_system
      - [x] Full end-to-end test on a clean machine
        - [x] Revert to raw image strategy for BusyBear
        - [x] Handle QEMU/SSH insanity
    - [x] Handle Scaleway
      - [x] Review models
      - [x] Open account
      - [x] Request access to mega-machine

- Studies
  - Linux tools
    - [x] Review guest resizing and sparse filesystem concepts

- Admin/tools
  - Tools
    - [x] Solve terminal emulator SSH key problem
  - Scripting
    - [x] Automate VMs start/stop

## Sun 14/Feb/2021

- Projects/Open source
  - QEMU research paper
    - [x] Issue with writes/flushes on slow driver
      - [x] Use qcow2 image format for working images, in order to solve slow disk issues
        - [x] Fight `guest(un)mount` poor async behavior
        - [x] Fight `qemu-img create` bug with backing image+size
    - [x] setup_system
      - [x] Attempt to use Debian-provided QEMU for Fedora preparation
      - [x] Minor tweaks
      - [ ] Full end-to-end test on a clean machine
        - [x] Setup clean VM

- Blog admin
  - [x] Decrease books font size
  - [x] Add new book

- Studies
  - Rust
    - [x] Review a Rust question

## Sat 13/Feb/2021

- Projects/Open source
  - ZFS
    - [x] Review and close discussion about space reclamation
    - [x] Help new contributor to add support for PopOS
  - QEMU research paper
    - [x] Run swaptions, and merge in
    - [x] Solve new issue with libxml2 compilation
    - [x] Compile facesim, and test
    - [x] Run VM script: add remove temp file
    - [x] Review exact working of QEMU -smp in relation to the riscv64 target
      - [x] Double check, summarize, and reply
    - setup_system improvements/fixes
      - [x] On data copy, copy only the `**/bin` directories
      - [x] Statically compile pigz
      - [x] Failed attempt at using the Debian-provided RISC-V cross-compiler
    - [x] Open source tooling
    - [ ] Issue with writes/flushes on slow drive
      - [x] Fight and diagnose

- Studies
  - Linux tools
    - [x] `rsync`: partial syncing of a tree, with certain conditions
  - Rust
    - [x] Rust high performance
      - Summary reading

## Fri 12/Feb/2021

- Projects/Open source
  - [ ] Benchmark: support the PARSEC tests
    - [x] Implement all the benchmarks
      - [x] Review benchmarks without input data
      - [x] Spread per-threads number runs
    - [x] Test run all the benchmarks
      - [x] Cap threads for benchs taking too long
      - [x] Find out volrend segfault cause
      - [x] Investigate and solve swaptions issue (mistake in documentation)
    - [ ] Review exact working of QEMU -smp in relation to the riscv64 target
      - [x] Complete analysis

## Thu 11/Feb/2021

- Studies
  - Git
    - Review `--rebase-merges`, and experiment

- Projects/Open source
  - QEMU research paper
    - [x] Complete PARSEC automated build
      - [x] Check/test if the ASM in `facesim` is compiled
    - [x] Test run of all the suites (`simmedium` input)
      - [x] Review failing ones
      - [x] Fix `parsec.vips`: requires libs
        - [x] Statically compile `vips`
          - [x] Fight chasing down and (statically) compiling the dependencies
          - [x] Fight subtly corrupted `parsec-benchmark` project
          - [x] Statically compile with workaround and test

## Wed 10/Feb/2021

- Projects/Open source
  - QEMU research paper
    - [x] Other instance of the QEMU networking issue
      - [x] Workaround (use loop device to transfer data, rather than tar)
        - [x] Refactor: write routine to setup loop device
    - [ ] Complete PARSEC automated build
      - [x] End-to-end test
        - [x] Fixes
        - [x] Handle Busybox missing bash
          - [x] Try stripping to make parsecmgmt ash-compatible
          - [x] Review bash (cross-)compilation
          - [x] Add to setup_system
      - [ ] Test run of all the suites
        - [x] Other fight with QEMU networking bug

## Tue 09/Feb/2021

- Projects/Open source
  - QEMU research paper
    - [ ] Complete PARSEC automated build
      - [x] Fix also `dedup` package
      - [x] Add copy stage
    - [x] Merge VM run scripts into a single one
    - [x] Fix an incorrect ROI patch
    - [x] Complete PARSEC automated build
      - [ ] End-to-end test
    - [x] Cleanups
      - [x] Move the Fedora `.prepared.run.qcow2` out of the way
      - [x] Add exit cleanup hook, and cleanup loop device/mountpoint variables
    - [ ] Benchmark: support the PARSEC tests
      - [x] Generalize guest benchmark script
      - [x] Basic implementation (blackscholes)
    - [ ] Other instance of the QEMU networking issue
      - [x] Review
    - [ ] Review exact working of QEMU -smp in relation to the riscv64 target
      - [x] Epic fight with the source code

## Mon 08/Feb/2021

- Projects/Open source
  - QEMU research paper
    - [ ] Complete PARSEC automated build
      - [x] Find the complete list of tests
      - [x] Check if compiling is parallel
        - [x] Check on `parsec.facesim`
        - [x] Check indirectly built projects (no; previously hacked `cmake/bootstrap`)
        - [x] Apply (and publish) improvement
      - [x] Update the splash2.raytrace Makefile to make it compile (and test)
      - [x] Implement parallel build
        - [x] Check out packages dependencies tree
      - [x] Build all
          - Exclude failing one (and dependent)
          - [x] Observe occupation
          - [x] Fix volrend
        - [x] Review failures
          - [x] canneal
          - [x] ssl/dedup

## Sun 07/Feb/2021

- Projects/Open source
  - QEMU research paper
    - [x] Packages compilation
      - [x] Try `-fcommon` solution
      - [x] Open PR + issue to `parsec-benchmark` repo

## Sat 06/Feb/2021

- Projects/Open source
  - QEMU research paper
    - [ ] Local tests run
      - [x] Scripting: minor refactorings/extensions to run scripts
      - [x] Add fedora preparation
        - [x] Fight QEMU networking issue
        - [x] Disable Fedora blocking service
        - [x] Build stage (single-process)
    - [ ] Packages compilation
      - [x] Epic fight with the remaining uncompiled package
  - ZFS
    - [x] Review and reply support reques

## Fri 05/Feb/2021

- Admin/tools
  - [x] Repositories sync script updates
    - [x] Add brojournal update
    - [x] Fully automate personal_notes handling
    - [x] Minor refactoring

- Projects/Open source
  - QEMU research paper
    - [ ] Local tests run
      - [ ] Add fedora preparation
        - [x] Preparation stage
        - [ ] Build stage (single-process)

## Thu 04/Feb/2021

- Projects/Open source
  - QEMU research paper
    - [ ] Local tests run
      - [ ] Add fedora preparation
        - [ ] Preparation stage
        - [ ] Build stage (single-process)

## Wed 03/Feb/2021

- Projects/Open source
  - QEMU research paper
    - [x] Check the native compiling on Fedora
      - [x] Review image preparation
        - [x] Prepare
      - [x] Compile all compilable
        - [x] Minor fix for a few
    - [ ] Local tests run
      - [x] Review/test run
        - [x] Review the input types
        - [x] Review how to run tests
          - [x] Test run, and find how to parse output
            - [x] Test run simple
              - Use `Total time without initialization :\s+(\d+)` (in µs), or `^real\s+(\S+)s$` when the first is not found
            - [x] ROI: `ROI time measured: (\d+\.\d+) seconds.`
              - [x] Fight investigate ROI
                - [x] Review `darchr` fork ROI logic
                - [x] Review source code for original timing logic
          - [x] Check `ns` and `NTHREADS`
        - [x] Run a few
          - [x] Test segfaulting ones with 16 GB
        - [x] ROI/fork testing
          - [x] Download `darchr` and compile a few tests
            - [x] Recompile all
          - [x] Fight enabling ROI

## Tue 02/Feb/2021

- Blog articles
  - [ ] High-performance Gaming/Content creation PC build
    - [x] Fight finding exact PCI lanes information

- Projects/Open source
  - QEMU research paper
    - [x] Cross-compile Parsec benchmarks
      - [x] Compile a few, and fight attempt to compile the rest
    - [ ] Check the native compiling on Fedora

## Mon 01/Feb/2021

- Admin/tools
  - [x] Clonezilla: manual run (restore) via tools, without boot
    - [x] Fix issue with dumps to be restored by the Focal clonezilla version
  - [x] Git
    - Fight `format-patch` accidental misconfiguration

- Projects/Open source
  - QEMU research paper
    - Scripts
      - [x] Add SMT support
      - [x] Create `run_all.sh` script
      - [x] Rename plotter `--compare` to `--scale`
    - [x] Test on bare Ubuntu Server
      - [x] Minor fixes
      - [x] Review results
    - [ ] Dive into the QEMU RISC-V implementation
      - [x] Find out if there is a core/hw_thread concept
      - [x] Summarize and send

## Sun 31/Jan/2021

- Studies
  - Other
    - [x] Python for text processing, instead of Perl
    - [x] Patch/diff, instead of Perl
  - Gnuplot
    - [x] Multiplots
      - [x] Handling diagrams with different x ranges

- Projects/Open source
  - QEMU research paper
    - [x] Scripting/features
      - [x] Quick cleanups/refactorings
      - [x] Informal HT local tests
      - [x] Add SMT disabling
      - [x] Add support for local (host testing)
      - [x] Add support for normalized/comparison diagram
    - [x] Personal workstation testing (host+guest)
      - [x] Base testing
      - [x] Pinning testing
        - [x] Review taskset+isolcpu
          - [x] Read resources
            - [x] https://stackoverflow.com/questions/14512197/what-does-taskset-in-linux-exactly-do
            - [x] https://www.codeblueprint.co.uk/2019/10/08/isolcpus-is-deprecated-kinda.html
          - [x] Think how to setup isolated setups so that they fit the non-isolated ones
            - [x] Implement
        - [x] Test isolcpu+pinning
    - [x] First look at Parsec

## Sat 30/Jan/2021

- Studies
  - Linux
    - [x] Check sudo behavior, and ask

- Blog articles
  - [x] Minor updates/fixes to some articles

- QEMU research paper
  - Scripting: Full environment preparation
    - [x] Add projects compiling/copy
    - [x] Split generated CSV files
    - [x] Test/clean/fix `prepare_components.sh`
    - [x] Set the `performance` frequency governor
    - [x] Test/clean/fix `benchmark_pigz.sh`
    - [x] Split diagram creation into separate script (see stash)
  - [ ] Test on bare Ubuntu Server
    - [x] Preparation

## Fri 29/Jan/2021

- Admin/tools
  - O/S
    - [x] Clonezilla: Workaround USB port issue

- Projects/Open source
  - QEMU research paper
    - Scripts: Full environment preparation
      - [x] Assume a standard busybear running locally
      - [x] New directories layout
        - [x] QEMU: can move firmware to a separate dir (`-bios`)?
      - [x] Split projects download/build into a separate script
      - [x] Run QEMU from the benchmark script
        - [x] Fight find snippet for caching sudo permissions
      - [x] Busybear: create empty disk with plenty of space (or extend existing)
      - [x] Linux repo: git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git
      - [x] Add projects download and patch
      - [ ] Add projects compiling/copy
        - [x] All except pigz/RISC-V toolchain

## Thu 28/Jan/2021

- Studies
  - Kernel (development)
    - [x] Read paper [Choosing the right timer interrupt frequency on Linux](http://repository.gunadarma.ac.id/313/1/Choosing%20The%20Right%20Timer_UG.pdf)
    - [x] Review all make config options
    - [x] Detail diff with 100Hz, and recompile

- Projects/Open source
  - QEMU research paper
    - [x] Understand details of the interrupt frequency, and update the environment

- Admin/tools
  - Scripting
    - [x] Brogramming: Automated handling of personal notes HEAD commit
  - O/S
    - [x] Prepare a clean ubuntu-server local to an USB key
      - [x] Fight Clonezilla breakages all over

## Wed 27/Jan/2021

- Projects/Open source
  - Busybear/QEMU
    - [x] Open issue with details about improved/working network configuration

- Admin/tools
  - [x] Build new workstation

## Tue 26/Jan/2021

- Studies
  - Busybear/QEMU
    - [x] Finally produce a working network configuration

- QEMU research paper
  - [x] Review providers
    - [x] Verify AWS models and availability
    - [x] Search all bare metal providers
  - [x] Review technical infos for custom kernels/params
  - [x] Prepare/test boot on Ubuntu server
  - [x] Reproduce setup
    - [x] Fix incompatibility with AMD (turned out: OpenSBI bug)
    - [x] Fix issue with emulated smp > 8 not allowed (turned out: kernel used was patched)
    - [ ] Prepare separate environment to use as host
      - [x] Install

## Mon 25/Jan/2021

- Studies
  - Rust
    - [x] Review 3 StackOverflow questions
      - [x] Answer 1
  - GNU Screen
    - [x] First review/test of commands sending
    - [ ] Think strategy for sending commands and waiting
      - [x] Check `tail -f` (source) for inotify

- Admin/tools
  - [x] Planning script
    - [x] Add support for next occurrence period (replan X in Y)
    - [x] Add --compare option
    - [x] Add support for "fixed" time (replan f X)

- Projects/Open source
  - [x] QEMU-pinning
    - [x] Revamp README examples/structure
    - [x] Review and integrate pinning patch
  - QEMU research paper
    - [x] Write test scripts: base version (SSH)
      - [x] End-to-end performance script test
      - [x] Add status messages

## Sun 24/Jan/2021

- Projects/Open source
  - QEMU research paper
    - [ ] AWS Metal
      - [x] Review instances
    - [ ] Write test scripts
      - [x] Bug: `results.data` has int time values

- Studies
  - Gnuplot
    - [x] Simplify project logic, by using stats
    - [x] Improve X tics
    - [x] Minor cleanups
    - [x] Investigate possible bug

## Sat 23/Jan/2021

- Admin/tools
  - [x] Fight issue with BIOS upgrade

- Projects/Open source
  - QEMU research paper
    - [ ] Plan AWS performance testing

## Fri 22/Jan/2021

- Projects/Open source
  - QEMU research paper
    - [ ] Execute improved tests
      - [ ] Write test scripts
        - [x] Fixes, base test, minor improvements
        - [x] Review standard deviation
        - [x] Improve directory layout

- Studies
  - Gnuplot
    - [ ] Fight gnuplot tics

## Thu 21/Jan/2021

- Projects/Open source
  - Investigate Busybear issues
    - [x] Test again
      - [x] check start-qemu.sh
      - [x] dump config
      - [x] fight bridge interface
      - [x] fight SSH connection
    - [x] Open other PR
  - QEMU research paper
    - [ ] Execute improved tests
      - [ ] Write test scripts
        - [x] Integrate password in cmdline
        - [x] Minor refactoring
        - [ ] Test/fixes

- Studies
  - Rust
    - [x] Join (part) Rust meetup

## Wed 20/Jan/2021

- Studies
  - Other
    - Gnuplot
      - [x] Review and reorganize existing notes
      - [x] Brutal fight find clean way to print multiple lines from research dataset

- Projects/Open source
  - QEMU research paper
    - [ ] Execute improved tests
      - [ ] Write test scripts
        - [x] Fight configure SSH on Busybear
          - [x] Open Issue+PR
        - [x] Use gnuplot
        - [ ] Test

## Tue 19/Jan/2021

- Projects/Open source
  - QEMU research paper
    - [x] Discuss my pigz/ray tracer preliminary test results
    - [ ] Execute improved tests
      - [x] Review cores pinning mapping
        - [x] Fight hwloc/source for skipping bridges
          - `hwloc`; `lstopo --of console --no-io --no-caches`
      - [ ] Write test scripts

- Blog articles
  - [x] Quickly building a custom Linux (Ubuntu) kernel, with modified configuration (kernel timer frequency)

## Mon 18/Jan/2021

- Blog articles
  - [x] Prepare QEMU user emulation
    - [x] Fight missing library
  - [x] Article RISC-V: Improve the tooling options section

- Projects/Open source
  - geet
    - [x] Fixes related to keyword arguments
  - QEMU-pinning
    - [x] Build script: add capstone dev library dependency #29
    - [x] Structure the repo so that build script/patch changes are recorded #26
    - [x] Build script: point directly to binaries #27
    - [x] Build script: add user space emulation support to build #28
  - QEMU research paper
    - [x] Improved tests preparation
      - [x] Fight install kernel with 100 Hz timer
      - [x] Test RISC-V QEMU with 16 cores

## Sun 17/Jan/2021

- Admin/tools
  - [x] Run interactive+background firefox on a headless machine
    - [x] Fight include missing library

- Studies
  - Rust
    - Articles/Smaller topics/Answers
      - [x] Review answers: 1
    - Ray tracer challenge
      - [ ] Further cleanups
        - [x] Evaluate parallel performance
          - 86% of theoretical max on 8 threads; SMT performance tanks
        - [x] Implement Slab/vector for bidirectional trees
          - Abandoned: very pervasive and time-consuming change
          - For the purpose of optimizing parallelization, the current version is already reasonably efficient
        - [x] Significant refactoring of the ObjParser

- Projects/Open source
  - QEMU research paper
    - [ ] Perfectly parallel test cases
      - [x] prepare ray tracer
        - [x] Prepare project
          - [x] Research RISC-V build
          - [x] Extract sdl interface to separate crate, in order to remove sdl dependency from the library
          - [x] headless_astronaut_rendering: Add commandline arguments
          - [x] headless_astronaut_rendering: Add conveniences for RISC-V build
      - [x] Further scripts tweaking and preliminary testing
        - [x] Prepare base scripts and smoke test
        - [x] headless_astronaut_rendering: Specify number of threads
        - [x] prepare image with ready files
      - [x] Run first pass guest tests
        - [x] run both tests
        - [x] verify `pig` host scalability on the same data
  - QEMU-pinning
    - [x] Add capstone requirement
    - [x] fix bug in vcpu computation and error message

## Sat 16/Jan/2021

- Projects/Open source
  - ZFS installer
    - [Space reclamation issue](https://github.com/saveriomiroddi/zfs-installer/issues/157)
      - [x] Try to reproduce it
      - [x] Think and implement workaround
      - [x] Write wiki page
    - [x] Investigate odd name resolution issue inside jail
      - Cause: VPN provider on the host

- Studies
  - Rust
    - Articles/Smaller topics/Answers
      - [x] Review a few StackOverflow answers

## Fri 15/Jan/2021

- Studies
  - Rust
    - Articles/Smaller topics/Answers
      - [x] Review a few StackOverflow answers
        - [x] Test and reply to one

## Thu 14/Jan/2021

- Studies
  - Javascript/WASM
    - Building JavaScript Games
      - [ ] Chapter 7

## Wed 13/Jan/2021

- Projects/Open source
  - QEMU research paper
    - [x] Test over-limit RISC-V cpus on all systems

- Studies
  - Javascript/WASM
    - Building JavaScript Games
      - [ ] Chapter 6

## Tue 12/Jan/2021

- Blog admin
  - [x] Fight latest post not updating on Google, and outdated index
    - [x] Rename incorrect post filenames

- Projects/Open source
  - Geet
    - [x] Clean Ruby 2.7 keyword argument warnings
      - [x] Make Ruby 2.6- compatible
    - [x] Add ruby 3.0 to build
  - QEMU research paper
    - [x] Review scripts
    - [ ] Reproduce paper system
      - [ ] Fight/research max limit of QEMU RISC-V cpus

- Studies
  - Rust
    - Articles/Smaller topics
      - [x] https://stackoverflow.com/questions/45116984/the-trait-cannot-be-made-into-an-object
      - [x] https://stackoverflow.com/questions/38215753/how-do-i-implement-copy-and-clone-for-a-type-that-contains-a-string-or-any-type
  - Javascript/WASM
    - Building JavaScript Games
      - [x] Chapter 5: notes

- Admin/tools
  - [ ] Notes: move old ones/write pending
    - [ ] text_processing.odt (2 pag.)

## Mon 11/Jan/2021

- Blog articles
  - [x] Quick RISC-V cross compilation and emulation

- Projects/Open source
  - QEMU research paper
    - pigz
      - [x] Review source code
      - [x] test scaling
    - [x] Reply, including gathered info

## Sun 10/Jan/2021

- Projects/Open source
  - RISC-V cross-compiling of pigz
    - [x] Test run fedora risc-v and compile(d) pigz

- Blog articles
  - [ ] Quick RISC-V cross compilation and emulation

## Sat 09/Jan/2021

- Projects/Open source
  - QEMU-pinning (for research paper)
    - [x] Build script improvements/extensions
      - [x] Add standard safety shopts
      - [x] Split into functions
      - [x] Add option to specify target arch
      - [x] Add --yes
      - [x] Add option to disable GUI modules
      - [x] General cosmetic cleanups
  - RISC-V cross-compiling of pigz
    - [x] Cross-compile
    - [ ] Fight setup riscv-tools

## Fri 08/Jan/2021

- Studies
  - Javascript/WASM
    - Building JavaScript Games
      - [x] Chapter 4
      - [ ] Chapter 5 (theory)
      - [ ] Chapter 6

## Thu 07/Jan/2021

- Admin/tools
  - [ ] Notes: move old ones/write pending
    - [ ] rails.odt

- Projects/Open source
  - [ ] Follow up research paper
    - [x] Find good candidates for perfect scaling test
    - [x] Investigate supposed mistake

## Wed 06/Jan/2021

- Blog articles
  - [x] Linux: Associating file types to applications

- Studies
  - Rust
    - Articles/Smaller topics
      - [x] https://stackoverflow.com/questions/25339603/how-to-test-for-equality-between-trait-objects
      - [ ] https://stackoverflow.com/questions/48952627/implementing-partialeq-on-a-boxed-trait
    - Larger topics
      - [ ] [Attribute macros](https://github.com/dtolnay/proc-macro-workshop)
        - [ ] Study
          - [ ] `derive(Builder)`
            - [ ] 01-parse
              - [x] Implement
  - Javascript/WASM
    - Building JavaScript Games
      - [ ] Chapter 4

- Projects/Open source
  - [x] Analyze research paper, and reply

- Admin/tools
  - [ ] Notes: move old ones/write pending
    - [ ] text_processing.odt

- Projects/Open source
  - ZFS installer
    - [x] Support-related documentation updates
      - [x] GitHub discussions: Add reference to README
      - [x] Update discussion pinned post

## Tue 05/Jan/2021

- Blog admin
  [x] Manage permissions

- Studies
  - Javascript/WASM
    - Building JavaScript Games
      - [x] Chapter 1
      - [x] Chapter 2
      - [x] Chapter 3
  - Articles/Smaller topics
    - Rust
      - [x] This week in rust #365
      - [x] This week in rust #366
        - [x] https://medium.com/centrality/why-work-in-blockchain-journey-from-c-to-rust-developer-eddbc9ccdc3d
      - [x] https://doc.rust-lang.org/nomicon/ownership.html
    - SWE
      - [x] [What is move semantics?](https://stackoverflow.com/questions/3106110/what-is-move-semantics)

- Blog articles
  - [ ] Linux: Associating file types to applications

## Mon 04/Jan/2021

- Blog admin
  - Consultancy
    - [x] Group by year + fix shadows and layout
      - [x] Integrate updated page
        - [x] Fight code not integrating
        - [x] Integrate revision
        - Close order
  - [x] Remove article-specific meta/data from bookshelf page

- Projects/Open source
  - openscripts
    - [x] Extend and open source `bedtime`
  - Drunken tomatoes
    - [x] Upgrade Rails
      - [x] 6.0
      - [x] 6.1
      - [x] Remove rubocop stuff
  - PPA Packaging
    - [x] Skip non-release Ruby versions (which cause issues with the release version upload)

- Studies
  - Javascript/WASM
    - [x] Check out content of the candidate JS books
    - [ ] Building JavaScript Games
      - [ ] Chapter 1

- Blog articles
  - [ ] Linux: Associating file types to applications

## Sun 03/Jan/2021

- Admin/tools
  - [x] Journal entry script: automate subjects index processing

- Studies
  - Rust
    - Ray tracer challenge
      - [ ] Further cleanups
        - [x] Apply new clippy version cleanup
        - [x] Try `weight_max()` Rayon optimization
          - Doesn't exist anymore
          - [x] Remove practically unnecessary overloading &Tuple - &Tuple
        - [x] Use atomic data type for shape IDs, instead of mutex+lazy_static

- Blog admin
  - Consultancy
    - [ ] Group by year + fix shadows and layout
      - [x] Discuss specs
      - [x] Reply other devs

- Projects/Open source
  - `fanotify`-based notification system for backups
    - Test implementation
      - [x] Fight basic fanotify implementation
        - [x] Further fix(es) to the manpage example
        - [x] Prepare and submit test case
        - [x] Fight find systematic location of different linux version man pages
          - [x] Correct bug, using 5.8 version
        - [x] Brutal debug fight
          - Traced back to issue with ZFS
  - ZFS installer
    - [x] Review and redirect support request; update, lock, and pin welcome thread
  - Drunken tomatoes
    - [ ] Upgrade Rails
      - [x] 5.2

## Sat 02/Jan/2021

- Admin/tools
  - [x] Find out how to schedule suspend via Systemd

- Studies
  - Rust
    - Tutoring
      - [x] Arrange meeting

- Blog admin
  - Blog consultancy
    - [x] Find/contact developers

- Projects/Open source
  - `fanotify`-based notification system for backups
    - [x] Review `fanotify` resources
      - [x] Review articles
        - [x] inotify: https://www.thegeekstuff.com/2010/04/inotify-c-program-example
          - [x] write and improve example
        - [ ] https://www.linux-magazine.com/Issues/2017/194/Core-Technologies
          - [x] inotify
        - Research: fanotify examples, and recursion support
          - [x] old: https://stackoverflow.com/questions/1835947/how-do-i-program-for-linuxs-new-fanotify-file-system-monitoring-feature
          - [x] uninteresting: https://stackoverflow.com/questions/59015209/fanotify-problem-with-new-flags-introduced-in-kernel-5-1
          - [x] obsolete: https://stackoverflow.com/questions/39290690/watching-a-directory-tree-whitout-inotify
          - [x] https://lwn.net/Articles/716973
          - [x] https://kernelnewbies.org/Linux_5.1#Improved_fanotify_for_better_file_system_monitorization
    - Test implementation
      - [x] https://www.linux-magazine.com/Issues/2017/194/Core-Technologies
      - [x] `man fanotify` has a full example
      - [x] Quick review [fanotify-rs](https://github.com/ZhangLei-cn/fanotify-rs)
      - [ ] Fight basic fanotify implementation
        - The manpage example doesn't work, and has at least one issue

## Fri 01/Jan/2021

- Studies
  - Rust
    - Tutoring
      - [x] Review codementor.io and send first message

- Blog admin
  - [ ] Split bookshelf by year
    - [x] Review Liquid and implement
    - [x] Fight HTML/CSS
