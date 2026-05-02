## Concise Control Flow with `if let` and `let...else`

Cú pháp `if let` cho phép bạn kết hợp `if` và `let` thành một cách viết ít verbose hơn để xử lý các values khớp một pattern, trong khi bỏ qua phần còn lại. Hãy xem chương trình trong Listing 6-6, nơi chúng ta `match` trên một giá trị `Option<u8>` trong biến `config_max` nhưng chỉ muốn execute code nếu value là variant `Some`.

<Listing number="6-6" caption="Một `match` chỉ quan tâm execute code khi value là `Some`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-06/src/main.rs:here}}
```

</Listing>

Nếu value là `Some`, chúng ta in ra giá trị bên trong variant `Some` bằng cách bind value đó vào biến `max` trong pattern. Chúng ta không muốn làm gì với value `None`. Để thỏa mãn `match` expression, chúng ta phải thêm `_ => ()` sau khi xử lý đúng một variant, điều này là boilerplate khá phiền.

Thay vào đó, chúng ta có thể viết ngắn hơn bằng `if let`. Đoạn code sau có behavior giống với `match` trong Listing 6-6:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-12-if-let/src/main.rs:here}}
```

Cú pháp `if let` nhận một pattern và một expression, ngăn cách bởi dấu bằng. Nó hoạt động giống `match`, nơi expression được đưa vào `match` và pattern là arm đầu tiên. Trong trường hợp này, pattern là `Some(max)`, và `max` bind với value bên trong `Some`. Chúng ta có thể dùng `max` trong body của block `if let` giống như đã dùng `max` trong match arm tương ứng. Code trong block `if let` chỉ chạy nếu value khớp pattern.

Dùng `if let` nghĩa là ít phải gõ hơn, ít indentation hơn, và ít boilerplate hơn. Tuy nhiên, bạn mất đi việc kiểm tra tính đầy đủ (exhaustive checking) mà `match` áp đặt để đảm bảo bạn không quên xử lý case nào. Việc chọn giữa `match` và `if let` tùy thuộc vào việc bạn đang làm trong tình huống cụ thể, và việc trade-off độ gọn gàng so với exhaustive checking có hợp lý hay không.

Nói cách khác, bạn có thể coi `if let` là syntax sugar của một `match` chạy code khi value khớp một pattern và bỏ qua các value khác.

Chúng ta có thể thêm `else` với `if let`. Block code đi với `else` giống block code đi với case `_` trong `match` expression tương đương với `if let` và `else`. Nhớ lại định nghĩa enum `Coin` trong Listing 6-4, nơi variant `Quarter` cũng chứa một giá trị `UsState`. Nếu chúng ta muốn đếm tất cả coins không phải quarter mà chúng ta thấy đồng thời thông báo bang của các quarters, ta có thể làm vậy bằng `match` expression như sau:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-13-count-and-announce-match/src/main.rs:here}}
```

Hoặc có thể dùng `if let` và `else` expression như sau:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-14-count-and-announce-if-let-else/src/main.rs:here}}
```

## Staying on the “Happy Path” with `let...else`

Pattern phổ biến là thực hiện một số computation khi value hiện diện và trả về một default value nếu không. Tiếp tục với ví dụ coins có giá trị `UsState`, nếu chúng ta muốn nói gì đó vui vui tùy thuộc vào việc bang trên quarter “già” đến mức nào, ta có thể thêm một method trên `UsState` để kiểm tra tuổi của bang như sau:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:state}}
```

Sau đó, chúng ta có thể dùng `if let` để match trên loại coin, tạo một biến `state` trong body của condition, như trong Listing 6-7.

<Listing number="6-7" caption="Kiểm tra một bang có tồn tại năm 1900 không bằng các condition lồng trong `if let`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:describe}}
```

</Listing>

Cách này hoàn thành công việc, nhưng nó đẩy phần xử lý vào body của `if let`, và nếu phần việc cần làm phức tạp hơn, sẽ khó mà theo dõi được các nhánh top-level quan hệ với nhau như thế nào. Chúng ta cũng có thể tận dụng việc expressions sinh ra value để hoặc là tạo ra `state` từ `if let`, hoặc return sớm, như trong Listing 6-8. (Bạn cũng có thể làm điều tương tự với `match`.)

<Listing number="6-8" caption="Dùng `if let` để sinh ra value hoặc return sớm">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-08/src/main.rs:describe}}
```

</Listing>

Tuy nhiên, theo dõi luồng chạy kiểu này cũng hơi khó theo! Một nhánh của `if let` sinh ra value, còn nhánh kia thì return khỏi function hoàn toàn.

Để làm cho pattern phổ biến này dễ biểu đạt hơn, Rust có `let...else`. Cú pháp `let...else` nhận một pattern ở bên trái và một expression ở bên phải, rất giống `if let`, nhưng nó không có nhánh `if` mà chỉ có nhánh `else`. Nếu pattern khớp, nó sẽ bind value từ pattern vào outer scope. Nếu pattern _không_ khớp, chương trình sẽ đi vào arm `else`, arm này bắt buộc phải return khỏi function.

Trong Listing 6-9, bạn có thể thấy Listing 6-8 trông như thế nào khi dùng `let...else` thay cho `if let`.

<Listing number="6-9" caption="Dùng `let...else` để làm rõ flow qua function">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-09/src/main.rs:describe}}
```

</Listing>

Hãy chú ý rằng cách này giữ luồng chạy trên “happy path” ở phần main body của function, không còn control flow khác biệt rõ giữa hai nhánh như `if let` nữa.

Nếu bạn gặp tình huống trong đó logic của chương trình quá verbose để diễn đạt bằng `match`, hãy nhớ rằng `if let` và `let...else` cũng nằm trong toolbox Rust của bạn.

## Summary

Chúng ta vừa đi qua cách dùng enums để tạo các custom types có thể là một trong một tập các giá trị liệt kê. Chúng ta đã chỉ ra cách `Option<T>` trong standard library giúp bạn dùng type system để tránh lỗi. Khi enum values có data bên trong, bạn có thể dùng `match` hoặc `if let` để extract và dùng các giá trị đó, tùy thuộc vào số lượng cases cần xử lý.

Rust programs của bạn giờ có thể biểu đạt các khái niệm trong domain bằng structs và enums. Việc tạo custom types để dùng trong API đảm bảo type safety: Compiler sẽ đảm bảo functions của bạn chỉ nhận các values đúng type mà mỗi function mong đợi.

Để cung cấp một API được tổ chức tốt cho users, dễ dùng và chỉ expose chính xác những gì họ cần, giờ hãy chuyển sang modules của Rust.