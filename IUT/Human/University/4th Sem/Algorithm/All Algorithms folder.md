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
PRIM(G, r):
    for each u in G.V:
        u.key = INF
        u.parent = NIL
    r.key = 0
    Q = G.V (priority queue)
    while Q is not empty:
        u = EXTRACT-MIN(Q)
        for each v in G.adj[u]:
            if v in Q and w(u, v) < v.key:
                v.parent = u
                v.key = w(u, v)
```

#### Recursive Analysis
- **Recursive Calls:** Standard implementation is iterative. A recursive version `primRecursive(u)` would visit neighbors and then call itself for the next closest vertex.
- **Recursion Frequency:** Would occur $V$ times (once for each vertex).
- **Recursive Alternatives for Loops:**
    - `while (!pq.empty())`: Can be replaced by `processPQ() { if !pq.empty(): ... processPQ() }`.
    - `for (auto& edge : adj[u])`: Can be replaced by `visitNeighbors(index)`.

#### Complexity Analysis
- **Loops:**
    - Main Loop: $V$ iterations.
    - Priority Queue operations: $O(E \log V)$ or $O(E \log E)$.
- **Overall Complexity:**
    - **Best Case:** $O((V + E) \log V)$
    - **Average Case:** $O((V + E) \log V)$
    - **Worst Case:** $O((V + E) \log V)$
- **Space Complexity:** $O(V + E)$ for adjacency list and priority queue.

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
    A = empty set
    for each vertex v in G.V:
        MAKE-SET(v)
    sort the edges of G.E into non-decreasing order by weight w
    for each edge (u, v) in G.E, taken in non-decreasing order by weight:
        if FIND-SET(u) != FIND-SET(v):
            A = A union {(u, v)}
            UNION(u, v)
    return A
```

#### Recursive Analysis
- **Recursive Calls:** `find(i)` is typically recursive with path compression.
- **Recursion Frequency:** Number of calls depends on the tree height; path compression makes it nearly constant $O(\alpha(V))$.
- **Recursive Alternatives for Loops:**
    - `for (auto& edge : edges)`: `processEdges(index) { if (index < edges.size()) { ... processEdges(index+1); } }`.
    - `sort()`: Sorting algorithms like Merge Sort or Quick Sort are inherently recursive.

#### Complexity Analysis
- **Loops:**
    - Sorting edges: $O(E \log E)$.
    - Main Loop: $E$ iterations.
    - DSU operations: $O(E \alpha(V))$, where $\alpha$ is the inverse Ackermann function.
- **Overall Complexity:**
    - **Best Case:** $O(E \log E)$.
    - **Average Case:** $O(E \log E)$.
    - **Worst Case:** $O(E \log E)$.
- **Space Complexity:** $O(V + E)$ to store edges and DSU parent array.

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
DIJKSTRA(G, w, s):
    INITIALIZE-SINGLE-SOURCE(G, s)
    S = empty set
    Q = G.V
    while Q is not empty:
        u = EXTRACT-MIN(Q)
        S = S union {u}
        for each vertex v in G.adj[u]:
            RELAX(u, v, w)
```

#### Recursive Analysis
- **Recursive Calls:** Standard Dijkstra is iterative. A recursive version would involve `dijkstraRecursive(pq)` where each call extracts one vertex and calls itself.
- **Recursion Frequency:** $V$ times.
- **Recursive Alternatives for Loops:**
    - `while (!pq.empty())`: `processDijkstra() { if (!pq.empty()) { ... processDijkstra(); } }`.
    - `for (auto& edge : adj[u])`: `relaxNeighbors(u, index) { if (index < adj[u].size()) { ... relaxNeighbors(u, index+1); } }`.

#### Complexity Analysis
- **Loops:**
    - Initialization: $O(V)$ Time.
    - While loop: $V$ iterations (extract min).
    - For loop: $E$ relaxations total across all iterations.
- **Overall Complexity:**
    - **Best Case:** $O(V + E \log V)$ or $O(E + V \log V)$ with Fibonacci heap.
    - **Average Case:** $O(E \log V)$.
    - **Worst Case:** $O(E \log V)$.
- **Space Complexity:** $O(V + E)$ for adjacency list and distance array.

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
```ps
BELLMAN-FORD(G, w, s):
    INITIALIZE-SINGLE-SOURCE(G, s)
    for i = 1 to |G.V| - 1:
        for each edge (u, v) in G.E:
            RELAX(u, v, w)
    for each edge (u, v) in G.E:
        if v.d > u.d + w(u, v):
            return FALSE
    return TRUE
