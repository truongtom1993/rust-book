## Methods

Methods giống như functions: Chúng ta khai báo chúng bằng từ khóa `fn` và một tên, chúng có thể có parameters và return value, và chúng chứa code được chạy khi method được gọi từ nơi khác. Khác với functions, methods được định nghĩa trong context của một struct (hoặc enum hoặc trait object, mà chúng ta sẽ đề cập trong [Chapter 6][enums] và [Chapter 18][trait-objects]), và parameter đầu tiên luôn là `self`, đại diện cho instance của struct mà method đang được gọi trên đó.

### Method Syntax

Hãy thay đổi function `area` có parameter là instance của `Rectangle` và thay vào đó làm một method `area` được định nghĩa trên struct `Rectangle`, như trong Listing 5-13.

<Listing number="5-13" file-name="src/main.rs" caption="Định nghĩa method `area` trên struct `Rectangle`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-13/src/main.rs}}
```

</Listing>

Để định nghĩa function trong context của `Rectangle`, chúng ta bắt đầu một `impl` block cho `Rectangle`. Mọi thứ trong `impl` block này sẽ liên kết với type `Rectangle`. Sau đó, chúng ta di chuyển function `area` vào trong ngoặc nhọn của `impl` và thay đổi parameter đầu tiên (và ở đây là duy nhất) thành `self` trong signature và mọi nơi trong body. Trong `main`, nơi chúng ta gọi function `area` và truyền `rect1` làm argument, chúng ta có thể sử dụng _method syntax_ để gọi method `area` trên instance `Rectangle` của mình. Method syntax đặt sau instance: Chúng ta thêm dấu chấm theo sau là tên method, ngoặc đơn, và bất kỳ arguments nào.

Trong signature của `area`, chúng ta dùng `&self` thay vì `rectangle: &Rectangle`. `&self` thực ra là viết tắt của `self: &Self`. Trong một `impl` block, type `Self` là alias cho type mà `impl` block dành cho. Methods phải có parameter tên `self` kiểu `Self` làm parameter đầu tiên, nên Rust cho phép viết tắt chỉ bằng tên `self` ở vị trí parameter đầu. Lưu ý rằng chúng ta vẫn cần dùng `&` trước `self` để chỉ ra method này borrow instance `Self`, giống như chúng ta làm với `rectangle: &Rectangle`. Methods có thể take ownership của `self`, borrow `self` immutably như chúng ta làm ở đây, hoặc borrow `self` mutably, giống như với bất kỳ parameter nào khác.

Chúng ta chọn `&self` ở đây vì lý do giống như dùng `&Rectangle` trong phiên bản function: Chúng ta không muốn take ownership, và chỉ muốn đọc data trong struct, không viết vào nó. Nếu muốn thay đổi instance mà chúng ta gọi method trên như một phần của việc method làm, chúng ta sẽ dùng `&mut self` làm parameter đầu tiên. Việc có method take ownership của instance bằng cách dùng chỉ `self` làm parameter đầu là hiếm; kỹ thuật này thường dùng khi method transform `self` thành thứ khác và bạn muốn ngăn caller dùng instance gốc sau transformation.

Lý do chính dùng methods thay vì functions, ngoài việc cung cấp method syntax và không phải lặp type của `self` trong mọi signature method, là để tổ chức code. Chúng ta đặt tất cả những gì có thể làm với instance của một type vào một `impl` block thay vì buộc người dùng tương lai phải tìm capabilities của `Rectangle` ở nhiều nơi trong library.

Lưu ý rằng chúng ta có thể đặt tên method giống tên một field của struct. Ví dụ, chúng ta có thể định nghĩa method trên `Rectangle` cũng tên `width`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-06-method-field-interaction/src/main.rs:here}}
```

</Listing>

Ở đây, chúng ta chọn làm method `width` return `true` nếu giá trị trong field `width` của instance lớn hơn `0` và `false` nếu là `0`: Chúng ta có thể dùng field trong method cùng tên cho bất kỳ mục đích nào. Trong `main`, khi theo sau `rect1.width` bằng ngoặc đơn, Rust biết chúng ta ám chỉ method `width`. Khi không dùng ngoặc đơn, Rust biết chúng ta ám chỉ field `width`.

Thường thì, nhưng không phải luôn, khi đặt tên method giống field chúng ta chỉ muốn nó return giá trị của field và không làm gì khác. Những methods như vậy gọi là _getters_, và Rust không tự implement chúng cho fields của struct như một số ngôn ngữ khác. Getters hữu ích vì bạn có thể làm field private nhưng method public, từ đó cho phép read-only access đến field đó như một phần của public API của type. Chúng ta sẽ thảo luận public và private là gì và cách chỉ định field hoặc method là public hoặc private trong [Chapter 7][public].

> ### Toán tử `->` ở đâu?
>
> Trong C và C++, hai toán tử khác nhau dùng để gọi methods: Dùng `.` nếu gọi method trực tiếp trên object và `->` nếu gọi method trên pointer đến object và cần dereference pointer trước. Nói cách khác, nếu `object` là pointer, `object->something()` giống `(*object).something()`.
>
> Rust không có tương đương `->`; thay vào đó, Rust có tính năng _automatic referencing and dereferencing_. Gọi methods là một trong số ít nơi Rust có hành vi này.
>
> Nó hoạt động thế này: Khi gọi method bằng `object.something()`, Rust tự động thêm `&`, `&mut`, hoặc `*` để `object` khớp signature của method. Nói cách khác, những cái sau là giống nhau:
>
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> Cái đầu trông sạch hơn nhiều. Hành vi automatic referencing này hoạt động vì methods có receiver rõ ràng—kiểu của `self`. Với receiver và tên method, Rust có thể xác định chắc chắn method đang đọc (`&self`), mutate (`&mut self`), hay consume (`self`). Việc Rust làm borrowing implicit cho method receivers là phần lớn làm ownership ergonomic trong thực tế.

