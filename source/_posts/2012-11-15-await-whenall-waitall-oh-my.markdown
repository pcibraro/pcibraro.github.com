---
layout: post
title: "await, WhenAll, WaitAll, oh my!!"
date: 2012-11-15
comments: true
categories: .NET
---

If you are dealing with asynchronous work in .NET, you might know that
the Task class has become the main driver for wrapping asynchronous
calls. Although this class was officially introduced in .NET 4.0, the
programming model for consuming tasks was much more simplified in C\#
5.0 in .NET 4.5 with the addition of the new async/await keywords. In a
nutshell, you can use these keywords to make asynchronous calls as if
they were sequential, and avoiding in that way any fork or callback in
the code. The compiler takes care of the rest.

I was yesterday writing some code for making multiple asynchronous calls
to backend services in parallel. The code looked as follow,

    var allResults = new List<Result>();

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
foreach(var provider in providers)
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
{
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
  var results = await provider.GetResults();
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
  allResults.AddRange(results);
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
}
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
return allResults;
~~~~

You see, I was using the await keyword to make multiple calls in
parallel. Something I did not consider was the overhead this code
implied after being compiled. I started an interesting discussion with
some smart folks in twitter. One of them, [Tugberk
Ugurlu](https://twitter.com/tourismgeek), had the brilliant idea of
actually write some code to make a performance comparison with another
approach using Task.WhenAll.

There are two additional methods you can use to wait for the results of
multiple calls in parallel, WhenAll and WaitAll.

WhenAll creates a new task and waits for results in that new task, so it
does not block the calling thread. WaitAll, on the other hand, blocks
the calling thread. This is the code Tugberk initially wrote, and I
modified afterwards to also show the results of WaitAll.

     class Program

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
    {
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
        private static Func<Stopwatch, Task>[] funcs = new Func<Stopwatch, Task>[] { 
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
            async (watch) => { watch.Start(); await Task.Delay(1000); 
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
                Console.WriteLine("1000 one has been completed."); },
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
            async (watch) => { await Task.Delay(1500); 
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
                Console.WriteLine("1500 one has been completed."); },
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
            async (watch) => { await Task.Delay(2000); 
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
                Console.WriteLine("2000 one has been completed."); watch.Stop(); 
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
                Console.WriteLine(watch.ElapsedMilliseconds + "ms has been elapsed."); }
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
        };
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
        static void Main(string[] args)
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
        {
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
            Console.WriteLine("Await in loop work starts...");
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
            DoWorkAsync().ContinueWith(task =>
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
            {
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
                Console.WriteLine("Parallel work starts...");
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
                DoWorkInParallelAsync().ContinueWith(t =>
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
                    {
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
                        Console.WriteLine("WaitAll work starts...");
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
                        WaitForAll();
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
                    });
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
            });
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
            Console.ReadLine();
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
        }
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
        static async Task DoWorkAsync()
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
        {
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
            Stopwatch watch = new Stopwatch();
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
            foreach (var func in funcs)
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
            {
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
                await func(watch);
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
            }
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
        }
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
        static async Task DoWorkInParallelAsync()
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
        {
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
            Stopwatch watch = new Stopwatch();
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
            await Task.WhenAll(funcs[0](watch), funcs[1](watch), funcs[2](watch));
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
        }
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
        static void WaitForAll()
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
        {
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
            Stopwatch watch = new Stopwatch();
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
            Task.WaitAll(funcs[0](watch), funcs[1](watch), funcs[2](watch));
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
        }
~~~~

~~~~ {style="font-size: 12px; font-family: consolas,'Courier New',courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"}
    }
~~~~

After running this code, the results were very concluding.

Await in loop work starts... \
1000 one has been completed. \
1500 one has been completed. \
2000 one has been completed. \
4532ms has been elapsed.

\
Parallel work starts... \
1000 one has been completed. \
1500 one has been completed. \
2000 one has been completed. \
2007ms has been elapsed. \

WaitAll work starts... \
1000 one has been completed. \
1500 one has been completed. \
2000 one has been completed. \
2009ms has been elapsed.

The await keyword in a loop does not really make the calls in parallel.

