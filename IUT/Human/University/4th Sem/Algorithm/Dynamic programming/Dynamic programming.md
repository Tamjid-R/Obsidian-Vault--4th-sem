# Dynamic Programming (DP)

Dynamic Programming is a powerful algorithmic paradigm used to solve optimization and counting problems by breaking them down into simpler, overlapping subproblems, solving each subproblem exactly once, and storing their solutions to avoid redundant calculations.

---

## 1. What is DP and How It Works?

Dynamic Programming works on the principle of **caching (storing)** solutions to subproblems so that when the same subproblems arise again, we do not have to recompute them.

### The Two Necessary Ingredients for DP
For a problem to be solvable using Dynamic Programming, it must satisfy two key properties:

1. **Optimal Substructure**: An optimal solution to the overall problem can be constructed efficiently from optimal solutions of its subproblems.
   * *Example:* The shortest path from A to C through B contains the shortest path from A to B and the shortest path from B to C.
2. **Overlapping Subproblems**: The recursive algorithm solves the same subproblems repeatedly rather than generating new subproblems at each step.
   * *Example:* In calculating the Fibonacci sequence, $F(n) = F(n-1) + F(n-2)$. Both $F(n-1)$ and $F(n-2)$ will independently call $F(n-3)$, leading to redundant computations.

---

## 2. Signs a Problem Might Need DP

You should consider using DP if the problem description contains:
* **Optimization Keywords:** Finding the "minimum", "maximum", "shortest", "longest", "cheapest", or "largest" value.
* **Counting Ways:** "Count the total number of ways to...", "How many distinct paths exist to...", or "Find the number of combinations...".
* **Overlapping Recursive Structure:** When you draw the recursion tree for a naive recursive solution, the same subproblem states appear multiple times.
* **Dependent Choices:** Making a choice at the current step depends on the optimal outcomes of choices made at earlier steps.

---

## 3. How to Exactly Solve a DP Problem

Every DP problem can be solved by systematically following these 5 steps:
1. **Define the State:** Describe mathematically what the subproblem represents (e.g., `dp[i]` represents the minimum cost to reach step `i`).
2. **Identify the Recurrence Relation:** Formulate the transition formula showing how to calculate the current state from previous states.
3. **Establish the Base Cases:** Solve the smallest possible subproblems directly (e.g., `dp[0]` or `dp[1]`).
4. **Choose the Approach:** Decide between **Memoization** (Top-Down) or **Tabulation** (Bottom-Up).
5. **Analyze and Optimize:** Determine the Time and Space complexity, and check if space can be optimized (e.g., rolling variables instead of a full array).

---

## 4. Memoization (Top-Down DP) vs. Tabulation (Bottom-Up DP)

### Memoization (Top-Down)
Memoization starts from the target state and recursively breaks it down into smaller subproblems. Before solving a subproblem, we check if it has already been computed and stored in a lookup table (memo table). If it exists, we return it instantly; otherwise, we compute it, save it in the table, and return it.

#### Step-by-Step Memoization Table Walkthrough ($F(4)$ Fibonacci)
Let's trace the calculation of Fibonacci $F(4) = F(3) + F(2)$ with memo array initialized to `-1`:

| Step | State Called | Memo Table State (`[0, 1, 2, 3, 4]`) | Action Taken |
| :--- | :--- | :--- | :--- |
| **1** | `F(4)` | `[-1, -1, -1, -1, -1]` | Calls `F(3)` |
| **2** | `F(3)` | `[-1, -1, -1, -1, -1]` | Calls `F(2)` |
| **3** | `F(2)` | `[-1, -1, -1, -1, -1]` | Calls `F(1)` |
| **4** | `F(1)` | `[-1, -1, -1, -1, -1]` | Base case: returns `1` |
| **5** | `F(2)` | `[-1, -1, -1, -1, -1]` | Calls `F(0)` |
| **6** | `F(0)` | `[-1, -1, -1, -1, -1]` | Base case: returns `0` |
| **7** | `F(2)` | `[0, 1, 1, -1, -1]` | Computes `F(2) = 1 + 0 = 1`. Stores `memo[2] = 1`. Returns `1` |
| **8** | `F(3)` | `[0, 1, 1, -1, -1]` | Calls `F(1)`. Hits base case, returns `1` |
| **9** | `F(3)` | `[0, 1, 1, 2, -1]` | Computes `F(3) = 1 + 1 = 2`. Stores `memo[3] = 2`. Returns `2` |
| **10**| `F(4)` | `[0, 1, 1, 2, -1]` | Calls `F(2)`. **Hits Memo Table:** `memo[2] = 1` returned instantly (No recursion!). |
| **11**| `F(4)` | `[0, 1, 1, 2, 3]` | Computes `F(4) = 2 + 1 = 3`. Stores `memo[4] = 3`. Returns `3` |

