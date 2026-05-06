## Phụ Lục A: Từ Khóa

Danh sách sau đây chứa các từ khóa được dành riêng cho việc sử dụng hiện tại hoặc tương lai bởi ngôn ngữ Rust. Do đó, chúng không thể được dùng làm identifier (ngoại trừ dưới dạng raw identifier, như chúng ta thảo luận trong phần ["Raw Identifiers"][raw-identifiers]<!-- ignore -->). _Identifier_ là tên của các function, biến, tham số, trường struct, module, crate, hằng số, macro, giá trị static, thuộc tính, kiểu dữ liệu, trait, hoặc lifetime.

[raw-identifiers]: #raw-identifiers

### Các Từ Khóa Đang Được Sử Dụng

Dưới đây là danh sách các từ khóa hiện đang được sử dụng, cùng với mô tả chức năng của chúng.

- **`as`**: Thực hiện ép kiểu nguyên thủy, phân biệt trait cụ thể chứa một item, hoặc đổi tên các item trong câu lệnh `use`.
- **`async`**: Trả về một `Future` thay vì chặn luồng hiện tại.
- **`await`**: Tạm dừng thực thi cho đến khi kết quả của một `Future` sẵn sàng.
- **`break`**: Thoát khỏi vòng lặp ngay lập tức.
- **`const`**: Định nghĩa các item hằng số hoặc con trỏ thô hằng số.
- **`continue`**: Tiếp tục đến lần lặp tiếp theo của vòng lặp.
- **`crate`**: Trong đường dẫn module, tham chiếu đến gốc của crate.
- **`dyn`**: Điều phối động đến một trait object.
- **`else`**: Nhánh dự phòng cho các cấu trúc điều khiển `if` và `if let`.
- **`enum`**: Định nghĩa một enumeration.
- **`extern`**: Liên kết một function hoặc biến bên ngoài.
- **`false`**: Literal boolean false.
- **`fn`**: Định nghĩa một function hoặc kiểu con trỏ function.
- **`for`**: Lặp qua các item từ một iterator, implement một trait, hoặc chỉ định một higher ranked lifetime.
- **`if`**: Rẽ nhánh dựa trên kết quả của một biểu thức điều kiện.
- **`impl`**: Implement chức năng inherent hoặc trait.
- **`in`**: Một phần của cú pháp vòng lặp `for`.
- **`let`**: Gắn kết một biến.
- **`loop`**: Lặp vô điều kiện.
- **`match`**: So khớp một giá trị với các pattern.
- **`mod`**: Định nghĩa một module.
- **`move`**: Khiến một closure lấy ownership của tất cả các giá trị mà nó capture.
- **`mut`**: Biểu thị tính có thể thay đổi trong các reference, con trỏ thô, hoặc pattern binding.
- **`pub`**: Biểu thị khả năng hiển thị công khai trong các trường struct, block `impl`, hoặc module.
- **`ref`**: Gắn kết bằng reference.
- **`return`**: Trả về từ function.
- **`Self`**: Bí danh kiểu cho kiểu mà chúng ta đang định nghĩa hoặc implementing.
- **`self`**: Đối tượng method hoặc module hiện tại.
- **`static`**: Biến toàn cục hoặc lifetime kéo dài suốt toàn bộ quá trình thực thi chương trình.
- **`struct`**: Định nghĩa một cấu trúc.
- **`super`**: Module cha của module hiện tại.
- **`trait`**: Định nghĩa một trait.
- **`true`**: Literal boolean true.
- **`type`**: Định nghĩa bí danh kiểu hoặc kiểu liên kết.
- **`union`**: Định nghĩa một [union][union]<!-- ignore -->; chỉ là từ khóa khi được dùng trong khai báo union.
- **`unsafe`**: Biểu thị code, function, trait, hoặc implementation không an toàn.
- **`use`**: Đưa các ký hiệu vào phạm vi.
- **`where`**: Biểu thị các mệnh đề ràng buộc một kiểu.
- **`while`**: Lặp có điều kiện dựa trên kết quả của một biểu thức.

[union]: ../reference/items/unions.html

### Các Từ Khóa Dành Cho Sử Dụng Trong Tương Lai

Các từ khóa sau đây chưa có bất kỳ chức năng nào nhưng được Rust dành riêng cho việc sử dụng tiềm năng trong tương lai:

- `abstract`
- `become`
- `box`
- `do`
- `final`
- `gen`
- `macro`
- `override`
- `priv`
- `try`
- `typeof`
- `unsized`
- `virtual`
- `yield`

### Raw Identifier

_Raw identifier_ là cú pháp cho phép bạn sử dụng các từ khóa ở những nơi mà chúng thường không được phép. Bạn sử dụng raw identifier bằng cách thêm tiền tố `r#` vào trước một từ khóa.

Ví dụ, `match` là một từ khóa. Nếu bạn cố biên dịch function sau sử dụng `match` làm tên:

<span class="filename">Tên file: src/main.rs</span>

```rust,ignore,does_not_compile
fn match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}
```

bạn sẽ nhận được lỗi sau:

```text
error: expected identifier, found keyword `match`
 --> src/main.rs:4:4
  |
4 | fn match(needle: &str, haystack: &str) -> bool {
  |    ^^^^^ expected identifier, found keyword
```

Lỗi cho thấy bạn không thể sử dụng từ khóa `match` làm identifier của function. Để sử dụng `match` làm tên function, bạn cần dùng cú pháp raw identifier, như sau:

<span class="filename">Tên file: src/main.rs</span>

```rust
fn r#match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}

fn main() {
    assert!(r#match("foo", "foobar"));
}
```

Code này sẽ biên dịch mà không có bất kỳ lỗi nào. Lưu ý tiền tố `r#` trên tên function trong phần định nghĩa của nó cũng như nơi function được gọi trong `main`.

Raw identifier cho phép bạn sử dụng bất kỳ từ nào bạn chọn làm identifier, ngay cả khi từ đó là một từ khóa dành riêng. Điều này cho chúng ta thêm tự do trong việc chọn tên identifier, cũng như cho phép chúng ta tích hợp với các chương trình được viết bằng ngôn ngữ mà những từ này không phải là từ khóa. Ngoài ra, raw identifier cho phép bạn sử dụng các thư viện được viết trong một Rust edition khác với edition mà crate của bạn sử dụng. Ví dụ, `try` không phải là từ khóa trong edition 2015 nhưng là từ khóa trong các edition 2018, 2021, và 2024. Nếu bạn phụ thuộc vào một thư viện được viết bằng edition 2015 và có một function `try`, bạn sẽ cần sử dụng cú pháp raw identifier, `r#try` trong trường hợp này, để gọi function đó từ code của bạn trên các edition mới hơn. Xem [Phụ Lục E][appendix-e]<!-- ignore --> để biết thêm thông tin về các edition.

[appendix-e]: appendix-05-editions.html
