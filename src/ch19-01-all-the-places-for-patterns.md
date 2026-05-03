## Tất Cả Những Nơi Pattern Có Thể Được Sử Dụng

Pattern xuất hiện ở nhiều nơi trong Rust, và bạn đã sử dụng chúng rất nhiều mà có thể không nhận ra! Phần này thảo luận tất cả những nơi mà pattern là hợp lệ.

### `match` Arms

Như đã thảo luận trong Chương 6, chúng ta sử dụng pattern trong các arms của `match` expressions. Chính thức, `match` expressions được định nghĩa là từ khóa `match`, một giá trị để so khớp, và một hoặc nhiều match arms gồm một pattern và một expression để chạy nếu giá trị khớp với pattern của arm đó, như thế này:

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in the body of a block like this!
-->

<pre><code>match <em>VALUE</em> {
    <em>PATTERN</em> => <em>EXPRESSION</em>,
    <em>PATTERN</em> => <em>EXPRESSION</em>,
    <em>PATTERN</em> => <em>EXPRESSION</em>,
}</code></pre>

Ví dụ, đây là `match` expression từ Listing 6-5 để so khớp với một giá trị `Option<i32>` trong biến `x`:

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

Các pattern trong `match` expression này là `None` và `Some(i)` bên trái mỗi mũi tên.

Một yêu cầu cho `match` expressions là chúng phải là exhaustive theo nghĩa là tất cả các khả năng cho giá trị trong `match` expression phải được tính đến. Một cách để đảm bảo rằng bạn đã bao gồm mọi khả năng là có một catch-all pattern cho arm cuối cùng: Ví dụ, một tên biến khớp với bất kỳ giá trị nào không bao giờ thất bại và do đó bao gồm mọi trường hợp còn lại.

Pattern cụ thể `_` sẽ khớp với bất kỳ thứ gì, nhưng nó không bao giờ bind với một biến, vì vậy nó thường được sử dụng trong match arm cuối cùng. Pattern `_` có thể hữu ích khi bạn muốn bỏ qua bất kỳ giá trị nào không được chỉ định. Chúng ta sẽ bao gồm pattern `_` chi tiết hơn trong ["Ignoring Values in a Pattern"][ignoring-values-in-a-pattern]<!-- ignore --> ở phần sau của chương này.

### `let` Statements

Trước chương này, chúng ta chỉ đã tường minh thảo luận về việc sử dụng pattern với `match` và `if let`, nhưng thực tế, chúng ta đã sử dụng pattern ở những nơi khác cũng như vậy, bao gồm các `let` statements. Ví dụ, hãy xem xét phép gán biến đơn giản này với `let`:

```rust
let x = 5;
```

Mỗi lần bạn sử dụng một `let` statement như thế này, bạn đã sử dụng pattern, mặc dù bạn có thể không nhận ra điều đó! Chính thức hơn, một `let` statement trông như thế này:

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in the body of a block like this!
-->

<pre>
<code>let <em>PATTERN</em> = <em>EXPRESSION</em>;</code>
</pre>

Trong các statement như `let x = 5;` với một tên biến trong khe PATTERN, tên biến chỉ là một dạng pattern đơn giản. Rust so sánh expression với pattern và gán bất kỳ tên nào mà nó tìm thấy. Vì vậy, trong ví dụ `let x = 5;`, `x` là một pattern có nghĩa là "bind những gì khớp ở đây với biến `x`." Vì tên `x` là toàn bộ pattern, pattern này có hiệu lực có nghĩa là "bind mọi thứ với biến `x`, dù giá trị là gì."

Để thấy rõ hơn khía cạnh pattern-matching của `let`, hãy xem xét Listing 19-1, sử dụng pattern với `let` để destructure một tuple.


