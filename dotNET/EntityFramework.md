# EntityFramework 란

EntityFramework는 .NET 진영의 ORM(Object Relational Mapper)이다.

현재 최신버전은 EntityFramework Core 6이다.

## 지원하는 데이터베이스

지원하는 데이테베이스는 Sql Server, Sqlite, InMemory, Azure Cosmos DB, PostgreSQL, MySQL 등등 굉장히 다양하다.

자세한 정보는 다음 링크에서 확인. [지원하는 데이터베이스](https://docs.microsoft.com/ko-kr/ef/core/providers/?tabs=dotnet-core-cli)

# 모델 만들기

EntityFramework(EF)는 기본적으로 Code First를 지향한다.

## 엔터티 형식

### 테이블명

테이블명은 기본적으로 DbSet 프로퍼티의 이름으로 자동으로 만들어진다.

명시적으로 테이블명을 작성하고 싶다면 Attribute 또는 Fluent API를 이용하면 된다.

```csharp
// Attribute 방식
[Table("BlogMaster")]
public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }
}

// Fluent API 방식
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
        .ToTable("BlogMaster");
}
```

### 스키마

Sql Server의 경우 기본 스키마는 `dbo`이다.

별도의 다른 스키마를 지정하고 싶은 경우 다음과 같이 하면 된다.

```csharp
// Attribute
[Table("blogs", Schema = "blogging")]
public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }
}

// Fluent API
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
        .ToTable("blogs", schema: "blogging");
}
```

각 테이블마다 설정하는 것이 아니라 전체의 기본 스키마를 변경하고 싶다면 다음과 같이 하면 된다.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.HasDefaultSchema("blogging");
}
```

### 모델에서 제외

특정 모델을 데이터 테이블로 변환하지 않으려면 다음과 같이 하면 된다.

```csharp
// Attribute
[NotMapped]
public class BlogMetadata
{
    public DateTime LoadedFromDatabase { get; set; }
}

// Fluent API
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Ignore<BlogMetadata>();
}
```

## 엔터티 속성

### 속성 제외

```csharp
// Attribute
public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }

    [NotMapped]
    public DateTime LoadedFromDatabase { get; set; }
}

// Fluent API
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
        .Ignore(b => b.LoadedFromDatabase);
}
```

### 컬럼명

```csharp
public class Blog
{
    [Column("blog_id")]
    public int BlogId { get; set; }

    public string Url { get; set; }
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
        .Property(b => b.BlogId)
        .HasColumnName("blog_id");
}
```

### 컬럼 데이터 타입

```csharp
public class Blog
{
    public int BlogId { get; set; }

    [Column(TypeName = "varchar(200)")]
    public string Url { get; set; }

    [Column(TypeName = "decimal(5, 2)")]
    public decimal Rating { get; set; }
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>(
        eb =>
        {
            eb.Property(b => b.Url).HasColumnType("varchar(200)");
            eb.Property(b => b.Rating).HasColumnType("decimal(5, 2)");
        });
}
```

### Nullable

기본적으로 Nullable로 설정되며 필수값인 경우 다음과 같이 하면 된다.

```csharp
public class Blog
{
    public int BlogId { get; set; }

    [Required]
    public string Url { get; set; }
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
        .Property(b => b.Url)
        .IsRequired();
}
```

추가적으로 C# 8.0부터 추가된 NRT라는 것을 이용하면 다음과 같이 할 수 있다.

```csharp
public class Customer
{
    public int Id { get; set; }
    public string FirstName { get; set; } // Required by convention
    public string LastName { get; set; } // Required by convention
    public string? MiddleName { get; set; } // Optional by convention

