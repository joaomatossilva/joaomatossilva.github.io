---
published: false
title: Code with value
layout: post
---

Not a long time ago, I was called to assist and manage an already ongoing project.
This is a small excerpt of a member of one call in the business layer:

![](http://i1299.photobucket.com/albums/ag77/kappyzor/Blog/bll_zpsd1f4cb75.png)

Well... Despite the obvious identation and naming/code stye, this small method has a lot of issues.
- we shouldn't dispose objects that we don't own. This applies to transaction and erm parameters;
- we're disposing erm and creating a new instance, but erm parameter isn't set as ref or out parameter, so the calling code only see the already disposed instance;
- isn't c# a object oriented programming language? then why so many parameters on the method signature;
- some parameters are only needed for logging. Why do we need to take them for a ride on every call? Infrastructure!!!!!
- catch statement is cathing every exception and soaking the result as null. If we don't do anything with it (only logging) then why are we catching in the first place?
- the cherry on the top of the cake: we're returning an `IQueriable`. Why are we catching exceptions when we're only returning the query, not the query execution?

Conclusion: there isn't absolutly no value what so ever in this method. The only thing that it does is to call the layer below. This kind of infrascruture only get's in the way, it doesn't help the programmer or in fact, anyone.