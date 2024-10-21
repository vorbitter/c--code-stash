Ao trabalhar com Code First, você define seu modelo definindo suas classes CLR de domínio. Por padrão, o Entity Framework usa as convenções Code First para mapear suas classes para o esquema de banco de dados. Se você usar as convenções de nomenclatura Code First, na maioria dos casos, poderá confiar no Code First para configurar relações entre suas tabelas com base nas chaves estrangeiras e nas propriedades de navegação definidas nas classes. Se você não seguir as convenções ao definir suas classes, ou se quiser alterar a maneira como as convenções funcionam, poderá usar a API fluente ou anotações de dados para configurar suas classes para que o Code First possa mapear as relações entre suas tabelas.
## Introdução

Ao configurar um relacionamento com a API fluent, você começa com a instância EntityTypeConfiguration e, em seguida, usa o método HasRequired, HasOptional ou HasMany para especificar o tipo de relacionamento do qual essa entidade participa. Os métodos HasRequired e HasOptional usam uma expressão lambda que representa uma propriedade de navegação de referência. O método HasMany usa uma expressão lambda que representa uma propriedade de navegação de coleção. Em seguida, você pode configurar uma propriedade de navegação inversa usando os métodos WithRequired, WithOptional e WithMany. Esses métodos têm sobrecargas que não usam argumentos e podem ser usados para especificar cardinalidade com navegações unidirecionais.

Em seguida, você pode configurar propriedades de chave estrangeira usando o método HasForeignKey. Esse método usa uma expressão lambda que representa a propriedade a ser usada como a chave estrangeira.

## Configurando uma relação obrigatória para opcional (um-para-zero-ou-um)

O exemplo a seguir configura uma relação um-para-zero-ou-um. O OfficeAssignment tem a propriedade InstructorID que é uma chave primária e uma chave estrangeira, porque o nome da propriedade não segue a convenção o método HasKey é usado para configurar a chave primária.

```c#
// Configure the primary key for the OfficeAssignment
modelBuilder.Entity<OfficeAssignment>()
    .HasKey(t => t.InstructorID);

// Map one-to-zero or one relationship
modelBuilder.Entity<OfficeAssignment>()
    .HasRequired(t => t.Instructor)
    .WithOptional(t => t.OfficeAssignment);
```
## Configurando um relacionamento em que ambas as extremidades são necessárias (um-para-um)

Na maioria dos casos, o Entity Framework pode inferir qual tipo é o dependente e qual é o principal em um relacionamento. No entanto, quando ambas as extremidades do relacionamento são necessárias ou ambos os lados são opcionais, o Entity Framework não pode identificar o dependente e o principal. Quando ambas as extremidades da relação forem necessárias, use WithRequiredPrincipal ou WithRequiredDependent após o método HasRequired. Quando ambas as extremidades da relação forem opcionais, use WithOptionalPrincipal ou WithOptionalDependent após o método HasOptional.

```c#
// Configure the primary key for the OfficeAssignment
modelBuilder.Entity<OfficeAssignment>()
    .HasKey(t => t.InstructorID);

modelBuilder.Entity<Instructor>()
    .HasRequired(t => t.OfficeAssignment)
    .WithRequiredPrincipal(t => t.Instructor);
```
## Configurando uma relação muitos-para-muitos

O código a seguir configura uma relação muitos-para-muitos entre os tipos Curso e Instrutor. No exemplo a seguir, as convenções Code First padrão são usadas para criar uma tabela de junção. Como resultado, a tabela CourseInstructor é criada com colunas Course_CourseID e Instructor_InstructorID.

```c#
modelBuilder.Entity<Course>()
    .HasMany(t => t.Instructors)
    .WithMany(t => t.Courses)
```

Se você quiser especificar o nome da tabela de associação e os nomes das colunas na tabela, será necessário fazer configurações adicionais usando o método Map. O código a seguir gera a tabela CourseInstructor com as colunas CourseID e InstructorID.

```c#
modelBuilder.Entity<Course>()
    .HasMany(t => t.Instructors)
    .WithMany(t => t.Courses)
    .Map(m =>
    {
        m.ToTable("CourseInstructor");
        m.MapLeftKey("CourseID");
        m.MapRightKey("InstructorID");
    });
```
## Configurando um relacionamento com uma propriedade de navegação

Uma relação unidirecional (também chamada de unidirecional) é quando uma propriedade de navegação é definida em apenas uma das extremidades da relação e não em ambas. Por convenção, o Code First sempre interpreta uma relação unidirecional como um-para-muitos. Por exemplo, se você quiser uma relação um-para-um entre o Instrutor e o OfficeAssignment, em que você tenha uma propriedade de navegação somente no tipo Instrutor, precisará usar a API fluente para configurar essa relação.

```c#
// Configure the primary Key for the OfficeAssignment
modelBuilder.Entity<OfficeAssignment>()
    .HasKey(t => t.InstructorID);

modelBuilder.Entity<Instructor>()
    .HasRequired(t => t.OfficeAssignment)
    .WithRequiredPrincipal();
```
## Habilitando a exclusão em cascata

Você pode configurar a exclusão em cascata em um relacionamento usando o método WillCascadeOnDelete. Se uma chave estrangeira na entidade dependente não for anulável, o Code First definirá a exclusão em cascata no relacionamento. Se uma chave estrangeira na entidade dependente for anulável, o Code First não definirá a exclusão em cascata no relacionamento e, quando a entidade de segurança for excluída, a chave estrangeira será definida como nula.

Você pode remover essas convenções de exclusão em cascata usando:

`modelBuilder.Conventions.Remove<OneToManyCascadeDeleteConvention>()`  
`modelBuilder.Conventions.Remove<ManyToManyCascadeDeleteConvention>()`

O código a seguir configura a relação a ser necessária e, em seguida, desabilita a exclusão em cascata.

```c#
modelBuilder.Entity<Course>()
    .HasRequired(t => t.Department)
    .WithMany(t => t.Courses)
    .HasForeignKey(d => d.DepartmentID)
    .WillCascadeOnDelete(false);
```
## Configurando uma chave estrangeira composta

Se a chave primária no tipo Departamento consistisse em propriedades DepartmentID e Name, você configuraria a chave primária para o Departamento e a chave estrangeira nos tipos De curso da seguinte maneira:

```c#
// Composite primary key
modelBuilder.Entity<Department>()
.HasKey(d => new { d.DepartmentID, d.Name });

// Composite foreign key
modelBuilder.Entity<Course>()  
    .HasRequired(c => c.Department)  
    .WithMany(d => d.Courses)
    .HasForeignKey(d => new { d.DepartmentID, d.DepartmentName });
```
## Renomeando uma chave estrangeira que não está definida no modelo

Se você optar por não definir uma chave estrangeira no tipo CLR, mas quiser especificar qual nome ela deve ter no banco de dados, faça o seguinte:

```c#
modelBuilder.Entity<Course>()
    .HasRequired(c => c.Department)
    .WithMany(t => t.Courses)
    .Map(m => m.MapKey("ChangedDepartmentID"));
```
## Configurando um nome de chave estrangeira que não segue a convenção Code First

Se a propriedade de chave estrangeira na classe Course fosse chamada SomeDepartmentID em vez de DepartmentID, você precisaria fazer o seguinte para especificar que deseja que SomeDepartmentID seja a chave estrangeira:

```c#
modelBuilder.Entity<Course>()
         .HasRequired(c => c.Department)
         .WithMany(d => d.Courses)
         .HasForeignKey(c => c.SomeDepartmentID);
```
## Modelo usado em amostras

O modelo Code First a seguir é usado para os exemplos nessa página.

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
