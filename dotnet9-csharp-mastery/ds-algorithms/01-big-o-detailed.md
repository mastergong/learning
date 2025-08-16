# DS&A 01 — Big-O และการวิเคราะห์ประสิทธิภาพของอัลกอริทึม

## 🎯 เป้าหมายการเรียนรู้
เมื่อเสร็จสิ้นบทเรียนนี้ คุณจะสามารถ:
- เข้าใจหลักการและความสำคัญของ Big-O Notation
- วิเคราะห์ความซับซ้อนด้านเวลา (Time Complexity) และหน่วยความจำ (Space Complexity) ของอัลกอริทึม
- เปรียบเทียบประสิทธิภาพของอัลกอริทึมที่แตกต่างกัน
- วิเคราะห์และปรับปรุงโค้ดให้มีประสิทธิภาพดีขึ้น

## 📝 Big-O Notation คืออะไร?

Big-O Notation เป็นวิธีทางคณิตศาสตร์ที่ใช้อธิบายพฤติกรรมของอัลกอริทึมเมื่อขนาดของข้อมูลนำเข้า (n) เพิ่มขึ้น โดยเฉพาะอย่างยิ่งเมื่อ n มีค่ามากๆ การวิเคราะห์ Big-O จะมุ่งเน้นที่ "worst-case scenario" หรือกรณีที่แย่ที่สุดที่อาจเกิดขึ้น

### เหตุผลที่ต้องใช้ Big-O
1. **เปรียบเทียบอัลกอริทึม**: ช่วยให้เราสามารถเปรียบเทียบประสิทธิภาพของอัลกอริทึมต่างๆ ได้โดยไม่ต้องวัดเวลาจริง
2. **เป็นอิสระจากฮาร์ดแวร์**: ไม่ขึ้นกับความเร็วของคอมพิวเตอร์หรือภาษาโปรแกรมมิ่ง
3. **คาดการณ์ความสามารถในการขยายตัว (Scalability)**: ช่วยให้เราเข้าใจว่าอัลกอริทึมจะทำงานอย่างไรเมื่อขนาดข้อมูลเพิ่มขึ้นมากๆ

## 📊 ระดับความซับซ้อนของ Big-O ที่พบบ่อย

### Time Complexity (ความซับซ้อนด้านเวลา)

| Big-O | ชื่อ | คำอธิบาย | ตัวอย่าง |
|-------|-----|----------|---------|
| O(1) | Constant Time | เวลาคงที่ไม่ขึ้นกับขนาดข้อมูล | การเข้าถึงอาร์เรย์โดยตรง, การทำงานกับค่าคงที่ |
| O(log n) | Logarithmic Time | เวลาเติบโตแบบ logarithm ของขนาดข้อมูล | Binary Search, Balanced BST operations |
| O(n) | Linear Time | เวลาเติบโตเป็นเส้นตรงตามขนาดข้อมูล | การอ่านอาร์เรย์ครบทุกตัว, Linear Search |
| O(n log n) | Linearithmic Time | เวลาเติบโตแบบ n * log(n) | Merge Sort, Quick Sort, Heap Sort |
| O(n²) | Quadratic Time | เวลาเติบโตเป็นกำลังสองของขนาดข้อมูล | Bubble Sort, Selection Sort, Nested loops |
| O(n³) | Cubic Time | เวลาเติบโตเป็นกำลังสามของขนาดข้อมูล | หลาย Nested loops, บางอัลกอริทึมเกี่ยวกับเมทริกซ์ |
| O(2ⁿ) | Exponential Time | เวลาเติบโตเป็นเลขยกกำลังตามขนาดข้อมูล | Recursive Fibonacci, Tower of Hanoi |
| O(n!) | Factorial Time | เวลาเติบโตแบบแฟกทอเรียลของขนาดข้อมูล | Traveling Salesman Problem (brute force) |

### Space Complexity (ความซับซ้อนด้านหน่วยความจำ)

Space Complexity อธิบายปริมาณหน่วยความจำที่อัลกอริทึมใช้เมื่อขนาดข้อมูลนำเข้าเพิ่มขึ้น:

- **O(1)**: ใช้หน่วยความจำคงที่ไม่ว่าขนาดข้อมูลจะเป็นเท่าไร
- **O(n)**: ใช้หน่วยความจำเพิ่มขึ้นตามขนาดข้อมูล
- **O(n²)**: ใช้หน่วยความจำเพิ่มขึ้นเป็นกำลังสองของขนาดข้อมูล