```

#### Recursive Analysis
- **Recursive Calls:** Iterative by design. Could be implemented as `relaxAllEdges(iterationCount)` where it calls itself until `iterationCount == V-1`.
- **Recursion Frequency:** $V-1$ calls.
- **Recursive Alternatives for Loops:**
    - Outer loop: `iterateRelaxation(k) { if (k < V-1) { relaxAll(); iterateRelaxation(k+1); } }`.
    - Inner loop: `relaxEdgeList(index) { if (index < E) { relax(edges[index]); relaxEdgeList(index+1); } }`.

#### Complexity Analysis
- **Loops:**
    - Outer loop: Runs $V-1$ times.
    - Inner loop: Runs $E$ times per outer iteration.
- **Overall Complexity:**
    - **Best Case:** $O(E)$ (if no relaxation happens in first pass).
    - **Average Case:** $O(VE)$.
    - **Worst Case:** $O(VE)$.
- **Space Complexity:** $O(V)$ to store distances.

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
FLOYD-WARSHALL(W):
    n = W.rows
    D(0) = W
    for k = 1 to n:
        let D(k) = d_ij(k) be a new n x n matrix
        for i = 1 to n:
            for j = 1 to n:
                d_ij(k) = min(d_ij(k-1), d_ik(k-1) + d_kj(k-1))
    return D(n)
```

#### Recursive Analysis
- **Recursive Calls:** The core logic is based on dynamic programming, which has a recursive relation: $d_{ij}^{(k)} = \min(d_{ij}^{(k-1)}, d_{ik}^{(k-1)} + d_{kj}^{(k-1)})$.
- **Recursion Frequency:** $n^3$ state transitions.
- **Recursive Alternatives for Loops:**
    - The triple nested loop can be converted to nested recursive calls: `solve(k, i, j)`.
    - `solve(k, i, j)` would call `solve(k-1, i, j)`, `solve(k-1, i, k)`, and `solve(k-1, k, j)`.

#### Complexity Analysis
- **Loops:**
    - Triple nested loops, each running $n$ times.
- **Overall Complexity:**
    - **Best Case:** $O(V^3)$.
    - **Average Case:** $O(V^3)$.
    - **Worst Case:** $O(V^3)$.
- **Space Complexity:** $O(V^2)$ to store the distance matrix.

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
BFS(G, s):
    for each vertex u in G.V - {s}:
        u.color = WHITE
        u.d = INF
        u.p = NIL
    s.color = GRAY
    s.d = 0
    s.p = NIL
    Q = empty queue
    ENQUEUE(Q, s)
    while Q is not empty:
        u = DEQUEUE(Q)
        for each v in G.adj[u]:
            if v.color == WHITE:
                v.color = GRAY
                v.d = u.d + 1
                v.p = u
                ENQUEUE(Q, v)
        u.color = BLACK
