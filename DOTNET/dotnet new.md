## Nome

`dotnet new` – Cria um novo projeto, arquivo de configuração ou solução com base no modelo especificado.
## Sinopse

```bash
dotnet new <TEMPLATE> [--dry-run] [--force] [-lang|--language {"C#"|"F#"|VB}]
    [-n|--name <OUTPUT_NAME>] [-f|--framework <FRAMEWORK>] [--no-update-check]
    [-o|--output <OUTPUT_DIRECTORY>] [--project <PROJECT_PATH>]
    [-d|--diagnostics] [--verbosity <LEVEL>] [Template options]

dotnet new -h|--help
```
## Descrição

O comando `dotnet new` cria um projeto do .NET ou outros artefatos com base em um modelo.

O comando chama o [mecanismo de modelo](https://github.com/dotnet/templating) para criar os artefatos em disco com base no modelo e nas opções especificadas.

Observação

A partir do SDK do .NET 7, a sintaxe `dotnet new` foi alterada:

- As opções `--list`, `--search`, `--install` e `--uninstall` se tornaram os subcomandos `list`, `search`, `install` e `uninstall`.
- A opção `--update-apply` se tornou o subcomando `update`.
- Para usar `--update-check`, use o subcomando `update` com a opção `--check-only`.

Outras opções que estavam disponíveis antes ainda estão disponíveis para serem usadas com os respectivos subcomandos. A ajuda separada para cada subcomando está disponível por meio da ou opção `-h` ou `--help`: `dotnet new <subcommand> --help` lista todas as opções com suporte para o subcomando.

Além disso, o preenchimento com Tab agora está disponível para `dotnet new`. Ele oferece suporte ao preenchimento de nomes de modelo instalados e às opções fornecidas por um modelo selecionado. Para ativar o preenchimento com Tab no SDK do .NET, confira [Habilitar o preenchimento com Tab](https://learn.microsoft.com/pt-br/dotnet/core/tools/enable-tab-autocomplete).
### Preenchimento de guias

A partir do SDK do .NET 7.0.100, o preenchimento da guia está disponível para `dotnet new`. Ele dá suporte ao preenchimento de nomes de modelo instalados, bem como das opções que um modelo selecionado oferece. Para ativar o preenchimento com Tab no SDK do .NET, confira [Habilitar o preenchimento com Tab](https://learn.microsoft.com/pt-br/dotnet/core/tools/enable-tab-autocomplete).
### Restauração implícita

Não é necessário executar [`dotnet restore`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-restore), pois ele é executado implicitamente por todos os comandos que exigem uma restauração, como `dotnet new`, `dotnet build`, `dotnet run`, `dotnet test`, `dotnet publish` e `dotnet pack`. Para desabilitar a restauração implícita, use a opção `--no-restore`.

O comando `dotnet restore` ainda é útil em determinados cenários em que realizar uma restauração explícita faz sentido, como [compilações de integração contínua no Azure DevOps Services](https://learn.microsoft.com/pt-br/azure/devops/build-release/apps/aspnet/build-aspnet-core) ou em sistemas de compilação que precisam controlar explicitamente quando a restauração ocorrerá.

Para obter informações sobre como gerenciar feeds do NuGet, confira a [documentação do `dotnet restore`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-restore).

[](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new#arguments)

## Argumentos

- **`TEMPLATE`**
    
    O modelo para o qual criar uma instância quando o comando é invocado. Cada modelo pode ter opções específicas que podem ser passadas. Para obter mais informações, consulte [Opções de modelo](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new#template-options).
    
    Você pode executar [`dotnet new list`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-list) para ver uma lista de todos os modelos instalados.
    
    Começando com o SDK do .NET Core 3.0 e terminando com o SDK do .NET 5.0.300, a CLI pesquisa modelos em NuGet.org quando você invoca o comando `dotnet new` nas seguintes condições:
    
    - Se a CLI não conseguir encontrar uma correspondência de modelo, nem mesmo parcial, ao invocar `dotnet new`.
    - Se houver uma versão mais recente do modelo disponível. Nesse caso, o projeto ou o artefato é criado, mas a CLI avisa que há uma versão atualizada do modelo.
    
    A partir do SDK do .NET 5.0.300, o comando [`search`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-search) deve ser usado para pesquisar modelos no NuGet.org.
    

A tabela a seguir mostra os modelos que vêm pré-instalados com o SDK do .NET. O idioma padrão do modelo é mostrado entre parênteses. Clique no link de nome curto para ver as opções de modelo específicas.

|Modelos|Nome curto|Idioma|Marcas|Introduzida|
|---|---|---|---|---|
|Aplicativo do Console|[`console`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#console)|[C#], F#, VB|Comum/Console|1,0|
|Biblioteca de classes|[`classlib`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#classlib)|[C#], F#, VB|Comum/Library|1,0|
|Aplicativo WPF|[`wpf`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#wpf)|[C#], VB|Comum/WPF|3.0 (5.0 para VB)|
|Biblioteca de classes do WPF|[`wpflib`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#wpf)|[C#], VB|Comum/WPF|3.0 (5.0 para VB)|
|Biblioteca de Controles Personalizados do WPF|[`wpfcustomcontrollib`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#wpf)|[C#], VB|Comum/WPF|3.0 (5.0 para VB)|
|Biblioteca de controle de usuário WPF|[`wpfusercontrollib`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#wpf)|[C#], VB|Comum/WPF|3.0 (5.0 para VB)|
|Aplicativo do WinForms (Windows Forms)|[`winforms`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#winforms)|[C#], VB|Comum/WinForms|3.0 (5.0 para VB)|
|Biblioteca de classes do WinForms (Windows Forms)|[`winformslib`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#winforms)|[C#], VB|Comum/WinForms|3.0 (5.0 para VB)|
|Serviço de trabalho|[`worker`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#web-others)|[C#]|Comum/trabalho/Web|3,0|
|Projeto de Teste de Unidade|[`mstest`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#test)|[C#], F#, VB|Teste/MSTest|1,0|
|Projeto de Teste do NUnit 3|[`nunit`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#nunit)|[C#], F#, VB|Teste/NUnit|2.1.400|
|Item de Teste do NUnit 3|`nunit-test`|[C#], F#, VB|Teste/NUnit|2,2|
|Projeto de Teste xUnit|[`xunit`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#test)|[C#], F#, VB|Teste/xUnit|1,0|
|Componente do Razor|`razorcomponent`|[C#]|Web/ASP.NET|3,0|
|Página do Razor|[`page`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#page)|[C#]|Web/ASP.NET|2,0|
|Importações de Exibição do MVC|[`viewimports`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#namespace)|[C#]|Web/ASP.NET|2,0|
|MVC ViewStart|`viewstart`|[C#]|Web/ASP.NET|2,0|
|Aplicativo Web Blazor|[`blazor`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#blazor)|[C#]|Web/Blazor|8.0.100|
|BlazorWebAssembly Aplicativo autônomo|[`blazorwasm`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#blazorwasm)|[C#]|Web/Blazor/WebAssembly/PWA|3.1.300|
|ASP.NET Core Vazio|[`web`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#web)|[C#], F#|Web/Vazio|1,0|
|Aplicativo Web ASP.NET Core (Modelo-Exibição-Controlador)|[`mvc`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#web-options)|[C#], F#|Web/MVC|1,0|
|Aplicativo Web ASP.NET Core|[`webapp, razor`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#web-options)|[C#]|Web/MVC/Razor Pages|2.2, 2.0|
|Biblioteca de Classes do Razor|[`razorclasslib`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#razorclasslib)|[C#]|Web/Razor/Biblioteca/Biblioteca de Classes do Razor|2.1|
|API Web do ASP.NET Core|[`webapi`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#webapi)|[C#], F#|Web/Web API/API/Service/WebAPI|1.0|
|API do ASP.NET Core|[`webapiaot`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#webapiaot)|[C#]|Web/Web API/API/Service|8.0|
|Controlador de API do ASP.NET Core|[`apicontroller`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#apicontroller)|[C#]|Web/ASP.NET|8.0|
|Serviço gRPC do ASP.NET Core|[`grpc`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#web-others)|[C#]|Web/gRPC|3,0|
|Arquivo dotnet gitignore|`gitignore`||Config|3,0|
|Arquivo global.json|[`globaljson`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#globaljson)||Config|2,0|
|Configuração do NuGet|`nugetconfig`||Config|1,0|
|Arquivo de manifesto da ferramenta local do dotnet|`tool-manifest`||Config|3,0|
|Configuração da Web|`webconfig`||Config|1,0|
|Arquivo de Solução|`sln`||Solução|1,0|
|Arquivo de buffer de protocolo|[`proto`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#namespace)||Web/gRPC|3,0|
|Arquivo EditorConfig|[`editorconfig`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#editorconfig)||Config|6,0|

A tabela a seguir mostra modelos que foram descontinuados e não vêm mais pré-instalados com o SDK do .NET. Clique no link de nome curto para ver as opções de modelo específicas.

|Modelos|Nome curto|Idioma|Marcações|Descontinuado desde|
|---|---|---|---|---|
|ASP.NET Core com Angular|[`angular`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#spa)|[C#]|Web/MVC/SPA|8.0|
|ASP.NET Core com React.js|[`react`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#spa)|[C#]|Web/MVC/SPA|8.0|
|Aplicativo para servidores Blazor|[`blazorserver`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#blazorserver)|[C#]|Web/Blazor|8.0|
|Blazor Aplicativo de servidor vazio|[`blazorserver-empty`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#blazorserver)|[C#]|Web/Blazor|8.0|
|BlazorWebAssembly Aplicativo vazio|[`blazorwasm-empty`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates#blazorwasm)|[C#]|Web/Blazor/WebAssembly|8.0|

## Opções

- **`--dry-run`**
    
    Exibe um resumo do que ocorreria se o comando fornecido fosse executado se resultasse na criação de um modelo. Disponível desde o SDK do .NET Core 2.2.
    
- **`--force`**
    
    Força o conteúdo a ser gerado mesmo se ele alterasse os arquivos existentes. Isso é necessário quando o modelo escolhido substituiria os arquivos existentes no diretório de saída.
    
- **`-?|-h|--help`**
    
    Imprime uma ajuda para o comando. Ele pode ser invocado para o próprio comando `dotnet new` ou para qualquer modelo. Por exemplo, `dotnet new mvc --help`.
    
- **`-lang|--language {C#|F#|VB}`**
    
    A linguagem do modelo a ser criada. A linguagem aceita varia de acordo com o modelo (consulte os padrões na seção [Argumentos](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new#arguments)). Não é válida para alguns modelos.
    
    Observação
    
    Alguns shells interpretam `#` como um caractere especial. Nesses casos, coloque o valor do parâmetro de idioma entre aspas. Por exemplo, `dotnet new console -lang "F#"`.
    
- **`-n|--name <OUTPUT_NAME>`**
    
    O nome para a saída criada. Se nenhum nome for especificado, o nome do diretório atual será usado.
    
- **`-f|--framework <FRAMEWORK>`**
    
    Especifica a [estrutura de destino](https://learn.microsoft.com/pt-br/dotnet/standard/frameworks). Ele espera um TFM (moniker de estrutura de destino). Exemplos: "net6.0", "net7.0-macos". Esse valor será refletido no arquivo de projeto.
    
- **`-no-update-check`**
    
    Desabilita a verificação de atualizações de pacote de modelo quando uma instância de modelo é criada. Disponível desde o SDK .NET 6.0.100. Ao criar uma instância do modelo por meio de um pacote de modelo que foi instalado usando o `dotnet new --install`, o `dotnet new` verifica se há uma atualização para o modelo. Do .NET 6 em diante, não é feita nenhuma verificação de atualização para modelos padrão do .NET. Para atualizar modelos padrão do .NET, instale a versão de patch do SDK do .NET.
    
- **`-o|--output <OUTPUT_DIRECTORY>`**
    
    Local para colocar a saída gerada. O padrão é o diretório atual.
    
- **`--project <PROJECT_PATH>`**
    
    O projeto ao qual o modelo é adicionado. Esse projeto é usado para avaliação de contexto. Se não for especificado, o projeto nos diretórios atuais ou no pai será usado. Disponível desde o SDK .NET 7.0.100.
    
- **`-d|--diagnostics`**
    
    Habilita a saída de diagnóstico. Disponível desde o SDK .NET 7.0.100.
    
- **`-v|--verbosity <LEVEL>`**
    
    Define o nível de detalhes do comando. Os valores permitidos são `q[uiet]`, `m[inimal]`, `n[ormal]` e `diag[nostic]`. Disponível desde o SDK .NET 7.0.100.
    
## Opções de modelo

Cada modelo pode ter opções adicionais definidas. Para obter mais informações, confira [modelos padrão do .NET para `dotnet new`](https://learn.microsoft.com/pt-br/dotnet/core/tools/dotnet-new-sdk-templates).