#### C++ Implementation (Memoization)
```cpp
#include <iostream>
#include <vector>
using namespace std;

// Helper function to recursively solve subproblems and fill the memo table
int fibMemoHelper(int n, vector<int>& memo) {
    if (n <= 1) return n;
    
    // Check if the subproblem solution already exists in the memo table
    if (memo[n] != -1) return memo[n];
    
    // Solve, store result in the memo table, and return it
    return memo[n] = fibMemoHelper(n - 1, memo) + fibMemoHelper(n - 2, memo);
}

// Main function to run memoized Fibonacci
int fibMemo(int n) {
    // Initialize memo table of size n + 1 with -1 (representing unsolved states)
    vector<int> memo(n + 1, -1);
    return fibMemoHelper(n, memo);
}
```

### Tabulation (Bottom-Up)
Tabulation starts from the base cases and iteratively fills a table (array) in a forward direction using a loop until the target state is reached.

#### Step-by-Step Tabulation Table Walkthrough ($F(4)$ Fibonacci)
Let's trace how the bottom-up table `dp` is filled for $F(4) = F(3) + F(2)$ with the table size $N+1 = 5$:

| Step | Loop Index `i` | dp Table State (`[0, 1, 2, 3, 4]`) | Action / Computation |
| :--- | :---: | :--- | :--- |
| **1** | — | `[0, 1, -1, -1, -1]` | Initialize base cases: `dp[0] = 0`, `dp[1] = 1`. |
| **2** | `i = 2` | `[0, 1, 1, -1, -1]` | Compute `dp[2] = dp[1] + dp[0] = 1 + 0 = 1`. |
| **3** | `i = 3` | `[0, 1, 1, 2, -1]` | Compute `dp[3] = dp[2] + dp[1] = 1 + 1 = 2`. |
| **4** | `i = 4` | `[0, 1, 1, 2, 3]` | Compute `dp[4] = dp[3] + dp[2] = 2 + 1 = 3`. Target reached. |

#### C++ Implementation (Tabulation)
```cpp
#include <iostream>
#include <vector>
using namespace std;

int fibTabulation(int n) {
    if (n <= 1) return n;
    
    // Create DP table of size n + 1 initialized to 0
    vector<int> dp(n + 1, 0);
    
    // Fill in base cases
    dp[0] = 0;
    dp[1] = 1;
    
    // Iteratively fill the table from bottom to top
    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    return dp[n];
}
```

### Visualizing 2D DP Tables: Top-Down vs. Bottom-Up
For a 2D dynamic programming problem (such as Knapsack or LCS), the grid states are resolved differently under the two approaches:

1. **Top-Down (Memoization - Sparse Filling)**:
   * Only the subproblems along the recursive pathways are calculated and stored.
   * Unreached states remain in their initial state (e.g., `-1` or `unsolved`).
   * Saves computation time if a significant portion of the state space is unreachable.

2. **Bottom-Up (Tabulation - Dense/Complete Filling)**:
   * The matrix is filled in a strict topological order (typically row-by-row, column-by-column).
   * Every cell in the grid is computed sequentially, ensuring all dependencies are resolved before they are needed.
   * Eliminates recursion stack overhead and enables easy space-optimization (e.g., keeping only the previous row/column).

#### Comparison Table

| Feature | Memoization (Top-Down) | Tabulation (Bottom-Up) |
| :--- | :--- | :--- |
| **Approach** | Recursive (Target $\to$ Base Cases) | Iterative (Base Cases $\to$ Target) |
| **State Fill** | Demand-driven (only solves visited states) | Systematic (solves every state sequentially) |
| **Overhead** | Recursive function call stack overhead | No stack overhead; fast execution |
| **Ease of Design** | Very intuitive to write from recursive relation | Can require careful ordering of nested loops |
| **Space Optimization** | Difficult to optimize space complexity | Highly flexible; easy to optimize space |

---

## 5. Standard Examples

### A. Frog Stairs (Climbing Stairs)
**Problem:** A frog wants to climb a staircase with $N$ steps. It can jump either 1 step or 2 steps at a time. Find the total number of distinct ways the frog can reach the top.

* **State:** `dp[i]` = Number of ways to reach stair `i`.
* **Recurrence Relation:** `dp[i] = dp[i-1] + dp[i-2]` (To reach stair `i`, the frog must have jumped from `i-1` or `i-2`).
* **Base Cases:**
  * `dp[0] = 1` (1 way to stay at start: do nothing)
  * `dp[1] = 1` (1 way to reach step 1: jump 1 step)

#### Pseudocode (Tabulation)
```text
FROG-STAIRS(N):
    if N <= 1:
        return 1
    create array dp of size N + 1
    dp[0] = 1
    dp[1] = 1
    for i = 2 to N:
        dp[i] = dp[i-1] + dp[i-2]
    return dp[N]
```

#### C++ Implementation (Space-Optimized)
```cpp
#include <iostream>
#include <vector>
using namespace std;

int countWaysToClimb(int n) {
    if (n <= 1) return 1;
    int prev2 = 1; // dp[i-2]
    int prev1 = 1; // dp[i-1]
    int current = 0;
    
    for (int i = 2; i <= n; i++) {
        current = prev1 + prev2;
        prev2 = prev1;
        prev1 = current;
    }
    return current;
}
```

