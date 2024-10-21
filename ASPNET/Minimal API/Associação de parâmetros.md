Associação de parâmetro é o processo de converter dados de solicitação em parâmetros fortemente tipados que são expressos por manipuladores de rota. A origem de uma associação determina de onde os parâmetros são associados. As origens de associação podem ser explícitas ou inferidas com base no método HTTP e no tipo de parâmetro.

Origens de associação com suporte:

- Valores da rota
- Cadeia de caracteres de consulta
- Cabeçalho
- Corpo (como JSON)
- Valores de formulário
- Serviços fornecidos pela injeção de dependência
- Personalizado

O manipulador de rota `GET` a seguir usa algumas dessas origens de associação de parâmetros:

C#

```
var builder = WebApplication.CreateBuilder(args);

// Added as service
builder.Services.AddSingleton<Service>();

var app = builder.Build();

app.MapGet("/{id}", (int id,
                     int page,
                     [FromHeader(Name = "X-CUSTOM-HEADER")] string customHeader,
                     Service service) => { });

class Service { }
```

A tabela a seguir mostra a relação entre os parâmetros usados no exemplo anterior e as origens de associação correspondentes.

|Parâmetro|Origem da associação|
|---|---|
|`id`|valor da rota|
|`page`|cadeia de caracteres de consulta|
|`customHeader`|cabeçalho|
|`service`|Fornecido pela injeção de dependência|

