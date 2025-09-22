# EF Core 查询

## EFCore查询方法比较

### RAW SQL

```csharp
var products = context.Products
    .FromSqlRaw("SELECT * FROM Products WHERE Price > {0}", minPrice)
    .ToList();
```

### LINQ

1. 查询语法

```csharp
var expensiveProducts = from p in context.Products
                       where p.Price > 100000
                       orderby p.Price descending
                       select p;
```

2. 方法语法

```csharp
var products = context.Products
    .Where(p => p.Price > minPrice)
    .OrderByDescending(p => p.Price)
    .Take(10)
    .ToList();
```

| 特性           | Raw SQL    | LINQ查询语法                             | LINQ方法语法     |
|--------------|------------|--------------------------------------|--------------|
| 性能           | 完全控制       | 依赖EF翻译，可能有冗余                         | 依赖EF翻译，可能有冗余 |
| 可读性          | 类似SQL      | 最接近自然语言                              | 链式调用         |
| 类型安全         | 运行时检查      | 编译时检查                                | 编译时检查        |
| 维护性          | 字符串难维护     | 结构化代码方便维护                            | 最易重构         |
| 复杂查询支持       | 无限制        | 多表连接友好(不支持窗口函数，CTE，stored procedure) | 复杂连接较难(不支持窗口函数，CTE，stored procedure)  |
| 数据库独立性       | 平台依赖       | 独立                                   | 独立           |
|分页 | 需要手动实现 | Skip((pageNumber -1) * pageSize).Take(pageSize) | Skip((pageNumber -1) * pageSize).Take(pageSize) |

## EF Core 查询优化

### 1. N+1 查询问题
```csharp
var customers = context.Customers.ToList();
foreach (var customer in customers)
{
    var orders = context.Orders.Where(order => order.CustomerId == customer.Id).ToList();
}
```
EF Core Lazy Loading 会为每个customer单独查询订单，导致N+1查询问题。

**解决方案**: 使用Eager Loading

1. 单层Include
```csharp
var customers = context.Customers
    .Include(c => c.Orders)
    .ToList();
```
`Include`会转换成`left join`，一次性加载所有相关数据。

2. 多层Include
```csharp
var customers = context.Customers
    .Include(c => c.Orders)
        .ThenInclude(o => o.OrderItems)
    .AsSplitQuery()
    .ToList();
```
EF Core 转换成的SQL
```sql
SELECT * FROM Customers;
SELECT * FROM Orders WHERE CustomerId in (1,2,3,...);
SELECT * FROM OrderItems WHERE Id in (1,2,3,...);
```

### 2. Not using Projection
```csharp
var customers = context.Customers.ToList();
```

**解决方案**: select necessary attributes
```csharp
var customers = await context.Customers.
            .Select(u => new {
                u.Id,
                u.Name,
                u.OrderNumber
            })
            .ToListAsync();
```

### 3. AsNoTracking
如果只是查询，关闭跟踪

### 4. 数据库索引

### 5. 过于复杂的SQL使用RAW SQL + `FromSqlInterpolated`，注意SQL注入

```csharp
var customers = await context.Customers.FromSqlInterpolated($"SELECT * FROM Customers WHERE Status = {status} AND OrderNumber >= {orderNumber}")
.toListAsync();
```

### 6. 对于给前端展示的情况，使用分页查询
```
var customers = await context.Customers.
    .OrderBy(c => c.Id)
    .Skip(0)
    .Take(100)
    .ToListAsync();
```

### 7. 缓存

