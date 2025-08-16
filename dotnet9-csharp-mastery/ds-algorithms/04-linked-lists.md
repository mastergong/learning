# DS&A 04 — Linked List (Singly/Doubly)

## โหนดและลิสต์แบบ Singly
```csharp
public class Node<T>
{
    public T Value;
    public Node<T>? Next;
    public Node(T v) => Value = v;
}

public class SinglyLinkedList<T>
{
    public Node<T>? Head;

    public void AddFirst(T v)
    {
        var n = new Node<T>(v) { Next = Head };
        Head = n;
    }

    public void AddLast(T v)
    {
        var n = new Node<T>(v);
        if (Head is null) { Head = n; return; }
        var cur = Head;
        while (cur.Next != null) cur = cur.Next;
        cur.Next = n;
    }
}
```

## Reverse Linked List
```csharp
Node<T>? Reverse<T>(Node<T>? head)
{
    Node<T>? prev = null, cur = head;
    while (cur != null)
    {
        var next = cur.Next;
        cur.Next = prev;
        prev = cur;
        cur = next;
    }
    return prev;
}
```
