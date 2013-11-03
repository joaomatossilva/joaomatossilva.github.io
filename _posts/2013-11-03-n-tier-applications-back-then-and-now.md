---
published: false
title: "N Tier applications - Back then and now"
layout: post
---

## N tier applications

N tier applications are a generally adopeted design pattern to address the separation of concerns, and application reusage. In a nutshell: every tier has a responsability, and all tiers are disposed vertically and each one only depends on the tier immediately below.

The number of tiers is variable, but the most used tipification is the 4 tier application:
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
A few new kids on the block appeared here. DDD (Domain driven design), Rx (Reactive), 
maybe a few others.

### Data Access

Here happened the biggest boom. For years the only way to do data access was ADO.Net. Now, ORMs rised to the ocasion. Entity Framework, NHibernate, Linq to Sql, etc...
Even micro ORMs (OMs - Object Mappers) have dethroned ADO.Net.
It doesn't make sence any more to use Commands, DataReaders, DataSets...

One common mistake I usually see is to create a Data Acess tier using an ORM. What's the purpose to it? Why have a super framework to create the queries, execute them and get results and relations, database independant, and hide all that inside a box? How can we get the most of it? You can't really.

In my vision, the ORM is the Data Access. features like linq, hql, criterias, are the glue that ORMs give to the business layer. Nowadays, we can give the business layer the hability to query the database.