#### Recursive Analysis
* **Recursive Call Structure (Alternate Recursive Version):**
  ```cpp
  int solveRec(int i, vector<int>& memo) {
      if (i <= 1) return 1;
      if (memo[i] != -1) return memo[i];
      return memo[i] = solveRec(i-1, memo) + solveRec(i-2, memo);
  }
  ```
* **Recursive Calls:** `solveRec(i-1, memo)` and `solveRec(i-2, memo)`.
* **Recursion Frequency:** For $N$ steps, there will be exactly $N+1$ unique states called, each resolved in $O(1)$ due to memoization.
* **Recursive Alternatives for Loops:** The iterative loop `for (int i = 2; i <= n; i++)` is replaced by the recursive call stack traversing downward from `n` to `1`.

#### Complexity Analysis
* **Loops:**
  * Iterative loop runs $N - 1$ times.
  * Inside the loop, it performs $O(1)$ additions.
* **Overall Complexity:**
  * **Best Case:** $O(N)$
  * **Average Case:** $O(N)$
  * **Worst Case:** $O(N)$
* **Space Complexity:** $O(1)$ space for optimized version (only tracking previous two states), or $O(N)$ if using full `dp` array.

---

### B. Frog K-Steps
**Problem:** A frog wants to climb a staircase with $N$ steps. It can jump any step size from $1$ to $K$ steps in a single bound. Find the total number of distinct ways to reach the top.

* **State:** `dp[i]` = Number of ways to reach stair `i`.
* **Recurrence Relation:** `dp[i] = sum_{j=1}^{K} dp[i-j]` (Sum of ways from all reachable steps below it).
* **Base Cases:**
  * `dp[0] = 1`
  * For $i < 0$, `dp[i] = 0`.

#### Pseudocode (Tabulation)
```text
FROG-K-STEPS(N, K):
    create array dp of size N + 1
    dp[0] = 1
    for i = 1 to N:
        dp[i] = 0
        for j = 1 to K:
            if i - j >= 0:
                dp[i] = dp[i] + dp[i - j]
    return dp[N]
```

#### C++ Implementation
```cpp
#include <iostream>
#include <vector>
using namespace std;

int countWaysKSteps(int n, int k) {
    vector<int> dp(n + 1, 0);
    dp[0] = 1;
    
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= k; j++) {
            if (i - j >= 0) {
                dp[i] += dp[i - j];
            }
        }
    }
    return dp[n];
}
```

#### Recursive Analysis
* **Recursive Call Structure (Alternate Recursive Version):**
  ```cpp
  int solveKRec(int i, int k, vector<int>& memo) {
      if (i == 0) return 1;
      if (i < 0) return 0;
      if (memo[i] != -1) return memo[i];
      int ans = 0;
      for (int j = 1; j <= k; j++) {
          ans += solveKRec(i - j, k, memo);
      }
      return memo[i] = ans;
  }
  ```
* **Recursive Calls:** `solveKRec(i-j, k, memo)` for $j \in [1, K]$.
* **Recursion Frequency:** $N$ unique recursive states, each triggering $K$ sub-branches.
* **Recursive Alternatives for Loops:** The outer loop `i` is replaced by the recursion stack traversing states $N \to 0$. The inner loop `j` can be replaced by a helper recursive function `sumPreviousStates(i, stepIndex)` that accumulates the values of `dp[i-stepIndex]` from `stepIndex = 1` to `k`.

#### Complexity Analysis
* **Loops:**
  * Outer loop runs $N$ times.
  * Inner loop runs $K$ times for each outer iteration.
* **Overall Complexity:**
  * **Best Case:** $O(N \cdot K)$
  * **Average Case:** $O(N \cdot K)$
  * **Worst Case:** $O(N \cdot K)$
* **Space Complexity:** $O(N)$ to store the `dp` array.

---

### C. 0/1 Knapsack Problem
**Problem:** You are given weights and values of $N$ items. Put these items in a knapsack of capacity $W$ to get the maximum total value. Each item can either be taken ($1$) or not taken ($0$).

* **State:** `dp[i][w]` = Maximum value obtainable using the first `i` items and a knapsack of capacity `w`.
* **Recurrence Relation:**
  * If weight of $i$-th item ($wt[i-1]$) is less than or equal to $w$:
    `dp[i][w] = max(dp[i - 1][w], val[i - 1] + dp[i - 1][w - wt[i - 1]])`
  * If weight of $i$-th item ($wt[i-1]$) is greater than $w$:
    `dp[i][w] = dp[i - 1][w]`
* **Base Cases:**
  * `dp[0][w] = 0` (0 items available $\to$ 0 value)
  * `dp[i][0] = 0` (capacity is 0 $\to$ 0 value)

