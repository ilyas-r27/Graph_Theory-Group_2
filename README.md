# Graph_Theory-Group_2

# Fleury’s Algorithm (Undirected) — Python (Jupyter-Ready)



## Assignment Spec

- **Input**: list of nodes, list of undirected edges `[(u, v), ...]`
- **Output**: print `Eulerian path: a-b-c-...` if exists; otherwise print `euler path not found`
- **Rule**: follow Fleury’s principle — **don’t burn bridges** (avoid choosing a bridge unless forced)

---

## Features

- Self-contained (no third-party libs)
- Deterministic (neighbors visited in sorted order)
- Handles **Eulerian circuit** (0 odd-degree vertices) and **Eulerian path** (2 odd-degree vertices)
- Treats graphs with **isolated vertices** as **not Eulerian** (to match common assignment expectations)

---

## Usage 



```python
nodes1 = [0, 1, 2, 3]
edges1 = [(0,1), (0,2), (1,2), (2,3)]
print_euler_result(nodes1, edges1)  # → Eulerian path: 2-0-1-2-3

nodes2 = [0, 1, 2, 3]
edges2 = [(0,1), (1,2)]
print_euler_result(nodes2, edges2)  # → euler path not found
```

**Input format**
- `nodes`: list of hashable labels (e.g., integers or strings)
- `edges`: list of **undirected** pairs `(u, v)`; multi-edges allowed

**Output format**
- If path exists → `Eulerian path: a-b-c-...`
- If no path/circuit → `euler path not found`

---

## Algorithm Summary (Fleury, Undirected)

1. Check degrees → must have **0 or 2** odd-degree vertices.
2. Start:  
   - **0 odd** → start at any vertex with degree > 0 (we pick the smallest).  
   - **2 odd** → start at one of the odd vertices (we pick the smaller).  
3. At each step, choose an edge to traverse: prefer **non-bridge**.  
   If all available edges are bridges, take one (forced).  
4. Stop when all edges are used exactly once.

**Connectivity rule used here:** single connected component over all listed nodes, **at least one edge**, and **no isolated vertices**.

---

## Complexity

Fleury requires frequent bridge checks. With a naive bridge test (`O(V+E)` via DFS) repeated up to `E` times, worst-case time is about **O(E × (V+E))**.  
For large graphs, consider **Hierholzer’s Algorithm**.

---

## Function-by-Function Explanation (with Code)

### 1) `build_adj(nodes, edges)`
- **Purpose:** Build an undirected **adjacency list** (supports multigraph).
- **Params:** `nodes` (labels), `edges` (pairs `(u, v)`).
- **Returns:** dict `{node: [neighbors...]}`.
- **Complexity:** `O(V + E)`.

```python
def build_adj(nodes, edges):
    adj = {v: [] for v in nodes}
    for u, v in edges:
        if u not in adj: adj[u] = []
        if v not in adj: adj[v] = []
        adj[u].append(v)
        adj[v].append(u)
    return adj
```

### 2) `erase_edge(adj, u, v)`
- **Purpose:** Remove one occurrence of edge `(u, v)` in both directions.
- **Complexity:** `O(deg(u) + deg(v))`.

```python
def erase_edge(adj, u, v):
    adj[u].remove(v)
    adj[v].remove(u)
```

### 3) `add_edge(adj, u, v)`
- **Purpose:** Add back one occurrence of edge `(u, v)` (used during bridge tests).
- **Complexity:** ~`O(1)`.

```python
def add_edge(adj, u, v):
    adj[u].append(v)
    adj[v].append(u)
```

### 4) `reachable_count(start, adj)`
- **Purpose:** Count how many vertices are reachable from `start` (BFS) with **current** edges.
- **Used by:** `connected_over_all_nodes`, `is_bridge`.
- **Complexity:** `O(V + E)`.

```python
from collections import deque

def reachable_count(start, adj):
    seen = {start}
    q = deque([start])
    while q:
        x = q.popleft()
        for y in adj[x]:
            if y not in seen:
                seen.add(y)
                q.append(y)
    return len(seen)
```

