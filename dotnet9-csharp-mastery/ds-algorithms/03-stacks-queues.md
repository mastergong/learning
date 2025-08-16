# DS&A 03 — Stack และ Queue (C#)

## Stack<T>
- LIFO, Push/Pop O(1)

```csharp
var stack = new Stack<int>();
stack.Push(1); stack.Push(2); stack.Push(3);
Console.WriteLine(stack.Pop()); // 3
```

## Queue<T>
- FIFO, Enqueue/Dequeue O(1)

```csharp
var q = new Queue<string>();
q.Enqueue("A"); q.Enqueue("B"); q.Enqueue("C");
Console.WriteLine(q.Dequeue()); // A
```

## Monotonic Stack ตัวอย่าง (หาค่าใกล้กว่าใหญ่กว่า)
```csharp
int[] NextGreater(int[] nums)
{
    var res = new int[nums.Length];
    Array.Fill(res, -1);
    var st = new Stack<int>(); // index
    for (int i = 0; i < nums.Length; i++)
    {
        while (st.Count > 0 && nums[i] > nums[st.Peek()])
        {
            var idx = st.Pop();
            res[idx] = nums[i];
        }
        st.Push(i);
    }
    return res;
}
```
