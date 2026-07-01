# All Algorithms

This file serves as a central repository for all algorithms studied in the 3rd Semester Algorithm course. Each entry includes C++ implementation, pseudocode, recursion analysis, and complexity details.

## Algorithm Summary Table

| Algorithm Name     | Best Case Time    | Average Case Time | Worst Case Time   | Space Complexity | Use Case                                               | Keywords                                                                     |
| :----------------- | :---------------- | :---------------- | :---------------- | :--------------- | :----------------------------------------------------- | :--------------------------------------------------------------------------- |
| **Prim's**          | $O((V+E) \log V)$  | $O((V+E) \log V)$  | $O((V+E) \log V)$  | $O(V + E)$       | Find MST in dense graphs; grows tree from a node.        | "Minimum weight spanning tree", "Priority queue", "Connected nodes".         |
| **Dijkstra's**      | $O(V + E \log V)$  | $O(E \log V)$      | $O(E \log V)$      | $O(V + E)$       | Single-source shortest path with non-negative weights.   | "Shortest path", "Non-negative weights", "Quickest route".                   |
| **Bellman-Ford**    | $O(E)$             | $O(VE)$            | $O(VE)$            | $O(V)$           | Single-source shortest path with negative weights.       | "Negative weights", "Negative cycle detection", "Relax all edges".           |
| **Floyd-Warshall**  | $O(V^3)$           | $O(V^3)$           | $O(V^3)$           | $O(V^2)$         | All-pairs shortest paths in small to medium graphs.      | "All pairs shortest path", "Intermediate vertices", "Matrix representation". |
| **Kruskal's**       | $O(E \log E)$      | $O(E \log E)$      | $O(E \log E)$      | $O(V + E)$       | Find MST in sparse graphs; sorts edges and adds them.    | "Minimum weight spanning tree", "Edge list", "DSU", "Sort edges".            |
| **BFS**             | $O(V + E)$         | $O(V + E)$         | $O(V + E)$         | $O(V)$           | Level-order search, shortest path on unweighted graphs.  | "Queue", "Level-order traversal", "Frontier search", "Breadth first".        |
| **DFS**             | $O(V + E)$         | $O(V + E)$         | $O(V + E)$         | $O(V)$           | Cycle detection, pathfinding, topological sorting.       | "Stack", "Backtracking", "Depth search", "DFS tree".                         |
| **Topological Sort**| $O(V + E)$         | $O(V + E)$         | $O(V + E)$         | $O(V)$           | Dependency resolution, task scheduling.                  | "DAG", "Kahn's algorithm", "Indegree", "Prerequisites".                      |
| **Bubble Sort**     | $O(N)$             | $O(N^2)$           | $O(N^2)$           | $O(1)$           | Small lists, checking if list is already sorted.         | "Adjacent swaps", "Early exit flag", "Bubble pass".                          |
| **Selection Sort**  | $O(N^2)$           | $O(N^2)$           | $O(N^2)$           | $O(1)$           | Minimizing swaps / writes in memory constraints.         | "Find minimum index", "Selection swaps".                                     |
| **Insertion Sort**  | $O(N)$             | $O(N^2)$           | $O(N^2)$           | $O(1)$           | Nearly sorted lists, sorting incoming stream online.     | "Card sorting analogy", "Sorted sublist insertion".                          |
| **Merge Sort**      | $O(N \log N)$      | $O(N \log N)$      | $O(N \log N)$      | $O(N)$           | Stable sorting, large arrays, linked list sorting.       | "Divide and conquer", "Merge recursion", "Two-way merge".                    |
| **Counting Sort**   | $O(N + K)$         | $O(N + K)$         | $O(N + K)$         | $O(N + K)$       | Integer arrays with small range of values ($K$).         | "Non-comparison", "Prefix sum array", "Frequency bucket".                    |
| **Radix Sort**      | $O(D(N + K))$      | $O(D(N + K))$      | $O(D(N + K))$      | $O(N + K)$       | Integer keys with fixed length / digit values.           | "LSD digit sorting", "Digit counting sort helper".                           |

### Complexity Notation Key:
* **$V$**: Number of vertices (nodes) in the graph.
* **$E$**: Number of edges (connections) in the graph.
* **$N$**: Number of elements (size) of the input array.
* **$K$**: Range of the input values (difference between maximum and minimum value) in non-comparison sorting.
* **$D$**: Number of digits (or length of keys) of the maximum element in radix sort.

---

## Time and Space Complexity Fundamentals

### 1. Asymptotic Notations
In academic algorithm analysis, we use **asymptotic notation** to describe the running time or memory requirements of an algorithm as the input size grows toward infinity. This allows us to compare algorithms independent of hardware, compiler, or programming language differences.

*   **Big O Notation ($O$):** Defines an asymptotic **upper bound**. A function $f(n) = O(g(n))$ if there exist positive constants $c$ and $n_0$ such that $f(n) \le c \cdot g(n)$ for all $n \ge n_0$. It represents the worst-case scenario.
*   **Big Omega Notation ($\Omega$):** Defines an asymptotic **lower bound**. A function $f(n) = \Omega(g(n))$ if there exist positive constants $c$ and $n_0$ such that $f(n) \ge c \cdot g(n)$ for all $n \ge n_0$. It represents the best-case scenario.
*   **Big Theta Notation ($\Theta$):** Defines an asymptotic **tight bound**. A function $f(n) = \Theta(g(n))$ if there exist positive constants $c_1$, $c_2$, and $n_0$ such that $c_1 \cdot g(n) \le f(n) \le c_2 \cdot g(n)$ for all $n \ge n_0$. It indicates that the running time is proportional to $g(n)$ in all cases.

### 2. Time Complexity Cases
*   **Best Case (Lower Bound):** The minimum number of steps required to execute an algorithm for an input of size $N$ that is in the most favorable configuration.
    *   *Example:* In [bubbleSort](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L829-L841), if the array is already sorted, the early exit optimization checks all elements in one pass without swapping, resulting in a best-case time complexity of $\Omega(N)$ (or $O(N)$).
*   **Average Case (Expected Case):** The expected number of steps over all possible inputs of size $N$. This requires calculating the probability distribution of all possible inputs.
    *   *Example:* In [insertionSort](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L960-L971), on average, we swap half of the elements in the sorted sub-array, leading to an average-case time complexity of $O(N^2)$.
*   **Worst Case (Upper Bound):** The maximum number of steps required for any input of size $N$. It guarantees that the algorithm will not take longer than this bound.
    *   *Example:* In [insertionSort](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L960-L971), when the input is sorted in reverse order, every element must be compared and shifted all the way to the beginning, resulting in a worst-case time complexity of $O(N^2)$.

### 3. Space Complexity & Memory Management
Space complexity measures the total memory space required by an algorithm to run to completion as a function of the input size.

$$\text{Total Space Complexity} = \text{Input Space} + \text{Auxiliary Space}$$

*   **Input Space:** The memory required to store the input data.
*   **Auxiliary Space:** The temporary/extra space allocated by the algorithm during its execution (excluding the input itself).
    *   **In-place Sorting:** Algorithms like [bubbleSort](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L829-L841), [selectionSort](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L895-L908), and [insertionSort](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L960-L971) require $O(1)$ auxiliary space because they swap elements directly inside the input array.
    *   **Out-of-place Sorting:** [mergeSort](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L1053-L1060) requires $O(N)$ auxiliary space to allocate temporary arrays `L` and `R` for storing divided sub-arrays before merging.
    *   **Recursion Stack Space:** Recursive algorithms consume call stack space proportional to the depth of recursion. For instance, [dfsRecursive](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L638-L647) has a worst-case call stack depth of $O(V)$ in a graph with a single linear path.

