Um `WebApplication` configurado oferece suporte a `Map{Verb}` e [MapMethods](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.endpointroutebuilderextensions.mapmethods) em que `{Verb}` é um método HTTP com caixa Pascal, como `Get`, `Post`, `Put` ou `Delete`:

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "This is a GET");
app.MapPost("/", () => "This is a POST");
app.MapPut("/", () => "This is a PUT");
app.MapDelete("/", () => "This is a DELETE");

app.MapMethods("/options-or-head", new[] { "OPTIONS", "HEAD" }, 
                          () => "This is an options or head request ");

app.Run();
```

Os argumentos [Delegate](https://learn.microsoft.com/pt-br/dotnet/api/system.delegate) passados para esses métodos são chamados de "manipuladores de rota".

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/route-handlers?view=aspnetcore-8.0#route-handlers)

## Manipuladores de rotas

Manipuladores de rotas são métodos executados quando a rota corresponde. Os manipuladores de rotas podem ser uma expressão lambda, uma função local, um método de instância ou um método estático. Os manipuladores de rota podem ser síncronos ou assíncronos.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/route-handlers?view=aspnetcore-8.0#lambda-expression)

### Expressão lambda

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/inline", () => "This is an inline lambda");

var handler = () => "This is a lambda variable";

app.MapGet("/", handler);

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/route-handlers?view=aspnetcore-8.0#local-function)

### Função local

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

string LocalFunction() => "This is local function";

app.MapGet("/", LocalFunction);

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/route-handlers?view=aspnetcore-8.0#instance-method)

### Método de instância

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

var handler = new HelloHandler();

app.MapGet("/", handler.Hello);

app.Run();

class HelloHandler
{
    public string Hello()
    {
        return "Hello Instance method";
    }
}
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/route-handlers?view=aspnetcore-8.0#static-method)

### Método estático

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", HelloHandler.Hello);

app.Run();

class HelloHandler
{
    public static string Hello()
    {
        return "Hello static method";
    }
}
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/route-handlers?view=aspnetcore-8.0#endpoint-defined-outside-of-programcs)

### Ponto de extremidade definido fora do `Program.cs`

As APIs mínimas não precisam estar localizadas em `Program.cs`.

`Program.cs`

C#

```
using MinAPISeparateFile;

var builder = WebApplication.CreateSlimBuilder(args);

var app = builder.Build();

TodoEndpoints.Map(app);

app.Run();
```

`TodoEndpoints.cs`

C#

```
namespace MinAPISeparateFile;

public static class TodoEndpoints
{
    public static void Map(WebApplication app)
    {
        app.MapGet("/", async context =>
        {
            // Get all todo items
            await context.Response.WriteAsJsonAsync(new { Message = "All todo items" });
        });

        app.MapGet("/{id}", async context =>
        {
            // Get one todo item
            await context.Response.WriteAsJsonAsync(new { Message = "One todo item" });
        });
    }
}
```

Também consulte [Grupos de rota](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/route-handlers?view=aspnetcore-8.0#route-groups) mais adiante neste artigo.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/route-handlers?view=aspnetcore-8.0#named-endpoints-and-link-generation)

### Pontos de extremidade nomeados e geração de link

Os pontos de extremidade podem receber nomes para gerar URLs para o ponto de extremidade. O uso de um ponto de extremidade nomeado evita ter que codificar caminhos em um aplicativo:

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/hello", () => "Hello named route")
   .WithName("hi");

app.MapGet("/", (LinkGenerator linker) => 
        $"The link to the hello route is {linker.GetPathByName("hi", values: null)}");

app.Run();
```

O código anterior exibe `The link to the hello route is /hello` do ponto de extremidade `/`.

**OBSERVAÇÃO**: os nomes de ponto de extremidade diferenciam maiúsculas e minúsculas.

Nomes de ponto de extremidade:

- Deve ser globalmente exclusivo.
- São usados como a ID da operação OpenAPI quando o suporte ao OpenAPI está habilitado. Para obter mais informações, confira [OpenAPI](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/openapi/aspnetcore-openapi?view=aspnetcore-8.0).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/route-handlers?view=aspnetcore-8.0#route-parameters)

### Parâmetros de rota

Os parâmetros de rota podem ser capturados como parte da definição do padrão de rota:

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/users/{userId}/books/{bookId}", 
    (int userId, int bookId) => $"The user id is {userId} and book id is {bookId}");

app.Run();
```

O código anterior retorna `The user id is 3 and book id is 7` do URI `/users/3/books/7`.

O manipulador de rotas pode declarar os parâmetros a serem capturados. Quando uma solicitação é feita em uma rota com parâmetros declarados para captura, os parâmetros são analisados e passados para o manipulador. Isso facilita a captura dos valores de uma maneira segura de tipo. No código anterior, `userId` e `bookId` são `int`.

No código anterior, se qualquer valor de rota não puder ser convertido em um `int`, uma exceção será gerada. A solicitação GET `/users/hello/books/3` gera a seguinte exceção:

**`BadHttpRequestException: Failed to bind parameter "int userId" from "hello".`**

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/route-handlers?view=aspnetcore-8.0#wildcard-and-catch-all-routes)

### Caractere curinga e capturar todas as rotas

A rota catch-all a seguir retorna `Routing to hello` do ponto de extremidade '/posts/hello':

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/posts/{*rest}", (string rest) => $"Routing to {rest}");

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/route-handlers?view=aspnetcore-8.0#route-constraints)

### Restrições de rota

As restrições de rota restringem o comportamento correspondente de uma rota.

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/todos/{id:int}", (int id) => db.Todos.Find(id));
app.MapGet("/todos/{text}", (string text) => db.Todos.Where(t => t.Text.Contains(text));
app.MapGet("/posts/{slug:regex(^[a-z0-9_-]+$)}", (string slug) => $"Post {slug}");

app.Run();
```

A tabela a seguir demonstra modelos de rota anteriores e seu comportamento:

|Modelo de rota|URI de correspondência de exemplo|
|---|---|
|`/todos/{id:int}`|`/todos/1`|
|`/todos/{text}`|`/todos/something`|
|`/posts/{slug:regex(^[a-z0-9_-]+$)}`|`/posts/mypost`|

Para obter mais informações, confira [Referência de restrição de rota](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/routing?view=aspnetcore-8.0) no [Roteamento no ASP.NET Core](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/routing?view=aspnetcore-8.0).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/route-handlers?view=aspnetcore-8.0#route-groups)

### Grupos de rotas

