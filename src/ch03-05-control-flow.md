## Control Flow

Khả năng chạy một số code tùy thuộc vào điều kiện có `true` hay không và khả năng chạy một số code nhiều lần trong khi một điều kiện là `true` là các building blocks cơ bản trong hầu hết các ngôn ngữ lập trình. Các cấu trúc phổ biến nhất cho phép bạn kiểm soát luồng thực thi của code Rust là `if` expressions và loops.

### `if` Expressions

Một `if` expression cho phép bạn phân nhánh code tùy thuộc vào điều kiện. Bạn cung cấp một điều kiện và sau đó nói, "Nếu điều kiện này được thỏa mãn, hãy chạy block code này. Nếu điều kiện không được thỏa mãn, đừng chạy block code này."

Tạo một project mới tên _branches_ trong thư mục _projects_ để khám phá `if` expression. Trong file _src/main.rs_, nhập vào như sau:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/src/main.rs}}
```

Tất cả `if` expressions bắt đầu bằng keyword `if`, theo sau là điều kiện. Trong trường hợp này, điều kiện kiểm tra xem variable `number` có giá trị nhỏ hơn 5 không. Chúng ta đặt block code để thực thi nếu điều kiện là `true` ngay sau điều kiện bên trong dấu ngoặc nhọn. Các blocks code được liên kết với các điều kiện trong `if` expressions đôi khi được gọi là _arms_, giống như các arms trong `match` expressions mà chúng ta đã thảo luận trong phần ["So Sánh Dự Đoán Với Số Bí Mật"][comparing-the-guess-to-the-secret-number]<!-- ignore --> của Chương 2.

Tùy chọn, chúng ta cũng có thể bao gồm một `else` expression, mà chúng ta đã chọn làm ở đây, để cung cấp cho chương trình một block code thay thế để thực thi nếu điều kiện đánh giá thành `false`. Nếu bạn không cung cấp `else` expression và điều kiện là `false`, chương trình sẽ chỉ bỏ qua block `if` và chuyển sang đoạn code tiếp theo.

Thử chạy code này; bạn sẽ thấy output sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/output.txt}}
```

Hãy thử thay đổi giá trị của `number` thành một giá trị làm cho điều kiện thành `false` để xem điều gì xảy ra:

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/src/main.rs:here}}
```

Chạy lại chương trình và xem output:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/output.txt}}
```

Cũng cần lưu ý rằng điều kiện trong code này _phải_ là `bool`. Nếu điều kiện không phải là `bool`, chúng ta sẽ nhận được lỗi. Ví dụ, thử chạy code sau:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/src/main.rs}}
```

Điều kiện `if` lần này đánh giá thành giá trị `3`, và Rust thông báo lỗi:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/output.txt}}
```

Lỗi cho biết Rust mong đợi `bool` nhưng nhận được integer. Không giống như các ngôn ngữ như Ruby và JavaScript, Rust sẽ không tự động cố convert các kiểu non-Boolean thành Boolean. Bạn phải rõ ràng và luôn cung cấp cho `if` một Boolean làm điều kiện. Nếu chúng ta muốn block code `if` chỉ chạy khi một số không bằng `0`, ví dụ, chúng ta có thể thay đổi `if` expression thành như sau:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-29-if-not-equal-0/src/main.rs}}
```

Chạy code này sẽ in `number was something other than zero`.

#### Xử Lý Nhiều Điều Kiện Với `else if`

Bạn có thể dùng nhiều điều kiện bằng cách kết hợp `if` và `else` trong một `else if` expression. Ví dụ:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/src/main.rs}}
```

Chương trình này có bốn đường đi có thể thực hiện. Sau khi chạy, bạn sẽ thấy output sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/output.txt}}
```

Khi chương trình này thực thi, nó kiểm tra lần lượt từng `if` expression và thực thi body đầu tiên mà điều kiện của nó đánh giá thành `true`. Lưu ý rằng mặc dù 6 chia hết cho 2, chúng ta không thấy output `number is divisible by 2`, cũng không thấy text `number is not divisible by 4, 3, or 2` từ block `else`. Đó là vì Rust chỉ thực thi block cho điều kiện `true` đầu tiên, và một khi tìm thấy một, nó thậm chí không kiểm tra phần còn lại.

Dùng quá nhiều `else if` expressions có thể làm code của bạn lộn xộn, vì vậy nếu bạn có nhiều hơn một, bạn có thể muốn refactor code của mình. Chương 6 mô tả một cấu trúc branching mạnh mẽ của Rust được gọi là `match` cho những trường hợp như vậy.

#### Dùng `if` Trong Một `let` Statement

Vì `if` là một expression, chúng ta có thể dùng nó ở phía bên phải của một `let` statement để gán kết quả cho một variable, như trong Listing 3-2.

<Listing number="3-2" file-name="src/main.rs" caption="Gán kết quả của `if` expression cho một variable">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-02/src/main.rs}}
```

