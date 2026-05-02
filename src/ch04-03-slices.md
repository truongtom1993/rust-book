## Kiểu Slice

_Slices_ cho phép bạn tham chiếu đến một chuỗi liên tiếp các phần tử trong một
[collection](ch08-00-common-collections.md)<!-- ignore -->. Một slice là một loại reference, vì vậy nó không có ownership.

Đây là một bài toán lập trình nhỏ: Viết một function nhận một chuỗi các từ được phân tách bằng dấu cách và trả về từ đầu tiên nó tìm thấy trong chuỗi đó. Nếu function không tìm thấy dấu cách trong chuỗi, toàn bộ chuỗi phải là một từ, vì vậy toàn bộ chuỗi nên được trả về.

> Lưu ý: Để giới thiệu slices, chúng ta giả định chỉ có ASCII trong
> phần này; một thảo luận kỹ lưỡng hơn về xử lý UTF-8 có trong phần
> ["Storing UTF-8 Encoded Text with Strings"][strings]<!-- ignore -->
> của Chương 8.

Hãy cùng xem cách chúng ta sẽ viết signature của function này mà không sử dụng slices, để hiểu vấn đề mà slices sẽ giải quyết:

```rust,ignore
fn first_word(s: &String) -> ?
```

Function `first_word` có một tham số kiểu `&String`. Chúng ta không cần ownership, vì vậy điều này ổn. (Theo cách idiomatic trong Rust, các functions không lấy ownership của các tham số trừ khi chúng cần, và lý do cho điều đó sẽ trở nên rõ ràng khi chúng ta tiếp tục.) Nhưng chúng ta nên trả về gì? Chúng ta thực sự không có cách nào để nói về *một phần* của một string. Tuy nhiên, chúng ta có thể trả về index của cuối từ, được chỉ ra bằng một dấu cách. Hãy thử cách đó, như được hiển thị trong Listing 4-7.

<Listing number="4-7" file-name="src/main.rs" caption="Function `first_word` trả về một giá trị byte index vào tham số `String`">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:here}}
```

</Listing>

Vì chúng ta cần đi qua `String` từng phần tử và kiểm tra xem một giá trị có phải là dấu cách không, chúng ta sẽ convert `String` của mình thành một mảng bytes bằng cách sử dụng method `as_bytes`.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:as_bytes}}
```

Tiếp theo, chúng ta tạo một iterator trên mảng bytes bằng cách sử dụng method `iter`:

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:iter}}
```

Chúng ta sẽ thảo luận về iterators chi tiết hơn trong [Chương 13][ch13]<!-- ignore -->. Bây giờ, hãy biết rằng `iter` là một method trả về từng phần tử trong một collection và `enumerate` bọc kết quả của `iter` và trả về từng phần tử như một phần của tuple. Phần tử đầu tiên của tuple được trả về từ `enumerate` là index, và phần tử thứ hai là một reference đến phần tử. Điều này tiện lợi hơn một chút so với tự tính index.

Vì method `enumerate` trả về một tuple, chúng ta có thể sử dụng patterns để destructure tuple đó. Chúng ta sẽ thảo luận nhiều hơn về patterns trong [Chương 6][ch6]<!-- ignore -->. Trong vòng lặp `for`, chúng ta chỉ định một pattern có `i` cho index trong tuple và `&item` cho byte đơn trong tuple. Vì chúng ta nhận một reference đến phần tử từ `.iter().enumerate()`, chúng ta sử dụng `&` trong pattern.

Bên trong vòng lặp `for`, chúng ta tìm kiếm byte đại diện cho dấu cách bằng cách sử dụng cú pháp byte literal. Nếu chúng ta tìm thấy một dấu cách, chúng ta trả về vị trí. Ngược lại, chúng ta trả về độ dài của string bằng cách sử dụng `s.len()`.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:inside_for}}
```

Bây giờ chúng ta có cách tìm ra index của cuối từ đầu tiên trong string, nhưng có một vấn đề. Chúng ta đang trả về một `usize` một mình, nhưng nó chỉ là một số có nghĩa trong ngữ cảnh của `&String`. Nói cách khác, vì đây là một giá trị riêng biệt từ `String`, không có gì đảm bảo rằng nó vẫn sẽ hợp lệ trong tương lai. Hãy xem xét chương trình trong Listing 4-8 sử dụng function `first_word` từ Listing 4-7.

