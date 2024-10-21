Let's modify the Program.cs file to resemble a basic CRUD application:

```csharp
public static List<User> Users = new()
{
    new User()
    {
        Id = 1,
        FirstName = "Callie",
        LastName = "Hackforth",
        BirthDate = new DateOnly(1995, 10, 3)
    },
    new User()
    {
        Id = 2,
        FirstName = "Odell",
        LastName = "Blowes",
        BirthDate = new DateOnly(1984, 4, 7)
    },
    new User()
    {
        Id = 3,
        FirstName = "Callie",
        LastName = "Corrett",
        BirthDate = new DateOnly(1991, 3, 4)
    }
};

var builder = WebApplication.CreateBuilder(args);

builder.Services
       .AddEndpointsApiExplorer()
       .AddSwaggerGen();
       
var app = builder.Build();

app.UseHttpsRedirection();

app.MapGet("", () => Collections.Users);

app.MapGet("/{id}", (int id) => Collections.Users
                                           .FirstOrDefault(user => user.Id == id));

app.MapPost("", (User user) => Collections.Users.Add(user));

app.MapPut("/{id}", (int id, User user) =>
{
    User currentUser = Collections.Users
                                  .FirstOrDefault(user => user.Id == id);
    
    currentUser.FirstName = user.FirstName;
    currentUser.LastName = user.LastName;
    currentUser.BirthDate = user.BirthDate;
});

app.MapDelete("/{id}", (int id) =>
{
    var userForDeletion = Collections.Users
                                     .FirstOrDefault(user => user.Id == id);
    
    Collections.Users.Remove(userForDeletion);
});

app.Run();

public class User
{
    public int Id { get; init; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public DateOnly BirthDate { get; set; }
}
```

As evident, everything resides within that single location, including registered services, endpoints, and even models. However, this approach remains acceptable only up to a certain point.