</Listing>

Variable `number` sẽ được gán giá trị dựa trên kết quả của `if` expression. Chạy code này để xem điều gì xảy ra:

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-02/output.txt}}
```

Hãy nhớ rằng các blocks code đánh giá thành expression cuối cùng trong chúng, và các số tự thân cũng là expressions. Trong trường hợp này, giá trị của toàn bộ `if` expression phụ thuộc vào block code nào thực thi. Điều này có nghĩa là các giá trị có khả năng là kết quả từ mỗi arm của `if` phải là cùng kiểu; trong Listing 3-2, kết quả của arm `if` và arm `else` đều là integers `i32`. Nếu các kiểu không khớp, như trong ví dụ sau, chúng ta sẽ nhận được lỗi:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/src/main.rs}}
```

Khi chúng ta cố compile code này, chúng ta sẽ nhận được lỗi. Các arms `if` và `else` có các kiểu giá trị không tương thích, và Rust chỉ ra chính xác nơi tìm vấn đề trong chương trình:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/output.txt}}
```

Expression trong block `if` đánh giá thành một integer, và expression trong block `else` đánh giá thành một string. Điều này sẽ không hoạt động, vì các variables phải có một kiểu duy nhất, và Rust cần biết chắc chắn tại compile time kiểu của variable `number` là gì. Biết kiểu của `number` cho phép compiler xác minh kiểu hợp lệ ở khắp nơi chúng ta dùng `number`. Rust sẽ không thể làm điều đó nếu kiểu của `number` chỉ được xác định tại runtime; compiler sẽ phức tạp hơn và ít đảm bảo hơn về code nếu nó phải theo dõi nhiều kiểu giả thuyết cho bất kỳ variable nào.

### Repetition Với Loops

Thường rất hữu ích khi thực thi một block code nhiều lần. Cho tác vụ này, Rust cung cấp một số _loops_, sẽ chạy qua code bên trong body của loop đến cuối và sau đó bắt đầu lại ngay từ đầu. Để thử nghiệm với loops, hãy tạo một project mới gọi là _loops_.

Rust có ba loại loops: `loop`, `while`, và `for`. Hãy thử từng loại.

#### Lặp Lại Code Với `loop`

Keyword `loop` yêu cầu Rust thực thi một block code lặp đi lặp lại mãi mãi hoặc cho đến khi bạn rõ ràng bảo nó dừng lại.

Ví dụ, thay đổi file _src/main.rs_ trong thư mục _loops_ của bạn trông như sau:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-loop/src/main.rs}}
```

Khi chạy chương trình này, chúng ta sẽ thấy `again!` được in liên tục cho đến khi chúng ta dừng chương trình theo cách thủ công. Hầu hết các terminal hỗ trợ keyboard shortcut <kbd>ctrl</kbd>-<kbd>C</kbd> để ngắt một chương trình đang bị kẹt trong một vòng lặp liên tục. Hãy thử:

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-32-loop
cargo run
CTRL-C
-->

```console
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.08s
     Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!
```

Ký hiệu `^C` đại diện cho nơi bạn nhấn <kbd>ctrl</kbd>-<kbd>C</kbd>.

Bạn có thể hoặc không thấy từ `again!` được in sau `^C`, tùy thuộc vào vị trí của code trong loop khi nó nhận được tín hiệu ngắt.

May mắn thay, Rust cũng cung cấp một cách để thoát khỏi loop bằng code. Bạn có thể đặt keyword `break` bên trong loop để cho chương trình biết khi nào dừng thực thi loop. Nhớ lại rằng chúng ta đã làm điều này trong guessing game trong phần ["Thoát Ra Sau Khi Đoán Đúng"][quitting-after-a-correct-guess]<!-- ignore --> của Chương 2 để thoát chương trình khi người dùng thắng trò chơi bằng cách đoán đúng số.