```

#### Recursive Analysis
- **Recursive Calls:** BFS is iterative due to the FIFO queue. A recursive alternative passes the queue to recursive steps: `bfsRecursive(adj, q, visited)`.
- **Recursion Frequency:** Will trigger $V$ calls to completely traverse the graph vertices.
- **Recursive Alternatives for Loops:**
  - `while (!q.empty())`: Replaced by `processQueue(q) { if(!q.empty()) { ... processQueue(q); } }`.
  - `for (int v : adj[u])`: Replaced by a helper function `visitNeighbors(u, index)`.

#### Complexity Analysis
- **Loops:**
  - Queue loop: Runs $V$ times (once per vertex).
  - Neighbor exploration loop: Runs $E$ times (or $2E$ for undirected graph) total across all dequeue steps.
- **Overall Complexity:**
  - **Best Case:** $O(V + E)$
  - **Average Case:** $O(V + E)$
  - **Worst Case:** $O(V + E)$
- **Space Complexity:** $O(V)$ for queue and visited array.

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
    for each vertex u in G.V:
        u.color = WHITE
        u.p = NIL
    time = 0
    for each vertex u in G.V:
        if u.color == WHITE:
            DFS-VISIT(G, u)

DFS-VISIT(G, u):
    time = time + 1
    u.d = time
    u.color = GRAY
    for each v in G.adj[u]:
        if v.color == WHITE:
            v.p = u
            DFS-VISIT(G, v)
    u.color = BLACK
    time = time + 1
    u.f = time
```

#### Recursive Analysis
- **Recursive Calls:** `dfsRecursive(v, ...)` is called recursively from inside the adjacency list traversal.
- **Recursion Frequency:** Recursion occurs $V$ times (once per vertex).
- **Recursive Alternatives for Loops:**
  - `for (int i = 0; i < n; i++)`: Replaced by `runDFS(index) { if (index < n) { if(!visited[index]) dfsRecursive(index); runDFS(index+1); } }`.
  - `for (int v : adj[u])`: Replaced by `exploreNeighbors(u, index)`.

#### Complexity Analysis
- **Loops:**
  - Component search loop: Runs $V$ times.
  - Neighbor traversal loop: Runs $E$ times in total.
- **Overall Complexity:**
  - **Best Case:** $O(V + E)$
  - **Average Case:** $O(V + E)$
  - **Worst Case:** $O(V + E)$
- **Space Complexity:** $O(V)$ auxiliary space for recursive call stack.

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
        for each v in G.adj[u]:
            in_degree[v] = in_degree[v] + 1
    Q = empty queue
    for i = 0 to V - 1:
        if in_degree[i] == 0:
            ENQUEUE(Q, i)
    L = empty list
    while Q is not empty:
        u = DEQUEUE(Q)
        append u to L
        for each v in G.adj[u]:
            in_degree[v] = in_degree[v] - 1
            if in_degree[v] == 0:
                ENQUEUE(Q, v)
    if L.size != V:
        return error "Graph contains cycle"
    return L