## 💻 การวิเคราะห์อัลกอริทึมด้วย Big-O ใน C#

### 1. O(1) - Constant Time
```csharp
// O(1) - เวลาคงที่ ไม่ขึ้นกับขนาดข้อมูล
public int GetFirst(int[] array)
{
    return array[0]; // เข้าถึงตัวแรกใช้เวลาเท่ากันเสมอ
}

public bool IsEven(int number)
{
    return number % 2 == 0; // การคำนวณใช้เวลาเท่ากันเสมอ
}
```

### 2. O(log n) - Logarithmic Time
```csharp
// O(log n) - การค้นหาแบบ Binary Search
public int BinarySearch(int[] sortedArray, int target)
{
    int left = 0;
    int right = sortedArray.Length - 1;
    
    while (left <= right)
    {
        int mid = left + (right - left) / 2;
        
        if (sortedArray[mid] == target)
            return mid;
        
        if (sortedArray[mid] < target)
            left = mid + 1;
        else
            right = mid - 1;
    }
    
    return -1; // ไม่พบ
}
```

### 3. O(n) - Linear Time
```csharp
// O(n) - การอ่านข้อมูลครบทุกตัว
public int Sum(int[] array)
{
    int sum = 0;
    for (int i = 0; i < array.Length; i++)
    {
        sum += array[i];
    }
    return sum;
}

// O(n) - การค้นหาแบบ Linear Search
public int FindIndex(int[] array, int target)
{
    for (int i = 0; i < array.Length; i++)
    {
        if (array[i] == target)
            return i;
    }
    return -1;
}
```

### 4. O(n log n) - Linearithmic Time
```csharp
// O(n log n) - Merge Sort
public int[] MergeSort(int[] array)
{
    if (array.Length <= 1)
        return array;
        
    int middle = array.Length / 2;
    int[] left = new int[middle];
    int[] right = new int[array.Length - middle];
    
    Array.Copy(array, 0, left, 0, middle);
    Array.Copy(array, middle, right, 0, array.Length - middle);
    
    left = MergeSort(left);
    right = MergeSort(right);
    
    return Merge(left, right);
}

private int[] Merge(int[] left, int[] right)
{
    int[] result = new int[left.Length + right.Length];
    int i = 0, j = 0, k = 0;
    
    while (i < left.Length && j < right.Length)
    {
        if (left[i] <= right[j])
        {
            result[k++] = left[i++];
        }
        else
        {
            result[k++] = right[j++];
        }
    }
    
    while (i < left.Length)
    {
        result[k++] = left[i++];
    }
    
    while (j < right.Length)
    {
        result[k++] = right[j++];
    }
    
    return result;
}
```

### 5. O(n²) - Quadratic Time
```csharp
// O(n²) - Bubble Sort
public void BubbleSort(int[] array)
{
    for (int i = 0; i < array.Length; i++)
    {
        for (int j = 0; j < array.Length - 1 - i; j++)
        {
            if (array[j] > array[j + 1])
            {
                // Swap
                int temp = array[j];
                array[j] = array[j + 1];
                array[j + 1] = temp;
            }
        }
    }
}

// O(n²) - หาทุกคู่ที่เป็นไปได้
public void FindAllPairs(int[] array)
{
    for (int i = 0; i < array.Length; i++)
    {
        for (int j = 0; j < array.Length; j++)
        {
            if (i != j)
            {
                Console.WriteLine($"({array[i]}, {array[j]})");
            }
        }
    }
}
```

### 6. O(2ⁿ) - Exponential Time
```csharp
// O(2ⁿ) - Recursive Fibonacci
public int Fibonacci(int n)
{
    if (n <= 1)
        return n;
        
    return Fibonacci(n - 1) + Fibonacci(n - 2);
}
```

## 📏 การวัดประสิทธิภาพของโค้ด C# ในทางปฏิบัติ

