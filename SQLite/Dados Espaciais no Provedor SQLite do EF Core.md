Essa página inclui informações adicionais sobre como usar dados espaciais com o provedor de banco de dados SQLite. Para obter informações de caráter geral sobre como usar dados espaciais no EF Core, consulte a documentação principal relativa a [Dados Espaciais](https://learn.microsoft.com/pt-br/ef/core/modeling/spatial).

[](https://learn.microsoft.com/pt-br/ef/core/providers/sqlite/spatial#installing-spatialite)

## Como instalar o SpatiaLite

No Windows, a biblioteca nativa mod_spatialite é distribuída como uma dependência do pacote NuGet. Outras plataformas precisam instalá-la separadamente. De modo geral, isso é feito usando um gerenciador de pacotes de software. Por exemplo, você pode usar o APT no Debian e no Ubuntu; e o Homebrew no MacOS.

Bash

```
# Debian/Ubuntu
apt-get install libsqlite3-mod-spatialite

# macOS
brew install libspatialite
```

Infelizmente, as versões mais recentes do PROJ (uma dependência do SpatiaLite) são incompatíveis com o [pacote SQLitePCLRaw](https://learn.microsoft.com/pt-br/dotnet/standard/data/sqlite/custom-versions#bundles) padrão do EF. Você pode evitar essa questão usando, alternativamente, a biblioteca SQLite do sistema.

XML

```
<ItemGroup>
  <!-- Use bundle_sqlite3 instead with SpatiaLite on macOS and Linux -->
  <!--<PackageReference Include="Microsoft.EntityFrameworkCore.Sqlite" Version="3.1.0" />-->
  <PackageReference Include="Microsoft.EntityFrameworkCore.Sqlite.Core" Version="3.1.0" />
  <PackageReference Include="SQLitePCLRaw.bundle_sqlite3" Version="2.0.4" />

  <PackageReference Include="Microsoft.EntityFrameworkCore.Sqlite.NetTopologySuite" Version="3.1.0" />
</ItemGroup>
```

No **macOS**, você também precisará definir uma variável de ambiente para que ele use a versão do Homebrew do SQLite, antes de executar seu aplicativo. No Visual Studio para Mac, você pode definir isso em **Projeto > Opções do Projeto > Executar > Configurações > Padrão**

Bash

```
DYLD_LIBRARY_PATH=/usr/local/opt/sqlite/lib
```

[](https://learn.microsoft.com/pt-br/ef/core/providers/sqlite/spatial#configuring-srid)

## Como configurar o SRID

No SpatiaLite, as colunas precisam especificar um SRID por coluna. O SRID padrão é `0`. Especifique um SRID diferente usando o método HasSrid.

C#

```
modelBuilder.Entity<City>().Property(c => c.Location)
    .HasSrid(4326);
```

Observação

4326 se refere ao WGS 84, um padrão usado no GPS e em outros sistemas geográficos.

[](https://learn.microsoft.com/pt-br/ef/core/providers/sqlite/spatial#dimension)

## Dimensão

As dimensões (ou ordenadas) padrão de uma coluna são X e Y. Para habilitar coordenadas adicionais como Z ou M, configure o tipo de coluna.

C#

```
modelBuilder.Entity<City>().Property(c => c.Location)
    .HasColumnType("POINTZ");
```

[](https://learn.microsoft.com/pt-br/ef/core/providers/sqlite/spatial#spatial-function-mappings)

## Mapeamentos de função espacial

Essa tabela mostra quais membros do pacote [NetTopologySuite](https://nettopologysuite.github.io/NetTopologySuite/) (NTS) são convertidos em quais funções SQL.

|.NET|SQL|
|---|---|
|geometry.Area|Area(@geometry)|
|geometry.AsBinary()|AsBinary(@geometry)|
|geometry.AsText()|AsText(@geometry)|
|geometry.Boundary|Boundary(@geometry)|
|geometry.Buffer(distance)|Buffer(@geometry, @distance)|
|geometry.Buffer(distance, quadrantSegments)|Buffer(@geometry, @distance, @quadrantSegments)|
|geometry.Centroid|Centroid(@geometry)|
|geometry.Contains(g)|Contains(@geometry, @g)|
|geometry.ConvexHull()|ConvexHull(@geometry)|
|geometry.CoveredBy(g)|CoveredBy(@geometry, @g)|
|geometry.Covers(g)|Covers(@geometry, @g)|
|geometry.Crosses(g)|Crosses(@geometry, @g)|
|geometry.Difference(other)|Difference(@geometry, @other)|
|geometry.Dimension|Dimension(@geometry)|
|geometry.Disjoint(g)|Disjoint(@geometry, @g)|
|geometry.Distance(g)|Distance(@geometry, @g)|
|geometry.Envelope|Envelope(@geometry)|
|geometry.EqualsTopologically(g)|Equals(@geometry, @g)|
|geometry.GeometryType|GeometryType(@geometry)|
|geometry.GetGeometryN(n)|GeometryN(@geometry, @n + 1)|
|geometry.InteriorPoint|PointOnSurface(@geometry)|
|geometry.Intersection(other)|Intersection(@geometry, @other)|
|geometry.Intersects(g)|Intersects(@geometry, @g)|
|geometry.IsEmpty|IsEmpty(@geometry)|
|geometry.IsSimple|IsSimple(@geometry)|
|geometry.IsValid|IsValid(@geometry)|
|geometry.IsWithinDistance(geom, distance)|Distance(@geometry, @geom)<= @distance|
|geometry.Length|GLength(@geometry)|
|geometry.NumGeometries|NumGeometries(@geometry)|
|geometry.NumPoints|NumPoints(@geometry)|
|geometry.OgcGeometryType|CASE GeometryType(@geometry) WHEN 'POINT' THEN 1 ... END|
|geometry.Overlaps(g)|Overlaps(@geometry, @g)|
|geometry.PointOnSurface|PointOnSurface(@geometry)|
|geometry.Relate(g, intersectionPattern)|Relate(@geometry, @g, @intersectionPattern)|
|geometry.Reverse()|ST_Reverse(@geometry)|
|geometry.SRID|SRID(@geometry)|
|geometry.SymmetricDifference(other)|SymDifference(@geometry, @other)|
|geometry.ToBinary()|AsBinary(@geometry)|
|geometry.ToText()|AsText(@geometry)|
|geometry.Touches(g)|Touches(@geometry, @g)|
|geometry.Union()|UnaryUnion(@geometry)|
|geometry.Union(other)|GUnion(@geometry, @other)|
|geometry.Within(g)|Within(@geometry, @g)|
|geometryCollection[i]|GeometryN(@geometryCollection, @i + 1)|
|geometryCollection.Count|NumGeometries(@geometryCollection)|
|lineString.Count|NumPoints(@lineString)|
|lineString.EndPoint|EndPoint(@lineString)|
|lineString.GetPointN(n)|PointN(@lineString, @n + 1)|
|lineString.IsClosed|IsClosed(@lineString)|
|lineString.IsRing|IsRing(@lineString)|
|lineString.StartPoint|StartPoint(@lineString)|
|multiLineString.IsClosed|IsClosed(@multiLineString)|
|point.M|M(@point)|
|point.X|X(@point)|
|point.Y|Y(@point)|
|point.Z|Z(@point)|
|polygon.ExteriorRing|ExteriorRing(@polygon)|
|polygon.GetInteriorRingN(n)|InteriorRingN(@polygon, @n + 1)|
|polygon.NumInteriorRings|NumInteriorRing(@polygon)|

[](https://learn.microsoft.com/pt-br/ef/core/providers/sqlite/spatial#aggregate-functions)

### Funções de agregação

| .NET                                                              | SQL                           | Adicionado  |
| ----------------------------------------------------------------- | ----------------------------- | ----------- |
| GeometryCombiner.Combine(group.Select(x => x.Property))           | Collect(Property)             | EF Core 7.0 |
| ConvexHull.Create(group.Select(x => x.Property))                  | ConvexHull(Collect(Property)) | EF Core 7.0 |
| UnaryUnionOp.Union(group.Select(x => x.Property))                 | GUnion(Propriedade)           | EF Core 7.0 |
| EnvelopeCombiner.CombineAsGeometry(group.Select(x => x.Property)) | Extent(Property)              | EF Core 7.0 |