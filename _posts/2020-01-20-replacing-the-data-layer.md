---
layout: post
published: false
title: Replacing the Data Layer
---
For the first time in my professional career, I’ve made the exercise of changing the data backing store of an application. In this case, I’ve changed from a document database to a relational database. The goal was simple… Change the data layer of the application without changing anything else.
The application followed a typical onion design of presentation, domain and data layers. This last one by the form of the very well known Repository pattern.

    public interface IBookRepository
    {
        Guid GetById(Guid bookId);
        IEnumerable<Book> GetAll(string name);
        void Save(Book book);
        void Delete(Book book);
    }

There is not much to be told about the Repository itself. Is as plain as it can be. A method to get an item, a method to search items, a save and another to delete items. The important bits to hold are the interface objects for input and output of the interface, those are domain objects. Remember the onion design I said before?

Ok, enough talk, on to the action. I got to say, the big part of the things went smoothly. Simple domain objects were mapped from one repository implementation to the new one without hassle.
The real problem rose with more complex domain objects. For example, a Book has a list of Authors. Since we were using a document database, that was really a no brainer. The authors list was denormalized and stored along side the book data. Every update on the authors was simply persisted by saving the entire document on an atomic operation.

But now on the relational database, each Author has its own row on a dedicated table. The million dollar question, how do we know which authors were changed?
From the DDD purity, the Domain objects should not have persistence details, so long for tracking changes. This means we need a way to keep sync between what we have on database and our domain object that was passed. 

One solution is to eagerly load the inner authors list from our database entity and enumerate each domain author from input to check if already exists on the database, if no then add it. Then we need to enumerate the list from our database and check if all of them are still present on the domain list, if no, then remove it. And don’t forget to update the ones in the middle if needed.

    public void Save(Book entity)
    {
        //get the existing Book from the Database, eagerly loading the Authors 
        var book = await dbContext.Books
            .Include(x => x.Authors)
            .FirstOrDefaultAsync(x => x.Id == entity.Id);

        if (book == null)
        {
            book = new Model.Book
            {
                Id = entity.Id,
            };

            dbContext.Bundle.Add(book);
        }

        //Update the entity Properties
        book.Name = entity.Name;


        //check for Authors added
        foreach (var domainAuthor in entity.Authors)
        {
            var author = book.Authors
                .FirstOrDefault(x => x.Id == domainAuthor.Id);

            if (author == null)
            {
                //load the existing Author entity from Database
                author = dbContext.Authors.FirstOrDefault(x => x.Id == domainAuthor.Id);
                book.Authors.Add(author);
            }
        }

        //check for authors removed
        var existingAuthors = book.Authors.ToList(); //create a copy list in order to allow us to remove items while enumerating
        foreach (var existingAuthor in existingAuthors)
        {
            var author = entity.Authors.FirstOrDefault(x => x.Id == existingAuthor.Id);
            if (author == null)
            {
                book.Authors.Remove(existingAuthor);
            }
        }


        await dbContext.SaveChanges();
    }

In this case we are using EF inside the repositories, but the same would apply using another ORM or even micro-ORM as there are no changes tracking.

Would this happen if the application was built using a relational database in the first place? I seriously doubt it. Chances are that either specific methods were created on the repository to add those entries on the inner list, or that the domain object would be replaced by the entity object itself.
I think it’s safe to say, that no matter what, our choice of the backing store still affects the way we design an application, from top to bottom. Of course we do our best to create that layer of abstraction, but we’re only humans. We can only predict based on what we know.

If it was my design, I’m pretty positive that no repositories would be seen anywhere. I wouldn’t use the Onion Layers design. I would create vertical slices on the domain and mapping directly the data objects on the application. I wouldn’t care for database layer abstraction because I really want to use what each database features serve me best. Would this exercise be as “easy” as it was using that model? No, I guess not. But again, in more than 15 years, this is the first time I actually did it, and as of now, it is more academic than professional.

