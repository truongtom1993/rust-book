## Advanced Types

Hệ thống kiểu Rust có một số tính năng mà chúng ta đã đề cập nhưng chưa thảo luận. Chúng ta sẽ bắt đầu bằng cách thảo luận newtypes nói chung khi chúng ta kiểm tra tại sao chúng hữu ích như là những loại. Sau đó, chúng ta sẽ chuyển đến type aliases, một tính năng tương tự như newtypes nhưng với ngữ nghĩa hơi khác. Chúng ta cũng sẽ thảo luận loại `!` và những loại được định kích thước động.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-newtype-pattern-for-type-safety-and-abstraction"></a>

### Type Safety and Abstraction with the Newtype Pattern

Phần này giả định rằng bạn đã đọc phần trước ["Implementing External Traits with the Newtype Pattern"][newtype]<!-- ignore -->. Newtype pattern cũng hữu ích cho những tác vụ ngoài những cái chúng ta đã thảo luận cho đến nay, bao gồm việc thực thi tĩnh rằng những giá trị không bao giờ được nhầm lẫn và chỉ ra những đơn vị của một giá trị. Bạn đã thấy một ví dụ của việc sử dụng newtypes để chỉ ra những đơn vị trong Listing 20-16: Gợi nhớ rằng những structs `Millimeters` và `Meters` bọc những giá trị `u32` trong một newtype. Nếu chúng ta viết một function với một tham số của loại `Millimeters`, chúng ta sẽ không thể compile một chương trình mà không cố gắng gọi function đó với một giá trị của loại `Meters` hoặc một `u32` thuần.

Chúng ta cũng có thể sử dụng newtype pattern để trừu tượng hóa một số chi tiết triển khai của một loại: Loại mới có thể thể hiện một public API mà khác với API của loại bên trong riêng tư.

Newtypes cũng có thể ẩn triển khai bên trong. Ví dụ, chúng ta có thể cung cấp một loại `People` để bọc một `HashMap<i32, String>` mà lưu trữ ID của một người được liên kết với tên của họ. Code sử dụng `People` sẽ chỉ tương tác với public API chúng ta cung cấp, chẳng hạn như một method để thêm một string tên vào collection `People`; code đó sẽ không cần phải biết rằng chúng ta gán một `i32` ID cho những tên trong nội bộ. Newtype pattern là một cách nhẹ để đạt được encapsulation để ẩn chi tiết triển khai, mà chúng ta đã thảo luận trong phần ["Encapsulation that Hides Implementation Details"][encapsulation-that-hides-implementation-details]<!-- ignore --> trong Chương 18.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-type-synonyms-with-type-aliases"></a>

### Type Synonyms and Type Aliases

Rust cung cấp khả năng để khai báo một _type alias_ để cung cấp một loại hiện có một tên khác. Để điều này chúng ta sử dụng từ khóa `type`. Ví dụ, chúng ta có thể tạo alias `Kilometers` cho `i32` như vậy:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:here}}
```

Bây giờ alias `Kilometers` là một _synonym_ cho `i32`; không giống như những loại `Millimeters` và `Meters` chúng ta tạo trong Listing 20-16, `Kilometers` không phải là một loại mới riêng, tinh tế. Những giá trị mà có loại `Kilometers` sẽ được xử lý giống như những giá trị của loại `i32`:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:there}}
```

Vì `Kilometers` và `i32` là cùng loại, chúng ta có thể thêm những giá trị của cả hai loại và có thể chuyển những giá trị `Kilometers` cho những functions mà lấy `i32` parameters. Tuy nhiên, sử dụng phương pháp này, chúng ta không nhận được những lợi ích kiểm tra loại mà chúng ta nhận được từ newtype pattern được thảo luận trước. Nói cách khác, nếu chúng ta nhầm lẫn `Kilometers` và `i32` những giá trị ở đâu đó, trình biên dịch sẽ không cung cấp cho chúng ta một error.

Trường hợp sử dụng chính cho những synonyms loại là để giảm lặp lại. Ví dụ, chúng ta có thể có một loại dài dòng như thế này:

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

