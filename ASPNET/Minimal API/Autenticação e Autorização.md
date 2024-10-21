As APIs mínimas dão suporte a todas as opções de autenticação e autorização disponíveis no ASP.NET Core e fornecem algumas funcionalidades adicionais para melhorar a experiência de trabalho com autenticação.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/security?view=aspnetcore-8.0#key-concepts-in-authentication-and-authorization)

## Principais conceitos de autenticação e autorização

Autenticação é o processo de determinar a identity de um usuário. Autorização é o processo de determinar se um usuário tem acesso a um recurso. Os cenários de autenticação e autorização compartilham semânticas de implementação semelhantes no ASP.NET Core. A autenticação é tratada pelo serviço de autenticação, [IAuthenticationService](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.authentication.iauthenticationservice), que é usado pelo [middleware](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/middleware) de autenticação. A autorização é tratada pelo serviço de autorização [, IAuthorizationService](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationservice), que é usado pelo middleware de autorização.

O serviço de autenticação usa manipuladores de autenticação registrados para realizar ações relacionadas à autenticação. Por exemplo, uma ação relacionada à autenticação é autenticar um usuário ou sair de um usuário. Esquemas de autenticação são nomes usados para identificar exclusivamente um manipulador de autenticação e suas opções de configuração. Os manipuladores de autenticação são responsáveis por implementar as estratégias de autenticação e gerar declarações de um usuário, considerando uma estratégia de autenticação específica, como OAuth ou OIDC. As opções de configuração também são exclusivas para a estratégia e fornecem ao manipulador a configuração que afeta o comportamento de autenticação, como URIs de redirecionamento.

Há duas estratégias para determinar o acesso do usuário aos recursos na camada de autorização:

