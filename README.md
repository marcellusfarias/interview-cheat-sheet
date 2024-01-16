# Interviews Cheat Sheet

Gathered questions and answers from practing to .Net Sr. Software Engineer interviews.

## C#

#### Data Structures

List, array, dictionary.

#### Reference types/value types
.

#### Memory management

Heap/stack, garbage collector

#### Locks

There are some lock types in C#.

* Lock keyword/monitor: Creates a basic code zone where only one thread can enter at the same time. This is precisely the same as using Monitor.Enter/Exit class. One can't use the **await** keyword on it and should not work with Tasks on it, as it would probably lead to thread starvation. Both code snippets below are equivalents:
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
* Mutex: can be named and shared between processes and async code (which lock keyword cannot). The mutex is provided by the OS, so a good use case is to share it between two different applications that run on the same machine. Note that there are also local mutexes (it exists on its process) and the way you use may vary from OS to OS.
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

* SemaphoreSlim: It allows you to fine-tune the number of threads that can enter into the critical zone. It's a good approach for a "lock" that must done in an async manner.
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