```

#### Recursive Analysis
- **Recursive Calls:** Kahn's is iterative. A DFS-based version utilizes post-order recursion to build ordering.
- **Recursion Frequency:** In DFS-based topological sort, recursion runs $V$ times.
- **Recursive Alternatives for Loops:**
  - `while (!q.empty())`: Replaced by recursive queue draining.
  - Neighbor updates: Replaced by recursion over neighbor index.

#### Complexity Analysis
- **Loops:**
  - Indegree setup loop: $O(V + E)$
  - Main loop: $V$ iterations.
  - Decrement loop: $E$ total.
- **Overall Complexity:**
  - **Best Case:** $O(V + E)$
  - **Average Case:** $O(V + E)$
  - **Worst Case:** $O(V + E)$
- **Space Complexity:** $O(V)$ for indegrees array and queue.

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
  - Outer loop: Runs $N-1$ times.
  - Inner loop: Runs $N-i-1$ times.
- **Overall Complexity:**
  - **Best Case:** $O(N)$ (Already sorted list).
  - **Average Case:** $O(N^2)$
  - **Worst Case:** $O(N^2)$ (Reverse sorted list).
- **Space Complexity:** $O(1)$ auxiliary space.

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
  - Outer loop: Runs $N-1$ times.
  - Inner loop: Runs $N - i - 1$ times.
- **Overall Complexity:**
  - **Best Case:** $O(N^2)$
  - **Average Case:** $O(N^2)$
  - **Worst Case:** $O(N^2)$
- **Space Complexity:** $O(1)$ auxiliary space.

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
  - Outer loop: Runs $N-1$ times.
  - Inner loop: Runs up to $i$ times.
- **Overall Complexity:**
  - **Best Case:** $O(N)$ (Already sorted inputs).
  - **Average Case:** $O(N^2)$
  - **Worst Case:** $O(N^2)$ (Reverse sorted inputs).
- **Space Complexity:** $O(1)$ auxiliary space.

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
MERGE-SORT(A, p, r):
    if p < r:
        q = floor((p + r) / 2)
        MERGE-SORT(A, p, q)
        MERGE-SORT(A, q + 1, r)
        MERGE(A, p, q, r)

MERGE(A, p, q, r):
    n1 = q - p + 1
    n2 = r - q
    let L[0..n1-1] and R[0..n2-1] be new arrays
    for i = 0 to n1 - 1:
        L[i] = A[p + i]
    for j = 0 to n2 - 1:
        R[j] = A[q + 1 + j]
    i = 0, j = 0, k = p
    while i < n1 and j < n2:
        if L[i] <= R[j]:
            A[k] = L[i]
            i = i + 1
        else:
            A[k] = R[j]
            j = j + 1
        k = k + 1
    while i < n1:
        A[k] = L[i]
        i = i + 1
        k = k + 1
    while j < n2:
        A[k] = R[j]
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
  - Array copy loop: $O(N)$
  - Array merge pointer updates: $O(N)$
- **Overall Complexity:**
  - **Best Case:** $O(N \log N)$
  - **Average Case:** $O(N \log N)$
  - **Worst Case:** $O(N \log N)$
- **Space Complexity:** $O(N)$ auxiliary space for buffers.

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
COUNTING-SORT(A, B, k):
    let C[0..k] be a new array
    for i = 0 to k:
        C[i] = 0
    for j = 0 to A.length - 1:
        C[A[j]] = C[A[j]] + 1
    for i = 1 to k:
        C[i] = C[i] + C[i - 1]
    for j = A.length - 1 downto 0:
        B[C[A[j]] - 1] = A[j]
        C[A[j]] = C[A[j]] - 1
```

#### Recursive Analysis
- **Recursive Calls:** Counting sort is inherently iterative. Loop logic can be replaced by tail recursion routines if needed.
- **Recursion Frequency:** 0 calls in standard versions.
- **Recursive Alternatives for Loops:**
  - Count: Replaced by recursion indexing.
  - Prefix Sum: Replaced by recursive addition steps.

#### Complexity Analysis
- **Loops:**
  - Count frequency: Runs $N$ times.
  - Prefix sum: Runs $K$ times.
  - Build output: Runs $N$ times.
- **Overall Complexity:**
  - **Best Case:** $O(N + K)$
  - **Average Case:** $O(N + K)$
  - **Worst Case:** $O(N + K)$
- **Space Complexity:** $O(N + K)$ space for count and output arrays.

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
  - Digit loop: Runs $D$ times.
  - Subroutine sort: $O(N + K)$ per iteration (where $K = 10$).
- **Overall Complexity:**
  - **Best Case:** $O(D \cdot (N + K))$
  - **Average Case:** $O(D \cdot (N + K))$
  - **Worst Case:** $O(D \cdot (N + K))$
- **Space Complexity:** $O(N + K)$ space for counting sort helper arrays.

---

## Dynamic Programming

For basic concepts, memoization vs tabulation, standard problems (Frog Stairs, Frog K-Steps), and solved questions (Minimum Coins, Friends Pairing), refer to the dedicated **[[Dynamic programming/Dynamic programming|Dynamic Programming Notes]]** folder.