- As estratégias baseadas em função determinam o acesso de um usuário com base na função atribuída, como `Administrator` ou `User`. Para obter mais informações sobre autorização baseada em função, confira a [documentação de autorização baseada em função](https://learn.microsoft.com/pt-br/aspnet/core/security/authorization/roles).
- As estratégias baseadas em declarações determinam o acesso de um usuário com base em declarações emitidas por uma autoridade central. Para obter mais informações sobre autorização baseada em declarações, confira a [documentação de autorização baseada em declarações](https://learn.microsoft.com/pt-br/aspnet/core/security/authorization/claims).

Em ASP.NET Core, ambas as estratégias são capturadas em um requisito de autorização. O serviço de autorização aproveita manipuladores de autorização para determinar se um usuário específico atende ou não aos requisitos de autorização aplicados a um recurso.

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/security?view=aspnetcore-8.0#enabling-authentication-in-minimal-apps)

## Habilitando a autenticação em aplicativos mínimos

Para habilitar a autenticação, chame [`AddAuthentication`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.extensions.dependencyinjection.authenticationservicecollectionextensions.addauthentication) para registrar os serviços de autenticação necessários no provedor de serviços do aplicativo.

C#

```
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddAuthentication();
var app = builder.Build();

app.MapGet("/", () => "Hello World!");
app.Run();
```

Normalmente, uma estratégia de autenticação específica é usada. No exemplo a seguir, o aplicativo é configurado com suporte para autenticação baseada em portador JWT. Este exemplo disponibiliza as APIs no pacote NuGet [`Microsoft.AspNetCore.Authentication.JwtBearer`](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.JwtBearer).

C#

```
var builder = WebApplication.CreateBuilder(args);
// Requires Microsoft.AspNetCore.Authentication.JwtBearer
builder.Services.AddAuthentication().AddJwtBearer();
var app = builder.Build();

app.MapGet("/", () => "Hello World!");
app.Run();
```

Por padrão, o [`WebApplication`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.webapplication) registra automaticamente os middlewares de autenticação e autorização se determinados serviços de autenticação e autorização estiverem habilitados. No exemplo a seguir, não é necessário invocar [`UseAuthentication`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.authappbuilderextensions.useauthentication) ou [`UseAuthorization`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.authorizationappbuilderextensions.useauthorization) para registrar os middlewares porque [`WebApplication`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.webapplication) isso faz isso automaticamente após `AddAuthentication` ou `AddAuthorization` serem chamados.

C#

```
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddAuthentication().AddJwtBearer();
builder.Services.AddAuthorization();
var app = builder.Build();

app.MapGet("/", () => "Hello World!");
app.Run();
```

Em alguns casos, como controlar a ordem do middleware, é necessário registrar explicitamente a autenticação e a autorização. No exemplo a seguir, o middleware de autenticação é executado _após_ a execução do middleware CORS. Para obter mais informações sobre middlewares e esse comportamento automático, consulte [Middleware em aplicativos de API mínimos](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/middleware).

C#

```
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddCors();
builder.Services.AddAuthentication().AddJwtBearer();
builder.Services.AddAuthorization();

var app = builder.Build();

app.UseCors();
app.UseAuthentication();
app.UseAuthorization();

app.MapGet("/", () => "Hello World!");
app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/security?view=aspnetcore-8.0#configuring-authentication-strategy)

### Configurando a estratégia de autenticação

As estratégias de autenticação normalmente dão suporte a diversas configurações carregadas por meio de opções. Opções mínimas de carregamento de suporte do aplicativo da configuração para as seguintes estratégias de autenticação:

- [Baseado em portador JWT](https://jwt.io/introduction)
- [Baseado em conexão OpenID](https://openid.net/developers/how-connect-works/)

A estrutura ASP.NET Core espera encontrar essas opções na seção `Authentication:Schemes:{SchemeName}` em [configuração](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/configuration). No exemplo a seguir, dois esquemas diferentes, `Bearer` e `LocalAuthIssuer`, são definidos com suas respectivas opções. A opção `Authentication:DefaultScheme` pode ser usada para configurar a estratégia de autenticação padrão usada.

JSON

```
{
  "Authentication": {
    "DefaultScheme":  "LocalAuthIssuer",
    "Schemes": {
      "Bearer": {
        "ValidAudiences": [
          "https://localhost:7259",
          "http://localhost:5259"
        ],
        "ValidIssuer": "dotnet-user-jwts"
      },
      "LocalAuthIssuer": {
        "ValidAudiences": [
          "https://localhost:7259",
          "http://localhost:5259"
        ],
        "ValidIssuer": "local-auth"
      }
    }
  }
}
```

No `Program.cs`, duas estratégias de autenticação baseadas em portador JWT são registradas, com:

- Nome do esquema "Portador".
- Nome do esquema "LocalAuthIssuer".

"Portador" é o esquema padrão típico em aplicativos habilitados com base no portador JWT, mas o esquema padrão pode ser substituído definindo a propriedade `DefaultScheme` como no exemplo anterior.

O nome do esquema é usado para identificar exclusivamente uma estratégia de autenticação e é usado como a chave de pesquisa ao resolver opções de autenticação da configuração, conforme mostrado no exemplo a seguir:

C#

```
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication()
  .AddJwtBearer()
  .AddJwtBearer("LocalAuthIssuer");
  
var app = builder.Build();

app.MapGet("/", () => "Hello World!");
app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/security?view=aspnetcore-8.0#configuring-authorization-policies-in-minimal-apps)

## Configurando políticas de autorização em aplicativos mínimos

A autenticação é usada para identificar e validar a identity dos usuários em relação a uma API. A autorização é usada para validar e verificar o acesso aos recursos em uma API e é facilitada pelo `IAuthorizationService` registrado pelo método de extensão `AddAuthorization`. No cenário a seguir, é adicionado um recurso `/hello` que exige que um usuário apresente uma `admin` declaração de função com uma `greetings_api` declaração de escopo.

A configuração de requisitos de autorização em um recurso é um processo de duas etapas que requer:

1. A configuração dos requisitos de autorização em uma política globalmente.
2. A aplicação de políticas individuais aos recursos.

No código a seguir, [AddAuthorizationBuilder](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.extensions.dependencyinjection.policyservicecollectionextensions.addauthorizationbuilder) é invocado, que:

- Adiciona serviços relacionados à autorização ao contêiner de DI.
- Retorna um [AuthorizationBuilder](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.authorization.authorizationbuilder) que pode ser usado para registrar diretamente as políticas de autenticação.

O código cria uma nova política de autorização, chamada `admin_greetings`, que encapsula dois requisitos de autorização:

- Um requisito baseado em função via [RequireRole](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.authorization.authorizationpolicybuilder.requirerole) para usuários com uma função `admin`.
- Um requisito baseado em declaração por meio de [RequireClaim](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.authorization.authorizationpolicybuilder.requireclaim) que o usuário deve fornecer uma `greetings_api` declaração de escopo.

A política `admin_greetings` é fornecida como uma política necessária para o ponto de extremidade `/hello`.

C#

```
using Microsoft.Identity.Web;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthorizationBuilder()
  .AddPolicy("admin_greetings", policy =>
        policy
            .RequireRole("admin")
            .RequireClaim("scope", "greetings_api"));

var app = builder.Build();

app.MapGet("/hello", () => "Hello world!")
  .RequireAuthorization("admin_greetings");

app.Run();
```

[](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/security?view=aspnetcore-8.0#use-dotnet-user-jwts-for-development-testing)

## Usar `dotnet user-jwts` para testes de desenvolvimento

Ao longo deste artigo, um aplicativo configurado com autenticação baseada em portador JWT é usado. A autenticação baseada em portador JWT exige que os clientes apresentem um token no cabeçalho da solicitação para validar a identity e as declarações. Normalmente, esses tokens são emitidos por uma autoridade central, como um servidor de identity.

Ao desenvolver no computador local, a ferramenta [`dotnet user-jwts`](https://learn.microsoft.com/pt-br/aspnet/core/security/authentication/jwt-authn) pode ser usada para criar tokens de portador.

CLI do .NET

```
dotnet user-jwts create
```

Observação

Quando invocada em um projeto, a ferramenta adiciona automaticamente as opções de autenticação correspondentes ao token gerado a `appsettings.json`.

Os tokens podem ser configurados com diversas personalizações. Por exemplo, para criar um token para a função `admin` e o escopo `greetings_api` esperados pela política de autorização no código anterior:

CLI do .NET

```
dotnet user-jwts create --scope "greetings_api" --role "admin"
```

Em seguida, o token gerado pode ser enviado como parte do cabeçalho na ferramenta de teste escolhida. Por exemplo, com curl:

CLI do .NET

```
curl -i -H "Authorization: Bearer {token}" https://localhost:{port}/hello
```

Para obter mais informações sobre a ferramenta `dotnet user-jwts`, leia a [documentação completa](https://learn.microsoft.com/pt-br/aspnet/core/security/authentication/jwt-authn).