<Listing number="19-1" caption="Sử dụng pattern để destructure một tuple và tạo ba biến cùng một lúc">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-01/src/main.rs:here}}
```

</Listing>

Ở đây, chúng ta so khớp một tuple với một pattern. Rust so sánh giá trị `(1, 2, 3)` với pattern `(x, y, z)` và thấy rằng giá trị khớp với pattern—tức là, nó thấy rằng số lượng phần tử là như nhau ở cả hai—vì vậy Rust bind `1` với `x`, `2` với `y`, và `3` với `z`. Bạn có thể coi pattern tuple này như lồng ba pattern biến cá nhân bên trong nó.

Nếu số lượng phần tử trong pattern không khớp với số lượng phần tử trong tuple, toàn bộ kiểu sẽ không khớp và chúng ta sẽ nhận được lỗi compiler. Ví dụ, Listing 19-2 cho thấy một nỗ lực destructure một tuple có ba phần tử thành hai biến, không hoạt động.

<Listing number="19-2" caption="Không đúng cách xây dựng một pattern mà các biến của nó không khớp với số lượng phần tử trong tuple">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-02/src/main.rs:here}}
```

</Listing>

Cố gắng biên dịch mã này dẫn đến lỗi kiểu này:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-02/output.txt}}
```

Để sửa lỗi, chúng ta có thể bỏ qua một hoặc nhiều giá trị trong tuple bằng cách sử dụng `_` hoặc `..`, như bạn sẽ thấy trong phần ["Ignoring Values in a Pattern"][ignoring-values-in-a-pattern]<!-- ignore -->. Nếu vấn đề là chúng ta có quá nhiều biến trong pattern, giải pháp là khiến các kiểu khớp bằng cách loại bỏ các biến sao cho số lượng biến bằng số lượng phần tử trong tuple.

### Conditional `if let` Expressions

Trong Chương 6, chúng ta đã thảo luận về cách sử dụng `if let` expressions chủ yếu như một cách ngắn hơn để viết tương đương với `match` chỉ khớp một trường hợp. Tùy chọn, `if let` có thể có một `else` tương ứng chứa mã để chạy nếu pattern trong `if let` không khớp.

Listing 19-3 cho thấy rằng cũng có thể trộn và kết hợp `if let`, `else if`, và `else if let` expressions. Làm như vậy giúp chúng ta linh hoạt hơn so với `match` expression trong đó chúng ta chỉ có thể diễn đạt một giá trị để so sánh với các pattern. Ngoài ra, Rust không yêu cầu rằng các điều kiện trong một loạt các `if let`, `else if`, và `else if let` arms liên quan đến nhau.

Mã trong Listing 19-3 xác định màu nền dựa trên một loạt kiểm tra cho nhiều điều kiện. Đối với ví dụ này, chúng ta đã tạo các biến với giá trị hardcoded mà một chương trình thực sẽ nhận được từ đầu vào của người dùng.

<Listing number="19-3" file-name="src/main.rs" caption="Trộn `if let`, `else if`, `else if let`, và `else`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-03/src/main.rs}}
```

</Listing>

Nếu người dùng chỉ định một màu yêu thích, màu đó được sử dụng làm màu nền. Nếu không chỉ định màu yêu thích và hôm nay là thứ Ba, màu nền là xanh lục. Nếu không, nếu người dùng chỉ định tuổi của họ dưới dạng chuỗi và chúng ta có thể phân tích nó dưới dạng một số thành công, màu là tím hoặc cam tùy thuộc vào giá trị của số. Nếu không có điều kiện nào trong những điều kiện này được áp dụng, màu nền là xanh dương.

Cấu trúc điều kiện này cho phép chúng ta hỗ trợ các yêu cầu phức tạp. Với các giá trị hardcoded mà chúng ta có ở đây, ví dụ này sẽ in `Using purple as the background color`.

Bạn có thể thấy rằng `if let` cũng có thể giới thiệu các biến mới shade các biến hiện có theo cách mà `match` arms có thể: Dòng `if let Ok(age) = age` giới thiệu một biến `age` mới chứa giá trị bên trong variant `Ok`, shadowing biến `age` hiện có. Điều này có nghĩa là chúng ta cần đặt điều kiện `if age > 30` trong khối đó: Chúng ta không thể kết hợp hai điều kiện này thành `if let Ok(age) = age && age > 30`. `age` mới mà chúng ta muốn so sánh với 30 không hợp lệ cho đến khi scope mới bắt đầu với dấu ngoặc nhọn.

