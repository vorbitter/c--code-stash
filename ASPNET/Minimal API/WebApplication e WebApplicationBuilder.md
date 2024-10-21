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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#working-with-ports)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#multiple-ports)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#set-the-port-from-the-command-line)

#### Defina a porta na linha de comando

O comando a seguir faz com que o aplicativo responda à porta `7777`:

CLI do .NET

```
dotnet run --urls="https://localhost:7777"
```

Se o ponto de extremidade Kestrel também estiver configurado no arquivo `appsettings.json`, a URL especificada do arquivo `appsettings.json`será usada. Para obter mais informações, consulte [Configuração de ponto de extremidade Kestrel](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/configuration/?view=aspnetcore-8.0#kestrel)

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#read-the-port-from-environment)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#set-the-ports-via-the-aspnetcore_urls-environment-variable)

#### Defina as portas por meio da variável de ambiente ASPNETCORE_URLS

A variável de ambiente `ASPNETCORE_URLS` está disponível para definir a porta:

```
ASPNETCORE_URLS=http://localhost:3000
```

`ASPNETCORE_URLS` dá suporte a várias URLs:

```
ASPNETCORE_URLS=http://localhost:3000;https://localhost:5000
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#listen-on-all-interfaces)

### Escute em todas as interfaces

Os exemplos a seguir demonstram a escuta em todas as interfaces

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#http3000)

#### http://*:3000

C#

```
var app = WebApplication.Create(args);

app.Urls.Add("http://*:3000");

app.MapGet("/", () => "Hello World");

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#http3000-1)

#### http://+:3000

C#

```
var app = WebApplication.Create(args);

app.Urls.Add("http://+:3000");

app.MapGet("/", () => "Hello World");

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#http00003000)

#### `http://0.0.0.0:3000`

C#

```
var app = WebApplication.Create(args);

app.Urls.Add("http://0.0.0.0:3000");

app.MapGet("/", () => "Hello World");

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#listen-on-all-interfaces-using-aspnetcore_urls)

### Escute em todas as interfaces usando ASPNETCORE_URLS

Os exemplos anteriores podem usar `ASPNETCORE_URLS`

```
ASPNETCORE_URLS=http://*:3000;https://+:5000;http://0.0.0.0:5005
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#listen-on-all-interfaces-using-aspnetcore_https_ports)

### Escute em todas as interfaces usando ASPNETCORE_HTTPS_PORTS

Os exemplos anteriores podem usar `ASPNETCORE_HTTPS_PORTS` e `ASPNETCORE_HTTP_PORTS`.

```
ASPNETCORE_HTTP_PORTS=3000;5005
ASPNETCORE_HTTPS_PORTS=5000
```

Para obter mais informações, consulte [Configure pontos de extremidade para o Kestrel servidor de rede ASP.NET Core](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/servers/kestrel/endpoints?view=aspnetcore-8.0)

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#specify-https-with-development-certificate)

### Especifique HTTPS com certificado de desenvolvimento

C#

```
var app = WebApplication.Create(args);

app.Urls.Add("https://localhost:3000");

app.MapGet("/", () => "Hello World");

app.Run();
```

Para obter mais informações sobre o certificado de desenvolvimento, consulte [Confie no certificado de desenvolvimento HTTPS ASP.NET Core no Windows e no macOS](https://learn.microsoft.com/pt-br/aspnet/core/security/enforcing-ssl?view=aspnetcore-8.0#trust).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#specify-https-using-a-custom-certificate)

### Especifique HTTPS usando um certificado personalizado

As seções a seguir mostram como especificar o certificado personalizado usando o arquivo `appsettings.json` e por meio de configuração.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#specify-the-custom-certificate-with-appsettingsjson)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#specify-the-custom-certificate-via-configuration)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#use-the-certificate-apis)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#read-the-environment)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#configuration)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#logging)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#access-the-dependency-injection-di-container)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#webapplicationbuilder)

## WebApplicationBuilder

Esta seção contém o código de exemplo usando o [WebApplicationBuilder](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.webapplicationbuilder).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#change-the-content-root-application-name-and-environment)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#change-the-content-root-app-name-and-environment-by-using-environment-variables-or-command-line)

### Altere a raiz do conteúdo, o nome do aplicativo e o ambiente usando variáveis de ambiente ou linha de comando

A tabela a seguir mostra a variável de ambiente e o argumento de linha de comando usados para alterar a raiz do conteúdo, o nome do aplicativo e o ambiente:

|recurso|Variável de ambiente|Argumento de linha de comando|
|---|---|---|
|Nome do aplicativo|ASPNETCORE_APPLICATIONNAME|--applicationName|
|Nome do ambiente|ASPNETCORE_ENVIRONMENT|--environment|
|Raiz do conteúdo|ASPNETCORE_CONTENTROOT|--contentRoot|

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#add-configuration-providers)

### Adicionar provedores de configuração

O exemplo a seguir adiciona o provedor de configuração INI:

C#

```
var builder = WebApplication.CreateBuilder(args);

builder.Configuration.AddIniFile("appsettings.ini");

var app = builder.Build();
```

Para maior detalhamento, consulte [Provedores de configuração de arquivo](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/configuration/?view=aspnetcore-8.0#file-configuration-provider) em [Configuração no ASP.NET Core](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/configuration/?view=aspnetcore-8.0).

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#read-configuration)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#read-the-environment-1)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#add-logging-providers)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#add-services)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#customize-the-ihostbuilder)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#customize-the-iwebhostbuilder)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#change-the-web-root)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#custom-dependency-injection-di-container)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#add-middleware)

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

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0#developer-exception-page)

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