### 4. Mathematical Analysis Techniques

#### Iterative Loop Analysis
To compute the time complexity of iterative code:
1.  Identify the loops and their nesting.
2.  Set up a summation representing the number of times the inner loop body executes.
3.  Solve the summation mathematically.

For example, in [selectionSort](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L895-L908), the outer loop runs $N-1$ times, and the inner loop runs from $i+1$ to $N-1$. The total number of comparisons is:

$$\sum_{i=0}^{N-2} \sum_{j=i+1}^{N-1} 1 = \sum_{i=0}^{N-2} (N - 1 - i) = (N-1) + (N-2) + \dots + 1 = \frac{N(N-1)}{2} = O(N^2)$$

#### Recursive Analysis (Divide and Conquer)
Recursive time complexity is expressed via recurrence relations. For example, [mergeSort](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L1053-L1060) splits the array of size $N$ into two sub-arrays of size $N/2$ and merges them in linear time:

$$T(N) = 2T\left(\frac{N}{2}\right) + O(N)$$

Using the **Master Theorem** for recurrences of the form $T(N) = aT(N/b) + f(N)$:
*   $a = 2, b = 2, f(N) = O(N^1)$
*   Compare $f(N) = O(N^d)$ with $N^{\log_b a} = N^{\log_2 2} = N^1$.
*   Since $d = \log_b a = 1$, Case 2 of the Master Theorem applies:
    
    $$T(N) = \Theta\left(N^{\log_b a} \log N\right) = \Theta(N \log N)$$

### 5. Practical Complexity Classes and Trade-offs

| Class | Growth Rate | Description | Examples |
| :--- | :--- | :--- | :--- |
| $O(1)$ | Constant | Operations do not depend on input size. | [DSU::unite](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L158-L162) (with path compression and union by rank). |
| $O(\log N)$ | Logarithmic | Input size is divided in half at each step. | Binary Search, Priority Queue insertion/extraction. |
| $O(N)$ | Linear | Step count scales proportionally with input size. | [countingSort](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L1132-L1156) (when $K$ is small). |
| $O(N \log N)$ | Linearithmic | Divide-and-conquer processing of elements. | [mergeSort](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L1053-L1060), [kruskal](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L165-L180) (due to sorting edges). |
| $O(N^2)$ | Quadratic | Nested iterations over the input size. | [bubbleSort](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L829-L841), [selectionSort](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L895-L908). |
| $O(V^3)$ | Cubic | Triply nested iterations over vertices. | [floydWarshall](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L460-L482). |
| $O(2^N)$ | Exponential | Doubling steps with each addition to input. | Unoptimized recursive Fibonacci or subset sum solutions. |

#### Real-World Time-Space Trade-off
*   **Counting Sort vs. Merge Sort:** [countingSort](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L1132-L1156) performs sorting in $O(N+K)$ time, which is faster than the $O(N \log N)$ lower bound of comparison-based sorting like [mergeSort](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L1053-L1060). However, it requires $O(N+K)$ auxiliary space. If the range of values $K$ is extremely large (e.g., $K = 10^9$ for a small array of size $N=10$), Counting Sort is highly impractical due to memory limits, making Merge Sort a better candidate.

