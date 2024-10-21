## Obter o Entity Framework Core

O EF Core é enviado como [pacotes NuGet](https://www.nuget.org/). Para adicionar o EF Core a um aplicativo, instale o pacote do NuGet do provedor de banco de dados que você quer usar. Consulte [_Provedores_](https://learn.microsoft.com/pt-br/ef/core/providers/) para obter uma lista dos provedores de banco de dados disponíveis.

Para instalar ou atualizar os pacotes NuGet, use a CLI (interface de linha de comando) do .NET Core, a Caixa de Diálogo do Gerenciador de Pacotes do Visual Studio ou o Console do Gerenciador de Pacotes do Visual Studio.
## Obtenha as ferramentas do Entity Framework Core

É possível instalar ferramentas para realizar tarefas relacionadas com o EF Core no projeto, como criar e aplicar migrações de banco de dados ou criar um modelo do EF Core baseado em um banco de dados existente.

Dois conjuntos de ferramentas estão disponíveis:

- As [ferramentas da CLI (interface de linha de comando) do .NET Core](https://learn.microsoft.com/pt-br/ef/core/cli/dotnet) podem ser usadas no Windows, no Linux ou no macOS. Os comandos começam com `dotnet ef`.
- As [ferramentas do PMC (Console do Gerenciador de Pacotes)](https://learn.microsoft.com/pt-br/ef/core/cli/powershell) são executadas no Visual Studio no Windows. Esses comandos começam com um verbo, por exemplo `Add-Migration`, `Update-Database`.

### Obtenha as ferramentas da CLI do .NET Core

As ferramentas da CLI do .NET Core precisam do SDK do .NET Core mencionado anteriormente nos [Pré-requisitos](https://learn.microsoft.com/pt-br/ef/core/get-started/overview/install#prerequisites).

- `dotnet ef` deve ser instalado como uma ferramenta global ou local. A maioria dos desenvolvedores prefere instalar `dotnet ef` como uma ferramenta global usando o seguinte comando:
```bash
dotnet tool install --global dotnet-ef
```

`dotnet ef` também pode ser usado como uma ferramenta local. Você também pode obtê-lo como uma ferramenta local quando você restaura as dependências de um projeto que o declara como uma dependência de ferramentas usando um [arquivo de manifesto de ferramenta](https://learn.microsoft.com/pt-br/dotnet/core/tools/global-tools#install-a-local-tool).
- Para atualizar as ferramentas, use o comando `dotnet tool update`.
- Instale o pacote `Microsoft.EntityFrameworkCore.Design` mais recente.
```bash
dotnet add package Microsoft.EntityFrameworkCore.Design
```

Importante
>Sempre use a versão do pacote de ferramentas que corresponda à versão principal dos pacotes em runtime.
