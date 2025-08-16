# DS&A 08 — Graphs, BFS/DFS

## กราฟแบบ AdjList
```csharp
public class Graph
{
    private readonly Dictionary<int, List<int>> _adj = new();
    public void AddEdge(int u, int v)
    {
        if (!_adj.ContainsKey(u)) _adj[u] = new();
        if (!_adj.ContainsKey(v)) _adj[v] = new();
        _adj[u].Add(v); _adj[v].Add(u);
    }

    public IEnumerable<int> Bfs(int s)
    {
        var q = new Queue<int>(); var seen = new HashSet<int>();
        q.Enqueue(s); seen.Add(s);
        while (q.Count>0)
        {
            int u=q.Dequeue(); yield return u;
            foreach (var v in _adj[u]) if (seen.Add(v)) q.Enqueue(v);
        }
    }

    public IEnumerable<int> Dfs(int s)
    {
        var st = new Stack<int>(); var seen = new HashSet<int>();
        st.Push(s);
        while (st.Count>0)
        {
            int u=st.Pop(); if (!seen.Add(u)) continue; yield return u;
            foreach (var v in _adj[u]) st.Push(v);
        }
    }
}
```