### Methods với Nhiều Parameters Hơn

Hãy thực hành dùng methods bằng cách implement method thứ hai trên struct `Rectangle`. Lần này chúng ta muốn instance của `Rectangle` nhận instance `Rectangle` khác và return `true` nếu `Rectangle` thứ hai có thể fit hoàn toàn trong `self` ( `Rectangle` đầu tiên); nếu không thì return `false`. Nghĩa là, sau khi định nghĩa method `can_hold`, chúng ta muốn viết chương trình như trong Listing 5-14.

<Listing number="5-14" file-name="src/main.rs" caption="Sử dụng method `can_hold` chưa viết">

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-14/src/main.rs}}
```

</Listing>

Output mong đợi sẽ như sau vì cả hai dimensions của `rect2` nhỏ hơn dimensions của `rect1`, nhưng `rect3` rộng hơn `rect1`:

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

Chúng ta biết muốn định nghĩa method, nên nó sẽ trong `impl Rectangle` block. Tên method là `can_hold`, và nó nhận immutable borrow của `Rectangle` khác làm parameter. Chúng ta có thể biết kiểu parameter bằng cách xem code gọi method: `rect1.can_hold(&rect2)` truyền `&rect2`, là immutable borrow của `rect2`, instance của `Rectangle`. Điều này hợp lý vì chúng ta chỉ cần đọc `rect2` (không viết, nghĩa là cần mutable borrow), và muốn `main` giữ ownership của `rect2` để dùng lại sau khi gọi `can_hold`. Return value của `can_hold` là Boolean, và implementation sẽ check width và height của `self` có lớn hơn width và height của `Rectangle` kia tương ứng không. Hãy thêm method `can_hold` mới vào `impl` block từ Listing 5-13, như trong Listing 5-15.

<Listing number="5-15" file-name="src/main.rs" caption="Implement method `can_hold` trên `Rectangle` nhận instance `Rectangle` khác làm parameter">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-15/src/main.rs:here}}
```

</Listing>

Khi chạy code này với `main` function trong Listing 5-14, chúng ta sẽ được output mong muốn. Methods có thể nhận nhiều parameters thêm vào signature sau `self` parameter, và những parameters đó hoạt động giống parameters trong functions.

### Associated Functions

Tất cả functions định nghĩa trong `impl` block gọi là _associated functions_ vì chúng liên kết với type sau `impl`. Chúng ta có thể định nghĩa associated functions không có `self` làm parameter đầu (và vậy không phải methods) vì chúng không cần instance của type để làm việc. Chúng ta đã dùng một function như vậy: function `String::from` định nghĩa trên type `String`.

Associated functions không phải methods thường dùng cho constructors return instance mới của struct. Chúng thường gọi `new`, nhưng `new` không phải tên đặc biệt và không built-in ngôn ngữ. Ví dụ, chúng ta có thể cung cấp associated function tên `square` có một dimension parameter và dùng nó cho cả width và height, làm dễ tạo `Rectangle` hình vuông hơn là phải chỉ định giá trị giống nhau hai lần:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-03-associated-functions/src/main.rs:here}}
```

Các keyword `Self` trong return type và body function là alias cho type sau `impl` keyword, ở đây là `Rectangle`.

Để gọi associated function này, dùng syntax `::` với tên struct; `let sq = Rectangle::square(3);` là ví dụ. Function này namespaced bởi struct: Syntax `::` dùng cho cả associated functions và namespaces từ modules. Chúng ta sẽ thảo luận modules trong [Chapter 7][modules].

### Multiple `impl` Blocks

Mỗi struct được phép có nhiều `impl` blocks. Ví dụ, Listing 5-15 tương đương code trong Listing 5-16, có mỗi method trong `impl` block riêng.

<Listing number="5-16" caption="Viết lại Listing 5-15 dùng nhiều `impl` blocks">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-16/src/main.rs:here}}
```

</Listing>

Không có lý do tách methods thành nhiều `impl` blocks ở đây, nhưng đây là syntax hợp lệ. Chúng ta sẽ thấy trường hợp nhiều `impl` blocks hữu ích trong Chapter 10, nơi thảo luận generic types và traits.

## Summary

Structs cho phép tạo custom types ý nghĩa cho domain của bạn. Bằng structs, bạn giữ các pieces data liên quan kết nối với nhau và đặt tên mỗi piece để code rõ ràng. Trong `impl` blocks, bạn định nghĩa functions liên kết với type, và methods là loại associated function cho phép chỉ định behavior của instances structs.

Nhưng structs không phải cách duy nhất tạo custom types: Hãy chuyển sang enum feature của Rust để thêm tool khác vào toolbox của bạn.

[enums]: ch06-00-enums.html
[trait-objects]: ch18-02-trait-objects.md
[public]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html