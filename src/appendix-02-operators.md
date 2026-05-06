## Phụ Lục B: Toán Tử và Ký Hiệu

Phụ lục này chứa bảng thuật ngữ về cú pháp của Rust, bao gồm các toán tử và các ký hiệu khác xuất hiện độc lập hoặc trong ngữ cảnh của đường dẫn, generics, trait bound, macro, thuộc tính, comment, tuple, và ngoặc.

### Toán Tử

Bảng B-1 chứa các toán tử trong Rust, một ví dụ về cách toán tử xuất hiện trong ngữ cảnh, giải thích ngắn gọn, và liệu toán tử đó có thể overload hay không. Nếu toán tử có thể overload, trait liên quan để overload toán tử đó được liệt kê.

<span class="caption">Bảng B-1: Toán Tử</span>

| Toán Tử                   | Ví dụ                                                   | Giải thích                                                            | Có thể Overload?  |
| ------------------------- | ------------------------------------------------------- | --------------------------------------------------------------------- | -------------- |
| `!`                       | `ident!(...)`, `ident!{...}`, `ident![...]`             | Khai triển macro                                                      |                |
| `!`                       | `!expr`                                                 | Phủ định bitwise hoặc logic                                           | `Not`          |
| `!=`                      | `expr != expr`                                          | So sánh không bằng                                                    | `PartialEq`    |
| `%`                       | `expr % expr`                                           | Phần dư số học                                                        | `Rem`          |
| `%=`                      | `var %= expr`                                           | Phần dư số học và gán                                                 | `RemAssign`    |
| `&`                       | `&expr`, `&mut expr`                                    | Borrow                                                                |                |
| `&`                       | `&type`, `&mut type`, `&'a type`, `&'a mut type`        | Kiểu con trỏ được borrow                                              |                |
| `&`                       | `expr & expr`                                           | AND bitwise                                                           | `BitAnd`       |
| `&=`                      | `var &= expr`                                           | AND bitwise và gán                                                    | `BitAndAssign` |
| `&&`                      | `expr && expr`                                          | AND logic ngắn mạch                                                   |                |
| `*`                       | `expr * expr`                                           | Nhân số học                                                           | `Mul`          |
| `*=`                      | `var *= expr`                                           | Nhân số học và gán                                                    | `MulAssign`    |
| `*`                       | `*expr`                                                 | Dereference                                                           | `Deref`        |
| `*`                       | `*const type`, `*mut type`                              | Con trỏ thô                                                           |                |
| `+`                       | `trait + trait`, `'a + trait`                           | Ràng buộc kiểu phức hợp                                               |                |
| `+`                       | `expr + expr`                                           | Cộng số học                                                           | `Add`          |
| `+=`                      | `var += expr`                                           | Cộng số học và gán                                                    | `AddAssign`    |
| `,`                       | `expr, expr`                                            | Dấu phân cách đối số và phần tử                                       |                |
| `-`                       | `- expr`                                                | Phủ định số học                                                       | `Neg`          |
| `-`                       | `expr - expr`                                           | Trừ số học                                                            | `Sub`          |
| `-=`                      | `var -= expr`                                           | Trừ số học và gán                                                     | `SubAssign`    |
| `->`                      | `fn(...) -> type`, <code>&vert;...&vert; -> type</code> | Kiểu trả về của function và closure                                   |                |
| `.`                       | `expr.ident`                                            | Truy cập trường                                                       |                |
| `.`                       | `expr.ident(expr, ...)`                                 | Gọi method                                                            |                |
| `.`                       | `expr.0`, `expr.1`, và tiếp theo                        | Lập chỉ mục tuple                                                     |                |
| `..`                      | `..`, `expr..`, `..expr`, `expr..expr`                  | Literal phạm vi không bao gồm bên phải                                | `PartialOrd`   |
| `..=`                     | `..=expr`, `expr..=expr`                                | Literal phạm vi bao gồm bên phải                                      | `PartialOrd`   |
| `..`                      | `..expr`                                                | Cú pháp cập nhật struct literal                                       |                |
| `..`                      | `variant(x, ..)`, `struct_type { x, .. }`               | Pattern binding "và phần còn lại"                                     |                |
| `...`                     | `expr...expr`                                           | (Không khuyến khích, dùng `..=` thay thế) Trong pattern: pattern phạm vi bao gồm |                |
| `/`                       | `expr / expr`                                           | Chia số học                                                           | `Div`          |
| `/=`                      | `var /= expr`                                           | Chia số học và gán                                                    | `DivAssign`    |
| `:`                       | `pat: type`, `ident: type`                              | Ràng buộc                                                             |                |
| `:`                       | `ident: expr`                                           | Khởi tạo trường struct                                                |                |
| `:`                       | `'a: loop {...}`                                        | Nhãn vòng lặp                                                         |                |
| `;`                       | `expr;`                                                 | Kết thúc câu lệnh và item                                             |                |
| `;`                       | `[...; len]`                                            | Một phần của cú pháp mảng kích thước cố định                          |                |
| `<<`                      | `expr << expr`                                          | Dịch trái                                                             | `Shl`          |
| `<<=`                     | `var <<= expr`                                          | Dịch trái và gán                                                      | `ShlAssign`    |
| `<`                       | `expr < expr`                                           | So sánh nhỏ hơn                                                       | `PartialOrd`   |
| `<=`                      | `expr <= expr`                                          | So sánh nhỏ hơn hoặc bằng                                             | `PartialOrd`   |
| `=`                       | `var = expr`, `ident = type`                            | Gán/tương đương                                                       |                |
| `==`                      | `expr == expr`                                          | So sánh bằng                                                          | `PartialEq`    |
| `=>`                      | `pat => expr`                                           | Một phần của cú pháp nhánh match                                      |                |
| `>`                       | `expr > expr`                                           | So sánh lớn hơn                                                       | `PartialOrd`   |
| `>=`                      | `expr >= expr`                                          | So sánh lớn hơn hoặc bằng                                             | `PartialOrd`   |
| `>>`                      | `expr >> expr`                                          | Dịch phải                                                             | `Shr`          |
| `>>=`                     | `var >>= expr`                                          | Dịch phải và gán                                                      | `ShrAssign`    |
| `@`                       | `ident @ pat`                                           | Pattern binding                                                       |                |
| `^`                       | `expr ^ expr`                                           | OR độc quyền bitwise                                                  | `BitXor`       |
| `^=`                      | `var ^= expr`                                           | OR độc quyền bitwise và gán                                           | `BitXorAssign` |
| <code>&vert;</code>       | <code>pat &vert; pat</code>                             | Các lựa chọn pattern                                                  |                |
| <code>&vert;</code>       | <code>expr &vert; expr</code>                           | OR bitwise                                                            | `BitOr`        |
| <code>&vert;=</code>      | <code>var &vert;= expr</code>                           | OR bitwise và gán                                                     | `BitOrAssign`  |
| <code>&vert;&vert;</code> | <code>expr &vert;&vert; expr</code>                     | OR logic ngắn mạch                                                    |                |
| `?`                       | `expr?`                                                 | Lan truyền lỗi                                                        |                |

