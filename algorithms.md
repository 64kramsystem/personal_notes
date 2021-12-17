# Algorithms

- [Algorithms](#algorithms)
  - [Notes](#notes)
  - [Generic](#generic)
    - [All permutations of a sequence](#all-permutations-of-a-sequence)
  - [Graphs](#graphs)
    - [A* search](#a-search)
    - [Min/Max Heap](#minmax-heap)

## Notes

The implementations are not intended to be optimized.

## Generic

### All permutations of a sequence

Swap the leftmost element of each subarray with each other element, and for each swap, recurse on the subarray starting at the next position.

Complexity: O(n!), however, if the block processes each permuted array by iterating it, it becomes O(n * n!); in this case, it's possible to reduce the complexity back to O(n!) by integrating the processing in this routine, at the cost of readability/coupling.

```rb
# Execute the block on each permutation.
def permutations(sequence, start_position = 0, &block)
  if start_position == sequence.size
    yield sequence
    return # technically optional, but confusing if omitted
  end

  start_position.upto(sequence.size - 1) do |new_position|
    sequence[start_position], sequence[new_position] = sequence[new_position], sequence[start_position]

    permutations(sequence, start_position + 1, &block)

    # By restoring the sequence, we don't need to use a temporary array.
    sequence[start_position], sequence[new_position] = sequence[new_position], sequence[start_position]
  end
end
```

## Graphs

### A* search

Terms:

- `G` = distance from start
- `H` (heuristic) = distance from target
- `F` = `G` + `H`

Simplified interpretation:

- for each iteration
- find the node with the lowest cost (/+distance) from the open ones
- close the node (=not searched anymore)
- compute the neighbors, and update them if advantageous/add if not in list
- repeat until the path to the target is found

Pseudo-pseudo-code:

```rb
open_nodes = []
closed_nodes = []

open_nodes.append start_node

loop do
  # If there are multiple, choose the one closest to the target
  current_node = find_lowest_f_cost_node_from open_nodes

  open_nodes.delete current_node
  closed_nodes.append current_node

  break if current_node == target_node

  current_node.neighbours.each do |neighbour_node|
    next if !neighbour_node.traversable? || neighbour_node.closed?

    neighbour_node_new_f_cost = compute_f_cost current_node, neighbour_node

    if neighbour_node_new_f_cost < neighbour_node.f_cost || !neighbour_node.in?(open_nodes)
      neighbour_node.f_cost = neighbour_node_new_f_cost
      neighbour_node.parent = current_node

      open_nodes.append(neighbour_node) if !neighbour_node.in?(open_nodes)
    end
  end
end
```

### Min/Max Heap

Min/max heaps are used to store comparable values, and quickly retrieve the highest priority value (min or max, depending on the heap type).

Resizing a heap is complex, so the easiest implementation has a fixed size, backed by an array.

All the levels except the last (which is variable) have fixed size - powers of 2; therefore, the parent/children positions of a given position can be found, respectively, via ((n - 1) / 2) and (2n + {1,2}).

It's semiordered:

- the values of a level are not ordered,
- but any value of a level has lower priority than the values of the previous level.

Addition:

- add as last
- while has higher priority than the parent, swap with it

Removal (of the highest priority):

- pop first and replace with last
- while it's lower priority than the children, swap with the higher priority one
