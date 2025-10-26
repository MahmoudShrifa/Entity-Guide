
## 🎯 مقدمة

لو إنت بتشتغل **Backend .NET Developer** فـ لازم تكون فاهم **Entity Framework Core (EF Core)** لأنها الأداة اللي بتخليك تتعامل مع قاعدة البيانات في C# بسهولة من غير ما تكتب SQL كتير.

EF Core ببساطة:
> Framework بتربط بين الكود بتاعك (C# Classes) وبين قاعدة البيانات (Tables) عن طريق مفهوم اسمه ORM – Object Relational Mapping.

---

## ⚙️ 1. إعداد المشروع

ابدأ مشروع Console أو Web API جديد في Visual Studio.

### ✅ أضف الحزم المطلوبة

في **Package Manager Console** اكتب:

```bash
Install-Package Microsoft.EntityFrameworkCore
Install-Package Microsoft.EntityFrameworkCore.SqlServer
Install-Package Microsoft.EntityFrameworkCore.Tools
```

---

## 🧩 2. إنشاء الـ DbContext

`DbContext` هو الكلاس اللي بيربط بين مشروعك وقاعدة البيانات.

```csharp
public class AppDbContext : DbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        options.UseSqlServer(@"Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=YourDatabaseName;Integrated Security=True");
    }

    public DbSet<Employee> Employees { get; set; }
}
```

> 💬 هنا عرفنا الاتصال بقاعدة البيانات (Connection String)  
> وقلنا إن عندنا جدول اسمه `Employees`.

---

## 👨‍💼 3. إنشاء كلاس الموديل (Model)

ده الكلاس اللي هيمثل الجدول في القاعدة.

```csharp
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

> EF Core هيحوّل الكلاس ده تلقائيًا إلى Table اسمه **Employees**.

---

## 🧱 4. إنشاء قاعدة البيانات (Migration)

الـ Migration هي اللي بتحول الكود بتاعك لجداول فعلية في SQL.

### إنشاء أول Migration:
```powershell
Add-Migration InitialCreate
```

### تنفيذها على القاعدة:
```powershell
Update-Database
```

### حذف Migration:
```powershell
Remove-Migration
```

> 💬 EF Core بيعمل Snapshot من الموديلات، وأي تعديل تعمل Migration جديدة بعده.

---

## ➕ 5. إضافة بيانات للقاعدة

```csharp
var _context = new AppDbContext();

var employee = new Employee
{
    Name = "Employee 1"
};

_context.Employees.Add(employee);
_context.SaveChanges();
```

> 🧠 الـ `Id` بيتولد أوتوماتيك لأنه **Primary Key**.

---

## 🔄 6. التراجع لنسخة سابقة من القاعدة

```powershell
Update-Database -Migration:0
Remove-Migration
```

---

## 🧾 7. إدخال بيانات داخل Migration

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.Sql("INSERT INTO Employees VALUES ('Employee 1')");
    migrationBuilder.Sql("INSERT INTO Employees VALUES ('Employee 2')");
}

protected override void Down(MigrationBuilder migrationBuilder)
{
    migrationBuilder.Sql("DELETE FROM Employees WHERE Name='Employee 1'");
}
```

---

## 📁 8. تنظيم المشروع بإضافة Folder Models

مثلاً نضيف كلاس Blog:

```csharp
public class Blog
{
    public int Id { get; set; }

    [Required]
    public string Url { get; set; }
}
```

ونربطه بالـ Context:

```csharp
public DbSet<Blog> Blogs { get; set; }
```

---

## 🧱 9. استخدام Fluent API

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
        .Property(b => b.Url)
        .IsRequired();
}
```

> 💬 بدل الـ Attributes نقدر نتحكم بكل حاجة من هنا.

---

## ⚙️ 10. فصل الإعدادات في Configuration Classes

```csharp
public class BlogEntityTypeConfig : IEntityTypeConfiguration<Blog>
{
    public void Configure(EntityTypeBuilder<Blog> builder)
    {
        builder.Property(b => b.Url).IsRequired();
    }
}
```

وفي الـ DbContext:

```csharp
modelBuilder.ApplyConfiguration(new BlogEntityTypeConfig());
```

---

## 🧩 11. العلاقات (Relationships)

### 🧍‍♂️ One-to-One
```csharp
modelBuilder.Entity<Blog>()
    .HasOne(b => b.BlogIMG)
    .WithOne(i => i.Blog)
    .HasForeignKey<BlogIMG>(b => b.BlogForeignKey);
```

### 👨‍👩‍👧 One-to-Many
```csharp
modelBuilder.Entity<Blog>()
    .HasMany(b => b.Posts)
    .WithOne(p => p.Blog);
```

### 🔁 Many-to-Many
```csharp
modelBuilder.Entity<Post>()
    .HasMany(p => p.Tags)
    .WithMany(t => t.Posts)
    .UsingEntity(j => j.ToTable("PostTags"));
```

---

## 🔍 12. الفهارس (Indexes)

```csharp
modelBuilder.Entity<Blog>()
    .HasIndex(b => b.Url);
