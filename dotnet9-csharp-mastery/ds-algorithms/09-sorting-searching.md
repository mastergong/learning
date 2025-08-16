# DS&A 09 — Sorting & Searching

## Binary Search
```csharp
int BinarySearch(int[] a, int target)
{
    int lo=0, hi=a.Length-1;
    while (lo<=hi)
    {
        int mid = lo + ((hi-lo)>>1);
        if (a[mid]==target) return mid;
        if (a[mid]<target) lo=mid+1; else hi=mid-1;
    }
    return -1;
}
```

## Merge Sort
```csharp
void MergeSort(int[] a)
{
    var buf = new int[a.Length];
    void Sort(int l, int r)
    {
        if (l>=r) return;
        int m=(l+r)/2; Sort(l,m); Sort(m+1,r);
        int i=l,j=m+1,k=l;
        while(i<=m&&j<=r) buf[k++]= a[i]<=a[j]?a[i++]:a[j++];
        while(i<=m) buf[k++]=a[i++];
        while(j<=r) buf[k++]=a[j++];
        for(int t=l;t<=r;t++) a[t]=buf[t];
    }
    Sort(0,a.Length-1);
}
```
