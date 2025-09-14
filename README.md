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
```