### Các Ký Hiệu Không Phải Toán Tử

Các bảng sau đây chứa tất cả các ký hiệu không hoạt động như toán tử; nghĩa là, chúng không hoạt động giống như một lời gọi function hoặc method.

Bảng B-2 hiển thị các ký hiệu xuất hiện độc lập và hợp lệ ở nhiều vị trí khác nhau.

<span class="caption">Bảng B-2: Cú Pháp Độc Lập</span>

| Ký Hiệu                                                                | Giải thích                                                             |
| ---------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `'ident`                                                               | Lifetime được đặt tên hoặc nhãn vòng lặp                              |
| Chữ số ngay sau `u8`, `i32`, `f64`, `usize`, v.v.                      | Literal số có kiểu cụ thể                                              |
| `"..."`                                                                | Literal chuỗi                                                          |
| `r"..."`, `r#"..."#`, `r##"..."##`, v.v.                               | Literal chuỗi thô; các ký tự escape không được xử lý                  |
| `b"..."`                                                               | Literal chuỗi byte; tạo một mảng byte thay vì chuỗi                   |
| `br"..."`, `br#"..."#`, `br##"..."##`, v.v.                            | Literal chuỗi byte thô; kết hợp của literal chuỗi thô và byte         |
| `'...'`                                                                | Literal ký tự                                                          |
| `b'...'`                                                               | Literal byte ASCII                                                     |
| <code>&vert;...&vert; expr</code>                                      | Closure                                                                |
| `!`                                                                    | Kiểu bottom luôn rỗng cho các function phân kỳ                        |
| `_`                                                                    | Pattern binding "bị bỏ qua"; cũng được dùng để làm cho literal số nguyên dễ đọc hơn |

