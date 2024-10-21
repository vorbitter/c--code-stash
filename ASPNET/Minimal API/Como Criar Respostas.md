Os ponto de extremidade mínimos dão suporte aos seguintes tipos de valores retornados:

1. `string` - Isso inclui `Task<string>` e `ValueTask<string>`.
2. `T` (Qualquer outro tipo) – Isso inclui `Task<T>` e `ValueTask<T>`.
3. Baseado em `IResult` - Isso inclui `Task<IResult>` e `ValueTask<IResult>`.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-8.0#string-return-values)

## Valores retornados de `string`

|Comportamento|Tipo de conteúdo|
|---|---|
|A estrutura grava a cadeia de caracteres diretamente na resposta.|`text/plain`|

Considere o manipulador de rotas a seguir, que retorna um texto `Hello world`.

C#

```
app.MapGet("/hello", () => "Hello World");
```

O código status `200` é retornado com o cabeçalho Content-Type `text/plain` e o conteúdo a seguir.

text

```
Hello World
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-8.0#t-any-other-type-return-values)

## `T` (Qualquer outro tipo) retorna valores

|Comportamento|Content-Type|
|---|---|
|A estrutura JSON serializa a resposta.|`application/json`|

Considere o manipulador de rotas a seguir, que retorna um tipo anônimo que contém uma propriedade de cadeia de caracteres `Message`.

C#

```
app.MapGet("/hello", () => new { Message = "Hello World" });
```

O código status `200` é retornado com o cabeçalho Content-Type `application/json` e o conteúdo a seguir.

JSON

```
{"message":"Hello World"}
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-8.0#iresult-return-values)

## `IResult` retorna valores

|Comportamento|Tipo de conteúdo|
|---|---|
|A estrutura chama [IResult.ExecuteAsync](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iresult.executeasync).|Decidido pela implementação de `IResult`.|

A interface `IResult` define um contrato que representa o resultado de um ponto de extremidade HTTP. A classe estática [Resultados](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.results) o [TypedResults](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.typedresults) estático são usados para criar vários objetos `IResult` que representam diferentes tipos de respostas.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-8.0#typedresults-vs-results)

### TypedResults x Results

As classes estáticas [Results](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.results) e [TypedResults](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.typedresults) fornecem conjuntos semelhantes de auxiliares de resultados. A classe `TypedResults` é o equivalente _tipado_ da classe `Results`. No entanto, o tipo de retorno dos auxiliares `Results` é [IResult](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iresult), enquanto o tipo de retorno de cada auxiliar de `TypedResults` é um dos tipos de implementação de `IResult`. A diferença significa que, para auxiliares `Results`, uma conversão é necessária quando o tipo concreto é necessário, por exemplo, para teste de unidade. Os tipos de implementação são definidos no namespace [Microsoft.AspNetCore.Http.HttpResults](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httpresults).

Retornar `TypedResults` em vez de `Results` tem as seguintes vantagens:

- Os auxiliares de `TypedResults` retornam objetos fortemente tipado, o que pode melhorar a legibilidade do código, o teste de unidade e reduzir a chance de erros de runtime.
- O tipo de implementação [fornece automaticamente os metadados de tipo de resposta para OpenAPI](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/openapi/aspnetcore-openapi#describe-response-types) para descrever o ponto de extremidade.

Considere o ponto de extremidade a seguir, para o qual um código de status `200 OK` com a resposta JSON esperada é produzida.

C#

```
app.MapGet("/hello", () => Results.Ok(new Message() { Text = "Hello World!" }))
    .Produces<Message>();
```

Para documentar esse ponto de extremidade corretamente, o método de extensões `Produces` é chamado. No entanto, não é necessário chamar `Produces` se `TypedResults` for usado em vez de `Results`, conforme mostrado no código a seguir. `TypedResults` fornece automaticamente os metadados para o ponto de extremidade.

C#

```
app.MapGet("/hello2", () => TypedResults.Ok(new Message() { Text = "Hello World!" }));
```

Para obter mais informações sobre como descrever um tipo de resposta, consulte [Suporte a OpenAPI em APIs mínimas](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/openapi/aspnetcore-openapi#describe-response-types-1).

Conforme mencionado anteriormente, ao usar `TypedResults`, uma conversão não é necessária. Considere a API mínima a seguir que retorna uma classe `TypedResults`

C#

```
public static async Task<Ok<Todo[]>> GetAllTodos(TodoGroupDbContext database)
{
    var todos = await database.Todos.ToArrayAsync();
    return TypedResults.Ok(todos);
}
```

O teste a seguir verifica o tipo concreto completo:

C#

```
[Fact]
public async Task GetAllReturnsTodosFromDatabase()
{
    // Arrange
    await using var context = new MockDb().CreateDbContext();

    context.Todos.Add(new Todo
    {
        Id = 1,
        Title = "Test title 1",
        Description = "Test description 1",
        IsDone = false
    });

    context.Todos.Add(new Todo
    {
        Id = 2,
        Title = "Test title 2",
        Description = "Test description 2",
        IsDone = true
    });

    await context.SaveChangesAsync();

    // Act
    var result = await TodoEndpointsV1.GetAllTodos(context);

    //Assert
    Assert.IsType<Ok<Todo[]>>(result);
    
    Assert.NotNull(result.Value);
    Assert.NotEmpty(result.Value);
    Assert.Collection(result.Value, todo1 =>
    {
        Assert.Equal(1, todo1.Id);
        Assert.Equal("Test title 1", todo1.Title);
        Assert.False(todo1.IsDone);
    }, todo2 =>
    {
        Assert.Equal(2, todo2.Id);
        Assert.Equal("Test title 2", todo2.Title);
        Assert.True(todo2.IsDone);
    });
}
```

Como todos os métodos em `Results` retornaam `IResult` em sua assinatura, o compilador infere automaticamente isso como o tipo de retorno do delegado de solicitação ao retornar resultados diferentes de um único ponto de extremidade. `TypedResults` requer o uso de `Results<T1, TN>` desses delegados.

O método a seguir é compilado porque [`Results.Ok`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.results.ok) e [`Results.NotFound`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.results.notfound) são declarados como retornando `IResult`, mesmo que os tipos concretos reais dos objetos retornados sejam diferentes:

C#

```
app.MapGet("/todoitems/{id}", async (int id, TodoDb db) =>
    await db.Todos.FindAsync(id)
        is Todo todo
            ? Results.Ok(todo)
            : Results.NotFound());
```

O método a seguir não é compilado, pois `TypedResults.Ok` e `TypedResults.NotFound` são declarados como retornando tipos diferentes e o compilador não tentará inferir o melhor tipo de correspondência:

C#

```
app.MapGet("/todoitems/{id}", async (int id, TodoDb db) =>
     await db.Todos.FindAsync(id)
     is Todo todo
        ? TypedResults.Ok(todo)
        : TypedResults.NotFound());
```

Para usar `TypedResults`, o tipo de retorno deve ser totalmente declarado, o que quando assíncrono requer o encapsulador `Task<>`. O uso de `TypedResults` é mais detalhado, mas essa é a compensação por ter as informações de tipo estaticamente disponíveis e, portanto, capazes de auto-descrever para o OpenAPI:

C#

```
app.MapGet("/todoitems/{id}", async Task<Results<Ok<Todo>, NotFound>> (int id, TodoDb db) =>
   await db.Todos.FindAsync(id)
    is Todo todo
       ? TypedResults.Ok(todo)
       : TypedResults.NotFound());
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-8.0#resultstresult1-tresultn)

### Resulta em <TResult1, TResultN>

Use [`Results<TResult1, TResultN>`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httpresults.results-2) como o tipo de retorno do manipulador de ponto de extremidade em vez de `IResult` quando:

- Vários tipos de implementação `IResult` são retornados do manipulador de ponto de extremidade.
- A classe estática `TypedResult` é usada para criar os objetos `IResult`.

Essa alternativa é melhor do que retornar `IResult` porque os tipos de união genéricos retêm automaticamente os metadados do ponto de extremidade. E como os tipos de união `Results<TResult1, TResultN>` implementam operadores de conversão implícita, o compilador pode converter automaticamente os tipos especificados nos argumentos genéricos em uma instância do tipo de união.

Essa funcionalidade tem o benefício adicional de verificar durante o tempo de compilação se um manipulador de rotas retorna apenas os resultados que ele declara retornar. Tentar retornar um tipo que não é declarado como um dos argumentos genéricos para `Results<>` resulta em um erro de compilação.

Considere o ponto de extremidade a seguir, para o qual um código de status `400 BadRequest` é retornado quando `orderId` é maior que `999`. Caso contrário, ele produzirá um `200 OK` com o conteúdo esperado.

C#

```
app.MapGet("/orders/{orderId}", IResult (int orderId)
    => orderId > 999 ? TypedResults.BadRequest() : TypedResults.Ok(new Order(orderId)))
    .Produces(400)
    .Produces<Order>();
```

Para documentar esse ponto de extremidade corretamente, o método de extensões `Produces` é chamado. No entanto, como o auxiliar de `TypedResults` inclui automaticamente os metadados para o ponto de extremidade, você pode retornar o tipo de união `Results<T1, Tn>` ao invés disso, conforme mostrado no código a seguir.

C#

```
app.MapGet("/orders/{orderId}", Results<BadRequest, Ok<Order>> (int orderId) 
    => orderId > 999 ? TypedResults.BadRequest() : TypedResults.Ok(new Order(orderId)));
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-8.0#built-in-results)

### Resultados internos

Auxiliares de resultados comuns existem no [Results](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.results) e nas classes estáticas [TypedResults](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.typedresults). É preferível retornar `TypedResults` a retornar `Results`. Para obter mais informações, consulte [TypedResults vs Resultados](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses#typedresults-vs-results).

As seções a seguir demonstram o uso dos auxiliares de resultados comuns.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-8.0#json)

#### JSON

C#

```
app.MapGet("/hello", () => Results.Json(new { Message = "Hello World" }));
```

[WriteAsJsonAsync](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httpresponsejsonextensions.writeasjsonasync) é uma maneira alternativa de retornar JSON:

C#

```
app.MapGet("/", (HttpContext context) => context.Response.WriteAsJsonAsync
    (new { Message = "Hello World" }));
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-8.0#custom-status-code)

#### Código de status personalizado

C#

```
app.MapGet("/405", () => Results.StatusCode(405));
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-8.0#text)

#### Texto

C#

```
app.MapGet("/text", () => Results.Text("This is some text"));
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-8.0#stream)

#### Stream

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

var proxyClient = new HttpClient();
app.MapGet("/pokemon", async () => 
{
    var stream = await proxyClient.GetStreamAsync("http://contoso/pokedex.json");
    // Proxy the response as JSON
    return Results.Stream(stream, "application/json");
});

app.Run();
```

as sobrecargas de [`Results.Stream`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.results.stream?view=aspnetcore-7.0&preserve-view=true) permitem o acesso ao fluxo de resposta HTTP subjacente sem buffer. O exemplo a seguir usa [ImageSharp](https://sixlabors.com/products/imagesharp) para retornar um tamanho reduzido da imagem especificada:

C#

```
using SixLabors.ImageSharp;
using SixLabors.ImageSharp.Formats.Jpeg;
using SixLabors.ImageSharp.Processing;

var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.MapGet("/process-image/{strImage}", (string strImage, HttpContext http, CancellationToken token) =>
{
    http.Response.Headers.CacheControl = $"public,max-age={TimeSpan.FromHours(24).TotalSeconds}";
    return Results.Stream(stream => ResizeImageAsync(strImage, stream, token), "image/jpeg");
});

async Task ResizeImageAsync(string strImage, Stream stream, CancellationToken token)
{
    var strPath = $"wwwroot/img/{strImage}";
    using var image = await Image.LoadAsync(strPath, token);
    int width = image.Width / 2;
    int height = image.Height / 2;
    image.Mutate(x =>x.Resize(width, height));
    await image.SaveAsync(stream, JpegFormat.Instance, cancellationToken: token);
}
```

O exemplo a seguir transmite uma imagem do [Armazenamento de Blobs do Azure](https://learn.microsoft.com/pt-br/azure/storage/blobs/storage-blobs-introduction):

C#

```
app.MapGet("/stream-image/{containerName}/{blobName}", 
    async (string blobName, string containerName, CancellationToken token) =>
{
    var conStr = builder.Configuration["blogConStr"];
    BlobContainerClient blobContainerClient = new BlobContainerClient(conStr, containerName);
    BlobClient blobClient = blobContainerClient.GetBlobClient(blobName);
    return Results.Stream(await blobClient.OpenReadAsync(cancellationToken: token), "image/jpeg");
});
```

O exemplo a seguir transmite um vídeo de um Blob do Azure:

C#

```
// GET /stream-video/videos/earth.mp4
app.MapGet("/stream-video/{containerName}/{blobName}",
     async (HttpContext http, CancellationToken token, string blobName, string containerName) =>
{
    var conStr = builder.Configuration["blogConStr"];
    BlobContainerClient blobContainerClient = new BlobContainerClient(conStr, containerName);
    BlobClient blobClient = blobContainerClient.GetBlobClient(blobName);
    
    var properties = await blobClient.GetPropertiesAsync(cancellationToken: token);
    
    DateTimeOffset lastModified = properties.Value.LastModified;
    long length = properties.Value.ContentLength;
    
    long etagHash = lastModified.ToFileTime() ^ length;
    var entityTag = new EntityTagHeaderValue('\"' + Convert.ToString(etagHash, 16) + '\"');
    
    http.Response.Headers.CacheControl = $"public,max-age={TimeSpan.FromHours(24).TotalSeconds}";

    return Results.Stream(await blobClient.OpenReadAsync(cancellationToken: token), 
        contentType: "video/mp4",
        lastModified: lastModified,
        entityTag: entityTag,
        enableRangeProcessing: true);
});
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-8.0#redirect)

#### Redirecionar

C#

```
app.MapGet("/old-path", () => Results.Redirect("/new-path"));
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-8.0#file)

#### Arquivo

C#

```
app.MapGet("/download", () => Results.File("myfile.text"));
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-8.0#httpresult-interfaces)

### Interfaces HttpResult

As interfaces abaixo no namespace [Microsoft.AspNetCore.Http](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http) fornecem uma maneira de detectar o tipo `IResult` em runtime, o que é um padrão comum em implementações de filtro:

- [IContentTypeHttpResult](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.icontenttypehttpresult)
- [IFileHttpResult](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.ifilehttpresult)
- [INestedHttpResult](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.inestedhttpresult)
- [IStatusCodeHttpResult](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.istatuscodehttpresult)
- [IValueHttpResult](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.ivaluehttpresult)
- [IValueHttpResult<\TValue>](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.ivaluehttpresult-1)

Aqui está um exemplo de um filtro que usa uma destas interfaces:

C#

```
app.MapGet("/weatherforecast", (int days) =>
{
    if (days <= 0)
    {
        return Results.BadRequest();
    }

    var forecast = Enumerable.Range(1, days).Select(index =>
       new WeatherForecast(DateTime.Now.AddDays(index), Random.Shared.Next(-20, 55), "Cool"))
        .ToArray();
    return Results.Ok(forecast);
}).
AddEndpointFilter(async (context, next) =>
{
    var result = await next(context);

    return result switch
    {
        IValueHttpResult<WeatherForecast[]> weatherForecastResult => new WeatherHttpResult(weatherForecastResult.Value),
        _ => result
    };
});
```

Para obter mais informações, consulte [Filtros em aplicativos de APIs mínimas](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/min-api-filters?view=aspnetcore-8.0) e [tipos de implementação IResult](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/test-min-api?view=aspnetcore-8.0#iresult-implementation-types).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-8.0#customizing-responses)

## Personalizando respostas

Aplicativos podem controlar as respostas implementando um tipo [IResult](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iresult) personalizado. O código a seguir é um exemplo de um tipo de resultado HTML:

C#

```
using System.Net.Mime;
using System.Text;
static class ResultsExtensions
{
    public static IResult Html(this IResultExtensions resultExtensions, string html)
    {
        ArgumentNullException.ThrowIfNull(resultExtensions);

        return new HtmlResult(html);
    }
}

class HtmlResult : IResult
{
    private readonly string _html;

    public HtmlResult(string html)
    {
        _html = html;
    }

    public Task ExecuteAsync(HttpContext httpContext)
    {
        httpContext.Response.ContentType = MediaTypeNames.Text.Html;
        httpContext.Response.ContentLength = Encoding.UTF8.GetByteCount(_html);
        return httpContext.Response.WriteAsync(_html);
    }
}
```

Recomenda-se adicionar um método de extensão a [Microsoft.AspNetCore.Http.IResultExtensions](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iresultextensions) para tornar esses resultados personalizados mais detectáveis.

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/html", () => Results.Extensions.Html(@$"<!doctype html>
<html>
    <head><title>miniHTML</title></head>
    <body>
        <h1>Hello World</h1>
        <p>The time on the server is {DateTime.Now:O}</p>
    </body>
</html>"));

app.Run();
```

Além disso, um tipo personalizado `IResult` pode fornecer sua própria anotação implementando a interface [IEndpointMetadataProvider](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.metadata.iendpointmetadataprovider). Por exemplo, o código a seguir adiciona uma anotação ao tipo anterior `HtmlResult` que descreve a resposta produzida pelo ponto de extremidade.

C#

```
class HtmlResult : IResult, IEndpointMetadataProvider
{
    private readonly string _html;

    public HtmlResult(string html)
    {
        _html = html;
    }

    public Task ExecuteAsync(HttpContext httpContext)
    {
        httpContext.Response.ContentType = MediaTypeNames.Text.Html;
        httpContext.Response.ContentLength = Encoding.UTF8.GetByteCount(_html);
        return httpContext.Response.WriteAsync(_html);
    }

    public static void PopulateMetadata(MethodInfo method, EndpointBuilder builder)
    {
        builder.Metadata.Add(new ProducesHtmlMetadata());
    }
}
```

O `ProducesHtmlMetadata` é uma implementação de [IProducesResponseTypeMetadata](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.metadata.iproducesresponsetypemetadata) que define o tipo de conteúdo de resposta produzido `text/html` e o código de status `200 OK`.

C#

```
internal sealed class ProducesHtmlMetadata : IProducesResponseTypeMetadata
{
    public Type? Type => null;

    public int StatusCode => 200;

    public IEnumerable<string> ContentTypes { get; } = new[] { MediaTypeNames.Text.Html };
}
```

Uma abordagem alternativa é usar o [Microsoft.AspNetCore.Mvc.ProducesAttribute](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.mvc.producesattribute) para descrever a resposta produzida. O código a seguir altera o método `PopulateMetadata` para usar `ProducesAttribute`.

C#

```
public static void PopulateMetadata(MethodInfo method, EndpointBuilder builder)
{
    builder.Metadata.Add(new ProducesAttribute(MediaTypeNames.Text.Html));
}
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-8.0#configure-json-serialization-options)

## Configurar opções de serialização JSON

Por padrão, aplicativos de APIs mínimas usam opções [`Web defaults`](https://learn.microsoft.com/pt-br/dotnet/standard/serialization/system-text-json-configure-options#web-defaults-for-jsonserializeroptions) durante a serialização e desserialização JSON.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-8.0#configure-json-serialization-options-globally)

### Configurar opções de serialização JSON globalmente

As opções de um aplicativo podem ser configuradas globalmente aplicativo invocando [ConfigureHttpJsonOptions](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.extensions.dependencyinjection.httpjsonserviceextensions.configurehttpjsonoptions). O exemplo a seguir inclui campos públicos e formata a saída JSON.

C#

```
var builder = WebApplication.CreateBuilder(args);

builder.Services.ConfigureHttpJsonOptions(options => {
    options.SerializerOptions.WriteIndented = true;
    options.SerializerOptions.IncludeFields = true;
});

var app = builder.Build();

app.MapPost("/", (Todo todo) => {
    if (todo is not null) {
        todo.Name = todo.NameField;
    }
    return todo;
});

app.Run();

class Todo {
    public string? Name { get; set; }
    public string? NameField;
    public bool IsComplete { get; set; }
}
// If the request body contains the following JSON:
//
// {"nameField":"Walk dog", "isComplete":false}
//
// The endpoint returns the following JSON:
//
// {
//    "name":"Walk dog",
//    "nameField":"Walk dog",
//    "isComplete":false
// }
```

Como os campos são incluídos, o código anterior lê `NameField` e o inclui na saída JSON.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-8.0#configure-json-serialization-options-for-an-endpoint)

### Configurar as opções de desserialização JSON de um ponto de extremidade

Para configurar as opções de serialização de um ponto de extremidade, invoque [Results.Json](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.results.json) e passe para ele um objeto [JsonSerializerOptions](https://learn.microsoft.com/pt-br/dotnet/api/system.text.json.jsonserializeroptions), conforme mostrado no exemplo a seguir:

C#

```
using System.Text.Json;

var app = WebApplication.Create();

var options = new JsonSerializerOptions(JsonSerializerDefaults.Web)
    { WriteIndented = true };

app.MapGet("/", () => 
    Results.Json(new Todo { Name = "Walk dog", IsComplete = false }, options));

app.Run();

class Todo
{
    public string? Name { get; set; }
    public bool IsComplete { get; set; }
}
// The endpoint returns the following JSON:
//
// {
//   "name":"Walk dog",
//   "isComplete":false
// }
```

Como alternativa, use uma sobrecarga de [WriteAsJsonAsync](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httpresponsejsonextensions.writeasjsonasync) que aceite um objeto [JsonSerializerOptions](https://learn.microsoft.com/pt-br/dotnet/api/system.text.json.jsonserializeroptions). O exemplo a seguir usa essa sobrecarga para formatar a saída JSON:


```
using System.Text.Json;

var app = WebApplication.Create();

var options = new JsonSerializerOptions(JsonSerializerDefaults.Web) {
    WriteIndented = true };

app.MapGet("/", (HttpContext context) =>
    context.Response.WriteAsJsonAsync<Todo>(
        new Todo { Name = "Walk dog", IsComplete = false }, options));

app.Run();

class Todo
{
    public string? Name { get; set; }
    public bool IsComplete { get; set; }
}
// The endpoint returns the following JSON:
//
// {
//   "name":"Walk dog",
//   "isComplete":false
// }
```