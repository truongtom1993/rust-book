## Advanced Functions and Closures

Phần này khám phá một số tính năng nâng cao liên quan đến functions và closures, bao gồm function pointers và trả về closures.

### Function Pointers

Chúng ta đã nói về cách để chuyển closures cho functions; bạn cũng có thể chuyển regular functions cho functions! Kỹ thuật này hữu ích khi bạn muốn chuyển một function bạn đã định nghĩa thay vì định nghĩa một closure mới. Functions coerce to loại `fn` (với một lowercase _f_), không được nhầm lẫn với closure trait `Fn`. Loại `fn` được gọi là _function pointer_. Chuyển functions với function pointers sẽ cho phép bạn sử dụng functions như arguments cho những functions khác.

Cú pháp để chỉ định rằng một tham số là một function pointer tương tự như những closure, như được hiển thị trong Listing 20-28, mà chúng ta đã định nghĩa một function `add_one` mà thêm 1 vào tham số của nó. Function `do_twice` lấy hai tham số: một function pointer đến bất kỳ function nào mà lấy một `i32` parameter và trả về một `i32`, và một giá trị `i32`. Function `do_twice` gọi function `f` hai lần, chuyển nó giá trị `arg`, sau đó thêm hai kết quả lệnh gọi function lại với nhau. Function `main` gọi `do_twice` với những arguments `add_one` và `5`.

<Listing number="20-28" file-name="src/main.rs" caption="Using the `fn` type to accept a function pointer as an argument">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-28/src/main.rs}}
```

</Listing>

Code này in `The answer is: 12`. Chúng ta chỉ định rằng tham số `f` trong `do_twice` là một `fn` mà lấy một parameter của loại `i32` và trả về một `i32`. Chúng ta có thể sau đó gọi `f` trong thân của `do_twice`. Trong `main`, chúng ta có thể chuyển tên function `add_one` như một argument đầu tiên cho `do_twice`.

Không giống như closures, `fn` là một loại thay vì một trait, vì vậy chúng ta chỉ định `fn` như một loại parameter trực tiếp thay vì khai báo một tham số loại generic với một trong những `Fn` traits như một trait bound.

Function pointers triển khai tất cả ba của những closure traits (`Fn`, `FnMut`, và `FnOnce`), có nghĩa là bạn luôn có thể chuyển một function pointer như một argument cho một function mà yêu cầu một closure. Tốt nhất là để viết functions sử dụng một loại generic và một trong những closure traits vì vậy những functions của bạn có thể chấp nhận hoặc những functions hoặc những closures.

Đó là, một ví dụ của nơi bạn sẽ muốn để chỉ chấp nhận `fn` và không những closures là khi tương tác với external code mà không có closures: C functions có thể chấp nhận functions như arguments, nhưng C không có closures.

Như một ví dụ của nơi bạn có thể sử dụng một closure được định nghĩa inline hoặc một named function, hãy xem xét một sử dụng của `map` method được cung cấp bởi `Iterator` trait trong thư viện chuẩn. Để sử dụng `map` method để bật một vector của những số vào một vector của những strings, chúng ta có thể sử dụng một closure, như trong Listing 20-29.

<Listing number="20-29" caption="Using a closure with the `map` method to convert numbers to strings">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-29/src/main.rs:here}}
```

</Listing>

Hoặc chúng ta có thể đặt tên một function như một argument cho `map` thay vì closure. Listing 20-30 cho thấy cái này sẽ trông như thế nào.

<Listing number="20-30" caption="Using the `String::to_string` function with the `map` method to convert numbers to strings">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-30/src/main.rs:here}}
```

</Listing>

Lưu ý rằng chúng ta phải sử dụng fully qualified syntax mà chúng ta đã nói về trong phần ["Advanced Traits"][advanced-traits]<!-- ignore --> vì có những functions có sẵn nhiều được đặt tên là `to_string`.

Ở đây, chúng ta sử dụng function `to_string` được định nghĩa trong trait `ToString`, mà thư viện chuẩn đã triển khai cho bất kỳ loại nào mà triển khai `Display`.

Gọi lại từ phần ["Enum Values"][enum-values]<!-- ignore --> trong Chương 6 rằng tên của mỗi enum variant mà chúng ta định nghĩa cũng trở thành một initializer function. Chúng ta có thể sử dụng những initializer functions này như những function pointers mà triển khai những closure traits, mà có nghĩa là chúng ta có thể chỉ định những initializer functions như những arguments cho những methods mà lấy những closures, như được thấy trong Listing 20-31.

<Listing number="20-31" caption="Using an enum initializer with the `map` method to create a `Status` instance from numbers">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-31/src/main.rs:here}}
```