#### Real-World Graph Algorithm Selection
*   **Dense vs. Sparse Graphs:** For finding a Minimum Spanning Tree:
    *   [kruskal](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L165-L180) takes $O(E \log E)$ (which is $O(E \log V)$ since $E \le V^2$).
    *   [prim](file:///Users/apple/Obsidian/IUT/Human/University/4th%20Sem/Algorithm/All%20Algorithms%20folder.md#L52-L75) using a binary heap takes $O((V+E) \log V)$.
    *   In a **sparse graph** where $E \approx V$, Kruskal's simplifies to $O(V \log V)$, which is efficient and straightforward to implement.
    *   In a **dense graph** where $E \approx V^2$, Kruskal's sorts $V^2$ edges, taking $O(V^2 \log V)$ time. Prim's algorithm using a Fibonacci heap can achieve a running time of $O(E + V \log V) \approx O(V^2)$, making it theoretically superior for dense networks.

---

## Minimum Spanning Tree (MST)

A Minimum Spanning Tree of a connected, undirected graph is a subgraph that is a tree and connects all the vertices together with the minimum possible total edge weight.


### 1. Prim's Algorithm

Prim's builds the MST starting from a single node, adding the cheapest edge from the tree to a vertex not yet in the tree.

#### C++ Implementation
```cpp
#include <iostream>
#include <vector>
#include <queue>

using namespace std;

typedef pair<int, int> pii;

void prim(int start, int n, vector<vector<pii>>& adj) {
    priority_queue<pii, vector<pii>, greater<pii>> pq;
    vector<bool> visited(n, false);
    pq.push({0, start});
    int cost = 0;

    while (!pq.empty()) {
        pii top = pq.top();
        pq.pop();
        int u = top.second;
        int w = top.first;

        if (visited[u]) continue;
        visited[u] = true;
        cost += w;

        for (auto& edge : adj[u]) {
            int v = edge.first;
            int weight = edge.second;
            if (!visited[v]) pq.push({weight, v});
        }
    }
    cout << "Total Cost: " << cost << endl;
}
```

#### Step-by-Step Code Walkthrough
1. **Priority Queue:** Uses a min-priority queue to store `(weight, vertex)` pairs. The `greater<pii>` ensures the smallest weight is at the top.
2. **Visited Array:** Keeps track of vertices already included in the MST to avoid cycles and redundant work.
3. **Start Node:** Pushes the start node with weight 0 into the PQ.
4. **Extraction:** Pops the vertex `u` with the smallest edge weight `w` connecting to the current MST.
5. **Greedy Step:** If `u` is not visited, mark it visited, add its weight to the total cost, and explore its neighbors.
6. **Neighbor Exploration:** For every neighbor `v` of `u`, if `v` is not in the MST, push `(weight, v)` into the PQ.

#### Pseudocode
```text
PRIM(G, start):
    visited = array of size V initialized to False
    PQ = Min-Priority Queue of (weight, vertex)
    PQ.push((0, start))
    total_cost = 0

    while PQ is not empty:
        (w, u) = PQ.pop()
        if visited[u] is True:
            continue
        visited[u] = True
        total_cost = total_cost + w

        for each neighbor (v, weight) of u:
            if visited[v] is False:
                PQ.push((weight, v))
```

#### Recursive Analysis
- **Recursive Calls:** Standard implementation is iterative. A recursive version `primRecursive(u)` would visit neighbors and then call itself for the next closest vertex.
- **Recursion Frequency:** Would occur $V$ times (once for each vertex).
- **Recursive Alternatives for Loops:**
    - `while (!pq.empty())`: Can be replaced by `processPQ() { if !pq.empty(): ... processPQ() }`.
    - `for (auto& edge : adj[u])`: Can be replaced by `visitNeighbors(index)`.

#### Complexity Analysis
- **Loops:**
    - Main Loop: $V$ iterations (each vertex is extracted from the priority queue exactly once).
    - Priority Queue operations: $O(E \log V)$ or $O(E \log E)$ (each edge is explored and potentially inserted/updated in the priority queue).
- **Overall Complexity:**
    - **Best Case:** $O((V + E) \log V)$ — Occurs when the graph is sparse or connected; we must still visit all $V$ vertices and examine all $E$ edges to verify the minimum weight spanning tree.
    - **Average Case:** $O((V + E) \log V)$ — Expected behavior over random graphs where all edges are checked and vertices are pushed to the priority queue.
    - **Worst Case:** $O((V + E) \log V)$ — Occurs in dense graphs where every edge relaxation results in a priority queue update.
- **Space Complexity:** $O(V + E)$ — Required to store the graph in an adjacency list ($O(V + E)$), the priority queue ($O(V)$), and tracking arrays like `visited` ($O(V)$).

#### Example
Graph: A-B (2), B-C (3), A-C (1). Start at A.
1. A's neighbors: B(2), C(1). Pick C.
2. C's neighbors: B(3). PQ has B(2) from A and B(3) from C. Pick B(2).
MST: A-C, A-B. Total cost: 3.

### 2. Kruskal's Algorithm

Kruskal's finds the MST by sorting all edges from lowest to highest weight and adding them to the MST if they don't form a cycle.

#### C++ Implementation
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

struct Edge {
    int u, v, w;
    bool operator<(const Edge& other) const {
        return w < other.w;
    }
};

struct DSU {
    vector<int> parent;
    DSU(int n) {
        parent.resize(n);
        for (int i = 0; i < n; i++) parent[i] = i;
    }
    int find(int i) {
        if (parent[i] == i) return i;
        return parent[i] = find(parent[i]);
    }
    void unite(int i, int j) {
        int root_i = find(i);
        int root_j = find(j);
        if (root_i != root_j) parent[root_i] = root_j;
    }
};

void kruskal(int n, vector<Edge>& edges) {
    sort(edges.begin(), edges.end());
    DSU dsu(n);
    int cost = 0;
    vector<Edge> mst;

    for (auto& edge : edges) {
        if (dsu.find(edge.u) != dsu.find(edge.v)) {
            dsu.unite(edge.u, edge.v);
            cost += edge.w;
            mst.push_back(edge);
        }
    }
    cout << "Total Cost: " << cost << endl;
}
```

#### Step-by-Step Code Walkthrough
1. **Edge Sorting:** All edges are sorted in non-decreasing order of their weights.
2. **Disjoint Set Union (DSU):** A data structure to keep track of connected components and detect cycles efficiently.
3. **Find Operation:** `find(i)` returns the root of the component containing `i`.
4. **Union Operation:** `unite(i, j)` merges the components containing `i` and `j`.
5. **Iterate Edges:** For each edge $(u, v)$, if $u$ and $v$ are in different components, add the edge to MST and unite the components.
6. **Cycle Prevention:** If `find(u) == find(v)`, adding $(u, v)$ would create a cycle, so we skip it.

#### Pseudocode
```text
KRUSKAL(G):
    sort the edges of G by weight in ascending order
    initialize DSU with V elements
    MST = empty list

    for each edge (u, v, weight) in sorted edges:
        if find(u) != find(v):
            union(u, v)
            add (u, v) to MST

    return MST
```

#### Recursive Analysis
- **Recursive Calls:** `find(i)` is typically recursive with path compression.
- **Recursion Frequency:** Number of calls depends on the tree height; path compression makes it nearly constant $O(\alpha(V))$.
- **Recursive Alternatives for Loops:**
    - `for (auto& edge : edges)`: `processEdges(index) { if (index < edges.size()) { ... processEdges(index+1); } }`.
    - `sort()`: Sorting algorithms like Merge Sort or Quick Sort are inherently recursive.

#### Complexity Analysis
- **Loops:**
    - Sorting edges: $O(E \log E)$ — Sorting all $E$ edges in non-decreasing order of weights.
    - Main Loop: $E$ iterations — Scanning each sorted edge once.
    - DSU operations: $O(E \alpha(V))$, where $\alpha$ is the inverse Ackermann function (extremely slow-growing, practically constant).
- **Overall Complexity:**
    - **Best Case:** $O(E \log E)$ — Sorting all edges is always performed first, dominating the time complexity.
    - **Average Case:** $O(E \log E)$ — Expected sorting time and subsequent union-find operations across random edge lists.
    - **Worst Case:** $O(E \log E)$ — Reverse-sorted or fully dense graph where sorting takes maximum operations.
- **Space Complexity:** $O(V + E)$ — Required for storing the edge list of size $E$ and the parent/rank arrays of size $V$ in the DSU.

#### Example
Graph: A-B (2), B-C (3), A-C (1).
1. Sorted edges: (A, C, 1), (A, B, 2), (B, C, 3).
2. Edge (A, C, 1): root(A) != root(C). Add to MST. DSU: {A, C}, {B}.
3. Edge (A, B, 2): root(A) != root(B). Add to MST. DSU: {A, B, C}.
4. Edge (B, C, 3): root(B) == root(C). Skip.
MST: (A, C), (A, B). Total cost: 3.


---

## Shortest Path Algorithms

Shortest path algorithms find the path between two vertices in a graph such that the sum of the weights of its constituent edges is minimized.

### 1. Dijkstra's Algorithm

Dijkstra's finds the shortest path from a source vertex to all other vertices in a weighted graph with non-negative edge weights.

#### C++ Implementation
```cpp
#include <iostream>
#include <vector>
#include <queue>

using namespace std;

const int INF = 1e9;
typedef pair<int, int> pii;

void dijkstra(int start, int n, vector<vector<pii>>& adj) {
    vector<int> dist(n, INF);
    priority_queue<pii, vector<pii>, greater<pii>> pq;

    dist[start] = 0;
    pq.push({0, start});

    while (!pq.empty()) {
        int d = pq.top().first;
        int u = pq.top().second;
        pq.pop();

        if (d > dist[u]) continue;

        for (auto& edge : adj[u]) {
            int v = edge.first;
            int weight = edge.second;
            if (dist[u] + weight < dist[v]) {
                dist[v] = dist[u] + weight;
                pq.push({dist[v], v});
            }
        }
    }
    for(int i=0; i<n; i++) cout << "Vertex " << i << " Distance: " << dist[i] << endl;
}
```

#### Step-by-Step Code Walkthrough
1. **Distance Array:** Initializes all distances to infinity (`INF`), except the start node which is 0.
2. **Min-Priority Queue:** Stores pairs of `(distance, vertex)`. We always process the vertex with the smallest known distance first.
3. **Relaxation:** For the current vertex `u`, we look at all neighbors `v`.
4. **Distance Update:** If the current distance to `u` + edge weight `(u, v)` is less than the currently known distance to `v`, we update `dist[v]`.
5. **PQ Update:** Whenever a distance is updated, the new `(dist[v], v)` is pushed into the PQ to potentially relax its own neighbors later.
6. **Convergence:** The process continues until the PQ is empty, meaning all reachable vertices have their shortest paths found.

#### Pseudocode
```text
DIJKSTRA(G, start):
    dist = array of size V filled with INF
    dist[start] = 0
    PQ = Min-Priority Queue of (distance, vertex)
    PQ.push((0, start))

    while PQ is not empty:
        (d, u) = PQ.pop()
        
        if d > dist[u]:
            continue

        for each neighbor (v, weight) of u:
            if dist[u] + weight < dist[v]:
                dist[v] = dist[u] + weight
                PQ.push((dist[v], v))
```

#### Recursive Analysis
- **Recursive Calls:** Standard Dijkstra is iterative. A recursive version would involve `dijkstraRecursive(pq)` where each call extracts one vertex and calls itself.
- **Recursion Frequency:** $V$ times.
- **Recursive Alternatives for Loops:**
    - `while (!pq.empty())`: `processDijkstra() { if (!pq.empty()) { ... processDijkstra(); } }`.
    - `for (auto& edge : adj[u])`: `relaxNeighbors(u, index) { if (index < adj[u].size()) { ... relaxNeighbors(u, index+1); } }`.

#### Complexity Analysis
- **Loops:**
    - Initialization: $O(V)$ time to set up the `dist` array.
    - While loop: $V$ iterations (extracting the minimum distance vertex from the priority queue).
    - For loop: $E$ iterations total across all steps to relax adjacent edges.
- **Overall Complexity:**
    - **Best Case:** $O(V + E \log V)$ (using binary heap, where few relaxations update the priority queue) or $O(E + V \log V)$ (using Fibonacci heap, which has amortized $O(1)$ decrease-key).
    - **Average Case:** $O(E \log V)$ — Expected time over random configurations where edge relaxation and heap updates occur dynamically.
    - **Worst Case:** $O(E \log V)$ (for connected graphs, where $E \ge V-1$) — Occurs when every edge relaxation updates the heap, leading to $O(E)$ heap updates, each taking $O(\log V)$.
- **Space Complexity:** $O(V + E)$ — Required for storing the graph in an adjacency list ($O(V + E)$), the priority queue ($O(V)$), and the distance array ($O(V)$).

#### Example
Graph: A-B (4), A-C (2), C-B (1), B-D (3). Source A.
1. Dist[A]=0, others INF. PQ: {(0, A)}.
2. Extract A. Neighbors: B(4), C(2). Dist[B]=4, Dist[C]=2. PQ: {(2, C), (4, B)}.
3. Extract C. Neighbor: B. Dist[B] was 4, new is 2+1=3. Dist[B]=3. PQ: {(3, B), (4, B)}.
4. Extract B. Neighbor: D. Dist[D]=3+3=6. PQ: {(6, D), (4, B)}.
Shortest distance to D is 6.

---

### 2. Bellman-Ford Algorithm

Bellman-Ford finds shortest paths from a single source to all other vertices in a weighted graph, even with negative edge weights (but no negative cycles).

#### C++ Implementation
```cpp
#include <iostream>
#include <vector>

using namespace std;

const int INF = 1e9;

struct Edge {
    int u, v, weight;
};

void bellmanFord(int start, int n, int e, vector<Edge>& edges) {
    vector<int> dist(n, INF);
    dist[start] = 0;

    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < e; j++) {
            if (dist[edges[j].u] != INF && dist[edges[j].u] + edges[j].weight < dist[edges[j].v]) {
                dist[edges[j].v] = dist[edges[j].u] + edges[j].weight;
            }
        }
    }

    // Check for negative cycles
    for (int j = 0; j < e; j++) {
        if (dist[edges[j].u] != INF && dist[edges[j].u] + edges[j].weight < dist[edges[j].v]) {
            cout << "Graph contains negative weight cycle" << endl;
            return;
        }
    }
}
```

#### Step-by-Step Code Walkthrough
1. **Distance Array:** Initializes all distances to `INF`, except `dist[start] = 0`.
2. **Relaxation Passes:** The algorithm runs $V-1$ passes over all edges.
3. **Core Relaxation:** For every edge `(u, v)` with weight `w`, it checks if `dist[u] + w < dist[v]`. If true, it updates `dist[v]`.
4. **Why $V-1$ Passes?** The shortest path in a graph with $V$ vertices can have at most $V-1$ edges. Each pass guarantees that paths of length $i$ are correctly calculated.
5. **Negative Cycle Detection:** After $V-1$ passes, it runs one more pass. If any distance can *still* be shortened, it means a negative cycle exists.

**Q. explain why bellman ford is not good**
**ANS:** Bellman-Ford is considered less efficient than other shortest-path algorithms like Dijkstra's primarily due to its **Time Complexity** of $O(VE)$. In a dense graph where $E \approx V^2$, the complexity becomes $O(V^3)$, which is significantly slower than Dijkstra's $O(E \log V)$. It performs redundant work by relaxing every single edge in every iteration, even if the shortest path to a vertex has already been finalized in an earlier pass.

#### Real-World Example: Large-Scale Network Routing
In massive networks like the global internet routing infrastructure (BGP), using a pure Bellman-Ford approach would be prohibitive. For a network with 100,000 routers ($V$) and 500,000 links ($E$), Bellman-Ford would require up to 50 billion operations, whereas Dijkstra would require roughly 10 million. This massive difference in computational cost is why Bellman-Ford is typically reserved only for graphs where negative edge weights are a possibility or for specific distance-vector protocols where its distributed nature is an advantage.

#### Pseudocode
```text
BELLMAN-FORD(G, start):
    dist = array of size V filled with INF
    dist[start] = 0

    // Relax all edges V-1 times
    for i = 1 to V - 1:
        for each edge (u, v, weight) in G.edges:
            if dist[u] != INF and dist[u] + weight < dist[v]:
                dist[v] = dist[u] + weight

    // Check for negative cycles
    for each edge (u, v, weight) in G.edges:
        if dist[u] != INF and dist[u] + weight < dist[v]:
            return "Negative cycle detected"

    return dist