O método de extensão [MapGroup](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.endpointroutebuilderextensions.mapgroup) ajuda a organizar grupos de pontos de extremidade com um prefixo comum. Isso reduz o código repetitivo e permite personalizar grupos inteiros de pontos de extremidade com uma única chamada a métodos como [RequireAuthorization](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.authorizationendpointconventionbuilderextensions.requireauthorization) e [WithMetadata](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.routingendpointconventionbuilderextensions.withmetadata), que adicionam os [metadados de ponto de extremidade](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/routing?view=aspnetcore-8.0#endpoint-metadata).

Por exemplo, o código a seguir cria dois grupos de pontos de extremidade semelhantes:

C#

```
app.MapGroup("/public/todos")
    .MapTodosApi()
    .WithTags("Public");

app.MapGroup("/private/todos")
    .MapTodosApi()
    .WithTags("Private")
    .AddEndpointFilterFactory(QueryPrivateTodos)
    .RequireAuthorization();


EndpointFilterDelegate QueryPrivateTodos(EndpointFilterFactoryContext factoryContext, EndpointFilterDelegate next)
{
    var dbContextIndex = -1;

    foreach (var argument in factoryContext.MethodInfo.GetParameters())
    {
        if (argument.ParameterType == typeof(TodoDb))
        {
            dbContextIndex = argument.Position;
            break;
        }
    }

    // Skip filter if the method doesn't have a TodoDb parameter.
    if (dbContextIndex < 0)
    {
        return next;
    }

    return async invocationContext =>
    {
        var dbContext = invocationContext.GetArgument<TodoDb>(dbContextIndex);
        dbContext.IsPrivate = true;

        try
        {
            return await next(invocationContext);
        }
        finally
        {
            // This should only be relevant if you're pooling or otherwise reusing the DbContext instance.
            dbContext.IsPrivate = false;
        }
    };
}
```

C#

```
public static RouteGroupBuilder MapTodosApi(this RouteGroupBuilder group)
{
    group.MapGet("/", GetAllTodos);
    group.MapGet("/{id}", GetTodo);
    group.MapPost("/", CreateTodo);
    group.MapPut("/{id}", UpdateTodo);
    group.MapDelete("/{id}", DeleteTodo);

    return group;
}
```

Nesse cenário, você pode usar um endereço relativo para o cabeçalho `Location` no resultado `201 Created`:

C#

```
public static async Task<Created<Todo>> CreateTodo(Todo todo, TodoDb database)
{
    await database.AddAsync(todo);
    await database.SaveChangesAsync();

    return TypedResults.Created($"{todo.Id}", todo);
}
```

O primeiro grupo de pontos de extremidade corresponderá apenas às solicitações prefixadas com `/public/todos` e estará acessível sem autenticação. O segundo grupo de pontos de extremidade corresponderá apenas às solicitações prefixadas com `/private/todos` e exigirá autenticação.

A `QueryPrivateTodos` [fábrica de filtro de ponto de extremidade](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/min-api-filters?view=aspnetcore-8.0) é uma função local que modifica os `TodoDb` parâmetros do manipulador de rota para permitir o acesso e armazenamento de dados privados de tarefas.

Os grupos de rotas também permitem grupos aninhados e padrões de prefixo complexos com parâmetros de rota e restrições. No exemplo a seguir, o manipulador de rotas mapeado para o grupo `user` pode capturar os parâmetros de rota `{org}` e `{group}` definidos nos prefixos do grupo externo.

O prefixo também pode estar vazio. Isso pode ser útil para adicionar metadados ou filtros de ponto de extremidade a um grupo de pontos de extremidade, sem alterar o padrão de rota.

C#

```
var all = app.MapGroup("").WithOpenApi();
var org = all.MapGroup("{org}");
var user = org.MapGroup("{user}");
user.MapGet("", (string org, string user) => $"{org}/{user}");
```

Adicionar filtros ou metadados a um grupo é igual a adicioná-los individualmente a cada ponto de extremidade, antes de adicionar filtros ou metadados extras que possam ter sido adicionados a um grupo interno ou ponto de extremidade específico.

C#

```
var outer = app.MapGroup("/outer");
var inner = outer.MapGroup("/inner");

inner.AddEndpointFilter((context, next) =>
{
    app.Logger.LogInformation("/inner group filter");
    return next(context);
});

outer.AddEndpointFilter((context, next) =>
{
    app.Logger.LogInformation("/outer group filter");
    return next(context);
});

inner.MapGet("/", () => "Hi!").AddEndpointFilter((context, next) =>
{
    app.Logger.LogInformation("MapGet filter");
    return next(context);
});
```

No exemplo acima, o filtro externo registrará a solicitação de entrada antes do filtro interno, mesmo que tenha sido adicionado depois. Como os filtros foram aplicados a grupos diferentes, a ordem em que foram adicionados em relação uns aos outros não importa. A ordem em que os filtros são adicionados importa se aplicados ao mesmo grupo ou ponto de extremidade específico.

Uma solicitação para `/outer/inner/` registrará o seguinte:

CLI do .NET

```
/outer group filter
/inner group filter
MapGet filter
```