Consider this scenario: You decide to incorporate a [repository pattern](https://code-maze.com/net-core-web-development-part4/?ref=blog.treblle.com), followed by the introduction of background jobs, a handful of middleware, and an additional set of endpoints catering to various resources.

Ultimately, Program.cs can transform into a tangled mass of code, leading to issues such as:

1. **Challenging Maintainability**: It becomes increasingly difficult to maintain.
2. **Unreadable Code**: The code becomes less readable and comprehensible.
3. **Violations of the Single Responsibility Principle**: It violates the Single Responsibility Principle by trying to manage too many responsibilities within one file.

However, two solutions can greatly enhance the cleanliness and maintainability of Minimal API:

1. **Endpoint Grouping (Introduced in .NET 7)**: This allows you to logically group your endpoints, offering a cleaner structure.
2. **Carter (External Library)**: By leveraging the [Carter library](https://github.com/CarterCommunity/Carter?ref=blog.treblle.com), you can achieve improved organization and maintainability in your Minimal API project.

I will provide a comprehensive discussion of these solutions and guide you through their implementation in your current project.

### Endpoint grouping

The first approach to achieving cleaner and more organized endpoints involves grouping them by feature.

In .NET 6, this could be accomplished using extension methods on the **IEndpointRouteBuilder** class. However, with the release of .NET 7, a new feature known as "**Endpoint grouping**" has been introduced. This feature enables you to apply constraints and rules to a collection of Minimal API endpoints simultaneously.

No more theory – let's streamline this API!

Before delving into endpoint modifications, let's tidy up the Program.cs file.

Create a directory named "Extensions" and within it, establish a static class named "**Configuration.cs**". This class will house two methods in this context:

- **RegisterServices -** This method will encompass the registration of all services in the dependency injection container.
- **RegisterMiddlewares** - This method will handle the registration of all middleware.

```csharp
public static class Configuration
{
    public static void RegisterServices(this WebApplicationBuilder builder)
    {
        builder.Services
               .AddEndpointsApiExplorer()
               .AddSwaggerGen();
    }

    public static void RegisterMiddlewares(this WebApplication app)
    {
        if (app.Environment.IsDevelopment())
        {
            app.UseSwagger()
               .UseSwaggerUI();
        }

        app.UseHttpsRedirection();
    }
}
```

The registration in Program.cs appears as follows:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.RegisterServices();

var app = builder.Build();

app.RegisterMiddlewares();

app.Run();
```

For the sake of simplicity, as I am using an in-memory collection, I will also relocate the static collection to another folder and file.

Establish a "Data" folder comprising a static class named "Collections" and a static list titled "Users."

```csharp
public static class Collections
{
    public static List<User> Users = new List<User>()
    {
        new User()
        {
            Id = 1,
            FirstName = "Callie",
            LastName = "Hackforth",
            BirthDate = new DateOnly(1995, 10, 3)
        },
        new User()
        {
            Id = 2,
            FirstName = "Odell",
            LastName = "Blowes",
            BirthDate = new DateOnly(1984, 4, 7)
        },
        new User()
        {
            Id = 3,
            FirstName = "Callie",
            LastName = "Corrett",
            BirthDate = new DateOnly(1991, 3, 4)
        },
        new User()
        {
            Id = 4,
            FirstName = "Channa",
            LastName = "McKeggie",
            BirthDate = new DateOnly(1985, 11, 13)
        },
        new User()
        {
            Id = 5,
            FirstName = "Angelita",
            LastName = "Jubert",
            BirthDate = new DateOnly(1990, 1, 9)
        }
    };
}
```

Within the same folder, include the "User" model

```csharp
public class User
{
    public int Id { get; init; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    internal DateOnly BirthDate { get; set; }
}
```

The initial phase of cleanup is complete; now, let's address the endpoints:

1. Create a folder at the project's root called "Endpoints"

![](https://blog.treblle.com/content/images/2023/09/endpoint-folder.png)

Endpoint folder for grouped endpoints

2. Inside the "Endpoints" folder, create a class with the corresponding name, which in our case will be "Users." This class must be static since we will be including an extension method that utilizes the endpoint grouping feature

```csharp
public static class Users
{
    public static void RegisterUserEndpoints(this IEndpointRouteBuilder routes)
    {
        // Grouped endpoint will go here
    }
}
```

It's good practice to name methods by appending the feature name followed by "Endpoints" (e.g., UserEndpoints).

3. Include grouped endpoints within the previously added method

```csharp
public static class Users
{
    public static void RegisterUserEndpoints(this IEndpointRouteBuilder routes)
    {
      var users = routes.MapGroup("/api/v1/users");
      
      users.MapGet("", () => Collections.Users);
      
      users.MapGet("/{id}", (int id) => Collections.Users
                                                   .FirstOrDefault(user => user.Id == id));
                                                   
      users.MapPost("", (User user) => Collections.Users.Add(user));

      users.MapPut("/{id}", (int id, User user) =>
      {
        User currentUser = Collections.Users
                                      .FirstOrDefault(user => user.Id == id);

        currentUser.FirstName = user.FirstName;
        currentUser.LastName = user.LastName;
        currentUser.BirthDate = user.BirthDate;
     });
     
     users.MapDelete("/{id}", (int id) =>
     {
        var userForDeletion = Collections.Users
                                         .FirstOrDefault(user => user.Id == id);

        Collections.Users.Remove(userForDeletion);
     });
   }
}
```

The MapGroup method is applied to an IEndpointRouteBuilder variable, which contains all the application's routes. It accepts prefixes as its sole parameter. Consequently, all endpoints added to this group will have the specified prefixes prepended, along with any additional information provided.

To complete the refactoring process, simply register the method in Program.cs, similar to how you registered services and middlewares. That's all there is to it!

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.RegisterServices();

var app = builder.Build();

app.RegisterMiddlewares();

app.RegisterUserEndpoints(); // <-- Add this line
```

What are the advantages of this approach?

- Supported out-of-the-box by the framework
- Promotes cleaner and more maintainable code
- Consolidates all changes within a single file
- Cross-cutting concerns can be directly attached to a group

One drawback of this approach is that the codebase can expand significantly when numerous endpoint groups are involved.  
However, if you prefer a more flexible approach and enjoy working with modules, then Carter is the choice for you.

### Carter

As per the GitHub readme file:

_**“**Carter is a framework that is a thin layer of extension methods and functionality over ASP.NET Core allowing the code to be more explicit and most importantly more enjoyable.**”**_

Essentially, it follows a similar approach to the first one, but with some additional features such as:

1. Automatic registration of all interface implementations for Carter components into ASP.NET Core DI.
2. Inclusion of FluentValidation extensions to validate incoming HTTP requests, a feature not found in ASP.NET Core Minimal APIs.

The setup can be completed in three straightforward steps:

- Add the library from NuGet.
- Register the Carter service into the DI container: **builder.Services.AddCarter();**
- Add the middleware: **app.MapCarter();**

```csharp
 public static void RegisterServices(this WebApplicationBuilder builder)
 {
     builder.Services
            .AddEndpointsApiExplorer()
            .AddCarter() // <-- Add this line
            .AddSwaggerGen();
 }

 public static void RegisterMiddlewares(this WebApplication app)
 {
     if (app.Environment.IsDevelopment())
     {
         app.UseSwagger()
            .UseSwaggerUI();
     }

     app.MapCarter(); // <-- And this
 }
```

The AddCarter method handles the necessary services provided by Carter, while MapCarter scans for all endpoints that implement the ICarterModule interface and integrates them with the existing endpoints.

As mentioned earlier, Carter operates with modules, and all endpoints will be encapsulated within their respective modules.

![](https://blog.treblle.com/content/images/2023/09/carter-modules.png)

Carter folder structure

The Module class should inherit the ICarterModule interface, which exposes only one method (AddRoutes). However, it is advisable to inherit from the CarterModule class, which provides additional features.

```csharp
public class Endpoints : CarterModule
{
    public override void AddRoutes(IEndpointRouteBuilder app)
    {
        app.MapGet("", () => Collections.Users);

        app.MapGet("/{id}", (int id) => Collections.Users
                                                   .FirstOrDefault(user => user.Id == id));

        app.MapPost("", (User user) => Collections.Users.Add(user));

        app.MapPut("/{id}", (int id, User user) =>
        {
            User currentUser = Collections.Users
                                          .FirstOrDefault(user => user.Id == id);

            currentUser.FirstName = user.FirstName;
            currentUser.LastName = user.LastName;
            currentUser.BirthDate = user.BirthDate;
        });

        app.MapDelete("/{id}", (int id) =>
        {
            var userForDeletion = Collections.Users
                                             .FirstOrDefault(user => user.Id == id);

            Collections.Users.Remove(userForDeletion);
        });
    }
}
```

Features from the base class are incorporated through the constructor within the module, and they pertain to the endpoints contained within it.  
Some of these features encompass:

1. Rate limiting
2. Authorization
3. CORS
4. Endpoint grouping
5. Pre and Post endpoint actions

```csharp
 public Endpoints() : base("api/v1/users")
 {
     this.RequireRateLimiting("fixedWindow")
         .WithCacheOutput("CacheOutput");
 }
```

Upon executing the application following the completion of the setup, you will achieve the same result as with the initial approach.

### What to do next after the successful restructuring of the API?

By employing this approach, you've addressed just one aspect of the puzzle. What about documenting your API, including all endpoints and their associated contracts? How about ensuring security and optimizing performance? There's a multitude of considerations for crafting a robust API.

Fortunately, the solution to your challenges lies in [Treblle](https://treblle.com/?ref=blog.treblle.com).

  
Treblle offers comprehensive coverage, spanning from observability to automated API documentation. The best part is that it seamlessly integrates with Minimal API.