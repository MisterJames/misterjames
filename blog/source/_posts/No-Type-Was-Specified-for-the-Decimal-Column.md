title: 'No Type Was Specified for the Decimal Column'
layout: post
tags:
  - EF Core
categories:
  - Development
authorId: james_chambers
featureImage: logo_579.png
originalUrl: 'http://jameschambers.com/'
date: 2019-06-21 07:19:24
---

When you create an entity for a database context in Entity Framework that has a `decimal` column, you may run into problems with truncation happening silently to you behind the scenes. With a default precision of 18 digits and 2 decimal places you may lose some data and the framework/runtime won't complain about it when you do.

You can fix that though data annotations or with an explicit wire-up in your database context. Let's look at how that's done.

<!-- more -->

Let's say you create an entity type that looks something like the following:

```
public class Part
{
    public Guid PartId { get; set; }
    public string Number { get; set; }
    public decimal Size { get; set; }
}
```

If you're dealing with dollars and cents, an underlaying field that records to the penny may be enough. In other scenarios you may be looking for more precision than 1/100. When you attempt to create a migration for the entity based on the class above, you'll get a warning that looks like this:

> No type was specified for the decimal column 'Size' on entity type 'Part'. This will cause values to be silently truncated if they do not fit in the default precision and scale. Explicitly specify the SQL server column type that can accommodate all the values using 'HasColumnType()'.


### Using Data Annotations

I find that most projects I'm on are already using data annotations. They are handy for model validation in our APIs (or UIs in ASP.NET) and are rather unobtrusive. They are also great at revealing rules for the underlaying data structure that we have in our project without having to dig into SQL Server. Said another way, an annotation shows a developer what the data is about and any constraints that exist in the database. They are concise and work great when the rules are straightforward.

Specifying the precision with an annotation is pretty straightforward, as is seen with `Part.Size` below.

```
public class Part
{
    public Guid PartId { get; set; }
    public string Number { get; set; }
    [Column(TypeName = "decimal(18,4)")]
    public decimal Size { get; set; }
}
```

This will store 18 digits in the database with 4 of those after the decimal.

Intellisense will prompt you to add the `System.ComponentModel.DataAnnotations.Schema` namespace to your class if you don't already have it.

### Using the ModelBuilder

Your database context class can use the `protected override void OnModelCreating` method with a `ModelBuilder` parameter to futher configure your database instance (see [Microsoft Docs](https://docs.microsoft.com/en-us/ef/core/modeling/)). In our case, we'll use the `ModelBuilder` to specify the type to use in SQL Server as well as the precision we wish to use.

```
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);
    modelBuilder.Entity<Part>()
        .Property(p => p.Size)
        .HasColumnType("decimal(18,4)");
}
```

The `ModelBuilder` is a good option and sometimes the only option when trying to do more complicated configuration. I will sometimes add a logging statement (for the development environment) in my `OnModelCreating` in a project that also uses data annotations because it at least stands a chance of being seen by a developer in the event that unexpected behaviour is being observed. Another option would be to leave a comment on any entity that is configured via `OnModelCreating` so that you're not leaving future versions of yourself scratching your head.

Happy coding!