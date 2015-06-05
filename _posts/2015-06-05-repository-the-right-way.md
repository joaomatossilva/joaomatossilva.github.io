---
published: true
title: Repository the right way
layout: post
---



## Repository the right way

So.. You just finished creating your super data access layer, a whole lot of repositories on top of Entity Framework, in order to make it testable. 
All tests are green, everything works, QA goes as expected, and life is good... 

Until... You reach production... You start seeing that your queries doesn't perform well on large datasets, and you start reviewing and debugging your code again... 
All the indexes are fine on Sql Server;
The query performs great when you make the query on Sql Management Studio;
You add some sql logging to your `DbContext` to check it out...

    public class HomeController : Controller
    {
        public readonly CustomerRepository _repository;
        public readonly StringBuilder _logBuffer;
        public HomeController()
        {
            var context = new RepoDataContext();
            _repository = new CustomerRepository(context);
            _logBuffer = new StringBuilder();
            context.Database.Log = _logBuffer.AppendLine;
        }
        public ActionResult ThisActionDoesntPerformWell()
        {
            _repository.Get(c => c.Name == "Kappy");
            ViewBag.Sql = _logBuffer.ToString();
            return View("Index");
        }
    }

You get this output:

    Opened connection at 05/06/2015 23:04:11 +01:00
    SELECT 
        [Extent1].[Id] AS [Id], 
        [Extent1].[Name] AS [Name]
        FROM [dbo].[Customers] AS [Extent1]
    -- Executing at 05/06/2015 23:04:11 +01:00
    -- Completed in 48 ms with result: SqlDataReader
    Closed connection at 05/06/2015 23:04:11 +01:00

But... How can that be.. The action is selection ALL the records from the Customers table. Your thoughts are like: "But the predicate `c => c.Name == "Kappy"` is there!! I Know it's there!!". You start digging it deeply....

    public class CustomerRepository
    {
        private readonly RepoDataContext _context;

        public CustomerRepository(RepoDataContext context)
        {
            _context = context;
        }

        public Customer[] Get(Func<Customer, bool> predicate)
        {
            return _context.Customers
                .Where(predicate)
                .ToArray();
        }
    }

Everything seems fine, no compiler errors, no compiler warnings... The world is ending, your boss starts to pressure you, because all this is already in Production. The client starts calling your office reporting this issue. 

Did you already spotted the error?

You probably did. 
Yup, it's the line `public Customer[] Get(Func<Customer, bool> predicate)` of the `CustomerRepository`.
By replacing that line with `public Customer[] Get(Expression<Func<Customer, bool>> predicate)` everything works again as expected.

    Opened connection at 05/06/2015 23:20:03 +01:00
    SELECT 
        [Extent1].[Id] AS [Id], 
        [Extent1].[Name] AS [Name]
        FROM [dbo].[Customers] AS [Extent1]
        WHERE N'Kappy' = [Extent1].[Name]
    -- Executing at 05/06/2015 23:20:03 +01:00
    -- Completed in 30 ms with result: SqlDataReader
    Closed connection at 05/06/2015 23:20:03 +01:00
    
    
## The Why

There is a big difference between a `Func<Customer, bool>` and a `Expression<Func<Customer, bool>>`, despite both can be assigned as `c => c.Name == "Kappy"`, the first one is the same as declaring inline the following function:

    bool Predicate(Customer c)
    {
        return c.Name == "Kappy";
    }

The later, it doesn't create the function (unless explicitly call `.Compile()`) but represents an expression graph of all the sentences (more on this later).

Another difference is that, the first case, we're not invoking the intended Linq extension `IQueriable<T> Where(this IQueriable<T> queriable, Expression<Func<Customer, bool>> predicate)` but instead we're invoking this one:
`IEnumerable<T> Where(this IEnumerable<T> enumerable, Func<Customer, bool> predicate)`.
With that in the clear, it comes very obvious that all the records are being pulled from the database, no matter what query would be there on the predicate.

## More on the Expression<>

Like I said before, an `Expression<Func<Customer, bool>>` is not the same as the `Func<Customer, bool>`. It doesn't actually execute lambda `c => c.Name == "Kappy"`, like the `Func` does.
Instead, it creates an expression graph composed of miltiple other smaller expressions that in hole represent the lambda that was specified.
Let's take a look at the composition of our example:
![Expression Graph](http://i1299.photobucket.com/albums/ag77/kappyzor/Blog/Expression_zpsqmgbwrno.png)
In red, you can see the left and right part of the expression, both of them Expressions of themselves.
In yellow, you can the type of operation that is being applied to both sides. 
It's with this information that EntityFramework, and other ORM's build the Sql statements for you.
