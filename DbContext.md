# Các hàm thông dụng
<br>

1. Thêm mới một đối tượng vào cơ sở dữ liệu

```
    var newProduct = new Product { Name = "New Product", Price = 9.99 };
    context.Products.Add(newProduct);
    context.SaveChanges();

// muốn  thêm nhiều dữ liệu thì dùng AddRange
    var entitiesToAdd = new List<YourEntity>
    {
        new YourEntity { /* initialize properties */ },
        new YourEntity { /* initialize properties */ },
        // Add more entities as needed
    };

    context.YourEntities.AddRange(entitiesToAdd);
    context.SaveChanges();

```


**Lưu ý phải SaveChanges() thì mới lưu vào được cơ sở dữ liệu**


2. Lấy dữ liệu ra theo key

```
    var entity = context.YourEntities.Find(id);
```

3. Update dữ liệu

```
    var entity = context.YourEntities.Find(id); // tìm bản ghi cần update
    if (entity != null) // nếu tìm thấy bản ghi đó thì mố update đƯợc
    {
        entity.PropertyToUpdate = newValue;
        context.SaveChanges(); // update đơn giản là thay đổi giá trị và sau đó là save lại thôi
    }
```

4. Xoá một bản ghi

```
    var entity = context.YourEntities.Find(id); // tìm bản ghi cần xoá
    if (entity != null) // nếu tồn tại bản ghi đó thì mới xoá được
    {
        context.YourEntities.Remove(entity);
        context.SaveChanges();
    }
```

5. Thực hiện truy vấn

### Truy vấn đơn giản

```
    // lấy ra tất cả các trường
    var entities = context.YourEntities.Where(e => e.Property == value).ToList();
    // e ở đây là đại diện cho 1 row
    // sẽ trả về 1 list các bản ghi nên cần làm gì thì foreach nó :D
    foreach (var entity in entities)
    {
        // Do something with each entity
    }
```


```
    // lấy ra một số trường
    var entities = context.YourEntities.Where(e => e.Property == value)
                    .Select(entity => new
                    {
                        Property1 = entity.Property1,
                        Property2 = entity.Property2
                    })
                    .ToList();
    
```

```
// Nếu có nhiều điều kiện thì đơn giản là dùng and (&&) và or (||) thôi
    var entities = context.YourEntities
        .Where(e => (e.Property1 == value && e.Property2 == value) || e.Property3 == value)
        .ToList();
// Ta có thể dùng tất cả các hàm toán học hoặc string để so sánh điều kiện
// Ví dụ lấy ra thuộc tính có chứa từ "test" thì dùng e.Property2.Contains("Test") => SQL: e.Property2 like "%Test%"
```

```
// order by
    var entities = context.YourEntities.
                    Where(e => e.Property == value)
                    .OrderBy(keySelector)
                    .ToList();
// Mặc định là sắp xếp tăng dần, muốn dùng sắp xếp giảm dần thì dùng OrderByDescending
```

```
// group by
    var result = context.Entities
                    .Where(entity => /* your condition */)
                    .GroupBy(entity => entity.GroupingProperty)
                    .Select(grouped => new
                    {
                        GroupingProperty = grouped.Key,
                        Count = grouped.Count(),
                        Sum = grouped.Sum(x => x.PropertyToSum),
                        // You can include other aggregations or properties here
                    });

```

```
// select from where group by having order by limit offset
    var query = _context.Tests
        .Where(test => test.Name == "H")
        .GroupBy(test => test.Name)
        .Where(grouped => grouped.Count() >= 2)
        .Select(grouped => new
        {
            Name = grouped.Key
        })
        .OrderBy(t => t.Name)
        .Skip(0)
        .Take(1)
        .ToQueryString();
```

### Truy vấn có join với lại bảng khác thông qua mối quan hệ

1. Include

Bảng brands và products là quan hệ 1-n => khoá chính của brands làm khoá ngoại của products.
Khi này từ bảng products ta muốn link qua bảng brands để lấy thông tin thì dùng hàm Include.
Ví dụ:

```
    var query = _context.Products.Include(p => p.Brand)
        .ToQueryString();
```

### Lưu ý: muốn thêm điều kiện lọc của brands thì chỉ cần where điều kiện p.Brand.Property thôi :D

**Thực ra thì chỉ cần có quan hệ select thuộc tính của nó là nó tự left join rùi**

```
    var query = _context.Products.Select(p => p.Brand.Name).ToQueryString();    
```

Còn muốn dùng inner join tường minh thì như này:

```
    var query = _context.Products
        .Join(_context.Brands,
            product => product.BrandId, // Khóa ngoại trong bảng Products
            brand => brand.Id,     // Khóa chính trong bảng Brands
            (product, brand) => new { brand.Name, product.Price}) // Chọn ra các trường từ các bảng
        .ToQueryString();
```