<Listing number="4-8" file-name="src/main.rs" caption="Lưu kết quả từ việc gọi function `first_word` và sau đó thay đổi nội dung `String`">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-08/src/main.rs:here}}
```

</Listing>

Chương trình này compile mà không có bất kỳ lỗi nào và cũng sẽ làm như vậy nếu chúng ta sử dụng `word` sau khi gọi `s.clear()`. Vì `word` không được kết nối với trạng thái của `s` chút nào, `word` vẫn chứa giá trị `5`. Chúng ta có thể sử dụng giá trị `5` đó với biến `s` để cố gắng trích xuất từ đầu tiên, nhưng đây sẽ là một bug vì nội dung của `s` đã thay đổi kể từ khi chúng ta lưu `5` trong `word`.

Phải lo lắng về việc index trong `word` bị lệch so với dữ liệu trong `s` là tẻ nhạt và dễ xảy ra lỗi! Quản lý các index này còn fragile hơn nếu chúng ta viết một function `second_word`. Signature của nó phải trông như thế này:

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

Bây giờ chúng ta đang theo dõi index bắt đầu _và_ kết thúc, và chúng ta có ngày càng nhiều giá trị được tính toán từ dữ liệu ở trạng thái cụ thể nhưng không được gắn kết với trạng thái đó chút nào. Chúng ta có ba biến không liên quan đang nổi trôi cần được giữ đồng bộ.

May mắn thay, Rust có giải pháp cho vấn đề này: string slices.

### String Slices

Một _string slice_ là một reference đến một chuỗi liên tiếp các phần tử của một `String`, và trông như thế này:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-17-slice/src/main.rs:here}}
```

Thay vì một reference đến toàn bộ `String`, `hello` là một reference đến một phần của `String`, được chỉ định trong phần `[0..5]` thêm vào. Chúng ta tạo slices bằng cách sử dụng một range trong dấu ngoặc vuông bằng cách chỉ định `[starting_index..ending_index]`, trong đó _`starting_index`_ là vị trí đầu tiên trong slice và _`ending_index`_ là một vị trí hơn vị trí cuối cùng trong slice. Về mặt nội bộ, cấu trúc dữ liệu slice lưu trữ vị trí bắt đầu và độ dài của slice, tương ứng với _`ending_index`_ trừ _`starting_index`_. Vì vậy, trong trường hợp `let world = &s[6..11];`, `world` sẽ là một slice chứa một pointer đến byte tại index 6 của `s` với giá trị độ dài là `5`.

Hình 4-7 cho thấy điều này trong một sơ đồ.

<img alt="Ba bảng: một bảng biểu diễn dữ liệu stack của s, trỏ đến
byte tại index 0 trong một bảng dữ liệu string &quot;hello world&quot; trên
heap. Bảng thứ ba biểu diễn dữ liệu stack của slice world, có giá trị
độ dài là 5 và trỏ đến byte 6 của bảng dữ liệu heap."
src="img/trpl04-07.svg" class="center" style="width: 50%;" />

<span class="caption">Hình 4-7: Một string slice tham chiếu đến một phần của
`String`</span>

Với cú pháp range `..` của Rust, nếu bạn muốn bắt đầu từ index 0, bạn có thể bỏ giá trị trước hai dấu chấm. Nói cách khác, những cái này bằng nhau:

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

Tương tự, nếu slice của bạn bao gồm byte cuối cùng của `String`, bạn có thể bỏ số trailing. Điều đó có nghĩa là những cái này bằng nhau:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

Bạn cũng có thể bỏ cả hai giá trị để lấy slice của toàn bộ string. Vì vậy, những cái này bằng nhau:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> Lưu ý: Các chỉ số range của string slice phải xuất hiện tại các ranh giới
> ký tự UTF-8 hợp lệ. Nếu bạn cố gắng tạo một string slice ở giữa một ký tự
> đa-byte, chương trình của bạn sẽ thoát với một lỗi.

