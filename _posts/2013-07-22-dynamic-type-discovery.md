---
published: false
layout: post
title: Dynamic Type Discovery
---

On one of my open source projects (check you my [github profile](https://github.com/kappy)) I use some dynamic type discovery and loading. I need these because I want to load every type that implements an interface or has a specified attribute. I also need to be able to load types that reference my asssembly (not known to my assembly).

I started by enumerating all types on the appdomain:
    var types = AppDomain.CurrentDomain.GetAssemblies()
        .ToList()
	    .SelectMany(a => a.GetTypes());

I released my library using this approach. I worked. nice...

Few days after, I received 2 new issues: You library is blowing out with my application.
    [ReflectionTypeLoadException: Unable to load one or more of the requested types.     Retrieve the LoaderExceptions property for more information.]
    System.Reflection.RuntimeModule.GetTypes(RuntimeModule module) +0
    System.Reflection.RuntimeModule.GetTypes() +4
    System.Reflection.Assembly.GetTypes() +78

Hum... ok.. fine, But it really works on my machine...
Let's digg a bit more about `ReflectionTypeLoadException` shall we. 
[http://msdn.microsoft.com/en-us/library/system.reflection.reflectiontypeloadexception.aspx](Class Reference)
> There is also another array of exceptions (LoaderExceptions property). This exception array represents the exceptions that were thrown by the class loader. The holes in the class array line up with the exceptions.

Hum.... yup, the problem wasn't on my library but on some missing dependencies of the application, but because the library tries to enumerate all the types on the AppDomain, it burst in flames.

So, what can we do? Not much really, just work with the good eggs and ignore the others.

    private static Type[] GetTypesFromAssemblySafe(Assembly assembly) {
        try {
            return assembly.GetTypes();
        } catch {
            return new Type[] {};
        }
    }
    
    var types = AppDomain.CurrentDomain.GetAssemblies()
        .ToList()
	    .SelectMany(GetTypesFromAssemblySafe);

Bottom line... Even if your code is working nice, tested and feels good, on a disfunctional context, it can be a bomb ready to explode.
