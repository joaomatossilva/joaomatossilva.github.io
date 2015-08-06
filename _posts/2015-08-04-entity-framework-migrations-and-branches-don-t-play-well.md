---
published: true
title: "Entity Framework migrations and branches don't play well together"
layout: post
---




My team and I, decided that was time to use Entity Framework Code first and their migrations. I used [FluentMigrator](https://github.com/schambers/fluentmigrator) to manage my migrations already a few times , so using the Entity Framework ones, shouldn't be that hard. At the time of this writing we're using version 6.1.3.

### We were wrong... What we found isn't pretty.

We use Bitbucket, feature branches and pull requests in our development cycle, and recently we just stumbled into the following scenario:
Alice and Bob are developers, each one with a specific task. Bob needs to add a trucks feature, while Alice needs to add a Name into the user profile (of course names and tasks are entirely random and they represent only an example).

Both start they work from a `HEAD 4fd9a75`. Bob creates branch bob-feature, implements his changes (on this example just creates an entity named Truck), runs the command `Add-Migration Trucks` and commit the `2c430d4 - Added Trucks feature`, containing the migration `201508011250003_Trucks.cs` as follow. 

    public partial class Trucks : DbMigration
    {
        public override void Up()
        {
            CreateTable(
                "dbo.Trucks",
                c => new
                    {
                        Id = c.Int(nullable: false, identity: true),
                        Name = c.String(),
                    })
                .PrimaryKey(t => t.Id);
            
        }
        
        public override void Down()
        {
            DropTable("dbo.Trucks");
        }
    }

After the commit he creates the Pull Request for his feature.

Alice does a very similar procedure. Creates branch alice-feature, implements her changes (just adding a simple `string Name` property on the already existing `ApplicationUser` class), runs the command `Add-Migration UserName` and commit the `2659e15 - Added User Name feature` containing the migration `201508011256120_User-Name.cs` as follow.

    public partial class UserName : DbMigration
    {
        public override void Up()
        {
            AddColumn("dbo.AspNetUsers", "Name", c => c.String());
        }
        
        public override void Down()
        {
            DropColumn("dbo.AspNetUsers", "Name");
        }
    }

Same applies, after the commit, she creates a Pull Request fo everyone to review.

TL TR: this is the repository status so far.
![](http://i1299.photobucket.com/albums/ag77/kappyzor/Blog/ef_migrations1_zpssqtp3w77.png)


Both branches have a migration, both branches compile and run individually with their own local databases.
Bob's feature gets reviewed and is accepted, so the team merges it into main. Same goes for Alice's feature. 
![](http://i1299.photobucket.com/albums/ag77/kappyzor/Blog/ef_migrations2_zpsnzgp9rym.png)

Both features are done, merged, finished. Until.....
Kelly (another developer) tries to run the project from the HEAD on main, and she complains the application can't run. 

![](http://i1299.photobucket.com/albums/ag77/kappyzor/Blog/EFmigrations3_zpsvodj4dwp.png)

Pending changes? but all my changes are there...

## Digging into it

Why does this error appear? Microsoft saves a model snapshot with every migration it takes (is the Source field on the migration resource file). Since the last two migrations were created on different branches, they don't know each other. So, the latest migration snapshot (the one from Alice, but could be the other way around) doesn't know the migration before already exceuted and inserted its content.
The `Update-Database` checks exactly that. In this particular case, it thinks the Truck entity isn't created, and tries to add it. It fails because that entity already exists.

Even if you do what the message says, and run Add-Migration againd, it will create a new migration creating the Truck entity again.

This is not a new issue, in fact, it has been like this from the start. There is even a [MSDN Article](https://msdn.microsoft.com/en-us/data/dn481501.aspx) about this behavior and workarounds for it. But the solutions are simply unacceptable. To have manual process steps every time we merge migrations, is a No-Go! 
Why don't they singe they're selves to reproduce all the migrations code and compare it to the actual model, I don't know. They should only execute the code and nothing else.

The more I use Entity Framework migrations, the more I like [FluentMigrator](https://github.com/schambers/fluentmigrator).

## Update 2015-08-06 - There is light at the end of the tunnel

Thanks you Jared Dykstra, for pointing me in the direction of this annoucement from one of the developers from EF. On Entity Framework 7 (at the time of writing still in beta6), this is addressed and the snapshot will only be used for reference. Source of the announcement [here](http://www.bricelam.net/2014/12/16/ef7-migrations-designtime.html).



