In this tutorial, you create a .NET Core console app that performs data access against a SQLite database using Entity Framework Core.

You can follow the tutorial by using Visual Studio on Windows, or by using the .NET CLI on Windows, macOS, or Linux.

[View this article's sample on GitHub](https://github.com/dotnet/EntityFramework.Docs/tree/main/samples/core/GetStarted).
## Create a new project

```bash
dotnet new console -o EFGetStarted
cd EFGetStarted
```
## Install Entity Framework Core

To install EF Core, you install the package for the EF Core database provider(s) you want to target. This tutorial uses SQLite because it runs on all platforms that .NET supports. For a list of available providers, see [Database Providers](https://learn.microsoft.com/en-us/ef/core/providers/).

```bash
dotnet add package Microsoft.EntityFrameworkCore.Sqlite
```
## Create the model

Define a context class and entity classes that make up the model.
- In the project directory, create **Model.cs** with the following code

```c#
using Microsoft.EntityFrameworkCore;
using System;
using System.Collections.Generic;

public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }

    public string DbPath { get; }

    public BloggingContext()
    {
        var folder = Environment.SpecialFolder.LocalApplicationData;
        var path = Environment.GetFolderPath(folder);
        DbPath = System.IO.Path.Join(path, "blogging.db");
    }

    // The following configures EF to create a Sqlite database file in the
    // special "local" folder for your platform.
    protected override void OnConfiguring(DbContextOptionsBuilder options)
        => options.UseSqlite($"Data Source={DbPath}");
}

public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }

    public List<Post> Posts { get; } = new();
}

public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }

    public int BlogId { get; set; }
    public Blog Blog { get; set; }
}
```

EF Core can also [reverse engineer](https://learn.microsoft.com/en-us/ef/core/managing-schemas/scaffolding/) a model from an existing database.

Tip: This application intentionally keeps things simple for clarity. [Connection strings](https://learn.microsoft.com/en-us/ef/core/miscellaneous/connection-strings) should not be stored in the code for production applications. You may also want to split each C# class into its own file.

---
## Create the database

The following steps use [migrations](https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/) to create a database.
- Run the following commands:

```bash
dotnet tool install --global dotnet-ef
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet ef migrations add InitialCreate
dotnet ef database update
```

This installs [dotnet ef](https://learn.microsoft.com/en-us/ef/core/cli/dotnet) and the design package which is required to run the command on a project. The `migrations` command scaffolds a migration to create the initial set of tables for the model. The `database update` command creates the database and applies the new migration to it.
## Create, read, update & delete
- Open _Program.cs_ and replace the contents with the following code:

```c#
    using System;
    using System.Linq;
    
    using var db = new BloggingContext();
    
    // Note: This sample requires the database to be created before running.
    Console.WriteLine($"Database path: {db.DbPath}.");
    
    // Create
    Console.WriteLine("Inserting a new blog");
    db.Add(new Blog { Url = "http://blogs.msdn.com/adonet" });
    db.SaveChanges();
    
    // Read
    Console.WriteLine("Querying for a blog");
    var blog = db.Blogs
        .OrderBy(b => b.BlogId)
        .First();
    
    // Update
    Console.WriteLine("Updating the blog and adding a post");
    blog.Url = "https://devblogs.microsoft.com/dotnet";
    blog.Posts.Add(
        new Post { Title = "Hello World", Content = "I wrote an app using EF Core!" });
    db.SaveChanges();
    
    // Delete
    Console.WriteLine("Delete the blog");
    db.Remove(blog);
    db.SaveChanges();
```
## Run the app

```bash
dotnet run
```