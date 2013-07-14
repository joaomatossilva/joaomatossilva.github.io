---
published: true
layout: post
title: Code Metrics Alarm
---

When you're building any kind of software development project, there are some red flags that you should be aware of.
One of them is the number of logical lines of code ([Source lines of code](http://en.wikipedia.org/wiki/Source_lines_of_code)) on a single class.

This is one of those cases:
![](http://i1299.photobucket.com/albums/ag77/kappyzor/CodeMEtrics_zps86d35f60.png)
3k LLoC on a single class is way too much... 

Lets dig a little bit more into this class. Lets build a simple project to analyse the class stats.

{% highlight csharp %}
    class Program
    {
        static void Main(string[] args)
        {
            var type = typeof (ConfigurationDBReadHandler);
            Console.WriteLine("Stats of {0}", type.Name);
            Console.WriteLine("Number of properties {0}", type.GetProperties().Count());
            Console.WriteLine("Number of methods {0}", type.GetMethods().Count());
            Console.WriteLine("Number of fields {0}", type.GetFields().Count());
            Console.ReadLine();
        }
    }
{% endhighlight %}

Here's the output:
    Stats of ConfigurationDBReadHandler
    Number of properties 3
    Number of methods 160
    Number of fields 0

Without starting picking on the name of the class, the reasons behind those 160 methods it's because this was supposed to be a data access class. A single data access class for the entire project. All entities were loaded and saved from this class. It was madness to find a single entity related methods.
A direct violation of [SRP](http://en.wikipedia.org/wiki/Single_responsibility_principle).