### 5) `has_isolated(adj)`
- **Purpose:** Check if any vertex has degree `0` (isolated).  
- **Note:** This implementation treats isolated vertices as **not Eulerian**.
- **Complexity:** `O(V)`.

```python
def has_isolated(adj):
    return any(len(nbrs) == 0 for nbrs in adj.values())
```

### 6) `connected_over_all_nodes(adj)`
- **Purpose:** Ensure graph has ≥1 edge, **no isolated vertices**, and all listed nodes are in **one connected component**.
- **Complexity:** `O(V + E)`.

```python
def connected_over_all_nodes(adj):
    m = sum(len(n) for n in adj.values()) // 2
    if m == 0: return False
    if has_isolated(adj): return False
    start = sorted(adj.keys())[0]
    return reachable_count(start, adj) == len(adj)
```

### 7) `odd_vertices(adj)`
- **Purpose:** Return list of vertices with **odd degree** (sorted for determinism).
- **Complexity:** `O(V)`.

```python
def odd_vertices(adj):
    return sorted([v for v, nbrs in adj.items() if len(nbrs) % 2 == 1])
```

### 8) `is_bridge(adj, u, v)`
- **Purpose:** Determine if edge `(u, v)` is a **bridge** in the current graph.
- **Method:** Compare reachable count from `u` before vs after temporarily removing `(u, v)`.
- **Complexity per test:** ≈ `O(V + E)`.

```python
def is_bridge(adj, u, v):
    before = reachable_count(u, adj)
    erase_edge(adj, u, v)
    after = reachable_count(u, adj) if adj[u] else 0
    add_edge(adj, u, v)
    return after < before
```

### 9) `fleury_euler_path(nodes, edges)`
- **Purpose:** Return Eulerian path as a list of vertices, or `None` if not exists.
- **Steps:**
  1. Build adjacency.
  2. Check connectivity + parity (0 or 2 odds).
  3. Choose start (odd if 2 odds, otherwise smallest degree>0).
  4. While edges remain: prefer **non-bridge**; if all are bridges, take one.
- **Worst-case:** ~`O(E × (V + E))`.

```python
def fleury_euler_path(nodes, edges):
    adj = build_adj(nodes, edges)
    if not connected_over_all_nodes(adj):
        return None
    odd = odd_vertices(adj)
    if len(odd) not in (0, 2):
        return None
    cur = min(odd) if len(odd) == 2 else min([v for v in adj if len(adj[v]) > 0])
    path = [cur]
    edges_left = sum(len(n) for n in adj.values()) // 2
    while edges_left > 0:
        nbrs = sorted(adj[cur])
        if not nbrs:
            return None
        nxt = None
        if len(nbrs) == 1:
            nxt = nbrs[0]
        else:
            for w in nbrs:
                if not is_bridge(adj, cur, w):
                    nxt = w
                    break
            if nxt is None:
                nxt = nbrs[0]
        erase_edge(adj, cur, nxt)
        edges_left -= 1
        cur = nxt
        path.append(cur)
    return path
```

### 10) `print_euler_result(nodes, edges)`
- **Purpose:** Print result in assignment format.

```python
def print_euler_result(nodes, edges):
    ans = fleury_euler_path(nodes, edges)
    if ans is None:
        print("euler path not found")
    else:
        print("Eulerian path:", "-".join(map(str, ans)))
```

---

## Examples

```python
print("Example 1 (from prompt):")
nodes1 = [0, 1, 2, 3]
edges1 = [(0,1), (0,2), (1,2), (2,3)]
print_euler_result(nodes1, edges1)

print("\nExample 2 (from prompt):")
nodes2 = [0, 1, 2, 3]
edges2 = [(0,1), (1,2)]
print_euler_result(nodes2, edges2)

print("\nExtra Check: Eulerian circuit (simple square)")
nodes3 = [0, 1, 2, 3]
edges3 = [(0,1), (1,2), (2,3), (3,0)]
print_euler_result(nodes3, edges3)
```

---


### Example Testcase ( Input & Output )


