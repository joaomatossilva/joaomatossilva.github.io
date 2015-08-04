---
published: false
title: "Entity Framework migrations and branches don't play well "
layout: post
---


My team and I, decided that was time already to use Entity Framework Code first and their migrations. I used already a few times [FluentMigrator](https://github.com/schambers/fluentmigrator), so using the Entity Framework ones, shouldn't be that hard.

### We were wrong... What we found isn't pretty.

We use Bitbucket, feature branches and pull requests in our development cycle, and recently we just stumbled into the following scenario:
Alice and Bob are developers, each one with a specific task. Bob needs to add a trucks feature, while Alice needs to add a Name into the user profile (of course names and tasks are entirely random and they represent only an example).

Both start they work from a `HEAD 4fd9a75`. Bob creates branch bob-feature, implements his changes (on this example just creates an entity named Truck), runs the command `Add-Migration Trucks` and commit the `2c430d4 - Added Trucks feature`, containing the migration 12312545345_Trucks.cs as follow. After the commit he creates the Pull Request for his feature.

Alice does a very similar procedure. Creates branch alice-feature, implements her changes (just adding a simple `string Name` property on the already existing `ApplicationUser` class), runs the command `Add-Migration UserName` and commit the `2659e15 - Added User Name feature` containing the migration 12312545345_UserName.cs as follow.

TL TR: this is the repository status so far.
![](http://i1299.photobucket.com/albums/ag77/kappyzor/Blog/ef_migrations1_zpssqtp3w77.png)


Both branches have a migration, both branches compile and run individually with their own local databases.
Bob's feature gets reviewed and is accepted, so the team merges it into main. Same goes for Alice's feature. 
![](http://i1299.photobucket.com/albums/ag77/kappyzor/Blog/ef_migrations2_zpsnzgp9rym.png)

Both features are done, merged, finished. Until.....
Kelly (another developer) tries to run the project from the HEAD on main, and she complains the application can't run. 

`error message here`

Why does this error appear? Microsoft saves a model snapshot with every migration it takes (is the Source field on the migration resource file). Since the last two migrations were created on different branches, they don't know each other. So, the latest migration snapshot (the one from Alice, but could be the other way around) doesn't know the migration before already exceuted and inserted its content.
The `Update-Database` checks exactly that. In this particular case, it thinks the Truck entity isn't created, and tries to add it. It fails because that entity already exists.

This is not a new issue, in fact, it has been like this from the start. There is even a [MSDN Article](https://msdn.microsoft.com/en-us/data/dn481501.aspx) about this behavior and workarounds for it. But the solutions are simply unacceptable, has to have manual process steps every time we merge migrations. Why don't they singe they're selves to reproduce all the migrations code and compare it to the actual model, I don't know. They should only execute the code and nothing else.

The more I use Entity Framework migrations, the more I like [FluentMigrator](https://github.com/schambers/fluentmigrator).