```

### فهرس فريد:
```csharp
[Index(nameof(Url), IsUnique = true)]
public class Blog
{
    public int Id { get; set; }
    public string Url { get; set; }
}
```

---

## 🔢 13. القيم الافتراضية (Default Values)

```csharp
modelBuilder.Entity<Blog>()
    .Property(b => b.Rating)
    .HasDefaultValue(2);

modelBuilder.Entity<Blog>()
    .Property(b => b.CreatedOn)
    .HasDefaultValueSql("GETDATE()");
```

---

## 🧮 14. الأعمدة المحسوبة (Computed Columns)

```csharp
modelBuilder.Entity<Author>()
    .Property(a => a.DisplayName)
    .HasComputedColumnSql("[LastName] + ', ' + [FirstName]");
```

---

## 🗃️ 15. الـ Seeding (بيانات تلقائية عند الإنشاء)

```csharp
modelBuilder.Entity<Blog>()
    .HasData(new Blog { Id = 1, Url = "www.google.com" });
```

أو يدويًا:

```csharp
context.Database.EnsureCreated();
if (!context.Blogs.Any())
    context.Blogs.Add(new Blog { Url = "www.google.com" });
context.SaveChanges();
```

---

## 🧱 16. التعامل مع قاعدة بيانات موجودة (Scaffold)

```bash
Scaffold-DbContext "Connection String" Microsoft.EntityFrameworkCore.SqlServer
```

🧠 تقدر تضيف:
- `-Tables Blogs,Posts` ← لجداول معينة  
- `-OutputDir Models` ← لمسار حفظ الموديلات  
- `-DataAnnotations` ← يخلي الإعدادات Attributes  

---

## 🔍 17. قراءة البيانات (Read Data)

```csharp
var employees = _context.Employees.ToList();
foreach (var emp in employees)
    Console.WriteLine(emp.Name);
```

### قراءة عنصر محدد:
```csharp
var emp = _context.Employees.Find(1);
```

---

## 🔁 18. استعلامات LINQ

```csharp
var result = _context.Employees
    .Where(e => e.Id > 5)
    .OrderBy(e => e.Name)
    .ToList();
```

### أهم الدوال:
`.Any()` – `.All()` – `.FirstOrDefault()` – `.Count()` – `.Distinct()` – `.Skip().Take()`

---

## 📊 19. Group By & Aggregation

```csharp
var result = _context.Employees
    .GroupBy(e => e.Department)
    .Select(g => new {
        Dept = g.Key,
        Count = g.Count()
    });
```

---

## 🔗 20. Joins

### Inner Join:
```csharp
var data = _context.Books
    .Join(_context.Authors,
        b => b.AuthorId,
        a => a.Id,
        (b, a) => new { b.Title, a.Name });
```

---

## ➕ 21. إضافة بيانات

```csharp
_context.Employees.Add(new Employee { Name = "Mahmoud" });
_context.SaveChanges();
```

### إضافة مجموعة:
```csharp
_context.Employees.AddRange(
    new Employee { Name = "Ali" },
    new Employee { Name = "Sara" }
);
```

---

## 📝 22. تعديل البيانات (Update)

```csharp
var emp = _context.Employees.Find(1);
emp.Name = "Updated Name";
_context.SaveChanges();
```

أو:
```csharp
_context.Update(new Employee { Id = 1, Name = "Updated" });
_context.SaveChanges();
```

---

## 🗑️ 23. حذف البيانات (Delete)

```csharp
var emp = _context.Employees.Find(3);
_context.Employees.Remove(emp);
_context.SaveChanges();
```

---

## 🔐 24. Transactions

```csharp
using var transaction = _context.Database.BeginTransaction();

try
{
    _context.Blogs.Add(new Blog { Url = "Blog 1" });
    _context.SaveChanges();

    transaction.CreateSavepoint("Step1");

    _context.Blogs.Add(new Blog { Url = "Blog 2" });
    _context.SaveChanges();

    transaction.Commit();
}
catch
{
    transaction.RollbackToSavepoint("Step1");
}
```

---

## 📉 25. MinBy / MaxBy

```csharp
var cheapest = products.MinBy(p => p.Price);
var mostExpensive = products.MaxBy(p => p.Price);
```

---

## 💡 ملاحظات ختامية

- استخدم **Fluent API** للتحكم الكامل في الموديلات والعلاقات.  
- خليك دايمًا منظم في الـ DbContext.  
- لما تعمل تعديل على الموديل، اعمل **Migration جديدة**.  
- استخدم **Seeding** لو عايز تضيف بيانات مبدئية.  
- EF Core قوية جدًا، بس لازم تفهمها قبل ما تعتمد عليها بالكامل.

---

## 💼 About the Author – محمود شريفة

👨‍💻 **Backend .NET Developer**  
خريج **كلية حاسبات ومعلومات – جامعة 6 أكتوبر**  
متخصص في تطوير الأنظمة باستخدام:  
**C# – .NET Core – ASP.NET – SQL – EF Core – LINQ – HTML – CSS – JavaScript**

🔗 [LinkedIn Profile](https://www.linkedin.com/in/mahmoud-shrifa)

---

> ⭐ *Created with passion to help other .NET developers understand EF Core in a simple and practical way.*
````

---
