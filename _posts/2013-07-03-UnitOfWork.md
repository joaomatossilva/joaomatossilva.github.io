---
published: false
layout: post
title: Unit Of Work
---

This time I want to indroduce my approach on the Unit Of Work pattern ([Unit of Work](http://martinfowler.com/eaaCatalog/unitOfWork.html)).
If you're reading this and thinking EF or Nhibernate, then don't waste your time. DbContext and ISession are already implementations of Unit of Work.

I use this aproach mostly with Micro ORM frameworks, like Dapper, Simple.Data, etc...
Are you still doing Ado.Net directly? In what year do you live in?

{% highlight csharp %}

    public class UnitOfWork : IDisposable
    {
        private readonly IDbConnection connection;
        private IDbTransaction transaction;

        public UnitOfWork(string connectionString, bool requireTransaction = true)
        {
            connection = GetConnection(connectionString);
            connection.Open();
            if (requireTransaction)
            {
                BeginTransaction();
            }
        }

        public void BeginTransaction()
        {
            if (transaction != null)
            {
                return;
            }
            transaction = connection.BeginTransaction();
        }

        public void CommitChanges()
        {
            if (transaction != null)
            {
                transaction.Commit();
            }
        }

        public void Dispose()
        {
            Dispose(true);
            GC.SuppressFinalize(this);
        }

        protected void Dispose(bool disposing)
        {
            if (disposing)
            {
                if (transaction != null)
                {
                    transaction.Dispose();
                }
                if (connection != null)
                {
                    connection.Dispose();
                }
            }
        }

        ~UnitOfWork()
        {
            Dispose(false);
        }
    }
{% endhighlight %}

This is the base code. Now we have a few different options:
- Change the protecion of `Connection` and `Transaction` properties to public and let 