Viết loại dài dòng này trong những function signatures và như những type annotations trên toàn bộ code có thể mệt mỏi và dễ bị lỗi. Hãy tưởng tượng có một dự án đầy code như thế trong Listing 20-25.

<Listing number="20-25" caption="Using a long type in many places">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-25/src/main.rs:here}}
```

</Listing>

Một type alias làm code này dễ quản lý hơn bằng cách giảm lặp lại. Trong Listing 20-26, chúng ta đã giới thiệu một alias được đặt tên là `Thunk` cho loại dài dòng và có thể thay thế tất cả các sử dụng của loại bằng alias ngắn hơn `Thunk`.

<Listing number="20-26" caption="Introducing a type alias, `Thunk`, to reduce repetition">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-26/src/main.rs:here}}
```

</Listing>

Code này dễ đọc và viết hơn nhiều! Chọn một tên có nghĩa cho một type alias có thể giúp giao tiếp ý định của bạn như là cũng (_thunk_ là một từ cho code để được đánh giá tại một thời gian sau, vì vậy nó là một tên thích hợp cho một closure mà được lưu trữ).

Type aliases cũng được sử dụng thường xuyên với loại `Result<T, E>` để giảm lặp lại. Xem xét module `std::io` trong thư viện chuẩn. Những hoạt động I/O thường xuyên trả về một `Result<T, E>` để xử lý tình huống khi những hoạt động không hoạt động. Thư viện này có một struct `std::io::Error` mà đại diện cho tất cả những lỗi I/O có thể. Nhiều functions trong `std::io` sẽ được trả về một `Result<T, E>` mà `E` là `std::io::Error`, chẳng hạn như những functions trong trait `Write`:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-05-write-trait/src/lib.rs}}
```

`Result<..., Error>` được lặp lại nhiều. Như vậy, `std::io` có khai báo type alias này:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:here}}
```

Vì khai báo này nằm trong module `std::io`, chúng ta có thể sử dụng alias đủ điều kiện `std::io::Result<T>`; đó là, một `Result<T, E>` với `E` được điền vào như `std::io::Error`. Những function signatures trait `Write` kết thúc trông giống như thế này:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:there}}
```

Type alias giúp theo hai cách: Nó làm code dễ dàng hơn để viết _và_ nó cung cấp cho chúng ta một interface nhất quán trên tất cả `std::io`. Vì nó là một alias, nó chỉ là một `Result<T, E>` khác, mà có nghĩa là chúng ta có thể sử dụng bất kỳ methods nào mà làm việc trên `Result<T, E>` với nó, cũng như syntax đặc biệt như toán tử `?`.

### The Never Type That Never Returns

Rust có một loại đặc biệt được đặt tên `!` mà được biết trong ngôn ngữ lý thuyết kiểu như _empty type_ vì nó không có những giá trị. Chúng ta thích gọi nó loại _never type_ vì nó đứng vào chỗ của return type khi một function sẽ không bao giờ trả về. Đây là một ví dụ:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-07-never-type/src/lib.rs:here}}
```

Code này được đọc là "function `bar` không bao giờ trả về." Những functions mà trả về never được gọi _diverging functions_. Chúng ta không thể tạo những giá trị của loại `!`, vì vậy `bar` không bao giờ có thể trả về.

Nhưng loại mà bạn không bao giờ có thể tạo những giá trị cho có những ích lợi gì? Nhớ lại code từ Listing 2-5, một phần của trò chơi đoán số; chúng ta đã tái tạo một chút của nó ở đây trong Listing 20-27.

<Listing number="20-27" caption="A `match` with an arm that ends in `continue`">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:ch19}}
```

</Listing>

Tại lúc đó, chúng ta đã bỏ qua một số chi tiết trong code này. Trong phần ["The `match` Control Flow Construct"][the-match-control-flow-construct]<!-- ignore --> trong Chương 6, chúng ta đã thảo luận rằng những arms `match` phải trả về cùng loại. Vì vậy, ví dụ, code sau không hoạt động:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-08-match-arms-different-types/src/main.rs:here}}
```

