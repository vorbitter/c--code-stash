Filtros mínimos de API permitem que os desenvolvedores implementem a lógica de negócios que dá suporte a:

- Executando o código antes e depois do manipulador de ponto de extremidade.
- Inspecionar e modificar parâmetros fornecidos durante uma invocação do manipulador de ponto de extremidade.
- Interceptando o comportamento de resposta de um manipulador de ponto de extremidade.

Os filtros podem ser úteis nos seguintes cenários:

- Validação dos parâmetros de solicitação e do corpo que são enviados a um ponto de extremidade.
- Registro em log de informações sobre a solicitação e a resposta.
- Validação se uma solicitação está buscando uma versão de API com suporte.

Os filtros podem ser registrados fornecendo um [Delegado](https://learn.microsoft.com/pt-br/dotnet/csharp/programming-guide/delegates/) que usa um [`EndpointFilterInvocationContext`](https://github.com/dotnet/aspnetcore/blob/main/src/Http/Http.Abstractions/src/EndpointFilterInvocationContext.cs) e retorna um [`EndpointFilterDelegate`](https://github.com/dotnet/aspnetcore/blob/main/src/Http/Http.Abstractions/src/EndpointFilterDelegate.cs). O `EndpointFilterInvocationContext` fornece acesso ao `HttpContext` da solicitação e a uma lista `Arguments` que indica os argumentos passados para o manipulador na ordem em que aparecem na declaração do manipulador.

C#

```
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

string ColorName(string color) => $"Color specified: {color}!";

app.MapGet("/colorSelector/{color}", ColorName)
    .AddEndpointFilter(async (invocationContext, next) =>
    {
        var color = invocationContext.GetArgument<string>(0);

        if (color == "Red")
        {
            return Results.Problem("Red not allowed!");
        }
        return await next(invocationContext);
    });

app.Run();
```

O código anterior:

- Chama o método de extensão `AddEndpointFilter` para adicionar um filtro ao ponto de extremidade `/colorSelector/{color}`.
- Retorna a cor especificada, exceto pelo valor `"Red"`.
- Retorna [Results.Problem](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.results.problem) quando o `/colorSelector/Red` é solicitado.
- `next` Usa como e `EndpointFilterDelegate``invocationContext` como o `EndpointFilterInvocationContext` para invocar o próximo filtro no pipeline ou o delegado de solicitação se o último filtro tiver sido invocado.

O filtro é executado antes do manipulador de ponto de extremidade. Quando várias invocações `AddEndpointFilter` são feitas em um manipulador:

- O código de filtro chamado antes do `EndpointFilterDelegate` (`next`) é chamado é executado na ordem da ordem FIFO (First In, First Out).
- O código de filtro chamado após o `EndpointFilterDelegate` (`next`) é chamado é executado na ordem da ordem FILO (First In, Last Out).

C#

```
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.MapGet("/", () =>
    {
        app.Logger.LogInformation("             Endpoint");
        return "Test of multiple filters";
    })
    .AddEndpointFilter(async (efiContext, next) =>
    {
        app.Logger.LogInformation("Before first filter");
        var result = await next(efiContext);
        app.Logger.LogInformation("After first filter");
        return result;
    })
    .AddEndpointFilter(async (efiContext, next) =>
    {
        app.Logger.LogInformation(" Before 2nd filter");
        var result = await next(efiContext);
        app.Logger.LogInformation(" After 2nd filter");
        return result;
    })
    .AddEndpointFilter(async (efiContext, next) =>
    {
        app.Logger.LogInformation("     Before 3rd filter");
        var result = await next(efiContext);
        app.Logger.LogInformation("     After 3rd filter");
        return result;
    });

app.Run();
```

No código anterior, os filtros e o ponto de extremidade registram a seguinte saída:

CLI do .NET

```
Before first filter
    Before 2nd filter
        Before 3rd filter
            Endpoint
        After 3rd filter
    After 2nd filter
After first filter
```

O código a seguir usa filtros que implementam a interface `IEndpointFilter`:

C#

```
using Filters.EndpointFilters;

var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.MapGet("/", () =>
    {
        app.Logger.LogInformation("Endpoint");
        return "Test of multiple filters";
    })
    .AddEndpointFilter<AEndpointFilter>()
    .AddEndpointFilter<BEndpointFilter>()
    .AddEndpointFilter<CEndpointFilter>();

app.Run();
```

No código anterior, os logs de filtros e manipuladores mostram a ordem em que são executados:

CLI do .NET

```
AEndpointFilter Before next
BEndpointFilter Before next
CEndpointFilter Before next
      Endpoint
CEndpointFilter After next
BEndpointFilter After next
AEndpointFilter After next
```

Os filtros que implementam a interface `IEndpointFilter` são mostrados no exemplo a seguir:

C#

```

namespace Filters.EndpointFilters;

public abstract class ABCEndpointFilters : IEndpointFilter
{
    protected readonly ILogger Logger;
    private readonly string _methodName;

    protected ABCEndpointFilters(ILoggerFactory loggerFactory)
    {
        Logger = loggerFactory.CreateLogger<ABCEndpointFilters>();
        _methodName = GetType().Name;
    }

    public virtual async ValueTask<object?> InvokeAsync(EndpointFilterInvocationContext context,
        EndpointFilterDelegate next)
    {
        Logger.LogInformation("{MethodName} Before next", _methodName);
        var result = await next(context);
        Logger.LogInformation("{MethodName} After next", _methodName);
        return result;
    }
}

class AEndpointFilter : ABCEndpointFilters
{
    public AEndpointFilter(ILoggerFactory loggerFactory) : base(loggerFactory) { }
}

class BEndpointFilter : ABCEndpointFilters
{
    public BEndpointFilter(ILoggerFactory loggerFactory) : base(loggerFactory) { }
}

class CEndpointFilter : ABCEndpointFilters
{
    public CEndpointFilter(ILoggerFactory loggerFactory) : base(loggerFactory) { }
}
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/min-api-filters?view=aspnetcore-8.0#validate-an-object-with-a-filter)

## Validar um objeto com um filtro

Considere um filtro que valida um objeto `Todo`:

C#

```
app.MapPut("/todoitems/{id}", async (Todo inputTodo, int id, TodoDb db) =>
{
    var todo = await db.Todos.FindAsync(id);

    if (todo is null) return Results.NotFound();

    todo.Name = inputTodo.Name;
    todo.IsComplete = inputTodo.IsComplete;
    
    await db.SaveChangesAsync();

    return Results.NoContent();
}).AddEndpointFilter(async (efiContext, next) =>
{
    var tdparam = efiContext.GetArgument<Todo>(0);

    var validationError = Utilities.IsValid(tdparam);

    if (!string.IsNullOrEmpty(validationError))
    {
        return Results.Problem(validationError);
    }
    return await next(efiContext);
});
```

No código anterior:

- O objeto `EndpointFilterInvocationContext` fornece acesso aos parâmetros associados a uma solicitação específica emitida para o ponto de extremidade por meio do método `GetArguments`.
- O filtro é registrado usando um `delegate` que usa um `EndpointFilterInvocationContext` e retorna um `EndpointFilterDelegate`.

Além de serem passados como delegados, os filtros podem ser registrados implementando a interface `IEndpointFilter`. O código a seguir mostra o filtro anterior encapsulado em uma classe que implementa `IEndpointFilter`:

C#

```
public class TodoIsValidFilter : IEndpointFilter
{
    private ILogger _logger;

    public TodoIsValidFilter(ILoggerFactory loggerFactory)
    {
        _logger = loggerFactory.CreateLogger<TodoIsValidFilter>();
    }

    public async ValueTask<object?> InvokeAsync(EndpointFilterInvocationContext efiContext, 
        EndpointFilterDelegate next)
    {
        var todo = efiContext.GetArgument<Todo>(0);

        var validationError = Utilities.IsValid(todo!);

        if (!string.IsNullOrEmpty(validationError))
        {
            _logger.LogWarning(validationError);
            return Results.Problem(validationError);
        }
        return await next(efiContext);
    }
}
```

Os filtros que implementam a interface `IEndpointFilter` podem resolve dependências da [DI (Injeção de Dependência),](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-8.0) conforme mostrado no código anterior. Embora os filtros possam resolve dependências da DI, os próprios filtros _**não**_ podem ser resolvidos por meio da DI.

O `ToDoIsValidFilter` é aplicado aos seguintes pontos de extremidade:

C#

```
app.MapPut("/todoitems2/{id}", async (Todo inputTodo, int id, TodoDb db) =>
{
    var todo = await db.Todos.FindAsync(id);

    if (todo is null) return Results.NotFound();

    todo.Name = inputTodo.Name;
    todo.IsComplete = inputTodo.IsComplete;

    await db.SaveChangesAsync();

    return Results.NoContent();
}).AddEndpointFilter<TodoIsValidFilter>();

app.MapPost("/todoitems", async (Todo todo, TodoDb db) =>
{
    db.Todos.Add(todo);
    await db.SaveChangesAsync();

    return Results.Created($"/todoitems/{todo.Id}", todo);
}).AddEndpointFilter<TodoIsValidFilter>();
```

O filtro a seguir valida o objeto `Todo` e modifica a propriedade `Name`:

C#

```
public class TodoIsValidUcFilter : IEndpointFilter
{
    public async ValueTask<object?> InvokeAsync(EndpointFilterInvocationContext efiContext, 
        EndpointFilterDelegate next)
    {
        var todo = efiContext.GetArgument<Todo>(0);
        todo.Name = todo.Name!.ToUpper();

        var validationError = Utilities.IsValid(todo!);

        if (!string.IsNullOrEmpty(validationError))
        {
            return Results.Problem(validationError);
        }
        return await next(efiContext);
    }
}
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/min-api-filters?view=aspnetcore-8.0#register-a-filter-using-an-endpoint-filter-factory)

## Registrar um filtro usando uma fábrica de filtros de ponto de extremidade

Em alguns cenários, pode ser necessário armazenar em cache algumas das informações fornecidas no [`MethodInfo`](https://learn.microsoft.com/pt-br/dotnet/api/system.reflection.methodinfo) em um filtro. Por exemplo, vamos supor que queríamos verificar se o manipulador ao qual um filtro de ponto de extremidade está anexado tem um primeiro parâmetro que é avaliado como um tipo `Todo`.

C#

```
app.MapPut("/todoitems/{id}", async (Todo inputTodo, int id, TodoDb db) =>
{
    var todo = await db.Todos.FindAsync(id);

    if (todo is null) return Results.NotFound();

    todo.Name = inputTodo.Name;
    todo.IsComplete = inputTodo.IsComplete;
    
    await db.SaveChangesAsync();

    return Results.NoContent();
}).AddEndpointFilterFactory((filterFactoryContext, next) =>
{
    var parameters = filterFactoryContext.MethodInfo.GetParameters();
    if (parameters.Length >= 1 && parameters[0].ParameterType == typeof(Todo))
    {
        return async invocationContext =>
        {
            var todoParam = invocationContext.GetArgument<Todo>(0);

            var validationError = Utilities.IsValid(todoParam);

            if (!string.IsNullOrEmpty(validationError))
            {
                return Results.Problem(validationError);
            }
            return await next(invocationContext);
        };
    }
    return invocationContext => next(invocationContext); 
});
```

No código anterior:

- O objeto `EndpointFilterFactoryContext` fornece acesso ao [`MethodInfo`](https://learn.microsoft.com/pt-br/dotnet/api/system.reflection.methodinfo) associado ao manipulador do ponto de extremidade.
- A assinatura do manipulador é examinada inspecionando `MethodInfo` da assinatura de tipo esperada. Se a assinatura esperada for encontrada, o filtro de validação será registrado no ponto de extremidade. Esse padrão de fábrica é útil para registrar um filtro que depende da assinatura do manipulador de ponto de extremidade de destino.
- Se uma assinatura correspondente não for encontrada, um filtro de passagem será registrado.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/min-api-filters?view=aspnetcore-8.0#register-a-filter-on-controller-actions)

## Registrar um filtro em ações do controlador

Em alguns cenários, pode ser necessário aplicar a mesma lógica de filtro para pontos de extremidade baseados no manipulador de rotas e ações do controlador. Para esse cenário, é possível invocar `AddEndpointFilter` para `ControllerActionEndpointConventionBuilder` dar suporte à execução da mesma lógica de filtro em ações e pontos de extremidade.

C#

```
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.MapController()
    .AddEndpointFilter(async (efiContext, next) =>
    {
        efiContext.HttpContext.Items["endpointFilterCalled"] = true;
        var result = await next(efiContext);
        return result;
    });

app.Run();
```