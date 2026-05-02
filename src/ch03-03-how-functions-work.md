## Functions

Functions rất phổ biến trong code Rust. Bạn đã thấy một trong những functions quan trọng nhất trong ngôn ngữ: function `main`, là điểm vào của nhiều chương trình. Bạn cũng đã thấy keyword `fn`, cho phép bạn khai báo các functions mới.

Code Rust dùng _snake case_ như là style quy ước cho function và tên variable, trong đó tất cả chữ cái đều viết thường và các từ được phân cách bằng dấu gạch dưới. Đây là một chương trình chứa ví dụ về function definition:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-16-functions/src/main.rs}}
```

Chúng ta định nghĩa một function trong Rust bằng cách nhập `fn` theo sau là tên function và một cặp dấu ngoặc đơn. Các dấu ngoặc nhọn cho compiler biết nơi function body bắt đầu và kết thúc.

Chúng ta có thể gọi bất kỳ function nào chúng ta đã định nghĩa bằng cách nhập tên của nó theo sau là cặp dấu ngoặc đơn. Vì `another_function` được định nghĩa trong chương trình, nó có thể được gọi từ bên trong function `main`. Lưu ý rằng chúng ta đã định nghĩa `another_function` _sau_ function `main` trong source code; chúng ta cũng có thể đã định nghĩa nó trước. Rust không quan tâm đến nơi bạn định nghĩa các functions, chỉ cần chúng được định nghĩa ở đâu đó trong một scope có thể được nhìn thấy bởi caller.

Hãy bắt đầu một binary project mới tên _functions_ để khám phá functions thêm. Đặt ví dụ `another_function` vào _src/main.rs_ và chạy nó. Bạn sẽ thấy output sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-16-functions/output.txt}}
```

Các dòng thực thi theo thứ tự chúng xuất hiện trong function `main`. Đầu tiên, thông báo "Hello, world!" được in, và sau đó `another_function` được gọi và thông báo của nó được in.

### Parameters

Chúng ta có thể định nghĩa functions có _parameters_, là các variables đặc biệt thuộc về signature của function. Khi một function có parameters, bạn có thể cung cấp cho nó các giá trị cụ thể cho những parameters đó. Về mặt kỹ thuật, các giá trị cụ thể được gọi là _arguments_, nhưng trong hội thoại thông thường, mọi người có xu hướng dùng các từ _parameter_ và _argument_ thay thế cho nhau để chỉ cả các variables trong định nghĩa của function lẫn các giá trị cụ thể được truyền vào khi bạn gọi một function.

Trong phiên bản `another_function` này chúng ta thêm một parameter:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/src/main.rs}}
```

Thử chạy chương trình này; bạn sẽ nhận được output sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/output.txt}}
```

Khai báo của `another_function` có một parameter tên `x`. Kiểu của `x` được chỉ định là `i32`. Khi chúng ta truyền `5` vào `another_function`, macro `println!` đặt `5` vào nơi có cặp dấu ngoặc nhọn chứa `x` trong format string.

Trong function signatures, bạn _phải_ khai báo kiểu của mỗi parameter. Đây là quyết định có chủ ý trong thiết kế của Rust: Yêu cầu type annotations trong function definitions có nghĩa là compiler hầu như không bao giờ cần bạn dùng chúng ở nơi khác trong code để tìm ra kiểu bạn muốn nói đến. Compiler cũng có thể đưa ra các thông báo lỗi hữu ích hơn nếu nó biết function mong đợi các kiểu nào.

Khi định nghĩa nhiều parameters, hãy phân tách các khai báo parameter bằng dấu phẩy, như sau:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/src/main.rs}}
```

Ví dụ này tạo một function tên `print_labeled_measurement` với hai parameters. Parameter đầu tiên tên `value` và là `i32`. Parameter thứ hai tên `unit_label` và kiểu là `char`. Function sau đó in text chứa cả `value` lẫn `unit_label`.

Hãy thử chạy code này. Thay thế chương trình hiện tại trong file _src/main.rs_ của project _functions_ bằng ví dụ trên và chạy nó dùng `cargo run`:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/output.txt}}
```

Vì chúng ta gọi function với `5` là giá trị cho `value` và `'h'` là giá trị cho `unit_label`, output của chương trình chứa những giá trị đó.

### Statements và Expressions

Function bodies được tạo thành từ một chuỗi các statements tùy chọn kết thúc bằng một expression. Cho đến nay, các functions chúng ta đã đề cập chưa bao gồm expression cuối, nhưng bạn đã thấy một expression là một phần của statement. Vì Rust là một ngôn ngữ dựa trên expression, đây là một sự phân biệt quan trọng cần hiểu. Các ngôn ngữ khác không có cùng sự phân biệt, vì vậy hãy cùng xem statements và expressions là gì và sự khác biệt của chúng ảnh hưởng đến bodies của functions như thế nào.

- _Statements_ là các instructions thực hiện một hành động nào đó và không trả về giá trị.
- _Expressions_ đánh giá thành một giá trị kết quả.

Hãy xem một số ví dụ.

