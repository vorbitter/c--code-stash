## `WebApplication`

O código a seguir é gerado por um modelo do ASP.NET Core:

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello World!");

app.Run();
```

O código anterior pode ser criado por meio de `dotnet new web` na linha de comando ou da seleção do modelo Empty Web no Visual Studio.

O código a seguir cria um [WebApplication](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.webapplication) (`app`) sem criar explicitamente um [WebApplicationBuilder](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.webapplicationbuilder):

C#

```
var app = WebApplication.Create(args);

app.MapGet("/", () => "Hello World!");

app.Run();
```

[`WebApplication.Create`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.webapplication.create) inicializa uma nova instância da classe [WebApplication](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.webapplication) com padrões pré-configurados.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#working-with-ports)

### Trabalho com portas

Quando um aplicativo Web é criado com o Visual Studio ou com o `dotnet new`, aparece um arquivo `Properties/launchSettings.json` que especifica as portas às quais responde aplicativo. Nos exemplos de configuração de portas a seguir, a execução do aplicativo no Visual Studio retorna uma caixa de diálogo de erro `Unable to connect to web server 'AppName'`. O Visual Studio retorna um erro porque espera a porta especificada em `Properties/launchSettings.json`, mas o aplicativo está usando a porta especificada por `app.Run("http://localhost:3000")`. Execute a porta a seguir, alterando exemplos da linha de comando.

As seções a seguir definem a porta à qual responde o aplicativo.

C#

```
var app = WebApplication.Create(args);

app.MapGet("/", () => "Hello World!");

app.Run("http://localhost:3000");
```

No código anterior, o aplicativo responde à porta `3000`.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#multiple-ports)

#### Várias portas

No código a seguir, o aplicativo responde às portas `3000` e `4000`.

C#

```
var app = WebApplication.Create(args);

app.Urls.Add("http://localhost:3000");
app.Urls.Add("http://localhost:4000");

app.MapGet("/", () => "Hello World");

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#set-the-port-from-the-command-line)

#### Defina a porta na linha de comando

O comando a seguir faz com que o aplicativo responda à porta `7777`:

CLI do .NET

```
dotnet run --urls="https://localhost:7777"
```

