---
published: false
title: "Wanted dead or alive: Middleware (part I)"
layout: post
---

## Introduction

Who never invoked a middleware may throw the first stone here.
I call it a middleware any piece of software that runs anywhere and anyhow, and was made by any. I don't need to know how it's made, just that I need to invoke a service and it gives me an answer, sometimes...

Let's focus on that sometimes.
- Sometimes its being deployed a new version and the service is down.
- Sometimes there is a bug, and the service blow up.
- Sometimes the server is so overloaded and the service times out.
- Sometimes we lose network and we can't reach the service.
- Well... there's probably like 1001 reasons why some service fails giving us a proper response.

## Can we do something about it?

Sure we can. Most of the errors are transient. Ok, we failed the invocation, shall we try again?
This is the right answer. Let's try again, up to a number of tries. If the problem was random and/or caused by a spike, invoking the service more than once can improve us some success rate.
NOTE: please keep in mind that one of the best practices for distributed systems is to allow message retries without error.

## A naive example

Let's simulate a faulty service:

    public class Service
    {
        private static readonly Random Rand = new Random();

        public string GetMyDate(DateTime dateTime)
        {
            if (Rand.NextDouble() <= .3)
            {
                throw new Exception("Fault!");
            }
            return string.Format("My date is {0}", dateTime);
        }
    }

This fake service will randomly blow up 30% of its invocations. This isn't perfect, but it suits our needs.
Let's look at the client how will consume this service.

    public class NaiveClient
    {
        private StatsCounter _counter;

        public NaiveClient(StatsCounter counter)
        {
            _counter = counter;
        }

        public string GetMyDate(DateTime date)
        {
            var faultService = new Service();
            try
            {
                _counter.TotalExecutions++;
                var mydate = faultService.GetMyDate(date);
                _counter.ExecutionSuccess++;
                return mydate;
            }
            catch (Exception)
            {                
                _counter.ExecutionError++;
                throw;
            }
        }
    }

We're passing a StatsCounter object on the constructor just to get some stats on the end of our demo. This client only invokes the services and keeps track of every execution, and which resulted in success or error.

And finally, here is our test program. Let's simulate 1000 executions. It's really not a big number, but it's enough to get us some data.

    class Program
    {
        const int TimesToInvoke = 1000;

        static void Main(string[] args)
        {
            var kernel = new StandardKernel();
            kernel.Load(new Module());

            var counter = kernel.Get<StatsCounter>();
            var client = kernel.Get<INaiveClient>();
            counter.Stopwatch.Start();
            for (var i = 0; i < TimesToInvoke; i++)
            {
                try
                {
                    client.GetMyDate(DateTime.Today.AddDays(i % 30));
                    counter.TotalSuccess++;
                }
                catch (Exception ex)
                {
                    counter.TotalError++;
                }
            }
            counter.Stopwatch.Stop();

            counter.PrintStats();
            Console.WriteLine("Press any key to exit");
            Console.ReadKey();
        }
    }


![client](http://i1299.photobucket.com/albums/ag77/kappyzor/Blog/client_zps5bfde724.png)

In approximately 23 milliseconds we invoked 1000 times the Service.GetMyDate and got 283 errors. It’s like 28,3% errors for those who know simple math (for those who don't just take my word for it). That was the expected result. Next post let's dive on how we can improve these results.

The full source code can be located [here](https://github.com/kappy/FaultyMiddleware)