Os métodos HTTP `GET`, `HEAD`, `OPTIONS`, e `DELETE` não se associam implicitamente a partir do corpo. Para associar a partir do corpo (como JSON) para esses métodos HTTP, [associe explicitamente](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#explicit-parameter-binding) com [`[FromBody]`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.mvc.frombodyattribute) ou leia de [HttpRequest](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httprequest).

O seguinte exemplo de manipulador de rota POST usa uma origem de associação de corpo (como JSON) para o parâmetro `person`:

C#

```
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.MapPost("/", (Person person) => { });

record Person(string Name, int Age);
```

Os parâmetros nos exemplos anteriores são todos associados automaticamente a partir de dados de solicitação. Para demonstrar a conveniência que a associação de parâmetros fornece, os seguintes manipuladores de rota mostram como ler dados de solicitação diretamente da solicitação:

C#

```
app.MapGet("/{id}", (HttpRequest request) =>
{
    var id = request.RouteValues["id"];
    var page = request.Query["page"];
    var customHeader = request.Headers["X-CUSTOM-HEADER"];

    // ...
});

app.MapPost("/", async (HttpRequest request) =>
{
    var person = await request.ReadFromJsonAsync<Person>();

    // ...
});
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#explicit-parameter-binding)

### Associação de parâmetro explícita

Os atributos podem ser usados para declarar explicitamente de onde os parâmetros são associados.

C#

```
using Microsoft.AspNetCore.Mvc;

var builder = WebApplication.CreateBuilder(args);

// Added as service
builder.Services.AddSingleton<Service>();

var app = builder.Build();


app.MapGet("/{id}", ([FromRoute] int id,
                     [FromQuery(Name = "p")] int page,
                     [FromServices] Service service,
                     [FromHeader(Name = "Content-Type")] string contentType) 
                     => {});

class Service { }

record Person(string Name, int Age);
```

|Parâmetro|Origem da associação|
|---|---|
|`id`|valor de rota com o nome `id`|
|`page`|cadeia de caracteres de consulta com o nome `"p"`|
|`service`|Fornecido pela injeção de dependência|
|`contentType`|cabeçalho com o nome `"Content-Type"`|

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#explicit-binding-from-form-values)

#### Associação explícita de valores de formulário

O atributo [`[FromForm]`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.mvc.fromformattribute) associa valores de formulário:

C#

```
app.MapPost("/todos", async ([FromForm] string name,
    [FromForm] Visibility visibility, IFormFile? attachment, TodoDb db) =>
{
    var todo = new Todo
    {
        Name = name,
        Visibility = visibility
    };

    if (attachment is not null)
    {
        var attachmentName = Path.GetRandomFileName();

        using var stream = File.Create(Path.Combine("wwwroot", attachmentName));
        await attachment.CopyToAsync(stream);
    }

    db.Todos.Add(todo);
    await db.SaveChangesAsync();

    return Results.Ok();
});

// Remaining code removed for brevity.
```

Uma alternativa é usar o atributo [`[AsParameters]`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.asparametersattribute) com um tipo personalizado que tenha propriedades anotadas com `[FromForm]`. Por exemplo, o código a seguir associa de valores de formulário a propriedades do struct de registro `NewTodoRequest`:

C#

```
app.MapPost("/ap/todos", async ([AsParameters] NewTodoRequest request, TodoDb db) =>
{
    var todo = new Todo
    {
        Name = request.Name,
        Visibility = request.Visibility
    };

    if (request.Attachment is not null)
    {
        var attachmentName = Path.GetRandomFileName();

        using var stream = File.Create(Path.Combine("wwwroot", attachmentName));
        await request.Attachment.CopyToAsync(stream);

        todo.Attachment = attachmentName;
    }

    db.Todos.Add(todo);
    await db.SaveChangesAsync();

    return Results.Ok();
});

// Remaining code removed for brevity.
```

C#

```
public record struct NewTodoRequest([FromForm] string Name,
    [FromForm] Visibility Visibility, IFormFile? Attachment);
```

Para obter mais informações, confira a seção sobre [AsParameters](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#parameter-binding-for-argument-lists-with-asparameters) mais adiante neste artigo.

O [código de exemplo completo](https://github.com/dotnet/AspNetCore.Docs.Samples/tree/main/fundamentals/minimal-apis/samples/IFormFile) está no repositório [AspNetCore.Docs.Samples](https://github.com/dotnet/AspNetCore.Docs.Samples).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#secure-binding-from-iformfile-and-iformfilecollection)

#### Associação segura do IFormFile e IFormFileCollection

Há suporte para a associação do formulário complexo usando [IFormFile](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iformfile) e [IFormFileCollection](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iformfilecollection) usando o [`[FromForm]`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.mvc.fromformattribute):

C#

```
using Microsoft.AspNetCore.Antiforgery;
using Microsoft.AspNetCore.Http.HttpResults;
using Microsoft.AspNetCore.Mvc;

var builder = WebApplication.CreateBuilder();

builder.Services.AddAntiforgery();

var app = builder.Build();
app.UseAntiforgery();

// Generate a form with an anti-forgery token and an /upload endpoint.
app.MapGet("/", (HttpContext context, IAntiforgery antiforgery) =>
{
    var token = antiforgery.GetAndStoreTokens(context);
    var html = MyUtils.GenerateHtmlForm(token.FormFieldName, token.RequestToken!);
    return Results.Content(html, "text/html");
});

app.MapPost("/upload", async Task<Results<Ok<string>, BadRequest<string>>>
    ([FromForm] FileUploadForm fileUploadForm, HttpContext context,
                                                IAntiforgery antiforgery) =>
{
    await MyUtils.SaveFileWithName(fileUploadForm.FileDocument!,
              fileUploadForm.Name!, app.Environment.ContentRootPath);
    return TypedResults.Ok($"Your file with the description:" +
        $" {fileUploadForm.Description} has been uploaded successfully");
});

app.Run();
```

Os parâmetros associados à solicitação com `[FromForm]` incluem um [token antifalsificação](https://learn.microsoft.com/pt-br/aspnet/core/security/anti-request-forgery?view=aspnetcore-8.0). O token antifalsificação é validado quando a solicitação é processada. Para obter mais informações, consulte [Antifalsificação com APIs mínimas](https://learn.microsoft.com/pt-br/aspnet/core/security/anti-request-forgery?view=aspnetcore-8.0&preserve-view=true&view=aspnetcore-8.0#afwma).

Para obter mais informações, consulte [Associação de formulário nas APIs mínimas](https://andrewlock.net/exploring-the-dotnet-8-preview-form-binding-in-minimal-apis/).

O [código de exemplo completo](https://github.com/dotnet/AspNetCore.Docs.Samples/tree/main/fundamentals/minimal-apis/samples/FormBinding) está no repositório [AspNetCore.Docs.Samples](https://github.com/dotnet/AspNetCore.Docs.Samples).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#parameter-binding-with-dependency-injection)

### Associação de parâmetros com injeção de dependência

A associação de parâmetros para APIs mínimas vincula parâmetros por meio de [injeção de dependência](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-8.0) quando o tipo é configurado como serviço. Não é necessário aplicar explicitamente o atributo [`[FromServices]`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.mvc.fromservicesattribute) a um parâmetro. No código a seguir, ambas as ações retornam a hora:

C#

```
using Microsoft.AspNetCore.Mvc;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddSingleton<IDateTime, SystemDateTime>();

var app = builder.Build();

app.MapGet("/",   (               IDateTime dateTime) => dateTime.Now);
app.MapGet("/fs", ([FromServices] IDateTime dateTime) => dateTime.Now);
app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#optional-parameters)

### Parâmetros opcionais

Os parâmetros declarados em manipuladores de rota são tratados conforme a exigência:

- Se uma solicitação corresponder à rota, o manipulador de rota só será executado se todos os parâmetros necessários forem fornecidos na solicitação.
- O não fornecimento de todos os parâmetros necessários resulta em erro.

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/products", (int pageNumber) => $"Requesting page {pageNumber}");

app.Run();
```

|URI|resultado|
|---|---|
|`/products?pageNumber=3`|retornou 3|
|`/products`|`BadHttpRequestException`: o parâmetro obrigatório "int pageNumber" não foi fornecido a partir da cadeia de caracteres de consulta.|
|`/products/1`|Erro HTTP 404, nenhuma rota correspondente|

Para tornar `pageNumber` opcional, defina o tipo como opcional ou forneça um valor padrão:

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/products", (int? pageNumber) => $"Requesting page {pageNumber ?? 1}");

string ListProducts(int pageNumber = 1) => $"Requesting page {pageNumber}";

app.MapGet("/products2", ListProducts);

app.Run();
```

|URI|resultado|
|---|---|
|`/products?pageNumber=3`|retornou 3|
|`/products`|retornou 1|
|`/products2`|retornou 1|

O valor padrão e anulável anteriores aplica-se a todas as fontes:

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapPost("/products", (Product? product) => { });

app.Run();
```

O código anterior chama o método com um produto nulo se nenhum corpo da solicitação for enviado.

**OBSERVAÇÃO**: se dados inválidos forem fornecidos e o parâmetro for anulável, o manipulador de rota _**não**_ será executado.

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/products", (int? pageNumber) => $"Requesting page {pageNumber ?? 1}");

app.Run();
```

|URI|resultado|
|---|---|
|`/products?pageNumber=3`|retornou `3`|
|`/products`|retornou `1`|
|`/products?pageNumber=two`|`BadHttpRequestException`: falha ao associar o parâmetro `"Nullable<int> pageNumber"` de "dois".|
|`/products/two`|Erro HTTP 404, nenhuma rota correspondente|

Obtenha mais informações na seção [Falhas de Associação](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#bf).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#special-types)

### Tipos especiais

Os seguintes tipos são associados sem atributos explícitos:

- [HttpContext](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httpcontext): o contexto que contém todas as informações sobre a solicitação ou resposta HTTP atual:
    
    C#
    

- ```
    app.MapGet("/", (HttpContext context) => context.Response.WriteAsync("Hello World"));
    ```
    
- [HttpRequest](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httprequest) e [HttpResponse](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httpresponse): a solicitação HTTP e a resposta HTTP:
    
    C#
    
- ```
    app.MapGet("/", (HttpRequest request, HttpResponse response) =>
        response.WriteAsync($"Hello World {request.Query["name"]}"));
    ```
    
- [CancellationToken](https://learn.microsoft.com/pt-br/dotnet/api/system.threading.cancellationtoken): o token de cancelamento associado à solicitação HTTP atual:
    
    C#
    
- ```
    app.MapGet("/", async (CancellationToken cancellationToken) => 
        await MakeLongRunningRequestAsync(cancellationToken));
    ```
    
- [ClaimsPrincipal](https://learn.microsoft.com/pt-br/dotnet/api/system.security.claims.claimsprincipal): o usuário associado à solicitação, associado de [HttpContext.User](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httpcontext.user):
    
    C#
    

- ```
    app.MapGet("/", (ClaimsPrincipal user) => user.Identity.Name);
    ```
    

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#bind-the-request-body-as-a-stream-or-pipereader)

#### Associar o corpo da solicitação como um `Stream` ou `PipeReader`

O corpo da solicitação pode ser associado como um [`Stream`](https://learn.microsoft.com/pt-br/dotnet/api/system.io.stream) ou [`PipeReader`](https://learn.microsoft.com/pt-br/dotnet/api/system.io.pipelines.pipereader) para dar o suporte necessário a cenários em que o usuário precisa processar dados e:

- Armazenar os dados no armazenamento de blobs ou enfileirar os dados para um provedor de fila.
- Processar os dados armazenados com um processo de trabalho ou uma função de nuvem.

Por exemplo, os dados podem ser enfileirados no [Armazenamento de Filas do Azure](https://learn.microsoft.com/pt-br/azure/storage/queues/storage-queues-introduction) ou armazenados no [Armazenamento de Blobs do Azure](https://learn.microsoft.com/pt-br/azure/storage/blobs/storage-blobs-introduction).

O código a seguir implementa uma fila em segundo plano:

C#

```
using System.Text.Json;
using System.Threading.Channels;

namespace BackgroundQueueService;

class BackgroundQueue : BackgroundService
{
    private readonly Channel<ReadOnlyMemory<byte>> _queue;
    private readonly ILogger<BackgroundQueue> _logger;

    public BackgroundQueue(Channel<ReadOnlyMemory<byte>> queue,
                               ILogger<BackgroundQueue> logger)
    {
        _queue = queue;
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        await foreach (var dataStream in _queue.Reader.ReadAllAsync(stoppingToken))
        {
            try
            {
                var person = JsonSerializer.Deserialize<Person>(dataStream.Span)!;
                _logger.LogInformation($"{person.Name} is {person.Age} " +
                                       $"years and from {person.Country}");
            }
            catch (Exception ex)
            {
                _logger.LogError(ex.Message);
            }
        }
    }
}

class Person
{
    public string Name { get; set; } = String.Empty;
    public int Age { get; set; }
    public string Country { get; set; } = String.Empty;
}
```

O código a seguir associa o corpo da solicitação a um `Stream`:

C#

```
app.MapPost("/register", async (HttpRequest req, Stream body,
                                 Channel<ReadOnlyMemory<byte>> queue) =>
{
    if (req.ContentLength is not null && req.ContentLength > maxMessageSize)
    {
        return Results.BadRequest();
    }

    // We're not above the message size and we have a content length, or
    // we're a chunked request and we're going to read up to the maxMessageSize + 1. 
    // We add one to the message size so that we can detect when a chunked request body
    // is bigger than our configured max.
    var readSize = (int?)req.ContentLength ?? (maxMessageSize + 1);

    var buffer = new byte[readSize];

    // Read at least that many bytes from the body.
    var read = await body.ReadAtLeastAsync(buffer, readSize, throwOnEndOfStream: false);

    // We read more than the max, so this is a bad request.
    if (read > maxMessageSize)
    {
        return Results.BadRequest();
    }

    // Attempt to send the buffer to the background queue.
    if (queue.Writer.TryWrite(buffer.AsMemory(0..read)))
    {
        return Results.Accepted();
    }

    // We couldn't accept the message since we're overloaded.
    return Results.StatusCode(StatusCodes.Status429TooManyRequests);
});
```

O código a seguir mostra o arquivo `Program.cs` completo:

C#

```
using System.Threading.Channels;
using BackgroundQueueService;

var builder = WebApplication.CreateBuilder(args);
// The max memory to use for the upload endpoint on this instance.
var maxMemory = 500 * 1024 * 1024;

// The max size of a single message, staying below the default LOH size of 85K.
var maxMessageSize = 80 * 1024;

// The max size of the queue based on those restrictions
var maxQueueSize = maxMemory / maxMessageSize;

// Create a channel to send data to the background queue.
builder.Services.AddSingleton<Channel<ReadOnlyMemory<byte>>>((_) =>
                     Channel.CreateBounded<ReadOnlyMemory<byte>>(maxQueueSize));

// Create a background queue service.
builder.Services.AddHostedService<BackgroundQueue>();
var app = builder.Build();

// curl --request POST 'https://localhost:<port>/register' --header 'Content-Type: application/json' --data-raw '{ "Name":"Samson", "Age": 23, "Country":"Nigeria" }'
// curl --request POST "https://localhost:<port>/register" --header "Content-Type: application/json" --data-raw "{ \"Name\":\"Samson\", \"Age\": 23, \"Country\":\"Nigeria\" }"
app.MapPost("/register", async (HttpRequest req, Stream body,
                                 Channel<ReadOnlyMemory<byte>> queue) =>
{
    if (req.ContentLength is not null && req.ContentLength > maxMessageSize)
    {
        return Results.BadRequest();
    }

    // We're not above the message size and we have a content length, or
    // we're a chunked request and we're going to read up to the maxMessageSize + 1. 
    // We add one to the message size so that we can detect when a chunked request body
    // is bigger than our configured max.
    var readSize = (int?)req.ContentLength ?? (maxMessageSize + 1);

    var buffer = new byte[readSize];

    // Read at least that many bytes from the body.
    var read = await body.ReadAtLeastAsync(buffer, readSize, throwOnEndOfStream: false);

    // We read more than the max, so this is a bad request.
    if (read > maxMessageSize)
    {
        return Results.BadRequest();
    }

    // Attempt to send the buffer to the background queue.
    if (queue.Writer.TryWrite(buffer.AsMemory(0..read)))
    {
        return Results.Accepted();
    }

    // We couldn't accept the message since we're overloaded.
    return Results.StatusCode(StatusCodes.Status429TooManyRequests);
});

app.Run();
```

- Ao ler dados, o `Stream` é o mesmo objeto que `HttpRequest.Body`.
- O corpo da solicitação não é armazenado em buffer por padrão. Depois que o corpo for lido, não será possível retrocedê-lo. O fluxo não pode ser lido várias vezes.
- O `Stream` e `PipeReader` não são utilizáveis fora do manipulador de ação mínima, pois os buffers subjacentes serão descartados ou reutilizados.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#file-uploads-using-iformfile-and-iformfilecollection)

#### Uploads de arquivos usando IFormFile e IFormFileCollection

O código a seguir usa [IFormFile](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iformfile) e [IFormFileCollection](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iformfilecollection) para carregar o arquivo:

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello World!");

app.MapPost("/upload", async (IFormFile file) =>
{
    var tempFile = Path.GetTempFileName();
    app.Logger.LogInformation(tempFile);
    using var stream = File.OpenWrite(tempFile);
    await file.CopyToAsync(stream);
});

app.MapPost("/upload_many", async (IFormFileCollection myFiles) =>
{
    foreach (var file in myFiles)
    {
        var tempFile = Path.GetTempFileName();
        app.Logger.LogInformation(tempFile);
        using var stream = File.OpenWrite(tempFile);
        await file.CopyToAsync(stream);
    }
});

app.Run();
```

Há suporte a solicitações de upload de arquivos autenticados com o uso de um [cabeçalho de autorização](https://developer.mozilla.org/docs/Web/HTTP/Headers/Authorization), um [certificado de cliente](https://learn.microsoft.com/pt-br/aspnet/core/security/authentication/certauth) ou um cabeçalho de cookie.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#binding-to-forms-with-iformcollection-iformfile-and-iformfilecollection)

#### Associação a formulários com IFormCollection, IFormFile e IFormFileCollection

Há suporte para associação de parâmetros baseados em formulário usando [IFormCollection](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iformcollection), [IFormFile](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iformfile) e [IFormFileCollection](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iformfilecollection). Metadados do [OpenAPI](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/openapi/aspnetcore-openapi?view=aspnetcore-8.0) são inferidos para parâmetros de formulário para dar suporte à integração com a[interface do usuário do Swagger](https://learn.microsoft.com/pt-br/aspnet/core/tutorials/web-api-help-pages-using-swagger?view=aspnetcore-8.0).

O código a seguir carrega arquivos usando a associação inferida do tipo `IFormFile`:

C#

```
using Microsoft.AspNetCore.Antiforgery;
using Microsoft.AspNetCore.Http.HttpResults;

var builder = WebApplication.CreateBuilder();

builder.Services.AddAntiforgery();

var app = builder.Build();
app.UseAntiforgery();

string GetOrCreateFilePath(string fileName, string filesDirectory = "uploadFiles")
{
    var directoryPath = Path.Combine(app.Environment.ContentRootPath, filesDirectory);
    Directory.CreateDirectory(directoryPath);
    return Path.Combine(directoryPath, fileName);
}

async Task UploadFileWithName(IFormFile file, string fileSaveName)
{
    var filePath = GetOrCreateFilePath(fileSaveName);
    await using var fileStream = new FileStream(filePath, FileMode.Create);
    await file.CopyToAsync(fileStream);
}

app.MapGet("/", (HttpContext context, IAntiforgery antiforgery) =>
{
    var token = antiforgery.GetAndStoreTokens(context);
    var html = $"""
      <html>
        <body>
          <form action="/upload" method="POST" enctype="multipart/form-data">
            <input name="{token.FormFieldName}" type="hidden" value="{token.RequestToken}"/>
            <input type="file" name="file" placeholder="Upload an image..." accept=".jpg, 
                                                                            .jpeg, .png" />
            <input type="submit" />
          </form> 
        </body>
      </html>
    """;

    return Results.Content(html, "text/html");
});

app.MapPost("/upload", async Task<Results<Ok<string>,
   BadRequest<string>>> (IFormFile file, HttpContext context, IAntiforgery antiforgery) =>
{
    var fileSaveName = Guid.NewGuid().ToString("N") + Path.GetExtension(file.FileName);
    await UploadFileWithName(file, fileSaveName);
    return TypedResults.Ok("File uploaded successfully!");
});

app.Run();
```

_**Aviso:**_ Ao implementar formulários, o aplicativo _**deve evitar**_ [ataques de falsificação de solicitação entre sites (XSRF/CSRF)](https://learn.microsoft.com/pt-br/aspnet/core/security/anti-request-forgery?view=aspnetcore-8.0&preserve-view=true&view=aspnetcore-8.0#afwma). No código anterior, o serviço [IAntiforgery](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.antiforgery.iantiforgery) é usado para evitar ataques XSRF gerando e validando um token antifalsificação:

C#

```
using Microsoft.AspNetCore.Antiforgery;
using Microsoft.AspNetCore.Http.HttpResults;

var builder = WebApplication.CreateBuilder();

builder.Services.AddAntiforgery();

var app = builder.Build();
app.UseAntiforgery();

string GetOrCreateFilePath(string fileName, string filesDirectory = "uploadFiles")
{
    var directoryPath = Path.Combine(app.Environment.ContentRootPath, filesDirectory);
    Directory.CreateDirectory(directoryPath);
    return Path.Combine(directoryPath, fileName);
}

async Task UploadFileWithName(IFormFile file, string fileSaveName)
{
    var filePath = GetOrCreateFilePath(fileSaveName);
    await using var fileStream = new FileStream(filePath, FileMode.Create);
    await file.CopyToAsync(fileStream);
}

app.MapGet("/", (HttpContext context, IAntiforgery antiforgery) =>
{
    var token = antiforgery.GetAndStoreTokens(context);
    var html = $"""
      <html>
        <body>
          <form action="/upload" method="POST" enctype="multipart/form-data">
            <input name="{token.FormFieldName}" type="hidden" value="{token.RequestToken}"/>
            <input type="file" name="file" placeholder="Upload an image..." accept=".jpg, 
                                                                            .jpeg, .png" />
            <input type="submit" />
          </form> 
        </body>
      </html>
    """;

    return Results.Content(html, "text/html");
});

app.MapPost("/upload", async Task<Results<Ok<string>,
   BadRequest<string>>> (IFormFile file, HttpContext context, IAntiforgery antiforgery) =>
{
    var fileSaveName = Guid.NewGuid().ToString("N") + Path.GetExtension(file.FileName);
    await UploadFileWithName(file, fileSaveName);
    return TypedResults.Ok("File uploaded successfully!");
});

app.Run();
```

Para obter mais informações sobre ataques XSRF, confira [Antifalsificação com APIs mínimas](https://learn.microsoft.com/pt-br/aspnet/core/security/anti-request-forgery?view=aspnetcore-8.0&preserve-view=true&view=aspnetcore-8.0#afwma)

Para obter mais informações, consulte [Vinculação de formulário em APIs mínimas](https://andrewlock.net/exploring-the-dotnet-8-preview-form-binding-in-minimal-apis/);

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#bind-to-collections-and-complex-types-from-forms)

### Associar a coleções e tipos complexos de formulários

A associação tem suporte para:

- Coleções, por exemplo, [Lista](https://learn.microsoft.com/pt-br/dotnet/api/system.collections.generic.list-1) e [Dicionário](https://learn.microsoft.com/pt-br/dotnet/api/system.collections.generic.dictionary-2)
- Tipos complexos, por exemplo, `Todo` ou `Project`

O código a seguir mostra:

- Um ponto de extremidade mínimo que associa uma entrada de formulário de várias partes a um objeto complexo.
- Como usar os serviços antifalsificação para dar suporte à geração e à validação de tokens antifalsificação.

C#

```
using Microsoft.AspNetCore.Antiforgery;
using Microsoft.AspNetCore.Http.HttpResults;
using Microsoft.AspNetCore.Mvc;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAntiforgery();

var app = builder.Build();

app.UseAntiforgery();

app.MapGet("/", (HttpContext context, IAntiforgery antiforgery) =>
{
    var token = antiforgery.GetAndStoreTokens(context);
    var html = $"""
        <html><body>
           <form action="/todo" method="POST" enctype="multipart/form-data">
               <input name="{token.FormFieldName}" 
                                type="hidden" value="{token.RequestToken}" />
               <input type="text" name="name" />
               <input type="date" name="dueDate" />
               <input type="checkbox" name="isCompleted" value="true" />
               <input type="submit" />
               <input name="isCompleted" type="hidden" value="false" /> 
           </form>
        </body></html>
    """;
    return Results.Content(html, "text/html");
});

app.MapPost("/todo", async Task<Results<Ok<Todo>, BadRequest<string>>> 
               ([FromForm] Todo todo, HttpContext context, IAntiforgery antiforgery) =>
{
    try
    {
        await antiforgery.ValidateRequestAsync(context);
        return TypedResults.Ok(todo);
    }
    catch (AntiforgeryValidationException e)
    {
        return TypedResults.BadRequest("Invalid antiforgery token");
    }
});

app.Run();

class Todo
{
    public string Name { get; set; } = string.Empty;
    public bool IsCompleted { get; set; } = false;
    public DateTime DueDate { get; set; } = DateTime.Now.Add(TimeSpan.FromDays(1));
}
```

No código anterior:

- O parâmetro de destino _**deve**_ ser anotado com o atributo [`[FromForm]`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.mvc.fromformattribute) para remover a ambiguidade com parâmetros que devem ser lidos do corpo JSON.
- _**Não há**_ suporte para a vinculação de tipos complexos ou de coleção para APIs mínimas compiladas com o Gerador de Representante de Solicitação.
- A marcação mostra uma entrada adicional oculta com um nome `isCompleted` e um valor `false`. Se a caixa de seleção `isCompleted` estiver marcada quando o formulário for enviado, ambos os valores `true` e `false` serão enviados como valores. Se a caixa de seleção estiver desmarcada, somente o valor de entrada oculto `false` será enviado. O processo de associação de modelo do ASP.NET Core lê apenas o primeiro valor ao associar a um valor `bool`, o que resulta em `true` para caixas de seleção marcadas e `false` para caixas de seleção desmarcadas.

Um exemplo dos dados de formulário enviados ao ponto de extremidade anterior tem a seguinte aparência:

txt

```
__RequestVerificationToken: CfDJ8Bveip67DklJm5vI2PF2VOUZ594RC8kcGWpTnVV17zCLZi1yrs-CSz426ZRRrQnEJ0gybB0AD7hTU-0EGJXDU-OaJaktgAtWLIaaEWMOWCkoxYYm-9U9eLV7INSUrQ6yBHqdMEE_aJpD4AI72gYiCqc
name: Walk the dog
dueDate: 2024-04-06
isCompleted: true
isCompleted: false
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#bind-arrays-and-string-values-from-headers-and-query-strings)

### Associe matrizes e valores de cadeia de caracteres a partir de cabeçalhos e cadeias de caracteres de consulta

O código a seguir demonstra a associação de cadeias de caracteres de consulta a uma matriz de tipos primitivos, matrizes de cadeia de caracteres e [StringValues](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.extensions.primitives.stringvalues):

C#

```
// Bind query string values to a primitive type array.
// GET  /tags?q=1&q=2&q=3
app.MapGet("/tags", (int[] q) =>
                      $"tag1: {q[0]} , tag2: {q[1]}, tag3: {q[2]}");

// Bind to a string array.
// GET /tags2?names=john&names=jack&names=jane
app.MapGet("/tags2", (string[] names) =>
            $"tag1: {names[0]} , tag2: {names[1]}, tag3: {names[2]}");

// Bind to StringValues.
// GET /tags3?names=john&names=jack&names=jane
app.MapGet("/tags3", (StringValues names) =>
            $"tag1: {names[0]} , tag2: {names[1]}, tag3: {names[2]}");
```

Há suporte à associação de cadeias de caracteres de consulta ou valores de cabeçalho a uma matriz de tipos complexos quando o tipo tem `TryParse` implementado. O código a seguir é associado a uma matriz de cadeia de caracteres e retorna todos os itens com as tags especificadas:

C#

```
// GET /todoitems/tags?tags=home&tags=work
app.MapGet("/todoitems/tags", async (Tag[] tags, TodoDb db) =>
{
    return await db.Todos
        .Where(t => tags.Select(i => i.Name).Contains(t.Tag.Name))
        .ToListAsync();
});
```

O código a seguir mostra o modelo e a implementação `TryParse` necessária:

C#

```
public class Todo
{
    public int Id { get; set; }
    public string? Name { get; set; }
    public bool IsComplete { get; set; }

    // This is an owned entity. 
    public Tag Tag { get; set; } = new();
}

[Owned]
public class Tag
{
    public string? Name { get; set; } = "n/a";

    public static bool TryParse(string? name, out Tag tag)
    {
        if (name is null)
        {
            tag = default!;
            return false;
        }

        tag = new Tag { Name = name };
        return true;
    }
}
```

O código a seguir é associado a uma matriz `int`:

C#

```
// GET /todoitems/query-string-ids?ids=1&ids=3
app.MapGet("/todoitems/query-string-ids", async (int[] ids, TodoDb db) =>
{
    return await db.Todos
        .Where(t => ids.Contains(t.Id))
        .ToListAsync();
});
```

Para testar o código anterior, adicione o seguinte ponto de extremidade para preencher o banco de dados com itens `Todo`:

C#

```
// POST /todoitems/batch
app.MapPost("/todoitems/batch", async (Todo[] todos, TodoDb db) =>
{
    await db.Todos.AddRangeAsync(todos);
    await db.SaveChangesAsync();

    return Results.Ok(todos);
});
```

Use uma ferramenta como o [`HttpRepl`](https://learn.microsoft.com/pt-br/aspnet/core/web-api/http-repl/?view=aspnetcore-8.0) para passar os seguintes dados para o ponto de extremidade anterior:

C#

```
[
    {
        "id": 1,
        "name": "Have Breakfast",
        "isComplete": true,
        "tag": {
            "name": "home"
        }
    },
    {
        "id": 2,
        "name": "Have Lunch",
        "isComplete": true,
        "tag": {
            "name": "work"
        }
    },
    {
        "id": 3,
        "name": "Have Supper",
        "isComplete": true,
        "tag": {
            "name": "home"
        }
    },
    {
        "id": 4,
        "name": "Have Snacks",
        "isComplete": true,
        "tag": {
            "name": "N/A"
        }
    }
]
```

O código a seguir associa-se à chave `X-Todo-Id` de cabeçalho e retorna os itens `Todo` com valores correspondentes `Id`:

C#

```
// GET /todoitems/header-ids
// The keys of the headers should all be X-Todo-Id with different values
app.MapGet("/todoitems/header-ids", async ([FromHeader(Name = "X-Todo-Id")] int[] ids, TodoDb db) =>
{
    return await db.Todos
        .Where(t => ids.Contains(t.Id))
        .ToListAsync();
});
```

Observação

Ao associar um `string[]` de uma cadeia de caracteres de consulta, a ausência de qualquer valor de cadeia de caracteres de consulta correspondente resultará em uma matriz vazia em vez de um valor nulo.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#parameter-binding-for-argument-lists-with-asparameters)

### Associação de parâmetros para listas de argumentos com [AsParameters]

[AsParametersAttribute](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.asparametersattribute) permite a associação de parâmetros simples a tipos e não a model binding complexa ou recursiva.

Considere o código a seguir:

C#

```
using Microsoft.EntityFrameworkCore;
using TodoApi.Models;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddDatabaseDeveloperPageExceptionFilter();
builder.Services.AddDbContext<TodoDb>(opt => opt.UseInMemoryDatabase("TodoList"));
var app = builder.Build();

app.MapGet("/todoitems", async (TodoDb db) =>
    await db.Todos.Select(x => new TodoItemDTO(x)).ToListAsync());

app.MapGet("/todoitems/{id}",
                             async (int Id, TodoDb Db) =>
    await Db.Todos.FindAsync(Id)
        is Todo todo
            ? Results.Ok(new TodoItemDTO(todo))
            : Results.NotFound());
// Remaining code removed for brevity.
```

Considere o ponto de extremidade `GET` a seguir:

C#

```
app.MapGet("/todoitems/{id}",
                             async (int Id, TodoDb Db) =>
    await Db.Todos.FindAsync(Id)
        is Todo todo
            ? Results.Ok(new TodoItemDTO(todo))
            : Results.NotFound());
```

O seguinte `struct` pode ser usado para substituir os parâmetros realçados anteriores:

C#

```
struct TodoItemRequest
{
    public int Id { get; set; }
    public TodoDb Db { get; set; }
}
```

O ponto de extremidade refatorado `GET` usa o anterior `struct` com o atributo [AsParameters](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.asparametersattribute?view=aspnetcore-7.0&preserve-view=true):

C#

```
app.MapGet("/ap/todoitems/{id}",
                                async ([AsParameters] TodoItemRequest request) =>
    await request.Db.Todos.FindAsync(request.Id)
        is Todo todo
            ? Results.Ok(new TodoItemDTO(todo))
            : Results.NotFound());
```

O código a seguir mostra pontos de extremidade adicionais no aplicativo:

C#

```
app.MapPost("/todoitems", async (TodoItemDTO Dto, TodoDb Db) =>
{
    var todoItem = new Todo
    {
        IsComplete = Dto.IsComplete,
        Name = Dto.Name
    };

    Db.Todos.Add(todoItem);
    await Db.SaveChangesAsync();

    return Results.Created($"/todoitems/{todoItem.Id}", new TodoItemDTO(todoItem));
});

app.MapPut("/todoitems/{id}", async (int Id, TodoItemDTO Dto, TodoDb Db) =>
{
    var todo = await Db.Todos.FindAsync(Id);

    if (todo is null) return Results.NotFound();

    todo.Name = Dto.Name;
    todo.IsComplete = Dto.IsComplete;

    await Db.SaveChangesAsync();

    return Results.NoContent();
});

app.MapDelete("/todoitems/{id}", async (int Id, TodoDb Db) =>
{
    if (await Db.Todos.FindAsync(Id) is Todo todo)
    {
        Db.Todos.Remove(todo);
        await Db.SaveChangesAsync();
        return Results.Ok(new TodoItemDTO(todo));
    }

    return Results.NotFound();
});
```

As seguintes classes são usadas para refatorar as listas de parâmetros:

C#

```
class CreateTodoItemRequest
{
    public TodoItemDTO Dto { get; set; } = default!;
    public TodoDb Db { get; set; } = default!;
}

class EditTodoItemRequest
{
    public int Id { get; set; }
    public TodoItemDTO Dto { get; set; } = default!;
    public TodoDb Db { get; set; } = default!;
}
```

O código a seguir mostra os pontos de extremidade refatorados usando `AsParameters`, os `struct` e classes anteriores:

C#

```
app.MapPost("/ap/todoitems", async ([AsParameters] CreateTodoItemRequest request) =>
{
    var todoItem = new Todo
    {
        IsComplete = request.Dto.IsComplete,
        Name = request.Dto.Name
    };

    request.Db.Todos.Add(todoItem);
    await request.Db.SaveChangesAsync();

    return Results.Created($"/todoitems/{todoItem.Id}", new TodoItemDTO(todoItem));
});

app.MapPut("/ap/todoitems/{id}", async ([AsParameters] EditTodoItemRequest request) =>
{
    var todo = await request.Db.Todos.FindAsync(request.Id);

    if (todo is null) return Results.NotFound();

    todo.Name = request.Dto.Name;
    todo.IsComplete = request.Dto.IsComplete;

    await request.Db.SaveChangesAsync();

    return Results.NoContent();
});

app.MapDelete("/ap/todoitems/{id}", async ([AsParameters] TodoItemRequest request) =>
{
    if (await request.Db.Todos.FindAsync(request.Id) is Todo todo)
    {
        request.Db.Todos.Remove(todo);
        await request.Db.SaveChangesAsync();
        return Results.Ok(new TodoItemDTO(todo));
    }

    return Results.NotFound();
});
```

Os seguintes tipos [`record`](https://learn.microsoft.com/pt-br/dotnet/csharp/language-reference/builtin-types/record) podem ser usados para substituir os parâmetros anteriores:

C#

```
record TodoItemRequest(int Id, TodoDb Db);
record CreateTodoItemRequest(TodoItemDTO Dto, TodoDb Db);
record EditTodoItemRequest(int Id, TodoItemDTO Dto, TodoDb Db);
```

Usar um `struct` com `AsParameters` pode ser mais eficaz do que usar um tipo `record`.

O [código de exemplo completo](https://github.com/dotnet/AspNetCore.Docs.Samples/tree/main/fundamentals/minimal-apis/samples/arg-lists) no repositório [AspNetCore.Docs.Samples](https://github.com/dotnet/AspNetCore.Docs.Samples).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#custom-binding)

### Associação personalizado

Há duas maneiras de personalizar a associação de parâmetros:

1. Para as origens da associação de rota, de consulta e de cabeçalho, associe tipos personalizados adicionando um método estático `TryParse` para o tipo.
2. Controlar o processo de associação implementando um método `BindAsync` em um tipo.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#tryparse)

#### TryParse

`TryParse` tem duas APIs:

C#

```
public static bool TryParse(string value, out T result);
public static bool TryParse(string value, IFormatProvider provider, out T result);
```

O código a seguir exibe `Point: 12.3, 10.1` com o URI `/map?Point=12.3,10.1`:

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// GET /map?Point=12.3,10.1
app.MapGet("/map", (Point point) => $"Point: {point.X}, {point.Y}");

app.Run();

public class Point
{
    public double X { get; set; }
    public double Y { get; set; }

    public static bool TryParse(string? value, IFormatProvider? provider,
                                out Point? point)
    {
        // Format is "(12.3,10.1)"
        var trimmedValue = value?.TrimStart('(').TrimEnd(')');
        var segments = trimmedValue?.Split(',',
                StringSplitOptions.RemoveEmptyEntries | StringSplitOptions.TrimEntries);
        if (segments?.Length == 2
            && double.TryParse(segments[0], out var x)
            && double.TryParse(segments[1], out var y))
        {
            point = new Point { X = x, Y = y };
            return true;
        }

        point = null;
        return false;
    }
}
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#bindasync)

#### BindAsync

`BindAsync` tem as seguintes APIs:

C#

```
public static ValueTask<T?> BindAsync(HttpContext context, ParameterInfo parameter);
public static ValueTask<T?> BindAsync(HttpContext context);
```

O código a seguir exibe `SortBy:xyz, SortDirection:Desc, CurrentPage:99` com o URI `/products?SortBy=xyz&SortDir=Desc&Page=99`:

C#

```
using System.Reflection;

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// GET /products?SortBy=xyz&SortDir=Desc&Page=99
app.MapGet("/products", (PagingData pageData) => $"SortBy:{pageData.SortBy}, " +
       $"SortDirection:{pageData.SortDirection}, CurrentPage:{pageData.CurrentPage}");

app.Run();

public class PagingData
{
    public string? SortBy { get; init; }
    public SortDirection SortDirection { get; init; }
    public int CurrentPage { get; init; } = 1;

    public static ValueTask<PagingData?> BindAsync(HttpContext context,
                                                   ParameterInfo parameter)
    {
        const string sortByKey = "sortBy";
        const string sortDirectionKey = "sortDir";
        const string currentPageKey = "page";

        Enum.TryParse<SortDirection>(context.Request.Query[sortDirectionKey],
                                     ignoreCase: true, out var sortDirection);
        int.TryParse(context.Request.Query[currentPageKey], out var page);
        page = page == 0 ? 1 : page;

        var result = new PagingData
        {
            SortBy = context.Request.Query[sortByKey],
            SortDirection = sortDirection,
            CurrentPage = page
        };

        return ValueTask.FromResult<PagingData?>(result);
    }
}

public enum SortDirection
{
    Default,
    Asc,
    Desc
}
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#binding-failures)

### Falhas de associação

Quando a associação falha, a estrutura registra uma mensagem de depuração e retorna vários códigos de status ao cliente, dependendo do modo de falha.

|Modo de falha|Tipo de parâmetro anulável|Origem da associação|Código de status|
|---|---|---|---|
|`{ParameterType}.TryParse` retorna `false`|sim|rota/consulta/cabeçalho|400|
|`{ParameterType}.BindAsync` retorna `null`|sim|personalizado|400|
|`{ParameterType}.BindAsync` gera|não importa|custom|500|
|Falha na desserialização do corpo JSON|não importa|body|400|
|Tipo de conteúdo incorreto (não `application/json`)|não importa|body|415|

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#binding-precedence)

### Precedência de associação

As regras que determinam uma origem de associação de um parâmetro:

1. Atributo explícito definido no parâmetro (De atributos*) na seguinte ordem:
    1. Valores da rota: [`[FromRoute]`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.mvc.fromrouteattribute)
    2. Cadeia de caracteres de consulta: [`[FromQuery]`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.mvc.fromqueryattribute)
    3. Cabeçalho: [`[FromHeader]`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.mvc.fromheaderattribute)
    4. Corpo: [`[FromBody]`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.mvc.frombodyattribute)
    5. Formulário: [`[FromForm]`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.mvc.fromformattribute)
    6. Serviço: [`[FromServices]`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.mvc.fromservicesattribute)
    7. Valores de parâmetro: [`[AsParameters]`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.asparametersattribute)
2. Tipos especiais
    1. [`HttpContext`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httpcontext)
    2. [`HttpRequest`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httprequest) ([`HttpContext.Request`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httpcontext.request#microsoft-aspnetcore-http-httpcontext-request))
    3. [`HttpResponse`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httpresponse) ([`HttpContext.Response`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httpcontext.response#microsoft-aspnetcore-http-httpcontext-response))
    4. [`ClaimsPrincipal`](https://learn.microsoft.com/pt-br/dotnet/api/system.security.claims.claimsprincipal) ([`HttpContext.User`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httpcontext.user#microsoft-aspnetcore-http-httpcontext-user))
    5. [`CancellationToken`](https://learn.microsoft.com/pt-br/dotnet/api/system.threading.cancellationtoken) ([`HttpContext.RequestAborted`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httpcontext.requestaborted#microsoft-aspnetcore-http-httpcontext-requestaborted))
    6. [`IFormCollection`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iformcollection) ([`HttpContext.Request.Form`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iformcollection))
    7. [`IFormFileCollection`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iformfilecollection) ([`HttpContext.Request.Form.Files`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iformcollection.files#microsoft-aspnetcore-http-iformcollection-files))
    8. [`IFormFile`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iformfile) ([`HttpContext.Request.Form.Files[paramName]`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iformfilecollection.item#microsoft-aspnetcore-http-iformfilecollection-item(system-string)))
    9. [`Stream`](https://learn.microsoft.com/pt-br/dotnet/api/system.io.stream) ([`HttpContext.Request.Body`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httprequest.body#microsoft-aspnetcore-http-httprequest-body))
    10. [`PipeReader`](https://learn.microsoft.com/pt-br/dotnet/api/system.io.pipelines.pipereader) ([`HttpContext.Request.BodyReader`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httprequest.bodyreader#microsoft-aspnetcore-http-httprequest-bodyreader))
3. O tipo de parâmetro tem um método estático [`BindAsync`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.ibindablefromhttpcontext-1.bindasync) válido.
4. O tipo de parâmetro é uma cadeia de caracteres ou tem um método estático [`TryParse`](https://learn.microsoft.com/pt-br/dotnet/api/system.iparsable-1.tryparse) válido.
    1. Se o nome do parâmetro existir no modelo de rota, por exemplo, `app.Map("/todo/{id}", (int id) => {});`, ele será vinculado a partir da rota.
    2. Associação a partir da cadeia de caracteres de consulta.
5. Se o tipo de parâmetro for um serviço fornecido pela injeção de dependência, ele usará esse serviço como origem.
6. O parâmetro se origina do corpo.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#configure-json-deserialization-options-for-body-binding)

### Configurar opções de desserialização JSON para associação de corpo

A origem da associação do corpo usa [System.Text.Json](https://learn.microsoft.com/pt-br/dotnet/api/system.text.json) para desserialização. _**Não**_ é possível alterar esse padrão, mas as opções de serialização e desserialização JSON podem ser configuradas.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#configure-json-deserialization-options-globally)

#### Configurar opções de desserialização JSON globalmente

As opções que se aplicam globalmente a um aplicativo podem ser configuradas por meio da invocação de [ConfigureHttpJsonOptions](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.extensions.dependencyinjection.httpjsonserviceextensions.configurehttpjsonoptions). O exemplo a seguir inclui campos públicos e formata a saída JSON.

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

Como o código de exemplo configura a serialização e a desserialização, ele pode ler `NameField` e incluir `NameField` na saída JSON.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#configure-json-deserialization-options-for-an-endpoint)

#### Configurar opções de desserialização JSON para um ponto de extremidade

[ReadFromJsonAsync](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httprequestjsonextensions.readfromjsonasync) tem sobrecargas que aceitam um objeto [JsonSerializerOptions](https://learn.microsoft.com/pt-br/dotnet/api/system.text.json.jsonserializeroptions). O exemplo a seguir inclui campos públicos e formata a saída JSON.

C#

```
using System.Text.Json;

var app = WebApplication.Create();

var options = new JsonSerializerOptions(JsonSerializerDefaults.Web) { 
    IncludeFields = true, 
    WriteIndented = true
};

app.MapPost("/", async (HttpContext context) => {
    if (context.Request.HasJsonContentType()) {
        var todo = await context.Request.ReadFromJsonAsync<Todo>(options);
        if (todo is not null) {
            todo.Name = todo.NameField;
        }
        return Results.Ok(todo);
    }
    else {
        return Results.BadRequest();
    }
});

app.Run();

class Todo
{
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
//    "isComplete":false
// }
```

Como o código anterior aplica as opções personalizadas somente à desserialização, o JSON de saída exclui `NameField`.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/parameter-binding?view=aspnetcore-8.0#read-the-request-body)

### Leia o corpo da solicitação.

Leia o corpo da solicitação diretamente usando um parâmetro [HttpContext](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httpcontext) ou [HttpRequest](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httprequest):

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapPost("/uploadstream", async (IConfiguration config, HttpRequest request) =>
{
    var filePath = Path.Combine(config["StoredFilesPath"], Path.GetRandomFileName());

    await using var writeStream = File.Create(filePath);
    await request.BodyReader.CopyToAsync(writeStream);
});

app.Run();
```

O código anterior:

- Acessa o corpo da solicitação usando [HttpRequest.BodyReader](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httprequest.bodyreader#microsoft-aspnetcore-http-httprequest-bodyreader).
- Copia o corpo da solicitação em um arquivo local.