![](https://i.imgur.com/tSZTs1f.png)
![](https://i.imgur.com/ePwfHTZ.png)
![](https://i.imgur.com/kTN5UlG.png)
![](https://i.imgur.com/39lo9ez.png)




# Fleury’s Algorithm (Directed)

---

## 1) Typing Setup and Node/Edge Definitions
The following code sets up type aliases and basic utilities for graph representation.
```python
from collections import deque
from typing import List, Tuple, Dict, Any, Optional

Node = Any
Edge = Tuple[Node, Node]
Adj  = Dict[Node, List[Node]]
```
**Explanation:**  
- `deque` is used for BFS.  
- Type hints from `typing` improve clarity.  
- Aliases: `Node` (graph vertex), `Edge` (directed pair `(u, v)`), `Adj` (adjacency list).

---

## 2) Directed Graph Structure
This section builds a directed adjacency list and provides degree/connectivity helpers.
```python
def build_adj_directed(nodes: List[Node], edges: List[Edge]) -> Adj:
    adj: Adj = {v: [] for v in nodes}
    for u, v in edges:
        if u not in adj: adj[u] = []
        if v not in adj: adj[v] = []
        adj[u].append(v)
    return adj

def indegrees(adj: Adj) -> Dict[Node, int]:
    indeg = {v: 0 for v in adj}
    for u in adj:
        for v in adj[u]:
            indeg[v] += 1
    return indeg

def undirected_view(adj: Adj) -> Dict[Node, List[Node]]:
    und: Dict[Node, List[Node]] = {v: [] for v in adj}
    for u in adj:
        for v in adj[u]:
            und[u].append(v)
            und[v].append(u)
    return und
```
**Explanation:**  
- `build_adj_directed`: constructs adjacency list for a directed graph.  
- `indegrees`: computes in-degree of each vertex.  
- `undirected_view`: creates an undirected view of the graph (useful for weak connectivity checks).

---

## 3) Side Operations & Reachability
Utility functions for modifying arcs and testing path existence with BFS.
```python
def erase_arc(adj: Adj, u: Node, v: Node) -> None:
    adj[u].remove(v)

def add_arc(adj: Adj, u: Node, v: Node) -> None:
    adj[u].append(v)

def exists_path(u: Node, v: Node, adj: Adj) -> bool:
    if u == v:
        return True
    seen = {u}
    q = deque([u])
    while q:
        x = q.popleft()
        for y in adj[x]:
            if y == v:
                return True
            if y not in seen:
                seen.add(y)
                q.append(y)
    return False
```
**Explanation:**  
- `erase_arc` and `add_arc` modify the adjacency list by removing/adding a directed edge.  
- `exists_path`: checks reachability from `u` to `v` using BFS.

---

## 4) Directed Euler Rules & Fleury’s Algorithm
Implements the rules for Eulerian paths/circuits in directed graphs and the Fleury algorithm itself.
```python
def has_isolated_directed(adj: Adj) -> bool:
    indeg = indegrees(adj)
    return any((len(adj[v]) + indeg[v]) == 0 for v in adj)

def connected_over_all_nodes_directed(adj: Adj) -> bool:
    m = sum(len(adj[v]) for v in adj)
    if m == 0:
        return False
    if has_isolated_directed(adj):
        return False
    und = undirected_view(adj)
    start = sorted(und.keys())[0]
    seen = {start}
    q = deque([start])
    while q:
        x = q.popleft()
        for y in und[x]:
            if y not in seen:
                seen.add(y)
                q.append(y)
    return len(seen) == len(und)

def degree_balance(adj: Adj) -> Dict[Node, int]:
    indeg = indegrees(adj)
    return {v: len(adj[v]) - indeg[v] for v in adj}

def pick_start_vertex(adj: Adj) -> Optional[Node]:
    bal = degree_balance(adj)
    starts = [v for v, b in bal.items() if b == 1]
    ends   = [v for v, b in bal.items() if b == -1]
    zeros  = [v for v, b in bal.items() if b == 0]
    if len(starts) == 1 and len(ends) == 1 and len(starts) + len(ends) + len(zeros) == len(adj):
        return starts[0]
    if len(starts) == 0 and len(ends) == 0 and len(zeros) == len(adj):
        nonempty = [v for v in adj if len(adj[v]) > 0]
        return min(nonempty) if nonempty else None
    return None

def is_cut_arc(adj: Adj, u: Node, v: Node) -> bool:
    erase_arc(adj, u, v)
    ok = exists_path(u, v, adj)
    add_arc(adj, u, v)
    return not ok

def fleury_euler_path_directed(nodes: List[Node], edges: List[Edge]) -> Optional[List[Node]]:
    adj = build_adj_directed(nodes, edges)
    if not connected_over_all_nodes_directed(adj):
        return None
    start = pick_start_vertex(adj)
    if start is None:
        return None
    cur = start
    path = [cur]
    edges_left = sum(len(adj[v]) for v in adj)
    while edges_left > 0:
        nbrs = sorted(adj[cur])
        if not nbrs:
            return None
        if len(nbrs) == 1:
            nxt = nbrs[0]
        else:
            nxt = None
            for w in nbrs:
                if not is_cut_arc(adj, cur, w):
                    nxt = w
                    break
            if nxt is None:
                nxt = nbrs[0]
        erase_arc(adj, cur, nxt)
        edges_left -= 1
        cur = nxt
        path.append(cur)
    return path
```
**Explanation:**  
- `has_isolated_directed`: checks if any node has indegree+outdegree = 0.  
- `connected_over_all_nodes_directed`: ensures weak connectivity.  
- `degree_balance`: computes outdegree − indegree.  
- `pick_start_vertex`: chooses correct start node based on balance rules.  
- `is_cut_arc`: checks if removing an edge disconnects reachability.  
- `fleury_euler_path_directed`: the main Fleury’s Algorithm, building an Eulerian path if possible.

---

## 5) Result Checker
Wrapper for running the algorithm and printing results.
```python
def print_euler_result(nodes: List[Node], edges: List[Edge]) -> None:
    ans = fleury_euler_path_directed(nodes, edges)
    if ans is None:
        print("euler path not found")
    else:
        print("Eulerian path:", "-".join(map(str, ans)))
```
**Explanation:**  
- If no Eulerian path is found, prints `"euler path not found"`.  
- Otherwise, prints the path in a human-readable format.

---

## 6) Examples (Reproducible)
Examples covering three scenarios: Eulerian circuit, Eulerian path only, and non-Eulerian case.
```python
print("Example A (Eulerian circuit):")
nodesA = [0, 1, 2]
edgesA = [(0,1), (1,2), (2,0)]
print_euler_result(nodesA, edgesA)

print("\nExample B (Eulerian path, not circuit):")
nodesB = [0, 1, 2, 3]
edgesB = [(0,1), (1,2), (2,0), (0,3)]
print_euler_result(nodesB, edgesB)

print("\nExample C (Not Eulerian: bad degree balance):")
nodesC = [0, 1, 2]
edgesC = [(0,1), (0,1), (1,2)]
print_euler_result(nodesC, edgesC)
```
**Expected output:**  
- **A** → `Eulerian path: 0-1-2-0` (an Eulerian *circuit*).  
- **B** → `Eulerian path: 0-1-2-0-3` (a valid path but not a circuit).  
- **C** → `euler path not found` (degree balance invalid).

---

## 7) Usage Notes
- Place all functions in a single script or notebook cell.  
- The algorithm chooses neighbors in sorted order for deterministic results.  

---

# Conclusion
This implementation demonstrates **Fleury’s Algorithm for directed graphs**:  
1. Check weak connectivity.  
2. Verify Eulerian conditions via degree balance.  
3. Construct Eulerian path/circuit by avoiding cut-arcs when possible.  

The examples validate three outcomes: **Eulerian circuit**, **Eulerian path**, and **non-Eulerian** case.

# Hierholzer’s Algorithm (Undirected) — Python (Jupyter-Ready)

## Assignment Spec

- **Input**: list of nodes, list of undirected edges `[(u, v), ...)`
- **Output**: print `Eulerian path: a-b-c-...` if exists; otherwise print `euler path not found`
- **Rule**: use **Hierholzer’s algorithm** — build and splice cycles until all edges are consumed.

---
## Features

- Self-contained (no third-party libs)
- Supports **multigraphs** (multi-edges)
- Handles **Eulerian circuit** (0 odd-degree vertices) and **Eulerian path/trail** (2 odd-degree vertices)
- Connectivity is checked only over vertices with degree > 0 (isolated vertices are **ignored**)
- Traversal order follows the **given edge order** per vertex (LIFO due to `pop()`)

---
## Usage 

```python
# Helper to run
def run_program(nodes, edges):
    res = euler_undirected(nodes, edges)
    if res is None:
        print("euler path not found")
    else:
        print(" -> ".join(map(str, res)))

# Quick try
nodes1 = [0, 1, 2, 3]
edges1 = [(0,1), (0,2), (1,2), (2,3)]
run_program(nodes1, edges1)   # prints something like: 2 -> 0 -> 1 -> 2 -> 3

nodes2 = [0, 1, 2, 3]
edges2 = [(0,1), (1,2)]
run_program(nodes2, edges2)   # prints: euler path not found
```

**Input format**
- `nodes`: list of hashable labels (e.g., integers or strings)
- `edges`: list of **undirected** pairs `(u, v)`; multi-edges allowed

**Output format**
- If path exists → `Eulerian path: a-b-c-...`
- If no path/circuit → `euler path not found`

---
## Algorithm Summary (Hierholzer, Undirected)

1. Check degrees → must have **0 or 2** odd-degree vertices.
2. Start:  
   - **0 odd** → start at any vertex with degree > 0.  
   - **2 odd** → start at one of the odd vertices.  
3. Main loop (stack):  
   While the current vertex has an unused incident edge, follow it (consume that edge). If no unused edges at the top of the stack, add the vertex to the path and backtrack. 
4. When done, the path (reversed) uses every edge exactly once.

**Connectivity rule used here:** connectivity can be checked on the subgraph induced by vertices with degree > 0.

---
## Complexity

Runs in **O(V + E)** time (each edge is consumed once); memory **O(V + E)**. If neighbor lists are globally sorted, add **O(E log E)** preprocessing.

---
## Code (Notebook-Compatible)

```python
from collections import defaultdict

def euler_undirected(nodes, edges):
    # build adjacency (multigraph) and degree
    adj = defaultdict(list)
    deg = defaultdict(int)
    for u, v in edges:
        adj[u].append(v); adj[v].append(u)
        deg[u] += 1; deg[v] += 1

    # active nodes = vertices with nonzero degree
    active = {x for x in nodes if deg[x] > 0}
    if not active:
        return None  # no edges → treat as not found (keep it simple)

    # degree condition: 0 odd (circuit) or 2 odd (trail)
    odd = [x for x in active if deg[x] % 2 == 1]
    if len(odd) not in (0, 2):
        return None

    # choose start: odd vertex if 2 odd, else any active
    start = odd[0] if len(odd) == 2 else next(iter(active))

    # connectivity check on active nodes (undirected)
    def connected():
        seen, stack = set(), [start]
        while stack:
            u = stack.pop()
            if u in seen: 
                continue
            if deg[u] == 0:
                continue
            seen.add(u)
            for w in adj[u]:
                stack.append(w)
        return seen == active
    if not connected():
        return None

    # Hierholzer (simple version using back-edge removal)
    m = len(edges)
    path = []
    stack = [start]
    while stack:
        v = stack[-1]
        if adj[v]:
            u = adj[v].pop()     # use one v–u
            adj[u].remove(v)     # remove the paired u–v
            stack.append(u)
        else:
            path.append(stack.pop())

    # must have used all edges → path length = m + 1
    if len(path) != m + 1:
        return None
    return path[::-1]  # start → end

def run_program(nodes, edges):
    res = euler_undirected(nodes, edges)
    if res is None:
        print("euler path not found")
    else:
        print(" -> ".join(map(str, res)))
```

---
## Function-by-Function Explanation (with Code)

### `euler_undirected(... )`
- **Purpose:** Compute Eulerian path/circuit using Hierholzer; returns list of vertices or `None`.
- **Notes:** Consumes each edge exactly once using a stack/backtracking; verifies degree parity and connectedness on the **active** subgraph.

```python
def euler_undirected(nodes, edges):
    # build adjacency (multigraph) and degree
    adj = defaultdict(list)
    deg = defaultdict(int)
    for u, v in edges:
        adj[u].append(v); adj[v].append(u)
        deg[u] += 1; deg[v] += 1

    # active nodes = vertices with nonzero degree
    active = {x for x in nodes if deg[x] > 0}
    if not active:
        return None  # no edges → treat as not found (keep it simple)

    # degree condition: 0 odd (circuit) or 2 odd (trail)
    odd = [x for x in active if deg[x] % 2 == 1]
    if len(odd) not in (0, 2):
        return None

    # choose start: odd vertex if 2 odd, else any active
    start = odd[0] if len(odd) == 2 else next(iter(active))

    # connectivity check on active nodes (undirected)
    def connected():
        seen, stack = set(), [start]
        while stack:
            u = stack.pop()
            if u in seen: 
                continue
            if deg[u] == 0:
                continue
            seen.add(u)
            for w in adj[u]:
                stack.append(w)
        return seen == active
    if not connected():
        return None

    # Hierholzer (simple version using back-edge removal)
    m = len(edges)
    path = []
    stack = [start]
    while stack:
        v = stack[-1]
        if adj[v]:
            u = adj[v].pop()     # use one v–u
            adj[u].remove(v)     # remove the paired u–v
            stack.append(u)
        else:
            path.append(stack.pop())

    # must have used all edges → path length = m + 1
    if len(path) != m + 1:
        return None
    return path[::-1]  # start → end
```

### `run_program(... )`
- **Purpose:** Print results in the required format.
- **Output:** Either `a -> b -> c -> ...` or `euler path not found`.

```python
def run_program(nodes, edges):
    res = euler_undirected(nodes, edges)
    if res is None:
        print("euler path not found")
    else:
        print(" -> ".join(map(str, res)))
```

---
## Examples

```python
print("undirected: eulerian circuit (simple cycle)")
nodes = ['A','B','C','D']
edges = [('A','B'),('B','C'),('C','D'),('D','A')]
run_program(nodes, edges)

print("\nundirected: eulerian trail (two odd degree vertices)")
nodes2 = [1,2,3,4]
edges2 = [(1,2),(2,3),(3,4)]
run_program(nodes2, edges2)

print("\nundirected: not eulerian (more than two odds)")
nodes3 = ['x','y','z']
edges3 = [('x','y'),('y','z')]
run_program(nodes3, edges3)
```

---
## Sample Inputs & Outputs

**Example 1 — Eulerian circuit (simple cycle)**  
**Input**
```python
nodes = ['A','B','C','D']
edges = [('A','B'),('B','C'),('C','D'),('D','A')]
run_program(nodes, edges)
```
**Possible Output**
```text
A -> B -> C -> D -> A
```
_Any rotation of the cycle (e.g., `C -> D -> A -> B -> C`) is valid._

---

**Example 2 — Eulerian trail (two odd-degree vertices)**  
**Input**
```python
nodes2 = [1,2,3,4]
edges2 = [(1,2),(2,3),(3,4)]
run_program(nodes2, edges2)
```
**Output**
```text
1 -> 2 -> 3 -> 4
```

---

**Example 3 — Another trail (path of length 2)**  
**Input**
```python
nodes3 = ['x','y','z']
edges3 = [('x','y'),('y','z')]
run_program(nodes3, edges3)
```
**Possible Output**
```text
x -> y -> z
```
_(Depending on which odd vertex is chosen as start, you may also see `z -> y -> x`.)_

---
## Notes on Determinism (Optional)

- **Start vertex** comes from a Python `set` (`active`), which is unordered → actual start can vary.
- **Neighbor selection** uses `pop()` (LIFO) from the adjacency list → path depends on input order.
- To make output deterministic, pick a specific start (e.g., the smallest label) and sort adjacency lists before traversal.

