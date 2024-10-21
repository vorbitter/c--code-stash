## Estrutura de Destino

O EF Core 8 destina-se ao .NET 8. Aplicativos destinados a versões anteriores do .NET, .NET Core e .NET Framework precisarão ser atualizados para serem destinados ao .NET 8.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#summary)

## Resumo

|**Alterações da falha**|**Impacto**|
|---|---|
|[`Contains` em consultas LINQ podem parar de funcionar em versões mais antigas do SQL Server](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#sqlserver-contains-compatibility)|Alto|
|[Enumerações no JSON são armazenadas como ints em vez de cadeias de caracteres por padrão](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#enums-as-ints)|Alto|
|[SQL Server `date` e `time` agora scaffold para .NET `DateOnly` e `TimeOnly`](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#sqlserver-date-time-only)|Médio|
|[Colunas boolianas com um valor gerado pelo banco de dados não são mais agrupadas como anuláveis](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#scaffold-bools)|Médio|
|[métodos `Math` SQLite agora são convertidos em SQL](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#sqlite-math)|Baixo|
|[ITypeBase substitui IEntityType em algumas APIs](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#type-base)|Baixo|
|[As expressões ValueGenerator devem usar APIs públicas](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#value-converters)|Baixo|
|[ExcludeFromMigrations não exclui mais outras tabelas em uma hierarquia TPC](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#exclude-from-migrations)|Baixo|
|[Chaves inteiras sem sombra são mantidas em documentos do Cosmos](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#persist-to-cosmos)|Baixo|
|[O modelo relacional é gerado no modelo compilado](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#compiled-relational-model)|Baixo|
|[Fazer scaffolding pode gerar nomes de navegação diferentes](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#navigation-names)|Baixo|
|[Os discriminadores agora têm um comprimento máximo](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#discriminators)|Baixo|
|[Os valores de chave do SQL Server são comparados sem diferenciar maiúsculas de minúsculas](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#casekeys)|Baixo|

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#high-impact-changes)

## Alterações de alto impacto

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#contains-in-linq-queries-may-stop-working-on-older-sql-server-versions)

### `Contains` em consultas LINQ podem parar de funcionar em versões mais antigas do SQL Server

[Problema de acompanhamento nº 13617](https://github.com/dotnet/efcore/issues/13617)

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#old-behavior)

#### Comportamento antigo

Anteriormente, quando o operador `Contains` era usado em consultas LINQ com uma lista de valores parametrizados, o EF gerava SQL ineficiente, mas funcionava em todas as versões do SQL Server.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#new-behavior)

#### Novo comportamento

A partir do EF Core 8.0, o EF agora gera o SQL mais eficiente, mas não tem suporte no SQL Server 2014 e anterior.

Observe que versões mais recentes do SQL Server podem ser configuradas com um nível de compatibilidade [mais antigo](https://learn.microsoft.com/pt-br/sql/t-sql/statements/alter-database-transact-sql-compatibility-level), tornando-as incompatíveis com o novo SQL. Isso também pode ocorrer com um banco de dados SQL do Azure que foi migrado de uma instância anterior do SQL Server local, carregando o nível de compatibilidade antigo.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#why)

#### Por que

O SQL anterior gerado pelo EF Core para `Contains` inseriu os valores parametrizados como constantes no SQL. Por exemplo, a seguinte consulta LINQ:

c#

```
var names = new[] { "Blog1", "Blog2" };

var blogs = await context.Blogs
    .Where(b => names.Contains(b.Name))
    .ToArrayAsync();
```

...seria traduzido para o seguinte SQL:

SQL

```
SELECT [b].[Id], [b].[Name]
FROM [Blogs] AS [b]
WHERE [b].[Name] IN (N'Blog1', N'Blog2')
```

Essa inserção de valores constantes no SQL cria muitos problemas de desempenho, derrotando o cache do plano de consulta e causando remoções desnecessárias de outras consultas. A nova tradução do EF Core 8.0 usa a função [`OPENJSON`](https://learn.microsoft.com/pt-br/sql/t-sql/functions/openjson-transact-sql) do SQL Server para transferir os valores como uma matriz JSON. Isso resolve os problemas de desempenho inerentes à técnica anterior; no entanto, a função `OPENJSON` não está disponível no SQL Server 2014 e abaixo.

Para obter mais informações sobre essa alteração, [consulte esta postagem no blog.](https://devblogs.microsoft.com/dotnet/announcing-ef8-preview-4/)

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#mitigations)

#### Mitigações

Se o banco de dados for SQL Server 2016 (13.x) ou mais recente ou se você estiver usando o SQL do Azure, verifique o nível de compatibilidade configurado do banco de dados por meio do seguinte comando:

SQL

```
SELECT name, compatibility_level FROM sys.databases;
```

Se o nível de compatibilidade estiver abaixo de 130 (SQL Server 2016), considere modificá-lo para um valor mais recente ([documentação](https://learn.microsoft.com/pt-br/sql/t-sql/statements/alter-database-transact-sql-compatibility-level#best-practices-for-upgrading-database-compatibility-leve)).

Caso contrário, se a versão do banco de dados for realmente mais antiga que o SQL Server 2016 ou estiver definida como um nível de compatibilidade antigo que você não pode alterar por algum motivo, configure o EF Core para reverter para o SQL mais antigo e menos eficiente da seguinte maneira:

c#

```
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder
        .UseSqlServer(@"<CONNECTION STRING>", o => o.UseCompatibilityLevel(120));
```

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#enums-in-json-are-stored-as-ints-instead-of-strings-by-default)

### Enumerações no JSON são armazenadas como ints em vez de cadeias de caracteres por padrão

[Problema de acompanhamento nº 13617](https://github.com/dotnet/efcore/issues/31100)

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#old-behavior-1)

#### Comportamento antigo

No EF7, as [enumerações mapeadas para JSON](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-7.0/whatsnew#json-columns) são, por padrão, armazenados como valores de cadeia de caracteres no documento JSON.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#new-behavior-1)

#### Novo comportamento

A partir do EF Core 8.0, o EF agora, por padrão, mapeia enumerações para valores inteiros no documento JSON.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#why-1)

#### Por que

O EF sempre mapeou, por padrão, as enumerações para uma coluna numérica em bancos de dados relacionais. Como o EF oferece suporte a consultas em que valores de JSON interagem com valores de colunas e parâmetros, é importante que os valores em JSON correspondam aos valores na coluna não-JSON.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#mitigations-1)

#### Mitigações

Para continuar usando cadeias de caracteres, configure a propriedade de enumeração com uma conversão. Por exemplo:

C#

```
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<User>().Property(e => e.Status).HasConversion<string>();
}
```

Ou, para todas as propriedades do tipo enumerado::

C#

```
protected override void ConfigureConventions(ModelConfigurationBuilder configurationBuilder)
{
    configurationBuilder.Properties<StatusEnum>().HaveConversion<string>();
}
```

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#medium-impact-changes)

## Alterações de impacto médio

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#sql-server-date-and-time-now-scaffold-to-net-dateonly-and-timeonly)

### SQL Server `date` e `time` agora scaffold para .NET `DateOnly` e `TimeOnly`

[Problema de acompanhamento nº 24507](https://github.com/dotnet/efcore/issues/24507)

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#old-behavior-2)

#### Comportamento antigo

Anteriormente, ao estruturar um banco de dados do SQL Server com as colunas `date` ou `time`, o EF gerava propriedades de entidade com tipos [DateTime](https://learn.microsoft.com/pt-br/dotnet/api/system.datetime) e [TimeSpan](https://learn.microsoft.com/pt-br/dotnet/api/system.timespan).

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#new-behavior-2)

#### Novo comportamento

A partir do EF Core 8.0 e `date` e `time` são scaffolded como [DateOnly](https://learn.microsoft.com/pt-br/dotnet/api/system.dateonly) e [TimeOnly](https://learn.microsoft.com/pt-br/dotnet/api/system.timeonly).

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#why-2)

#### Por que

[DateOnly](https://learn.microsoft.com/pt-br/dotnet/api/system.dateonly) e [TimeOnly](https://learn.microsoft.com/pt-br/dotnet/api/system.timeonly) foram introduzidos no .NET 6.0 e são uma correspondência perfeita para mapear os tipos de data e hora do banco de dados. [DateTime](https://learn.microsoft.com/pt-br/dotnet/api/system.datetime) notavelmente contém um componente de tempo que não é usado e pode causar confusão ao mapeá-lo para `date`, e [TimeSpan](https://learn.microsoft.com/pt-br/dotnet/api/system.timespan) representa um intervalo de tempo, possivelmente de dias, em vez de uma hora do dia em que um evento ocorre. O uso dos novos tipos evita bugs e confusão e fornece clareza de intenção.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#mitigations-2)

#### Atenuações

Essa alteração afeta apenas os usuários que repetem o scaffolding no seu banco de dados regularmente em um modelo de código do EF (fluxo "database-first").

É recomendável reagir a essa alteração modificando seu código para usar os tipos [DateOnly](https://learn.microsoft.com/pt-br/dotnet/api/system.dateonly) e [TimeOnly](https://learn.microsoft.com/pt-br/dotnet/api/system.timeonly) cujo scaffold foi feito recentemente. No entanto, se isso não for possível, você poderá editar os modelos de scaffolding para reverter para o mapeamento anterior. Para fazer isso, configure os modelos conforme descrito [nessa página](https://learn.microsoft.com/pt-br/ef/core/managing-schemas/scaffolding/templates). Em seguida, edite o arquivo `EntityType.t4`, localize onde as propriedades da entidade são geradas (pesquise `property.ClrType`) e altere o código para o seguinte:

c#

```
        var clrType = property.GetColumnType() switch
        {
            "date" when property.ClrType == typeof(DateOnly) => typeof(DateTime),
            "date" when property.ClrType == typeof(DateOnly?) => typeof(DateTime?),
            "time" when property.ClrType == typeof(TimeOnly) => typeof(TimeSpan),
            "time" when property.ClrType == typeof(TimeOnly?) => typeof(TimeSpan?),
            _ => property.ClrType
        };

        usings.AddRange(code.GetRequiredUsings(clrType));

        var needsNullable = Options.UseNullableReferenceTypes && property.IsNullable && !clrType.IsValueType;
        var needsInitializer = Options.UseNullableReferenceTypes && !property.IsNullable && !clrType.IsValueType;
#>
    public <#= code.Reference(clrType) #><#= needsNullable ? "?" : "" #> <#= property.Name #> { get; set; }<#= needsInitializer ? " = null!;" : "" #>
<#
```

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#boolean-columns-with-a-database-generated-value-are-no-longer-scaffolded-as-nullable)

### Colunas boolianas com um valor gerado pelo banco de dados não são mais agrupadas como anuláveis

[Problema de acompanhamento nº 15070](https://github.com/dotnet/efcore/issues/15070)

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#old-behavior-3)

#### Comportamento antigo

Anteriormente, colunas `bool` não anuláveis com uma restrição padrão de banco de dados eram agrupadas como propriedades `bool?` anuláveis.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#new-behavior-3)

#### Novo comportamento

A partir do EF Core 8.0, colunas `bool` não anuláveis são sempre agrupadas como propriedades não anuláveis.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#why-3)

#### Por que

O valor de uma propriedade `bool` não será enviado para o banco de dados se esse valor for `false`, que é o padrão CLR. Se o banco de dados tiver um valor padrão de `true` para a coluna, mesmo que o valor da propriedade seja `false`, o valor no banco de dados acabará como `true`. No entanto, no EF8, a sentinela usada para determinar se uma propriedade tem um valor pode ser alterada. Isso é feito automaticamente para propriedades `bool` com um valor gerado por banco de dados de `true`, o que significa que não é mais necessário agrupar as propriedades como anuláveis.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#mitigations-3)

#### Atenuações

Essa alteração afeta apenas os usuários que repetem o scaffolding no seu banco de dados regularmente em um modelo de código do EF (fluxo "database-first").

É recomendável reagir a essa alteração modificando seu código para usar a propriedade bool não anulável. No entanto, se isso não for possível, você poderá editar os modelos de scaffolding para reverter para o mapeamento anterior. Para fazer isso, configure os modelos conforme descrito [nessa página](https://learn.microsoft.com/pt-br/ef/core/managing-schemas/scaffolding/templates). Em seguida, edite o arquivo `EntityType.t4`, localize onde as propriedades da entidade são geradas (pesquise `property.ClrType`) e altere o código para o seguinte:

c#

```
#>
        var propertyClrType = property.ClrType != typeof(bool)
                              || (property.GetDefaultValueSql() == null && property.GetDefaultValue() != null)
            ? property.ClrType
            : typeof(bool?);
#>
    public <#= code.Reference(propertyClrType) #><#= needsNullable ? "?" : "" #> <#= property.Name #> { get; set; }<#= needsInitializer ? " = null!;" : "" #>
<#
<#
```

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#low-impact-changes)

## Alterações de baixo impacto

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#sqlite-math-methods-now-translate-to-sql)

### métodos `Math`SQLite agora são convertidos em SQL

[Problema de acompanhamento nº 18843](https://github.com/dotnet/efcore/issues/18843)

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#old-behavior-4)

#### Comportamento antigo

Anteriormente, somente os métodos Abs, Max, Min e Round em `Math` eram convertidos em SQL. Todos os outros membros seriam avaliados no cliente se aparecessem na expressão Select final de uma consulta.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#new-behavior-4)

#### Novo comportamento

No EF Core 8.0, todos os métodos `Math` com funções matemáticas [SQLite correspondentes](https://sqlite.org/lang_mathfunc.html) são convertidos em SQL.

Essas funções matemáticas foram habilitadas na biblioteca nativa do SQLite que fornecemos por padrão (por meio de nossa dependência no pacote NuGet SQLitePCLRaw.bundle_e_sqlite3). Eles também foram habilitados na biblioteca fornecida pelo SQLitePCLRaw.bundle_e_sqlcipher. Se você estiver usando uma dessas bibliotecas, seu aplicativo não deverá ser afetado por essa alteração.

No entanto, há uma chance de que os aplicativos, incluindo a biblioteca nativa do SQLite, por outros meios, não habilitem as funções matemáticas. Nesses casos, os métodos `Math` serão convertidos em SQL e não encontrarão _tal função_ erros quando executados.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#why-4)

#### Por que

O SQLite adicionou funções matemáticas internas na versão 3.35.0. Mesmo que estejam desabilitados por padrão, eles se tornaram difundidos o suficiente para que decidimos fornecer traduções padrão para eles em nosso provedor EF Core SQLite.

Também colaboramos com Eric Sink no projeto SQLitePCLRaw para habilitar funções matemáticas em todas as bibliotecas nativas do SQLite fornecidas como parte desse projeto.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#mitigations-4)

#### Mitigações

A maneira mais simples de corrigir quebras é, quando possível, habilitar a função matemática é a biblioteca nativa do SQLite especificando a opção [SQLITE_ENABLE_MATH_FUNCTIONS](https://sqlite.org/compile.html#enable_math_functions) tempo de compilação.

Se você não controlar a compilação da biblioteca nativa, também poderá corrigir quebras criando as funções por conta própria no runtime usando as APIs [Microsoft.Data.Sqlite](https://learn.microsoft.com/pt-br/dotnet/standard/data/sqlite/user-defined-functions).

C#

```
sqliteConnection
    .CreateFunction<double, double, double>(
        "pow",
        Math.Pow,
        isDeterministic: true);
```

Como alternativa, você pode forçar a avaliação do cliente dividindo a expressão Select em duas partes separadas por `AsEnumerable`.

C#

```
// Before
var query = dbContext.Cylinders
    .Select(
        c => new
        {
            Id = c.Id
            // May throw "no such function: pow"
            Volume = Math.PI * Math.Pow(c.Radius, 2) * c.Height
        });

// After
var query = dbContext.Cylinders
    // Select the properties you'll need from the database
    .Select(
        c => new
        {
            c.Id,
            c.Radius,
            c.Height
        })
    // Switch to client-eval
    .AsEnumerable()
    // Select the final results
    .Select(
        c => new
        {
            Id = c.Id,
            Volume = Math.PI * Math.Pow(c.Radius, 2) * c.Height
        });
```

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#itypebase-replaces-ientitytype-in-some-apis)

### ITypeBase substitui IEntityType em algumas APIs

[Problema de acompanhamento nº 13947](https://github.com/dotnet/efcore/issues/13947)

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#old-behavior-5)

#### Comportamento antigo

Anteriormente, todos os tipos estruturais mapeados eram tipos de entidade.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#new-behavior-5)

#### Novo comportamento

Com a introdução de tipos complexos no EF8, algumas APIs que antes usavam um `IEntityType` agora usam `ITypeBase` para que as APIs possam ser usadas com tipos de entidade ou tipos complexos. Isso inclui:

- `IProperty.DeclaringEntityType` agora está obsoleto e `IProperty.DeclaringType` deve ser usado.
- `IEntityTypeIgnoredConvention` agora está obsoleto e `ITypeIgnoredConvention` deve ser usado.
- `IValueGeneratorSelector.Select` agora aceita um `ITypeBase` que pode ser, mas não precisa ser, um `IEntityType`.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#why-5)

#### Por que

Com a introdução de tipos complexos no EF8, essas APIs podem ser usadas com `IEntityType` ou `IComplexType`.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#mitigations-5)

#### Mitigações

As APIs antigas estão obsoletas, mas não serão removidas até o EF10. O código deve ser atualizado para usar as novas APIs o mais rápido possível.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#valueconverter-and-valuecomparer-expressions-must-use-public-apis-for-the-compiled-model)

### As expressões ValueConverter e ValueComparer devem usar APIs públicas para o modelo compilado

[Problema de acompanhamento nº 24896](https://github.com/dotnet/efcore/issues/24896)

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#old-behavior-6)

#### Comportamento antigo

Anteriormente, as definições `ValueConverter` e `ValueComparer` não eram incluídas no modelo compilado e, portanto, podiam conter código arbitrário.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#new-behavior-6)

#### Novo comportamento

O EF agora extrai as expressões dos objetos `ValueConverter` e `ValueComparer` e inclui essas C# no modelo compilado. Isso significa que essas expressões só devem usar API pública.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#why-6)

#### Por que

A equipe do EF está movendo gradualmente mais constructos para o modelo compilado para oferecer suporte ao uso do EF Core com AOT no futuro.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#mitigations-6)

#### Mitigações

Torne públicas as APIs usadas pelo comparador. Por exemplo, considere este conversor simples:

C#

```
public class MyValueConverter : ValueConverter<string, byte[]>
{
    public MyValueConverter()
        : base(v => ConvertToBytes(v), v => ConvertToString(v))
    {
    }

    private static string ConvertToString(byte[] bytes)
        => ""; // ... TODO: Conversion code

    private static byte[] ConvertToBytes(string chars)
        => Array.Empty<byte>(); // ... TODO: Conversion code
}
```

Para usar esse conversor em um modelo compilado com EF8, os métodos `ConvertToString` e `ConvertToBytes` devem ser tornados públicos. Por exemplo:

C#

```
public class MyValueConverter : ValueConverter<string, byte[]>
{
    public MyValueConverter()
        : base(v => ConvertToBytes(v), v => ConvertToString(v))
    {
    }

    public static string ConvertToString(byte[] bytes)
        => ""; // ... TODO: Conversion code

    public static byte[] ConvertToBytes(string chars)
        => Array.Empty<byte>(); // ... TODO: Conversion code
}
```

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#excludefrommigrations-no-longer-excludes-other-tables-in-a-tpc-hierarchy)

### ExcludeFromMigrations não exclui mais outras tabelas em uma hierarquia TPC

[Problema de acompanhamento nº 30079](https://github.com/dotnet/efcore/issues/30079)

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#old-behavior-7)

#### Comportamento antigo

Anteriormente, usar `ExcludeFromMigrations` em uma tabela em uma hierarquia TPC também excluiria outras tabelas na hierarquia.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#new-behavior-7)

#### Novo comportamento

A partir do EF Core 8.0, `ExcludeFromMigrations` não afeta outras tabelas.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#why-7)

#### Por que

O comportamento antigo era um bug e impedia que migrações fossem usadas para gerenciar hierarquias entre projetos.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#mitigations-7)

#### Mitigações

Use `ExcludeFromMigrations` explicitamente em qualquer outra tabela que deva ser excluída.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#non-shadow-integer-keys-are-persisted-to-cosmos-documents)

### Chaves inteiras sem sombra são mantidas em documentos do Cosmos

[Problema de acompanhamento nº 31664](https://github.com/dotnet/efcore/issues/31664)

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#old-behavior-8)

#### Comportamento antigo

Anteriormente, as propriedades inteiras sem sombra que correspondiam aos critérios para ser uma propriedade de chave sintetizada não eram mantidas no documento JSON, mas eram ressintetizadas na saída.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#new-behavior-8)

#### Novo comportamento

A partir do EF Core 8.0, essas propriedades agora são mantidas.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#why-8)

#### Por que

O comportamento antigo era um bug e impedia que propriedades que correspondessem aos critérios-chave sintetizados fossem mantidas no Cosmos.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#mitigations-8)

#### Mitigações

[Exclua a propriedade do modelo](https://learn.microsoft.com/pt-br/ef/core/modeling/entity-properties#included-and-excluded-properties) se seu valor não deve ser mantido. Além disso, você pode desabilitar totalmente esse comportamento definindo a opção `Microsoft.EntityFrameworkCore.Issue31664` do AppContext como `true`, consulte [AppContext para consumidores de biblioteca](https://learn.microsoft.com/pt-br/dotnet/api/system.appcontext#ForConsumers) para obter mais detalhes.

C#

```
AppContext.SetSwitch("Microsoft.EntityFrameworkCore.Issue31664", isEnabled: true);
```

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#relational-model-is-generated-in-the-compiled-model)

### O modelo relacional é gerado no modelo compilado

[Problema de acompanhamento nº 24896](https://github.com/dotnet/efcore/issues/24896)

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#old-behavior-9)

#### Comportamento antigo

Anteriormente, o modelo relacional era computado em tempo de execução mesmo ao usar um modelo compilado.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#new-behavior-9)

#### Novo comportamento

A partir do EF Core 8.0, o modelo relacional faz parte do modelo compilado gerado. No entanto, para modelos particularmente grandes, o arquivo gerado pode falhar ao compilar.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#why-9)

#### Por que

Isso foi feito para melhorar ainda mais o tempo de inicialização.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#mitigations-9)

#### Mitigações

Edite o arquivo `*ModelBuilder.cs` gerado e remova a linha `AddRuntimeAnnotation("Relational:RelationalModel", CreateRelationalModel());`, bem como o método `CreateRelationalModel()`.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#scaffolding-may-generate-different-navigation-names)

### Fazer scaffolding pode gerar nomes de navegação diferentes

[Problema de acompanhamento nº 27832](https://github.com/dotnet/efcore/issues/27832)

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#old-behavior-10)

#### Comportamento antigo

Anteriormente, ao fazer scaffolding de `DbContext` e tipos de entidade e de um banco de dados existente, os nomes de navegação de relações às vezes eram derivados de um prefixo comum de vários nomes de coluna de chave estrangeira.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#new-behavior-10)

#### Novo comportamento

A partir do EF Core 8.0, prefixos comuns de nomes de coluna de uma chave estrangeira composta não são mais usados para gerar nomes de navegação.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#why-10)

#### Por que

Esta é uma regra de nomenclatura obscura que, às vezes, gera nomes muito ruins, como, `S`, `Student_` ou até mesmo apenas `_`. Sem esta regra, nomes estranhos não são mais gerados e as convenções de nomenclatura para navegação também são mais simples, facilitando a compreensão e a previsão de quais nomes serão gerados.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#mitigations-10)

#### Mitigações

O [EF Core Power Tools](https://github.com/ErikEJ/EFCorePowerTools/issues/2143) tem uma opção para continuar gerando navegação da maneira antiga. Como alternativa, o código gerado pode ser totalmente personalizado usando [modelos T4](https://learn.microsoft.com/pt-br/ef/core/managing-schemas/scaffolding/templates). Isso pode ser usado para exemplificar as propriedades de chave estrangeira de relações de scaffolding e usar qualquer regra apropriada para o código para gerar os nomes de navegação necessários.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#discriminators-now-have-a-max-length)

### Os valores discriminatórios agora têm um comprimento máximo

[Problema de acompanhamento nº 10691](https://github.com/dotnet/efcore/issues/10691)

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#old-behavior-11)

#### Comportamento antigo

Anteriormente, colunas discriminatórias criadas para [mapeamento de herança de TPH](https://learn.microsoft.com/pt-br/ef/core/modeling/inheritance) eram configuradas como `nvarchar(max)` no SQL Server/SQL do Azure ou o tipo de cadeia de caracteres não associada equivalente em outros bancos de dados.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#new-behavior-11)

#### Novo comportamento

A partir do EF Core 8.0, as colunas discriminatórias são criadas com um comprimento máximo que abrange todos os valores discriminatórios conhecidos. O EF gerará uma migração para fazer essa alteração. No entanto, se a coluna discriminatória for restrita de alguma forma (por exemplo, como parte de um índice), então o `AlterColumn` criado pelas Migrações poderá falhar.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#why-11)

#### Por que

As colunas `nvarchar(max)` são ineficientes e desnecessárias quando os comprimentos de todos os valores possíveis são conhecidos.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#mitigations-11)

#### Mitigações

O tamanho da coluna pode ser explicitamente não associado:


```c#
modelBuilder.Entity<Foo>()
    .Property<string>("Discriminator")
    .HasMaxLength(-1);
```

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#sql-server-key-values-are-compared-case-insensitively)

### Os valores de chave do SQL Server são comparados sem diferenciar maiúsculas de minúsculas

[Problema de acompanhamento nº 27526](https://github.com/dotnet/efcore/issues/27526)

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#old-behavior-12)

#### Comportamento antigo

Anteriormente, ao rastrear entidades com chaves de cadeia de caracteres com os provedores de banco de dados SQL Server/SQL do Azure, os valores de chave eram comparados usando o comparador ordinal que diferencia maiúsculas de minúsculas do .NET padrão.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#new-behavior-12)

#### Novo comportamento

A partir do EF Core 8.0, os valores de chave de cadeia de caracteres do SQL Server/SQL do Azure são comparados usando o comparador ordinal que não diferencia maiúsculas de minúsculas do .NET padrão.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#why-12)

#### Por que

Por padrão, o SQL Server usa comparações que não diferenciam maiúsculas de minúsculas ao comparar valores de chave estrangeira para correspondências com valores de chave principal. Isso significa que, quando o EF usa comparações que diferenciam maiúsculas de minúsculas, ele pode não conectar uma chave estrangeira a uma chave principal quando deveria.

[](https://learn.microsoft.com/pt-br/ef/core/what-is-new/ef-core-8.0/breaking-changes#mitigations-12)

#### Mitigações

Comparações que diferenciam maiúsculas de minúsculas podem ser usadas definindo um `ValueComparer` personalizado. Por exemplo:

```c#
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    var comparer = new ValueComparer<string>(
        (l, r) => string.Equals(l, r, StringComparison.Ordinal),
        v => v.GetHashCode(),
        v => v);

    modelBuilder.Entity<Blog>()
        .Property(e => e.Id)
        .Metadata.SetValueComparer(comparer);

    modelBuilder.Entity<Post>(
        b =>
        {
            b.Property(e => e.Id).Metadata.SetValueComparer(comparer);
            b.Property(e => e.BlogId).Metadata.SetValueComparer(comparer);
        });
}
```