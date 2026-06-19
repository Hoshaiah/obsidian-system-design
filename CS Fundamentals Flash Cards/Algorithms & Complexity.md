---
tags:
  - flashcards/algorithms
---

# Algorithms & Complexity

Core algorithms, asymptotic analysis, and interview-grade problem-solving patterns.

---

What's the difference between Big-O, Big-Theta, and Big-Omega?
?
**Quick:** Big-O is upper bound, Big-Omega is lower bound, Big-Theta is tight (both).<br>
**Deeper:** Interviewers say "Big-O" but usually mean Theta. The distinction matters when discussing best/worst/average cases. Quicksort is O(n^2) worst-case, O(n log n) average, Omega(n log n) lower-bound for comparison sorts. Also be aware of amortized (averaged over a sequence) vs expected (over randomness) vs worst-case bounds.

Why is quicksort usually faster than mergesort in practice despite the same O(n log n)?
?
**Quick:** Quicksort sorts in-place with great cache locality and low constant factors; mergesort needs O(n) auxiliary memory and copying.<br>
**Deeper:** Quicksort's partition step is a sequential scan, ideal for prefetchers, while mergesort's merge step jumps between two arrays plus a buffer. However, quicksort's O(n^2) worst case on adversarial inputs forces real implementations to use introsort (switches to heapsort at deep recursion) or use median-of-three / random pivot selection. Mergesort is stable and predictable, which is why it's used for external sorting and Python's Timsort.

When is binary search applicable and what are common off-by-one pitfalls?
?
**Quick:** Binary search needs a monotonic predicate over a sorted/sortable domain; mind the loop invariant and the lo/hi update rule.<br>
**Deeper:** It generalizes far beyond sorted arrays: search the answer space (e.g., "smallest capacity such that we ship in K days"). The classic bug is `mid = (lo + hi) / 2` overflowing for large ints; use `lo + (hi - lo) / 2`. Decide upfront whether your interval is `[lo, hi]` or `[lo, hi)` and update consistently or you'll loop forever.

When should you use BFS vs DFS on a graph?
?
**Quick:** BFS finds shortest paths in unweighted graphs and explores level by level; DFS is for connectivity, topological sort, cycle detection, and uses less memory on deep graphs.<br>
**Deeper:** BFS uses O(width) memory in a queue; DFS uses O(depth) on a stack. For shortest path in weighted graphs, BFS becomes Dijkstra (with a priority queue). DFS underlies recursion-heavy problems like SCCs (Tarjan/Kosaraju), articulation points, and backtracking. Iterative DFS with an explicit stack is essential when call-stack depth might overflow.

What are the three steps of dynamic programming?
?
**Quick:** Define a recurrence with overlapping subproblems and optimal substructure, then memoize or tabulate.<br>
**Deeper:** First, identify the state (what's the minimum info needed to describe a subproblem). Second, express the answer recursively in terms of smaller states. Third, choose top-down memoization (cleaner, lazy) or bottom-up tabulation (often allows O(1) space rolling arrays). DP isn't magic; it's just careful state design. Common pitfall: states must be acyclic in dependency.

When does a greedy algorithm work, and when does it fail?
?
**Quick:** Greedy works when local optimal choices lead to a global optimum (exchange argument or matroid structure); otherwise you need DP or search.<br>
**Deeper:** Classic correct greedies: Huffman coding, Kruskal/Prim MST, interval scheduling by earliest end-time, Dijkstra. Classic failures: 0/1 knapsack (use DP), coin change with arbitrary denominations. Proving greedy correctness usually requires showing that any optimal solution can be transformed into the greedy one without loss.

What is divide and conquer and what's the Master Theorem useful for?
?
**Quick:** Divide and conquer splits a problem into smaller independent subproblems; the Master Theorem solves recurrences of form T(n) = aT(n/b) + f(n).<br>
**Deeper:** Examples: mergesort, quicksort, FFT, Strassen's matrix multiplication, closest-pair-of-points. Master Theorem cases compare f(n) to n^(log_b a): if subproblems dominate it's O(n^log_b a), if work-per-level dominates it's O(f(n) log n) when balanced, etc. For unbalanced recurrences (like quicksort worst-case) you need a recursion tree or substitution.

What's the difference between recursion and iteration, and when does recursion bite you?
?
**Quick:** Recursion is elegant for tree/divide-conquer problems but uses stack space proportional to depth and can overflow.<br>
**Deeper:** Tail-call optimization can eliminate stack growth but isn't guaranteed (Python and Java don't do it; many C++ compilers do; Scala/Scheme require it). For tree traversals or deeply recursive DPs, convert to iterative form with an explicit stack/queue to avoid stack overflow on inputs you'd see in production.

What does P vs NP mean and what is NP-Complete?
?
**Quick:** P is problems solvable in polynomial time; NP is problems whose solutions are verifiable in polynomial time; NP-Complete is the hardest in NP.<br>
**Deeper:** If any NP-Complete problem (SAT, 3-SAT, traveling salesman, vertex cover) has a polynomial-time solution, all of NP does. We strongly suspect P != NP. Practically, recognizing NP-Hardness tells you to stop looking for an exact polynomial algorithm and reach for heuristics, approximation algorithms (e.g., 2-approximation for vertex cover), or constraint solvers.

How does Dijkstra's algorithm work and when can't you use it?
?
**Quick:** Dijkstra greedily expands the closest unvisited node using a min-heap; it requires non-negative edge weights.<br>
**Deeper:** Runs in O((V+E) log V) with a binary heap. Fails on negative weights because a "settled" node could be improved later; use Bellman-Ford O(VE) instead, which also detects negative cycles. For all-pairs shortest paths on dense graphs, Floyd-Warshall is O(V^3). A* generalizes Dijkstra by adding a heuristic to guide search.

What is a stable sort and why does it matter?
?
**Quick:** A stable sort preserves the relative order of equal elements; matters when sorting by multiple keys.<br>
**Deeper:** If you sort records by city, then by name with a stable sort, names with the same city stay alphabetical. Mergesort, insertion sort, and Timsort are stable; quicksort and heapsort are not by default. Stability often matters in database query results and UI sorting where users expect predictable ordering.

How do you choose between iterative and recursive solutions in an interview?
?
**Quick:** Recursive when the problem is tree-shaped or naturally divide-and-conquer; iterative when state is linear or you risk stack overflow.<br>
**Deeper:** A good heuristic: if you'd draw a tree to explain it, recursion is clearer; if you'd draw a number line or loop, iteration is clearer. Express the recursive version first to clarify the recurrence, then convert to iterative for production. Interviewers reward candidates who can do both transformations explicitly.
