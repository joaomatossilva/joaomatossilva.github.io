---
published: true
title: Code with value
layout: post
---

Not a long time ago, I was called to assist and manage an already ongoing project.
This is a small excerpt of a member of one call in the business layer:

![](http://i1299.photobucket.com/albums/ag77/kappyzor/Blog/bll_zpsd1f4cb75.png)

Well... Despite the obvious indentation and naming/code style, this small method has a lot of issues.
- It shouldn't dispose objects that we don't own. This applies to transaction and erm parameters;
- Its disposing erm and creating a new instance, but erm parameter isn't set as ref or out parameter, so the calling code only see the already disposed instance;
- Isn't c# a object oriented programming language? then why so many parameters on the method signature;
- Some parameters are only needed for logging. Why do we need to take them for a ride on every call? Infrastructure!!!!!
- `catch` statement is cacthing every exception and soaking the result as null. If we don't do anything with it (only logging) then why are we catching in the first place?
- The cherry on the top of the cake: we're returning an `IQueriable`. Why are we catching exceptions when we're only returning the query, not the query execution?

Conclusion: there isn't absolutely no value what so ever in this method. The only thing that it does is to call the layer below. This kind of infrastructure only gets in the way, it doesn't help the programmer or in fact, anyone.