Chúng ta cũng đã dùng `continue` trong guessing game, điều mà trong một loop yêu cầu chương trình bỏ qua bất kỳ code còn lại nào trong lần lặp hiện tại của loop và chuyển sang lần lặp tiếp theo.

#### Trả Về Giá Trị Từ Loops

Một trong những cách dùng của `loop` là thử lại một thao tác bạn biết có thể thất bại, chẳng hạn như kiểm tra xem một thread đã hoàn thành công việc chưa. Bạn cũng có thể cần truyền kết quả của thao tác đó ra khỏi loop đến phần còn lại của code. Để làm điều này, bạn có thể thêm giá trị bạn muốn trả về sau expression `break` bạn dùng để dừng loop; giá trị đó sẽ được trả về từ loop để bạn có thể dùng nó, như ví dụ sau:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-33-return-value-from-loop/src/main.rs}}
```

Trước loop, chúng ta khai báo một variable tên `counter` và khởi tạo nó thành `0`. Sau đó, chúng ta khai báo một variable tên `result` để giữ giá trị được trả về từ loop. Trên mỗi lần lặp của loop, chúng ta cộng thêm `1` vào variable `counter`, và sau đó kiểm tra xem `counter` có bằng `10` không. Khi bằng, chúng ta dùng keyword `break` với giá trị `counter * 2`. Sau loop, chúng ta dùng dấu chấm phẩy để kết thúc statement gán giá trị cho `result`. Cuối cùng, chúng ta in giá trị trong `result`, trong trường hợp này là `20`.

Bạn cũng có thể `return` từ bên trong một loop. Trong khi `break` chỉ thoát ra khỏi loop hiện tại, `return` luôn thoát ra khỏi function hiện tại.

<!-- Old headings. Do not remove or links may break. -->
<a id="loop-labels-to-disambiguate-between-multiple-loops"></a>

#### Phân Biệt Bằng Loop Labels

Nếu bạn có loops lồng nhau, `break` và `continue` áp dụng cho loop trong cùng tại điểm đó. Bạn có thể tùy chọn chỉ định một _loop label_ trên một loop mà sau đó bạn có thể dùng với `break` hoặc `continue` để chỉ định rằng các keywords đó áp dụng cho loop được đánh nhãn thay vì loop trong cùng. Loop labels phải bắt đầu bằng dấu nháy đơn. Đây là ví dụ với hai loops lồng nhau:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/src/main.rs}}
```

Outer loop có nhãn `'counting_up`, và nó sẽ đếm từ 0 đến 2. Inner loop không có nhãn đếm ngược từ 10 đến 9. `break` đầu tiên không chỉ định nhãn sẽ chỉ thoát khỏi inner loop. Câu lệnh `break 'counting_up;` sẽ thoát khỏi outer loop. Code này in:

```console
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/output.txt}}
```

<!-- Old headings. Do not remove or links may break. -->
<a id="conditional-loops-with-while"></a>

#### Đơn Giản Hóa Conditional Loops Với while

Một chương trình thường cần đánh giá một điều kiện trong một loop. Trong khi điều kiện là `true`, loop chạy. Khi điều kiện không còn là `true`, chương trình gọi `break`, dừng loop. Có thể implement hành vi như này bằng cách kết hợp `loop`, `if`, `else`, và `break`; bạn có thể thử ngay bây giờ trong một chương trình nếu muốn. Tuy nhiên, pattern này rất phổ biến đến mức Rust có một cấu trúc ngôn ngữ tích hợp sẵn cho nó, được gọi là `while` loop. Trong Listing 3-3, chúng ta dùng `while` để lặp chương trình ba lần, đếm ngược mỗi lần, và sau đó, sau loop, in một thông báo và thoát.

<Listing number="3-3" file-name="src/main.rs" caption="Dùng `while` loop để chạy code trong khi điều kiện đánh giá thành `true`">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-03/src/main.rs}}
```

</Listing>

Cấu trúc này loại bỏ nhiều lớp lồng nhau cần thiết nếu bạn dùng `loop`, `if`, `else`, và `break`, và nó rõ ràng hơn. Trong khi điều kiện đánh giá thành `true`, code chạy; nếu không, nó thoát loop.

#### Lặp Qua Một Collection Với `for`

Bạn có thể chọn dùng cấu trúc `while` để lặp qua các phần tử của một collection, chẳng hạn như một array. Ví dụ, loop trong Listing 3-4 in mỗi phần tử trong array `a`.

<Listing number="3-4" file-name="src/main.rs" caption="Lặp qua mỗi phần tử của một collection dùng `while` loop">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-04/src/main.rs}}
```

