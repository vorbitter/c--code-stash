O EF (Entity Framework) Core é uma versão leve, extensível, de [software livre](https://github.com/dotnet/efcore) e multiplataforma da popular tecnologia de acesso a dados do Entity Framework.

O EF Core pode servir como um O/RM (mapeador relacional de objeto), que:

- Permite que os desenvolvedores do .NET trabalhem com um banco de dados usando objetos .NET.
- Elimina a necessidade da maior parte do código de acesso a dados que normalmente precisa ser gravado.

O EF Core é compatível com vários mecanismos de banco de dados, consulte detalhes em [Provedores de Banco de Dados](https://learn.microsoft.com/pt-br/ef/core/providers/).

[](https://learn.microsoft.com/pt-br/ef/core/#the-model)

## O modelo

Com o EF Core, o acesso a dados é executado usando um modelo. Um modelo é feito de classes de entidade e um objeto de contexto que representa uma sessão com o banco de dados. O objeto de contexto permite consultar e salvar dados. Para saber mais, confira [Criação de um modelo](https://learn.microsoft.com/pt-br/ef/core/modeling/).

O EF dá suporte às seguintes abordagens de desenvolvimento de modelo:

- Gere um modelo de um banco de dados existente.
- Codifique um modelo para corresponder ao banco de dados.
- Depois de criar um modelo, use [migrações de EF](https://learn.microsoft.com/pt-br/ef/core/managing-schemas/migrations/) para criar um banco de dados a partir do modelo. As migrações permitem evoluir o banco de dados conforme o modelo é alterado.

C#

```
using System.Collections.Generic;
using Microsoft.EntityFrameworkCore;

namespace Intro;

public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True;ConnectRetryCount=0");
    }
}

public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }
    public int Rating { get; set; }
    public List<Post> Posts { get; set; }
}

public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }

    public int BlogId { get; set; }
    public Blog Blog { get; set; }
}
```

[](https://learn.microsoft.com/pt-br/ef/core/#querying)

## Consultas

Instâncias de suas classes de entidade são recuperadas do banco de dados usando a [LINQ (Consulta Integrada à Linguagem)](https://learn.microsoft.com/pt-br/dotnet/csharp/programming-guide/concepts/linq/). Para saber mais, confira [Consulta a dados](https://learn.microsoft.com/pt-br/ef/core/querying/).

C#

```
using (var db = new BloggingContext())
{
    var blogs = db.Blogs
        .Where(b => b.Rating > 3)
        .OrderBy(b => b.Url)
        .ToList();
}
```

[](https://learn.microsoft.com/pt-br/ef/core/#saving-data)

## Salvando dados

Dados são criados, excluídos e modificados no banco de dados usando as instâncias de suas classes de entidade. Consulte [Salvar dados](https://learn.microsoft.com/pt-br/ef/core/saving/) para saber mais.

C#

```
using (var db = new BloggingContext())
{
    var blog = new Blog { Url = "http://sample.com" };
    db.Blogs.Add(blog);
    db.SaveChanges();
}
```

[](https://learn.microsoft.com/pt-br/ef/core/#ef-orm-considerations)

## Considerações sobre EF O/RM

Embora o EF Core seja bom em abstrair muitos detalhes de programação, há algumas melhores práticas aplicáveis a qualquer O/RM que ajudam a evitar armadilhas comuns em aplicativos de produção:

- O conhecimento de nível intermediário ou superior do servidor de banco de dados subjacente é essencial para arquitetar, depurar, definir perfis e migrar dados em aplicativos de produção de alto desempenho. Por exemplo, conhecimento de chaves primárias e estrangeiras, restrições, índices, normalização, instruções DML e DDL, tipos de dados, criação de perfil etc.
- Teste funcional e de integração: é importante replicar o ambiente de produção o mais próximo possível para:
    - Encontrar problemas no aplicativo que só aparecem ao usar uma versão ou edição específica do servidor de banco de dados.
    - Capturar alterações interruptivas ao atualizar o EF Core e outras dependências. Por exemplo, adicionar ou atualizar estruturas como ASP.NET Core, OData ou AutoMapper. Essas dependências podem afetar o EF Core de maneiras inesperadas.
- Teste de desempenho e estresse com cargas representativas. O uso ingênuo de alguns recursos não ganha escala. Por exemplo, várias coleções Incluem uso intenso de carregamento lento, consultas condicionais em colunas não indexadas, atualizações e inserções massivas com valores gerados pelo repositório, falta de manipulação de simultaneidade, modelos grandes, política de cache inadequada.
- Revisão de segurança: por exemplo, tratamento de cadeias de conexão e de outros segredos, permissões de banco de dados para operação de não implantação, validação de entrada para SQL bruto, criptografia para dados confidenciais.
- Verifique se o registro em log e o diagnóstico são suficientes e utilizáveis. Por exemplo, configuração de registro em log apropriada, marcas de consulta e Application Insights.
- Recuperação de erro. Prepare contingências para cenários comuns de falha, como reversão de versão, servidores de fallback, expansão e balanceamento de carga, mitigação de DoS e backups de dados.
- implantação e migração de aplicativos. Planeje como as migrações serão aplicadas durante a implantação; fazer isso no início do aplicativo pode gerar problemas de simultaneidade e exigir permissões maiores do que as necessárias para a operação normal. Use o preparo para facilitar a recuperação de erros fatais durante a migração. Para saber mais, confira [Aplicar migrações](https://learn.microsoft.com/pt-br/ef/core/managing-schemas/migrations/applying).
- Exame e teste detalhados de migrações geradas. As migrações devem ser cuidadosamente testadas antes de serem aplicadas aos dados de produção. A forma do esquema e os tipos de coluna não podem ser facilmente alterados quando as tabelas contêm dados de produção. Por exemplo, no SQL Server, `nvarchar(max)` e `decimal(18, 2)` raramente são os melhores tipos para colunas mapeadas para propriedades decimais e de cadeia de caracteres, mas esses são os padrões que o EF usa porque não tem conhecimento do seu cenário específico.