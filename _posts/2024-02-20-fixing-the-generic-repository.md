---
layout: post
published: true
title: Fixing the Generic Repository
---
It has already been 4 years since my last post and almost 8 since I was actually writing on this blog, but I keep seeing developers using bad practices that are advertised as good and clean. The most obvious of them is the Generic Repository.

The Generic Repository is a pretty popular method of abstracting the data layer from the core business, and my personal opinion that why it got so popular is because it is easy and requires quite low code. You only implement the generic class, then all entities specific repository instances derive by generic instantiation, without any extra implementation. More so, there are innumerous implementations on the internet that can be easily adapted. The problem is that all of them have an issue while querying data. Every entity is queried differently. I've seen a few different flavours out there.

The most gross one is this `public IEnumerable<TTentiry> GetAll()`. It doesn't accept any parameter and it returns an `IEnumerable<TEntity>`. Would you use his method to query a table with millions of rows? For example, we have a table with Books, would you use this to find a book by title, or a list of books by author? Using this method to query any kind of information results in loading into memory all the rows of the table and only then filtering the rows. This would prove extremely inefficient. You're not making use of the query features of your database, and that is one of the main reasons they exist in the first place.

A close second is `public IQueryable<TTEntity> GetAll()`. Difference is that now we can use linq to query the data. But now I wonder... What is the value this layer generates if I can query whatever from my business? What prevents a developer from making a query that doesn't have the right indexes on the database? We're completely leaking the queries to the business layer where it is quite easy to make mistakes that will result in poor inefficient queries (and then we'll just blame Entity Framework for performance issues).

Another flavour, and this one diverges a bit from the Generic Repository, is to have specific repository classes that derive from the Generic one just to be able to create specific query methods adapted to its entity. This sacrifices the "no-code" approach but addresses the last two issues that were just mentioned. The only problem of this flavour is scalability. How would we still handle all the different ways to query books?

```
/* ... */
PageOfBooks GetBooksByAuthor(string author, int page, int pageSize);
PageOfBooks GetBooksByYear(itn year, int page, int pageSize);
PageOfBooks GetBooksByGenre(string genre, int page, int pageSize);
Book GetBooksISBN(string isbn);
/* ... */
```

For each query, we need to create a new method. Everytime we need to change/add a query, we are violating the Open/Closed principle since we're always modifying the entire Repository class.

I've promised a fix, and no, it isn't just a click bait. The way I propose to fix the Generic Repository is Query Encapsulation. By this, each query is written in its own class and our repository would have just a single method like `TOut Query<TOut>(IQuery<TEntity, TOut> query);`.
Queries would be encapsulated just as this:

```
public class GetBooksByAuthorQuery(string author, int page, int pageSize) : IQuery<PageOfBooks, BookEntity>
{
    public override PageOfBooks Execute(IDbSet<BookEntity> set)
    {
        return set
            .Where(book => book.Author == author)
            .Skip((page - 1) * pageSize)
            .Take(pageSize)
            .ToPage<PageOfBooks>()
    }
}
```

Using it on the business layer is simple as creating the query instance and passing it to the repository instance.

```
var booksByAuthor = repository.Query(new GetBooksByAuthorQuery(author, 1, 10));
```

A last note on why use Repositories and EntityFramework (or other kinds of ORM/micro-ORMs). I don't have any. I simply don't see a point. I could understand the point where developers want to be agnostic of Entity Framework to be able to replace it with other things in the future, but in 20 years, I never saw that happening. Like I said before, chances are, the business code would need to be modified as well. A change like that will always require a great investment because we would need to at least re-write an entire layer from scratch and we all know technical debt is always the least prioritisation because it doesn't generate money.

My advice is keep it simple, and if you're already using EntityFramework, you don't need to abstract it.

If you don't feel up to writing your own queries, and don't feel up to reinventing the wheel, take a look at [Ardalis Specification](https://specification.ardalis.com/) nuget package. It's pretty cool!