Loại của `guess` trong code này sẽ phải là một integer _và_ một string, và Rust yêu cầu rằng `guess` có chỉ một loại. Vì vậy, cái gì `continue` trả về? Làm thế nào chúng ta được cho phép để trả về một `u32` từ một arm và có một arm khác mà kết thúc với `continue` trong Listing 20-27?

Như bạn có thể đã đoán, `continue` có một giá trị `!`. Đó là, khi Rust tính toán loại của `guess`, nó nhìn vào cả hai match arms, former với một giá trị của `u32` và latter với một giá trị `!`. Vì `!` không bao giờ có thể có một giá trị, Rust quyết định loại của `guess` là `u32`.

Cách chính thức để mô tả hành vi này là những expressions của loại `!` có thể được ép buộc thành bất kỳ loại nào khác. Chúng ta được phép để kết thúc `match` arm này với `continue` vì `continue` không trả về một giá trị; thay vào đó, nó di chuyển kiểm soát trở lại đầu vòng lặp, vì vậy trong trường hợp `Err`, chúng ta không bao giờ gán một giá trị cho `guess`.

Loại never cũng hữu ích với macro `panic!`. Gọi lại function `unwrap` mà chúng ta gọi trên những giá trị `Option<T>` để tạo một giá trị hoặc panic với định nghĩa này:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-09-unwrap-definition/src/lib.rs:here}}
```

Trong code này, cái gì tương tự xảy ra như trong `match` trong Listing 20-27: Rust thấy rằng `val` có loại `T` và `panic!` có loại `!`, vì vậy kết quả của match expression tổng thể là `T`. Code này hoạt động vì `panic!` không tạo một giá trị; nó kết thúc chương trình. Trong trường hợp `None`, chúng ta sẽ không trả về một giá trị từ `unwrap`, vì vậy code này là hợp lệ.

Một expression cuối cùng mà có loại `!` là một loop:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-10-loop-returns-never/src/main.rs:here}}
```

Ở đây, vòng lặp không bao giờ kết thúc, vì vậy `!` là giá trị của expression. Tuy nhiên, điều này sẽ không đúng nếu chúng ta bao gồm một `break`, vì vòng lặp sẽ kết thúc khi nó đến `break`.

### Dynamically Sized Types and the `Sized` Trait

Rust cần biết những chi tiết cụ thể về những loại của nó, chẳng hạn như bao nhiêu không gian để phân bổ cho một giá trị của một loại cụ thể. Điều này để lại một góc của hệ thống kiểu của nó một chút nhầm lẫn lúc đầu: khái niệm của _dynamically sized types_. Đôi khi được gọi là _DSTs_ hoặc _unsized types_, những loại này cho phép chúng ta viết code sử dụng những giá trị có kích thước mà chúng ta có thể biết chỉ tại thời gian runtime.