Bảng B-3 hiển thị các ký hiệu xuất hiện trong ngữ cảnh của đường dẫn qua hệ thống phân cấp module đến một item.

<span class="caption">Bảng B-3: Cú Pháp Liên Quan Đến Đường Dẫn</span>

| Ký Hiệu                                 | Giải thích                                                                                                   |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------|
| `ident::ident`                          | Đường dẫn namespace                                                                                          |
| `::path`                                | Đường dẫn tương đối với gốc crate (nghĩa là đường dẫn tuyệt đối rõ ràng)                                    |
| `self::path`                            | Đường dẫn tương đối với module hiện tại (nghĩa là đường dẫn tương đối rõ ràng)                              |
| `super::path`                           | Đường dẫn tương đối với cha của module hiện tại                                                              |
| `type::ident`, `<type as trait>::ident` | Hằng số, function và kiểu liên kết                                                                           |
| `<type>::...`                           | Item liên kết cho một kiểu không thể được đặt tên trực tiếp (ví dụ: `<&T>::...`, `<[T]>::...`, v.v.)         |
| `trait::method(...)`                    | Phân biệt một lời gọi method bằng cách đặt tên trait định nghĩa nó                                          |
| `type::method(...)`                     | Phân biệt một lời gọi method bằng cách đặt tên kiểu mà nó được định nghĩa                                   |
| `<type as trait>::method(...)`          | Phân biệt một lời gọi method bằng cách đặt tên trait và kiểu                                                |

Bảng B-4 hiển thị các ký hiệu xuất hiện trong ngữ cảnh sử dụng tham số kiểu generic.

<span class="caption">Bảng B-4: Generics</span>

| Ký Hiệu                        | Giải thích                                                                                                                                          |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `path<...>`                    | Chỉ định tham số cho một kiểu generic trong một kiểu (ví dụ: `Vec<u8>`)                                                                             |
| `path::<...>`, `method::<...>` | Chỉ định tham số cho một kiểu generic, function, hoặc method trong một biểu thức; thường được gọi là _turbofish_ (ví dụ: `"42".parse::<i32>()`)    |
| `fn ident<...> ...`            | Định nghĩa function generic                                                                                                                         |
| `struct ident<...> ...`        | Định nghĩa cấu trúc generic                                                                                                                         |
| `enum ident<...> ...`          | Định nghĩa enumeration generic                                                                                                                      |
| `impl<...> ...`                | Định nghĩa implementation generic                                                                                                                   |
| `for<...> type`                | Ràng buộc lifetime bậc cao hơn                                                                                                                      |
| `type<ident=type>`             | Một kiểu generic nơi một hoặc nhiều kiểu liên kết có gán cụ thể (ví dụ: `Iterator<Item=T>`)                                                         |

Bảng B-5 hiển thị các ký hiệu xuất hiện trong ngữ cảnh ràng buộc tham số kiểu generic với trait bound.

<span class="caption">Bảng B-5: Ràng Buộc Trait Bound</span>

