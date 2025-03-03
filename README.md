# 🎯 Entity Framework Core - LINQ ve Include() Kullanımı

(HALA Düzenlenme Aşamasında)

## 🔍 1️⃣ LINQ Sorguları - Entity Framework Core
LINQ (Language Integrated Query), veri üzerinde *filtreleme, sıralama ve gruplama* işlemleri yapmak için kullanılan güçlü bir sorgulama yöntemidir. Entity Framework Core ile LINQ kullanarak veritabanı işlemlerini kolayca gerçekleştirebiliriz.

---

### 📌 1️⃣1️⃣ `Where()` Kullanımı
`Where()`, belirli bir koşulu sağlayan verileri seçmek için kullanılır.

```csharp
var products = context.Products.Where(p => p.Price > 100).ToList();
```

🔹 *SQL Karşılığı:*
```sql
SELECT * FROM Products WHERE Price > 100;
```

---

### 📌 1️⃣2️⃣ `OrderBy()` ve `OrderByDescending()` Kullanımı

✔ *Küçükten büyüğe sıralama:*

```csharp
var products = context.Products.OrderBy(p => p.Price).ToList();
```

🔹 *SQL Karşılığı:*

```sql
SELECT * FROM Products ORDER BY Price ASC;
```

✔ *Büyükten küçüğe sıralama:*

```csharp
var products = context.Products.OrderByDescending(p => p.Price).ToList();
```

🔹 *SQL Karşılığı:*

```sql
SELECT * FROM Products ORDER BY Price DESC;
```

### 📌 1️⃣3️⃣ `Select()` Kullanımı

`Select()`, sadece belirli alanları seçmek için kullanılır.

```csharp
var names = context.Products.Select(p => p.Name).ToList();
```

🔹 *SQL Karşılığı:*

```sql
SELECT Name FROM Products;

```
---

### 📌 1️⃣4️⃣ `GroupBy()` Kullanımı

`GroupBy()`, verileri belirli bir alana göre gruplar.

```csharp
var groups = context.Products.GroupBy(p => p.CategoryId)
                             .Select(g => new { CategoryId = g.Key, Count = g.Count() })
                             .ToList();
```

🔹 *SQL Karşılığı:*

```sql
SELECT CategoryId, COUNT(*) FROM Products
 GROUP BY CategoryId;
```
### 📌 1️⃣5️⃣ `Join()` Kullanımı

`Join()`, iki tabloyu ilişkilendirerek veri çeker.

```csharp
var result = context.Products
             .Join(context.Categories,
                   p => p.CategoryId,
                   c => c.Id,
                   (p, c) => new { p.Name, CategoryName = c.Name })
             .ToList();
```

🔹 *SQL Karşılığı:*

```sql
SELECT p.Name, c.Name FROM Products p
 INNER JOIN
 Categories c ON p.CategoryId = c.Id;
```

### 📌 1️⃣6️⃣ `First()` ve `FirstOrDefault()` Kullanımı

`First()`, koleksiyondaki ilk öğeyi döndürür. Eğer öğe yoksa hata fırlatır.

```csharp
var product = context.Products.First(p => p.Price > 100);
```

🔹 *SQL Karşılığı:*

```sql
SELECT TOP 1 * FROM Products WHERE Price > 100;
```

`FirstOrDefault()`, koleksiyondaki ilk öğeyi döndürür, eğer öğe yoksa *null döner.

```csharp
var product = context.Products.FirstOrDefault(p => p.Price > 100);
```

### 📌 1️⃣7️⃣ `Single()` ve `SingleOrDefault()` Kullanımı

`Single()`, koleksiyonda yalnızca *tek bir öğe varsa döndürür, birden fazla varsa hata fırlatır.

```csharp
var product = context.Products.Single(p => p.Id == 1);
```

🔹 *SQL Karşılığı:*

```sql
SELECT * FROM Products WHERE Id = 1;
```

`SingleOrDefault()`, koleksiyonda yalnızca *tek bir öğe varsa döndürür, hiç öğe yoksa null döner.

```csharp
var product = context.Products.SingleOrDefault(p => p.Id == 1);
```

### 📌 1️⃣8️⃣ `Any()` Kullanımı

`Any()`, koleksiyonda belirli bir koşulu sağlayan herhangi bir öğe olup olmadığını kontrol eder.

```csharp
bool exists = context.Products.Any(p => p.Price > 500);
```

🔹 *SQL Karşılığı:*

```sql
SELECT CASE WHEN EXISTS (
    SELECT 1 FROM Products WHERE Price > 500
) THEN 1 ELSE 0 END;
```

### 📌 1️⃣9️⃣ `Count()` Kullanımı

`Count()`, koleksiyonda belirli bir koşulu sağlayan öğelerin sayısını döndürür.

```csharp
int count = context.Products.Count(p => p.CategoryId == 2);
```

🔹 *SQL Karşılığı:*

```sql
SELECT COUNT(*) FROM Products WHERE CategoryId = 2;
```

## 🔗 2️⃣ `Include()` Kullanımı

İlişkili tabloları tek sorguda getirmek için `Include()` kullanılır.

```csharp
var products = context.Products.Include(p => p.Category).ToList();
```

🔹 *SQL Karşılığı:*

```sql
SELECT p.*, c.* FROM Products p
 INNER JOIN Categories c
 ON p.CategoryId = c.Id;
```

### 📌 2️⃣2️⃣ `ThenInclude()` Kullanımı

Daha derin ilişkili verileri çekmek için `ThenInclude()` kullanılır.

```csharp
var products = context.Products.Include(p => p.Category).ThenInclude(c => c.Supplier).ToList();
```

🔹 *SQL Karşılığı:*

```sql
SELECT p.*, c.*, s.* FROM Products p
 INNER JOIN Categories c
 ON p.CategoryId = c.Id
 INNER JOIN Suppliers s ON c.SupplierId = s.Id;
```

### 📌 2️⃣3️⃣ `Include()` ile Filtreleme

Belirli kategorilere ait ürünleri çekmek için `Where()` ile filtreleme yapılır.

```csharp
var products = context.Products.Include(p => p.Category).Where(p => p.Category.Name == "Elektronik").ToList();
```

🔹 *SQL Karşılığı:*

```sql
SELECT p.*, c.* FROM Products p
 INNER JOIN Categories c
 ON p.CategoryId = c.Id 
WHERE c.Name = 'Elektronik';
```

### 📌 2️⃣4️⃣ `Include()` ile `Select()` Kullanımı
Belirli alanları çekmek için `Select()` kullanılır.

```csharp
var products = context.Products.Include(p => p.Category).Select(p => new { p.Name, Category = p.Category.Name }).ToList();
```

🔹 *SQL Karşılığı:*

```sql
SELECT p.Name, c.Name FROM Products p
 INNER JOIN Categories c 
ON p.CategoryId = c.Id;
```

## 🚀 Sonuç
Bu rehberde, *Entity Framework Core* ile *LINQ* ve *Include()* kullanarak nasıl etkili sorgular yazılacağını öğrendik. Doğru tekniklerle, hem performans hem de okunabilirlik açısından verimli sorgular yazabilirsiniz! 🎯
