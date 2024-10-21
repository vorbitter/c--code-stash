## Exceções

Em um aplicativo de API mínima, há dois mecanismos centralizados internos diferentes para lidar com exceções sem tratamento:

- [Middleware da Página de exceção do desenvolvedor](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/handle-errors?view=aspnetcore-8.0#developer-exception-page) (para uso **apenas no ambiente de desenvolvimento**).
- [Middleware do manipulador de exceção](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/handle-errors?view=aspnetcore-8.0#exception-handler)

Esta seção se refere ao aplicativo de amostra a seguir para demonstrar formas de tratar exceções em uma API Mínima. Ele gera uma exceção quando o ponto de extremidade `/exception` é solicitado:

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/exception", () => 
{
    throw new InvalidOperationException("Sample Exception");
});

app.MapGet("/", () => "Test by calling /exception");

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/handle-errors?view=aspnetcore-8.0#developer-exception-page)

### Página de exceção do desenvolvedor

A _Página de exceção do desenvolvedor_ exibe informações detalhadas sobre as exceções de solicitação não tratadas. Ele usa [DeveloperExceptionPageMiddleware](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.diagnostics.developerexceptionpagemiddleware) para capturar exceções síncronas e assíncronas do pipeline HTTP e para gerar respostas de erro. A página de exceção do desenvolvedor é executada no início do pipeline do middleware, para que ele possa capturar exceções sem tratamento geradas no middleware a seguir.

Os aplicativos ASP.NET Core habilitam a página de exceção do desenvolvedor por padrão quando ambos:

- Em execução no [Ambiente de desenvolvimento](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/environments?view=aspnetcore-8.0).
- O aplicativo foi criado com os modelos atuais, ou seja, usando [WebApplication.CreateBuilder](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.webapplication.createbuilder).

Os aplicativos criados usando modelos anteriores, ou seja, usando [WebHost.CreateDefaultBuilder](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.webhost.createdefaultbuilder), podem ativar a página de exceção do desenvolvedor chamando [`app.UseDeveloperExceptionPage`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.developerexceptionpageextensions.usedeveloperexceptionpage#microsoft-aspnetcore-builder-developerexceptionpageextensions-usedeveloperexceptionpage(microsoft-aspnetcore-builder-iapplicationbuilder)).

Aviso

Não habilite a Página de exceção do desenvolvedor **a menos que o aplicativo esteja em execução no Ambiente de desenvolvimento**. Não compartilhe informações de exceção detalhadas publicamente quando o aplicativo é executado em produção. Para obter mais informações sobre como configurar ambientes, confira [Use vários ambientes no ASP.NET Core](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/environments?view=aspnetcore-8.0).

A Página de Exceção do Desenvolvedor pode incluir as seguintes informações sobre a exceção e a solicitação:

- Rastreamento de pilha
- Parâmetros de cadeia de caracteres de consulta, se houver
- Cookies, se houver algum
- Cabeçalhos
- Metadados de ponto de extremidade, se houver

Não existe garantia de que a Página de Exceção do Desenvolvedor fornecerá alguma informação. Use [Registro em log](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/logging/?view=aspnetcore-8.0) para obter informações de erro completas.

A imagem a seguir mostra uma página de exceção do desenvolvedor de exemplo com animação para mostrar as guias e as informações exibidas:

![Página de exceção do desenvolvedor animada para mostrar cada guia selecionada.](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/error-handling/_static/aspnetcore-developer-page-improvements.gif?view=aspnetcore-8.0)

Em resposta a uma solicitação com um cabeçalho `Accept: text/plain`, a Página de Exceção do Desenvolvedor retorna texto sem formatação em vez de HTML. Por exemplo:

text

```
Status: 500 Internal Server Error
Time: 9.39 msSize: 480 bytes
FormattedRawHeadersRequest
Body
text/plain; charset=utf-8, 480 bytes
System.InvalidOperationException: Sample Exception
   at WebApplicationMinimal.Program.<>c.<Main>b__0_0() in C:\Source\WebApplicationMinimal\Program.cs:line 12
   at lambda_method1(Closure, Object, HttpContext)
   at Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddlewareImpl.Invoke(HttpContext context)

HEADERS
=======
Accept: text/plain
Host: localhost:7267
traceparent: 00-0eab195ea19d07b90a46cd7d6bf2f
```

Para ver a página de exceção do desenvolvedor:

- Execute o aplicativo de exemplo no [Ambiente de desenvolvimento](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/environments?view=aspnetcore-8.0).
- Vá para o ponto de extremidade `/exception`.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/handle-errors?view=aspnetcore-8.0#exception-handler)

### Manipulador de exceção

Em ambientes de não desenvolvimento, use o [Middleware do manipulador de exceções](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/error-handling?view=aspnetcore-8.0#exception-handler-page) para produzir um conteúdo de erro. Para configurar o `Exception Handler Middleware`, chame [UseExceptionHandler](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.exceptionhandlerextensions.useexceptionhandler).

Por exemplo, o código a seguir altera o aplicativo para responder com um conteúdo compatível com [RFC 7807](https://tools.ietf.org/html/rfc7807) para o cliente. Para obter mais informações, consulte a seção [Detalhes do Problema](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/handle-errors?view=aspnetcore-8.0#problem-details) mais adiante neste artigo.

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseExceptionHandler(exceptionHandlerApp 
    => exceptionHandlerApp.Run(async context 
        => await Results.Problem()
                     .ExecuteAsync(context)));

app.MapGet("/exception", () => 
{
    throw new InvalidOperationException("Sample Exception");
});

app.MapGet("/", () => "Test by calling /exception");

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/handle-errors?view=aspnetcore-8.0#client-and-server-error-responses)

## Respostas de erro de cliente e servidor

Cogite o aplicativo de API mínima a seguir.

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/users/{id:int}", (int id) 
    => id <= 0 ? Results.BadRequest() : Results.Ok(new User(id)));

app.MapGet("/", () => "Test by calling /users/{id:int}");

app.Run();

public record User(int Id);
```

O ponto de extremidade `/users` produz `200 OK` com uma representação `json` do `User` quando `id` for maior que `0`, caso contrário, produzirá um código de status `400 BAD REQUEST` sem um corpo de resposta. Para obter mais informações sobre como criar uma resposta, confira [Criar respostas em aplicativos de API mínimo](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses).

O [`Status Code Pages middleware`](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/error-handling?view=aspnetcore-8.0#sestatuscodepages) pode ser configurado para produzir um conteúdo de corpo comum, **quando vazio**, para todas as respostas de cliente HTTP (`400`-`499`) ou servidor (`500` -`599`). O middleware é configurado chamando o método de extensão [UseStatusCodePages](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.statuscodepagesextensions.usestatuscodepages).

O exemplo a seguir altera o aplicativo para responder com uma carga compatível com [RFC 7807](https://tools.ietf.org/html/rfc7807) para o cliente para todas as respostas de cliente e servidor, incluindo erros de roteamento (por exemplo, `404 NOT FOUND`). Para obter mais informações, confira a seção [Detalhes do problema](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/handle-errors?view=aspnetcore-8.0#problem-details).

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseStatusCodePages(async statusCodeContext 
    => await Results.Problem(statusCode: statusCodeContext.HttpContext.Response.StatusCode)
                 .ExecuteAsync(statusCodeContext.HttpContext));

app.MapGet("/users/{id:int}", (int id) 
    => id <= 0 ? Results.BadRequest() : Results.Ok(new User(id)) );

app.MapGet("/", () => "Test by calling /users/{id:int}");

app.Run();

public record User(int Id);
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/handle-errors?view=aspnetcore-8.0#problem-details)

## Detalhes do problema

Os [Detalhes do problema](https://www.rfc-editor.org/rfc/rfc7807.html) não são o único formato de resposta a descrever um erro de API HTTP; no entanto, eles geralmente são usados para relatar erros para APIs HTTP.

O serviço de detalhes do problema implementa a interface [IProblemDetailsService](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iproblemdetailsservice), que dá suporte à criação de detalhes do problema no ASP.NET Core. O método de extensão [AddProblemDetails(IServiceCollection)](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.extensions.dependencyinjection.problemdetailsservicecollectionextensions.addproblemdetails#microsoft-extensions-dependencyinjection-problemdetailsservicecollectionextensions-addproblemdetails(microsoft-extensions-dependencyinjection-iservicecollection)) em [IServiceCollection](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.extensions.dependencyinjection.iservicecollection) regista a implementação `IProblemDetailsService` padrão.

Em aplicativos ASP.NET Core, o middleware abaixo gera respostas HTTP de detalhes do problema quando `AddProblemDetails` é chamado, exceto quando o [cabeçalho HTTP da solicitação `Accept`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept) não inclui um dos tipos de conteúdo com suporte pelo [IProblemDetailsWriter](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iproblemdetailswriter) registrado (padrão: `application/json`):

- [ExceptionHandlerMiddleware](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.diagnostics.exceptionhandlermiddleware): gera uma resposta de detalhes do problema quando um manipulador personalizado não é definido.
- [StatusCodePagesMiddleware](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.diagnostics.statuscodepagesmiddleware): gera uma resposta de detalhes do problema por padrão.
- [DeveloperExceptionPageMiddleware](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.diagnostics.developerexceptionpagemiddleware): gera uma resposta de detalhes do problema no desenvolvimento quando o cabeçalho HTTP da solicitação `Accept` não inclui `text/html`.

Aplicativos de API mínima podem ser configurados para gerar uma resposta de detalhes do problema para todas as respostas de erro de cliente e servidor HTTP que _**ainda não tenham conteúdo de corpo**_ usando o método de extensão `AddProblemDetails`.

O código a seguir configura o aplicativo para gerar detalhes do problema:

C#

```
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddProblemDetails();

var app = builder.Build();

app.UseExceptionHandler();
app.UseStatusCodePages();

app.MapGet("/users/{id:int}", (int id) 
    => id <= 0 ? Results.BadRequest() : Results.Ok(new User(id)));

app.MapGet("/", () => "Test by calling /users/{id:int}");

app.Run();

public record User(int Id);
```

Para obter mais informações sobre como usar `AddProblemDetails`, confira [Detalhes do problema](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/error-handling?view=aspnetcore-7.0&preserve-view=true#pds7)

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/handle-errors?view=aspnetcore-8.0#iproblemdetailsservice-fallback)

## Fallback de IProblemDetailsService

No código a seguir, `httpContext.Response.WriteAsync("Fallback: An error occurred.")` retornará um erro se a implementação [IProblemDetailsService](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iproblemdetailsservice) não conseguir gerar um [ProblemDetails](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.mvc.problemdetails):

C#

```
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddProblemDetails();

var app = builder.Build();

app.UseExceptionHandler(exceptionHandlerApp =>
{
    exceptionHandlerApp.Run(async httpContext =>
    {
        var pds = httpContext.RequestServices.GetService<IProblemDetailsService>();
        if (pds == null
            || !await pds.TryWriteAsync(new() { HttpContext = httpContext }))
        {
            // Fallback behavior
            await httpContext.Response.WriteAsync("Fallback: An error occurred.");
        }
    });
});

app.MapGet("/exception", () =>
{
    throw new InvalidOperationException("Sample Exception");
});

app.MapGet("/", () => "Test by calling /exception");

app.Run();
```

O código anterior:

- Grava uma mensagem de erro com o código de fallback se `problemDetailsService` não conseguir gravar um `ProblemDetails`. Por exemplo, um ponto de extremidade em que o [cabeçalho de solicitação Accept](https://developer.mozilla.org/docs/Web/HTTP/Headers/Accept) especifica um tipo de mídia com a qual o `DefaulProblemDetailsWriter` não é compatível.
- Usa o [Middleware do manipulador de exceção](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/error-handling?view=aspnetcore-8.0#exception-handler-page).

O exemplo a seguir é semelhante ao anterior, exceto que ele chama o [`Status Code Pages middleware`](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/error-handling?view=aspnetcore-8.0#usestatuscodepages).

C#

```
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddProblemDetails();

var app = builder.Build();

app.UseStatusCodePages(statusCodeHandlerApp =>
{
    statusCodeHandlerApp.Run(async httpContext =>
    {
        var pds = httpContext.RequestServices.GetService<IProblemDetailsService>();
        if (pds == null
            || !await pds.TryWriteAsync(new() { HttpContext = httpContext }))
        {
            // Fallback behavior
            await httpContext.Response.WriteAsync("Fallback: An error occurred.");
        }
    });
});

app.MapGet("/users/{id:int}", (int id) =>
{
    return id <= 0 ? Results.BadRequest() : Results.Ok(new User(id));
});

app.MapGet("/", () => "Test by calling /users/{id:int}");

app.Run();

public record User(int Id);
```