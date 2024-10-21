O [`WebApplication`](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/webapplication?view=aspnetcore-8.0) adiciona automaticamente o seguinte middleware a [`Minimal API applications`](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/overview?view=aspnetcore-8.0), dependendo de determinadas condições:

- [`UseDeveloperExceptionPage`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.diagnostics.developerexceptionpagemiddleware) é adicionado primeiro quando o [`HostingEnvironment`](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/environments?view=aspnetcore-8.0) é `"Development"`.
- [`UseRouting`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.endpointroutingapplicationbuilderextensions.userouting) será adicionado em segundo se o código do usuário ainda não tiver chamado `UseRouting` e se houver pontos de extremidade configurados, por exemplo `app.MapGet`.
- [`UseEndpoints`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.endpointroutingapplicationbuilderextensions.useendpoints) será adicionado no final do pipeline de middleware se algum ponto de extremidade estiver configurado.
- [`UseAuthentication`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.authappbuilderextensions.useauthentication) será adicionado imediatamente após `UseRouting` se o código do usuário ainda não tiver chamado `UseAuthentication` e se [`IAuthenticationSchemeProvider`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.authentication.iauthenticationschemeprovider) puder ser detectado no provedor de serviços. `IAuthenticationSchemeProvider` é adicionado por padrão ao usar [`AddAuthentication`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.extensions.dependencyinjection.authenticationservicecollectionextensions.addauthentication), e os serviços são detectados usando [`IServiceProviderIsService`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.extensions.dependencyinjection.iserviceproviderisservice).
- [`UseAuthorization`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.authorizationappbuilderextensions.useauthorization) será adicionado em seguida se o código do usuário ainda não tiver chamado `UseAuthorization` e se [`IAuthorizationHandlerProvider`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationhandlerprovider) puder ser detectado no provedor de serviços. `IAuthorizationHandlerProvider` é adicionado por padrão ao usar [`AddAuthorization`](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.extensions.dependencyinjection.authenticationservicecollectionextensions.addauthentication), e os serviços são detectados usando `IServiceProviderIsService`.
- O middleware e os pontos de extremidade configurados pelo usuário são adicionados entre `UseRouting` e `UseEndpoints`.

O código a seguir é de fato aquilo que o middleware automático que está sendo adicionado ao aplicativo produz:

C#

```
if (isDevelopment)
{
    app.UseDeveloperExceptionPage();
}

app.UseRouting();

if (isAuthenticationConfigured)
{
    app.UseAuthentication();
}

if (isAuthorizationConfigured)
{
    app.UseAuthorization();
}

// user middleware/endpoints
app.CustomMiddleware(...);
app.MapGet("/", () => "hello world");
// end user middleware/endpoints

app.UseEndpoints(e => {});
```

Em alguns casos, a configuração de middleware padrão não é a correta para o aplicativo e requer modificação. Por exemplo, [UseCors](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.corsmiddlewareextensions.usecors) deve ser chamado antes de [UseAuthentication](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.authappbuilderextensions.useauthentication) e de [UseAuthorization](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.authorizationappbuilderextensions.useauthorization). O aplicativo precisa chamar `UseAuthentication` e `UseAuthorization` se `UseCors` for chamado:

C#

```
app.UseCors();
app.UseAuthentication();
app.UseAuthorization();
```

Se o middleware tiver que ser executado antes da correspondência de rotas ocorrer, [UseRouting](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.endpointroutingapplicationbuilderextensions.userouting) deverá ser chamado e o middleware deverá ser colocado antes da chamada para `UseRouting`. [UseEndpoints](https://learn.microsoft.com/pt-br/dotnet/api/microsoft.aspnetcore.builder.endpointroutingapplicationbuilderextensions.useendpoints) não é necessário nesse caso, já que é adicionado automaticamente conforme descrito anteriormente:

C#

```
app.Use((context, next) =>
{
    return next(context);
});

app.UseRouting();

// other middleware and endpoints
```

Ao adicionar um middleware de terminal:

- O middleware deve ser adicionado após `UseEndpoints`.
- O aplicativo precisa chamar `UseRouting` e `UseEndpoints` para que o middleware do terminal possa ser colocado no local correto.

C#

```
app.UseRouting();

app.MapGet("/", () => "hello world");

app.UseEndpoints(e => {});

app.Run(context =>
{
    context.Response.StatusCode = 404;
    return Task.CompletedTask;
});
```

O middleware de terminal é um middleware que é executado se nenhum ponto de extremidade lidar com a solicitação.

Para obter informações sobre middleware antifalsificação em APIs mínimas, consulte [Evitar ataques de falsificação de solicitação entre sites (XSRF/CSRF) no ASP.NET Core](https://learn.microsoft.com/pt-br/aspnet/core/security/anti-request-forgery?view=aspnetcore-8.0#afwma)

Para saber mais sobre middleware, confira [ASP.NET Core Middleware](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/middleware/?view=aspnetcore-8.0) e a [lista de middleware internos](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/middleware/?view=aspnetcore-8.0#built-in-middleware) que podem ser adicionados aos aplicativos.

Para obter mais informações sobre as APIs mínimas, confira [`Minimal APIs overview`](https://learn.microsoft.com/pt-br/aspnet/core/fundamentals/minimal-apis/overview?view=aspnetcore-8.0).