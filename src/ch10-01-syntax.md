## Kiểu dữ liệu tổng quát (Generic Data Types)

Chúng ta sử dụng generic để tạo ra các định nghĩa cho những thành phần như chữ ký hàm hoặc struct, sau đó có thể dùng lại với nhiều kiểu dữ liệu cụ thể (concrete types) khác nhau. Trước hết, hãy xem cách định nghĩa hàm, struct, enum và method sử dụng generic. Sau đó, chúng ta sẽ thảo luận generic ảnh hưởng đến hiệu năng mã nguồn như thế nào.

### Trong định nghĩa hàm (Function Definitions)

Khi định nghĩa một hàm sử dụng generic, ta đặt các tham số kiểu (generic type parameters) trong chữ ký của hàm, tại vị trí mà thông thường ta sẽ chỉ định kiểu dữ liệu của tham số và giá trị trả về. Làm như vậy giúp mã linh hoạt hơn và cung cấp nhiều chức năng hơn cho phía gọi hàm, đồng thời tránh lặp mã.

Tiếp tục với hàm `largest` của chúng ta, Liệt kê 10-4 (Listing 10-4) cho thấy hai hàm đều tìm giá trị lớn nhất trong một slice. Sau đó, chúng ta sẽ kết hợp hai hàm này thành một hàm duy nhất sử dụng generic.

<Listing number="10-4" file-name="src/main.rs" caption="Hai hàm chỉ khác nhau về tên và kiểu trong chữ ký hàm">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-04/src/main.rs:here}}
```

</Listing>

Hàm `largest_i32` là hàm chúng ta đã tách ra trong Liệt kê 10-3 để tìm phần tử `i32` lớn nhất trong một slice. Hàm `largest_char` tìm ký tự `char` lớn nhất trong một slice. Phần thân hàm sử dụng cùng một đoạn mã, vì vậy chúng ta sẽ loại bỏ sự trùng lặp bằng cách giới thiệu một tham số kiểu generic trong một hàm duy nhất.

Để tham số hóa (parameterize) kiểu trong một hàm duy nhất mới, ta cần đặt tên cho tham số kiểu, giống như cách ta đặt tên cho tham số giá trị (value parameter) của hàm. Bạn có thể dùng bất kỳ định danh (identifier) nào làm tên tham số kiểu. Tuy nhiên, chúng ta sẽ dùng `T` vì theo quy ước, tên tham số kiểu trong Rust thường ngắn, thường chỉ một chữ cái, và theo quy ước đặt tên kiểu của Rust là UpperCamelCase. Viết tắt của _type_, `T` là lựa chọn mặc định của hầu hết lập trình viên Rust.

Khi sử dụng một tham số trong thân hàm, ta phải khai báo tên tham số đó trong chữ ký hàm để trình biên dịch biết tên đó có ý nghĩa gì. Tương tự, khi dùng tên tham số kiểu trong chữ ký hàm, ta phải khai báo tên tham số kiểu trước khi sử dụng. Để định nghĩa hàm generic `largest`, ta đặt các khai báo tên kiểu bên trong cặp ngoặc nhọn nhọn (angle brackets), `<>`, giữa tên hàm và danh sách tham số, như sau:

```rust,ignore
fn largest<T>(list: &[T]) -> &T {
```

Ta có thể đọc định nghĩa này như: “Hàm `largest` là generic trên một kiểu nào đó `T`.” Hàm này có một tham số tên là `list`, là một slice các giá trị thuộc kiểu `T`. Hàm `largest` sẽ trả về một tham chiếu đến một giá trị cùng kiểu `T`.

Liệt kê 10-5 (Listing 10-5) cho thấy định nghĩa hàm `largest` đã được kết hợp, sử dụng kiểu dữ liệu generic trong chữ ký hàm. Liệt kê này cũng thể hiện cách chúng ta có thể gọi hàm với một slice các giá trị `i32` hoặc các giá trị `char`. Lưu ý rằng đoạn mã này hiện tại vẫn chưa biên dịch được.

<Listing number="10-5" file-name="src/main.rs" caption="Hàm `largest` sử dụng tham số kiểu generic; đoạn mã này hiện chưa biên dịch được">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/src/main.rs}}
```

</Listing>

Nếu chúng ta biên dịch đoạn mã này ngay bây giờ, sẽ nhận được lỗi sau:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/output.txt}}
```

Phần trợ giúp (help text) đề cập đến `std::cmp::PartialOrd`, đây là một trait, và chúng ta sẽ bàn về trait trong phần tiếp theo. Tạm thời, hãy hiểu rằng lỗi này nói rằng phần thân hàm `largest` sẽ không hoạt động với mọi kiểu có thể gán cho `T`. Bởi vì trong thân hàm, chúng ta muốn so sánh các giá trị kiểu `T`, nên ta chỉ có thể dùng những kiểu mà giá trị của chúng có thể được sắp thứ tự (có khả năng so sánh thứ tự). Để hỗ trợ so sánh, thư viện chuẩn cung cấp trait `std::cmp::PartialOrd` mà bạn có thể triển khai (implement) cho các kiểu (xem Phụ lục C để biết thêm về trait này). Để sửa Liệt kê 10-5, ta có thể làm theo gợi ý trong phần trợ giúp và ràng buộc (restrict) các kiểu hợp lệ cho `T` chỉ còn những kiểu triển khai `PartialOrd`. Khi đó, liệt kê này sẽ biên dịch được vì thư viện chuẩn đã triển khai `PartialOrd` cho cả `i32` và `char`.

### Trong định nghĩa struct (Struct Definitions)

Chúng ta cũng có thể định nghĩa struct sử dụng tham số kiểu generic cho một hoặc nhiều trường bằng cú pháp `<>`. Liệt kê 10-6 định nghĩa struct `Point<T>` để lưu trữ các giá trị tọa độ `x` và `y` với bất kỳ kiểu nào.

<Listing number="10-6" file-name="src/main.rs" caption="Struct `Point<T>` lưu trữ các giá trị `x` và `y` thuộc kiểu `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-06/src/main.rs}}
```

</Listing>

Cú pháp sử dụng generic trong định nghĩa struct tương tự như trong định nghĩa hàm. Đầu tiên, ta khai báo tên tham số kiểu bên trong cặp ngoặc nhọn nhọn ngay sau tên struct. Sau đó, ta dùng kiểu generic trong định nghĩa struct tại những vị trí mà bình thường ta sẽ chỉ định kiểu dữ liệu cụ thể.

Lưu ý rằng vì chúng ta chỉ sử dụng một kiểu generic để định nghĩa `Point<T>`, định nghĩa này cho biết struct `Point<T>` là generic t