## `RefCell<T>` và Pattern Interior Mutability

_Interior mutability_ là một design pattern trong Rust cho phép bạn thay đổi dữ liệu ngay cả khi có những immutable reference đến dữ liệu đó; bình thường, hành động này bị không cho phép bởi các quy tắc borrowing. Để thay đổi dữ liệu, pattern này sử dụng code `unsafe` bên trong một data structure để uốn các quy tắc thông thường của Rust về mutation và borrowing. Unsafe code cho compiler biết rằng chúng ta đang kiểm tra các quy tắc theo cách thủ công thay vì dựa vào compiler để kiểm tra cho chúng ta; chúng ta sẽ thảo luận về unsafe code chi tiết hơn trong Chapter 20.

Chúng ta chỉ có thể sử dụng các type sử dụng interior mutability pattern khi chúng ta có thể đảm bảo rằng các quy tắc borrowing sẽ được tuân theo tại runtime, mặc dù compiler không thể đảm bảo điều đó. Code `unsafe` liên quan sau đó được bọc trong một safe API, và type bên ngoài vẫn là immutable.

Hãy khám phá concept này bằng cách xem xét type `RefCell<T>` tuân theo interior mutability pattern.

<!-- Old headings. Do not remove or links may break. -->

<a id="enforcing-borrowing-rules-at-runtime-with-refcellt"></a>

### Thực thi Borrowing Rules tại Runtime

Không giống như `Rc<T>`, type `RefCell<T>` đại diện cho single ownership trên dữ liệu mà nó giữ. Vậy, điều gì làm cho `RefCell<T>` khác với một type như `Box<T>`? Hãy nhớ lại các quy tắc borrowing bạn đã học trong Chapter 4:

- Tại bất kỳ thời điểm nào, bạn có thể có _hoặc_ một mutable reference hoặc bất kỳ số lượng immutable reference nào (nhưng không cả hai).
- Reference phải luôn luôn hợp lệ.

Với references và `Box<T>`, các invariant của borrowing rules được thực thi tại compile time. Với `RefCell<T>`, các invariant này được thực thi _tại runtime_. Với references, nếu bạn phá vỡ các quy tắc này, bạn sẽ nhận được một compiler error. Với `RefCell<T>`, nếu bạn phá vỡ các quy tắc này, chương trình của bạn sẽ panic và thoát.

Những lợi thế của việc kiểm tra các borrowing rules tại compile time là các lỗi sẽ được phát hiện sớm hơn trong quá trình phát triển, và không có ảnh hưởng đến runtime performance vì tất cả phân tích được hoàn thành trước đó. Vì những lý do đó, kiểm tra các borrowing rules tại compile time là sự lựa chọn tốt nhất trong hầu hết các trường hợp, đây là lý do tại sao đây là mặc định của Rust.

Lợi thế của việc kiểm tra các borrowing rules tại runtime thay vào đó là những tình huống memory-safe nhất định được cho phép, nơi chúng sẽ bị không cho phép bởi các kiểm tra compile-time. Static analysis, như Rust compiler, về cơ bản là bảo thủ. Một số thuộc tính của code là không thể phát hiện bằng cách phân tích code: Ví dụ nổi tiếng nhất là Halting Problem, vượt ra ngoài phạm vi của cuốn sách này nhưng là một topic thú vị để nghiên cứu.

Vì một số phân tích là không thể, nếu Rust compiler không thể chắc chắn rằng code tuân thủ các quy tắc ownership, nó có thể từ chối một chương trình đúng; theo cách này, nó là bảo thủ. Nếu Rust chấp nhận một chương trình không đúng, người dùng sẽ không thể tin tưởng các đảm bảo mà Rust đưa ra. Tuy nhiên, nếu Rust từ chối một chương trình đúng, lập trình viên sẽ bị làm phiền, nhưng không có gì thảm khốc có thể xảy ra. Type `RefCell<T>` hữu ích khi bạn chắc chắn rằng code của bạn tuân theo các borrowing rules nhưng compiler không thể hiểu và đảm bảo điều đó.

Giống như `Rc<T>`, `RefCell<T>` chỉ dành để sử dụng trong các tình huống single-threaded và sẽ cho bạn một compile-time error nếu bạn cố gắng sử dụng nó trong context multithreaded. Chúng ta sẽ nói về cách có được functionality của `RefCell<T>` trong một multithreaded program trong Chapter 16.