```

#### Recursive Analysis
- **Recursive Calls:** Iterative by design. Could be implemented as `relaxAllEdges(iterationCount)` where it calls itself until `iterationCount == V-1`.
- **Recursion Frequency:** $V-1$ calls.
- **Recursive Alternatives for Loops:**
    - Outer loop: `iterateRelaxation(k) { if (k < V-1) { relaxAll(); iterateRelaxation(k+1); } }`.
    - Inner loop: `relaxEdgeList(index) { if (index < E) { relax(edges[index]); relaxEdgeList(index+1); } }`.

#### Complexity Analysis
- **Loops:**
    - Outer loop: Runs $V-1$ times (since the longest path without cycles in a graph can have at most $V-1$ edges).
    - Inner loop: Iterates through all $E$ edges to perform relaxations in each pass.
- **Overall Complexity:**
    - **Best Case:** $O(E)$ — Occurs if an early-exit optimization checks whether any distance updates occurred, and the first pass relaxes the edges in topological order (propagating all updates in one step).
    - **Average Case:** $O(VE)$ — General scenario where paths propagate slowly over multiple iterations.
    - **Worst Case:** $O(VE)$ — Occurs when the graph structure forces updates to propagate one edge at a time (e.g., a linear chain processed in reverse edge-list order), requiring all $V-1$ sweeps of $E$ edges plus one cycle-detection pass of $O(E)$.
- **Space Complexity:** $O(V)$ — Storing the 1D distance array `dist` of size $V$. The input edge representation is outside of this auxiliary calculation.

#### Example
A-B (1), B-C (3), A-C (10). Source A.
- Pass 1: Dist[A]=0, Dist[B]=1, Dist[C]=4.
- Pass 2: No changes.
Result: Shortest paths found in $V-1$ passes.

---

## All-Pairs Shortest Path (APSP)

The All-Pairs Shortest Path (APSP) problem aims to find the shortest paths between all pairs of vertices $u, v \in V$ in a graph.

### Summary of APSP Approaches

| Approach | Negative Edges? | Complexity (Sparse) | Complexity (Dense) | Best Suited For |
| :--- | :--- | :--- | :--- | :--- |
| **$V \times$ Dijkstra** | No | $O(VE + V^2 \log V)$ | $O(V^3)$ | Sparse graphs with no negative edges |
| **$V \times$ Bellman-Ford** | Yes | $O(V^2E)$ | $O(V^4)$ | Never preferred (highly inefficient) |
| **Floyd-Warshall** | Yes (no negative cycles) | $O(V^3)$ | $O(V^3)$ | Small/dense graphs with negative edges |

---

### 1. Floyd-Warshall Algorithm

Floyd-Warshall is an all-pairs shortest path algorithm. It finds the shortest distances between every pair of vertices in a weighted graph (can have negative weights, but no negative cycles).

### C++ Implementation
```cpp
#include <iostream>
#include <vector>

