# Interviews Cheat Sheet

Gathered questions and answers from practing to .Net Sr. Software Engineer interviews.

## C#

### Data Structures

There are some collections on C#. They vary on complexity and use cases. For deeper understanding, consult [this link](https://learn.microsoft.com/en-us/dotnet/standard/collections/#algorithmic-complexity-of-collections).

Generic collections vs non-generic: generic typically offer better performance. We will not cover non-generic.

#### Generic collections

Before going into the collections provided by the framework, we should understand some concepts. Collections implement data structures behind the scenes, and these data structures will guide us on things like Time and Space complexity. Some of the most common data structures are:
* Array: are stored sequentially in the memory and with pre-reserved amount of data.
* Dynamic Array: just like array, is stored sequentially. The difference is greater capacity is needed, the CLR copies the entire array to a new space of memory with doubled capacity.
* Double Linked-List
* HashTable: stored using a Hash calculation, using the GetHashCode function. 
* Red-Black Tree: a structure that has great complexity for every operation.

|                        | .Net Collection                              | Add(value)   | InsertAt   | Add Beyond Capacity | Remove/RemoveAt | Item[index] | Contains(value) |
|------------------------|----------------------------------------------|--------------|------------|---------------------|-----------------|-------------|-----------------|
| **Array**              | Array                                        |     O(1)     |    O(n)    |          ?          |       O(n)      |     O(1)    |       O(n)      |
| **Dynamic Array**      | List<T>, Stack<T>, Queue<T>                  |     O(1)     |    O(n)    |         O(n)        |       O(n)      |     O(1)    |       O(n)      |
| **Double Linked-List** | LinkedList<T>                                |  O(1)/O(n) * | O(1)/O(n)* |         O(1)        |   O(1)/O(n) *   |     O(n)    |       O(n)      |
| **HashTables**         | Dictionary<TKey, TValue>, HashSet<T>         |  O(1)/O(n)*  |      -     |         O(n)        |    O(1)/O(n)*   |  O(1)/O(n)* |       O(n)      |
| **Red-black tree**     | SortedDictionary<TKey, TValue>, SortedSet<T> |   O(log n)   |  O(log n)  |       O(log n)      |     O(log n)    |   O(log n)  |     O(log n)    |

Commentaries:
* Of course Stack and Queue do not have InsertAt nor RemoveAt operations.
* Double Linked-List: * O(1) if already has a pointer to the item before/after. O(n) if must find the position
* HashTables: it's normally O(1). If there is collision on the hash value, it will store the collided items in an array (that's why O(n)). Obviously no Contains on HashSet.

#### Thread safe

It's good to know that many collection have a thread-safe or immutablew option.

Often immutable collection types are less performant but provide immutability - which is often a valid comparative benefit.

* ConcurrentDictionary<TKey, TValue>
* ReadOnlyDictionary<TKey, TValue>
* ImmutableDictionary<TKey, TValue>
* ImmutableList
* ImmutableArray
* ConcurrentStack<T>
* ConcurrentQueue<T>

and others.

Sources:
* https://github.com/RehanSaeed/.NET-Big-O-Algorithm-Complexity-Cheat-Sheet/blob/main/Cheat%20Sheet.pdf
* https://learn.microsoft.com/en-us/dotnet/standard/collections/#algorithmic-complexity-of-collections
* https://hovermind.com/csharp/runtime-complexity-of-generic-collection.html
* https://www.bigocheatsheet.com/

### Reference types/value types

1. Value Types: directly contain their data, and each instance of a value type has its own copy of the data. They are typically stored on the stack, which is a region of memory that is faster to allocate and deallocate. Examples of value types in C# include simple types like integers, float,  char, and **custom structs**.

2. Reference Types: store a reference to the memory location where the actual data is held. They don't contain the data directly and are typically stored on the heap, which is a region of memory that is more flexible but slower to allocate and deallocate. Examples of reference types in C# include classes, interfaces, **arrays**, and **strings**.