</Listing>

Ở đây, code đếm lên qua các phần tử trong array. Nó bắt đầu tại index `0` và sau đó lặp cho đến khi đạt đến index cuối cùng trong array (tức là khi `index < 5` không còn là `true`). Chạy code này sẽ in mỗi phần tử trong array:

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-04/output.txt}}
```

Tất cả năm giá trị array xuất hiện trong terminal như mong đợi. Mặc dù `index` sẽ đạt giá trị `5` tại một thời điểm nào đó, loop dừng thực thi trước khi cố lấy giá trị thứ sáu từ array.

Tuy nhiên, cách tiếp cận này dễ xảy ra lỗi; chúng ta có thể khiến chương trình panic nếu giá trị index hoặc điều kiện kiểm tra không đúng. Ví dụ, nếu bạn thay đổi định nghĩa của array `a` để có bốn phần tử nhưng quên cập nhật điều kiện thành `while index < 4`, code sẽ panic. Nó cũng chậm, vì compiler thêm runtime code để thực hiện kiểm tra điều kiện xem index có nằm trong giới hạn của array trên mỗi lần lặp qua loop không.

Là một thay thế ngắn gọn hơn, bạn có thể dùng `for` loop và thực thi một số code cho mỗi phần tử trong một collection. Một `for` loop trông như code trong Listing 3-5.

<Listing number="3-5" file-name="src/main.rs" caption="Lặp qua mỗi phần tử của một collection dùng `for` loop">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-05/src/main.rs}}
```

</Listing>

Khi chúng ta chạy code này, chúng ta sẽ thấy cùng output như trong Listing 3-4. Quan trọng hơn, chúng ta đã tăng tính an toàn của code và loại bỏ khả năng xảy ra bugs có thể xảy ra do vượt quá cuối array hoặc không đi đủ xa và bỏ sót một số phần tử. Machine code được tạo ra từ `for` loops cũng có thể hiệu quả hơn vì index không cần được so sánh với độ dài của array trên mỗi lần lặp.

Khi dùng `for` loop, bạn sẽ không cần nhớ thay đổi bất kỳ code nào khác nếu bạn thay đổi số lượng giá trị trong array, như bạn phải làm với phương pháp được dùng trong Listing 3-4.

Tính an toàn và ngắn gọn của `for` loops làm cho chúng là cấu trúc loop được dùng phổ biến nhất trong Rust. Ngay cả trong các tình huống mà bạn muốn chạy một số code một số lần nhất định, như trong ví dụ đếm ngược dùng `while` loop trong Listing 3-3, hầu hết các Rustaceans sẽ dùng `for` loop. Cách để làm điều đó là dùng một `Range`, được cung cấp bởi standard library, tạo ra tất cả các số theo thứ tự bắt đầu từ một số và kết thúc trước một số khác.

Đây là cách đếm ngược trông như thế nào khi dùng `for` loop và một method khác chúng ta chưa nói đến, `rev`, để đảo ngược range:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-34-for-range/src/main.rs}}
```

Code này trông đẹp hơn một chút, phải không?

## Tóm Tắt

Bạn đã làm được! Đây là một chương đáng kể: Bạn đã học về variables, scalar và compound data types, functions, comments, `if` expressions và loops! Để luyện tập với các khái niệm được thảo luận trong chương này, hãy thử xây dựng các chương trình để làm những việc sau:

- Chuyển đổi nhiệt độ giữa Fahrenheit và Celsius.
- Tạo số Fibonacci thứ _n_.
- In lời bài hát Giáng sinh "The Twelve Days of Christmas," tận dụng lợi thế của sự lặp lại trong bài hát.

Khi bạn sẵn sàng tiếp tục, chúng ta sẽ nói về một khái niệm trong Rust mà _không_ thường tồn tại trong các ngôn ngữ lập trình khác: ownership.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[quitting-after-a-correct-guess]: ch02-00-guessing-game-tutorial.html#quitting-after-a-correct-guess