using namespace std;

const int INF = 1e9;

void floydWarshall(int n, vector<vector<int>>& graph) {
    vector<vector<int>> dist = graph;

    for (int k = 0; k < n; k++) {
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (dist[i][k] != INF && dist[k][j] != INF && dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }

    // Print results
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            if (dist[i][j] == INF) cout << "INF ";
            else cout << dist[i][j] << " ";
        }
        cout << endl;
    }
}
```

#### Step-by-Step Code Walkthrough
1. **Adjacency Matrix:** Starts with a distance matrix `dist` where `dist[i][j]` is the weight of the edge `(i, j)`.
2. **Intermediate Nodes ($k$):** The outermost loop iterates through each vertex `k`. We ask: "Can we find a shorter path from $i$ to $j$ by going through $k$?"
3. **Triply Nested Loop:**
    - `k`: Intermediate vertex.
    - `i`: Source vertex.
    - `j`: Destination vertex.
4. **The Update Rule:** `dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])`. If the path $i \to k \to j$ is shorter than the direct $i \to j$, we update the matrix.
5. **Handling INF:** We must check `dist[i][k] != INF` to avoid adding to infinity and causing overflow or logic errors.
6. **Result:** After $n$ iterations of $k$, `dist[i][j]` contains the absolute shortest distance between any two vertices.

#### Pseudocode
```text
FLOYD-WARSHALL(graph):
    dist = copy of graph matrix (where graph[i][j] is weight of edge (i,j))
    n = number of vertices

    for k = 0 to n - 1:
        for i = 0 to n - 1:
            for j = 0 to n - 1:
                if dist[i][k] != INF and dist[k][j] != INF:
                    dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])
                    
    return dist
```

#### Recursive Analysis
- **Recursive Calls:** The core logic is based on dynamic programming, which has a recursive relation: $d_{ij}^{(k)} = \min(d_{ij}^{(k-1)}, d_{ik}^{(k-1)} + d_{kj}^{(k-1)})$.
- **Recursion Frequency:** $n^3$ state transitions.
- **Recursive Alternatives for Loops:**
    - The triple nested loop can be converted to nested recursive calls: `solve(k, i, j)`.
    - `solve(k, i, j)` would call `solve(k-1, i, j)`, `solve(k-1, i, k)`, and `solve(k-1, k, j)`.

#### Complexity Analysis
- **Loops:**
    - Triple nested loops: Outer loop iterates over intermediate vertices $k$ ($V$ times), middle loop over sources $i$ ($V$ times), and inner loop over destinations $j$ ($V$ times).
- **Overall Complexity:**
    - **Best Case:** $O(V^3)$ — The algorithm consists of three static, unconditional loops that must execute fully to compute all paths, regardless of input configurations.
    - **Average Case:** $O(V^3)$ — Expected time over any standard input graph.
    - **Worst Case:** $O(V^3)$ — Worst-case execution matches the best case, running exactly $V^3$ basic matrix update steps.
- **Space Complexity:** $O(V^2)$ — Storing the $V \times V$ dynamic programming table (`dist`) representing the shortest path distances between all vertex pairs.

#### Example
Graph Matrix (3x3):
```
0   5   INF
INF 0   3
1   INF 0
```
- $k=0$ (Intermediate vertex 1): No changes.
- $k=1$ (Intermediate vertex 2): $dist[1][3] = \min(\text{INF}, 5+3) = 8$. Matrix becomes:
```
0   5   8
INF 0   3
1   INF 0
```
- $k=2$ (Intermediate vertex 3): $dist[2][1] = \min(\text{INF}, 3+1) = 4$. $dist[1][1] = \min(0, 8+1) = 0$.
Final shortest path matrix found.



## Graph Traversal & Ordering Algorithms

### 1. Breadth-First Search (BFS)

BFS explores nodes layer-by-layer starting from a source vertex. It is ideal for finding the shortest path in unweighted graphs.

#### C++ Implementation
```cpp
#include <iostream>
#include <vector>
#include <queue>

using namespace std;

void bfs(int start, int n, vector<vector<int>>& adj) {
    vector<bool> visited(n, false);
    queue<int> q;

    visited[start] = true;
    q.push(start);

    while (!q.empty()) {
        int u = q.front();
        q.pop();
        cout << u << " ";

        for (int v : adj[u]) {
            if (!visited[v]) {
                visited[v] = true;
                q.push(v);
            }
        }
    }
    cout << endl;
}
```

#### Step-by-Step Code Walkthrough
1. **Queue & Visited Setup:** Initialize a queue `q` to store node frontiers and a boolean vector `visited` to prevent duplicate visits.
2. **Push Start:** Mark the start node as visited and push it into the queue.
3. **Dequeue & Print:** Pop the front element `u` and process/print it.
4. **Queue Neighbors:** For each unvisited neighbor `v` of `u`, mark `v` as visited and enqueue it.
5. **Level-Order:** This ensures nodes at distance $d$ are processed before nodes at distance $d+1$.

#### Pseudocode
```text
BFS(G, start):
    visited = array of size V filled with False
    Q = empty queue
    
    visited[start] = True
    ENQUEUE(Q, start)

    while Q is not empty:
        u = DEQUEUE(Q)
        process u

        for each neighbor v of u:
            if visited[v] is False:
                visited[v] = True
                ENQUEUE(Q, v)
```

#### Recursive Analysis
- **Recursive Calls:** BFS is iterative due to the FIFO queue. A recursive alternative passes the queue to recursive steps: `bfsRecursive(adj, q, visited)`.
- **Recursion Frequency:** Will trigger $V$ calls to completely traverse the graph vertices.
- **Recursive Alternatives for Loops:**
  - `while (!q.empty())`: Replaced by `processQueue(q) { if(!q.empty()) { ... processQueue(q); } }`.
  - `for (int v : adj[u])`: Replaced by a helper function `visitNeighbors(u, index)`.

#### Complexity Analysis
- **Loops:**
  - Queue loop: Dequeues and processes each of the $V$ reachable vertices exactly once.
  - Neighbor exploration loop: Runs over the adjacency list of each vertex, visiting all $E$ edges in total (or $2E$ for undirected graphs).
- **Overall Complexity:**
  - **Best Case:** $O(V + E)$ — When all nodes and edges are reachable and traversed.
  - **Average Case:** $O(V + E)$ — Traversal time over typical graphs where nodes are distributed.
  - **Worst Case:** $O(V + E)$ — Occurs when the graph is fully connected; all vertices and edges must be checked to complete the search.
- **Space Complexity:** $O(V)$ — Required for the queue tracking the search frontier (which can contain up to $V$ nodes in the worst case) and the `visited` boolean tracking array.

---

### 2. Depth-First Search (DFS)

DFS explores as deep as possible along each branch before backtracking. It is useful for connectivity and topological sorting.

#### C++ Implementation
```cpp
#include <iostream>
#include <vector>

using namespace std;

void dfsRecursive(int u, vector<vector<int>>& adj, vector<bool>& visited) {
    visited[u] = true;
    cout << u << " ";

    for (int v : adj[u]) {
        if (!visited[v]) {
            dfsRecursive(v, adj, visited);
        }
    }
}

