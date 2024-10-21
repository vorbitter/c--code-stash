
O provedor SQLite tem várias limitações de migrações. A maioria dessas limitações é resultado de limitações no mecanismo de banco de dados SQLite subjacente e não são específicas do EF.
## Limitações de modelagem

A biblioteca relacional comum (compartilhada por provedores de banco de dados relacionais do Entity Framework) define APIs para conceitos de modelagem que são comuns à maioria dos mecanismos de banco de dados relacionais. Alguns desses conceitos não são compatíveis com o provedor SQLite.
- Esquemas
- Sequências
## Limitações da consulta

O SQLite não dá suporte nativo aos seguintes tipos de dados. O EF Core pode ler e gravar valores desses tipos, e também há suporte para a consulta por igualdade (`where e.Property == value`). No entanto, outras operações, como comparação e ordenação, exigirão avaliação no cliente.

- DateTimeOffset
- Decimal
- TimeSpan
- UInt64

Em vez de `DateTimeOffset`, é recomendável usar valores DateTime. Ao manipular vários fusos horários, recomendamos converter os valores em UTC antes de salvar e converter novamente no fuso horário apropriado.

O tipo `Decimal` fornece um alto nível de precisão. No entanto, se você não precisar desse nível de precisão, recomendamos usar o duplo. Você pode usar um [conversor de valor](https://learn.microsoft.com/pt-br/ef/core/modeling/value-conversions) para continuar usando decimal em suas classes.

```c#
modelBuilder.Entity<MyEntity>()
    .Property(e => e.DecimalProperty)
    .HasConversion<double>();
```
## Limitações de migrações

O mecanismo de banco de dados SQLite não dá suporte a várias operações de esquema compatíveis com a maioria dos outros bancos de dados relacionais. Se você tentar aplicar uma das operações sem suporte a um banco de dados SQLite, uma `NotSupportedException` será lançada.

Uma recompilação será tentada para executar determinadas operações. Recompilações só são possíveis para artefatos de banco de dados que fazem parte do modelo do EF Core. Se um artefato de banco de dados não fizer parte do modelo, por exemplo, se ele tiver sido criado manualmente dentro de uma migração, um `NotSupportedException` ainda será lançado.

|Operação|Compatível?|
|---|---|
|AddCheckConstraint|✔ (rebuild)|
|AddColumn|✔|
|AddForeignKey|✔ (rebuild)|
|AddPrimaryKey|✔ (rebuild)|
|AddUniqueConstraint|✔ (rebuild)|
|AlterColumn|✔ (rebuild)|
|CreateIndex|✔|
|CreateTable|✔|
|DropCheckConstraint|✔ (rebuild)|
|DropColumn|✔ (rebuild)|
|DropForeignKey|✔ (rebuild)|
|DropIndex|✔|
|DropPrimaryKey|✔ (rebuild)|
|DropTable|✔|
|DropUniqueConstraint|✔ (rebuild)|
|RenameColumn|✔|
|RenameIndex|✔ (rebuild)|
|RenameTable|✔|
|EnsureSchema|✔ (no-op)|
|DropSchema|✔ (no-op)|
|Inserir|✔|
|Atualizar|✔|
|Delete|✔|
### Solução alternativa de limitações de migrações

Você pode solucionar algumas dessas limitações escrevendo manualmente o código em suas migrações para executar uma recompilação. As recompilações de tabela envolvem a criação de uma nova tabela, a cópia de dados para a nova tabela, a remoção da tabela antiga, a renomeação da nova tabela. Você precisará usar o método `Sql(string)` para executar algumas dessas etapas.

Consulte [Como fazer outros tipos de alterações de esquema de tabela](https://sqlite.org/lang_altertable.html#otheralter) na documentação do SQLite para obter mais detalhes.

### Limitações de script idempotente

Ao contrário de outros bancos de dados, o SQLite não inclui uma linguagem processual. Por isso, não há como gerar a lógica if-then exigida pelos scripts de migração idempotentes.

Se você souber a última migração aplicada a um banco de dados, poderá gerar um script dessa migração para a migração mais recente.

```basg
dotnet ef migrations script CurrentMigration
```

Caso contrário, recomendamos usar `dotnet ef database update` para aplicar migrações. Você pode especificar o arquivo de banco de dados ao executar o comando.

```bash
dotnet ef database update --connection "Data Source=My.db"
```