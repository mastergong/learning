# DS&A 10 — Dynamic Programming (Intro)

แนวคิด: แก้ปัญหาใหญ่ด้วยผลลัพธ์ย่อยที่ทับซ้อน (overlapping subproblems) + โครงสร้างเหมาะย่อย (optimal substructure)

## Fibonacci — Top-down (Memoization)
```csharp
int Fib(int n, Dictionary<int,int> memo)
{
    if (n<=1) return n;
    if (memo.TryGetValue(n, out var v)) return v;
    v = Fib(n-1, memo) + Fib(n-2, memo);
    memo[n] = v; return v;
}
```

## Fibonacci — Bottom-up (Tabulation)
```csharp
int FibTab(int n)
{
    if (n<=1) return n;
    int a=0,b=1; for(int i=2;i<=n;i++){ (a,b)=(b,a+b);} return b;
}
```

## 0/1 Knapsack (น้ำหนัก-มูลค่า)
```csharp
int Knapsack(int[] w, int[] v, int W)
{
    int n=w.Length; var dp = new int[n+1, W+1];
    for(int i=1;i<=n;i++)
        for(int cap=0;cap<=W;cap++)
            dp[i,cap] = w[i-1] > cap
                ? dp[i-1,cap]
                : Math.Max(dp[i-1,cap], v[i-1] + dp[i-1,cap - w[i-1]]);
    return dp[n,W];
}
```
