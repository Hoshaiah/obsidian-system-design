---
tags:
  - flashcards/data-structures
---

# Data Structures

Core data structures, their operations, and when to reach for each.

---

What is the difference between an array and a linked list in terms of access and mutation costs?
?
**Quick:** Arrays give O(1) random access but O(n) insert/delete in the middle; linked lists give O(1) insert/delete at a known node but O(n) access.<br>
**Deeper:** Arrays store contiguous memory so the CPU loves them (cache locality, prefetching), making sequential scans far faster than linked lists in practice even when Big-O is identical. Linked lists win when you need stable iterators during mutation or splicing whole sublists. In real systems, dynamic arrays (vector, ArrayList) dominate because amortized O(1) append plus locality usually beats theoretical advantages of lists.

How does a hash table achieve O(1) average-case lookup, and when does it degrade?
?
**Quick:** It maps a key to a bucket via a hash function; degrades to O(n) when collisions cluster or load factor is too high.<br>
**Deeper:** Performance hinges on a uniform hash, a low load factor (typically resized at 0.7-0.75), and a good collision strategy (chaining vs open addressing with linear/quadratic probing or Robin Hood). Adversarial inputs can force worst-case O(n), which is why Java 8+ promotes long chains to red-black trees and why HashDoS mitigations randomize seeds. Open addressing has better cache behavior; chaining handles high load factors more gracefully.

What is a Binary Search Tree and why do we need self-balancing variants like AVL or Red-Black?
?
**Quick:** A BST keeps left < node < right for O(log n) search, but degenerates to a linked list O(n) if inserted in sorted order; balanced trees enforce height invariants.<br>
**Deeper:** AVL trees maintain a strict height balance (differ by at most 1), giving faster lookups but more rotations on insert/delete. Red-Black trees relax balance (longest path at most 2x shortest), so they have fewer rotations and are preferred in mutation-heavy contexts like the Linux kernel scheduler and most std::map implementations. Choosing between them is a read-vs-write tradeoff.

Why are B-Trees used for databases and filesystems instead of binary trees?
?
**Quick:** B-Trees have high fanout, minimizing disk I/O by packing many keys per node aligned to a page.<br>
**Deeper:** With nodes sized to a disk block (e.g., 4-16 KB), a B-Tree of billions of keys is only 3-4 levels deep, so a lookup is 3-4 disk seeks instead of 30+ in a binary tree. B+ Trees go further by keeping all data in leaves linked together, making range scans (very common in SQL) sequential. This page-oriented design also plays well with the OS buffer cache.

When would you use a Trie over a hash table?
?
**Quick:** Use a Trie when you need prefix queries, autocomplete, or longest-common-prefix in O(k) where k is key length.<br>
**Deeper:** Hash tables can't answer "all keys starting with foo" without scanning everything; Tries traverse character by character in time independent of n. They also enable space sharing for common prefixes (e.g., dictionaries, IP routing tables via radix tries). Downsides: memory overhead per node and worse cache locality compared to compact hash structures.

What is a heap and what problem does it efficiently solve?
?
**Quick:** A heap is a complete binary tree maintaining the heap property (parent <= children for min-heap), giving O(log n) insert and O(1) peek-min.<br>
**Deeper:** Heaps power priority queues used in Dijkstra's, A*, top-k problems, event simulators, and OS schedulers. Binary heaps are array-backed (no pointer overhead, great locality). For decrease-key heavy workloads, Fibonacci heaps offer better amortized bounds, but constants are bad in practice. Pairing heaps strike a real-world balance.

How does a stack differ from a queue, and where do they appear in systems?
?
**Quick:** Stack is LIFO; queue is FIFO. Stacks model recursion/undo; queues model fair scheduling and buffering.<br>
**Deeper:** Stack semantics underpin function call frames, expression evaluation, DFS, and backtracking. Queues underpin BFS, producer-consumer pipelines, kernel run queues, and message brokers. Deques generalize both and are the basis of many concurrent structures (e.g., work-stealing schedulers use deques for lock-free push/pop).

What's the difference between an adjacency list and adjacency matrix for graphs?
?
**Quick:** Adjacency list uses O(V+E) space and is fast for sparse graphs; adjacency matrix uses O(V^2) space and gives O(1) edge lookup.<br>
**Deeper:** Most real-world graphs (social, web, road) are sparse, so adjacency lists dominate; they iterate neighbors in O(degree). Matrices win for dense graphs and algorithms that need fast edge existence checks (Floyd-Warshall, transitive closure). For massive graphs, compressed sparse row (CSR) or edge lists in column stores are preferred.

What is amortized analysis and why does it matter for dynamic arrays?
?
**Quick:** Amortized analysis averages cost across operations; dynamic array append is O(1) amortized even though some appends trigger O(n) resizes.<br>
**Deeper:** Doubling the array on resize means the total resize work across n inserts is O(n), so each insert pays O(1) on average. If you grew by a constant instead of a factor, you'd get O(n) amortized per insert. This is why Python lists, Java ArrayLists, and C++ vectors all use geometric growth (factor 1.5-2). The choice trades off memory waste vs allocation frequency.

When should you pick a skip list over a balanced BST?
?
**Quick:** Skip lists offer similar O(log n) bounds with simpler code and easier lock-free concurrent implementations.<br>
**Deeper:** Redis uses skip lists for sorted sets because they're easier to implement concurrently and have predictable performance under range queries. They have higher constant factors and worse memory locality than a B-tree, but easier to reason about than red-black trees. The randomization avoids worst-case rebalancing logic entirely.

What is a Bloom filter and what tradeoff does it make?
?
**Quick:** A space-efficient probabilistic set that gives no false negatives but possible false positives.<br>
**Deeper:** Bloom filters use k hash functions to set bits in a bit array; a query checks all k bits. They are heavily used in databases (LSM trees like Cassandra/RocksDB check Bloom filters before disk reads), CDNs, and web crawlers to avoid expensive lookups. Tradeoff: you trade certainty for ~10x space savings, but you can never delete (counting Bloom filters relax this).

How does a Union-Find (Disjoint Set) achieve near-constant time?
?
**Quick:** Path compression plus union-by-rank give nearly O(1) amortized per operation, technically O(alpha(n)).<br>
**Deeper:** Each element points to a parent; find walks to the root and flattens the path. Union-by-rank attaches the shorter tree under the taller. Used in Kruskal's MST, network connectivity, image segmentation, and dynamic graph connectivity. The inverse Ackermann function alpha(n) is below 5 for any practical n.
