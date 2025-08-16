# DS&A 05 — Trees & Binary Search Tree (BST)

## โครงสร้างต้นไม้
```csharp
public class TreeNode<T>
{
    public T Value;
    public TreeNode<T>? Left;
    public TreeNode<T>? Right;
    public TreeNode(T v) => Value = v;
}
```

## BST Insert / Search (int)
```csharp
public class Bst
{
    public TreeNode<int>? Root;

    public void Insert(int v)
    {
        Root = Insert(Root, v);
    }
    private TreeNode<int> Insert(TreeNode<int>? n, int v)
    {
        if (n is null) return new(v);
        if (v < n.Value) n.Left = Insert(n.Left, v);
        else if (v > n.Value) n.Right = Insert(n.Right, v);
        return n;
    }

    public bool Contains(int v)
    {
        var cur = Root;
        while (cur != null)
        {
            if (v == cur.Value) return true;
            cur = v < cur.Value ? cur.Left : cur.Right;
        }
        return false;
    }
}
```

## Traversals
```csharp
void InOrder(TreeNode<int>? n)
{
    if (n is null) return;
    InOrder(n.Left);
    Console.Write(n.Value + " ");
    InOrder(n.Right);
}
```
