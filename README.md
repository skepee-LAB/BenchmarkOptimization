# Benchmark Optimization

## Introduction
C# Optimizations tips and tricks.

### 1. ReadOnlySpan() instead of Substring()
Instead of using ```.Substring()``` it is suggested of using [```.AsSpan()```](https://learn.microsoft.com/en-us/dotnet/api/system.memoryextensions.asspan?view=net-7.0)
Click [here](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1846) for a detailed description
and [here](https://learn.microsoft.com/en-us/dotnet/api/system.span-1?view=net-8.0) and then [here](https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/memory-t-usage-guidelines)and [here](https://learn.microsoft.com/en-us/archive/msdn-magazine/2018/january/csharp-all-about-span-exploring-a-new-net-mainstay).


```
Span<T> is a ref struct that is allocated on the stack rather than on the managed heap. 
Ref struct types have a number of restrictions to ensure that they cannot be promoted to the managed heap, 
including that they can't be boxed, they can't be assigned to variables of type Object, dynamic or to any interface type, 
they can't be fields in a reference type, and they can't be used across await and yield boundaries. In addition, calls to two methods, 
Equals(Object) and GetHashCode, throw a NotSupportedException.
```

#### Remarks:
No memory allocation.
Looking at this code:

```
string s = "Hello world. This is a test!";
var a = s.AsSpan(0, 4);
var b = s.AsSpan(22);
```

In Visual Studio it is possible to show Memory allocation during Debug [here](https://learn.microsoft.com/en-us/visualstudio/debugger/memory-windows?view=vs-2022):

By writing ```s```  it is possible to show the memory allocated for the string:
![SubstringMemAllocation1](https://user-images.githubusercontent.com/13406481/226690281-1187573c-ece6-4a22-9ffb-92c60cc4cd66.png)

Insted by writing ```a``` or ```b``` we can see:
![SubstringMemAllocation2](https://user-images.githubusercontent.com/13406481/226690311-01d0724b-642c-4fe3-81e6-a1feac4da23e.png)

![SubstringMemAllocation3](https://user-images.githubusercontent.com/13406481/226690326-b470dfff-c014-43cc-9801-21186b05d6ec.png)

### Benchmarks

We have used [BenchmarkDotNet](https://github.com/dotnet/BenchmarkDotNet) to test the performance of using ```AsSpan()``` comparing different ways of using a ```substring``` and a ```concat```.

```c#
  [Benchmark]
  public void TestSubstring()
  {
      var res= string.Concat(test.Substring(0, 5), test.Substring(22));
  }

  [Benchmark]
  public void TestSpan()
  {
      var res = string.Concat(test.AsSpan(0, 4), test.AsSpan(22));
  }

  [Benchmark]
  public void TestSpanToString()
  {
      var res = test.AsSpan(0, 5).ToString() + test.AsSpan(22).ToString();
  }

  [Benchmark]
  public void TestSpanToStringConcat()
  {
      var res = string.Concat(test.AsSpan(0, 5).ToString(), test.AsSpan(22).ToString());
  }
```

As we can see the use of ```ReadOnlySpan``` has less memory allocation and then best performance.

|                 Method |      Mean |     Error |    StdDev |   Gen0 | Allocated |
|----------------------- |----------:|----------:|----------:|-------:|----------:|
|               TestSpan |  9.881 ns | 0.1063 ns | 0.1591 ns | 0.0076 |      48 B |
|          TestSubstring | 26.882 ns | 0.3570 ns | 0.5343 ns | 0.0191 |     120 B |
|       TestSpanToString | 27.900 ns | 0.4150 ns | 0.6083 ns | 0.0191 |     120 B |
| TestSpanToStringConcat | 28.036 ns | 0.3850 ns | 0.5762 ns | 0.0191 |     120 B |


