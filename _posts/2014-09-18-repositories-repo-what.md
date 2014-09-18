---
published: true
title: "Repositories! Repo what??"
layout: post
---

Lately, the Repository pattern has become the conversion topic almost every day, mostly because nowadays it's a so popular pattern, and most people get admired on the fact that I prefer to avoid it, in favor to expose the ORM to the service layer.
So I thought about publicly exposing my why, so I can remit here and not repeat myself every time.

For introduction to the Repository, check [here](http://martinfowler.com/eaaCatalog/repository.html).
This post will also be an extension from what I've already said on my previous [post](http://www.kspace.pt/posts/n-tier-applications-back-then-and-now/).


Let's get to the point. My first problem with Repositories is that they hide, and often blocks you the use of advanced features of the actual ORM. Future queries, cascade deletes, batch updates, etc...

I've seen two different strategies with repositories. The most radical one, it's a full closed box, where they implement every query that needs to be issued. They main advantage of this method is that there's a single place where all queries sit, at the cost of filling each repository with methods for each different type.
    public interface ICarRepository : IRepository
    {
        Car GetById(int id);
        Car GetBySerialNumber(string serialNumber);
        IEnumerable<Car> GetRedCars();
        IEnumerable<Car> GetRedCarsInPortugal();
        IEnumerable<Car> GetClientOrderCars(int clientId);
        /* And other....  */
    }

The other way, is build one generic, fits all repository. This is also the most common example you can find on the internet. I usually find in this way some flaws:

* Exposing the Collection-Like as `IEnumerable`: although this defers the query, this blocks you the ability to make joins with entities, or even perform aggregations and other linq extensions.
* Exposing a `SaveChanges()` or internally call the orm `SaveChanges` on adding/updating items: It violates your unit of work, and blocks you the ability to save different entities on the same transaction.

        public void CreateAndAddItemToShoppingCart(object newItem)
        {
            var cart = new ShoppingCart();
            _shoppingCartRepository.Add(cart);
            _shoppingCartRepository.SaveChanges();
            newItem.CartId = cart.Id;
            _shoppingCarRowtRepository.Add(newItem);
            _shoppingCarRowtRepository.SaveChanges();
        }
        
        public void AddItemToShoppingCart(int cartId, object newItem)
        {
            var cart = _shoppingCartRepository.GetCart(cartId);
            _shoppingCartRowRepository.Add(cart, newItem);
            cart.Total += newItem.Cost;
            _shoppingCartRepository.UpdateCart(cart);
        }
* Just abstracting method by method the Orm: This is just a proxy and it really don't add anything to the table.


So.... Let's analyse this the other way around. What's the reasons I ear this pattern being defended...

* I can mock and unit test my code. - Well, you can do that too without Repositories. If your goal is to test your queries, you can mock the Orm.
* It add a security layer on my junior developers to protect them from doing something wrong. - Are you sure it's not the other way around? If your junior developers don't know what's on inside that black box, how can you be sure they're fetching the data right?
* I can replace my Orm if I don't like it. - Sure you can, but if you are doing that, you'll need to refactor all your repositories, which would take the same cost of changing the Orm itself on the services.
* Repositories abstract my data access layer. - Yes, but the Orm too. So you have a double data access layer. Good for you.



Ok, so what do you use then? **Queries and Commands**!
But I'll explain that a little bit better on my next post.