Dưới đây là tóm tắt những lý do để chọn `Box<T>`, `Rc<T>`, hoặc `RefCell<T>`:

- `Rc<T>` cho phép multiple owners của cùng một dữ liệu; `Box<T>` và `RefCell<T>` có single owners.
- `Box<T>` cho phép immutable hoặc mutable borrows được kiểm tra tại compile time; `Rc<T>` chỉ cho phép immutable borrows được kiểm tra tại compile time; `RefCell<T>` cho phép immutable hoặc mutable borrows được kiểm tra tại runtime.
- Bởi vì `RefCell<T>` cho phép mutable borrows được kiểm tra tại runtime, bạn có thể thay đổi giá trị bên trong `RefCell<T>` ngay cả khi `RefCell<T>` là immutable.

Thay đổi giá trị bên trong một immutable value là interior mutability pattern. Hãy xem xét một tình huống mà interior mutability hữu ích và kiểm tra cách nó có thể được thực hiện.

<!-- Old headings. Do not remove or links may break. -->

<a id="interior-mutability-a-mutable-borrow-to-an-immutable-value"></a>

### Sử dụng Interior Mutability

Một hệ quả của các borrowing rules là khi bạn có một immutable value, bạn không thể mượn nó theo cách mutable. Ví dụ, code này sẽ không compile:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/src/[main.rs](http://main.rs)}}
```

Nếu bạn cố gắng compile code này, bạn sẽ nhận được lỗi sau:

```console
{{#include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/output.txt}}
```

Tuy nhiên, có những tình huống trong đó sẽ hữu ích nếu một value tự thay đổi trong các method của nó nhưng xuất hiện immutable đối với code khác. Code bên ngoài các method của value không thể thay đổi value. Sử dụng `RefCell<T>` là một cách để có khả năng có interior mutability, nhưng `RefCell<T>` không vượt qua các borrowing rules hoàn toàn: Borrow checker trong compiler cho phép interior mutability này, và các borrowing rules được kiểm tra tại runtime thay vào đó. Nếu bạn vi phạm các quy tắc, bạn sẽ nhận được một `panic!` thay vì một compiler error.

Hãy làm việc thông qua một ví dụ thực tế nơi chúng ta có thể sử dụng `RefCell<T>` để thay đổi một immutable value và xem tại sao điều đó hữu ích.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-use-case-for-interior-mutability-mock-objects"></a>

#### Kiểm tra với Mock Objects

Đôi khi trong quá trình kiểm tra, một lập trình viên sẽ sử dụng một type thay thế cho một type khác, để quan sát hành vi cụ thể và khẳng định rằng nó được triển khai chính xác. Type placeholder này được gọi là _test double_. Hãy nghĩ về nó theo nghĩa của một stunt double trong filmmaking, nơi một người bước vào và thay thế cho một actor để làm một scene đặc biệt khó khăn. Test doubles đứng thay cho các type khác khi chúng ta đang chạy tests. _Mock objects_ là các type cụ thể của test doubles ghi lại những gì xảy ra trong quá trình test để bạn có thể khẳng định rằng các hành động đúng đã diễn ra.

Rust không có objects theo cùng ý nghĩa như các ngôn ngữ khác có objects, và Rust không có mock object functionality được tích hợp trong standard library như một số ngôn ngữ khác có. Tuy nhiên, bạn chắc chắn có thể tạo một struct sẽ phục vụ cùng các mục đích như một mock object.

Đây là tình huống chúng ta sẽ kiểm tra: Chúng ta sẽ tạo một library theo dõi một value so với một maximum value và gửi messages dựa trên mức độ gần với maximum value mà current value là. Library này có thể được sử dụng để theo dõi quota của người dùng cho số lượng API calls mà họ được phép thực hiện, ví dụ.

Library của chúng ta sẽ chỉ cung cấp functionality của việc theo dõi mức độ gần với maximum của một value là bao nhiêu và những messages nào sẽ có tại những lúc nào. Các applications sử dụng library của chúng ta sẽ được dự kiến cung cấp cơ chế để gửi messages: Application có thể hiển thị message cho người dùng trực tiếp, gửi một email, gửi một text message, hoặc làm cái gì đó khác. Library không cần biết chi tiết đó. Tất cả những gì nó cần là cái gì đó triển khai một trait mà chúng ta sẽ cung cấp, gọi là `Messenger`. Listing 15-20 hiển thị code của library.

<Listing number="15-20" file-name="src/lib.rs" caption="A library to keep track of how close a value is to a maximum value and warn when the value is at certain levels">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-20/src/lib.rs}}
```

</Listing>

Một phần quan trọng của code này là trait `Messenger` có một method gọi là `send` nhận một immutable reference đến `self` và text của message. Trait này là interface mà mock object của chúng ta cần triển khai để mock có thể được sử dụng theo cách mà một real object. Phần quan trọng khác là chúng ta muốn kiểm tra hành vi của method `set_value` trên `LimitTracker`. Chúng ta có thể thay đổi những gì chúng ta truyền vào cho parameter `value`, nhưng `set_value` không trả về bất cứ điều gì để chúng ta đưa ra assertions. Chúng ta muốn có thể nói rằng nếu chúng ta tạo một `LimitTracker` với cái gì đó triển khai trait `Messenger` và một giá trị cụ thể cho `max`, thì messenger được bảo để gửi các appropriate messages khi chúng ta truyền các số khác nhau cho `value`.

Chúng ta cần một mock object mà, thay vì gửi một email hoặc text message khi chúng ta gọi `send`, sẽ chỉ theo dõi các messages mà nó được bảo để gửi. Chúng ta có thể tạo một new instance của mock object, tạo một `LimitTracker` sử dụng mock object, gọi method `set_value` trên `LimitTracker`, và sau đó kiểm tra rằng mock object có các messages mà chúng ta mong đợi. Listing 15-21 hiển thị một attempt để triển khai một mock object để làm chính xác điều đó, nhưng borrow checker sẽ không cho phép nó.

<Listing number="15-21" file-name="src/lib.rs" caption="Một attempt để triển khai một `MockMessenger` không được cho phép bởi borrow checker">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-21/src/lib.rs:here}}
```

</Listing>

Code test này định nghĩa một struct `MockMessenger` có một field `sent_messages` với một `Vec` của các `String` values để theo dõi các messages mà nó được bảo để gửi. Chúng ta cũng định nghĩa một associated function `new` để thuận tiện trong việc tạo các new `MockMessenger` values bắt đầu với một empty list các messages. Chúng ta sau đó triển khai trait `Messenger` cho `MockMessenger` để chúng ta có thể cho một `MockMessenger` đến một `LimitTracker`. Trong định nghĩa của method `send`, chúng ta nhận message được truyền vào như một parameter và lưu trữ nó trong list `sent_messages` của `MockMessenger`.

Trong test, chúng ta đang kiểm tra những gì xảy ra khi `LimitTracker` được bảo để đặt `value` thành cái gì đó nhiều hơn 75 phần trăm của giá trị `max`. Đầu tiên, chúng ta tạo một new `MockMessenger`, sẽ bắt đầu với một empty list các messages. Sau đó, chúng ta tạo một new `LimitTracker` và cho nó một reference đến new `MockMessenger` và một `max` value của `100`. Chúng ta gọi method `set_value` trên `LimitTracker` với một value của `80`, nhiều hơn 75 phần trăm của 100. Sau đó, chúng ta assert rằng list của các messages mà `MockMessenger` đang theo dõi bây giờ phải có một message trong đó.

Tuy nhiên, có một vấn đề với test này, như được hiển thị ở đây:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-21/output.txt}}
```

Chúng ta không thể sửa `MockMessenger` để theo dõi các messages, bởi vì method `send` nhận một immutable reference đến `self`. Chúng ta cũng không thể lấy đề xuất từ text lỗi để sử dụng `&mut self` trong cả impl method và trait definition. Chúng ta không muốn thay đổi trait `Messenger` chỉ vì sake của testing. Thay vào đó, chúng ta cần tìm một cách để làm cho code test của chúng ta hoạt động chính xác với design hiện tại của chúng ta.

Đây là một tình huống trong đó interior mutability có thể giúp! Chúng ta sẽ lưu trữ `sent_messages` bên trong một `RefCell<T>`, và sau đó method `send` sẽ có thể sửa `sent_messages` để lưu trữ các messages mà chúng ta đã thấy. Listing 15-22 hiển thị cái đó trông như thế nào.

<Listing number="15-22" file-name="src/lib.rs" caption="Sử dụng `RefCell<T>` để thay đổi một inner value trong khi outer value được coi là immutable">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-22/src/lib.rs:here}}
```

</Listing>

Field `sent_messages` bây giờ là type `RefCell<Vec<String>>` thay vì `Vec<String>`. Trong function `new`, chúng ta tạo một new instance `RefCell<Vec<String>>` xung quanh empty vector.

Để triển khai method `send`, parameter đầu tiên vẫn là một immutable borrow của `self`, phù hợp với trait definition. Chúng ta gọi `borrow_mut` trên `RefCell<Vec<String>>` trong `self.sent_messages` để có được một mutable reference đến giá trị bên trong `RefCell<Vec<String>>`, đó là vector. Sau đó, chúng ta có thể gọi `push` trên mutable reference đến vector để theo dõi các messages được gửi trong quá trình test.

Sự thay đổi cuối cùng chúng ta phải làm là trong assertion: Để xem có bao nhiêu items trong inner vector, chúng ta gọi `borrow` trên `RefCell<Vec<String>>` để có được một immutable reference đến vector.

Bây giờ bạn đã thấy cách sử dụng `RefCell<T>`, hãy đào sâu vào cách nó hoạt động!

<!-- Old headings. Do not remove or links may break. -->

<a id="keeping-track-of-borrows-at-runtime-with-refcellt"></a>

#### Theo Dõi Borrows tại Runtime

Khi tạo immutable và mutable references, chúng ta sử dụng cú pháp `&` và `&mut` tương ứng. Với `RefCell<T>`, chúng ta sử dụng các methods `borrow` và `borrow_mut`, là một phần của safe API thuộc về `RefCell<T>`. Method `borrow` trả về smart pointer type `Ref<T>`, và `borrow_mut` trả về smart pointer type `RefMut<T>`. Cả hai types triển khai `Deref`, vì vậy chúng ta có thể coi chúng như các regular references.

`RefCell<T>` theo dõi bao nhiêu smart pointers `Ref<T>` và `RefMut<T>` hiện đang active. Mỗi lần chúng ta gọi `borrow`, `RefCell<T>` tăng count của bao nhiêu immutable borrows đang active. Khi một value `Ref<T>` đi ra ngoài scope, count của immutable borrows giảm đi 1. Giống như compile-time borrowing rules, `RefCell<T>` cho phép chúng ta có nhiều immutable borrows hoặc một mutable borrow tại bất kỳ điểm nào trong thời gian.

Nếu chúng ta cố gắng vi phạm các quy tắc này, thay vì nhận được một compiler error như chúng ta sẽ nhận với references, triển khai `RefCell<T>` sẽ panic tại runtime. Listing 15-23 hiển thị một modification của triển khai method `send` trong Listing 15-22. Chúng ta đang cố ý cố gắng tạo hai mutable borrows active cho cùng một scope để minh họa rằng `RefCell<T>` ngăn chúng ta làm điều này tại runtime.

<Listing number="15-23" file-name="src/lib.rs" caption="Tạo hai mutable references trong cùng một scope để thấy rằng `RefCell<T>` sẽ panic">

```rust,ignore,panics
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-23/src/lib.rs:here}}
```

</Listing>

Chúng ta tạo một variable `one_borrow` cho smart pointer `RefMut<T>` được trả về từ `borrow_mut`. Sau đó, chúng ta tạo một mutable borrow khác theo cùng cách trong variable `two_borrow`. Điều này tạo ra hai mutable references trong cùng một scope, không được cho phép. Khi chúng ta chạy tests cho library của chúng ta, code trong Listing 15-23 sẽ compile mà không có bất kỳ errors, nhưng test sẽ fail:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-23/output.txt}}
```