</Listing>

Ở đây, chúng ta tạo những instances `Status::Value` sử dụng mỗi `u32` giá trị trong phạm vi mà `map` được gọi trên bằng cách sử dụng initializer function của `Status::Value`. Một số người thích cách này style và một số người thích sử dụng closures. Chúng compile để cùng code, vì vậy sử dụng cái nào style là rõ ràng cho bạn.

### Returning Closures

Closures được đại diện bởi traits, mà có nghĩa là bạn không thể trả về closures trực tiếp. Trong hầu hết những trường hợp nơi bạn có thể muốn để trả về một trait, bạn có thể thay vào đó sử dụng loại cụ thể mà triển khai trait như return value của function. Tuy nhiên, bạn thường không thể làm điều đó với closures vì chúng không có một loại cụ thể mà là returnable; bạn không được phép sử dụng function pointer `fn` như một return type nếu closure captures bất kỳ những giá trị nào từ scope của nó, ví dụ.

Thay vào đó, bạn sẽ thường sử dụng `impl Trait` syntax chúng ta học về trong Chương 10. Bạn có thể trả về bất kỳ loại function nào, sử dụng `Fn`, `FnOnce`, và `FnMut`. Ví dụ, code trong Listing 20-32 sẽ compile mà không gặp sự cố.

<Listing number="20-32" caption="Returning a closure from a function using the `impl Trait` syntax">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-32/src/lib.rs}}
```

</Listing>

Tuy nhiên, như chúng ta đã lưu ý trong phần ["Inferring and Annotating Closure Types"][closure-types]<!-- ignore --> trong Chương 13, mỗi closure cũng là loại riêng biệt của nó. Nếu bạn cần để làm việc với những functions nhiều mà có cùng signature nhưng những triển khai khác nhau, bạn sẽ cần để sử dụng một trait object cho chúng. Xem xét cái gì xảy ra nếu bạn viết code giống như những được hiển thị trong Listing 20-33.

<Listing file-name="src/main.rs" number="20-33" caption="Creating a `Vec<T>` of closures defined by functions that return `impl Fn` types">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-33/src/main.rs}}
```

</Listing>

Ở đây chúng ta có hai functions, `returns_closure` và `returns_initialized_closure`, mà cả hai trả về `impl Fn(i32) -> i32`. Lưu ý rằng những closures mà chúng trả về là khác nhau, thậm chí mặc dù chúng triển khai cùng loại. Nếu chúng ta cố gắng compile điều này, Rust để chúng ta biết rằng nó sẽ không hoạt động:

```text
{{#include ../listings/ch20-advanced-features/listing-20-33/output.txt}}
```

Thông báo lỗi nói với chúng ta rằng bất cứ khi nào chúng ta trả về một `impl Trait`, Rust tạo một unique _opaque type_, một loại mà chúng ta không thể nhìn vào những chi tiết của những gì Rust tạo cho chúng ta, cũng như chúng ta có thể không đoán loại Rust sẽ tạo để viết chúng ta. Vì vậy, thậm chí mặc dù những functions này trả về những closures mà triển khai cùng trait, `Fn(i32) -> i32`, những opaque types mà Rust tạo cho mỗi là khác nhau. (Đây tương tự như cách Rust tạo những loại cụ thể khác nhau cho những distinct async blocks thậm chí khi chúng có cùng output type, như chúng ta thấy trong ["The `Pin` Type and the `Unpin` Trait"][future-types]<!-- ignore --> trong Chương 17.) Chúng ta đã thấy một giải pháp cho vấn đề này một vài lần bây giờ: Chúng ta có thể sử dụng một trait object, như trong Listing 20-34.

<Listing number="20-34" caption="Creating a `Vec<T>` of closures defined by functions that return `Box<dyn Fn>` so that they have the same type">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-34/src/main.rs:here}}
```

</Listing>

Code này sẽ compile mà không gặp sự cố. Để nhiều về những trait objects, xem phần ["Using Trait Objects To Abstract over Shared Behavior"][trait-objects]<!-- ignore --> trong Chương 18.

Tiếp theo, chúng ta hãy xem xét macros!

[advanced-traits]: ch20-02-advanced-traits.html#advanced-traits
[enum-values]: ch06-01-defining-an-enum.html#enum-values
[closure-types]: ch13-01-closures.html#closure-type-inference-and-annotation
[future-types]: ch17-03-more-futures.html
[trait-objects]: ch18-02-trait-objects.html
