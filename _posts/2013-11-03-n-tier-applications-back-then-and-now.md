---
published: true
title: "N Tier applications - Back then and now"
layout: post
---

## N tier applications

N tier applications are a generally adopted design pattern to address the separation of concerns, and application reuse. In a nutshell: every tier has a responsibility, and all tiers are disposed vertically and each one only depends on the tier immediately below.

The number of tiers is variable, but the most used is the 4 tier application:
- Presentation (Web, rich client, web service)
- Business (validations, transformations)
- Data Access (Data i/O)
- Data (Database persistence)

### Presentation Layer

Since the 2000's, web applications and web services dominated the world. Let's face it. It's easy to install, it's easy to upgrade, it runs on pretty well know ports and protocols.
Today, more and more web application appear. HTTP is here, it works, it's easy (kinda...).
MVC, REST, Web API, Sockets, are they keywords today.


### Business

Probably the most important layer on the application. Business rules, data validation and entity manipulation happens here.
A few new kids on the block appeared here. DDD (Domain driven design), Rx (Reactive), maybe a few others.

### Data Access

Here happened the biggest boom. For years the only way to do data access was ADO.Net. Now, ORMs rise to the occasion. Entity Framework, NHibernate, Linq to Sql, etc...
Even micro ORMs (OMs - Object Mappers) have dethroned ADO.Net.
It doesn't make sence any more to use Commands, DataReaders, DataSets...

One common mistake I usually see is to create a Data Access tier using an ORM. What's the purpose to it? Why have a super framework to create the queries, execute them and get results and relations, database independant, and hide all that inside a box? How can we get the most of it? You can't really.
In my vision, the ORM 'IS' the Data Access. Features like linq, hql, criterias, are the glue that ORMs give to the business layer. I'm not saying that business layers should be making the queries and updates to the database, but there are other players that we can use to store and reuse queries.

### Data

Well... Databases will be databases. Not much to say here. Only NoSql databases are winning some share here. The world isn't a set of element relations. It's more like composition of elements.


### My Conslusions

A lot has changed in these years, and we have to adapt. Doing Ado.Net directly nowadays is unthinkable. There are tools and frameworks to help us. We don't need to reinvent the wheel every project. That isn't the way to be productive. In the end, it's one of your major goals.