    // Note the following use of constructor binding, which avoids compiled warnings
    // for uninitialized non-nullable properties.
    public Customer(string firstName, string lastName, string? middleName = null)
    {
        FirstName = firstName;
        LastName = lastName;
        MiddleName = middleName;
    }
}
```

## 키

### 기본키 구성

프로퍼티명을 `Id` 또는 `<Entity>Id` 식으로 지정하면 EF가 자동으로 기본키로 설정한다.

또는 `[Key]` Attribute을 사용하면 된다.

SQL Server에서 숫자형 기본키는 자동으로 `Identity` 열로 설정된다.

```csharp
public class Food1
{
    public int Id { get; set; }
    public string FoodName { get; set; }
}

public class Food2
{
    public int FoodId { get; set; }
    public string FoodName { get; set; }
}

internal class Car
{
    [Key]
    public string LicensePlate { get; set; }

    public string Make { get; set; }
    public string Model { get; set; }
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Car>()
        .HasKey(c => c.LicensePlate);
}
```

## 생성된 값

### 기본값(default) 설정

기본적으로 지원되는 프로퍼티 특성을 이용한 방법은 없는 듯 하다.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
        .Property(b => b.Rating)
        .HasDefaultValue(3);
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
        .Property(b => b.Created)
        .HasDefaultValueSql("getdate()");
}
```

### 계산 열

```csharp
modelBuilder.Entity<Person>()
    .Property(p => p.DisplayName)
    .HasComputedColumnSql("[LastName] + ', ' + [FirstName]");
```

## Shadow

### 외래 키 섀도 속성

> 섀도 속성은 두 엔터티 간의 관계가 데이터베이스의 외래 키 값으로 표현되지만 엔터티 형식 간의 탐색 속성을 사용하여 엔터티 형식에서 관계가 관리되는 외래 키 속성에 가장 자주 사용됩니다. 규칙에 따라 EF는 관계가 검색되지만 종속 엔터티 클래스에서 외래 키 속성을 찾을 수 없는 경우 섀도 속성을 도입합니다.

```csharp
internal class MyContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }
}

public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }

    public List<Post> Posts { get; set; }
}

public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }

    // Since there is no CLR property which holds the foreign
    // key for this relationship, a shadow property is created.
    public Blog Blog { get; set; }
}
```

## 관계

### 용어 정의

- 종속 엔터티
  - 외래 키 속성을 포함하는 엔터티입니다. 관계의 '자식'이라고도 합니다.
- 주 엔터티
  - 기본/대체 키 속성을 포함하는 엔터티입니다. 관계의 '부모'라고도 합니다.
- 주 키
  - 주 엔터티를 고유하게 식별하는 속성입니다. 기본 키 또는 대체 키일 수 있습니다.
- 외래 키
  - 관련 엔터티의 보안 주체 키 값을 저장하는 데 사용되는 종속 엔터티의 속성입니다.
- 탐색 속성
  - 관련 엔터티를 참조하는 보안 주체 및 종속 엔터티에 정의된 속성입니다.
  - 종류
    - 컬렉션 탐색 속성
      - 많은 관련 엔터티에 대한 참조를 포함하는 탐색 속성입니다.
    - 참조 탐색 속성
      - 단일 관련 엔터티에 대한 참조를 보유하는 탐색 속성입니다.
    - 역 탐색 속성
      - 특정 탐색 속성에 대해 논의할 때 이 용어는 관계의 다른 쪽 끝에 있는 탐색 속성을 참조합니다.
- 자체 참조 관계
  - 종속 엔터티 형식과 주 엔터티 형식이 동일한 관계입니다.

다음 코드는 일대다 관계를 나타낸다.