Some quick answers:
* Structs can not have a reference type on its fields.
* Strings are immutable, meaning their values cannot be changed after they are created. When you modify a string, you are actually creating a new string instance rather than modifying the existing one (that's the reason StringBuilder is better, as it uses a resizable buffer). String literals are stored in a special area of memory known as the intern pool. 
* On methods, if you do not use the **ref** keyword on the signature and it is a value type, you will pass a copy of the value. Reference types are always being passed as references.

```
// Passing by value
void ModifyValue(int num) 
{
    num = 20; // Changes inside the method do not affect the original value
}

// Passing by reference
void ModifyReferenceType(ref int[] array) 
{
    array[0] = 42; // Changes inside the method affect the original array
}
```

### Memory management

There are three topics on this section: heaps, stack and garbage collector.  To check for memory usage, one may use [memory usage](https://learn.microsoft.com/en-us/visualstudio/profiling/memory-usage?view=vs-2022) or [performance profiler](https://learn.microsoft.com/en-us/visualstudio/profiling/profiling-feature-tour?view=vs-2022#post_mortem) (both in Visual Studio). 

1. **Heaps**

It divides the heap into three segments: 0, 1 and 2 (the last one is splitted into two pieces). It organizes it into two pieces: SOH (small object heap) and LOH (large object heap):
* Large Object Heap: things that dont change (constants, static objects, etc) and objects larger than 85KB. Consists of the second piece of segment 2.
* Small Object Heap: consists of segment 0, 1 and the first piece of 2. Generally, developers only write into the 0 segment.

The segments work as a way for the Garbage Collector (GC) to clean up things. For example:
* If asked to clean up 0, 0 is going to be cleaned up
* If asked to clean up 1, 0 **and** 1 are going to be cleaned up
* If asked to clean up 2, all will be cleaned up (including LOH).

Note that objects that have a reference to itself are not going to be cleaned up.

For a object to be allocated on segment 1, GC must have ran and a reference to the objects should exist. That tells GC "this object is a little relevant" and what happens is that it updates the boundary from segment 1 to include those objects. Check the following image.

<p align="center">
  <img src="./gc_1.png" alt="Example" width="400">
</p>

If GC runs a compact strategy, it may happen it could compact segment 1, move the objects to a location where it could narrow and make the segment 1 space smaller.

For an object to be allocated on second generation segment, GC must run again and notice the object on segment 1 is still relevant and can be moved to the second one.

Developers should avoid objects to be allocated on the segment 2 for aiming for optimization. According to this [video](https://www.youtube.com/watch?v=TnDRzHZbOio), the accessing time is not the issue. The problem is with memory allocation. Allocating things in LOH costs much more than allocating in SOH. Another tip from the video is that allocating bigger objects less times is better than allocating smaller objects more time.

Challenge: write a program that adds many class to the list. Check if what makes it being moved to LOH is the size of the list (which are references to the object) or the size of the objects summed up. Check if what is moved to LOH is only the list or also the objects.


2. **Garbage collector and Dispose**:

GC works only on managed resources'; i.e. resources that are controlled by the CLR. Unmanaged resources are e.g. file handles, network connections, database connections. 

#### Managed resources

There are two modes:
* Sweep/mark: flips a bit on the object that says the memory can be occupied by something else.
* Compact: remove the objects and compact the remaining ones.

GC runs on a separate thread from the program, so the program doesnt get halted for it's execution. There is a configuration for the GC that one may change: telling the GC it's running on a workstation or on a server. If it's on a server, it's presumed the server has enough resources to the GC to run more frequently, so it won't make the program performance slower. Anyway, normally the automatic settings work ok.

#### Unmanaged resources

Differences between Finalize and Dispose and GC:
* Finalize is called by GC, so it's a last time to release resources.
* Dispose is called by developer, so it reclaims space before GC runs. Normally by using the **using** block, or by expliciting calling the Dispose method.

The best practice is to implement the IDisposable interface on classes that handle unmanaged resources, since dealing with unmanaged resources is a developer responsability (and not doing that can potentially lead to memory leaks). In addition to that, developers normally use the Finalize (aka destructor) to dispose the unmanaged resources in case the developer forgot to call the Dispose previously.

Example with the Dispose Pattern applied: 

```
public class MixedClass : IDisposable
{
    private StreamWriter _writer; // managed resource
    private Excel.Application _excel; // unmanaged resource
    private bool disposedValue;

    public void Initialize()
    {
      _writer = new StreamWriter("output_file.txt");
      _excel = new Excel.Application();
    }

    [SupportedOSPlatform("windows")]
    protected virtual void Dispose(bool disposing)
    {
        if (!disposedValue)
        {
            if (disposing)
            {
                _writer?.Dispose();
                Console.WriteLine("Disposing of writer");
            }
			
            if(_excel != null)
            {
                _excel.Quit();
                Marshal.ReleaseComObject(_excel);
                Console.WriteLine("Releasing Excel");
            }
			
            disposedValue = true;
        }
    }

    // // TODO: override finalizer only if 'Dispose(bool disposing)' has code to free unmanaged resources
    [SupportedOSPlatform("windows")]
    ~MixedClass()
    {
        // Do not change this code. Put cleanup code in 'Dispose(bool disposing)' method
        Dispose(disposing: false);
    }
    [SupportedOSPlatform("windows")]
    public void Dispose()
    {
        // Do not change this code. Put cleanup code in 'Dispose(bool disposing)' method
		    Dispose(disposing: true);
        GC.SuppressFinalize(this);
    }
}
```

### Locks

There are some lock types in C#. Here we will cover lock (monitor), mutex and semaphore/slimsemaphore.

1. **Lock keyword/monitor**: creates a basic code zone where only one thread can enter at the same time. This is precisely the same as using Monitor.Enter/Exit class. One can't use the **await** keyword on it and should not work with Tasks on it, as it would probably lead to thread starvation. Both code snippets below are equivalents:
```
lock (x)
{
    // Your code...
}
```
```
object __lockObj = x;
bool __lockWasTaken = false;
try
{
    System.Threading.Monitor.Enter(__lockObj, ref __lockWasTaken);
    // Your code...
}
finally
{
    if (__lockWasTaken) System.Threading.Monitor.Exit(__lockObj);
}
```
2. **Mutex**: can be named and shared between processes and async code (which lock keyword cannot). The mutex is provided by the OS, so a good use case is to share it between two different applications that run on the same machine. Note that there are also local mutexes (it exists on its process) and the way you use may vary from OS to OS.
```
Mutex m = new Mutex(false, "MyMutex");
        
// Try to gain control of the named mutex. If the mutex is 
// controlled by another thread, wait for it to be released.        
Console.WriteLine("Waiting for the Mutex.");
m.WaitOne();

// Keep control of the mutex until the user presses
// ENTER.
Console.WriteLine("This application owns the mutex. " +
    "Press ENTER to release the mutex and exit.");
Console.ReadLine();

m.ReleaseMutex();
```
3. **SemaphoreSlim**: It allows you to fine-tune the number of threads that can enter into the critical zone. It's a good approach for a "lock" that must done in an async manner.
```
using System;
using System.Threading;
using System.Threading.Tasks;

public class Example
{
    private static SemaphoreSlim semaphore;
    // A padding interval to make the output more orderly.
    private static int padding;

    public static void Main()
    {
        // Create the semaphore.
        semaphore = new SemaphoreSlim(0, 3);
        Console.WriteLine("{0} tasks can enter the semaphore.",
                          semaphore.CurrentCount);
        Task[] tasks = new Task[5];

        // Create and start five numbered tasks.
        for (int i = 0; i <= 4; i++)
        {
            tasks[i] = Task.Run(() =>
            {
                // Each task begins by requesting the semaphore.
                Console.WriteLine("Task {0} begins and waits for the semaphore.",
                                  Task.CurrentId);
                
                int semaphoreCount;
                semaphore.Wait();
                try
                {
                    Interlocked.Add(ref padding, 100);

                    Console.WriteLine("Task {0} enters the semaphore.", Task.CurrentId);

                    // The task just sleeps for 1+ seconds.
                    Thread.Sleep(1000 + padding);
                }
                finally {
                    semaphoreCount = semaphore.Release();
                }
                Console.WriteLine("Task {0} releases the semaphore; previous count: {1}.",
                                  Task.CurrentId, semaphoreCount);
            });
        }

        // Wait for half a second, to allow all the tasks to start and block.
        Thread.Sleep(500);

        // Restore the semaphore count to its maximum value.
        Console.Write("Main thread calls Release(3) --> ");
        semaphore.Release(3);
        Console.WriteLine("{0} tasks can enter the semaphore.",
                          semaphore.CurrentCount);
        // Main thread waits for the tasks to complete.
        Task.WaitAll(tasks);

        Console.WriteLine("Main thread exits.");
    }
}
// The example displays output like the following:
//       0 tasks can enter the semaphore.
//       Task 1 begins and waits for the semaphore.
//       Task 5 begins and waits for the semaphore.
//       Task 2 begins and waits for the semaphore.
//       Task 4 begins and waits for the semaphore.
//       Task 3 begins and waits for the semaphore.
//       Main thread calls Release(3) --> 3 tasks can enter the semaphore.
//       Task 4 enters the semaphore.
//       Task 1 enters the semaphore.
//       Task 3 enters the semaphore.
//       Task 4 releases the semaphore; previous count: 0.
//       Task 2 enters the semaphore.
//       Task 1 releases the semaphore; previous count: 0.
//       Task 3 releases the semaphore; previous count: 0.
//       Task 5 enters the semaphore.
//       Task 2 releases the semaphore; previous count: 1.
//       Task 5 releases the semaphore; previous count: 2.
//       Main thread exits.
```


#### Multi threading
.

#### Performance and monitoring tools
.

#### Others

Delegates: what is the difference between Func<T> vs ACtion<T> vs Predicate<T> 
* Func returns a type
* Action returns void
* Predicate returns boolean

Unsafe code => unaccessable for CLR "I want to read that specific part of memory".

Reflection: abuse could turn on bad performance.

ConfigureAwait(true) -> instructing the state machine to run on the same thread. 

string.IsNullOrEmpty();
string.IsNullOrWhiteSpace(); => whitespace is faster.

LINQ: Language Integrated Query is a set of technologies that aim to express queries directly in C#. On its system there is for example IEnumerable and IQueryable.
* IEnumerable: provides iteration features by returning an IEnumerator. Enumerator has MoveNext(), Reset(), Current property.
* IQueryable: inherits IEnumerable. It has several extension methods that must be implemented by the provider (IQueryableProvider) to perform queries.
* IQueryableProvider: there are several out there. For example,. LINQ to SQL, LINQ to XML, LINQ to Objects (System.Linq.Enumerable) etc. [Link](https://stackoverflow.com/questions/471502/what-is-linq-and-what-does-it-do)
* Anonymous methods: a method that accepts a lambda expression and may return an anonymous object or another object. An anonymous object is created at compile time. Example: ``` myCustomers.Select(c => new { Name = c.Name; Age = c.Age; })```

## .NET

#### Features
* Cross-platform
* Open source
* Lightweight
* High performance

#### ASP .NET
* Kestrel
* Middleware and filters
* Configuration
* Logging
* Dependency Injection: it is the ability of the framework to provide the dependencies that a certain class has. It helps mocking on tests and to follow SOLID patterns, since it's based on dependendy inversion and inversion of container principles.
* Lifetime scopes

#### Ef Core

* Tracking, 
* Lazy vs eager loading, 
* LINQ/Lambda functions
* Transactions
* Migrations and code first

#### MVC

##### Views
* Partial views vs View components: partial views work for parts of view that does not need server processing, while view components can do.
Partial view discovery:
* /Areas/<Area-Name>/Views/<Controller-Name>
* /Areas/<Area-Name>/Views/Shared
* /Views/Shared
* /Pages/Shared

* View discovery: return View(); "NameOfTheView", "../Manage/Index", "~/AbsolutePath/View.cshtml". You can customize the default convention for how views are located within the app by using a custom IViewLocationExpander.

ViewModel are strongly typed models. They are passed by the controller as a parameter (_return View(viewModel)_) and can be referenced on the view using the following syntax: _@model WebApplication1.ViewModels_. They provide validation using data annotation on the viewmodel attributes, and on the controller one can use _Model.IsValid_ to check for it. 

ViewData and ViewBag  are weakly typed model types. The ViewData property is a dictionary of weakly typed objects. The ViewBag property is a wrapper around ViewData that provides dynamic properties for the underlying ViewData collection. They are dynamically resolved at runtime. One may pass data not only between controller and view, but also between views.

ViewData:
```
Controller:

ViewData["Greeting"] = "Hello";
ViewData["Address"]  = new Address()
{
    Name = "Steve",
    Street = "123 Main St",
    City = "Hudson",
    State = "OH",
    PostalCode = "44236"
};

View:
@{
    // Since Address isn't a string, it requires a cast.
    var address = ViewData["Address"] as Address;
}

@ViewData["Greeting"] World!

<address>
    @address.Name<br>
    @address.Street<br>
    @address.City, @address.State @address.PostalCode
</address>
```

ViewBag:
```
Controller:

ViewBag.Greeting = "Hello";
ViewBag.Address  = new Address()
{
    Name = "Steve",
    Street = "123 Main St",
    City = "Hudson",
    State = "OH",
    PostalCode = "44236"
};

View:
@ViewBag.Greeting World!

<address>
    @ViewBag.Address.Name<br>
    @ViewBag.Address.Street<br>
    @ViewBag.Address.City, @ViewBag.Address.State @ViewBag.Address.PostalCode
</address>

```


#### Others

* AOT
* Reflection

## SQL

#### Query performance
.

#### Index creation

Clustered vs non-clustered

#### Transactions
.

## Git

#### Rebase
.

#### Merge
.

#### Cherry pick
.

## Software Engineering

#### Design Patterns

**Singleton**: a single instance of a determined class throughout the project.
**Facade**: a "big interface" that normally has several adapters related to a specific task.
**Factory**: a class that can generate other classes. One use case is that a factory is provided some parameters and return an Interface object which may be any object that implement it.
**DDD**: separation of concerns.

#### Quality 

Code quality:
* DRY, KISS, YAGNI, SOLID
* High modular, decoupled code.
* Variable naming.
* Test coverage & test types

Workflow quality:
* Pair programming
* Code review & pull requests
* Feature branches
* Different environments
* CI/CD pipeline
* Scrum/Kanbam

#### Security

How would you secure the apis? Think of approaches.
* Authentication with JWT and Authorization with Attributes
* SSL certificates (HTTPS)
* Parameter validation with FluentValidation
* Parameter encoding
* Logging injection
* Using secure libraries, not using OS commands, validating when downloading or accessing external resource
* Sql Injection
* Misconfiguration, exposed
* Hash + salt for passwords