### การใช้ Stopwatch เพื่อวัดเวลาการทำงาน
```csharp
using System;
using System.Diagnostics;
using System.Linq;

class Program
{
    static void Main()
    {
        // สร้างข้อมูลทดสอบ
        int size = 100_000;
        int[] array = Enumerable.Range(0, size).ToArray();
        
        // วัดเวลาการทำงานของ Linear Search (O(n))
        Console.WriteLine("Linear Search Test:");
        MeasureExecutionTime(() => LinearSearch(array, size - 1), "Best Case (Last Item)");
        MeasureExecutionTime(() => LinearSearch(array, -1), "Worst Case (Not Found)");
        
        // วัดเวลาการทำงานของ Binary Search (O(log n))
        Console.WriteLine("\nBinary Search Test:");
        MeasureExecutionTime(() => BinarySearch(array, size / 2), "Middle Item");
        MeasureExecutionTime(() => BinarySearch(array, -1), "Not Found");
        
        // วัดเวลาการทำงานของ Bubble Sort (O(n²))
        Console.WriteLine("\nSorting Test:");
        int[] randomArray = GetRandomArray(size);
        MeasureExecutionTime(() => BubbleSortTest(randomArray.ToArray()), "Bubble Sort O(n²)");
        MeasureExecutionTime(() => Array.Sort(randomArray.ToArray()), "Array.Sort O(n log n)");
    }
    
    static void MeasureExecutionTime(Action action, string description)
    {
        GC.Collect(); // ทำ garbage collection ก่อนวัดเวลา
        
        Stopwatch stopwatch = new Stopwatch();
        stopwatch.Start();
        
        action();
        
        stopwatch.Stop();
        Console.WriteLine($"{description}: {stopwatch.ElapsedMilliseconds} ms");
    }
    
    static int LinearSearch(int[] array, int target)
    {
        for (int i = 0; i < array.Length; i++)
        {
            if (array[i] == target)
                return i;
        }
        return -1;
    }
    
    static int BinarySearch(int[] sortedArray, int target)
    {
        int left = 0;
        int right = sortedArray.Length - 1;
        
        while (left <= right)
        {
            int mid = left + (right - left) / 2;
            
            if (sortedArray[mid] == target)
                return mid;
            
            if (sortedArray[mid] < target)
                left = mid + 1;
            else
                right = mid - 1;
        }
        
        return -1;
    }
    
    static void BubbleSortTest(int[] array)
    {
        for (int i = 0; i < array.Length; i++)
        {
            for (int j = 0; j < array.Length - 1 - i; j++)
            {
                if (array[j] > array[j + 1])
                {
                    int temp = array[j];
                    array[j] = array[j + 1];
                    array[j + 1] = temp;
                }
            }
        }
    }
    
    static int[] GetRandomArray(int size)
    {
        Random rand = new Random(42); // ใช้ seed เดียวกันเพื่อผลลัพธ์ที่เหมือนกัน
        return Enumerable.Range(0, size)
            .Select(i => rand.Next(0, size * 10))
            .ToArray();
    }
}
```

## 🔍 วิธีการปรับปรุงประสิทธิภาพของโค้ด C#

### 1. เลือกโครงสร้างข้อมูลที่เหมาะสม
- ใช้ `Dictionary<TKey, TValue>` หรือ `HashSet<T>` แทน `List<T>` สำหรับการค้นหาที่รวดเร็ว (O(1) vs O(n))
- ใช้ `SortedDictionary<TKey, TValue>` หรือ `SortedSet<T>` สำหรับข้อมูลที่ต้องการเรียงลำดับ (O(log n))

### 2. ลดการจัดสรรหน่วยความจำที่ไม่จำเป็น
- ใช้ `Span<T>` หรือ `Memory<T>` เพื่อหลีกเลี่ยง memory allocations
- หลีกเลี่ยง Boxing/Unboxing โดยใช้ Generics และ Value Types

```csharp
// แย่: Boxing/Unboxing
ArrayList list = new ArrayList();
list.Add(5); // Boxing เกิดขึ้นที่นี่
int number = (int)list[0]; // Unboxing เกิดขึ้นที่นี่

// ดี: ใช้ Generic List
List<int> genericList = new List<int>();
genericList.Add(5); // ไม่มี boxing
int genericNumber = genericList[0]; // ไม่มี unboxing
```

### 3. ใช้ Algorithms ที่มีประสิทธิภาพกว่า
- แทนที่ O(n²) algorithms ด้วย O(n log n) หรือดีกว่า
- ใช้ Memoization เพื่อเก็บผลลัพธ์ที่คำนวณแล้วในฟังก์ชันเรียกซ้ำ

```csharp
// แย่: O(2ⁿ) recursive Fibonacci
public int FibonacciSlow(int n)
{
    if (n <= 1) return n;
    return FibonacciSlow(n - 1) + FibonacciSlow(n - 2);
}

// ดี: O(n) dynamic programming approach
public int FibonacciFast(int n)
{
    if (n <= 1) return n;
    
    int[] fib = new int[n + 1];
    fib[0] = 0;
    fib[1] = 1;
    
    for (int i = 2; i <= n; i++)
    {
        fib[i] = fib[i - 1] + fib[i - 2];
    }
    
    return fib[n];
}
```

