---
published: false
title: CQS on middle sized MVC projects
layout: post
---

In my [previous post](http://www.kspace.pt/posts/a-severe-case-of-servicitis/) I've introduced it by saying it exists multiple flavors on the Business Layer of multiple layer architecture.

On this post, I want to talk a bit on [CQS](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation) applied on a MVC project. Also related is the [Command Pattern](https://en.wikipedia.org/wiki/Command_pattern).

The CQS pattern is simple, any operation on the application is either a Read or a Write, but never both. Applying this concept on a MVC application, it means that every interaction with the site can be defined on either queries or commands. A command is any operation that results on changing data (a POST), and query is everything that retrieves data from the server but doesn't change it (a GET).

So, when a request arrives on the web application, either a command or a query (both are requests) are created, then they get delivered to a [Mediator](https://en.wikipedia.org/wiki/Mediator_pattern). This mediator based on the current request, will find the proper handler to execute it.

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
        
        /* Other actions below*/
    }

In this example, every time we get a request on /Draws/Index we issue a GetDraws query. The Mediator will then lookup an handler that handle the `GetDraws` request. This is an example of such an handler:

    public class GetDrawsHandler : IAsyncRequestHandler<GetDraws,Paged<Draw>>
    {
        private readonly ApplicationDbContext _db;

        public GetDrawsHandler(ApplicationDbContext db)
        {
            _db = db;
        }

        public async Task<Paged<Draw>> Handle(GetDraws message)
        {
            var basequery = _db.Draws.Where(d => d.UserId == message.UserId);
            if (message.GameId != null)
            {
                basequery = basequery.Where(d => d.GameId == message.GameId);
            }
            if (message.Date != null)
            {
                basequery = basequery.Where(d => DbFunctions.TruncateTime(d.Date) == message.Date);
            }

            var total = await basequery.CountAsync();
            var draws = await basequery.Include(d => d.Game).Include(d => d.User)
                .OrderByDescending(d => d.Date)
                .Skip((message.Page - 1) * message.ItemsPerPage)
                .Take(message.ItemsPerPage)
                .ToListAsync();

            return new Paged<Draw>
            {
                Items = draws,
                ItemsTotalCount = total,
                ItemsPerPage = message.ItemsPerPage,
                CurrentPage = message.Page
            };
        }

For the command side if the requests, everything is similar, simply in this case, we don't need the result value. Here's an example of the /Draws/Edit POST action:

        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<ActionResult> Edit([Bind(Include = "Id,Date,Cost,Award")] Draw draw)
        {
            if (ModelState.IsValid)
            {
                await _mediator.SendAsync(new EditDraw { Draw = draw });
                return RedirectToAction("Index");
            }
            return View(draw);
        }
        
And the handler for `EditDraw` is like this:
   
    public class EditDrawHandler : IAsyncRequestHandler<EditDraw, Unit>
    {
        private readonly ApplicationDbContext _db;

        public EditDrawHandler(ApplicationDbContext db)
        {
            _db = db;
        }

        public async Task<Unit> Handle(EditDraw message)
        {
            var drawEntry = await _db.Draws.FindAsync(message.Draw.Id);
            drawEntry.Date = message.Draw.Date;
            drawEntry.Cost = message.Draw.Cost;
            drawEntry.Award = message.Draw.Award;

            await _db.SaveChangesAsync();
            return Unit.Value;
        }
    }

The `Unit` type is a replacement for void, because we don't need a return value.

## Why Middle sized?

Well, I really think that middle sized applications really shine using this architecture. Applying this architecture on smaller applications, is too much of architecture. For simple and demo applications, I really don't have nothing against using the ORM or equivalent directly on the controller.

For larger applications, this architecture still shines, but we can evolve this into a more traditional CQRS architecture, with different data-sources for read and writes and using event sourcing for notifications.

## Final Considerations

Conceptually, this pattern is very straight forward. Every-time we receive a request, we bind it (using the MVC model binder, or manually) to a request message, and then the mediator delivers it to the corresponding handler that is responsible to execute the request.

This architecture applied, keeps the web project business clean, and with only one dependency, the mediator. This last bullet for me is a big winner. Many time I see multiple dependencies on a controller, that aren't used on the many actions.

Because this is a message based communication, this is very scalable. Moving from a simple Mediator to a Service Bus, is a breeze. This is also very easy to apply cross-cutting behavior like logging, security or validation, using decorators on the handlers. But I'll show you a bit more on this on my next posts.
