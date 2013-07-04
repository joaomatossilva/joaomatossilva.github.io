---
published: true
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
- 1. Add public properties of `Connection` and `Transaction` and let the outside world to manage them.
- 2. Add protected properties of `Connection` and `Transaction` and derive the class to a data access specific class.
- 3. Add to it the public interface of your favorite micro Orm library.

For option \#1, I really don't like it. It most certainly will lead to micro managing connection and transactions...
For option \#2 and \#3, it's really up to you. You can even mix and match them.

I use option \#3. Why? because is simpler, and I mostly just use Dapper on my projects. It's a matter of personal preference.

This is what I end up with:

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

        public int Execute(string query, object param = null, CommandType commandType = CommandType.Text)
        {
            return connection.Execute(query, param, transaction, commandType: commandType);
        }

        public IEnumerable<T> Query<T>(string query, object param = null, CommandType commandType = CommandType.Text)
        {
            return connection.Query<T>(query, param, transaction, commandType: commandType);
        }

        public IEnumerable<dynamic> Query(string query, object param = null, CommandType commandType = CommandType.Text)
        {
            return connection.Query(query, param, transaction, commandType: commandType);
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
