---
published: true
title: "Wanted dead or alive: Middleware (part III)"
layout: post
---

This is a series of blog posts that focuses on faulty middleware services invocations, and how can we reduce the impact of those faults.

[Part I](http://www.kspace.pt/posts/wanted-dead-or-alive-middleware-part-i/)

[Part II](http://www.kspace.pt/posts/wanted-dead-or-alive-middleware-part-ii/)

Part III (this)

##Improving performance
Let's face it. If we throw a ball to a solid wall, it will ricochet every time. So why bother throw it all again, and again, and again.
The same way, most services that outputs data that isn't changing every second (an employee name, address, status, etc..) can be safely cached. This way we surely are less error prone to our `FaultyService` and we should gain performance because we'll use the service less.

##Do we need to change?
No we do not. We already made the required changes when we first introduced the interceptor.
Nothing keeps us from adding a new peace between and let that piece decide if we need to really invoke the service or we already know the result and output it right away.

To simulate a cache, I created a simple cache provider.

    public interface ICacheProvider
    {
        bool TryGet(object key, out object value);
        void Set(object key, object value);
    }
    
I also created a `PoorMansCacheProvider` that only stores the values on an `Hastable`. Please do note that this cache isn't production quality. It's just an academic proof of concept.

Here's the cache interceptor:
    public class CacheInterceptor : IInterceptor
    {
        private readonly ICacheProvider _cacheProvider;

        public CacheInterceptor(ICacheProvider cacheProvider)
        {
            _cacheProvider = cacheProvider;
        }

        public void Intercept(IInvocation invocation)
        {
            var arguments = invocation.Request.Arguments;
            var methodName = invocation.Request.Method.Name;
            // create an identifier for the cache key
            var key = methodName + "_" + string.Join("", arguments.Select(a => a ?? ""));  
            object value;
            if (_cacheProvider.TryGet(key, out value))
            {
                invocation.ReturnValue = value;
                return;
            }
            
            invocation.Proceed();

            _cacheProvider.Set(key, invocation.ReturnValue);
        }
    }
On it, I’m just generating a key that allows me to differentiate action invocations (even so, in this example we only got 1) and also differentiate invocations by parameters.
If the key exists, we return the value we got on store. If not, we proceed the chain of execution and save the result. Could it be simpler?


##Setup
Most of the hard work is done, but a still a detail on the setup of our interceptors.

    public class Module : NinjectModule
    {
        public override void Load()
        {
            Kernel.Bind<StatsCounter>().ToConstant(new StatsCounter());
            Kernel.Bind<ICacheProvider>().ToConstant(new PoorMansCacheProvider());
            var binding = Kernel.Bind<INaiveClient>().To<NaiveClient>();
            binding.Intercept().With<CacheInterceptor>().InOrder(1);
            binding.Intercept().With<RetryInterceptor>().InOrder(2);
        }
    }
Note the `InOrder` extension. That's the way we setup the execution order of the interceptors.

##Show me the results!
![ClientImprovedCached](http://i1299.photobucket.com/albums/ag77/kappyzor/Blog/ClientimprovedCached_zps5ba87cc2.png)
Well, surprisingly, we didn't get a performance boost from our first example. But that can easily being explained by the hash lookup and string key manipulations.
As for the resulting numbers:
0.2% error rate. We're increasing performance and in the process we increased the resilience. By not needing to invoke the service so many times, we did increased the resilience by a significant order of magnitude.
Notice the Execution Success: 30. This 30 is the different invocations we have. For the most sharp reader, we're invoking the service like this:
    client.GetMyDate(DateTime.Today.AddDays(i % 30));
This causes the cache to store 30 different values, hence the 30 success invocations.
To achieve those 30 success we needed to call the service 48 times.
2 times out of those 1000, we couldn't get any response.


##Series conclusion
I don't know about you, but this is a really neat way of improving an application performance.
The awesome part of this series is that we didn't have to mess around the client implementation. We extracted it an interface, but that's about it, and it should be already your way to do things. 
> Program to interfaces, not implementations

Dependency Injection and Interception are indeed tools that you should have always on your pocket.