Thực ra chúng ta đã dùng statements và expressions. Tạo một variable và gán một giá trị cho nó bằng keyword `let` là một statement. Trong Listing 3-1, `let y = 6;` là một statement.

<Listing number="3-1" file-name="src/main.rs" caption="Một khai báo function `main` chứa một statement">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-01/src/main.rs}}
```

</Listing>

Function definitions cũng là statements; toàn bộ ví dụ trên là một statement trong chính nó. (Như chúng ta sẽ thấy ngay sau đây, việc gọi một function không phải là statement.)

Statements không trả về giá trị. Do đó, bạn không thể gán một `let` statement cho một variable khác, như code sau đây cố làm; bạn sẽ nhận được lỗi:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/src/main.rs}}
```

Khi chạy chương trình này, lỗi bạn nhận được sẽ trông như sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/output.txt}}
```

Statement `let y = 6` không trả về giá trị, vì vậy không có gì để `x` gán vào. Điều này khác với những gì xảy ra trong các ngôn ngữ khác, chẳng hạn như C và Ruby, nơi assignment trả về giá trị của phép gán. Trong những ngôn ngữ đó, bạn có thể viết `x = y = 6` và cả `x` lẫn `y` đều có giá trị `6`; điều đó không xảy ra trong Rust.

Expressions đánh giá thành một giá trị và tạo thành phần lớn phần còn lại của code mà bạn sẽ viết trong Rust. Hãy xem một phép toán toán học, chẳng hạn như `5 + 6`, là một expression đánh giá thành giá trị `11`. Expressions có thể là một phần của statements: Trong Listing 3-1, `6` trong statement `let y = 6;` là một expression đánh giá thành giá trị `6`. Gọi một function là một expression. Gọi một macro là một expression. Một new scope block được tạo bằng dấu ngoặc nhọn là một expression, ví dụ:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-20-blocks-are-expressions/src/main.rs}}
```

Expression này:

```rust,ignore
{
    let x = 3;
    x + 1
}
```

là một block mà trong trường hợp này, đánh giá thành `4`. Giá trị đó được gán cho `y` như một phần của `let` statement. Lưu ý dòng `x + 1` không có dấu chấm phẩy ở cuối, khác với hầu hết các dòng bạn đã thấy cho đến nay. Expressions không bao gồm dấu chấm phẩy kết thúc. Nếu bạn thêm dấu chấm phẩy vào cuối của một expression, bạn biến nó thành statement, và nó sẽ không trả về giá trị. Hãy nhớ điều này khi bạn khám phá function return values và expressions tiếp theo.

### Functions với Return Values

Functions có thể trả về giá trị cho code gọi chúng. Chúng ta không đặt tên cho các return values, nhưng chúng ta phải khai báo kiểu của chúng sau một mũi tên (`->`). Trong Rust, return value của function đồng nghĩa với giá trị của expression cuối cùng trong block của body của function. Bạn có thể return sớm từ một function bằng cách dùng keyword `return` và chỉ định một giá trị, nhưng hầu hết các functions đều ngầm return expression cuối cùng. Đây là ví dụ về một function trả về giá trị:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/src/main.rs}}
```

Không có function calls, macros, hay thậm chí `let` statements trong function `five` — chỉ có số `5` một mình. Đó là một function hoàn toàn hợp lệ trong Rust. Lưu ý rằng return type của function cũng được chỉ định, là `-> i32`. Thử chạy code này; output sẽ trông như sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/output.txt}}
```

`5` trong `five` là return value của function, đó là lý do tại sao return type là `i32`. Hãy xem xét điều này chi tiết hơn. Có hai điểm quan trọng: Thứ nhất, dòng `let x = five();` cho thấy chúng ta đang dùng return value của một function để khởi tạo một variable. Vì function `five` trả về `5`, dòng đó tương đương với:

```rust
let x = 5;
```

Thứ hai, function `five` không có parameters và định nghĩa kiểu của return value, nhưng body của function là một `5` đơn độc không có dấu chấm phẩy vì nó là một expression có giá trị chúng ta muốn trả về.

Hãy xem một ví dụ khác:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-22-function-parameter-and-return/src/main.rs}}
```

Chạy code này sẽ in `The value of x is: 6`. Nhưng điều gì xảy ra nếu chúng ta đặt dấu chấm phẩy ở cuối dòng chứa `x + 1`, biến nó từ một expression thành một statement?

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/src/main.rs}}
```

Compile code này sẽ gây ra lỗi, như sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/output.txt}}
```

Thông báo lỗi chính, `mismatched types`, tiết lộ vấn đề cốt lõi với code này. Định nghĩa của function `plus_one` nói rằng nó sẽ trả về một `i32`, nhưng statements không đánh giá thành một giá trị, được biểu thị bởi `()`, kiểu unit. Do đó, không có gì được trả về, điều này mâu thuẫn với định nghĩa function và dẫn đến lỗi. Trong output này, Rust cung cấp một thông báo có thể giúp khắc phục vấn đề này: Nó đề xuất xóa dấu chấm phẩy, điều đó sẽ sửa lỗi.
