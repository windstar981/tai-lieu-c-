# Các hàm thông dụng
<br>
1. Thêm mới một đối tượng vào cơ sở dữ liệu
```c
var newProduct = new Product { Name = "New Product", Price = 9.99 };
context.Products.Add(newProduct);
context.SaveChanges();

```