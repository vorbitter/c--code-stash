Ao trabalhar com o Entity Framework Code First, o comportamento padrão é mapear suas classes POCO para tabelas usando um conjunto de convenções feitas no EF. Às vezes, no entanto, você não pode ou não deseja seguir essas convenções e precisa mapear entidades para algo diferente do que as convenções ditam.

Há duas maneiras principais de você configurar o EF para usar algo diferente das convenções, ou seja, [anotações](https://learn.microsoft.com/pt-br/ef/ef6/modeling/code-first/data-annotations) ou a API fluente do EF. As anotações abrangem apenas um subconjunto da funcionalidade fluente da API, portanto, há cenários de mapeamento que não podem ser alcançados usando anotações. Este artigo foi projetado para demonstrar como usar a API fluente para configurar propriedades.

A primeira API fluente do código é mais comumente acessada substituindo o método [OnModelCreating](https://msdn.microsoft.com/library/system.data.entity.dbcontext.onmodelcreating.aspx) em seu [DbContext](https://msdn.microsoft.com/library/system.data.entity.dbcontext.aspx) derivado. Os exemplos a seguir são projetados para mostrar como realizar várias tarefas com a API fluente e permitir que você copie o código e personalize-o para se adequar ao seu modelo, caso deseje ver o modelo com o qual eles podem ser usados no momento em que se encontram, ele será fornecido no final deste artigo.
## Configurações de todo o modelo

### Esquema padrão (EF6 em diante)

A partir do EF6, você pode usar o método HasDefaultSchema no DbModelBuilder para especificar o esquema de banco de dados a ser usado para todas as tabelas, procedimentos armazenados etc. Essa configuração padrão será substituída para todos os objetos para os quais você configura explicitamente um esquema diferente.

```c#
modelBuilder.HasDefaultSchema("sales");
```

### Convenções Personalizadas (EF6 em diante)

A partir do EF6, você pode criar suas próprias convenções para complementar as incluídas no Code First. Para obter mais detalhes, consulte [As Convenções Personalizadas de Code First](https://learn.microsoft.com/pt-br/ef/ef6/modeling/code-first/conventions/custom).
## Mapeamento de propriedade

O método [Property](https://msdn.microsoft.com/library/system.data.entity.infrastructure.dbentityentry.property.aspx) é usado para configurar atributos para cada propriedade que pertence a uma entidade ou tipo complexo. O método Property é usado para obter um objeto de configuração para uma determinada propriedade. As opções no objeto de configuração são específicas para o tipo que está sendo configurado; IsUnicode está disponível apenas em propriedades de cadeia de caracteres, por exemplo.
### Configurando uma chave primária

A convenção do Entity Framework para chaves primárias é:

1. Sua classe define uma propriedade cujo nome é "ID" ou "ID"
2. ou um nome de classe seguido por "ID" ou "ID"

Para definir explicitamente uma propriedade como uma chave primária, você pode usar o método HasKey. No exemplo a seguir, o método HasKey é usado para configurar a chave primária InstructorID no tipo OfficeAssignment.

```c#
modelBuilder.Entity<OfficeAssignment>().HasKey(t => t.InstructorID);
```

### Configurando uma chave primária composta

O exemplo a seguir configura as propriedades DepartmentID e Name como a chave primária composta do tipo Departamento.

```c#
modelBuilder.Entity<Department>().HasKey(t => new { t.DepartmentID, t.Name });
```
### Desativar identidade para chaves primárias numéricas

O exemplo a seguir define a propriedade DepartmentID como System.ComponentModel.DataAnnotations.DatabaseGeneratedOption.None para indicar que o valor não será gerado pelo banco de dados.

C#Copiar

```
modelBuilder.Entity<Department>().Property(t => t.DepartmentID)
    .HasDatabaseGeneratedOption(DatabaseGeneratedOption.None);
```

[](https://learn.microsoft.com/pt-br/ef/ef6/modeling/code-first/fluent/types-and-properties#specifying-the-maximum-length-on-a-property)

### Especificando o comprimento máximo em uma propriedade

No exemplo a seguir, a propriedade Name não deve ter mais de 50 caracteres. Se você tornar o valor maior que 50 caracteres, obterá uma exceção [DbEntityValidationException](https://msdn.microsoft.com/library/system.data.entity.validation.dbentityvalidationexception.aspx). Se o Code First criar um banco de dados com base nesse modelo, ele também definirá o comprimento máximo da coluna Name como 50 caracteres.

C#Copiar

```
modelBuilder.Entity<Department>().Property(t => t.Name).HasMaxLength(50);
```

[](https://learn.microsoft.com/pt-br/ef/ef6/modeling/code-first/fluent/types-and-properties#configuring-the-property-to-be-required)

### Configurando a propriedade a ser necessária

No exemplo a seguir, a propriedade Name é necessária. Se você não especificar o Nome, obterá uma exceção DbEntityValidationException. Se o Code First criar um banco de dados com base nesse modelo, a coluna usada para armazenar essa propriedade geralmente não será anulável.

 Observação

Em alguns casos, talvez não seja possível que a coluna no banco de dados seja não anulável, mesmo que a propriedade seja necessária. Por exemplo, ao usar dados de estratégia de herança TPH para vários tipos é armazenado em uma única tabela. Se um tipo derivado incluir uma propriedade necessária, a coluna não poderá ser tornada não anulável, pois nem todos os tipos na hierarquia terão essa propriedade.

C#Copiar

```
modelBuilder.Entity<Department>().Property(t => t.Name).IsRequired();
```

[](https://learn.microsoft.com/pt-br/ef/ef6/modeling/code-first/fluent/types-and-properties#configuring-an-index-on-one-or-more-properties)

### Configurando um índice em uma ou mais propriedades

 Observação

**Somente EF6.1 em diante** – o atributo Index foi introduzido no Entity Framework 6.1. Se você estiver usando uma versão anterior, as informações nesta seção não se aplicarão.

A criação de índices não é suportada nativamente pela API fluente, mas você pode usar o suporte para **IndexAttribute** por meio da API fluente. Os atributos de índice são processados incluindo uma anotação de modelo no modelo que, em seguida, é transformada em um Índice no banco de dados posteriormente no pipeline. Você pode adicionar manualmente essas mesmas anotações usando a API fluente.

A maneira mais fácil de fazer isso é criar uma instância de **IndexAttribute** que contenha todas as configurações para o novo índice. Em seguida, você pode criar uma instância de **IndexAnnotation** que é um tipo específico de EF que converterá as configurações de **IndexAttribute** em uma anotação de modelo que pode ser armazenada no modelo EF. Em seguida, eles podem ser passados para o método **HasColumnAnnotation** na API fluente, especificando o nome **Index** para a anotação.

C#Copiar

```
modelBuilder
    .Entity<Department>()
    .Property(t => t.Name)
    .HasColumnAnnotation("Index", new IndexAnnotation(new IndexAttribute()));
```

Para obter uma lista completa das configurações disponíveis no **IndexAttribute**, consulte a seção _Índice_ das [Anotações de Dados do Code First](https://learn.microsoft.com/pt-br/ef/ef6/modeling/code-first/data-annotations). Isso inclui personalizar o nome do índice, criar índices exclusivos e criar índices de várias colunas.

Você pode especificar várias anotações de índice em uma única propriedade passando uma matriz de **IndexAttribute** para o construtor de **IndexAnnotation**.

C#Copiar

```
modelBuilder
    .Entity<Department>()
    .Property(t => t.Name)
    .HasColumnAnnotation(
        "Index",  
        new IndexAnnotation(new[]
            {
                new IndexAttribute("Index1"),
                new IndexAttribute("Index2") { IsUnique = true }
            })));
```

[](https://learn.microsoft.com/pt-br/ef/ef6/modeling/code-first/fluent/types-and-properties#specifying-not-to-map-a-clr-property-to-a-column-in-the-database)

### Especificando não mapear uma propriedade CLR para uma coluna no banco de dados

O exemplo a seguir mostra como especificar que uma propriedade em um tipo CLR não é mapeada para uma coluna no banco de dados.

C#Copiar

```
modelBuilder.Entity<Department>().Ignore(t => t.Budget);
```

[](https://learn.microsoft.com/pt-br/ef/ef6/modeling/code-first/fluent/types-and-properties#mapping-a-clr-property-to-a-specific-column-in-the-database)

### Mapeando uma propriedade CLR para uma coluna específica no banco de dados

O exemplo a seguir mapeia a propriedade Nome CLR para a coluna de banco de dados DepartmentName.

C#Copiar

```
modelBuilder.Entity<Department>()
    .Property(t => t.Name)
    .HasColumnName("DepartmentName");
```

[](https://learn.microsoft.com/pt-br/ef/ef6/modeling/code-first/fluent/types-and-properties#renaming-a-foreign-key-that-is-not-defined-in-the-model)

### Renomeando uma chave estrangeira que não está definida no modelo

Se você optar por não definir uma chave estrangeira em um tipo CLR, mas quiser especificar qual nome ela deve ter no banco de dados, faça o seguinte:

C#Copiar

```
modelBuilder.Entity<Course>()
    .HasRequired(c => c.Department)
    .WithMany(t => t.Courses)
    .Map(m => m.MapKey("ChangedDepartmentID"));
```

[](https://learn.microsoft.com/pt-br/ef/ef6/modeling/code-first/fluent/types-and-properties#configuring-whether-a-string-property-supports-unicode-content)

### Configurando se uma propriedade string dá suporte a conteúdo Unicode

Por padrão, as cadeias de caracteres são Unicode (nvarchar no SQL Server). Você pode usar o método IsUnicode para especificar que uma cadeia de caracteres deve ser do tipo varchar.

C#Copiar

```
modelBuilder.Entity<Department>()
    .Property(t => t.Name)
    .IsUnicode(false);
```

[](https://learn.microsoft.com/pt-br/ef/ef6/modeling/code-first/fluent/types-and-properties#configuring-the-data-type-of-a-database-column)

### Configurando o tipo de dados de uma coluna de banco de dados

O método [HasColumnType](https://msdn.microsoft.com/library/system.data.entity.modelconfiguration.configuration.stringcolumnconfiguration.hascolumntype.aspx) permite o mapeamento para diferentes representações do mesmo tipo básico. O uso desse método não permite que você execute nenhuma conversão dos dados em tempo de execução. Observe que IsUnicode é a maneira preferencial de definir colunas como varchar, pois é independente de banco de dados.
```c#
modelBuilder.Entity<Department>()   
    .Property(p => p.Name)   
    .HasColumnType("varchar");
```
### Configurando propriedades em um tipo complexo

Há duas maneiras de configurar propriedades escalares em um tipo complexo.

Você pode chamar Propriedade em ComplexTypeConfiguration.

```c#
modelBuilder.ComplexType<Details>()
    .Property(t => t.Location)
    .HasMaxLength(20);
```

Você também pode usar a notação de ponto para acessar uma propriedade de um tipo complexo.

```c#
modelBuilder.Entity<OnsiteCourse>()
    .Property(t => t.Details.Location)
    .HasMaxLength(20);
```
### Configurando uma propriedade a ser usada como um token de simultaneidade otimista

Para especificar que uma propriedade em uma entidade representa um token de simultaneidade, você pode usar o atributo ConcurrencyCheck ou o método IsConcurrencyToken.
```c#
modelBuilder.Entity<OfficeAssignment>()
    .Property(t => t.Timestamp)
    .IsConcurrencyToken();
```

Você também pode usar o método IsRowVersion para configurar a propriedade para ser uma versão de linha no banco de dados. Definir a propriedade como uma versão de linha a configura automaticamente como um token de simultaneidade otimista.

```c#
modelBuilder.Entity<OfficeAssignment>()
    .Property(t => t.Timestamp)
    .IsRowVersion();
```

## Mapeamento de tipo

### Especificando que uma classe é um tipo complexo

Por convenção, um tipo que não tem nenhuma chave primária especificada é tratado como um tipo complexo. Há alguns cenários em que o Code First não detectará um tipo complexo (por exemplo, se você tiver uma propriedade chamada ID, mas não significa que ela seja uma chave primária). Nesses casos, você usaria a API fluente para especificar explicitamente que um tipo é um tipo complexo.

```c#
modelBuilder.ComplexType<Details>();
```

### Especificando não mapear um tipo de entidade CLR para uma tabela no banco de dados

O exemplo a seguir mostra como excluir um tipo CLR de ser mapeado para uma tabela no banco de dados.

```c#
modelBuilder.Ignore<OnlineCourse>();
```
### Mapeando um tipo de entidade para uma tabela específica no banco de dados

Todas as propriedades do Departamento serão mapeadas para colunas em uma tabela chamada Departamento de t_.

```c#
modelBuilder.Entity<Department>()  
    .ToTable("t_Department");
```

Você também pode especificar o nome do esquema como este:

```c#
modelBuilder.Entity<Department>()  
    .ToTable("t_Department", "school");
```
### Mapeando a herança TPH (tabela por hierarquia)

No cenário de mapeamento de TPH, todos os tipos em uma hierarquia de herança são mapeados para uma única tabela. Uma coluna discriminatória é usada para identificar o tipo de cada linha. Ao criar seu modelo com o Code First, o TPH é a estratégia padrão para os tipos que participam da hierarquia de herança. Por padrão, a coluna discriminatória é adicionada à tabela com o nome "Discriminador" e o nome do tipo CLR de cada tipo na hierarquia é usado para os valores discriminatórios. Você pode modificar o comportamento padrão usando a API fluente.

```c#
modelBuilder.Entity<Course>()  
    .Map<Course>(m => m.Requires("Type").HasValue("Course"))  
    .Map<OnsiteCourse>(m => m.Requires("Type").HasValue("OnsiteCourse"));
```

### Mapeando a herança TPT (tabela por tipo)

No cenário de mapeamento TPT, todos os tipos são mapeados para tabelas individuais. As propriedades que pertencem exclusivamente a um tipo base ou tipo derivado são armazenadas em uma tabela mapeada para esse tipo. As tabelas mapeadas para tipos derivados também armazenam uma chave estrangeira que une a tabela derivada à tabela base.

```c#
modelBuilder.Entity<Course>().ToTable("Course");  
modelBuilder.Entity<OnsiteCourse>().ToTable("OnsiteCourse");
```

### Mapeando a herança da classe TPC (Table-Per-Concrete Class)

No cenário de mapeamento de TPC, todos os tipos não abstratos na hierarquia são mapeados para tabelas individuais. As tabelas mapeadas para as classes derivadas não têm relação com a tabela mapeada para a classe base no banco de dados. Todas as propriedades de uma classe, incluindo propriedades herdadas, são mapeadas para colunas da tabela correspondente.

Chame o método MapInheritedProperties para configurar cada tipo derivado. MapInheritedProperties remapeia todas as propriedades que foram herdadas da classe base para novas colunas na tabela da classe derivada.

 Observação

Observe que, como as tabelas que participam da hierarquia de herança de TPC não compartilham uma chave primária, haverá chaves de entidade duplicadas ao inserir em tabelas mapeadas para subclasses se você tiver valores gerados pelo banco de dados com a mesma semente de identidade. Para resolver esse problema, você pode especificar um valor de semente inicial diferente para cada tabela ou desativar a identidade na propriedade de chave primária. Identidade é o valor padrão para propriedades de chave de inteiro ao trabalhar com o Code First.

```c$
modelBuilder.Entity<Course>()
    .Property(c => c.CourseID)
    .HasDatabaseGeneratedOption(DatabaseGeneratedOption.None);

modelBuilder.Entity<OnsiteCourse>().Map(m =>
{
    m.MapInheritedProperties();
    m.ToTable("OnsiteCourse");
});

modelBuilder.Entity<OnlineCourse>().Map(m =>
{
    m.MapInheritedProperties();
    m.ToTable("OnlineCourse");
});
```
### Mapeando propriedades de um tipo de entidade para várias tabelas no banco de dados (divisão de entidade)

A divisão de entidades permite que as propriedades de um tipo de entidade sejam distribuídas entre várias tabelas. No exemplo a seguir, a entidade Departamento é dividida em duas tabelas: Departamento e DepartmentDetails. A divisão de entidade usa várias chamadas para o método Map para mapear um subconjunto de propriedades para uma tabela específica.

```c#
modelBuilder.Entity<Department>()
    .Map(m =>
    {
        m.Properties(t => new { t.DepartmentID, t.Name });
        m.ToTable("Department");
    })
    .Map(m =>
    {
        m.Properties(t => new { t.DepartmentID, t.Administrator, t.StartDate, t.Budget });
        m.ToTable("DepartmentDetails");
    });
```
### Mapeando vários tipos de entidade para uma tabela no banco de dados (divisão de tabela)

O exemplo a seguir mapeia dois tipos de entidade que compartilham uma chave primária para uma tabela.

```c#
modelBuilder.Entity<OfficeAssignment>()
    .HasKey(t => t.InstructorID);

modelBuilder.Entity<Instructor>()
    .HasRequired(t => t.OfficeAssignment)
    .WithRequiredPrincipal(t => t.Instructor);

modelBuilder.Entity<Instructor>().ToTable("Instructor");

modelBuilder.Entity<OfficeAssignment>().ToTable("Instructor");
```

### Mapeando um tipo de entidade para inserir/atualizar/excluir procedimentos armazenados (EF6 em diante)

A partir do EF6, você pode mapear uma entidade para usar procedimentos armazenados para inserir atualização e exclusão. Para obter mais detalhes, consulte os [Procedimentos armazenados de inserção/atualização/exclusão de Code First](https://learn.microsoft.com/pt-br/ef/ef6/modeling/code-first/fluent/cud-stored-procedures).

## Modelo usado em exemplos

O modelo Code First a seguir é usado para os exemplos nesta página.

```c#
using System.Data.Entity;
using System.Data.Entity.ModelConfiguration.Conventions;
// add a reference to System.ComponentModel.DataAnnotations DLL
using System.ComponentModel.DataAnnotations;
using System.Collections.Generic;
using System;

public class SchoolEntities : DbContext
{
    public DbSet<Course> Courses { get; set; }
    public DbSet<Department> Departments { get; set; }
    public DbSet<Instructor> Instructors { get; set; }
    public DbSet<OfficeAssignment> OfficeAssignments { get; set; }

    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        // Configure Code First to ignore PluralizingTableName convention
        // If you keep this convention then the generated tables will have pluralized names.
        modelBuilder.Conventions.Remove<PluralizingTableNameConvention>();
    }
}

public class Department
{
    public Department()
    {
        this.Courses = new HashSet<Course>();
    }
    // Primary key
    public int DepartmentID { get; set; }
    public string Name { get; set; }
    public decimal Budget { get; set; }
    public System.DateTime StartDate { get; set; }
    public int? Administrator { get; set; }

    // Navigation property
    public virtual ICollection<Course> Courses { get; private set; }
}

public class Course
{
    public Course()
    {
        this.Instructors = new HashSet<Instructor>();
    }
    // Primary key
    public int CourseID { get; set; }

    public string Title { get; set; }
    public int Credits { get; set; }

    // Foreign key
    public int DepartmentID { get; set; }

    // Navigation properties
    public virtual Department Department { get; set; }
    public virtual ICollection<Instructor> Instructors { get; private set; }
}

public partial class OnlineCourse : Course
{
    public string URL { get; set; }
}

public partial class OnsiteCourse : Course
{
    public OnsiteCourse()
    {
        Details = new Details();
    }

    public Details Details { get; set; }
}

public class Details
{
    public System.DateTime Time { get; set; }
    public string Location { get; set; }
    public string Days { get; set; }
}

public class Instructor
{
    public Instructor()
    {
        this.Courses = new List<Course>();
    }

    // Primary key
    public int InstructorID { get; set; }
    public string LastName { get; set; }
    public string FirstName { get; set; }
    public System.DateTime HireDate { get; set; }

    // Navigation properties
    public virtual ICollection<Course> Courses { get; private set; }
}

public class OfficeAssignment
{
    // Specifying InstructorID as a primary
    [Key()]
    public Int32 InstructorID { get; set; }

    public string Location { get; set; }

    // When Entity Framework sees Timestamp attribute
    // it configures ConcurrencyCheck and DatabaseGeneratedPattern=Computed.
    [Timestamp]
    public Byte[] Timestamp { get; set; }

    // Navigation property
    public virtual Instructor Instructor { get; set; }
}
```