Se o ponto de extremidade Kestrel também estiver configurado no arquivo `appsettings.json`, a URL especificada do arquivo `appsettings.json`será usada. Para obter mais informações, consulte [Configuração de ponto de extremidade Kestrel](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/configuration/?view=aspnetcore-8.0#kestrel)

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#read-the-port-from-environment)

#### Leia a porta do ambiente

O código a seguir lê a porta do ambiente:

C#

```
var app = WebApplication.Create(args);

var port = Environment.GetEnvironmentVariable("PORT") ?? "3000";

app.MapGet("/", () => "Hello World");

app.Run($"http://localhost:{port}");
```

A maneira preferencial de definir a porta do ambiente é usar a variável de ambiente `ASPNETCORE_URLS`, que é mostrada na seção a seguir.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#set-the-ports-via-the-aspnetcore_urls-environment-variable)

#### Defina as portas por meio da variável de ambiente ASPNETCORE_URLS

A variável de ambiente `ASPNETCORE_URLS` está disponível para definir a porta:

```
ASPNETCORE_URLS=http://localhost:3000
```

`ASPNETCORE_URLS` dá suporte a várias URLs:

```
ASPNETCORE_URLS=http://localhost:3000;https://localhost:5000
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#listen-on-all-interfaces)

### Escute em todas as interfaces

Os exemplos a seguir demonstram a escuta em todas as interfaces

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#http3000)

#### http://*:3000

C#

```
var app = WebApplication.Create(args);

app.Urls.Add("http://*:3000");

app.MapGet("/", () => "Hello World");

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#http3000-1)

#### http://+:3000

C#

```
var app = WebApplication.Create(args);

app.Urls.Add("http://+:3000");

app.MapGet("/", () => "Hello World");

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#http00003000)

#### `http://0.0.0.0:3000`

C#

```
var app = WebApplication.Create(args);

app.Urls.Add("http://0.0.0.0:3000");

app.MapGet("/", () => "Hello World");

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#listen-on-all-interfaces-using-aspnetcore_urls)

### Escute em todas as interfaces usando ASPNETCORE_URLS

Os exemplos anteriores podem usar `ASPNETCORE_URLS`

```
ASPNETCORE_URLS=http://*:3000;https://+:5000;http://0.0.0.0:5005
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#listen-on-all-interfaces-using-aspnetcore_https_ports)

### Escute em todas as interfaces usando ASPNETCORE_HTTPS_PORTS

Os exemplos anteriores podem usar `ASPNETCORE_HTTPS_PORTS` e `ASPNETCORE_HTTP_PORTS`.

```
ASPNETCORE_HTTP_PORTS=3000;5005
ASPNETCORE_HTTPS_PORTS=5000
```

Para obter mais informações, consulte [Configure pontos de extremidade para o Kestrel servidor de rede ASP.NET Core](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/servers/kestrel/endpoints?view=aspnetcore-8.0)

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#specify-https-with-development-certificate)

### Especifique HTTPS com certificado de desenvolvimento

C#

```
var app = WebApplication.Create(args);

app.Urls.Add("https://localhost:3000");

app.MapGet("/", () => "Hello World");

app.Run();
```

Para obter mais informações sobre o certificado de desenvolvimento, consulte [Confie no certificado de desenvolvimento HTTPS ASP.NET Core no Windows e no macOS](https://learn.microsoft.com/pt-br/aspnet/core/security/enforcing-ssl?view=aspnetcore-8.0#trust).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#specify-https-using-a-custom-certificate)

### Especifique HTTPS usando um certificado personalizado

As seções a seguir mostram como especificar o certificado personalizado usando o arquivo `appsettings.json` e por meio de configuração.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#specify-the-custom-certificate-with-appsettingsjson)

#### Especifique o certificado personalizado com `appsettings.json`

JSON

```
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "Kestrel": {
    "Certificates": {
      "Default": {
        "Path": "cert.pem",
        "KeyPath": "key.pem"
      }
    }
  }
}
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#specify-the-custom-certificate-via-configuration)

#### Especifique o certificado personalizado por meio de configuração

C#

```
var builder = WebApplication.CreateBuilder(args);

// Configure the cert and the key
builder.Configuration["Kestrel:Certificates:Default:Path"] = "cert.pem";
builder.Configuration["Kestrel:Certificates:Default:KeyPath"] = "key.pem";

var app = builder.Build();

app.Urls.Add("https://localhost:3000");

app.MapGet("/", () => "Hello World");

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#use-the-certificate-apis)

#### Use as APIs de certificado

C#

```
using System.Security.Cryptography.X509Certificates;

var builder = WebApplication.CreateBuilder(args);

builder.WebHost.ConfigureKestrel(options =>
{
    options.ConfigureHttpsDefaults(httpsOptions =>
    {
        var certPath = Path.Combine(builder.Environment.ContentRootPath, "cert.pem");
        var keyPath = Path.Combine(builder.Environment.ContentRootPath, "key.pem");

        httpsOptions.ServerCertificate = X509Certificate2.CreateFromPemFile(certPath, 
                                         keyPath);
    });
});

var app = builder.Build();

app.Urls.Add("https://localhost:3000");

app.MapGet("/", () => "Hello World");

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#read-the-environment)

### Leia o ambiente

C#

```
var app = WebApplication.Create(args);

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/oops");
}

app.MapGet("/", () => "Hello World");
app.MapGet("/oops", () => "Oops! An error happened.");

app.Run();
```

Para obter mais informações sobre como usar o ambiente, consulte [Use vários ambientes no ASP.NET Core](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/environments?view=aspnetcore-8.0)

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#configuration)

### Configuração

O código a seguir lê a partir do sistema de configuração:

C#

```
var app = WebApplication.Create(args);

var message = app.Configuration["HelloKey"] ?? "Config failed!";

app.MapGet("/", () => message);

app.Run();
```

Para obter mais informações, consulte [Configuração no ASP.NET Core](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/configuration/?view=aspnetcore-8.0)

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#logging)

### Registrando em log

O código a seguir grava uma mensagem na inicialização do aplicativo de logon:

C#

```
var app = WebApplication.Create(args);

app.Logger.LogInformation("The app started");

app.MapGet("/", () => "Hello World");

app.Run();
```

Para obter mais informações, consulte [Fazendo login no .NET Core e no ASP.NET Core](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/logging/?view=aspnetcore-8.0)

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#access-the-dependency-injection-di-container)

### Acesse o contêiner Injeção de Dependência (DI)

O código a seguir mostra como obter serviços do contêiner de DI durante a inicialização do aplicativo:

C#

```

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddScoped<SampleService>();

var app = builder.Build();

app.MapControllers();

using (var scope = app.Services.CreateScope())
{
    var sampleService = scope.ServiceProvider.GetRequiredService<SampleService>();
    sampleService.DoSomething();
}

app.Run();
```

O código a seguir mostra como acessar chaves do contêiner de DI usando o atributo [`[FromKeyedServices]`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.extensions.dependencyinjection.fromkeyedservicesattribute):

C#

```
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddKeyedSingleton<ICache, BigCache>("big");
builder.Services.AddKeyedSingleton<ICache, SmallCache>("small");

var app = builder.Build();

app.MapGet("/big", ([FromKeyedServices("big")] ICache bigCache) => bigCache.Get("date"));

app.MapGet("/small", ([FromKeyedServices("small")] ICache smallCache) => smallCache.Get("date"));

app.Run();

public interface ICache
{
    object Get(string key);
}
public class BigCache : ICache
{
    public object Get(string key) => $"Resolving {key} from big cache.";
}

public class SmallCache : ICache
{
    public object Get(string key) => $"Resolving {key} from small cache.";
}
```

Para obter mais informações sobre DI, confira [Injeção de dependência no ASP.NET Core](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-8.0).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#webapplicationbuilder)

## WebApplicationBuilder

Esta seção contém o código de exemplo usando o [WebApplicationBuilder](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.webapplicationbuilder).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#change-the-content-root-application-name-and-environment)

### Altere a raiz do conteúdo, o nome do aplicativo e o ambiente

O código a seguir define a raiz do conteúdo, o nome do aplicativo e o ambiente:

C#

```
var builder = WebApplication.CreateBuilder(new WebApplicationOptions
{
    Args = args,
    ApplicationName = typeof(Program).Assembly.FullName,
    ContentRootPath = Directory.GetCurrentDirectory(),
    EnvironmentName = Environments.Staging,
    WebRootPath = "customwwwroot"
});

Console.WriteLine($"Application Name: {builder.Environment.ApplicationName}");
Console.WriteLine($"Environment Name: {builder.Environment.EnvironmentName}");
Console.WriteLine($"ContentRoot Path: {builder.Environment.ContentRootPath}");
Console.WriteLine($"WebRootPath: {builder.Environment.WebRootPath}");

var app = builder.Build();
```

O [WebApplication.CreateBuilder](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.webapplication.createbuilder) inicializa uma nova instância da classe [WebApplicationBuilder](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.webapplicationbuilder) com os padrões pré-configurados.

Para obter mais informações, consulte [visão geral dos conceitos básicos do ASP.NET Core](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/?view=aspnetcore-8.0)

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#change-the-content-root-app-name-and-environment-by-using-environment-variables-or-command-line)

### Altere a raiz do conteúdo, o nome do aplicativo e o ambiente usando variáveis de ambiente ou linha de comando

A tabela a seguir mostra a variável de ambiente e o argumento de linha de comando usados para alterar a raiz do conteúdo, o nome do aplicativo e o ambiente:

|recurso|Variável de ambiente|Argumento de linha de comando|
|---|---|---|
|Nome do aplicativo|ASPNETCORE_APPLICATIONNAME|--applicationName|
|Nome do ambiente|ASPNETCORE_ENVIRONMENT|--environment|
|Raiz do conteúdo|ASPNETCORE_CONTENTROOT|--contentRoot|

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#add-configuration-providers)

### Adicionar provedores de configuração

O exemplo a seguir adiciona o provedor de configuração INI:

C#

```
var builder = WebApplication.CreateBuilder(args);

builder.Configuration.AddIniFile("appsettings.ini");

var app = builder.Build();
```

Para maior detalhamento, consulte [Provedores de configuração de arquivo](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/configuration/?view=aspnetcore-8.0#file-configuration-provider) em [Configuração no ASP.NET Core](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/configuration/?view=aspnetcore-8.0).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#read-configuration)

### Leia a configuração

Por padrão, o [WebApplicationBuilder](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.webapplicationbuilder) lê a configuração de várias fontes, incluindo:

- `appSettings.json` e `appSettings.{environment}.json`
- Variáveis de ambiente
- A linha de comando

Para obter uma lista completa das fontes de configuração lidas, confira a [Configuração padrão](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/configuration/?view=aspnetcore-8.0#default-configuration) em [Configuração no ASP.NET Core](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/configuration/?view=aspnetcore-8.0).

O código a seguir lê `HelloKey` a partir da configuração e exibe o valor no ponto de extremidade`/`. Se o valor de configuração for nulo, "Hello" será atribuído a `message`:

C#

```
var builder = WebApplication.CreateBuilder(args);

var message = builder.Configuration["HelloKey"] ?? "Hello";

var app = builder.Build();

app.MapGet("/", () => message);

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#read-the-environment-1)

### Leia o ambiente

C#

```
var builder = WebApplication.CreateBuilder(args);

if (builder.Environment.IsDevelopment())
{
    Console.WriteLine($"Running in development.");
}

var app = builder.Build();

app.MapGet("/", () => "Hello World!");

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#add-logging-providers)

### Adicione provedores de login

C#

```
var builder = WebApplication.CreateBuilder(args);

// Configure JSON logging to the console.
builder.Logging.AddJsonConsole();

var app = builder.Build();

app.MapGet("/", () => "Hello JSON console!");

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#add-services)

### Adicionar serviços

C#

```
var builder = WebApplication.CreateBuilder(args);

// Add the memory cache services.
builder.Services.AddMemoryCache();

// Add a custom scoped service.
builder.Services.AddScoped<ITodoRepository, TodoRepository>();
var app = builder.Build();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#customize-the-ihostbuilder)

### Personalize o IHostBuilder

Os métodos de extensão existentes no [IHostBuilder](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.extensions.hosting.ihostbuilder) podem ser acessados usando a [propriedade Host](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.extensions.hosting.ihostbuilder.properties#microsoft-extensions-hosting-ihostbuilder-properties):

C#

```
var builder = WebApplication.CreateBuilder(args);

// Wait 30 seconds for graceful shutdown.
builder.Host.ConfigureHostOptions(o => o.ShutdownTimeout = TimeSpan.FromSeconds(30));

var app = builder.Build();

app.MapGet("/", () => "Hello World!");

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#customize-the-iwebhostbuilder)

### Personalize o IWebHostBuilder

Os métodos de extensão no [IWebHostBuilder](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder) podem ser acessados usando a propriedade [WebApplicationBuilder.WebHost](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.webapplicationbuilder.webhost#microsoft-aspnetcore-builder-webapplicationbuilder-webhost).

C#

```
var builder = WebApplication.CreateBuilder(args);

// Change the HTTP server implemenation to be HTTP.sys based
builder.WebHost.UseHttpSys();

var app = builder.Build();

app.MapGet("/", () => "Hello HTTP.sys");

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#change-the-web-root)

### Altere a raiz da Web

Por padrão, a raiz da Web refere-se à raiz do conteúdo na pasta `wwwroot`. A raiz da Web é o local onde o middleware de arquivos estáticos procura arquivos estáticos. A raiz da Web pode ser alterada com o `WebHostOptions`, com a linha de comando ou com o método [UseWebRoot](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.usewebroot):

C#

```
var builder = WebApplication.CreateBuilder(new WebApplicationOptions
{
    Args = args,
    // Look for static files in webroot
    WebRootPath = "webroot"
});

var app = builder.Build();

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#custom-dependency-injection-di-container)

### Personalize o contêiner de Injeção de Dependência (DI)

O exemplo a seguir usa o [Autofac](https://autofac.readthedocs.io/en/latest/integration/aspnetcore.html):

C#

```
var builder = WebApplication.CreateBuilder(args);

builder.Host.UseServiceProviderFactory(new AutofacServiceProviderFactory());

// Register services directly with Autofac here. Don't
// call builder.Populate(), that happens in AutofacServiceProviderFactory.
builder.Host.ConfigureContainer<ContainerBuilder>(builder => builder.RegisterModule(new MyApplicationModule()));

var app = builder.Build();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#add-middleware)

### Adicione o middleware

Qualquer middleware do ASP.NET Core pode ser configurado no `WebApplication`:

C#

```
var app = WebApplication.Create(args);

// Setup the file server to serve static files.
app.UseFileServer();

app.MapGet("/", () => "Hello World!");

app.Run();
```

Para obter mais informações, consulte [Middleware do ASP.NET Core](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/middleware/?view=aspnetcore-8.0)

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#developer-exception-page)

### Página de exceção do desenvolvedor

[WebApplication.CreateBuilder](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.webapplication.createbuilder) inicializa uma nova instância da classe [WebApplicationBuilder](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.webapplicationbuilder) com padrões pré-configurados. A página de exceção do desenvolvedor está habilitada nos padrões pré-configurados. Quando o código a seguir é executado no [ambiente de desenvolvimento](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/environments?view=aspnetcore-8.0), navegar para `/` renderiza uma página amigável que mostra a exceção.

C#

```
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.MapGet("/", () =>
{
    throw new InvalidOperationException("Oops, the '/' route has thrown an exception.");
});

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#aspnet-core-middleware)

## Middleware do ASP.NET Core

A tabela a seguir lista alguns dos middleware usados com frequência com as APIs mínimas.

|Middleware|Descrição|API|
|---|---|---|
|[Autenticação](https://learn.microsoft.com/pt-br/aspnet/core/security/authentication/?view=aspnetcore-6.0)|Fornece suporte à autenticação.|[UseAuthentication](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.authappbuilderextensions.useauthentication)|
|[Autorização](https://learn.microsoft.com/pt-br/aspnet/core/security/authorization/introduction?view=aspnetcore-8.0)|Fornece suporte à autorização.|[UseAuthorization](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.authorizationappbuilderextensions.useauthorization)|
|[CORS](https://learn.microsoft.com/pt-br/aspnet/core/security/cors?view=aspnetcore-6.0)|Configura o Compartilhamento de Recursos entre Origens.|[UseCors](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.corsmiddlewareextensions.usecors)|
|[Manipulador de exceção](https://learn.microsoft.com/pt-br/aspnet/core/web-api/handle-errors?view=aspnetcore-6.0)|Lida globalmente com exceções geradas pelo pipeline de middleware.|[UseExceptionHandler](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.exceptionhandlerextensions.useexceptionhandler)|
|[Cabeçalhos encaminhados](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/middleware/?view=aspnetcore-6.0#forwarded-headers-middleware-order)|Encaminha cabeçalhos como proxy para a solicitação atual.|[UseForwardedHeaders](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.forwardedheadersextensions.useforwardedheaders)|
|[Redirecionamento de HTTPS](https://learn.microsoft.com/pt-br/aspnet/core/security/enforcing-ssl?view=aspnetcore-6.0)|Redireciona todas as solicitações HTTP para HTTPS.|[UseHttpsRedirection](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.httpspolicybuilderextensions.usehttpsredirection)|
|[Segurança de Transporte Estrita de HTTP (HSTS)](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/middleware/?view=aspnetcore-6.0#middleware-order)|Middleware de aprimoramento de segurança que adiciona um cabeçalho de resposta especial.|[UseHsts](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.hstsbuilderextensions.usehsts)|
|[Registro em log de solicitação](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/logging/?view=aspnetcore-6.0)|Fornece suporte para registro de solicitações e respostas HTTP.|[UseHttpLogging](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.httploggingbuilderextensions.usehttplogging)|
|[Tempos Limite de Solicitação](https://learn.microsoft.com/pt-br/aspnet/core/performance/timeouts?view=aspnetcore-8.0)|Fornece suporte para configurar tempos limite de solicitação, padrão global e por ponto de extremidade.|`UseRequestTimeouts`|
|[Registro em log de solicitação em W3C](https://www.w3.org/TR/WD-logfile.html)|Fornece suporte para registro em log de solicitações e respostas HTTP no [formato W3C](https://www.w3.org/TR/WD-logfile.html).|[UseW3CLogging](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.httploggingbuilderextensions.usew3clogging)|
|[Cache de resposta](https://learn.microsoft.com/pt-br/aspnet/core/performance/caching/middleware?view=aspnetcore-8.0)|Fornece suporte para as respostas em cache.|[UseResponseCaching](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.responsecachingextensions.useresponsecaching)|
|[Compactação de resposta](https://learn.microsoft.com/pt-br/aspnet/core/performance/response-compression?view=aspnetcore-8.0)|Fornece suporte para a compactação de respostas.|[UseResponseCompression](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.responsecompressionbuilderextensions.useresponsecompression)|
|[Sessão](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/app-state?view=aspnetcore-8.0)|Fornece suporte para gerenciar sessões de usuário.|[UseSession](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.sessionmiddlewareextensions.usesession)|
|[Arquivos estáticos](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/static-files?view=aspnetcore-8.0)|Fornece suporte para servir arquivos estáticos e pesquisa no diretório.|[UseStaticFiles](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.staticfileextensions.usestaticfiles), [UseFileServer](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.fileserverextensions.usefileserver)|
|[WebSockets](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/websockets?view=aspnetcore-8.0)|Habilita o protocolo WebSockets.|[UseWebSockets](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.websocketmiddlewareextensions.usewebsockets)|

As seções a seguir abrangem o tratamento de solicitações: roteamento, associação de parâmetros e respostas.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#routing)

## Roteamento

Um `WebApplication` configurado dá suporte a `Map{Verb}` e [MapMethods](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.endpointroutebuilderextensions.mapmethods), onde `{Verb}` é um método HTTP com maiúsculas e minúsculas, como `Get`, `Post`, `Put` ou `Delete`:

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#route-handlers)

### Manipuladores de rota

Manipuladores de rotas são métodos executados quando a rota corresponde. Os manipuladores de rotas podem ser uma expressão lambda, uma função local, um método de instância ou um método estático. Os manipuladores de rota podem ser síncronos ou assíncronos.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#lambda-expression)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#local-function)

### Função local

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

string LocalFunction() => "This is local function";

app.MapGet("/", LocalFunction);

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#instance-method)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#static-method)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#endpoint-defined-outside-of-programcs)

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

Também consulte [Grupos de rota](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#route-groups) mais adiante neste artigo.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#named-endpoints-and-link-generation)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#route-parameters)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#wildcard-and-catch-all-routes)

### Caractere curinga e capturar todas as rotas

A rota catch-all a seguir retorna `Routing to hello` do ponto de extremidade '/posts/hello':

C#

```
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/posts/{*rest}", (string rest) => $"Routing to {rest}");

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#route-constraints)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#route-groups)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#parameter-binding)

## Associação de parâmetros

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

Os métodos HTTP `GET`, `HEAD`, `OPTIONS`, e `DELETE` não se associam implicitamente a partir do corpo. Para associar a partir do corpo (como JSON) para esses métodos HTTP, [associe explicitamente](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#explicit-parameter-binding) com [`[FromBody]`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.mvc.frombodyattribute) ou leia de [HttpRequest](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.httprequest).

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#explicit-parameter-binding)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#explicit-binding-from-form-values)

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

Para obter mais informações, confira a seção sobre [AsParameters](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#parameter-binding-for-argument-lists-with-asparameters) mais adiante neste artigo.

O [código de exemplo completo](https://github.com/dotnet/AspNetCore.Docs.Samples/tree/main/fundamentals/minimal-apis/samples/IFormFile) está no repositório [AspNetCore.Docs.Samples](https://github.com/dotnet/AspNetCore.Docs.Samples).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#secure-binding-from-iformfile-and-iformfilecollection)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#parameter-binding-with-dependency-injection)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#optional-parameters)

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

Obtenha mais informações na seção [Falhas de Associação](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#bf).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#special-types)

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
    

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#bind-the-request-body-as-a-stream-or-pipereader)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#file-uploads-using-iformfile-and-iformfilecollection)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#binding-to-forms-with-iformcollection-iformfile-and-iformfilecollection)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#bind-to-collections-and-complex-types-from-forms)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#bind-arrays-and-string-values-from-headers-and-query-strings)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#parameter-binding-for-argument-lists-with-asparameters)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#custom-binding)

### Associação personalizado

Há duas maneiras de personalizar a associação de parâmetros:

1. Para as origens da associação de rota, de consulta e de cabeçalho, associe tipos personalizados adicionando um método estático `TryParse` para o tipo.
2. Controlar o processo de associação implementando um método `BindAsync` em um tipo.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#tryparse)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#bindasync)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#binding-failures)

### Falhas de associação

Quando a associação falha, a estrutura registra uma mensagem de depuração e retorna vários códigos de status ao cliente, dependendo do modo de falha.

|Modo de falha|Tipo de parâmetro anulável|Origem da associação|Código de status|
|---|---|---|---|
|`{ParameterType}.TryParse` retorna `false`|sim|rota/consulta/cabeçalho|400|
|`{ParameterType}.BindAsync` retorna `null`|sim|personalizado|400|
|`{ParameterType}.BindAsync` gera|não importa|custom|500|
|Falha na desserialização do corpo JSON|não importa|body|400|
|Tipo de conteúdo incorreto (não `application/json`)|não importa|body|415|

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#binding-precedence)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#configure-json-deserialization-options-for-body-binding)

### Configurar opções de desserialização JSON para associação de corpo

A origem da associação do corpo usa [System.Text.Json](https://learn.microsoft.com/pt-br/dotnet/api/system.text.json) para desserialização. _**Não**_ é possível alterar esse padrão, mas as opções de serialização e desserialização JSON podem ser configuradas.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#configure-json-deserialization-options-globally)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#configure-json-deserialization-options-for-an-endpoint)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#read-the-request-body)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#responses)

## Respostas

Os manipuladores de rota dão suporte aos seguintes tipos de valores retornados:

1. Baseado em `IResult` - Inclui `Task<IResult>` e `ValueTask<IResult>`
2. `string` - Inclui `Task<string>` e `ValueTask<string>`
3. `T` (Qualquer outro tipo) – inclui `Task<T>` e `ValueTask<T>`

|Valor retornado|Comportamento|Tipo de conteúdo|
|---|---|---|
|`IResult`|A estrutura chama [IResult.ExecuteAsync](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iresult.executeasync)|Decidido pela implementação de `IResult`|
|`string`|A estrutura grava a cadeia de caracteres diretamente na resposta|`text/plain`|
|`T` (Qualquer outro tipo)|A estrutura JSON serializa a resposta|`application/json`|

Obtenha um guia mais detalhado dos valores retornados do manipulador de rota em [Crie respostas em aplicativos de APIs mínimas](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-8.0)

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#example-return-values)

### Exemplo de valores retornados

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#string-return-values)

#### valores retornados de cadeias de caracteres

C#

```
app.MapGet("/hello", () => "Hello World");
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#json-return-values)

#### Valores JSON retornados

C#

```
app.MapGet("/hello", () => new { Message = "Hello World" });
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#return-typedresults)

#### Retornar TypedResults

O código a seguir retorna um [TypedResults](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.typedresults):

C#

```
app.MapGet("/hello", () => TypedResults.Ok(new Message() {  Text = "Hello World!" }));
```

É preferível retornar `TypedResults` a retornar [Results](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.results). Para obter mais informações, consulte [TypedResults vs Resultados](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses#typedresults-vs-results).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#iresult-return-values)

#### Valores retornados de IResult

C#

```
app.MapGet("/hello", () => Results.Ok(new { Message = "Hello World" }));
```

O exemplo a seguir usa os tipos de resultados internos para personalizar a resposta:

C#

```
app.MapGet("/api/todoitems/{id}", async (int id, TodoDb db) =>
         await db.Todos.FindAsync(id) 
         is Todo todo
         ? Results.Ok(todo) 
         : Results.NotFound())
   .Produces<Todo>(StatusCodes.Status200OK)
   .Produces(StatusCodes.Status404NotFound);
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#json)

#### JSON

C#

```
app.MapGet("/hello", () => Results.Json(new { Message = "Hello World" }));
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#custom-status-code)

#### Código de status personalizado

C#

```
app.MapGet("/405", () => Results.StatusCode(405));
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#text)

#### Texto

C#

```
app.MapGet("/text", () => Results.Text("This is some text"));
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#stream)

#### Fluxo

C#

```
var proxyClient = new HttpClient();
app.MapGet("/pokemon", async () => 
{
    var stream = await proxyClient.GetStreamAsync("http://contoso/pokedex.json");
    // Proxy the response as JSON
    return Results.Stream(stream, "application/json");
});
```

Obtenha mais exemplos em [Crie respostas em aplicativos de APIs mínimas](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-8.0#stream7).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#redirect)

#### Redirecionar

C#

```
app.MapGet("/old-path", () => Results.Redirect("/new-path"));
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#file)

#### Arquivo

C#

```
app.MapGet("/download", () => Results.File("myfile.text"));
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#built-in-results)

### Resultados internos

Auxiliares de resultados comuns existem no [Results](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.results) e nas classes estáticas [TypedResults](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.typedresults). É preferível retornar `TypedResults` a retornar `Results`. Para obter mais informações, consulte [TypedResults vs Resultados](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses#typedresults-vs-results).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#customizing-results)

### Personalizando resultados

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#typed-results)

### Resultados de tipos

A interface [IResult](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.iresult) pode representar valores retornados de APIs mínimas que não aproveitam o suporte implícito para JSON, serializando o objeto retornado para a resposta HTTP. A classe estática [Results](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.results) é usada para criar objetos `IResult` variados que representam diferentes tipos de respostas. Por exemplo, configurando o código de status de resposta ou redirecionando para outra URL.

Os tipos de implementação `IResult` são públicos, permitindo instruções de declaração de tipo durante o teste. Por exemplo:

C#

```
[TestClass()]
public class WeatherApiTests
{
    [TestMethod()]
    public void MapWeatherApiTest()
    {
        var result = WeatherApi.GetAllWeathers();
        Assert.IsInstanceOfType(result, typeof(Ok<WeatherForecast[]>));
    }      
}
```

É possível examinar os tipos de retorno dos métodos correspondentes na classe estática [TypedResults](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.http.typedresults) e localizar o tipo público `IResult` correto para converter.

Obtenha mais exemplos em [Crie respostas em aplicativos de APIs mínimas](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/responses?view=aspnetcore-8.0).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#filters)

## Filtros

Para obter mais informações, veja [Filtros em aplicativos API mínimos](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/min-api-filters?view=aspnetcore-8.0).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#authorization)

## Autorização

Rotas podem ser protegidas por meio de políticas de autorização. É possível declará-las por meio do atributo [`[Authorize]`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute) ou utilizando o método [RequireAuthorization](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.authorizationendpointconventionbuilderextensions.requireauthorization):

C#

```
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using WebRPauth.Data;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddAuthorization(o => o.AddPolicy("AdminsOnly", 
                                  b => b.RequireClaim("admin", "true")));

var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString));
builder.Services.AddDatabaseDeveloperPageExceptionFilter();

builder.Services.AddDefaultIdentity<IdentityUser>(options => options.SignIn.RequireConfirmedAccount = true)
    .AddEntityFrameworkStores<ApplicationDbContext>();

var app = builder.Build();

app.UseAuthorization();

app.MapGet("/auth", [Authorize] () => "This endpoint requires authorization.");
app.MapGet("/", () => "This endpoint doesn't require authorization.");
app.MapGet("/Identity/Account/Login", () => "Sign in page at this endpoint.");

app.Run();
```

O código anterior pode ser gravado com [RequireAuthorization](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.authorizationendpointconventionbuilderextensions.requireauthorization):

C#

```
app.MapGet("/auth", () => "This endpoint requires authorization")
   .RequireAuthorization();
```

O exemplo a seguir utiliza a [autorização baseada em políticas](https://learn.microsoft.com/pt-br/aspnet/core/security/authorization/policies?view=aspnetcore-8.0):

C#

```
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using WebRPauth.Data;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddAuthorization(o => o.AddPolicy("AdminsOnly", 
                                  b => b.RequireClaim("admin", "true")));

var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString));
builder.Services.AddDatabaseDeveloperPageExceptionFilter();

builder.Services.AddDefaultIdentity<IdentityUser>(options => options.SignIn.RequireConfirmedAccount = true)
    .AddEntityFrameworkStores<ApplicationDbContext>();

var app = builder.Build();

app.UseAuthorization();

app.MapGet("/admin", [Authorize("AdminsOnly")] () => 
                             "The /admin endpoint is for admins only.");

app.MapGet("/admin2", () => "The /admin2 endpoint is for admins only.")
   .RequireAuthorization("AdminsOnly");

app.MapGet("/", () => "This endpoint doesn't require authorization.");
app.MapGet("/Identity/Account/Login", () => "Sign in page at this endpoint.");

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#allow-unauthenticated-users-to-access-an-endpoint)

### Permita o acesso de usuários não autenticados a um ponto de extremidade

O [`[AllowAnonymous]`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.authorization.allowanonymousattribute) permite o acesso de usuários não autenticados a pontos de extremidade:

C#

```
app.MapGet("/login", [AllowAnonymous] () => "This endpoint is for all roles.");


app.MapGet("/login2", () => "This endpoint also for all roles.")
   .AllowAnonymous();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#cors)

## CORS

Rotas podem ser habilitadas para [CORS](https://learn.microsoft.com/pt-br/aspnet/core/security/cors?view=aspnetcore-6.0) com as [políticas CORS](https://learn.microsoft.com/pt-br/aspnet/core/security/cors?view=aspnetcore-6.0#cors-policy-options). O CORS pode ser declarado por meio do atributo [`[EnableCors]`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.cors.enablecorsattribute) ou utilizando o método [RequireCors](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.corsendpointconventionbuilderextensions.requirecors). As seguintes amostras habilitam o CORS:

C#

```
const string MyAllowSpecificOrigins = "_myAllowSpecificOrigins";

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddCors(options =>
{
    options.AddPolicy(name: MyAllowSpecificOrigins,
                      builder =>
                      {
                          builder.WithOrigins("http://example.com",
                                              "http://www.contoso.com");
                      });
});

var app = builder.Build();
app.UseCors();

app.MapGet("/",() => "Hello CORS!");

app.Run();
```

C#

```
using Microsoft.AspNetCore.Cors;

const string MyAllowSpecificOrigins = "_myAllowSpecificOrigins";

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddCors(options =>
{
    options.AddPolicy(name: MyAllowSpecificOrigins,
                      builder =>
                      {
                          builder.WithOrigins("http://example.com",
                                              "http://www.contoso.com");
                      });
});

var app = builder.Build();
app.UseCors();

app.MapGet("/cors", [EnableCors(MyAllowSpecificOrigins)] () => 
                           "This endpoint allows cross origin requests!");
app.MapGet("/cors2", () => "This endpoint allows cross origin requests!")
                     .RequireCors(MyAllowSpecificOrigins);

app.Run();
```

Para obter mais informações, consulte [Habilitar solicitações entre origens (CORS) no ASP.NET Core](https://learn.microsoft.com/pt-br/aspnet/core/security/cors?view=aspnetcore-6.0)

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-8.0#validatescopes-and-validateonbuild)

## ValidateScopes e ValidateOnBuild

[ValidateScopes](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.extensions.dependencyinjection.serviceprovideroptions.validatescopes#microsoft-extensions-dependencyinjection-serviceprovideroptions-validatescopes) e [ValidateOnBuild](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.extensions.dependencyinjection.serviceprovideroptions.validateonbuild#microsoft-extensions-dependencyinjection-serviceprovideroptions-validateonbuild) são habilitados por padrão no ambiente de [Desenvolvimento](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/environments?view=aspnetcore-8.0), mas desabilitados em outros ambientes.

Quando `ValidateOnBuild` é `true`, o contêiner de DI valida a configuração do serviço no momento do build. Se a configuração do serviço for inválida, o build falhará na inicialização do aplicativo, em vez de em runtime, quando o serviço for solicitado.

Quando `ValidateScopes` é `true`, o contêiner de DI valida que um serviço com escopo não está resolvido no escopo raiz. Resolver um serviço com escopo no escopo raiz pode resultar em perda de memória porque o serviço é mantido na memória por mais tempo do que o escopo da solicitação.

`ValidateScopes` e `ValidateOnBuild` são falsos por padrão nos modos que não são de desenvolvimento por questão de desempenho.

O código a seguir mostra que `ValidateScopes` está habilitada por padrão no modo de desenvolvimento, mas desabilitada no modo de versão:

C#

```
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddScoped<MyScopedService>();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    Console.WriteLine("Development environment");
}
else
{
    Console.WriteLine("Release environment");
}

app.MapGet("/", context =>
{
    // Intentionally getting service provider from app, not from the request
    // This causes an exception from attempting to resolve a scoped service
    // outside of a scope.
    // Throws System.InvalidOperationException:
    // 'Cannot resolve scoped service 'MyScopedService' from root provider.'
    var service = app.Services.GetRequiredService<MyScopedService>();
    return context.Response.WriteAsync("Service resolved");
});

app.Run();

public class MyScopedService { }
```

O código a seguir mostra que `ValidateOnBuild` está habilitada por padrão no modo de desenvolvimento, mas desabilitada no modo de versão:

C#

```
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddScoped<MyScopedService>();
builder.Services.AddScoped<AnotherService>();

// System.AggregateException: 'Some services are not able to be constructed (Error
// while validating the service descriptor 'ServiceType: AnotherService Lifetime:
// Scoped ImplementationType: AnotherService': Unable to resolve service for type
// 'BrokenService' while attempting to activate 'AnotherService'.)'
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    Console.WriteLine("Development environment");
}
else
{
    Console.WriteLine("Release environment");
}

app.MapGet("/", context =>
{
    var service = context.RequestServices.GetRequiredService<MyScopedService>();
    return context.Response.WriteAsync("Service resolved correctly!");
});

app.Run();

public class MyScopedService { }

public class AnotherService
{
    public AnotherService(BrokenService brokenService) { }
}

public class BrokenService { }
```

O código a seguir desabilita `ValidateScopes` e `ValidateOnBuild` em `Development`:

C#

```
var builder = WebApplication.CreateBuilder(args);

if (builder.Environment.IsDevelopment())
{
    Console.WriteLine("Development environment");
    // Doesn't detect the validation problems because ValidateScopes is false.
    builder.Host.UseDefaultServiceProvider(options =>
    {
        options.ValidateScopes = false;
        options.ValidateOnBuild = false;
    });
}
```