void dfs(int n, vector<vector<int>>& adj) {
    vector<bool> visited(n, false);
    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            dfsRecursive(i, adj, visited);
        }
    }
    cout << endl;
}
```

#### Step-by-Step Code Walkthrough
1. **Recursive Visit:** Mark node `u` as visited and print it.
2. **Deep Descent:** Loop through neighbors `v` of `u`. If a neighbor is unvisited, immediately recurse by calling `dfsRecursive(v, ...)`.
3. **Graph Scanning:** The main `dfs` loop scans all indices to handle disconnected graph structures.
4. **Backtracking:** Once all neighbors of a node are explored, the recursion returns (backtracks) to the caller.

#### Pseudocode
```text
DFS(G):
    visited = array of size V filled with False
    for each vertex u in G.V:
        if visited[u] is False:
            DFS-VISIT(G, u, visited)

DFS-VISIT(G, u, visited):
    visited[u] = True
    process u
    
    for each neighbor v of u:
        if visited[v] is False:
            DFS-VISIT(G, v, visited)
```

#### Recursive Analysis
- **Recursive Calls:** `dfsRecursive(v, ...)` is called recursively from inside the adjacency list traversal.
- **Recursion Frequency:** Recursion occurs $V$ times (once per vertex).
- **Recursive Alternatives for Loops:**
  - `for (int i = 0; i < n; i++)`: Replaced by `runDFS(index) { if (index < n) { if(!visited[index]) dfsRecursive(index); runDFS(index+1); } }`.
  - `for (int v : adj[u])`: Replaced by `exploreNeighbors(u, index)`.

#### Complexity Analysis
- **Loops:**
  - Component search loop: Runs $V$ times to check each vertex for unvisited components.
  - Neighbor traversal loop: Traverses the adjacency list for each vertex, visiting all $E$ edges.
- **Overall Complexity:**
  - **Best Case:** $O(V + E)$ — Standard traversal over all connected nodes and edges.
  - **Average Case:** $O(V + E)$ — Expected behavior for graph connectivity scans.
  - **Worst Case:** $O(V + E)$ — Traversal through dense graphs where every vertex and edge must be processed.
- **Space Complexity:** $O(V)$ — Storing the `visited` array ($O(V)$) and the recursive call stack space, which can reach $O(V)$ depth in the worst case (e.g., a linear graph structure).

---

### 3. Topological Sorting

Topological Sort finds a linear ordering of vertices in a Directed Acyclic Graph (DAG) such that for every directed edge $u \to v$, $u$ comes before $v$.

#### C++ Implementation (Kahn's Algorithm)
```cpp
#include <iostream>
#include <vector>
#include <queue>

using namespace std;

vector<int> topologicalSort(int n, vector<vector<int>>& adj) {
    vector<int> inDegree(n, 0);
    for (int u = 0; u < n; u++) {
        for (int v : adj[u]) {
            inDegree[v]++;
        }
    }

    queue<int> q;
    for (int i = 0; i < n; i++) {
        if (inDegree[i] == 0) {
            q.push(i);
        }
    }

    vector<int> order;
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        order.push_back(u);

        for (int v : adj[u]) {
            inDegree[v]--;
            if (inDegree[v] == 0) {
                q.push(v);
            }
        }
    }

    if (order.size() != n) {
        return {}; // Cycle detected
    }
    return order;
}
```

#### Step-by-Step Code Walkthrough
1. **In-Degree Calculation:** Count incoming edges for every vertex.
2. **Initial Frontier:** Insert all vertices with in-degree 0 into the queue.
3. **Extract & Append:** Pop `u` from queue and append to order list.
4. **Relax Indegrees:** For each neighbor `v` of `u`, decrement its `inDegree`. If it hits 0, add it to the queue.
5. **Cycle Detection Check:** If the list size is less than $N$, it means a cycle exists, making topological order impossible.

#### Pseudocode
```text
TOPOLOGICAL-SORT-KAHN(G):
    in_degree = array of size V filled with 0
    for each vertex u in G.V:
        for each neighbor v of u:
            in_degree[v] = in_degree[v] + 1
            
    Q = empty queue
    for i = 0 to V - 1:
        if in_degree[i] == 0:
            ENQUEUE(Q, i)
            
    order = empty list
    while Q is not empty:
        u = DEQUEUE(Q)
        add u to order
        for each neighbor v of u:
            in_degree[v] = in_degree[v] - 1
            if in_degree[v] == 0:
                ENQUEUE(Q, v)
                
    if size of order != V:
        return "Cycle detected, no topological sort exists"
    return order
```

#### Recursive Analysis
- **Recursive Calls:** Kahn's is iterative. A DFS-based version utilizes post-order recursion to build ordering.
- **Recursion Frequency:** In DFS-based topological sort, recursion runs $V$ times.
- **Recursive Alternatives for Loops:**
  - `while (!q.empty())`: Replaced by recursive queue draining.
  - Neighbor updates: Replaced by recursion over neighbor index.

#### Complexity Analysis
- **Loops:**
  - Indegree setup loop: $O(V + E)$ to calculate the initial in-degree of all vertices by traversing the graph.
  - Main loop: Runs $V$ times to process each vertex.
  - Decrement loop: Runs $E$ times in total to decrement the in-degree of neighbors.
- **Overall Complexity:**
  - **Best Case:** $O(V + E)$ — Processes all dependencies in a Directed Acyclic Graph (DAG) in linear time.
  - **Average Case:** $O(V + E)$ — Expected sorting time across DAG structures.
  - **Worst Case:** $O(V + E)$ — Runs to completion or detects a cycle, checking all vertices and edges.
- **Space Complexity:** $O(V)$ — Storing the `inDegree` array ($O(V)$), the queue ($O(V)$), and the final sorted `order` array ($O(V)$).

---

## Sorting Algorithms

### Sorting Concepts: In-place Sort vs. Stable Sort

* **In-place Sort:** A sorting algorithm is **in-place** if it does not require auxiliary memory that scales with the size of the input. It uses a constant amount of extra memory space ($O(1)$) to perform swaps directly inside the input array.
  - *In-place:* Bubble Sort, Selection Sort, Insertion Sort, Quick Sort, Heap Sort.
  - *Out-of-place:* Merge Sort (needs $O(N)$ memory), Counting Sort (needs $O(N+K)$ memory).
* **Stable Sort:** A sorting algorithm is **stable** if it preserves the relative original sequence of elements that possess equal keys/values. If element $X$ appears before element $Y$ in the input, and $X = Y$, a stable sort guarantees $X$ will remain before $Y$ in the sorted output.
  - *Stable:* Bubble Sort, Insertion Sort, Merge Sort, Counting Sort.
  - *Unstable:* Selection Sort, Quick Sort, Heap Sort.

---

### 1. Bubble Sort

Bubble Sort works by repeatedly swapping adjacent elements if they are in the wrong order, "bubbling" the largest element to the end.

#### C++ Implementation
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

void bubbleSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; i++) {
        bool swapped = false;
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr[j], arr[j + 1]);
                swapped = true;
            }
        }
        if (!swapped) break; // Early exit optimization
    }
}
```

#### Step-by-Step Code Walkthrough
1. **Passes Loop:** The outer loop executes $N-1$ times.
2. **Scan & Compare:** The inner loop compares adjacent items.
3. **Swap Element:** If `arr[j] > arr[j+1]`, they swap.
4. **Exit Check:** If no swaps happen during a pass, the array is sorted; break immediately.

#### Pseudocode
```text
BUBBLE-SORT(A):
    n = A.length
    for i = 0 to n - 2:
        swapped = false
        for j = 0 to n - i - 2:
            if A[j] > A[j+1]:
                swap A[j] with A[j+1]
                swapped = true
        if swapped == false:
            break
```

#### Recursive Analysis
- **Recursive Calls:** A recursive version `bubbleSortRec(arr, n)` swaps elements on a pass and calls itself with `n-1`.
- **Recursion Frequency:** Runs $N$ times.
- **Recursive Alternatives for Loops:**
  - Outer loop: Replaced by recursion decrementing boundaries.
  - Inner loop: Replaced by recursive traversal `swapPass(index)`.