Lưu ý rằng code đã panic với message `already borrowed: BorrowMutError`. Đây là cách `RefCell<T>` xử lý các violations của borrowing rules tại runtime.

Chọn để catch borrowing errors tại runtime thay vì compile time, như chúng ta đã làm ở đây, có nghĩa là bạn có khả năng tìm thấy các mistakes trong code của bạn sau hơn trong quá trình phát triển: có thể không cho đến khi code của bạn được deployed tới production. Ngoài ra, code của bạn sẽ gánh chịu một nhỏ runtime performance penalty như một kết quả của việc theo dõi các borrows tại runtime thay vì compile time. Tuy nhiên, sử dụng `RefCell<T>` làm cho nó có thể viết một mock object có thể sửa chính nó để theo dõi các messages mà nó đã thấy trong khi bạn đang sử dụng nó trong một context nơi chỉ immutable values được cho phép. Bạn có thể sử dụng `RefCell<T>` bất chấp những trade-offs của nó để có được nhiều functionality hơn những regular references cung cấp.

<!-- Old headings. Do not remove or links may break. -->

<a id="having-multiple-owners-of-mutable-data-by-combining-rc-t-and-ref-cell-t"></a>
<a id="allowing-multiple-owners-of-mutable-data-with-rct-and-refcellt"></a>