### 4. ใช้ Parallel Processing เมื่อเหมาะสม
```csharp
// แบบ Sequential
public long SumArray(int[] array)
{
    long sum = 0;
    for (int i = 0; i < array.Length; i++)
    {
        sum += array[i];
    }
    return sum;
}

// แบบ Parallel
public long ParallelSumArray(int[] array)
{
    return array.AsParallel().Sum(x => (long)x);
}
```

## 📊 กราฟเปรียบเทียบความซับซ้อนของ Big-O

```
              ↑
              │
              │  O(n!)
              │   ↗
              │  ↗
              │ ↗  O(2ⁿ)
              │↗   ↗
 Execution    │   ↗
   Time       │  ↗
              │ ↗   O(n²)
              │↗   ↗
              │   ↗
              │  ↗     O(n log n)
              │ ↗      ↗
              │↗      ↗
              │      ↗        O(n)
              │     ↗        ↗
              │    ↗       ↗       O(log n)
              │   ↗     ↗         ↗
              │  ↗   ↗          ↗         O(1)
              │↗  ↗           ↗          ────────→
              └───────────────────────────────→
                                Input Size (n)
```

## 📝 แบบฝึกหัด

### Exercise 1: วิเคราะห์ความซับซ้อนของ Code
วิเคราะห์ Time Complexity และ Space Complexity ของฟังก์ชันต่อไปนี้:

```csharp
public int Mystery1(int n)
{
    int result = 0;
    for (int i = 0; i < n; i++)
    {
        result += i;
    }
    return result;
}

public void Mystery2(int n)
{
    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < n; j++)
        {
            Console.WriteLine($"{i}, {j}");
        }
    }
}

public int Mystery3(int n)
{
    if (n <= 0) return 0;
    return n + Mystery3(n - 1);
}

public bool Mystery4(int[] sortedArray, int target)
{
    int left = 0;
    int right = sortedArray.Length - 1;
    
    while (left <= right)
    {
        int mid = left + (right - left) / 2;
        if (sortedArray[mid] == target) return true;
        if (sortedArray[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    
    return false;
}
```

### Exercise 2: ปรับปรุง Algorithm
ปรับปรุงฟังก์ชัน FindDuplicates ต่อไปนี้ให้มีประสิทธิภาพดีขึ้นจาก O(n²) เป็น O(n):

```csharp
// Original function: O(n²)
public List<int> FindDuplicates(int[] array)
{
    List<int> duplicates = new List<int>();
    
    for (int i = 0; i < array.Length; i++)
    {
        for (int j = i + 1; j < array.Length; j++)
        {
            if (array[i] == array[j] && !duplicates.Contains(array[i]))
            {
                duplicates.Add(array[i]);
                break;
            }
        }
    }
    
    return duplicates;
}
```

## 🔑 เคล็ดลับสำคัญ
- **ลำดับความสำคัญ**: เวลา (Time) > หน่วยความจำ (Space) > ความซับซ้อนของโค้ด (Code Complexity)
- **เลือกโครงสร้างข้อมูลให้เหมาะสม**: การเลือกโครงสร้างข้อมูลที่เหมาะสมสามารถลดความซับซ้อนจาก O(n) ไปเป็น O(1) หรือ O(log n) ได้
- **รู้ข้อจำกัด**: เข้าใจว่าอัลกอริทึมของคุณจะทำงานกับข้อมูลขนาดใหญ่เพียงใด
- **ทดสอบด้วยข้อมูลจริง**: ทำการทดสอบด้วยข้อมูลขนาดใหญ่เพื่อยืนยันผลการวิเคราะห์ Big-O ในทางทฤษฎี
- **อย่า Optimize ก่อนเวลาอันควร**: "Premature optimization is the root of all evil" - Donald Knuth

## 📚 แหล่งข้อมูลเพิ่มเติม
- [Big-O Cheat Sheet](https://www.bigocheatsheet.com/)
- [Microsoft .NET Performance Tips](https://docs.microsoft.com/en-us/dotnet/framework/performance/performance-tips)
- [Data Structures and Algorithms in C#](https://www.amazon.com/Data-Structures-Algorithms-Using-Beginning/dp/0470878207)
- [C# Performance Tricks](https://github.com/dotnet/performance/tree/master/docs)

## 🚀 ต่อไป
- [Arrays & Lists](./02-arrays-lists.md) - โครงสร้างข้อมูลพื้นฐานที่ใช้บ่อย