#### Complexity Analysis
- **Loops:**
  - Outer loop: Performs up to $N-1$ passes over the array.
  - Inner loop: Compares adjacent elements, running $N-i-1$ times for the $i$-th pass.
- **Overall Complexity:**
  - **Best Case:** $O(N)$ — Occurs when the input array is already sorted. The inner loop makes $N-1$ comparisons, detects that no swaps were made, and triggers the early exit optimization.
  - **Average Case:** $O(N^2)$ — Expected execution for randomly ordered arrays, requiring $\approx \frac{N(N-1)}{4}$ comparisons and swaps.
  - **Worst Case:** $O(N^2)$ — Occurs when the array is sorted in reverse order, requiring $\frac{N(N-1)}{2}$ comparisons and swaps as elements must bubble all the way to their destination.
- **Space Complexity:** $O(1)$ — Performs sorting entirely in-place, requiring only a single boolean flag `swapped` and temporary swap variables.

---

### 2. Selection Sort

Selection Sort divides the array into sorted and unsorted regions, repeatedly finding the minimum element from the unsorted region and putting it at the beginning.

#### C++ Implementation
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

void selectionSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; i++) {
        int minIndex = i;
        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIndex]) {
                minIndex = j;
            }
        }
        if (minIndex != i) {
            swap(arr[i], arr[minIndex]);
        }
    }
}
```

#### Step-by-Step Code Walkthrough
1. **Starting Boundary:** The outer loop selects position `i` where the next minimum element belongs.
2. **Scan Unsorted:** The inner loop finds the index of the minimum element in `arr[i...n-1]`.
3. **Swap Minimum:** The minimum element is swapped with the element at index `i`.
4. **Static Performance:** Always performs all comparisons regardless of array state.

#### Pseudocode
```text
SELECTION-SORT(A):
    n = A.length
    for i = 0 to n - 2:
        minIndex = i
        for j = i + 1 to n - 1:
            if A[j] < A[minIndex]:
                minIndex = j
        if minIndex != i:
            swap A[i] with A[minIndex]
```

#### Recursive Analysis
- **Recursive Calls:** `selectionSortRec(arr, index, n)` finds minimum, swaps it, and calls itself with `index+1`.
- **Recursion Frequency:** Runs $N-1$ times.
- **Recursive Alternatives for Loops:**
  - Outer loop: Replaced by incrementing index recursion.
  - Inner loop: Replaced by recursive minimum lookup.

#### Complexity Analysis
- **Loops:**
  - Outer loop: Runs $N-1$ times to set the boundary for the sorted sub-array.
  - Inner loop: Scans the remaining $N-i-1$ unsorted elements to find the index of the minimum element.
- **Overall Complexity:**
  - **Best Case:** $O(N^2)$ — Even if the array is already sorted, the algorithm has no early exit and always scans the entire unsorted region, performing $\frac{N(N-1)}{2}$ comparisons.
  - **Average Case:** $O(N^2)$ — Expected execution time for randomly ordered inputs.
  - **Worst Case:** $O(N^2)$ — Same execution count as the best case, performing $O(N^2)$ comparisons.
- **Space Complexity:** $O(1)$ — Performs sorting in-place, using only variable track markers for indices.

---

### 3. Insertion Sort

Insertion Sort builds the final sorted array one item at a time, moving elements that are larger than the current "key" to the right.

#### C++ Implementation
```cpp
#include <iostream>
#include <vector>

using namespace std;

void insertionSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 1; i < n; i++) {
        int key = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}
```

#### Step-by-Step Code Walkthrough
1. **Choose Key:** The outer loop chooses `arr[i]` as the key to insert.
2. **Shifting Elements:** The inner loop compares `key` with sorted elements on its left and shifts them right.
3. **Insertion Point:** Insert the key into its correct sorted location.
4. **Adaptive sorting:** Performs highly on nearly sorted arrays.

#### Pseudocode
```text
INSERTION-SORT(A):
    n = A.length
    for i = 1 to n - 1:
        key = A[i]
        j = i - 1
        while j >= 0 and A[j] > key:
            A[j+1] = A[j]
            j = j - 1
        A[j+1] = key
```

#### Recursive Analysis
- **Recursive Calls:** `insertionSortRec(arr, n)` recursively sorts $n-1$ elements and inserts the last.
- **Recursion Frequency:** Runs $N$ times.
- **Recursive Alternatives for Loops:**
  - Outer loop: Replaced by recursive function calls.
  - Inner loop: Replaced by recursive shifts.

#### Complexity Analysis
- **Loops:**
  - Outer loop: Iterates $N-1$ times, choosing `arr[i]` as the key to insert into the sorted region.
  - Inner loop: Shifts elements in the sorted region `arr[0...i-1]` that are larger than the key to the right.
- **Overall Complexity:**
  - **Best Case:** $O(N)$ — Occurs when the array is already sorted. The inner loop condition `arr[j] > key` fails immediately on the first comparison for every outer iteration, resulting in only $N-1$ comparisons and 0 shifts.
  - **Average Case:** $O(N^2)$ — Expected complexity on random lists where on average we must shift half of the elements in the sorted sub-array.
  - **Worst Case:** $O(N^2)$ — Occurs when the input is sorted in reverse order. For each element at index $i$, we must shift all $i$ elements in the sorted sub-array to insert it at index 0, performing $\frac{N(N-1)}{2}$ shifts.
- **Space Complexity:** $O(1)$ — Performs sorting entirely in-place by shifting elements within the input array.

---

### 4. Merge Sort

Merge Sort is a stable, divide-and-conquer algorithm that splits the array in halves, recursively sorts them, and merges them.

#### C++ Implementation
```cpp
#include <iostream>
#include <vector>

using namespace std;

void merge(vector<int>& arr, int l, int m, int r) {
    int n1 = m - l + 1;
    int n2 = r - m;
    vector<int> L(n1), R(n2);

    for (int i = 0; i < n1; i++) L[i] = arr[l + i];
    for (int j = 0; j < n2; j++) R[j] = arr[m + 1 + j];

    int i = 0, j = 0, k = l;
    while (i < n1 && j < n2) {
        if (L[i] <= R[j]) {
            arr[k] = L[i];
            i++;
        } else {
            arr[k] = R[j];
            j++;
        }
        k++;
    }

    while (i < n1) {
        arr[k] = L[i];
        i++; k++;
    }
    while (j < n2) {
        arr[k] = R[j];
        j++; k++;
    }
}

void mergeSort(vector<int>& arr, int l, int r) {
    if (l >= r) return;
    int m = l + (r - l) / 2;
    mergeSort(arr, l, m);
    mergeSort(arr, m + 1, r);
    merge(arr, l, m, r);
}
```

#### Step-by-Step Code Walkthrough
1. **Divide range:** Midpoint `m` divides range.
2. **Recursive Divide:** Recursively call `mergeSort` on left and right subranges.
3. **Merge Routine:** Copy values to temporary buffers `L` and `R`.
4. **Pointer Comparison:** Iteratively insert smallest available values back to `arr` stably.

#### Pseudocode
```text
MERGE-SORT(A, low, high):
    if low < high:
        mid = floor((low + high) / 2)
        MERGE-SORT(A, low, mid)
        MERGE-SORT(A, mid + 1, high)
        MERGE(A, low, mid, high)

