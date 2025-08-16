# DS&A 07 — Heaps & PriorityQueue

.NET มี PriorityQueue<TElement,TPriority>

```csharp
var pq = new PriorityQueue<string, int>();
pq.Enqueue("low", 5);
pq.Enqueue("high", 1);
Console.WriteLine(pq.Dequeue()); // high
```

## Max-Heap แบบง่าย (array-based)
```csharp
public class MaxHeap
{
    private readonly List<int> _a = new();
    private void Swap(int i, int j) { (_a[i], _a[j]) = (_a[j], _a[i]); }

    public void Push(int v)
    {
        _a.Add(v);
        int i = _a.Count - 1;
        while (i>0)
        {
            int p=(i-1)/2; if (_a[p] >= _a[i]) break; Swap(p,i); i=p;
        }
    }

    public int Pop()
    {
        int n=_a.Count-1; Swap(0,n);
        int ret=_a[n]; _a.RemoveAt(n); Heapify(0);
        return ret;
    }

    private void Heapify(int i)
    {
        int n=_a.Count; while(true)
        {
            int l=2*i+1, r=2*i+2, m=i;
            if (l<n && _a[l]>_a[m]) m=l;
            if (r<n && _a[r]>_a[m]) m=r;
            if (m==i) break; Swap(i,m); i=m;
        }
    }
}
```
