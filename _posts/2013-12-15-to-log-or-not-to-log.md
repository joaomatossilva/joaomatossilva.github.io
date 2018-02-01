---
published: true
title: To log or not to log...
layout: post
---

A common mistake I've seen sometimes is mixing up logging and auditing. Here's an example:

![Log Class](http://www.kspace.pt/images/blog/Log_zpsc2ea63fa.PNG)

The first public method, `Log_Action`, was used to register actions (like logons, logoffs, external services actions, etc...). This is a clear audit related action.

The third public method, `Log_Error`, was used to register application errors in the database. This is a clear log related action. 

The second method, `Log_DB`, was used to register saves, updates and deletes on every object in the database. This is where the confusion rises. For debugging, having a record of every entity change is great, but the real reason for this action is for auditing.
A clear use case for this last action is an entity change history.

Logging to the database is a bloat of information. It costs application resources, when it shouldn't, and it should be transparent. Logging is mainly a diagnosing  tool and it should always be part of the solution, not part of the problem.

Can you see any parameter of type `Exception` on the `Log_Error` method? Nop... There isn't any. The only information we register, is a simple description text...


A little tip: Logging should be transparent. If we remove the logging from the source code, the application should run the same way. Having 358 coded references to the concrete type isn't probably the best way to do it.