MERGE(A, low, mid, high):
    left = slice of A from low to mid
    right = slice of A from mid + 1 to high
    
    i = 0, j = 0, k = low
    while i < left.length and j < right.length:
        if left[i] <= right[j]:
            A[k] = left[i]
            i = i + 1
        else:
            A[k] = right[j]
            j = j + 1
        k = k + 1
        
    while i < left.length:
        A[k] = left[i]
        i = i + 1
        k = k + 1
    while j < right.length:
        A[k] = right[j]
        j = j + 1
        k = k + 1
```

#### Recursive Analysis
- **Recursive Calls:** `mergeSort(arr, l, m)` and `mergeSort(arr, m+1, r)`.
- **Recursion Frequency:** Calls occur $2N-1$ times.
- **Recursive Alternatives for Loops:**
  - Copy loops: Replaced by recursion indexing.
  - Merge loops: Replaced by recursive comparison pointers.

#### Complexity Analysis
- **Loops:**
  - Array copy loop: $O(N)$ total time spent copying elements into temporary `L` and `R` arrays.
  - Array merge pointer updates: $O(N)$ total comparisons and pointer steps to merge the sub-arrays.
- **Overall Complexity:**
  - **Best Case:** $O(N \log N)$ — The array is split into two halves recursively ($\log N$ levels of recursion), and each level requires $O(N)$ time to merge, regardless of element order.
  - **Average Case:** $O(N \log N)$ — Expected behavior over random configurations.
  - **Worst Case:** $O(N \log N)$ — Splitting and merging execution paths are independent of initial array state.
- **Space Complexity:** $O(N)$ — Required to allocate temporary helper arrays `L` and `R` of size $N$ during the merge steps. (Also requires $O(\log N)$ stack space for the recursion).

---

### 5. Counting Sort

Counting Sort is a non-comparison sorting algorithm that counts the frequency of each element and computes prefix sums to place elements directly in their sorted output indices.

#### C++ Implementation
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

void countingSort(vector<int>& arr) {
    if (arr.empty()) return;
    int maxVal = *max_element(arr.begin(), arr.end());
    int minVal = *min_element(arr.begin(), arr.end());
    int range = maxVal - minVal + 1;

    vector<int> count(range, 0);
    vector<int> output(arr.size());

    for (int x : arr) {
        count[x - minVal]++;
    }

    for (int i = 1; i < range; i++) {
        count[i] += count[i - 1];
    }

    for (int i = arr.size() - 1; i >= 0; i--) {
        output[count[arr[i] - minVal] - 1] = arr[i];
        count[arr[i] - minVal]--;
    }

    arr = output;
}
```

#### Step-by-Step Code Walkthrough
1. **Find Bounds:** Determine the range size using `min` and `max`.
2. **Frequency Tally:** Populate the frequency counts array.
3. **Cumulative Sums:** Convert counts to cumulative indices (prefix sums).
4. **Stable Reassembly:** Scan input right-to-left. Match element values to their target counts, place them in the output array, and decrement counts.

#### Pseudocode
```text
COUNTING-SORT(A):
    max_val = maximum element in A
    min_val = minimum element in A
    range = max_val - min_val + 1
    count = array of size range initialized to 0
    output = array of same size as A
    
    for each num in A:
        count[num - min_val] = count[num - min_val] + 1
        
    for i = 1 to range - 1:
        count[i] = count[i] + count[i - 1]
        
    for j = A.length - 1 downto 0:
        val = A[j]
        output[count[val - min_val] - 1] = val
        count[val - min_val] = count[val - min_val] - 1
        
    copy output to A
```

#### Recursive Analysis
- **Recursive Calls:** Counting sort is inherently iterative. Loop logic can be replaced by tail recursion routines if needed.
- **Recursion Frequency:** 0 calls in standard versions.
- **Recursive Alternatives for Loops:**
  - Count: Replaced by recursion indexing.
  - Prefix Sum: Replaced by recursive addition steps.

#### Complexity Analysis
- **Loops:**
  - Count frequency: Iterates through the input array of size $N$ once to build frequency counts.
  - Prefix sum: Iterates through the `count` array of size $K$ to compute prefix sums.
  - Build output: Scans the input array of size $N$ from right to left to construct the sorted array.
- **Overall Complexity:**
  - **Best Case:** $O(N + K)$ — The algorithm performs fixed sequential passes over the input array and counting buckets.
  - **Average Case:** $O(N + K)$ — Expected complexity over all random distributions of inputs.
  - **Worst Case:** $O(N + K)$ — The steps executed are entirely determined by input size $N$ and range $K$, running independently of element values.
- **Space Complexity:** $O(N + K)$ — Requires $O(K)$ auxiliary space for the `count` frequency array and $O(N)$ auxiliary space for the `output` array.

---

### 6. Radix Sort

Radix Sort sorts elements digit-by-digit from the least significant digit (LSD) to the most significant digit (MSD) using a stable sort (like Counting Sort) as a subroutine.

#### C++ Implementation
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

void countingSortForRadix(vector<int>& arr, int exp) {
    int n = arr.size();
    vector<int> output(n);
    vector<int> count(10, 0);

    for (int i = 0; i < n; i++) {
        count[(arr[i] / exp) % 10]++;
    }

    for (int i = 1; i < 10; i++) {
        count[i] += count[i - 1];
    }

    for (int i = n - 1; i >= 0; i--) {
        int digit = (arr[i] / exp) % 10;
        output[count[digit] - 1] = arr[i];
        count[digit]--;
    }

    arr = output;
}

void radixSort(vector<int>& arr) {
    if (arr.empty()) return;
    int maxVal = *max_element(arr.begin(), arr.end());

    for (int exp = 1; maxVal / exp > 0; exp *= 10) {
        countingSortForRadix(arr, exp);
    }
}
```

#### Step-by-Step Code Walkthrough
1. **Find maximum value:** This dictates the number of digits.
2. **Iterate Exponent:** Multiply `exp` by 10 in each pass to isolate digits (units, tens, hundreds...).
3. **Subroutine Sort:** Invoke Counting Sort matching on the isolated digit values.
4. **Stable Progression:** The stable subroutine preserves prior sorts.

#### Pseudocode
```text
RADIX-SORT(A, d):
    for i = 1 to d:
        use a stable sort to sort array A on digit i
```

#### Recursive Analysis
- **Recursive Calls:** The main loop over digits can be replaced by `radixSortRec(arr, exp, maxVal)` which calls itself with `exp * 10`.
- **Recursion Frequency:** Runs $D$ times where $D$ is the number of digits in the maximum value.
- **Recursive Alternatives for Loops:**
  - Digits loop: Replaced by digit-by-digit recursive step-down.

#### Complexity Analysis
- **Loops:**
  - Digit loop: Iterates $D$ times (where $D$ is the number of digits in the maximum element).
  - Subroutine sort: Executes Counting Sort stably in each digit pass, taking $O(N + K)$ time (where base $K = 10$ for decimal numbers).
- **Overall Complexity:**
  - **Best Case:** $O(D \cdot (N + K))$ — The algorithm performs fixed Counting Sort passes for all $D$ digit positions.
  - **Average Case:** $O(D \cdot (N + K))$ — Standard execution time over average data.
  - **Worst Case:** $O(D \cdot (N + K))$ — Determined purely by the number of digits $D$, value range base $K$, and array size $N$, regardless of the sorting state of the elements.
- **Space Complexity:** $O(N + K)$ — Requires $O(N)$ space for the dynamic output buffer and $O(K)$ space for digit frequency counts during counting sort subroutine execution.

---

## Dynamic Programming

For basic concepts, memoization vs tabulation, standard problems (Frog Stairs, Frog K-Steps), and solved questions (Minimum Coins, Friends Pairing), refer to the dedicated **[[Dynamic programming/Dynamic programming|Dynamic Programming Notes]]** folder.
