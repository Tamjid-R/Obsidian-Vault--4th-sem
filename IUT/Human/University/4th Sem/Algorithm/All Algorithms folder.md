# All Algorithms

This file serves as a central repository for all algorithms studied in the 3rd Semester Algorithm course. Each entry includes C++ implementation, pseudocode, recursion analysis, and complexity details.

## Algorithm Summary Table

| Algorithm Name     | Best Case Time    | Average Case Time | Worst Case Time   | Space Complexity | Use Case                                               | Keywords                                                                     |
| :----------------- | :---------------- | :---------------- | :---------------- | :--------------- | :----------------------------------------------------- | :--------------------------------------------------------------------------- |
| **Prim's**         | $O(E \log V)$     | $O((V+E) \log V)$ | $O((V+E) \log V)$ | $O(V + E)$       | Find MST in dense graphs; grows tree from a node.      | "Minimum weight spanning tree", "Priority queue", "Connected nodes".         |
| **Dijkstra's**     | $O(V + E \log V)$ | $O(E \log V)$     | $O(E \log V)$     | $O(V + E)$       | Single-source shortest path with non-negative weights. | "Shortest path", "Non-negative weights", "Quickest route".                   |
| **Bellman-Ford**   | $O(E)$            | $O(VE)$           | $O(VE)$           | $O(V)$           | Single-source shortest path with negative weights.     | "Negative weights", "Negative cycle detection", "Relax all edges".           |
| **Floyd-Warshall** | $O(V^3)$          | $O(V^3)$          | $O(V^3)$          | $O(V^2)$         | All-pairs shortest paths in small to medium graphs.    | "All pairs shortest path", "Intermediate vertices", "Matrix representation". |
| **Kruskal's**      | $O(E \log E)$     | $O(E \log E)$     | $O(E \log E)$     | $O(V + E)$       | Find MST in sparse graphs; sorts edges and adds them.  | "Minimum weight spanning tree", "Edge list", "DSU", "Sort edges".            |

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
    - **Best Case:** $O(E \log V)$.
    - **Average Case:** $O(E \log V)$.
    - **Worst Case:** $O(E \log V)$.
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

## Floyd-Warshall Algorithm

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

---