Hãy đi vào chi tiết của một dynamically sized type được gọi là `str`, mà chúng ta đã sử dụng trong suốt sách. Đó là, không phải `&str`, nhưng `str` trên chính nó, là một DST. Trong nhiều trường hợp, chẳng hạn như khi lưu trữ văn bản được nhập bởi một người dùng, chúng ta không thể biết những bao lâu string là cho đến khi runtime. Điều đó có nghĩa là chúng ta không thể tạo một biến của loại `str`, cũng như chúng ta không thể lấy một đối số của loại `str`. Xem xét code sau, mà không hoạt động:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-11-cant-create-str/src/main.rs:here}}
```

Rust cần biết bao nhiêu bộ nhớ để phân bổ cho bất kỳ giá trị nào của một loại cụ thể, và tất cả những giá trị của một loại phải sử dụng cùng một lượng bộ nhớ. Nếu Rust cho phép chúng ta viết code này, những giá trị `str` này sẽ cần để lấy cùng một lượng không gian. Nhưng chúng có những độ dài khác nhau: `s1` cần 12 bytes lưu trữ và `s2` cần 15. Đây là lý do tại sao không thể tạo một biến giữ một dynamically sized type.

Vì vậy, chúng ta làm gì? Trong trường hợp này, bạn đã biết câu trả lời: Chúng ta tạo loại của `s1` và `s2` string slice (`&str`) thay vì `str`. Gọi lại từ phần ["String Slices"][string-slices]<!-- ignore --> trong Chương 4 rằng cấu trúc dữ liệu slice chỉ lưu trữ vị trí bắt đầu và độ dài của slice. Vì vậy, mặc dù `&T` là một giá trị duy nhất mà lưu trữ địa chỉ bộ nhớ của nơi `T` được định vị, một string slice là _hai_ giá trị: địa chỉ của `str` và độ dài của nó. Do đó, chúng ta có thể biết kích thước của một giá trị string slice tại thời gian compile: Nó là hai lần độ dài của một `usize`. Đó là, chúng ta luôn biết kích thước của một string slice, không quan trọng những bao lâu string nó đề cập đến là. Nói chung, đây là cách mà những dynamically sized types được sử dụng trong Rust: Chúng có một chút thêm metadata mà lưu trữ kích thước của thông tin động. Quy tắc vàng của những dynamically sized types là chúng ta phải luôn đặt những giá trị của những dynamically sized types phía sau một pointer của một số loại.

Chúng ta có thể kết hợp `str` với tất cả các loại pointers: ví dụ, `Box<str>` hoặc `Rc<str>`. Trong thực tế, bạn đã thấy điều này trước nhưng với một dynamically sized type khác: traits. Mỗi trait là một dynamically sized type chúng ta có thể tham khảo bằng cách sử dụng tên của trait. Trong phần ["Using Trait Objects to Abstract over Shared Behavior"][using-trait-objects-to-abstract-over-shared-behavior]<!-- ignore --> trong Chương 18, chúng ta đã đề cập rằng để sử dụng những traits như những trait objects, chúng ta phải đặt chúng phía sau một pointer, chẳng hạn như `&dyn Trait` hoặc `Box<dyn Trait>` (Rc<dyn Trait>` sẽ hoạt động cũng).

Để làm việc với DSTs, Rust cung cấp trait `Sized` để xác định xem kích thước của một loại có được biết tại thời gian compile hay không. Trait này được triển khai tự động cho mọi thứ có kích thước được biết tại thời gian compile. Ngoài ra, Rust ngầm thêm một ràng buộc trên `Sized` cho mỗi generic function. Đó là, một định nghĩa generic function như thế này:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-12-generic-fn-definition/src/lib.rs}}
```

được thực tế xử lý như thế này:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-13-generic-implicit-sized-bound/src/lib.rs}}
```

Theo mặc định, những generic functions sẽ hoạt động chỉ trên những loại mà có một kích thước được biết tại thời gian compile. Tuy nhiên, bạn có thể sử dụng syntax đặc biệt sau đây để nới lỏng hạn chế này:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-14-generic-maybe-sized/src/lib.rs}}
```

Một trait bound trên `?Sized` có nghĩa "`T` có thể hoặc không thể là `Sized`," và notation này ghi đè default rằng những loại generic phải có một kích thước được biết tại thời gian compile. Cú pháp `?Trait` với ý nghĩa này chỉ có sẵn cho `Sized`, không phải những traits khác.

Cũng lưu ý rằng chúng ta chuyển loại của tham số `t` từ `T` đến `&T`. Vì loại có thể không được `Sized`, chúng ta cần sử dụng nó phía sau một số loại con trỏ. Trong trường hợp này, chúng ta đã chọn một reference.

Tiếp theo, chúng ta sẽ nói về functions và closures!

[encapsulation-that-hides-implementation-details]: ch18-01-what-is-oo.html#encapsulation-that-hides-implementation-details
[string-slices]: ch04-03-slices.html#string-slices
[the-match-control-flow-construct]: ch06-02-match.html#the-match-control-flow-construct
[using-trait-objects-to-abstract-over-shared-behavior]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[newtype]: ch20-02-advanced-traits.html#implementing-external-traits-with-the-newtype-pattern