#### Pseudocode (Tabulation)
```text
KNAPSAK(W, wt, val, N):
    create 2D array dp of size (N + 1) x (W + 1) initialized to 0
    for i = 1 to N:
        for w = 1 to W:
            if wt[i-1] <= w:
                dp[i][w] = max(dp[i-1][w], val[i-1] + dp[i-1][w - wt[i-1]])
            else:
                dp[i][w] = dp[i-1][w]
    return dp[N][W]
```

#### C++ Implementation
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int knapsack01(int W, const vector<int>& wt, const vector<int>& val, int n) {
    vector<vector<int>> dp(n + 1, vector<int>(W + 1, 0));
    
    for (int i = 1; i <= n; i++) {
        for (int w = 1; w <= W; w++) {
            if (wt[i - 1] <= w) {
                dp[i][w] = max(dp[i - 1][w], val[i - 1] + dp[i - 1][w - wt[i - 1]]);
            } else {
                dp[i][w] = dp[i - 1][w];
            }
        }
    }
    return dp[n][W];
}
```

#### Recursive Analysis
* **Recursive Call Structure (Alternate Recursive Version):**
  ```cpp
  int solveKnap(int i, int w, const vector<int>& wt, const vector<int>& val, vector<vector<int>>& memo) {
      if (i == 0 || w == 0) return 0;
      if (memo[i][w] != -1) return memo[i][w];
      if (wt[i-1] <= w) {
          return memo[i][w] = max(solveKnap(i-1, w, wt, val, memo), 
                                  val[i-1] + solveKnap(i-1, w - wt[i-1], wt, val, memo));
      }
      return memo[i][w] = solveKnap(i-1, w, wt, val, memo);
  }
  ```
* **Recursive Calls:** `solveKnap(i-1, w, ...)` and `solveKnap(i-1, w - wt[i-1], ...)`.
* **Recursion Frequency:** There are $N \cdot W$ distinct subproblem states, each calculated in $O(1)$ time with memoization.
* **Recursive Alternatives for Loops:** The outer loop `i` is replaced by the recursion stack tracking the item index. The inner loop `w` is replaced by recursive calls checking capacity configurations.

#### Complexity Analysis
* **Loops:**
  * Outer loop runs $N$ times.
  * Inner loop runs $W$ times for each item.
* **Overall Complexity:**
  * **Best Case:** $O(N \cdot W)$
  * **Average Case:** $O(N \cdot W)$
  * **Worst Case:** $O(N \cdot W)$
* **Space Complexity:** $O(N \cdot W)$ to store the 2D array. Can be space-optimized to $O(W)$ by keeping only the previous row.

#### 0/1 Knapsack DP Tables Comparison
Given a knapsack of capacity $W = 4$ and three items: Item 1 ($wt=1, val=15$), Item 2 ($wt=2, val=20$), and Item 3 ($wt=3, val=30$).

##### 1. Bottom-Up Tabulation Table (Fully Computed)
Every state `dp[i][w]` is populated systematically row-by-row:

| i \ w | 0 | 1 | 2 | 3 | 4 |
| :---: | :---: | :---: | :---: | :---: | :---: |
| **0 (empty)** | 0 | 0 | 0 | 0 | 0 |
| **1 (Item 1)** | 0 | 15 | 15 | 15 | 15 |
| **2 (Item 2)** | 0 | 15 | 20 | 35 | 35 |
| **3 (Item 3)** | 0 | 15 | 20 | 35 | **45** |

##### 2. Top-Down Memoization Table (Sparsely Computed)
Only required subproblems are solved recursively. Unvisited states remain `-1`:
*(Note: Base cases where $i=0$ or $w=0$ return $0$ directly without memo table writes).*

| i \ w | 0 | 1 | 2 | 3 | 4 |
| :---: | :---: | :---: | :---: | :---: | :---: |
| **0 (empty)** | — | — | — | — | — |
| **1 (Item 1)** | — | 15 | 15 | **-1** | 15 |
| **2 (Item 2)** | — | 15 | **-1** | **-1** | 35 |
| **3 (Item 3)** | — | **-1** | **-1** | **-1** | **45** |

---

### D. Longest Common Subsequence (LCS)
**Problem:** Given two strings $S1$ and $S2$, find the length of their longest common subsequence. A subsequence is a sequence that appears in the same relative order, but not necessarily contiguously.

* **State:** `dp[i][j]` = Length of LCS of prefix strings $S1[0 \dots i-1]$ and $S2[0 \dots j-1]$.
* **Recurrence Relation:**
  * If characters match ($S1[i-1] == S2[j-1]$):
    `dp[i][j] = 1 + dp[i-1][j-1]`
  * If characters do not match ($S1[i-1] \neq S2[j-1]$):
    `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`
* **Base Cases:**
  * `dp[0][j] = 0` (empty $S1$)
  * `dp[i][0] = 0` (empty $S2$)

#### Pseudocode (Tabulation)
```text
LCS(S1, S2):
    m = S1.length, n = S2.length
    create 2D array dp of size (m + 1) x (n + 1) initialized to 0
    for i = 1 to m:
        for j = 1 to n:
            if S1[i-1] == S2[j-1]:
                dp[i][j] = 1 + dp[i-1][j-1]
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    return dp[m][n]
```

#### C++ Implementation
```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int getLCS(string S1, string S2) {
    int m = S1.length();
    int n = S2.length();
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (S1[i - 1] == S2[j - 1]) {
                dp[i][j] = 1 + dp[i - 1][j - 1];
            } else {
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    return dp[m][n];
}
```

#### Recursive Analysis
* **Recursive Call Structure (Alternate Recursive Version):**
  ```cpp
  int solveLCS(int i, int j, string& S1, string& S2, vector<vector<int>>& memo) {
      if (i == 0 || j == 0) return 0;
      if (memo[i][j] != -1) return memo[i][j];
      if (S1[i-1] == S2[j-1]) {
          return memo[i][j] = 1 + solveLCS(i-1, j-1, S1, S2, memo);
      }
      return memo[i][j] = max(solveLCS(i-1, j, S1, S2, memo), solveLCS(i, j-1, S1, S2, memo));
  }
  ```
* **Recursive Calls:** `solveLCS(i-1, j-1, ...)` or choice of `solveLCS(i-1, j, ...)` / `solveLCS(i, j-1, ...)`.
* **Recursion Frequency:** There are $M \cdot N$ unique states (where $M = |S1|$, $N = |S2|$), each resolved once.
* **Recursive Alternatives for Loops:** The loops `i` and `j` are replaced by recursion branches traveling from final lengths to base cases ($0$).

#### Complexity Analysis
* **Loops:**
  * Outer loop runs $M$ times.
  * Inner loop runs $N$ times.
* **Overall Complexity:**
  * **Best Case:** $O(M \cdot N)$
  * **Average Case:** $O(M \cdot N)$
  * **Worst Case:** $O(M \cdot N)$
* **Space Complexity:** $O(M \cdot N)$ for storing the DP table. Can be space-optimized to $O(N)$ since only the previous row is needed.

#### LCS DP Tables Comparison
Given two strings: $S1 = \text{"AET"}$ and $S2 = \text{"AFT"}$.

##### 1. Bottom-Up Tabulation Table (Fully Computed)
Every state `dp[i][j]` is populated sequentially:

| i \ j | 0 (ø) | 1 ('A') | 2 ('F') | 3 ('T') |
| :---: | :---: | :---: | :---: | :---: |
| **0 (ø)** | 0 | 0 | 0 | 0 |
| **1 ('A')** | 0 | 1 | 1 | 1 |
| **2 ('E')** | 0 | 1 | 1 | 1 |
| **3 ('T')** | 0 | 1 | 1 | **2** |

##### 2. Top-Down Memoization Table (Sparsely Computed)
Only reachable subproblems are solved. Unvisited states remain `-1`:
*(Note: Base cases where $i=0$ or $j=0$ return $0$ directly).*

| i \ j | 0 (ø) | 1 ('A') | 2 ('F') | 3 ('T') |
| :---: | :---: | :---: | :---: | :---: |
| **0 (ø)** | — | — | — | — |
| **1 ('A')** | — | 1 | 1 | **-1** |
| **2 ('E')** | — | 1 | 1 | **-1** |
| **3 ('T')** | — | **-1** | **-1** | **2** |

---

## 6. Solved Problems (Q&A)

**Q. Minimum Coins Problem:**
You have coins of denominations: `{1, 3, 4}`. Find the minimum number of coins needed to make a total sum of $6$ units. A coin denomination may be used any number of times. Solve using DP by identifying the state, recurrence relation, and base cases. Show a step-by-step tabulation table.

**ANS:**
### 1. DP formulation
* **State:** Let `dp[x]` be the minimum number of coins required to form the sum `x`.
* **Recurrence Relation:** 
  `dp[x] = 1 + min(dp[x - 1], dp[x - 3], dp[x - 4])` (for $x \ge \text{coin value}$)
  Generally: `dp[x] = 1 + min_{c \in \{1, 3, 4\}, c \le x} (dp[x - c])`
* **Base Case:** 
  `dp[0] = 0` (0 coins are needed to make a sum of 0).
  `dp[x] = INF` for $x < 0$.

### 2. Step-by-Step Tabulation Table Trace
We initialize `dp` table of size $7$ ($0$ to $6$) with $\infty$ (infinity), except `dp[0] = 0`. Let's calculate:

* **$x = 0$:** `dp[0] = 0`
* **$x = 1$:** `1 + dp[1 - 1] = 1 + dp[0] = 1 + 0 = 1`. Hence `dp[1] = 1`.
* **$x = 2$:** `1 + dp[2 - 1] = 1 + dp[1] = 1 + 1 = 2`. Hence `dp[2] = 2`.
* **$x = 3$:** `1 + min(dp[3 - 1], dp[3 - 3]) = 1 + min(dp[2], dp[0]) = 1 + min(2, 0) = 1`. Hence `dp[3] = 1`.
* **$x = 4$:** `1 + min(dp[4 - 1], dp[4 - 3], dp[4 - 4]) = 1 + min(dp[3], dp[1], dp[0]) = 1 + min(1, 1, 0) = 1`. Hence `dp[4] = 1`.
* **$x = 5$:** `1 + min(dp[5 - 1], dp[5 - 3], dp[5 - 4]) = 1 + min(dp[4], dp[2], dp[1]) = 1 + min(1, 2, 1) = 2`. Hence `dp[5] = 2`.
* **$x = 6$:** `1 + min(dp[6 - 1], dp[6 - 3], dp[6 - 4]) = 1 + min(dp[5], dp[3], dp[2]) = 1 + min(2, 1, 2) = 2`. Hence `dp[6] = 2`.

| Sum ($x$) | Options Used: $1 + dp[x - c]$ | Chosen Minimum | dp Table State |
| :---: | :--- | :---: | :--- |
| **0** | None (Base Case) | 0 | `[0, INF, INF, INF, INF, INF, INF]` |
| **1** | $1 + dp[0] = 1$ | 1 | `[0, 1, INF, INF, INF, INF, INF]` |
| **2** | $1 + dp[1] = 2$ | 2 | `[0, 1, 2, INF, INF, INF, INF]` |
| **3** | $\min(1+dp[2], 1+dp[0]) = \min(3, 1) = 1$ | 1 | `[0, 1, 2, 1, INF, INF, INF]` |
| **4** | $\min(1+dp[3], 1+dp[1], 1+dp[0]) = \min(2, 2, 1) = 1$ | 1 | `[0, 1, 2, 1, 1, INF, INF]` |
| **5** | $\min(1+dp[4], 1+dp[2], 1+dp[1]) = \min(2, 3, 2) = 2$ | 2 | `[0, 1, 2, 1, 1, 2, INF]` |
| **6** | $\min(1+dp[5], 1+dp[3], 1+dp[2]) = \min(3, 2, 3) = 2$ | 2 | `[0, 1, 2, 1, 1, 2, 2]` |

**Result:** Minimum coins to spend $6$ units is `dp[6] = 2` (using coins $\{3, 3\}$).

### 3. C++ Implementation (Tabulation)
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

const int INF = 1e9;

int getMinCoins(vector<int>& coins, int sum) {
    vector<int> dp(sum + 1, INF);
    dp[0] = 0; // Base case

    for (int x = 1; x <= sum; x++) {
        for (int coin : coins) {
            if (x - coin >= 0) {
                dp[x] = min(dp[x], 1 + dp[x - coin]);
            }
        }
    }
    return dp[sum] == INF ? -1 : dp[sum];
}

int main() {
    vector<int> coins = {1, 3, 4};
    int target = 6;
    cout << "Min coins for " << target << " is: " << getMinCoins(coins, target) << endl;
    return 0;
}
```

### 4. Pseudocode
```text
MIN-COINS(Coins, Sum):
    create array dp of size Sum + 1 filled with INF
    dp[0] = 0
    for x = 1 to Sum:
        for each coin in Coins:
            if x - coin >= 0:
                dp[x] = min(dp[x], 1 + dp[x - coin])
    return dp[Sum]
```

### 5. Recursive Analysis
* **Recursive Call Structure:**
  ```cpp
  int minCoinsRec(int x, vector<int>& coins, vector<int>& memo) {
      if (x == 0) return 0;
      if (x < 0) return INF;
      if (memo[x] != -1) return memo[x];
      int minCoins = INF;
      for (int coin : coins) {
          minCoins = min(minCoins, 1 + minCoinsRec(x - coin, coins, memo));
      }
      return memo[x] = minCoins;
  }
  ```
* **Recursive Calls:** `minCoinsRec(x - coin, coins, memo)`.
* **Recursion Frequency:** For target sum $S$, there are $S$ unique subproblems. Each subproblem checks $C$ coins, giving a total recurrence activation frequency of $S \cdot C$ calls.
* **Recursive Alternatives for Loops:** The outer loop `x` can be replaced by the recursion stack, and the inner loop over coins can be replaced by recursive sub-branches executing for each element in the coin list.

### 6. Complexity Analysis
* **Loops:**
  * Outer loop runs $S$ times (where $S = \text{Sum}$).
  * Inner loop runs $C$ times (where $C = \text{number of coin denominations}$).
* **Overall Complexity:**
  * **Best Case:** $O(S \cdot C)$
  * **Average Case:** $O(S \cdot C)$
  * **Worst Case:** $O(S \cdot C)$
* **Space Complexity:** $O(S)$ to hold the `dp` array.

---

**Q. Friends Pairing Problem:**
Given $N$ friends, each one can remain single or can be paired up with some other friend. Each friend can be paired only once. Find out the total number of ways in which friends can remain single or can be paired up. Solve using DP by identifying the state, recurrence relation, and base cases.

**ANS:**
### 1. DP Formulation
* **State:** Let `dp[i]` be the total number of ways in which `i` friends can remain single or be paired up.
* **Recurrence Relation:** 
  Consider the $i$-th friend. They have two choices:
  1. **Remain Single:** The remaining $i - 1$ friends can pair up in `dp[i - 1]` ways.
  2. **Pair Up with someone:** The $i$-th friend can pair up with any of the remaining $i - 1$ friends (there are $i - 1$ possible partners). Once paired, the remaining $i - 2$ friends can pair up in `dp[i - 2]` ways.
  Therefore:
  `dp[i] = dp[i - 1] + (i - 1) * dp[i - 2]`
* **Base Cases:**
  * `dp[0] = 1` (1 way to pair 0 people: do nothing)
  * `dp[1] = 1` (1 way for 1 person: remain single)
  * `dp[2] = 2` (2 ways for 2 people: both single `(A, B)` or paired up `(AB)`)

### 2. C++ Implementation (Tabulation)
```cpp
#include <iostream>
#include <vector>
using namespace std;

int countFriendsPairingWays(int n) {
    if (n <= 1) return 1;
    vector<int> dp(n + 1, 0);
    dp[0] = 1;
    dp[1] = 1;

    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i - 1] + (i - 1) * dp[i - 2];
    }
    return dp[n];
}

int main() {
    int n = 4;
    cout << "Total ways to pair " << n << " friends: " << countFriendsPairingWays(n) << endl;
    return 0;
}
```

### 3. Pseudocode
```text
FRIENDS-PAIRING(N):
    if N <= 1:
        return 1
    create array dp of size N + 1
    dp[0] = 1
    dp[1] = 1
    for i = 2 to N:
        dp[i] = dp[i-1] + (i - 1) * dp[i-2]
    return dp[N]
```

### 4. Recursive Analysis
* **Recursive Call Structure:**
  ```cpp
  int solveRec(int i, vector<int>& memo) {
      if (i <= 1) return 1;
      if (memo[i] != -1) return memo[i];
      return memo[i] = solveRec(i - 1, memo) + (i - 1) * solveRec(i - 2, memo);
  }
  ```
* **Recursive Calls:** `solveRec(i - 1, memo)` and `solveRec(i - 2, memo)`.
* **Recursion Frequency:** For $N$ friends, there are exactly $N + 1$ states, each visited and calculated exactly once.
* **Recursive Alternatives for Loops:** The iterative loop `for (int i = 2; i <= n; i++)` is replaced by the recursive call stack traversing from $N$ down to base cases.

### 5. Complexity Analysis
* **Loops:**
  * Loop runs $N - 1$ times, with $O(1)$ operations inside.
* **Overall Complexity:**
  * **Best Case:** $O(N)$
  * **Average Case:** $O(N)$
  * **Worst Case:** $O(N)$
* **Space Complexity:** $O(N)$ to hold the `dp` table. Can be optimized to $O(1)$ by using variables `prev1` and `prev2`.

---

## 7. Practice Problems (No Solutions)

Test your understanding of dynamic programming by solving these classic problems. For each problem, review the **explanation**, **state**, **recurrence relation**, **base cases**, and **complexity details**, and then try to implement them on your own!

---

### 1. House Robber (Maximum Sum of Non-Adjacent Elements)
* **Problem:** You are planning to rob houses along a street. Each house has a certain amount of money stashed. However, adjacent houses have security systems connected, and it will automatically contact the police if two adjacent houses are broken into on the same night. Given a list of non-negative integers representing the amount of money in each house, determine the maximum amount of money you can rob tonight without alerting the police.
* **Explanation:** At the $i$-th house, you have two choices:
  1. **Rob the house:** You get its money (`money[i]`) but cannot rob the previous house, so you add this to the optimal rob value up to the house $i-2$.
  2. **Skip the house:** The maximum money you can get is the optimal rob value up to the house $i-1$.
  Your goal is to maximize the payoff of these choices.
* **State:** `dp[i]` = Maximum money obtainable from the first $i$ houses.
* **Recurrence Relation:**
  `dp[i] = max(dp[i - 1], money[i] + dp[i - 2])`
* **Base Cases:**
  * `dp[0] = money[0]` (Only one house $\to$ rob it)
  * `dp[1] = max(money[0], money[1])` (Two houses $\to$ rob the richer one)
* **Complexity:**
  * **Time Complexity:** $O(N)$ because we iterate through the list of houses exactly once.
  * **Space Complexity:** $O(N)$ to store the DP table (can be optimized to $O(1)$ by keeping track of only the previous two outcomes).

---

### 2. Grid Minimum Path Sum
* **Problem:** Given an $M \times N$ grid filled with non-negative numbers, find a path from the top-left corner to the bottom-right corner that minimizes the sum of all numbers along its path. You can only move either down or right at any point in time.
* **Explanation:** To find the minimum path sum to reach any cell `(i, j)`, you look at the cells you could have come from: directly above `(i-1, j)` or directly to the left `(i, j-1)`. The optimal cost to reach the current cell is its own grid value plus the minimum cost of reaching one of those two preceding cells.
* **State:** `dp[i][j]` = Minimum path sum from the start `(0, 0)` to cell `(i, j)`.
* **Recurrence Relation:**
  `dp[i][j] = grid[i][j] + min(dp[i - 1][j], dp[i][j - 1])`
* **Base Cases:**
  * `dp[0][0] = grid[0][0]` (Starting cell)
  * For first row ($i = 0$): `dp[0][j] = dp[0][j-1] + grid[0][j]` (Can only move right)
  * For first column ($j = 0$): `dp[i][0] = dp[i-1][0] + grid[i][0]` (Can only move down)
* **Complexity:**
  * **Time Complexity:** $O(M \cdot N)$ as we visit every cell in the grid exactly once.
  * **Space Complexity:** $O(M \cdot N)$ to store the 2D DP matrix (can be optimized to $O(N)$ by keeping only the previous row in memory).

---

### 3. Partition Equal Subset Sum
* **Problem:** Given an array of positive integers, determine if the array can be partitioned into two subsets such that the sum of elements in both subsets is equal.
* **Explanation:** First, check if the total sum of the array is even; if it is odd, it is impossible to partition it equally. If even, the target sum is $Target = TotalSum / 2$. This is a variation of the Subset Sum problem: we want to find if there is a subset of elements that sums up to exactly $Target$. For each element `nums[i-1]`, we can either:
  1. **Exclude it:** The target sum $j$ must be achievable using the first $i-1$ elements.
  2. **Include it:** Check if the remaining sum $j - nums[i-1]$ is achievable using the first $i-1$ elements.
* **State:** `dp[i][j]` = Boolean (`true` or `false`) indicating if a subset sum $j$ is achievable using the first $i$ elements.
* **Recurrence Relation:**
  * If $nums[i-1] \le j$: `dp[i][j] = dp[i-1][j] || dp[i-1][j - nums[i-1]]`
  * If $nums[i-1] > j$: `dp[i][j] = dp[i-1][j]`
* **Base Cases:**
  * `dp[i][0] = true` for all $i$ (A target sum of 0 is always achievable with an empty subset)
  * `dp[0][j] = false` for $j > 0$ (An empty set cannot form a positive sum)
* **Complexity:**
  * **Time Complexity:** $O(N \cdot Target)$ where $N$ is the number of elements and $Target = TotalSum / 2$.
  * **Space Complexity:** $O(N \cdot Target)$ for the DP table (can be optimized to $O(Target)$).

---

### 4. Longest Increasing Subsequence (LIS)
* **Problem:** Given an integer array, return the length of the longest strictly increasing subsequence.
* **Explanation:** To find the longest increasing subsequence ending at index $i$, compare the element `arr[i]` with all previous elements `arr[j]` (where $j < i$). If `arr[j] < arr[i]`, it means `arr[i]` can extend the increasing subsequence that ends at index $j$. We check all valid preceding elements, find the maximum subsequence length ending at any of those indices, and add 1 to it.
* **State:** `dp[i]` = Length of the longest increasing subsequence ending at index `i`.
* **Recurrence Relation:**
  `dp[i] = 1 + max({dp[j]})` for all $0 \le j < i$ such that `arr[j] < arr[i]` (If no such $j$ exists, `dp[i] = 1`)
* **Base Case:**
  * `dp[i] = 1` for all $i$ (Initially, every single element represents an increasing subsequence of length 1)
* **Complexity:**
  * **Time Complexity:** $O(N^2)$ because for each element we scan all previous elements. (Note: An $O(N \log N)$ approach is possible using binary search).
  * **Space Complexity:** $O(N)$ to store the `dp` array.

---

### 5. Edit Distance (Levenshtein Distance)
* **Problem:** Given two strings `word1` and `word2`, return the minimum number of operations (insert, delete, or replace a character) required to convert `word1` to `word2`.
* **Explanation:** We process both strings character by character from the end.
  * If the characters match (`word1[i-1] == word2[j-1]`), no operation is needed, and we recurse for `i-1` and `j-1`.
  * If they do not match, we take the minimum cost among three choices:
    1. **Insert:** Solve for the target matching after inserting (`dp[i][j-1] + 1`).
    2. **Delete:** Solve after removing the character (`dp[i-1][j] + 1`).
    3. **Replace:** Solve after substituting the character (`dp[i-1][j-1] + 1`).
* **State:** `dp[i][j]` = Minimum operations needed to convert prefix `word1[0...i-1]` to prefix `word2[0...j-1]`.
* **Recurrence Relation:**
  * If characters match: `dp[i][j] = dp[i-1][j-1]`
  * If characters do not match: `dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])`
* **Base Cases:**
  * `dp[i][0] = i` (Converting string of length $i$ to empty string requires $i$ deletions)
  * `dp[0][j] = j` (Converting empty string to string of length $j$ requires $j$ insertions)
* **Complexity:**
  * **Time Complexity:** $O(M \cdot N)$ where $M$ and $N$ are the lengths of `word1` and `word2`.
  * **Space Complexity:** $O(M \cdot N)$ to store the 2D operations table (can be optimized to $O(N)$).

---