```csharp
public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }

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

- `Post` 는 종속 엔터티입니다.
- `Blog` 는 주 엔터티입니다.
- `Blog.BlogId` 는 주 키(이 경우 대체 키가 아닌 기본 키)입니다.
- `Post.BlogId` 는 외래 키입니다.
- `Post.Blog` 는 참조 탐색 속성입니다.
- `Blog.Posts` 는 컬렉션 탐색 속성입니다.
- `Post.Blog` 는 역 탐색 속성(`Blog.Posts` 및 그 반대의 경우)입니다.

### 규칙

기본적으로 형식에서 탐색 속성이 검색되면 관계가 만들어진다. 가리키는 형식을 현재 데이터베이스 공급자가 스칼라 형식으로 매핑할 수 없는 경우 탐색 속성으로 간주된다.

### 하위 삭제

규칙에 따라 필수 관계의 경우 계단식 삭제가 Cascade로 설정되고 선택적 관계는 ClientSetNull이 설정된다.

Cascade는 종속 엔터티도 삭제됨을 의미한다. ClientSetNull은 메모리에 로드되지 않은 종속 엔터티가 변경되지 않고 수동으로 삭제되거나 유효한 주체 엔터티를 가리키도록 업데이트되어야 한다는 것을 의미한다.

### 명시적 하위 삭제

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Post>()
        .HasOne(p => p.Blog)
        .WithMany(b => b.Posts)
        .OnDelete(DeleteBehavior.Cascade);
}
```

### 일대일

```csharp
public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }

    public BlogImage BlogImage { get; set; }
}

public class BlogImage
{
    public int BlogImageId { get; set; }
    public byte[] Image { get; set; }
    public string Caption { get; set; }

    public int BlogId { get; set; }
    public Blog Blog { get; set; }
}
```

### 다대다

```csharp
public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }

    public ICollection<Tag> Tags { get; set; }
}

public class Tag
{
    public string TagId { get; set; }

    public ICollection<Post> Posts { get; set; }
}
```

```csharp
internal class MyContext : DbContext
{
    public MyContext(DbContextOptions<MyContext> options)
        : base(options)
    {
    }

    public DbSet<Post> Posts { get; set; }
    public DbSet<Tag> Tags { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Post>()
            .HasMany(p => p.Tags)
            .WithMany(p => p.Posts)
            .UsingEntity<PostTag>(
                j => j
                    .HasOne(pt => pt.Tag)
                    .WithMany(t => t.PostTags)
                    .HasForeignKey(pt => pt.TagId),
                j => j
                    .HasOne(pt => pt.Post)
                    .WithMany(p => p.PostTags)
                    .HasForeignKey(pt => pt.PostId),
                j =>
                {
                    j.Property(pt => pt.PublicationDate).HasDefaultValueSql("CURRENT_TIMESTAMP");
                    j.HasKey(t => new { t.PostId, t.TagId });
                });
    }
}

public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }

    public ICollection<Tag> Tags { get; set; }
    public List<PostTag> PostTags { get; set; }
}

public class Tag
{
    public string TagId { get; set; }

    public ICollection<Post> Posts { get; set; }
    public List<PostTag> PostTags { get; set; }
}

public class PostTag
{
    public DateTime PublicationDate { get; set; }

    public int PostId { get; set; }
    public Post Post { get; set; }

    public string TagId { get; set; }
    public Tag Tag { get; set; }
}
```

## 인덱스

인덱스와 관련된 성능에 대한 자세한 설명은 다음 링크를 확인. [인덱스 섹션](https://docs.microsoft.com/ko-kr/ef/core/performance/efficient-querying#use-indexes-properly)

```csharp
[Index(nameof(Url))]
public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
        .HasIndex(b => b.Url);
}
```

### 복합 인덱스

```csharp
[Index(nameof(FirstName), nameof(LastName))]
public class Person
{
    public int PersonId { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Person>()
        .HasIndex(p => new { p.FirstName, p.LastName });
}
```

### 고유 인덱스

```csharp
[Index(nameof(Url), IsUnique = true)]
public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }
}

protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
        .HasIndex(b => b.Url)
        .IsUnique();
}
```

### Check 제약 조건

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Product>()
        .HasCheckConstraint("CK_Prices", "[Price] > [DiscountedPrice]", c => c.HasName("CK_Product_Prices"));
}
```