### Cho Phép Multiple Owners của Mutable Data

Một cách phổ biến để sử dụng `RefCell<T>` là trong kết hợp với `Rc<T>`. Nhớ lại rằng `Rc<T>` cho phép bạn có multiple owners của một số dữ liệu, nhưng nó chỉ cung cấp immutable access đến dữ liệu đó. Nếu bạn có một `Rc<T>` giữ một `RefCell<T>`, bạn có thể có được một value có thể có multiple owners _và_ mà bạn có thể thay đổi!

Ví dụ, nhớ lại example cons list trong Listing 15-18 nơi chúng ta sử dụng `Rc<T>` để cho phép multiple lists chia sẻ ownership của một list khác. Bởi vì `Rc<T>` chỉ giữ immutable values, chúng ta không thể thay đổi bất kỳ các values nào trong list sau khi chúng ta đã tạo chúng. Hãy thêm vào `RefCell<T>` cho khả năng của nó để thay đổi các values trong các lists. Listing 15-24 hiển thị rằng bằng cách sử dụng một `RefCell<T>` trong định nghĩa `Cons`, chúng ta có thể sửa value được lưu trữ trong tất cả các lists.

<Listing number="15-24" file-name="src/main.rs" caption="Sử dụng `Rc<RefCell<i32>>` để tạo một `List` mà chúng ta có thể thay đổi">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-24/src/main.rs}}
```

</Listing>

Chúng ta tạo một value là một instance của `Rc<RefCell<i32>>` và lưu trữ nó trong một variable được đặt tên là `value` để chúng ta có thể truy cập nó trực tiếp sau. Sau đó, chúng ta tạo một `List` trong `a` với một `Cons` variant giữ `value`. Chúng ta cần clone `value` để cả `a` và `value` đều có ownership của inner `5` value thay vì chuyển ownership từ `value` đến `a` hoặc có `a` borrow từ `value`.

Chúng ta bọc list `a` trong một `Rc<T>` để khi chúng ta tạo lists `b` và `c`, chúng cả đều có thể tham chiếu đến `a`, đó là những gì chúng ta đã làm trong Listing 15-18.

Sau khi chúng ta đã tạo các lists trong `a`, `b`, và `c`, chúng ta muốn thêm 10 vào value trong `value`. Chúng ta làm điều này bằng cách gọi `borrow_mut` trên `value`, sử dụng automatic dereferencing feature mà chúng ta đã thảo luận trong ["Toán tử `->` ở đâu?"][wheres-the---operator]<!-- ignore --> trong Chapter 5 để dereference `Rc<T>` đến inner `RefCell<T>` value. Method `borrow_mut` trả về một smart pointer `RefMut<T>`, và chúng ta sử dụng dereference operator trên nó và thay đổi inner value.

Khi chúng ta in `a`, `b`, và `c`, chúng ta có thể thấy rằng chúng tất cả đều có modified value của `15` thay vì `5`:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-24/output.txt}}
```

Technique này khá gọn! Bằng cách sử dụng `RefCell<T>`, chúng ta có một outwardly immutable `List` value. Nhưng chúng ta có thể sử dụng các methods trên `RefCell<T>` cung cấp access đến interior mutability của nó để chúng ta có thể sửa dữ liệu của chúng ta khi chúng ta cần đến. Runtime checks của borrowing rules bảo vệ chúng ta khỏi data races, và đôi khi nó đáng để trade một chút speed cho sự linh hoạt này trong data structures của chúng ta. Lưu ý rằng `RefCell<T>` không hoạt động cho multithreaded code! `Mutex<T>` là thread-safe version của `RefCell<T>`, và chúng ta sẽ thảo luận `Mutex<T>` trong Chapter 16.

[wheres-the---operator]: ch05-03-method-syntax.html#wheres-the---operator
