# NthItemUtils

A library for getting the nth smallest value, the nth largest value, etc. from a randomly accessible data structure like `Span<T>`,`ReadOnlySpan<T>`, `IReadOnlyList<T>`.
Internally, it uses QuickSelect as well as C++ `std::nth_element()`, so the average performance is O(n).

The source code consists of **only one file**, so you can easily use it by copy and past.

[source code](https://github.com/nenoNaninu/NthItemUtils/blob/master/NthItemUtils/NthItemUtils.cs)

or [nuget](https://www.nuget.org/packages/NthItemUtils/).
```
dotnet add package NthItemUtils
```

# API
API is defined as an extension method of `Span<T>`, `ReadOnlySpan<T>`, and `IReadOnlyList<T>`.
The range of n is between 0 and source.Length - 1.
All return value types are `struct ItemWithIndex<T>{ T Item; int index }`.
```cs
NthSmallest<T>(this Span<T> source, int n)
NthSmallest<T>(this Span<T> source, int n, Comparer<T> comparer)

NthLargest<T>(this Span<T> source, int n)
NthLargest<T>(this Span<T> source, int n, Comparer<T> comparer)

MaxWithIndex<T>(this Span<T> source)
MaxWithIndex<T>(this Span<T> source, Comparer<T> comparer)

MinWithIndex<T>(this Span<T> source)
MinWithIndex<T>(this Span<T> source, Comparer<T> comparer)
```

Since QuickSelect is used internally, this is also published as an API.


```cs
public static class QuickSelect
{
    public static void Iota(Span<int> indices);
    
    public static void Execute<T>(ReadOnlySpan<T> source, Span<int> indices, int n);
    public static void Execute<T>(ReadOnlySpan<T> source, Span<int> indices, int n, Comparer<T> comparer);
    
    public static void Execute<T>(IReadOnlyList<T> source, Span<int> indices, int n);
    public static void Execute<T>(IReadOnlyList<T> source, Span<int> indices, int n, Comparer<T> comparer);
}

```
# Example
Extension Method.
```cs
var random = new Random();
var randomSource = Enumerable.Range(0, 200).Select(_ => random.NextDouble() * 50).ToArray();

var order = randomSource.OrderBy(x => x).ToArray();

int n = random.Next(200);

Assert.IsTrue(order[n] == randomSource.AsSpan().NthSmallest(n).Item); // always true.
```

QuickSelect

```cs
ReadOnlySpan<int> source = new int[] { 4, 4, 8, 1, 2, 5, 4, 4, 4, 6, 7, 3 }.AsSpan();
var pool = ArrayPool<int>.Shared.Rent(source.Length);
var indices = pool.AsSpan(0, source.Length);

int n = 6;

QuickSelect.Iota(indices);
QuickSelect.Execute(source, indices, n);

var pivot = source[indices[n]];
for (int i = 0; i < n; i++)
{
    var result = Comparer<double>.Default.Compare(source[indices[i]], pivot);

    Assert.IsTrue(result <= 0); // always true.
}

for (int i = n + 1; i < source.Length; i++)
{
    var result = Comparer<double>.Default.Compare(source[indices[i]], pivot);

    Assert.IsTrue(0 <= result); // always true.
}

ArrayPool<int>.Shared.Return(pool);
```
Get items in some range
```cs
ReadOnlySpan<int> randomSource = Enumerable.Range(0, 100).Shuffle().ToArray().AsSpan();
var pool = ArrayPool<int>.Shared.Rent(randomSource.Length);
var indices = pool.AsSpan(0, randomSource.Length);

QuickSelect.Iota(indices);

//Get 50 <= item < 60
QuickSelect.Execute(randomSource, indices, 50);
QuickSelect.Execute(randomSource, indices[50..], 10);

for (int i = 0; i < 10; i++)
{
    Console.Write($"{randomSource[indices[50 + i]]}, ");
}
// output : 50, 56, 51, 57, 55, 54, 53, 52, 58, 59, 

ArrayPool<int>.Shared.Return(pool);
```