| Ký Hiệu                       | Giải thích                                                                                                                                 |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `T: U`                        | Tham số generic `T` bị ràng buộc với các kiểu implement `U`                                                                               |
| `T: 'a`                       | Kiểu generic `T` phải tồn tại lâu hơn lifetime `'a` (nghĩa là kiểu không thể chứa bất kỳ reference nào có lifetime ngắn hơn `'a`)         |
| `T: 'static`                  | Kiểu generic `T` không chứa bất kỳ reference được borrow nào ngoài các reference `'static`                                                |
| `'b: 'a`                      | Lifetime generic `'b` phải tồn tại lâu hơn lifetime `'a`                                                                                  |
| `T: ?Sized`                   | Cho phép tham số kiểu generic là một kiểu có kích thước động                                                                              |
| `'a + trait`, `trait + trait` | Ràng buộc kiểu phức hợp                                                                                                                   |

Bảng B-6 hiển thị các ký hiệu xuất hiện trong ngữ cảnh gọi hoặc định nghĩa macro và chỉ định thuộc tính trên một item.

<span class="caption">Bảng B-6: Macro và Thuộc Tính</span>

| Ký Hiệu                                     | Giải thích              |
| ------------------------------------------- | ----------------------- |
| `#[meta]`                                   | Thuộc tính ngoài        |
| `#![meta]`                                  | Thuộc tính trong        |
| `$ident`                                    | Thay thế macro          |
| `$ident:kind`                               | Metavariable macro      |
| `$(...)...`                                 | Lặp macro               |
| `ident!(...)`, `ident!{...}`, `ident![...]` | Gọi macro               |

Bảng B-7 hiển thị các ký hiệu tạo comment.

<span class="caption">Bảng B-7: Comment</span>

| Ký Hiệu    | Giải thích                      |
| ---------- | ------------------------------- |
| `//`       | Comment dòng                    |
| `//!`      | Comment tài liệu dòng trong     |
| `///`      | Comment tài liệu dòng ngoài     |
| `/*...*/`  | Comment khối                    |
| `/*!...*/` | Comment tài liệu khối trong     |
| `/**...*/` | Comment tài liệu khối ngoài     |

Bảng B-8 hiển thị các ngữ cảnh sử dụng dấu ngoặc tròn.

<span class="caption">Bảng B-8: Dấu Ngoặc Tròn</span>

| Ký Hiệu                  | Giải thích                                                                                  |
| ------------------------ | ------------------------------------------------------------------------------------------- |
| `()`                     | Tuple rỗng (hay unit), cả literal lẫn kiểu                                                  |
| `(expr)`                 | Biểu thức trong ngoặc                                                                       |
| `(expr,)`                | Biểu thức tuple một phần tử                                                                 |
| `(type,)`                | Kiểu tuple một phần tử                                                                      |
| `(expr, ...)`            | Biểu thức tuple                                                                             |
| `(type, ...)`            | Kiểu tuple                                                                                  |
| `expr(expr, ...)`        | Biểu thức gọi function; cũng được dùng để khởi tạo `struct` tuple và variant `enum` tuple  |

Bảng B-9 hiển thị các ngữ cảnh sử dụng dấu ngoặc nhọn.

<span class="caption">Bảng B-9: Dấu Ngoặc Nhọn</span>

| Ngữ cảnh     | Giải thích              |
| ------------ | ----------------------- |
| `{...}`      | Biểu thức khối          |
| `Type {...}` | Literal struct          |

Bảng B-10 hiển thị các ngữ cảnh sử dụng dấu ngoặc vuông.

<span class="caption">Bảng B-10: Dấu Ngoặc Vuông</span>

| Ngữ cảnh                                           | Giải thích                                                                                                                    |
| -------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `[...]`                                            | Literal mảng                                                                                                                  |
| `[expr; len]`                                      | Literal mảng chứa `len` bản sao của `expr`                                                                                   |
| `[type; len]`                                      | Kiểu mảng chứa `len` instance của `type`                                                                                      |
| `expr[expr]`                                       | Lập chỉ mục collection; có thể overload (`Index`, `IndexMut`)                                                                 |
| `expr[..]`, `expr[a..]`, `expr[..b]`, `expr[a..b]` | Lập chỉ mục collection giả vờ là slicing collection, sử dụng `Range`, `RangeFrom`, `RangeTo`, hoặc `RangeFull` làm "index"    |
