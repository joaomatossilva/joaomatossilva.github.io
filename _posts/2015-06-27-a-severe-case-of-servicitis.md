---
published: false
title: A severe case of Servicitis
layout: post
---

Before we go any further, let me start by stating two important thigs:

- Disambiguation from "Cervicitis" or any other disease;
- The title choice was only to be catchy, no pun intended there.


A very common 3 (or 4) layer architecture is defined by having a Presentation, Business and Data Access layers.

Particularly in the Business layer, also exist multiple flavors... Managers, Services, Beans, Handlers, etc... All of them have in common:

- They should encapsulate the business logic of the application;
- They shouldn't leak implementations details;
- They shouldn't leak data entities.

Blá blá blá... Enough of introductions...

Lets imagine the follow scenario. We have a web application on which the business applications is defined by services. Each entity has a service (similar to Entity Services, which many consider an Anti-pattern, but that concept has a different scope than this (SOA), and clearly out of this post scope).

On to the point already... We have a `CustomerService` and a `ShopService`. We need to create a new operation to retrieve the shop the customer as chosen as it's favorite.
Where would you create that operation? Into the `CustomerService` or `ShopService`? 
The truth is, there's no right answer on this. Both of them are correct. The only catch is, whatever your choice was, it needs to be coherent on the application, and thus, on the team, for further new operations.

In order to get an order for the Customer would you create it on the `OrderService` or on the `CustomerService`? I'm almost sure that you'll chose the `OrderService` on this case. And the answer is the same as above. Both of them are right, as long as you keep coherence.

##Let's look into the anatomy of the operation

> We need to create a new operation to retrieve the shop the customer as chosen as its favorite

Input: Customer
Output: A shop
Operation: From a list of shops, select the shop the customer has as favorite.

So, what is the most preeminent entity here, shop or a customer? My option is shop, what's yours?

What I see here is that I'm looking up a shop, and my customer is just a filter on my query. If I would write this down on Sql, this would be my query:

    SELECT shop.Id, Shop.Name
    FROM shop
    JOIN customer ON shop.Id = customer.favoriteShopId

My conclusion? I would definitely put it on the `ShopService` because that is in fact the Entity I'm looking for. The customer here is just a query parameter. But like I said, I don't consider the other option wrong, as long as that option is chosen coherently on all the application.