Với tất cả thông tin này trong đầu, hãy viết lại `first_word` để trả về một slice. Kiểu biểu thị "string slice" được viết là `&str`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-18-first-word-slice/src/main.rs:here}}
```

</Listing>

Chúng ta nhận index cho cuối từ theo cách tương tự như chúng ta đã làm trong Listing 4-7, bằng cách tìm kiếm lần xuất hiện đầu tiên của dấu cách. Khi chúng ta tìm thấy một dấu cách, chúng ta trả về một string slice sử dụng đầu của string và index của dấu cách làm index bắt đầu và kết thúc.

Bây giờ khi chúng ta gọi `first_word`, chúng ta nhận lại một giá trị duy nhất được gắn kết với dữ liệu bên dưới. Giá trị được tạo thành từ một reference đến điểm bắt đầu của slice và số phần tử trong slice.

Trả về một slice cũng sẽ hoạt động cho một function `second_word`:

```rust,ignore
fn second_word(s: &String) -> &str {
```

Bây giờ chúng ta có một API đơn giản khó bị làm rối hơn nhiều vì compiler sẽ đảm bảo các references vào `String` vẫn hợp lệ. Hãy nhớ bug trong chương trình trong Listing 4-8, khi chúng ta nhận được index đến cuối từ đầu tiên nhưng sau đó xóa string để index của chúng ta không hợp lệ? Code đó về mặt logic không đúng nhưng không hiển thị bất kỳ lỗi ngay lập tức nào. Các vấn đề sẽ xuất hiện sau nếu chúng ta tiếp tục cố gắng sử dụng index từ đầu tiên với một string đã bị xóa. Slices khiến bug này không thể xảy ra và cho chúng ta biết sớm hơn nhiều rằng chúng ta có vấn đề với code. Sử dụng phiên bản slice của `first_word` sẽ throw lỗi compile-time:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/src/main.rs:here}}
```

</Listing>

Đây là lỗi compiler:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/output.txt}}
```

Nhớ lại từ các quy tắc borrowing rằng nếu chúng ta có một immutable reference đến thứ gì đó, chúng ta không thể cũng lấy một mutable reference. Vì `clear` cần truncate `String`, nó cần nhận một mutable reference. `println!` sau lệnh gọi `clear` sử dụng reference trong `word`, vì vậy immutable reference phải vẫn còn hoạt động tại thời điểm đó. Rust không cho phép mutable reference trong `clear` và immutable reference trong `word` tồn tại cùng một lúc, và compilation thất bại. Rust không chỉ làm cho API của chúng ta dễ sử dụng hơn, mà còn loại bỏ toàn bộ một lớp lỗi tại compile time!

<!-- Old headings. Do not remove or links may break. -->

<a id="string-literals-are-slices"></a>

#### String Literals là Slices

Nhớ lại rằng chúng ta đã nói về string literals được lưu trữ bên trong binary. Bây giờ mà chúng ta biết về slices, chúng ta có thể hiểu đúng string literals:

```rust
let s = "Hello, world!";
```

Kiểu của `s` ở đây là `&str`: Đây là một slice trỏ đến điểm cụ thể đó của binary. Đây cũng là lý do tại sao string literals là immutable; `&str` là một immutable reference.

#### String Slices làm Tham số

Biết rằng bạn có thể lấy slices của literals và các giá trị `String` dẫn chúng ta đến một cải tiến nữa trên `first_word`, và đó là signature của nó:

```rust,ignore
fn first_word(s: &String) -> &str {
```

Một Rustacean có kinh nghiệm hơn sẽ viết signature được hiển thị trong Listing 4-9 vì nó cho phép chúng ta sử dụng cùng một function trên cả giá trị `&String` và `&str`.

<Listing number="4-9" caption="Cải thiện function `first_word` bằng cách sử dụng một string slice cho kiểu của tham số `s`">

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:here}}
```

</Listing>

Nếu chúng ta có một string slice, chúng ta có thể truyền nó trực tiếp. Nếu chúng ta có một `String`, chúng ta có thể truyền một slice của `String` hoặc một reference đến `String`. Tính linh hoạt này tận dụng deref coercions, một tính năng chúng ta sẽ đề cập trong phần ["Using Deref Coercions in Functions and Methods"][deref-coercions]<!-- ignore --> của Chương 15.

Định nghĩa một function nhận một string slice thay vì một reference đến `String` làm cho API của chúng ta tổng quát và hữu ích hơn mà không mất bất kỳ chức năng nào:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:usage}}
```

</Listing>

### Các Slice Khác

String slices, như bạn có thể tưởng tượng, là đặc trưng cho strings. Nhưng cũng có một kiểu slice tổng quát hơn. Hãy xem xét mảng này:

```rust
let a = [1, 2, 3, 4, 5];
```

Cũng như chúng ta có thể muốn tham chiếu đến một phần của string, chúng ta có thể muốn tham chiếu đến một phần của một mảng. Chúng ta sẽ làm như vậy như thế này:

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

Slice này có kiểu `&[i32]`. Nó hoạt động theo cùng một cách như string slices, bằng cách lưu trữ một reference đến phần tử đầu tiên và một độ dài. Bạn sẽ sử dụng loại slice này cho tất cả các loại collection khác. Chúng ta sẽ thảo luận về các collection này chi tiết khi chúng ta nói về vectors trong Chương 8.

## Tóm tắt

Các khái niệm về ownership, borrowing và slices đảm bảo an toàn bộ nhớ trong các chương trình Rust tại compile time. Ngôn ngữ Rust cho bạn kiểm soát việc sử dụng bộ nhớ theo cách tương tự như các ngôn ngữ lập trình hệ thống khác. Nhưng việc owner của dữ liệu tự động dọn dẹp dữ liệu đó khi owner ra khỏi scope có nghĩa là bạn không phải viết và debug code thêm để có được sự kiểm soát này.

Ownership ảnh hưởng đến cách hoạt động của nhiều phần khác của Rust, vì vậy chúng ta sẽ nói về các khái niệm này thêm trong suốt phần còn lại của cuốn sách. Hãy chuyển sang Chương 5 và xem xét cách nhóm các phần dữ liệu lại với nhau trong một `struct`.

[ch13]: ch13-02-iterators.html
[ch6]: ch06-02-match.html#patterns-that-bind-to-values
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[deref-coercions]: ch15-02-deref.html#using-deref-coercions-in-functions-and-methods
