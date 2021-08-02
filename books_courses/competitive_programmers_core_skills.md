# Competitive Programmer's Core Skills

- [Competitive Programmer's Core Skills](#competitive-programmers-core-skills)
  - [General notes](#general-notes)

## General notes

- Read the definition exactly; a nuance may be the key
- Pay attention to unexpected values
- Always check the smallest/empty and largest case; too slow is also a bug

- Loop invariants (pre/post)
- Search space

- Brute force: 1. Find the search space 2. Found how to enumerate it

- Think the worst possible case and include it in the tests

- Invariant: correct statement about data being worked on
- Use invariants in places of the workflow to find nearby errors
- Use asserts

- Greedy algorithms: Prove that the restricted search space contains the optimal solution
- Start with a known optimal solution; if it doesn't belong, tweak the algorithm