Nhược điểm của việc sử dụng `if let` expressions là compiler không kiểm tra exhaustiveness, trong khi với `match` expressions thì có. Nếu chúng ta bỏ đi khối `else` cuối cùng và do đó bỏ lỡ xử lý một số trường hợp, compiler sẽ không cảnh báo chúng ta về lỗi logic có thể xảy ra.

### `while let` Conditional Loops

Tương tự trong cấu trúc với `if let`, vòng lặp điều kiện `while let` cho phép một vòng lặp `while` chạy miễn là một pattern tiếp tục khớp. Trong Listing 19-4, chúng ta cho thấy một vòng lặp `while let` chờ các thông điệp được gửi giữa các thread, nhưng trong trường hợp này kiểm tra một `Result` thay vì một `Option`.

<Listing number="19-4" caption="Sử dụng vòng lặp `while let` để in các giá trị miễn là `rx.recv()` trả về `Ok`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-04/src/main.rs:here}}
```

</Listing>

Ví dụ này in `1`, `2`, và sau đó `3`. Method `recv` lấy thông điệp đầu tiên ra khỏi phía receiver của channel và trả về một `Ok(value)`. Khi chúng ta lần đầu tiên thấy `recv` trở lại trong Chương 16, chúng ta đã unwrap lỗi trực tiếp, hoặc chúng ta tương tác với nó như một iterator sử dụng vòng lặp `for`. Tuy nhiên, như Listing 19-4 cho thấy, chúng ta cũng có thể sử dụng `while let`, vì method `recv` trả về một `Ok` mỗi lần một thông điệp đến, miễn là sender tồn tại, và sau đó tạo ra một `Err` sau khi phía sender ngắt kết nối.

### `for` Loops

Trong vòng lặp `for`, giá trị theo sau trực tiếp từ khóa `for` là một pattern. Ví dụ, trong `for x in y`, `x` là pattern. Listing 19-5 minh họa cách sử dụng pattern trong vòng lặp `for` để destructure, hoặc phá vỡ, một tuple như một phần của vòng lặp `for`.


<Listing number="19-5" caption="Sử dụng pattern trong vòng lặp `for` để destructure một tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-05/src/main.rs:here}}
```

</Listing>

Mã trong Listing 19-5 sẽ in như sau:


```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-05/output.txt}}
```

Chúng ta điều chỉnh một iterator sử dụng method `enumerate` để nó tạo ra một giá trị và chỉ mục cho giá trị đó, được đặt trong một tuple. Giá trị đầu tiên được tạo ra là tuple `(0, 'a')`. Khi giá trị này được so khớp với pattern `(index, value)`, index sẽ là `0` và value sẽ là `'a'`, in dòng đầu tiên của output.


### Function Parameters

Các tham số function cũng có thể là pattern. Mã trong Listing 19-6, khai báo một function được đặt tên là `foo` lấy một tham số có tên là `x` của kiểu `i32`, đến bây giờ nên trông quen thuộc.

<Listing number="19-6" caption="Một signature function sử dụng pattern trong các tham số">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-06/src/main.rs:here}}
```

</Listing>

Phần `x` là một pattern! Như chúng ta đã làm với `let`, chúng ta có thể so khớp một tuple trong các argument của function với pattern. Listing 19-7 chia các giá trị trong một tuple khi chúng ta truyền nó vào một function.

<Listing number="19-7" file-name="src/main.rs" caption="Một function với các tham số destructure một tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-07/src/main.rs}}
```

</Listing>

Mã này in `Current location: (3, 5)`. Các giá trị `&(3, 5)` khớp với pattern `&(x, y)`, vì vậy `x` là giá trị `3` và `y` là giá trị `5`.

Chúng ta cũng có thể sử dụng pattern trong danh sách tham số closure theo cách tương tự như trong danh sách tham số function vì closures tương tự như các function, như đã thảo luận trong Chương 13.

Tại thời điểm này, bạn đã thấy một số cách để sử dụng pattern, nhưng pattern không hoạt động giống nhau ở mọi nơi chúng ta có thể sử dụng chúng. Ở một số nơi, các pattern phải là irrefutable; ở những trường hợp khác, chúng có thể là refutable. Chúng ta sẽ thảo luận hai khái niệm này tiếp theo.

[ignoring-values-in-a-pattern]: ch19-03-pattern-syntax.html#ignoring-values-in-a-pattern
