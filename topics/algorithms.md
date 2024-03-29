# Algorithms

- [Algorithms](#algorithms)
  - [Notes](#notes)
  - [Generic](#generic)
    - [All permutations of a sequence](#all-permutations-of-a-sequence)
  - [Matrices](#matrices)
  - [Sort](#sort)
    - [Counting sort](#counting-sort)
  - [Graphs](#graphs)
    - [A* search](#a-search)
    - [Min/Max Heap](#minmax-heap)
    - [Matrix smoothing (moving mean/average)](#matrix-smoothing-moving-meanaverage)

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

## Matrices

- (Row/Column)-major array storage: elements of each (R/C) are contiguous in memory

## Sort

### Counting sort

Don't forget counting sort, which is linear! And don't forget that the array must include the **number** of entries, not the **presence**!!

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

### Matrix smoothing (moving mean/average)

Unoptimized version:

- set the kernel size; must be a square with an odd side size
- each destination element in the result is the average of kernel of the corresponding source element (including the element itself)
  - because of the elements on the sides, surround the source matrix with copies of the respective nearest element

Optimized version:

- two stages: horizontal and vertical (order is not important)
- on each stage
  - in order to average an element, use only the two surrounding elements before and after
    - this can be optimized because instead of recalculating the average, the previous element can be subtracted, and the new one added (take into account the division by 3)
  - use the surround strategy, which requires only the addition of two lines (no whole surround)
- apply one stage first, then apply the second on the result of the first
