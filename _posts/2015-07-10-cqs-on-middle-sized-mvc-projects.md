---
published: false
title: CQS on middle sized MVC projects
layout: post
---

In my [previous post](http://www.kspace.pt/posts/a-severe-case-of-servicitis/) I've indroduced it by saying it exists multiple flavors on the Business Layer of multiple layer architecture.

On this post, I want to talk a bit on [CQS](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation) applied on a MVC project. Also related is the [Command Pattern](https://en.wikipedia.org/wiki/Command_pattern).

The CQS pattern is simple, any operation on the application is either a Read or a Write, but never both. Applying this concept on a MVC application, it means that every interaction with the site can be defined on either queries or commands. A command is any operation that results on changing data (a POST), and query is everything that retreives data from the server but doesn't change it (a GET).

So, when a request arrives on the web application, either a command or a query (both are request) are created, then they get delivered to a [Mediator](https://en.wikipedia.org/wiki/Mediator_pattern). This mediator based on the current request, will find the proper handler to execute it.

## See it in Action

Here's an example of a controller using CQS:

    public class DrawsController : Controller
    {
        private readonly IMediator _mediator;
        private const int ItemsPerPage = 10;

        public DrawsController(IMediator mediator)
        {
            _mediator = mediator;
        }

        // GET: Draws
        public async Task<ActionResult> Index(GetDraws query)
        {
            query.ItemsPerPage = ItemsPerPage;
            var results = await _mediator.SendAsync(query);
            return View(results);
        }
    }



