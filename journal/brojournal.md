## Mon 11/Jul/2022

- Admin
  - [x] Prepare and send talk submission to EuroRust
- Open Source: Punchy
  - [x] Review recent PRs
  - [x] Open issue about enemy energy bars
  - [x] Refactor fighters spawning
  - [x] Study attacks logic

## Sun 10/Jul/2022

- Open Source: Rust Analyzer
  - [x] Investigate and report [bug on type inference](https://github.com/rust-lang/rust-analyzer/issues/12738)
- Open Source: Punchy
  - [x] Direct ingame start functionality
  - [x] Limit players distance
  - [x] Implement left movement boundary
  - [x] Implement a correct multiplayer camera movement

## Sat 09/Jul/2022

- Projects
  - Rust Game Ports: Rusty Roguelike
    - [x] Fix state management
- Writings
  - Learn Bevy's ECS by ripping off someone else's project
    - [x] Update states management chapter with correct design
    - [x] Explain difference between `run_in_state()` and `run_if_resource_equals()`

## Fri 08/Jul/2022

- Studies: Rust
  - [x] Investigate `Vec#set_len()` assert not invoked/compiled
- Open Source: Punchy
  - [x] Review next milestone issues, PRs etc.
- Projects: Schedule Manager
  - [x] Fix: On fixed timestamp, the time was not copied to the new line
- Programming Streams
  - [David Pedersen](https://www.youtube.com/channel/UCDmSWx6SK0zCU2NqPJ0VmDQ)
    - [ ] Educational Rust live coding - Building a web app - Part 1
- SWE: Practice
  - Publish latest article in Reddit
  - Leetcode
    - [x] Medium    80%: 5 tests, 3 submissions, 22'00", "2330. Valid Palindrome IV"
      - Mistakes
        - [x] Misunderstood problem
        - [x] Typo
        - [x] Not thought edge case
        - [x] Incorrect thinking of cases
      - Disaster

## Thu 07/Jul/2022

- SWE: Practice
  - Programming Streams
    - [David Pedersen](https://www.youtube.com/channel/UCDmSWx6SK0zCU2NqPJ0VmDQ)
      - [ ] Educational Rust live coding - Building a web app - Part 1
- Open Source
  - [x] Open Gamedev meeting
- Studies: Gamedev/CS
  - [ ] Game Programming Algorithms and Techniques
    - [ ] 7. Physics

## Wed 06/Jul/2022

- Project: Schedule Manager
  - [x] Fixed timestamp: Improve error message
  - [x] Fixed timestamp: Allow no dot/spaces between time and description
- Project: Fish Fight: Punchy
  - [x] Discuss Bevy state handling
- Writings
  - [x] Article "A Rust game developer enters a bar"
- Project: Rust Game Ports
  - Rusty Roguelike
    - [x] Apply and test [state handling solution](https://discord.com/channels/691052431525675048/742884593551802431/956147241435955203)
  - Boing
    - [x] Fight find ggez procedure to change the logical size
- Open source: ggez
  - [x] Ask question about idiomatic procedure to change the logical size
  - [x] File issue about broken link

## Tue 05/Jul/2022

- Open Source: Fish Fight: Punchy
  - [x] Investigate and fix crash (#51)
  - [x] Discuss state machine problem
- Open Source: Rust Game Ports: Rusty Roguelike
  - [x] Brutal fight with [reported issue](https://github.com/64kramsystem/rust-game-ports/issues/97)
    - [x] Ask on channel if the diagnosis is correct
    - [x] Brief check of the problem in the source code

## Mon 04/Jul/2022

- Open Source: Openscripts
  - [x] Move SystemHelper from `fill_labels` to standalone file
  - [x] fill_dhl_packet_slip: Replace former system_helper require with new one (bug fix)
- Studies: Gamedev/Rust
  - [x] Review HECS
- SWE: Practice
  - Programming Streams
    - [David Pedersen](https://www.youtube.com/channel/UCDmSWx6SK0zCU2NqPJ0VmDQ)
      - [ ] Educational Rust live coding - Building a web app - Part 1
    - [Veloren Code Reading club](https://www.youtube.com/playlist?list=PLMI4vW_LISAkB0AzfIOvS-W504x3vrXvw)
      - [x] Download a few streams
- Studies: Rust
  - [x] Modularize, clean and make idiomation the Rust Game Ports support macro

## Sun 03/Jul/2022

- Project: Schedule Manager
  - [x] Implement cleaner logic to detect user's unexpected spaces (and similar cases)
  - [x] Add brackets when adding replan lines, and there aren't enough brackets
    - Fight odd edge case handling of `String#split`
      - Review Ruby source (`rb_str_split_m`)
- Project: Geet
  - [x] PR creation: Always ensure that the tree is clean
  - [x] PR Creation: Always run in automated mode
- Studies: Gamedev
  - [x] Ask question about debugging unintended changes on ECS systems
- Open Source: Fish Fight: Punchy
  - [x] Fix [Throwable Item animation bug](https://github.com/fishfight/punchy/issues/37)
  - [x] Discuss Punchy/Bevy design
  - [x] Investigate/fix (workaround) [Investigate/find workaround for attack bug](https://github.com/fishfight/punchy/issues/48)
  - [ ] Brutal fight with bundles/transforms
- Admin/Gamedev
  - [x] Write Erlend about projects and gamedev opinion
  - [x] Review Open Gamedev (Rust) projects
- SWE: Practice
  - Leetcode
    - [ ] Medium    80%: 1 tests, 1 submissions, 0'0", "1382. Balance a Binary Search Tree"
      - Failed - got to a solution in ~22", and currently not clear why it's rejected
      - Overly complicated solution

## Sat 02/Jul/2022

- Misc
  - [x] Check open source game listing websites
  - [x] Check FFmpeg compile options
  - [x] Announce mini-book on blog
- Admin
  - [x] Setup OBS on laptop
  - [x] Flush/plan cycle
- Open Source
  - Schedule manager
    - [x] Minor refactorings/improvements
    - [x] Add whitelist to work lines check
    - [x] Enforce timestamp when `f` flag (fixed timestamp) is provided
- Studies: Gamedev/Rust
  - [ ] Review Fyrox [platformer demo](https://fyrox-book.github.io/fyrox/tutorials/platformer/part1.html)

## Fri 01/Jul/2022

- Admin
  - [x] Fight twitch issue with audio/channels
- SWE: Practice
  - Leetcode
    - [x] Medium    80%: 4 tests, 1 submissions, 23'0", "2079. Watering Plants"
      - Mistakes
        - Used capacity on comparison, instead of current water
        - 1 debug run
        - Didn't clamp the capacity when adding water
      - Bad run; multiple mistakes; submitted by accident

## Thu 30/Jun/2022

- Studies: Bevy
  - [x] Review jam project
- SWE: Practice
  - Admin
    - [x] Fight setup Youtube for streaming
  - Leetcode
    - [x] Medium    80%: 2 tests, 1 submissions, 12'32", "1008. Construct Binary Search Tree from Preorder Traversal"
      - Mistakes
        - Forgot while cycle
      - Lost large part of the time, because misunderstood the problem

## Wed 29/Jun/2022

- Open Source
  - [x] Rust Game Ports/RustConf Arcade Cabinet: Add pad support
    - [x] Add in-game pad support to Boing/ggez
    - [x] Check out pad support in bracket-lib and Fyrox
- Writings: Learn Bevy's ECS by ripping off someone else's project
  - [x] 9.2: World
    - [x] Accessing World, and exclusive systems
    - [x] Clearing entities
  - [x] Write conclusion and exercise
- Admin
  - [x] Add Boing to list of ggez projects

## Tue 28/Jun/2022

- Open Source
  - Punchy
    - [x] Add GitHub actions CI, with format check
    - [x] Brief review of Clippy lints
    - [x] Build discussion
- Open Source
  - Rust Gamedev Newsletter
    - [x] Add Rust Game Ports
  - Bevy
    - [x] Send PR for broken documentation link
  - Rust Game Ports
    - [x] `Fixes to rusty_roguelike_step.sh`
- Writings: Learn Bevy's ECS by ripping off someone else's project
  - [x] Review remaining chapters
  - [x] 7.3: Events passing

## Mon 27/Jun/2022

- Open Source
  - [x] Review Spicy Lobster projects
  - [x] Report Ripcord issues
- Tools
  - [x] Setup Ripcord
- Open Source: Geet
  - [x] PR merge: Add upstream support
  - [x] PR create: On difference local/remote, offer diff/force push options
    - [x] A few extensions to GitClient

## Sun 26/Jun/2022

- Admin
  - [x] Fork/update jam repo
  - [x] Search Rust dev streams
  - [x] Prepare custom compiled FFmpeg
  - [x] Install, configure and test OBS
- Open Source
  - Geet
    - [x] Automatically add upstream repo on PR creation, if not existing
- Studies
  - Bevy Kira plugin
    - [x] experiment multichannel
    - [x] investigate channels lifetime (removal?)
  - Rust
    - [x] Make warnings errors/discussion

## Sat 25/Jun/2022

- Rust/Gamedev: Rusty Jam
  - [x] Implement Game over/game cycle
    - [x] Required some restructurings

## Fri 24/Jun/2022

- Rust/Gamedev: Rusty Jam
  - [x] Fix crash
  - [ ] Gameover scene+state management
    - [ ] Fight center text

## Thu 23/Jun/2022

- Rust/Gamedev: Rusty Jam
  - [x] Add scoreboard
  - [x] Enemy movement

## Wed 22/Jun/2022

- Rust/Gamedev: Rusty Jam
  - [x] Enemy fire
  - [x] Destruction of player's extra tiles

## Tue 21/Jun/2022

- Rust/Gamedev
  - Rusty Jam
    - [x] Fix Joshi's bullets hitbox (consider the distance travelled)
    - [x] Loot type/drop/attach
- Open Source
  - Bundler
    - [x] Investigate and open issue about improving gems installation (compilation) times
- Writings: Learn Bevy's ECS by ripping off someone else's project
  - [x] 07.01-02: Turn-based Games: Wandering/Turn-based
    - [x] designing_the_states.md: Simplify the design (4 stages instead of 5)
    - [x] Complete implementing_states_in_bevy.md
  - [x] Submit to Bevy Assets repo

## Mon 20/Jun/2022

- Open source
  - Geet
    - [x] Don't replace process when opening files
- Rust/Gamedev
  - Rusty Jam
    - [x] Loot locking
      - [x] Fight algorithm
    - [x] Loot attachment
    - [x] Debug Joshi's collision bug
    - [x] Fix Joshi bug were enemies health was not decreased
    - [x] When aliens die, they now spawn loot

## Sun 19/Jun/2022

- Rust/Gamedev
  - Rusty Jam
    - [x] Add/spawn enemies
    - [x] Add env var to skip main menu
    - [x] Replace terrain texture with better stubs
    - [x] Replace enemy texture with better stub
    - [x] Add player, and minor refactorings
    - [x] ingame: Disable stub update/teardown logging
    - [x] ingame: Add move player logic (move camera is a stub)
    - [x] ingame: Move camera
    - [x] Add loot stub
    - [x] Added assets
    - [x] Change enemy texture
    - [x] Add Pet stub
    - [x] Use (two) new assets
    - [x] Add pet (hook) move with mouse
    - [x] Add pet target chasing
    - [x] Split camera-related functions to a `camera_utils.rs` module
    - [x] Move systems to own module (dir), and global constants too (to file)
    - [x] Stub pet pick loot
    - [x] Implement loot transportation
- Tools
  - Alarm script
    - [x] Refactor
    - [x] Add volume parameter support
- Open source
  - Fyrox book
    - [x] Send PR with streaming option key name fix
  - Fyrox website
    - [x] Add Soccer to games list
  - github-readme-stats
    - [x] Report bug with commits count
  - Geet
    - [x] Recover PR summary (typically, on unexpected interruption)
  - Rust Game Ports
    - [x] Idiomatically set the widgets enable state
    - [x] Don't set the goal widget enable state on every frame
    - [x] Merge remaining shared imports into prelude
- Admin
  - [x] Check out Fish fight and other Open Gamedev School stuff
  - [x] Search Rust gamedev streams
  - [x] Add Rust Gamedev server
    - [x] Search Rust Game Ports old announcement
- SWE/Practice
  - [ ] Study MurderUserDungeon source
    - Fight compilation

## Sat 18/Jun/2022

- Studies: Rust/Gamedev
  - [x] Bevy
    - [x] Study 2d examples
    - [x] Study remaining main topics
    - [x] Study Breakout
    - [x] Study Heron
- Rust/Gamedev
  - Rusty Jam
    - (...)

## Fri 17/Jun/2022

- Admin
  - [x] Review MIT/Apache licenses/compatibility
- Studies: Rust
  - [x] Fight Cargo per-profile properties
- Open Source
  - Bevy Kira audio
    - [x] Open PR: Fix links in `examples/README.md`
- Studies: Rust/Gamedev
  - [x] Review proposed template
  - [ ] Review Bevy notes
  - [x] Fight Study Bevy audio

## Thu 16/Jun/2022

- Projects: Rust Game Ports
  - [x] Port Soccer to Fyrox
    - [x] Convert to proper scene graph
        - [x] Several refactorings
        - [x] Update MenuScreen
        - [x] Update GameHud
        - [x] Update GameOverScreen
  - Rusty Roguelike
    - [x] Review and remove unnecessary state `PlayerCollisions`
- Studies: Rust
  - [x] Review profile optimization options
- Studies: Rust/Gamedev
  - [x] Review [Rusty Jam Template](https://github.com/septum/rusty_jam_bevy_template)

## Wed 15/Jun/2022

- Open Source
  - Mysql2 Ruby gem
    - [x] README: Fix streaming MySQL documentation link
    - [x] README: Expand section on streaming, about early terminating a result set iteration
- Projects: Port Soccer to Fyrox
  - [ ] Convert to proper scene graph
    - [x] Several restructurings required
    - [x] Arrow
    - [x] Reset Game instead of reinstantiating it
    - [x] Add the pitch node only once
    - [x] Rename sprites/widget drawing methods to new semantics

## Tue 14/Jun/2022

- Projects
  - Port Soccer to Fyrox
    - [x] Make win score customizable
    - [x] Replace reinitializatin of actors with reset
      - [x] Ball
      - [x] Player
      - [x] Goal
    - [x] Rebase scene graph branch

## Mon 13/Jun/2022

- Open Source
  - PM-Spotlight
    - [x] Add robot emoji, plus source image conversion information
- Tools
  - [x] Fight VSC syncing issue
- Studies: Rust
  - [x] Flush notes
- Writings: Learn Bevy's ECS by ripping off someone else's project
  - [ ] 07.01-02: Turn-based Games: Wandering/Turn-based
    - [x] Add commands flushing/stages modeling to `07_01-02/a_look_at_the_states_model_in_the_source_project.md`
    - [x] Add `07_01-02/designing_the_states.md`
    - [ ] Add `07_01-02/implementing_the_states_in_bevy.md`
      - [x] Think carefully about port states design
- SWE: Practice
  - Leetcode
    - [x] Medium    81%: 1 tests, 1 submissions, 3'29", "1877. Minimize Maximum Pair Sum in Array"
      - Mistakes: none
- Projects
  - Port Soccer to Fyrox
    - [ ] Convert to proper scene graph
    - [x] Fix game over screen(s) transparency

## Sun 12/Jun/2022

- Admin
  - [x] Plan cycle
  - Rust
    - [x] VSC: disable rust analyzer separate target on all the projects
    - [x] Move linker cfg to global Cargo config dir
    - [x] Fight VSC: issue rust analyzer on macro (maybe: https://github.com/rust-lang/rust-analyzer/issues/6029)
  - System
    - [x] Automate kernel version handling on O/S install script(s)
- Projects: Rust Game Ports
  - [x] Remove compiler speed tweaks
- Gamedesv
  - [x] Test Rust Game Port games on higher refresh rates

## Sat 11/Jun/2022

- Open source
  - Fyrox book
    - [x] Add streaming sound explanation
- Projects
  - Port Soccer to Fyrox
    - [x] Implement Camera
    - [x] Implement HUD via GUI widgets
      - [x] Transparency issue; ask on chat
        - [x] Attempt solution
        - [x] Attempt other solution
      - [x] Add Media APIS to handle GUI widgets (currently, not nested)
- Projects
  - Rust Game Ports
    - [x] Take screenshots
    - [x] Update README: Extend descriptions, and improve general structure
    - [ ] Publish
      - [x] Prepare screenshots collage for newsletter
- Writings: Learn Bevy's ECS by ripping off someone else's project
  - [ ] 07.01-02: Turn-based Games: Wandering/Turn-based
    - [x] Add 07.01-02: Introduction
    - [x] Add 07_01-02/a_look_at_the_states_model_in_the_source_project.md
    - [ ] Add 07_01-02/a_basic_states_model_in_bevy.md
- Studies
  - Rust
    - [x] Find out if build dir can be configured per-target; ask

## Fri 10/Jun/2022

- Projects: Port Soccer to Fyrox
  - [x] Refactoring: Remove pivot node
  - [x] Think path to new design
  - [ ] Refactor into separate scenes
    - [x] Implement Scenes type
    - [x] Basic wiring of Scenes
      - [x] Brutal fight with scenes (unclear behavior of active flag)
    - [x] Draw to all the scenes
      - [x] Brutal fight with iteration of all scenes
    - [x] Make the player/goal/ball nodes permanent
      - [x] Brutal fight with scene transition bug
    - [ ] Fight with looped sound nodes
      - [x] Crashing
      - [ ] Not stopping on scene deactivation
  - [x] Close separate scenes plan
  - [ ] Implemente Camera
    - [x] Implement scrolling

## Thu 09/Jun/2022

- Projects: Port Soccer to Fyrox
  - [x] Fix bug due to wrong function comment in the source project
  - [x] Think new true scene-based design
  - [ ] Think path to new design
  - [x] Soccer-fyrox: Fix game over images z depth

## Wed 08/Jun/2022

- Open Source
  - [x] Publish Bevy/Fyrox small review
- Projects: Port Soccer to Fyrox
  - [x] Make path cross-platform
  - [x] Fight with cross-compilation issues
  - [x] Media: Whitelist resources by extension
  - [x] Brutal fight error on streaming
- Admin
  - VMPlayer/3D
    - [x] Fight issue with compilation/vmware tools
    - [x] Fight issue with speed
    - [x] Fight issue wih 3d support

## Tue 07/Jun/2022

- Projects: Port Soccer to Fyrox
  - [x] Make `targetable()` accept `Goal`
    - [x] Implement Target
    - [x] Experiment with Any
  - [x] Fix bugs
    - [x] Crash: "Attempt to borrow destroyed object"
    - [x] Fixed nasty bug that made computer ball owners go left
    - [x] Ball not being chased when without player
  - [x] Update README
  - [x] Send request to Rust GameDev newsletter

## Mon 06/Jun/2022

- Admin/System
  - [x] Try new kernel
- Projects: Port Soccer to Fyrox
  - [x] Fight access in Game#update
  - [x] Brutal fight access in Game#update
  - [x] Lots of other changes
  - [x] Completed source port (although there are still bugs to fix)

## Sun 05/Jun/2022

- Projects: Port Soccer to Fyrox
  - [x] Add util script `prefix_commits.sh`
  - [x] Minor refactoring/changes/fixes
  - [x] Port Game#update
    - [x] Convert images to GIF, in order to workaround Fyrox bug
  - [x] Implement draw ordering using Z depth
  - [x] Investigate and report other image bug, with transparency
  - [x] Brutal fight with fix drawing offset calculations
  - [x] Fix enemies not drawn
  - [ ] Port Player#update
- Studies/SWE
  - [x] Interval of integers exactly representable in IEEE754
- Open Source
  - Code the Classics Vol. 1
    - [x] Open PR with fix for two bugs

## Sat 04/Jun/2022

- Projects: Port Soccer to Fyrox
  - [x] Implement Game#draw
  - [x] Complete Game initializer (excluding debug code)
  - [x] Find and load all the resources, without static lists
    - [x] Symlink sounds/theme.ogg to music/theme.ogg, for simplicity, and simplify sounds referencing
  - [x] Minor improvements/fixes
  - [x] Follow ups, mainly related to Game#update
    - [x] Brutal fight with design polymorphic `Player#mark` association
  - [x] Convert `Rc`s to Fyrox's `Pool`
    - [x] Implement placeholder handles
  - [x] Investigate untyped arenas (Fyrox and others)
    - [x] Check also Macroquad
    - [x] Think a simple implementation based on Fyrox `Pool`
- Studies/Rust
  - [x] Investigate stategy to turn `Vec<RCC<T>>` into `Vec<&dyn U>`

## Fri 03/Jun/2022

- Admin
  - [x] Fight google analytics
- Writings: Learn Bevy's ECS by ripping off someone else's project
  - [x] (rust-game-ports project) `rusty_roguelike_step.sh`: Add `compare_port_prev` mode
  - [x] 06.02
    - Querying: Entities and Param sets
- Projects: Port Soccer to Fyrox
  - [x] Fight GUI images implementation
  - [x] Anchoring
  - [x] Player initializer
  - [x] Ball initializer
  - [x] `BareActor`, progress with `Game#reset`, derive `my_actor_based` for `Player`

## Thu 02/Jun/2022

- Open Source
  - RVM
    - [x] Close contributions and explain
- Admin/System
  - [x] Test GPU acceleration for video playback
- Projects: Port Soccer to Fyrox
  - [x] Redesign MyActor trait
  - [x] Implement MyActor helper macro

## Wed 01/Jun/2022

- SWE: Practice
  - Leetcode
    - [x] Medium    81%: 1 tests, 4 submissions, 10'11", "1817. Finding the Users Active Minutes"
      - Mistakes
        - 2*incorrect Array.new update after code change (!!)
        - keys instead of values in hash iteration
- Writings: Learn Bevy's ECS by ripping off someone else's project
  - [x] 06.01
    - [x] Complete last chapter
    - [x] Minor improvements, and restructuring of the snippets file referencing
- Projects: Port Soccer to Fyrox
  - [x] Review and implement a prelude design
- Studies: Rust
  - [x] Brutal fight with macros, to create a macro that add fields and implements a trait

## Tue 31/May/2022

- Admin/System
  - [x] Laptop O/S upgrade
    - [x] Tweaks
    - [x] Fight issue with Tilix select all selecting only the bottom part of the buffer (=last screen)
      - [x] Try multiple versions
      - [x] Try on VM
      - [x] Report issue on Ubuntu bug tracker
    - [x] Investigate desktop performance issue
    - [x] Backport new O/S script improvements to old O/S script
    - [x] Reinstall old O/S
- Projects
  - Openscripts
    - [x] Replace `inhibit_mate_screensaver.sh` with new (strategy) `set_display_sleep_time`
- Projects
  - Port Soccer to Fyrox
    - [ ] Follow up port
- Studies
  - Rust
    - [x] Review weak references

## Mon 30/May/2022

- Studies: Rust
  - [x] Review new RSpec-style test libraries
  - [x] Quick review Rust-Ruby bridges
- SWE: Practice
  - Leetcode
    - [x] Medium    81%: 2 tests, 1 submissions, 15'22", "1551. Minimum Operations to Make Array Equal"
      - Mistakes
        - [x] Mistakenly thought elements were powers, not sums, of 2
      - Proceeded in a generally messy way
- Admin/System
  - [x] Review AMDGPU-Pro difference
  - [x] Merge/update phone setup documents/scripts
  - [x] Test Linux games without amdgpu pro
    - [ ] Investigate excessive speed on high display refresh rates
  - [ ] Laptop O/S upgrade
    - [x] Installation and main upgrade
      - [x] Upgrade fixes
    - [ ] Tweaks

## Sun 29/May/2022

- Open Surce
  - Winit
    - [x] Follow up bug fix; report testing results
- Projects: Port Soccer to Fyrox
  - [x] Apply winit fix
  - [ ] Try implementing `Control` with lifetimes
- Writings: Learn Bevy's ECS by ripping off someone else's project
  - [x] rusty_roguelike_step.sh: Compare current mode: Add step selection
  - [x] 06.01: Query basics
    - [x] First section
    - [x] Query conditionals
    - [x] Commands/Resources
    - [x] (get_)single(_mut)

## Sat 28/May/2022

- Open source: Winit
  - [x] Report Window issues with resizing
- Open source: Openscripts
  - encode_to_m4a
    - [x] Check if destination exists
    - [x] Cosmetic improvements
- Projects: Port Soccer to Fyrox
  - [x] Try images bug workaround
  - [x] Complete draw() port
- Open source: Fyrox Cheat book
  - [ ] Look at reverb effect
- Studies: Gamedev/CS
  - [ ] Game Programming Algorithms and Techniques
    - [ ] 7. Physics

## Fri 27/May/2022

- Open Source: Fyrox
  - [x] Open Fyroxed issue about crash on dirs with symlinks
  - [x] Open Fyrox issue about red areas in some PNG images
- Open Source: Fyrox Cheat book
  - [x] Add sound chapter
- Projects: Port Soccer to Fyrox
  - [x] Investigate window resizing+locking
    - [x] Ask/discuss
    - [x] Find workaround
- Tools: IRC
  - [x] Try irccloud

## Thu 26/May/2022

- Openscripts
  - [x] git_rename_commits: Automatically cd to repo top level
- Projects: Port Soccer to Fyrox
  - [x] Media: Convert standard (2d library) coordinates to Fyrox ones
    - [x] Fight find texture dimensions
    - [x] Fight images (nodes) scaling
  - [ ] Investigate problem with texture red border

## Wed 25/May/2022

- Projects: Port Soccer to Fyrox
  - [x] Add music support
  - [x] Move sound nodes to a separate tree, and clean them on stop
  - [x] Redesign media handling
  - [x] Media: Generalize support for looping sounds
  - [x] Follow up with port
- Open Source: OpenScripts
  - [x] git_rename_commits: Update Perl flags (use 0777 mode)

## Tue 24/May/2022

- Open Source: RustDesk
  - [x] Remove remote.rs static mut booleans unsafe code, by using AtomicBool
    - [ ] Investigate other unsafe code
  - [x] Test pynput package, and open update README PR
  - [x] Strip release binary using Rust toolchain
- Projects: Port Soccer to Fyrox
  - [x] Async resources load
  - [x] Use Pivot as root node
  - [x] Refactor resources loading, to use `join_all()`
  - [x] Add sound support
    - [ ] Fight sound played with delay
- Studies: Rust
  - [x] Review build time profiling tools
    - [x] Investigate rustdesk build times
- Admin
  - [x] Preliminary verifications for running Ubuntu 22.04

## Mon 23/May/2022

- Projects: New Poor Man's Spotlight
  - [x] File search: allow spaces
  - [x] Update code to be compatible with new fltk-rs release
- Studies: Rust
  - [x] Review Rust double static/dynamic dispatching
- SWE: Practice
  - Leetcode
    - [x] Review dynamic programming [solution](https://leetcode.com/problems/all-paths-from-source-to-target/solution) to "797. All Paths From Source to Target"
    - [x] Medium    81%: 6 tests, 1 submissions, 21'51", "797. All Paths From Source to Target" (Dynamic programming sol.)
      - Mistakes:
        - `flat_map` was not the right approach
        - put the terminating condition in the wrong place (inside the child loop)
        - wrong caching: was caching the path from the current node, instead of the child path
        - (2x debug runs)
- Admin
  - [x] Cleanup Ruby playground repo, to prepare for the book
- Studies: Gamedev/CS
  - [ ] Game Programming Algorithms and Techniques
    - [ ] 4. 3D Graphics (2nd pass)
- Projects: Port Soccer to Fyrox
  - [x] Significant restructurings
    - [x] Introduced Game (GameGlobal) addition
    - [x] Restructured APIs to load images
      - [x] Some look at macros
  - [x] New menu logic
  - [x] New update() play/game over states

## Sun 22/May/2022

- Projects: Port Soccer to Fyrox
  - [x] Correct scene graph design
  - [x] Complete menu drawing
  - [x] Fight Fyrox problem with input handling
    - [x] discuss issue
- Studies: Rust
  - [x] Review how mdbook examples work
- Open Source: Improve Fyrox book documentation
  - [x] Add some code snippets to Data management subchapter
  - [x] Fix two code snippets in `platformer/part1.md`
- Open Source: RustDesk
  - [x] Cleanup global boolean unsafe code (first PR: `KEYBOARD_HOOKED`)
  - [x] Investigate and help deb package bug
- SWE: Practice
  - Leetcode
    - [x] Medium    81%: 1 tests, 1 submissions, 11'39", "797. All Paths From Source to Target"
      - Mistakes: none
      - Actually written code for cyclical graph
- Writings: Learn Bevy's ECS by ripping off someone else's project
  - [ ] 06.01
    - [x] Add Entities section (to existing "Components and Resources")
    - [x] Add introduction to system sets
- Admin
  - [x] Flush pool, and plan next cycle
- Studies: Rust/CS
  - [ ] Hands-On Concurrency with Rust
    - [ ] 6. Atomics - the Primitives of Synchronization
      - [x] Research linearizability

## Sat 21/May/2022

- Open Source
  - [x] Examine RustDesk source code
- Writings: Learn Bevy's ECS by ripping off someone else's project
  - [ ] 06.01
    - [x] Minor cosmetic improvements to existing chapters
    - [x] Considerably expand the systems section
    - [x] Write the Resources section
    - [x] Write Keyboard Input chapters
  - [x] Fight add Google Analytics
- SWE: Practice
  - Leetcode
    - [x] Medium    81%: - tests, 1 submissions, -, "339. Nested List Weight Sum"
      - Very bad and confusing Ruby problem design
    - [x] Medium    81%: 1 tests, 1 submissions, 6'0", "1329. Sort the Matrix Diagonally"
      - Mistakes (none)
- Projects: Port Soccer to Fyrox
  - [x] Base structure, with menu display
    - [x] Brutal fight with nodes graph

## Fri 20/May/2022

- Projects: Port Soccer to Fyrox
  - [x] Study cheat book
    - [x] 2.1. Getting started
    - 2.2. Scene
    - 2.10. Tutorials
    - (others)
  - [x] Study/experiment 2d example
    - [x] Ask questions
    - [x] Discussion/Brutal fight with window issue

## Thu 19/May/2022

- Studies: Rust
  - Rust for Rustaceans
    - [ ] Chapter 2: Types
    - [ ] Chapter 3: Designing Interfaces

## Wed 18/May/2022

- Writings: Learn Bevy's ECS by ripping off someone else's project
  - [x] Update project: Remove unnecessary Bevy App ownership workaround
  - [x] Add README to book repository
  - [ ] 06.01
    - [ ] World and systems
    - [ ] Components and resources

## Tue 17/May/2022

- Writings: Learn Bevy's ECS by ripping off someone else's project
  - [x] Test and prepare github pages
  - [x] Introduction chapter
  - [ ] 06.01
    - [x] Base structure

## Mon 16/May/2022

- Admin
  - [x] Cycle planning
- Tools
  - [ ] Phone setup
    - [x] Write script patching phone boot rom
    - [ ] Merge installation notes
  - Rust
    - [x] Try Split Rust Analyzer target dir
    - [x] Try mold linker
- Open source/Tools: Schedule Manager
  - [x] Reworker: Add support for higher-level closing entries
  - [x] Replanner: Replan to the same time bracket
- Studies: Rust
  - Rust for Rustaceans
    - [x] Ask design question about trait objects
  - [ ] Fight check if it's possible to use `$HOME` in config.toml
- SWE: Practice
  - Leetcode
    - [ ] Medium    81%: 3 tests, 0 submissions, 20'0", "1329. Sort the Matrix Diagonally"
      - Mistakes
        - Misunderstood definition of diago
        - Forgot return var
        - Forgot initialize var after copy/paste
- [ ] Projects: Port Soccer to Fyrox
  - [x] Collect/Review Fyrox material
  - [ ] Study cheat book
    - [ ] 2.1. Getting started
      - [x] Check out Rust ECS libraries (notable: HECS)

## Sun 15/May/2022

- Studies: Rust
  - Rust for Rustaceans
  - [ ] Chapter 2: Types
  - [ ] Research fat pointers/dispatch

## Sat 14/May/2022

- Studies: Rust
  - [ ] Rust for Rustaceans
    - [ ] Chapter 2: Types

## Fri 13/May/2022

- SWE: Practice
  - Leetcode
    - [?] Medium    82%: 4 tests, 1 submissions, 33'17", "2149. Rearrange Array Elements by Sign" (in-place)
      - Mistakes
        - syntax (forgot remove block var), after changing loop type
        - didn't think about a condition
        - same condition as above, was still wrong
      - Problem is not clear if it's expected for the rearrangement to happen in place; this approach is too slow
    - [x] Medium    82%: 3 tests, 1 submissions, 1'32", "2149. Rearrange Array Elements by Sign" (with support arrays)
      - Mistakes
        - forgot the `end` of one conditional
        - accidentally added return of original array
- Writings: Rusty Roguelike port mdbook (without refinements)
  - [x] Study mdBook
- Studies
  - Misc sysadmin stuff
    - [x] Github: download latest release
    - [x] Check if GH can host multiple GH pages for a single account
    - [x] jq
- Misc
  - Method chaining: "function(al) composition" (https://softwareengineering.stackexchange.com/q/359274/363129)

## Thu 12/May/2022

- Studies: Rust
  - [x] Read https://shopify.engineering/porting-yjit-ruby-compiler-to-rust
- Open source: Schedule manager
  - [x] Remove `{{date}}` on archived entry
  - [x] Find workaround for Timecop bug
    - [x] Publish example and workaround on Timecop Github issue
- SWE: Practice
  - Leetcode
    - [x] Medium    82%: 4 tests, 1 submissions, 6'39", "2268. Minimum Number of Keypresses"
      - Mistakes
        - Not taken frequency into account
        - Debug run
        - Excess each_char(), due to having added code without adjusting the code after
      - [x] Publish solution

## Wed 11/May/2022

- [x] Projects: Port Rusty Roguelike to Bevy (base port)
  - [x] Port `14_DeeperDungeons_more_levels`
  - [x] Port `15_Loot_01_loot_tables`
  - [x] Port `15_Loot_02_better_combat`

## Tue 10/May/2022

- Tools/MySQL
  - [x] Review cluster management/monitoring solutions
- SWE: Practice
  - Leetcode
    - [x] Medium    85%: 3 tests, 1 submissions, 10'55", "2265. Count Nodes Equal to Average of Subtree"
      - Mistakes
        - Forgot invoking Array#sum
        - Didn't fully read spec, assuming that a subtree doesn't include the head
    - Trivial solution, with `SC=O(n^2)`, without sliding average
- Open source: Openscripts
  - [x] Copy `ll_node.rb` to openscripts
- Tools
  - [x] Search/open script: open files at the given lines
    - [x] Publish solution on ag's Github issue
- Blog: User comments
  - [x] Review comment: https://saveriomiroddi.github.io/Linux-associating-file-types-to-applications/#comment-5852020103
    - [x] Fix the article
- Open source
  - VSC Markdown grammars syntax highlight
    - [x] Further help user request

## Mon 09/May/2022

- [ ] Projects: Port Rusty Roguelike to Bevy (base port)
  - [ ] Port `14_DeeperDungeons_more_levels`
    - [x] Failed attempt to follow up; requires redesign
- Studies: Computer Graphics
  - Computer Graphics from Scratch (Rasterization-oriented)
    - [x] Copy linear algebra notes
- Studies: Latex
  - [x] Express (some) linear algebra formulas in Rust
- SWE: Practice
  - Leetcode
    - [x] Medium    82%: ≈6 tests, 1 submissions, ≈20'0", "2130. Maximum Twin Sum of a Linked List" (SC=O(1) solution)
      - Mistakes
        - Disaster, primarily, missed next() invocations
        - Had to debug twice
- Admin
  - [x] Flush pool/plan next cycle
- Tools
  - [x] Stop/start services script: Add travis monitoring script

## Sun 08/May/2022

- Open Source
  - [x] User issue: Help with Markdown ASM integration
- Studies: Rust
  - [ ] Rust for Rustaceans
    - [x] Chapter 1: Foundations
    - [ ] Chapter 2: Types
- Studies: Computer Graphics
  - [x] Computer Graphics from Scratch (Rasterization-oriented)
    - [x] 3. Light
    - [x] 13. Shading
    - [x] 14. Textures
    - [x] 15. Extending the Rasterizer
    - [x] 16. Linear Algebra

## Sat 07/May/2022

- Studies: Computer Graphics
  - [ ] Computer Graphics from Scratch (Rasterization-oriented)
    - [ ] 3. Light
- SWE: Practice
  - Leetcode
    - [ ] Review solutions of last problem
      - [x] SC=O(1): https://is.gd/M56F5h

## Fri 06/May/2022

- Studies: Computer Graphics
  - [ ] Computer Graphics from Scratch (Rasterization-oriented)
    - [x] 11. Clipping
    - [x] 12. Hidden Surface Removal
    - [ ] 3. Light
- Studies: Rust
  - Rust for Rustaceans
    - [ ] Chapter 1: Foundations

## Thu 05/May/2022

- Studies: Computer Graphics
  - [ ] Computer Graphics from Scratch (Rasterization-oriented)
    - [x] 10. Describing and Rendering a Scene
    - [ ] 11. Clipping

## Wed 04/May/2022

- [ ] Projects: Port Rusty Roguelike to Bevy (base port)
  - [ ] Port `14_DeeperDungeons_more_levels`
- Studies: Computer Graphics
  - [ ] Computer Graphics from Scratch (Rasterization-oriented)
    - [ ] 10. Describing and Rendering a Scene

## Tue 03/May/2022

- [ ] Projects: Port Rusty Roguelike to Bevy (base port)
  - [x] Port `10_WhatCanISee_02_eyesight`
  - [x] Port `10_WhatCanISee_03_memory`
  - [x] Port `11_MoreInterestingDungeons_01_traits`
  - [x] Port `11_MoreInterestingDungeons_02_traits_rooms`
  - [x] Port `11_MoreInterestingDungeons_03_cellular`
  - [x] Port `11_MoreInterestingDungeons_04_output_harness`
  - [x] Port `11_MoreInterestingDungeons_05_drunkard`
  - [x] Port `11_MoreInterestingDungeons_06_prefab`
  - [x] Port `12_MapTheming`
  - [x] Port `13_InventoryAndPowerUps_01_potions_and_scrolls`
  - [x] Port `13_InventoryAndPowerUps_02_carrying_items`
- SWE: Practice
  - Leetcode
    - [x] Medium    82%: 4 tests, 1 submissions, 13'19", "2130. Maximum Twin Sum of a Linked List"
      - Mistakes
        - Forgot to pass parameter after adding a new one
        - Wrong API: `Array#unshift` instead of `#shift`
        - Wrong place for post-recursion logic
      - Thought in-place solution first, then went for very stupid solution; try better
    - [x] Medium    83%: 1 tests, 1 submissions, 5'35", "2161. Partition Array According to Given Pivot"
      - Mistakes
        - Typo
      - Thought of an in-place solution for a while, then just went for the st00pid simple

## Mon 02/May/2022

- Studies: Computer Graphics
  - [ ] Computer Graphics from Scratch (Rasterization-oriented)
    - [ ] 10. Describing and Rendering a Scene
- Open Source: RVM
  - [x] Submit issue/PR for fixing bug in installation from branch
  - [x] Submit issue/PR for documentation issue
  - [ ] Rubinius support PR: double check
    - [x] Fight patchsets now being read on installation
      - RVM is unstable when they're symlinks
    - [x] Fight downloaded Rubygems checksum issue (https://rubygems.org/rubygems/rubygems-3.0.9.tgz)
      - Likely bug in RVM, that stores the checksum for partially downloaded files
  - [x] Fight find most recent 4.x stable
    - None is stable (!)
- SWE: Practice
  - Leetcode
  - [ ] Review competition (https://leetcode.com/contest/biweekly-contest-77)
    - [x] Review const* time solution for https://leetcode.com/problems/count-unguarded-cells-in-the-grid
- [ ] Projects: Port Rusty Roguelike to Bevy (base port)
  - [x] Port `09_WinningAndLosing_03_winning`
  - [x] rusty_roguelike_step.sh: Convenient additions to (next mode)
  - [x] Port `10_WhatCanISee_01_fov`

## Sun 01/May/2022

- Tools
  - [x] Scripting
- [ ] Computer Graphics from Scratch (Rasterization-oriented)
  - [x] 9. Perspective Projection
  - [ ] 10. Describing and Rendering a Scene
- Open Source: RVM
  - [ ] Rubinius support PR: double check
    - [x] v4.7
    - [x] v5.0
      - Fails ~15% of the time, when rvm runs `gem wrappers regenerate` post-compilation
  - [ ] Review installation from branch bug (and documentation error)
    - [x] Find fix

## Sat 30/Apr/2022

- SWE: Practice
  - [x] Competitive programming contest
    - [x] Separately test latest rev of exercise 3
- [ ] Projects: Port Rusty Roguelike to Bevy (base port)
  - [x] rusty_roguelike_step.sh: Add reset functionality
  - [x] Port `08_HealthSimpleMelee_02_combat`
  - [x] Port `08_HealthSimpleMelee_03_healing`
  - [x] Port `09_WinningAndLosing_01_gauntlet`
  - [x] Port `09_WinningAndLosing_02_losing`
    - [x] Fight translate game over logic
    - [x] Review resources clearing (events etc.)

## Fri 29/Apr/2022

- Studies
  - Bevy
    - [x] Study Bevy's App/runner/Schedule
- [ ] Projects: Port Rusty Roguelike to Bevy (base port)
  - [x] util/rusty_roguelike_step.sh: Update for new .cargo/target locations
  - [x] Use stages to handle commands flushing
    - [x] Make sure that multiple stages are run for each update()
    - [x] Restructure 07.03 to match the source project's turn system
    - [x] Backport to 07.02
  - [x] Try removing the `ecs.runner` extraction
  - [x] Port `08_HealthSimpleMelee_01_health`
- SWE: Practice
  - Leetcode
    - [x] Medium    84%: 1 tests, 1 submissions, 6'15", "1490. Clone N-ary Tree"
    - [x] Medium    87%: 3 tests, 1 submissions, 9'55", "2181. Merge Nodes in Between Zeros"
      - Mistakes
        - Missed specification to remove zeros
        - Wrong termination condition for zero-deleting pass

## Thu 28/Apr/2022

- Studies: Rust/Gamedev
  - [x] Bevy cheatbook
    - [x] Chapter 9: Bevy Programming Framework
    - Skipped all the unread chapters
- SWE: Practice
  - Leetcode
    - [x] Medium    90%: 2 tests, 1 submissions, 4'48", "1570. Dot Product of Two Sparse Vectors" (Naive sol.)
      - Mistakes
        - Misunderstood the class structure
    - [x] Medium    90%: 1 tests, 1 submissions, ~2'06", "1570. Dot Product of Two Sparse Vectors" (Hash sol.)
    - [x] Medium    90%: 1 tests, 1 submissions, ~3'56", "1570. Dot Product of Two Sparse Vectors" (Array pairs sol)
- [ ] Projects: Port Rusty Roguelike to Bevy (base port)
  - [ ] Use stages to handle commands flushing
    - [x] Update and verify on 07.03

## Wed 27/Apr/2022

- Admin
  - [x] Review Crystal multithreading discussions
- Studies
  -  Rust
    - [x] Const generics
  - Bevy
    - [x] Review `iyes_loopless` resource-based conditionals
- [ ] Projects: Port Rusty Roguelike to Bevy (base port)
  - [x] Backport `iyes_loopless` logic to 07.02 (and remove 07.01)
  - [x] Implement state machine using const generics
  - [x] Implement rendering based on resource-based conditionals
- SWE: Practice
  - Leetcode
- Studies: Computer Graphics
  - [ ] Computer Graphics from Scratch (Rasterization-oriented)

## Tue 26/Apr/2022

- Open Source: Bevy Cheatbook
  - [x] PR for missing trait required in labels chapter
    - [x] Update PR
- Studies: Rust/Gamedev
  - Bevy
    - [x] Study `iyes_loopless` crate
- [ ] Projects: Port Rusty Roguelike to Bevy (base port)
  - [x] Fight fix flushing, by implementing `iyes_loopless` crate

## Mon 25/Apr/2022

- Studies: Rust/Gamedev
  - Bevy/Arguably Better Breakout
    - [x] Upgrade to Bevy v0.6 and test
- [ ] Projects: Port Rusty Roguelike to Bevy (base port)
  - [x] Clean warnings in source project
  - [x] Minor cleanups to port steps
  - [x] Port `07_TurnBasedGames_03_intent`
    - [x] Brutal fight with getting component for entity
  - [x] Brutal fight with system sets ordering/states filtering
  - [x] Fix multiple keypresses bug
  - [x] Fix/Improve cargo configuration structure
  - [x] Fight fix player_input system not filtering out enemies
  - [ ] Brutal fight with fix ghosting (to backport)
    - [x] Review Legion cycles per tick/systems ordering
- Open Source: Bevy Cheatbook
  - [x] Open PR for missing trait required in labels chapter
    - [x] Fight improper rendering of ```[`PartialEq`][std::PartialEq]```

## Sun 24/Apr/2022

- Studies: Perl
  - [x] Review scalar/list contexts
- Studies: Rust/Gamedev
  - Bevy
    - [x] System sets/ordering
    - [x] `ParamSet`s
    - [x] Review run criteria/states issue
      - https://bevy-cheatbook.github.io/features/fixed-timestep.html#caveats
      - https://bevy-cheatbook.github.io/programming/states.html#combining-with-other-run-criteria
      - https://github.com/IyesGames/iyes_loopless
      - https://github.com/bevyengine/rfcs/pull/45
    - [x] Study States
- [ ] Projects: Port Rusty Roguelike to Bevy (base port)
  - [x] Add "compare current" functionality to util script
  - [x] Use World to simplify setp 06/01 input handling
  - [x] Use World to mimick source projects "non-systems"
    - [x] Consider conversion to full-system-based design
  - [x] Port `06_EntitiesComponentsAndSystems_02_dungeonecs`
  - [x] Port `07_TurnBasedGames_01_wandering`
  - [x] Port `07_TurnBasedGames_02_turnbased`
    - [x] Fight modeling states/system sets
- SWE: Practice
  - Leetcode
    - [x] Medium    92%: 2 tests, 2 submissions, 8'13", "1874. Minimize Product Sum of Two Arrays"
      - Mistakes
        - The max heap solution is fast enough, but the library's implementation is very slow!
      - Solved with count sort
    - [x] Investigate `algorithms` library heap speed

## Sat 23/Apr/2022

- Studies: Computer Graphics
  - [ ] Computer Graphics from Scratch (Rasterization-oriented)
    - [x] 8. Shaded Triangles
    - [ ] 9. Perspective Projection
- Admin
  - [x] Review Vulkan books
- SWE: Practice
  - Leetcode
    - [ ] Medium    91%: - tests, - submissions, -, "1874. Minimize Product Sum of Two Arrays"
      - Confusing spec: the solutions rely anyway on a form of sorting
- Studies: Rust/Gamedev
  - [x] Review Rust GPU libraries
  - Bevy
    - [x] Review design of Legion scheduler and translation
    - [x] Investigate more idiomatic ways for used Bevy patterns
- [ ] Projects: Port Rusty Roguelike to Bevy (base port)
  - [x] Add convenience script to manage porting new steps
    - [x] Fight perl/awk
  - [x] Update README
  - [x] Search and add Rusty Roguelike license

## Fri 22/Apr/2022

- Studies: Rust/Gamedev
  - [ ] Bevy
    - [x] Review and update existing notes
      - [x] Review and update personal notes
      - [x] Review [migration guide to 0.6](https://bevyengine.org/learn/book/migration-guides/0.5-0.6)
      - [x] Review [migration guide to 0.7](https://bevyengine.org/learn/book/migration-guides/0.6-0.7)
    - [ ] Review specific topics/questions
      - [x] Review custom game loop
    - [x] Fight fix dynamic linking in VSC
      - [x] Publish article
- Open source
  - Blog tools
    - [x] `add_new_article.sh`: Fix title double quotes not fully escaped
    - [x] `update_article.sh`: Fix last post search
- [ ] Projects: Port Rusty Roguelike to Bevy (base port)
  - [x] Port first step: `06_EntitiesComponentsAndSystems_01_playerecs`
    - [x] Implement custom game loop
    - [x] Reorganize projects layout
    - [x] Port (fight several things)

## Thu 21/Apr/2022

- Open Source
  - QEMU-pinning
    - [x] Review and update fork to v7.0.0
- Writings
  - [x] Update blog post with email suggestion
  - [x] Blog util script update_article: allow passing a filename
- Admin
  - [x] Review leetcode premium and join
- Tools
  - [x] Journal leet addition script: Yet Another Entry Layout Reorder
- SWE: Practice
  - Leetcode
    - [x] Medium    94%: 1 tests, 1 submissions, 2'35", "1265. Print Immutable Linked List in Reverse"
- Studies: Rust/Gamedev
  - [ ] Bevy
    - [x] Fight setup workspace/CodeLLB dynamic library issue
  - [ ] Bevy cheatbook
    - [x] Update previous hello world
- [ ] Projects: Port Rusty Roguelike to Bevy (base port)
  - [ ] Start conversion
    - [ ] Brutal fight with system former `#config()` API

## Wed 20/Apr/2022

- Studies: Rust
  - [x] Review workspaces
- [ ] Projects: Port Rusty Roguelike to Bevy (base port)
  - [x] Review projects status
  - [x] Restructure the repository
  - [x] Update the README
  - [x] Preliminary test Bevy 0.7

## Tue 19/Apr/2022

- Projects
  - New Poor Man's Spotlight (tweaks)
    - [x] Add wildcard support to skip paths
      - [x] Refactor
    - [x] Review and add stripping
    - [x] Review and update fltk-rs version
    - [x] File searcher: Add wildcard support to pattern
    - [x] Add support for "alternate execute"
      - [x] File searcher: Add alternate execution (copy canonical path)
- Admin
  - [x] Flush pool
- SWE: Practice
  - [x] Investigate Ruby Set source code: how it compares to Array.uniq
- Studies: Rust
- [x] Investigate OsStr/Path types and filenames handling

## Mon 18/Apr/2022

- Projects
  - New Poor Man's Spotlight (tweaks)
    - [x] EmojiSearcher: Exit after execution
    - [x] Remove dependency on regex crate (use manual char manipulation)
    - [x] File searcher: Also ignore 1-char length patterns
    - [x] File searcher: Ignore nonexisting search paths
    - [x] Add window icon
      - [x] Fight issue with Desktop env theming
    - [x] Update README for old and new projects
- Admin
  - [x] Update Github pinned projects
- SWE: Practice
  - [x] Study solution to the hard problem
    - https://leetcode.com/problems/maximum-score-of-a-node-sequence
    - https://leetcode.com/contest/biweekly-contest-76
    - [x] Fight Python script version/imports
    - [x] Investigate Python <> Ruby performance

## Sun 17/Apr/2022

- Study
  - [x] Git
    - [x] Experiment with non-interactive interactive rebase
- Projects
  - [x] New Poor Man's Spotlight (first release)
    - [x] Brutal fight with odd interactive rebase behavior
    - [x] Implement configuration parsing
      - [x] Investigate TOML parsing
      - [x] Add configuration loading
    - [x] Write file search support (but not file search)
      - [x] Add async support for searches
      - [x] Implement search id
      - [x] Add support for stopping the current search
    - [x] Implement file search
      - [x] Test stop conditional performance (no difference on 32k files)
      - [x] Brutal fight with routine to generate short names
      - [x] Implement search
    - [x] Bug fixes
      - [x] Input doesn't respond (properly) anymore to tapping enter
      - [x] Tapping Enter multiple times on the input without selection caused panic
      - [x] Minor refactorings
    - [x] Implement File searcher execution

## Sat 16/Apr/2022

- Open source
  - fltk-rs
    - [x] Investigate and ask issue with HoldBrowser Enter callback
      - [x] Build and test a C++ FLTK program
    - [x] Brutal fight to figure out why multiple events are fired/triggered in some circumstances
- Studies: Rust/GUI
  - [x] Summarize and copy fltk(-rs) notes
- Studies: Rust
  - [x] Investigate common serialization formats and crates
  - [ ] Review O/S string data types and conversions
- SWE: Practice
  - [x] Competitive programming contest
- Projects
  - [ ] New Poor Man's Spotlight
    - [ ] Implement file search
      - [x] Base version of search, without patterns, unwired
        - [ ] Brutal fight file paths

## Fri 15/Apr/2022

- Projects
  - [ ] New Poor Man's Spotlight
    - [x] Generalize icon images type to SharedImage (due to released fltk-rs fix)
- Studies
  - Rust
    - [x] Investigate trait objects <> generics on fltk API
- Open Source
  - Rust
    - [x] Report compiler bug on `cargo doc`
    - [x] Review `fltk-rs` docs build, and open an issue to switch to stable Rust
  - Tools (Blog)
    - [x] Add add new article script
      - [x] Add tags
    - [x] Add update last article script
  - Rust/Gamedev
    - [x] Review Bevy current status (and new manual)
      - [ ] Report system test on [official issue](https://github.com/bevyengine/bevy/issues/3606)
- SWE: Practice
  - Cracking the Coding Interview
    - [x] IX.3 Stacks and Queues
      - [x] 03.06. 3 runs,  8'18", "Animal Shelter"
        - Mistakes
          - typo: `raise` missing parentheses
          - typo: asserts missing comma
          - copypasta: didn't change "cat" after copying (to "dog")
- Studies: Computer Graphics
  - [ ] Computer Graphics from Scratch (Rasterization-oriented)
    - [x] 7. Filled Triangles

## Thu 14/Apr/2022

- Projects
  - [ ] New Poor Man's Spotlight
    - [x] Workaround libxft bug (https://bugs.launchpad.net/ubuntu/+source/git/+bug/1852985)
      - [x] Discuss alternative fonts
      - [x] Attempt font-based workaround
      - [x] Workaround emoji issue via icons
        - [x] Produce icons #1
        - [x] Implement
          - [x] Diagnose and report bug with [PngImage->SharedImage](https://github.com/fltk-rs/fltk-rs/issues/1186)
        - [x] Produce icons #2
    - [x] Make use of newly released FLTK-rs Browser data support
    - [x] Massive refactoring

## Wed 13/Apr/2022

- Projects
  - [ ] New Poor Man's Spotlight
    - [x] Implement Emoji backend
      - [x] Implement backend registration
      - [x] Working prototype
      - [x] Large refactoring
      - [x] Fight attempt to workaround issue (https://bugs.launchpad.net/ubuntu/+source/git/+bug/1852985)
        - [x] Workaround libxft issue
      - [x] Full implementation
    - [x] Fight copypasta instability
    - [x] Investigate FLTK (-rs) Browser data handling
      - [x] Open feature request issue on fltk-rs project
    - [x] Try Druid
    - [x] Implement Browser data handling workaround
    - [x] Open libxcf issue on fltk-rs project
    - [x] EmojiSearcher: Store emoji data internally, to simulate a View pattern
    - [x] Reset input/list on execution
      - [x] Fight fltk(-rs) limitation on selection
    - [x] Fix multiple events being triggered on Enter
    - [ ] Implement file search
      - [x] Review directory traversal solutions

## Tue 12/Apr/2022

- Admin
  - [x] Review c64 ASM resources
- Open Source
  - Tools (Blog)
    - [x] add_new_book.sh: Escape description symbols
- SWE: Practice
  - [x] Code the Classics Vol.1 (Code Reading)
    - [x] 4. Fixed Shooter (Myriapod)
    - [x] 5. Football Game (Soccer)
- SWE: Practice
  - Cracking the Coding Interview
    - [x] 03.05. 3 runs,  7'19", "Sort Stack2"
      - Mistakes
        - Wrong condition (!nil? -> !empty?)
        - Test cases mistakenly expected a top-highest stack!
    - [x] 03.04. 2 runs,  5'27", "Queue via Stacks"
      - Mistakes
        - Forgot to initialize instance in test case!
- Projects
  - [ ] New Poor Man's Spotlight
    - [x] Update old repository and add status to README
    - [x] Start study FLTK
    - [x] Implement base protype
      - [x] Layout
      - [x] Actions
- Studies: Computer Graphics
  - [ ] Computer Graphics from Scratch (Rasterization-oriented)
    - [x] 6. Lines

## Mon 11/Apr/2022

- Tools
  - [x] Add TOC update to Libreoffice Macros
- Open source
  - [x] Extract and publish Hagar the Terrible source code
    - [x] Fight extraction of the files from the disks
    - [x] Prepare repository
    - [x] Extract assembly files; process PETSCII one
- Admin
  - [x] Summarize and buy computer graphics/optimization books
- SWE: Practice
  - Code the Classics Vol.1 (Code Reading)
    - [ ] 4. Fixed Shooter (Myriapod)

## Sun 10/Apr/2022

- Open source
  - [x] Wikipedia: Add new game engine entry
  - [x] Create/extend mirror of Driller Data Extractor
  - [x] Evaluate participating to https://github.com/open-telemetry/opentelemetry-ruby
- Projects
  - New Poor Man's Spotlight
    - [x] Choose GUI library
      - [x] Read https://dev.to/davidedelpapa/rust-gui-introduction-a-k-a-the-state-of-rust-gui-libraries-as-of-january-2021-40gl
      - [x] Test the libraries

## Sat 09/Apr/2022

- SWE: Practice
  - Code the Classics Vol.1 (Code Reading)
- Studies: Ruby VM/JITs
  - [ ] Rhizome
- SWE: Practice
  - Cracking the Coding Interview
    - [x] 03.03. 1 runs, ~7'00", "Stack of Plates (Follow up)"
      - Build on top of previous
    - [x] 03.03. 1 runs,  9'19", "Stack of Plates"

## Fri 08/Apr/2022

- Open source
  - Mydumper
    - [x] Investigate and fix [Clang compilation issue](https://github.com/mydumper/mydumper/issues/563)

## Thu 07/Apr/2022

- SWE: Practice
  - Code the Classics Vol.1 (Code Reading)
    - [ ] 4. Fixed Shooter (Myriapod)
      - [ ] Codebase

## Wed 06/Apr/2022

- Studies: Gamedev (/Rust)
  - [x] Understand exactly what game libraries abstract
    - [x] Investigate batching (/SDL)
  - [x] Review previously checked Ruby game libraries
- Studies: Rust
  - [x] Investigate switches for faster compile times

## Tue 05/Apr/2022

- SWE: Practice
  - Cracking the Coding Interview
    - [x] 03.02. "Stack Min"
      - implementation not required
      - did not find the solution
    - [x] 03.01. "Three in One"
      - implementation not required
- Open Source/Projects
  - [ ] Plan what to do with Code the Classics
    - [x] Review current state of Rust game engines
- SWE: Practice
  - Code the Classics Vol.1 (Code Reading)
    - [ ] 4. Fixed Shooter (Myriapod)
      - [x] Book chapter
      - [ ] Codebase

## Sun 03/Apr/2022

- Studies: Ruby VM/JITs
  - [ ] Rhizome
    - [x] [Interpreter](doc/interpreter.md)
      - [ ] Theory
    - [x] [Inline caching](doc/inline-caching.md)
      - [ ] Theory
- SWE: Practice
  - Code the Classics Vol.1 (Code Reading)
    - [ ] 1. Tennis (Boing)

## Sat 02/Apr/2022

- Studies: Ruby VM/JITs
  - [ ] Rhizome
    - [ ] [Bytecode](doc/bytecode.md)
      - [x] Theory
    - [ ] [Interpreter](doc/interpreter.md)
      - [ ] Theory
- Studies: Rust/Gamedev, SWE: Practice
  - [x] Hands-on Rust: Effective Learning through 2D Game Development
    - [x] 13. Inventory and Power-Ups
      - [x] Study source code
    - [x] 14. Deeper Dungeons
      - [x] Study source code
    - [x] 15. Combat Systems and Loot
      - [x] Study source code
- Open source
  - [x] Review project: https://github.com/joeyates/thunderbird
- Admin
  - [x] Organize books
  - [x] Choose+Prepare next
- Cracking the Coding Interview
  - [ ] IX.3 Stacks and Queues
    - [x] Theory

## Fri 01/Apr/2022

- Studies: Rust/Gamedev, SWE: Practice
  - [ ] Hands-on Rust: Effective Learning through 2D Game Development
    - [x] 11. More Interesting Dungeons
      - [x] Study source code
    - [x] 12. Map Themes
      - [x] Study source code
- Open source
  - [ ] Review project: https://github.com/joeyates/thunderbird

## Thu 31/Mar/2022

- Open Source: RVM
  - [x] Fix Rubinius support
    - [x] Find out how default versions work, and patch to default to v4.20 + v5.0
    - [x] Write and integrate gettid patch
      - [x] Verify how RVM patch system works
- Studies: Rust/Gamedev, SWE: Practice
  - [ ] Hands-on Rust: Effective Learning through 2D Game Development
    - [ ] 11. More Interesting Dungeons
      - [ ] Study source code

## Wed 30/Mar/2022

- SWE: Practice
  - Cracking the Coding Interview exercises
    - [x] 02.08. 4 runs,  0'38", "Loop detection"
      - Mistakes
        - returned false instead of nil
        - forgot to call current_node.next
        - wrong test case!! Ruby doesn't raise an error on self-referencing declarations (`a = Set.new([a]`)
      - Lazy: used a set
    - [x] 02.07. 3 runs, ~11'09", "Intersection"
      - Mistakes
        - 2, in next logic
      - Recursive solution; iterative is easier, but less interesting
- Studies: Rust/Gamedev, SWE: Practice
  - [x] Hands-on Rust: Effective Learning through 2D Game Development
    - [ ] 11. More Interesting Dungeons
      - [x] Study source code
    - [ ] 12. Map Themes
      - [x] Study chapter
    - [ ] 13. Inventory and Power-Ups
      - [x] Study chapter
    - [ ] 14. Deeper Dungeons
      - [x] Study chapter
    - [ ] 15. Combat Systems and Loot
      - [x] Study chapter
    - [x] 16. Final Steps and Finishing Touches
- Open Source: RVM
  - [ ] Fix Rubinius support
    - [x] Rubinius: Default modern versions to use Ruby 2.7
    - [x] Fix Clang requirements for modern (20.04) Ubuntu versions
    - [x] Rubinius: Handle the configscript name in versions from 4.6 onwards
    - [ ] Find out why v4.1 is the default version for v4
- Open Source/Tools
  - Cracking the Coding Interview exercises
    - [x] LLNode: Add inspection support for circular lists

## Tue 29/Mar/2022

- Open Source: RVM
  - [x] Write proposal for fixing the Rubinius compilation issues
- Open Source: Rubinius
  - [x] Open PR fixing the gettid() definition issue
    - [x] Study noexcept specifier
    - [x] Investigate issue in details
      - [x] Figure out new solution
- SWE: Practice
  - Cracking the Coding Interview exercises
    - [x] 02.07. 2 runs, ~15'01", "Intersection"
      - Mistakes
        - Typo: space after `LLNode.new`
      - Spent ~5 minutes writing UTs
      - Logic was easy, but spent considerable time simplifying it
      - Found that I misunderstood the problem (assumed the index of the intersection was the same)

## Mon 28/Mar/2022

- SWE: Practice
  - Cracking the Coding Interview exercises
    - [x] 02.06. 5 runs,  17'30", "Palindrome"
      - Mistakes
        - LLnode -> LLNode
        - Forgot to write `find_list_size`
        - UTs: forgot call `palindrome()`
        - Forgot to divide by 2 when computing list mid point
      - Implemented extra no-allocation (destructive) solution
- Studies: Rust/Gamedev, SWE: Practice
  - [ ] Hands-on Rust: Effective Learning through 2D Game Development
    - [x] 8. Health and Melee Combat
      - [x] Study source code
    - [x] 9. Victory and Defeat
      - [x] Study source code
    - [x] 10. Fields of View
      - [x] Study source code
    - [ ] 11. More Interesting Dungeons
      - [x] Study chapter
- Studies: Ruby VM/JITs
  - [ ] Rhizome
    - [ ] [Parser](doc/parser.md)
      - [ ] Theory
      - [x] Make Rubinius compile
        - [x] Update RVM llvm definitions
        - [x] Brutal fight with fix compilation problems and crashes on Clang 13
          - [x] Fight RVM patch format
        - [x] Fix compilation on Clang 6.0
        - [x] Brutal fight with missing `thread` gem (reported) error
      - [ ] Review RVM Rubinius/Ubuntu fixes

## Sun 27/Mar/2022

- Tools
  - [x] Fight find way to crop pdfs
- Open Source/Tools
  - Simplify Cracking Interview
    - [x] Update scripts to accept a number suffix, for alt. exercises
- Open Source/Studies
  - [x] Hands-on Rust study repository project
    - [x] Automated warning cleanup
    - [x] Mass-rename all the dirs following the book ordering
    - [x] Publish repository
- Studies: Rust/Gamedev, SWE: Practice
  - [ ] Hands-on Rust: Effective Learning through 2D Game Development
    - [ ] 9. Victory and Defeat
      - [x] Study chapter
    - [ ] 10. Fields of View
      - [x] Study chapter
- SWE: Practice
  - Cracking the Coding Interview exercises
    - [x] 02.05. 4 runs, 14'57", "Sum Lists (Follow up)"
      - Mistakes
        - One test case wrong - copy/paste from previous exercise
        - Typo (`a...` -> `b...`)
        - Forgot to invoke `#divmod`!!
    - [x] 02.05. 3 runs, 10'14", "Sum Lists"
      - Mistakes
        - One test case wrong!!
        - Didn't consider carry. Considered at the beginning, but quickly forgot in the rust
- Studies: Ruby VM/JITs
  - [ ] ruby-compilers.com analyses
    - [Rubinius](https://ruby-compilers.com/rubinius)

## Sat 26/Mar/2022

- SWE: Practice
  - Cracking the Coding Interview exercises
    - [x] 02.04. 4 runs,  17:05", "Partition"
      - Mistakes
        - Flaw in appending logic
        - Append to new list last node didn't consider first iteration, where itself was nil
        - Didn't encode truncation logic correctly for the considered edge case
    - [x] 02.03. 4 runs,  6'42", "Delete Middle Node"
      - Mistakes
        - Returned current node instead of head
        - 2*debug (used incorrect API)
      - The test case was wrong!! Logic was correct after 1st correction
    - [x] 02.02. 3 runs,  6'54", "Return Kth to last"
      - Mistakes
        - 2 typos (`LLnode` -> `LLNode`, `Struct.new(val)` -> `Struct.new(:val)`)
      - Recursive solution; no logical mistakes, but proceeded confusingly
- Tools
  - [x] Fight bug with ebooks sync on file sharing app
  - [x] Test other pdf annotating apps
- Studies: Ruby
  - [x] Review libraries for handling JSON5
- Studies: Rust/Gamedev, SWE: Practice
  - [ ] Hands-on Rust: Effective Learning through 2D Game Development
    - [x] 6. Compose Dungeon Denizens
      - [x] Study source code
    - [x] 7. Take Turns with the Monsters
  - [ ] 8. Health and Melee Combat
    - [x] Study chapter

## Fri 25/Mar/2022

- SWE: Practice
  - Cracking the Coding Interview exercises
    - [x] 01.07. 1 runs, 22'06", "Rotate Matrix"
      - Mistakes
        - `each_cons` missing parameter
        - the test case had an excess pair of outer parentheses
      - Took 4:30 just to write the test case!!
      - Intentionally the more difficult approach
      - Pretty good - the logic had zero mistakes
- Studies: Rust/Gamedev, SWE: Practice
  - [ ] Hands-on Rust: Effective Learning through 2D Game Development
    - [x] 4. Design a Dungeon Crawler
    - [x] 5. Build a Dungeon Crawler
    - [ ] 6. Compose Dungeon Denizens
      - [x] Study chapter
    - [ ] 7. Take Turns with the Monsters
      - [ ] Study chapter

## Thu 24/Mar/2022

- Open Source/Tools
  - Cracking the Coding Interview exercises
    - [x] `exercise`: When no params are passed, run the latest exercise
    - [x] Add linked list node data structure
      - [x] Add pretty printing
        - [x] Investigate `test-unit` output to make errors pretty
- Tools
  - Journal script
    - [x] Simplify Cracking Interview input format
- SWE: Practice
  - Cracking the Coding Interview exercises
    - [x] 02.01. ? runs,  ?'??", "Remove Dups"

## Wed 23/Mar/2022

- Tools
  - Journal script
    - [x] Integrate invocation of Cracking Interview project `exercise` script
- SWE: Practice
  - Cracking the Coding Interview exercises
    - [x] 01.09. 1 runs,  2'37", "String Rotation"
      - Mistakes
        - Rushed and written `s1 + s2` instead of `s1 + a1`
    - [x] 01.08. 2 runs,  5'40", "Zero Matrix"
      - Mistakes
        - Typo (`cols_i` <> `col_i`)
      - UTs were time consuming to write!!
    - [x] 01.06. 3 runs, 10'21", "String Compression"
      - Mistakes
        - Appended last_char instead of char
        - Mistake due to changed code
      - Solved at ~8', but mistakenly thought it was still broken

## Tue 22/Mar/2022

- Tools
  - Journal script
    - [x] Add support for Cracking exercise
    - [x] Minor refactorings
    - [x] Extract the section header (for matching) instead of hardcoding it
  - [x] VSC: Review new history navigation (`sworkbench.editor.navigationScope`)
- Open source
  - Cracking the Coding Interview
    - [x] `exercise`: Name the exercise method after the exercise name
      - Minor refactorings and improvements
- SWE: Practice
  - Cracking the Coding Interview exercises
    - [x] 1.05. 1 runs, 11'57", "One Away"
    - [x] 1.04. 2 runs,  3'59", "Palindrome Permutation"
      - Mistakes
        - Nested two `if` conditionals in the wrong way
    - [x] 1.03. 4 runs, 15'30", "URLify"
      - Mistakes
        - Forgot to set `is_leading_space=false`
        - Add debug info
        - Set `is_leading_space=false` in the wrong branch
- Studies: Rust/Gamedev, SWE: Practice
  - [ ] Hands-on Rust: Effective Learning through 2D Game Development
    - ~1. Rust and Your Development Environment~
    - ~2. First Steps with Rust~
    - [x] 3. Build Your First Game with Rust
- Studies: Ruby
  - [x] Idiomatic stdin lines reading
- Studies: Ruby VM/JITs
  - [x] First look at Artichoke codebase
  - [ ] ruby-compilers.com analyses
    - [ ] [Rubinius](https://ruby-compilers.com/rubinius)

## Mon 21/Mar/2022

- Admin
  - [ ] Plan new cycle and next 6 months
    - [x] Organize JIT-related references
- Studies: Rust
  - Rust Brain Teasers
    - [x] Copy notes
- SWE: Practice
  - [ ] Cracking the Coding Interview exercises
    - [x] Create project
      - [x] Create convenient script for exercises
    - [x] 1.01. 1 runs,  2'40", "Is Unique"
    - [x] 1.01. 1 runs,  2'39", "Is Unique" (alt.)
    - [x] 1.02. 1 runs,  2'05", "Check Permutation"

## Sun 20/Mar/2022

- SWE: Practice
  - Leetcode
    - [ ] Medium    59%,  131: 0 tests, 0 submissions, 52'0", "Palindrome Partitioning"
      - Can't solve via dynamic programming; too confusing
- Competitive Programmer's Core Skills
  - [ ] Week 5
    - [ ] Videos

## Sat 19/Mar/2022

- Admin
  - [x] Contact Code Curious
  - [x] Follow up blog improvement with another dev
- Studies: Rust
  - [x] Rust Brain Teasers

## Fri 18/Mar/2022

- Tools
  - Journal script
    - [x] Minor regex refactoring
    - [x] Add support for skipping H3+ sections
- SWE: Practice
  - Leetcode
    - [ ] Medium    59%,  131: - tests, - submissions, -", "Palindrome Partitioning"
      - Misunderstood the problem
- Admin
  - [ ] Plan new cycle and next 6 months

## Thu 17/Mar/2022

- SWE: Practice
  - Leetcode
    - [ ] Easy      51%,   70: - tests, - submissions, -", "Climbing Stairs"
      - Mistakes
        - Increased the result on each recursion, rather than on termination
      - Two approaches, both too slow (and the first too complex)
      - The brute force was modeled on missing steps; by counting the steps done, caching could be performed
    - [x] Easy      51%,  101: 5 tests, 1 submissions, 9'51", "Symmetric Tree"
      - Mistakes
        - Forgot `children.` in `children[0..(size/2)]`
        - Typo
        - Didn't realize that had to compare nodes values instead of nodes
        - Didn't use safe navigation when extracting node values
    - [x] Easy      53%,  202: 4 tests, 1 submissions, 9'07", "Happy Number"
      - Mistakes
        - Typo
        - Accidentally appended sum, where sum was done already via inject
        - Accidentally removed one termination condition, after moving a block of code
    - [x] Easy      54%,  121: 4 tests, 3 submissions, 14'23", "Best Time to Buy and Sell Stock"
      - Mistakes
        - 2*Forgot edge cases
        - Skipped entry on wrong side of the max prices array selection
      - Rushed into solution
- Open source
  - Mydumper
    - [ ] Investigate issue with RDS restores
- Study
  - Linux
    - [ ] Further investigation LUKS programmatic handling

## Wed 16/Mar/2022

- SWE: Practice
  - Leetcode
    - [x] Easy      55%,  350: 3 tests, 1 submissions, 6'30", "Intersection of Two Arrays II"
      - Mistakes
        - Misunderstood spec
        - Used `each_with_object()` instead of `inject` where change was `+=` (pay attention!)
    - [x] Easy      57%,  387: 1 tests, 1 submissions, 2'59", "First Unique Character in a String"
    - [x] Easy      59%,  268: 3 tests, 1 submissions, 3'07", "Missing Number"
      - Mistakes
        - Missed edge case
        - Typo
    - [x] Medium    59%,  378: 1 tests, 1 submissions, 24'46", "Kth Smallest Element in a Sorted Matrix"
      - Dumb solution
    - [x] Medium    59%,  328: 3 tests, 3 submissions, 24'0", "Odd Even Linked List"
      - Mistakes
        - subm.: missed part of logic (not considered one of the two termination cases )
        - typo
        - copy/paste from previous logic
        - subm.: didn't remove conditional after implementing new logic
    - [ ] Medium    62%,   62: - tests, - submissions, -, "Unique Paths"
      - Mistakes
        - Use #size on a check, where the value was an integer!!
      - Solved with backtracking, which is not time-acceptable
      - With solution known: 2'39"
    - [x] Medium    60%,  102: - tests, 1 submissions, 0'00", "Binary Tree Level Order Traversal"
      - Mistakenly returned the nodes instead, which was not obvious, and lead to massive loss of time

## Tue 15/Mar/2022

- SWE: Practice
  - Leetcode
    - [x] Medium    62%,  122: 5 tests, 1 submissions, 21'54", "Best Time to Buy and Sell Stock II"
      - Mistakes
        - Used `.max {}` instead of `max_by` (or `.map {}.max`)
        - Inadvert shadowing; typo
        - Terms of the max profix subtraction were inverted
      - Misunderstood the problem for Best Time I, and spent big chunk of the time
    - [x] Medium    62%,  289: 3 tests, 1 submissions, 18'23", "Game of Life"
      - Mistakes
        - Forgot to add diff params
        - Misunderstood requirement (modify original board)
- SWE: Career
  - Readings

## Mon 14/Mar/2022

- SWE: Career
  - Readings
- SWE: Practice
  - Leetcode
    - [x] Medium    63%,  215: 6 tests, 1 submissions, 12'31", "Kth Largest Element in an Array"
      - Mistakes
        - 3*Wrong variable, residual from rolling back code
        - 2*Debug
      - Very easy, but misunderstood the code, which resulted in lots of errors due to incomplete corrections
        - Don't rush!!
    - [x] Medium    64%,  238: 2 tests, 1 submissions, 9'14", "Product of Array Except Self"
      - Mistakes
        - `reverse()`d the wrong variable 🤦
    - [x] Medium    64%,  347: 2 tests, 2 submissions, 10'19", "Top K Frequent Elements"
      - Mistakes
        - Wrong algo!!
      - Spent some time thinking of a better than O(n log(n)) solution
    - [x] Medium    66%,   48: 8 tests, 1 submissions, 52'04", "Rotate Image"
      - Horror
      - Mistakes
        - 2*typo; forgot to change previous API invoked (each_cons)
        - charted wrong elements
        - flaw in algo impl
        - for inner matrixes, the upper bound was not correct(ed)
        - the number of elements on each side was wrong (ceil(half) -> size - 1)
      - Remember solution (from linear algebra): transpose->reverse hor. (CW), opposite (CCW)
    - [x] Medium    66%,  230: 1 tests, 1 submissions, 4'0", "Kth Smallest Element in a BST"
      - Cheated by using SortedSet
      - Fails with to unexplained error on the website, but succeeds locally, possibly because SortedSet requirement
  - [x] DCG traversal (ratios problem): 18'30"
- SWE: Design
  - [ ] Grokking the System Design Interview

## Sun 13/Mar/2022

- Open source: Openscripts
  - [x] Complete support for LUKS encrypted partitions
- Tools
  - Journal script
    - [x] Leet: Yet another update to template format
- SWE: Practice
  - Leetcode
    - [x] Medium    70%,   22: 2 tests, 1 submissions, 7'29", "Generate Parentheses"
      - Backtracking version
    - [x] Medium    70%,   22: 2 tests, 1 submissions, 19'53", "Generate Parentheses"
      - Mistakes
        - Forgot to add 0 balance condition
      - Found only the brute force solution (via bitmap)
    - [x] Medium    71%,   78: 2 tests, 1 submissions, 2'0", "Subsets" (solution known)
      - Mistakes
        - Forgot to add intermediate result to global one
    - [ ] Medium    71%,   78: 0 tests, 0 submissions, 0'0", "Subsets"
      - Couldn't find (efficient) solution
    - [x] Medium    72%,   46: 7 tests, 1 submissions, 16'44", "Permutations"
      - Mistakes
        - Forgot to add variable after changing recursive method signature
        - Typo
        - Wrong variable referenced
        - Wrong start index in current_i cycle
        - Debug
        - Forgot to clone() the arrays put in the result
      - Took significant time to recall the logic
    - [x] Easy      60%,  191: 3 tests, 1 submissions, 3'49", "Number of 1 Bits"
      - Mistakes
        - Forgot to update `n` on each cycle
        - One test (with extra code) lost because the specification is confusing
    - [x] Easy      60%,  171: 1 tests, 1 submissions, 12'09", "Excel Sheet Column Number"
    - [x] Easy      61%,  242: 1 tests, 1 submissions, 2'07", "Valid Anagram"
    - [x] Easy      63%,  169: 2 tests, 1 submissions, 3'28", "Majority Element"
      - Mistakes
        - Wrong variable! Original array -> Aggregates array
    - [x] Easy      64%,  118: 5 tests, 1 submissions, 8'18", "Pascal's Triangle"
      - Disaster; took a few minutes trying the recursive approach, which is terrible performance-wise
      - Mistakes
        - Forgot comma
        - 2x: Wrong indexing (1-based)
        - Forgot to append `map()` to `each_cons()`
- SWE: Career
  - Readings
- SWE: Design
  - [ ] Grokking the System Design Interview
    - [x] 1.07. Designing Twitter
- SWE: CS
  - [x] Study backtracking

## Sat 12/Mar/2022

- SWE: Career
  - Readings
- SWE: Design
  - [ ] Grokking the System Design Interview
    - [x] 2. Long-Polling vs WebSockets vs Server-Sent Events
    - [x] 1.06. Designing Facebook Messenger
- Tools
  - Player script
    - [x] Refactor
    - [x] Add sort option
  - Journal script
    - [x] Leet: Update template format
- Open source: Openscripts
  - ejectdisk
    - [ ] Preliminary look into unmounting LUKS partitions
- SWE: Practice
  - Leetcode
    - [x] Easy      66%,  108: 2 tests, 1 submissions, 19'32", "Convert Sorted Array to Binary Search Tree"
      - Mistakes
        - Inverted conditional (`<=` -> `>=`) on the condition of one recursion
    - [x] Easy      66%,  412: 0 tests, 0 submissions, 2'20", "Fizz Buzz"
    - [x] Easy      69%,  136: 3 tests, 1 submissions, 4', "Single Number"
      - Mistakes
        - Wrong Array.new() arguments (order)
        - Wrong normalization when extracting values out of array
    - [x] Easy      70%,  206: 1 tests, 1 submissions, 5', "Reverse Linked List" (iterative)
    - [x] Easy      70%,  206: 4 tests, 1 submissions, 14', "Reverse Linked List" (recursive, using refs)
      - Mistakes
        - Recursed on the node itself again!!
        - Wrong logic (reversed)
        - Didn't truncate new tail
    - [x] Easy      71%,  237: 1 tests, 1 submissions, 2', "Delete Node in a Linked List"
    - [x] Easy      72%,  104: 2 tests, 1 submissions, 3', "Maximum Depth of Binary Tree"
      - Mistakes
        - Recursed on the node itself, rather than the children

## Fri 11/Mar/2022

- SWE: Design
  - [x] Organize resources
  - [ ] Grokking the System Design Interview
    - [x] 1.03. Designing Pastebin
    - [x] Consistent Hashing
    - [x] 1.04. Designing Instagram
    - [x] 1.05. Designing Dropbox
    - [x] 2. Caching.pdf
- SWE: Practice
  - Interview Cake
    - [x] Front problem, solved in mind, ~8'
      - Used different algorithm - prefix sum

## Thu 10/Mar/2022

- Tools
  - [x] Resume script Bluetooth handling: add disable the BT device
- SWE: Design
  - [ ] Grokking the System Design Interview
    - [x] 1.02. Designing a URL Shortening service like TinyURL
    - [ ] 1.03. Designing Pastebin

## Wed 09/Mar/2022

- SWE: Design
  - [ ] Grokking the System Design Interview
    - [x] Fight store to study on tablet
    - [ ] 1.02. Designing a URL Shortening service like TinyURL
      - [ ] Consistent Hashing

## Tue 08/Mar/2022

- SWE: Practice
  - [ ] Cracking the Coding Interview
    - [ ] IX.2 Linked Lists
      - [x] 2.1 -> 2.5.1
        - Programmed 2.2; others solved in mind
        - Confused by 2.3

## Mon 07/Mar/2022

- SWE: Practice
  - Study
    - [x] Search large scale system resources
    - [ ] Cracking the Coding Interview (theory)
      - [x] VII. Technical Questions
      - [x] VIII. The Offer and Beyond
  - [ ] Cracking the Coding Interview (exercises)
    - [x] IX.1 Arrays and Strings
      - Solved in mind
        - 1.7: actually programmed (in-place solution)
    - [ ] IX.16 Moderate
      - [x] 16.6: Solved in mind
  - Leetcode
    - [x] Easy      45%,  716: 3 tests, 1 submissions, 8'
      - Found the efficient solution, but it requires linked list + binary tree APIs
      - Mistakes
        - Wrong API knowledge interpretation (Array#delete deletes all the elements, not 1)
        - Improper deletion (deletion from left, rather than from right)
    - [x] Easy      48%,    1: 1 tests, 1 submissions, 2' (more efficient solution)
    - [x] Easy      60%,   21: 4 tests, 1 submissions, 18'
      - Mistakes
        - 2*debug
        - lost lots of time on two misnamed variables
- Studies
  - Ruby
    - [x] Review all data structures in the stdlib
- Tools/Reverse engineering
  - [x] Test Ghidra with Willy the Worm

## Sun 06/Mar/2022

- SWE
  - Leetcode
    - [x] Medium 28%,  151: 1 tests, 1 submissions, 9'
    - [x] Medium 39%,  146: 1 tests, 1 submissions, 8'
      - This is trivial, by using Ruby's hash keys ordering
    - [x] Medium 40%,   54: 5 tests, 1 submissions, 18'
      - Mistakes
        - Wrong modulo array access
        - Typo
        - Wrong modulo array access again
        - Wrong bidimensional array access `[a,b]` instead of `[a][b]`
      - Solution is simpler than the proposed ones!
    - [x] Medium 43%,  545: 3 tests, 1 submissions, 38'
      - Mistakes
        - Wrong debug code!
        - Copy pasted, then modified source, then forgot to modify copy
        - Missed difficult implicit condition: outermost boundary nodes are never added to leaves, so they don't need to be popped
      - Ignored correct submission with excessive output length
    - [x] Medium 53%,   17: 1 tests, 1 submissions, 13'
    - [x] Medium 53%,  200: 2 tests, 1 submissions, 36'
      - Mistakes
        - Copy paste error when refactoring a condition!!
      - Thought an optimized approach was required, and wasted 15 minutes on it
    - [x] Medium 54%,  692: 3 tests, 1 submissions, 8'
      - Mistakes
        - Wrong variable (words <> words_count)
        - Mistaken result type of an API (Hash#sort_by -> Array<>Hash)
- Tools
  - Journal script
    - [x] New leet entry format (prefilled, instead of completed)
    - [x] Don't add anymore leet entry parent bullet points

## Sat 05/Mar/2022

- SWE: Practice
  - Try recursion approach via Ruby pseudo-references
  - Leetcode
    - [x] Medium 56%,  394: 3 tests, 3 submissions, 33'
      - Mistakes
        - omitted returning "i" in the last statement of the recursive fx
          - main function was not unpacking the result of the recursive fx, after updating the above
        - did not read again the full spec; a condition was hidden in the constraints
        - two consecutive conditions interfered with each other (should have been nested, or consider each other)
      - Distaster after the first two mistakes
    - [x] Medium 56%, 1405: 4 tests, 1 submissions, 19'
      - Mistakes
        - used hash symbol keys instead of chars
        - used a simple break to exit outer loop, but exited inner one
          - placed wrong conditional after hastily correcting the above
        - checked wrong fence value: > 2 instead of >= 2
      - Last mistake was very expensive; put debug code from the beginning!
    - [x] Medium 56%,  116: 3 tests, 1 submissions, 18'
      - Mistakes
        - Missed a statement (children clearing)
        - Didn't reverse a conditional (parents -> children) after fixing the above
        - Missed one test case in the specification!! (lost lots of time)
    - [x] Medium 56%, 1647: 3 tests, 2 submissions, 26'
      - Mistakes
        - Used Hash#keys instead of #values
        - Omitted a branch
        - Added (if) conditional but forgot to turn if into elsif
        - 1 submission wasted due to not fully read the spec!
    - [ ] Medium 57%,  462: Couldn't find efficient solution
    - [x] Medium 57%,  348: 3 tests, 2 submissions, 18'
      - Mistakes
        - Used array in place of int
        - Made a simplification before submitting, whose assumption was wrong!
      - 1 submission has been wasted due to incomplete specification!
  - Study
    - [ ] Cracking the Coding Interview
      - [ ] VII. Technical Questions

## Fri 04/Mar/2022

- SWE: Practice
  - Leetcode
    - [x] Medium 63%, 1344: 1 tests, 1 submissions, 11'
    - [x] Medium 64%,   49: 2 tests, 1 submissions, 6'
      - Faster than almost all, but suboptimal!
      - Mistake: Forgot to invoke String#chars
    - [ ] Medium 67%,  362: 0 tests, 0 submissions, 20'
      - Failed in the given time; used wrong algorithm
  - Study
    - [ ] Codility lessons
      - [x] 1-4
    - [ ] Cracking the Coding Interview
      - [x] Introduction: I-VI
- Admin
  - [x] Research/buy exercises book

## Thu 03/Mar/2022

- SWE: Practice
  - Leetcode
    - [ ] Medium 71%,  979: failed
    - [x] Easy   77%, 1304: 2 tests, 2 submissions, 12'
    - [x] Easy   68%, 1822: 1 tests, 1 submissions, 3'
    - [x] Easy   64%,  706: x tests, x submissions, ~25'
      - 2 errors: typo, and inadvert shadowing
      - headache: used slow array instantiation API
    - [x] Easy   55%, 1275: 3 tests, 3 submissions, ~25'
      - did not find efficient solution; read it and implemented it
      - inverted modulo operators!!
      - confused array with entry
      - one-off error!
      - missed requirement: movements can be less than 9

## Wed 02/Mar/2022

- SWE: Practice
  - [ ] Codility: Exercises on lessons 5
    - [x] Incorrect: CountDiv
    - [x] Slow?: GenomicRangeQuery
  - Leetcode
    - [ ] Medium 42%,  984: failed
    - [x] Medium 73%, 1448: 2 tests, 1 submissions, 10/15'

## Tue 01/Mar/2022

- System
  - [x] Update remote sessions script to automatically change keyboard layout
- SWE: Practice
  - [x] Codility: Exercises on lessons 1-4
    - [x] Research Ruby speed on array resizing strategies

## Mon 28/Feb/2022

- Studies: C64 ASM Programming
  - [x] Minor checks to SID-generated random numbers
- Tools/Studies
  - [x] Player script
    - [x] Ruby: Review child processes handling
    - [x] Fight player odd handling of Alt+F4 <> SIGINT
    - [x] Linux: Review xdotool for programmatic Alt+F4 sending

## Sun 27/Feb/2022

- Studies: C64 ASM Programming
  - [x] Machine Code Games Routines for the Commodore 64
    - [x] Homing Motion routine
      - [x] Extend with sprite drawing
      - [x] Implement optimization
      - [x] Implement simplification
    - [x] Fix and dump ASM for Hail of Barbs
    - [x] Mark listings project and blog article as completed
  - [x] Investigate SID-generated random numbers frequency
  - [x] Fix VICE issue with keyboard
- Open source
  - [x] Review Crystal-lang and consider for contribution
  - llamaSource project
    - [x] Send PR fixing a README link
- Admin
  - [x] Organize the c64 resources
  - [x] Plan cycle
  - [x] Review potential projects/languages

## Sat 26/Feb/2022

- Studies: C64 ASM Programming
  - [ ] Machine Code Games Routines for the Commodore 64
    - [x] Projecting a Landscape
      - [x] Fix
      - [x] Fight with addition of element plotting
      - [x] Implement simplification
    - [x] Array routines

## Fri 25/Feb/2022

- Studies: C64 ASM Programming
  - [ ] Machine Code Games Routines for the Commodore 64
    - [x] Window Projection
    - [x] Think general loop optimization
- Open source: Openscripts
  - [x] inhibit_screensaver: test, remove obsolence note, and extend comments/help
- Studies: MySQL
  - [ ] High Performance MySQL (4th ed.)
    - [x] 9. Replication

## Thu 24/Feb/2022

- Admin/System
  - [x] Investigate snapd autorestart and disable it
- Studies: MySQL
  - [ ] High Performance MySQL (4th ed.)
    - [ ] 9. Replication
- Studies: C64 ASM Programming
  - [ ] Machine Code Games Routines for the Commodore 64
    - [x] Interrupt-driven Tune Player

## Wed 23/Feb/2022

- Tools: zsh-autosuggestions
  - [x] Find and disable right arrow for accepting a suggestion
- Open source: VSC Kick Assembler Studio extension
  - [x] Investigate and apply patch from mailing list
  - [x] Test most compatible VICE version
  - [x] Open issue about workaround information
  - [x] Create fork with patch and document it

## Tue 22/Feb/2022

- Open Source: Visual Studio Code
  - [x] Follow up discussion with new test case
    - [x] Test other editors
- Studies: C64 ASM Programming
  - [ ] Machine Code Games Routines for the Commodore 64
    - [x] Massive fight with diagnosing issue with VSC plugin<>x64 monitor port
    - [x] Massive fight with sprite vectoring BASIC program port to ASM

## Mon 21/Feb/2022

- Tools
  - Email client directories management
    - [x] Implement archival functionality
  - Subsync start script
    - [x] Create, with snapd handling
- Studies: Regexes
  - [x] Investigate interesting `^` case
- Open Source: Visual Studio Code
  - [x] Prepare test case and report Ruby bug
- Studies: Ruby/Rails
  - [ ] The Complete Guide to Rails Performance
    - [x] Copy notes

## Sun 20/Feb/2022

- Studies: MySQL
  - [ ] High Performance MySQL (4th ed.)
    - [x] 1. MySQL Architecture
- Studies: Bash
  - [x] Review multi-char string splitting
- Tools
  - [x] Update journal script logic
    - [x] Fight regexes
- Studies
  - C64 ASM Programming
    - Machine Code Games Routines for the Commodore 64
      - [x] Test sprite vectoring routine
      - [x] Study/Fight VICE monitor

## Sat 19/Feb/2022

- Tools
  - [x] Fight PDF editor zooming
- Studies: MySQL
  - [ ] High Performance MySQL (4th ed.)
    - [ ] 1. MySQL Architecture

## Fri 18/Feb/2022

- SWE
  - Leetcode
    - [x] Medium 83%, 1409: 4 tests, 1 submissions, 18'
      - Misnamed var, logic bug, used each instead of map
- Studies
  - Ruby/Rails
    - The Complete Guide to Rails Performance
      - [x] Profiling Ruby Memory Usage

## Thu 17/Feb/2022

- SWE
  - Leetcode
    - [x] Medium 83%, 2120: 3 tests, 1 submissions, 13'
      - Missing `each()` again!! Typoed one variable.
- Studies
  - C64 ASM Programming
    - Machine Code Games Routines for the Commodore 64
      - [ ] Review optimized Homing Motion routine
        - [x] Study carry/arithmetic

## Wed 16/Feb/2022

- SWE
  - Leetcode
    - [x] Medium 83%, 2125: 2 tests, 1 submissions, 17'
      - Missing `each()`!
- Studies
  - C64 ASM Programming
    - [x] Machine Code Games Routines for the Commodore 64
      - [x] 10. Indirect Routines
        - [x] Fight simplify and optimize homing routine
      - [x] Appendix A: Utility Programs
      - ~Appendix B: Mini-assembler~
      - [x] Appendix C: Advanced Machine Characteristics
      - ~Appendix D: Laserbike~
    - [x] Search next ASM game programming resources
- Admin
  - [ ] Plan next cycle

## Tue 15/Feb/2022

- SWE
  - Leetcode
    - [x] Medium 83%, 654, ? tests, 1 submission, ~2h
      - Disaster: Spent 1.5+ hours on efficient solution, then 20' or so with inefficient but successful one
- Studies
  - C64 ASM Programming
    - [ ] Machine Code Games Routines for the Commodore 64
      - [ ] 10. Indirect Routines

## Mon 14/Feb/2022

- SWE
  - Leetcode
    - [x] Medium 83%,  535: 2 tests, 1 submissions, 12'
      - [x] Ruby mistake about referencing main scope local variables!
- Tools
  - [x] Contact PDF Expert support with suggestion about feature design
- Open Source
  - Kick Assembler Studio VSC plugin
    - [x] Find test case to reproduce bug
  - [x] Follow up discussion on Ubuntu open issue about bluetooth problem
- Studies
  - C64 ASM Programming
    - [ ] Machine Code Games Routines for the Commodore 64
      - [ ] 10. Indirect Routines
  - Ruby/Rails
    - The Complete Guide to Rails Performance
      - [x] Profiling with Ruby-Prof and Stackprof

## Sun 13/Feb/2022

- SWE
  - Leetcode
    - [x] Medium 84%, 1637: 1 tests, 1 submissions, ~5'
      - Used sorting API; not clear if this is intended to be forbidden or not
- Tools
  - [x] Journal script: Make leet option separately add the entry
- Projects
  - QEMU pinning
    - [x] Review, test, and integrate contributed 6.2.0 patch
- Studies
  - C64 ASM Programming
    - [x] Study segments
    - [ ] Machine Code Games Routines for the Commodore 64
      - [ ] 10. Indirect Routines
        - [ ] Fight routine with presumed multiple bugs

## Sat 12/Feb/2022

- SWE
  - Leetcode
    - [x] Medium 84%, 1038: 2 tests, 1 submissions, 16'
      - Typo (missing param in two fx)
- Tools
  - [x] Update download script
  - [x] Add (base) leet functionality to journal script
  - [x] Add (base) inbox handling script
  - [x] Test all (remaining) PDF annotators
- Open source
  - [x] Report Ubuntu BT breakage by recent package upgrade
- Studies
  - C64 ASM programming
    - [ ] Machine Code Games Routines for the Commodore 64
      - [ ] 10. Indirect Routines

## Fri 11/Feb/2022

- SWE
  - Practice
    - Leetcode
      - [x] Medium 85%, 1315: 3 tests, 1 submissions, 17'
        - Mistaken some data structures (node, array) with their content (int)
- Tools
  - Journal copy script
    - [x] Refactor; add support for fixed additions
  - [x] Review https://www.dendron.so
- Admin/System
  - [x] Laptop: automatic BT restart on resume
- Open Source
  - Schedule manager
    - [x] Move todo checking from the planner to the mover
  - Open scripts
    - [x] Migrate the remaining publishable scripts from the private repository
      - [x] Make all of them generic

## Thu 10/Feb/2022

- SWE
  - Practice
    - Leetcode
      - [x] Medium 85%, 1282: 1 tests, 1 submissions, ~9'
- Studies
  - C64 ASM Programming
    - [ ] Machine Code Games Routines for the Commodore 64
      - [x] Appendix A: Utility Programs (-> UDG)

## Wed 09/Feb/2022

- System
  - [x] Solve kernel vs. admgpu problem
    - [x] Fight 5.4 kernel vs zfs
    - [x] Fight find compatible 5.11 kernel
- Studies
  - C64 ASM Programming
    - [ ] Machine Code Games Routines for the Commodore 64
      - [ ] 10. Indirect Routines
      - [ ] Appendix A: Utility Programs (-> UDG)

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
