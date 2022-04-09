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
