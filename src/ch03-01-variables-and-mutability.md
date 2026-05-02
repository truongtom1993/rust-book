## Variables và Mutability

Như đã đề cập trong phần ["Lưu Trữ Giá Trị Với Variables"][storing-values-with-variables]<!-- ignore -->, theo mặc định, variables là immutable (bất biến). Đây là một trong nhiều cách mà Rust khuyến khích bạn viết code theo hướng tận dụng tính an toàn và khả năng concurrency dễ dàng mà Rust cung cấp. Tuy nhiên, bạn vẫn có tùy chọn để làm cho variables trở nên mutable. Hãy cùng tìm hiểu cách thức và lý do Rust khuyến khích ưu tiên sử dụng immutability, và tại sao đôi khi bạn có thể muốn chọn không dùng nó.

Khi một variable là immutable, một khi đã gán một giá trị cho tên đó, bạn không thể thay đổi giá trị đó. Để minh họa điều này, hãy tạo một project mới tên _variables_ trong thư mục _projects_ của bạn bằng cách dùng `cargo new variables`.

Sau đó, trong thư mục _variables_ mới tạo, mở _src/main.rs_ và thay thế code bằng đoạn code sau đây — code này sẽ chưa compile được:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/src/main.rs}}
```

Lưu và chạy chương trình bằng `cargo run`. Bạn sẽ nhận được thông báo lỗi về immutability, như trong output sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/output.txt}}
```

Ví dụ này cho thấy cách compiler giúp bạn tìm ra lỗi trong chương trình. Lỗi từ compiler có thể gây bực bội, nhưng thực ra chúng chỉ có nghĩa là chương trình của bạn chưa thực hiện đúng những gì bạn muốn một cách an toàn; chúng _không_ có nghĩa là bạn là một lập trình viên tồi! Ngay cả những Rustaceans kỳ cựu cũng vẫn gặp lỗi compiler.

Bạn nhận được thông báo lỗi `` cannot assign twice to immutable variable `x` `` vì bạn đã cố gán giá trị thứ hai cho variable immutable `x`.

Điều quan trọng là chúng ta nhận được lỗi compile-time khi cố thay đổi một giá trị được chỉ định là immutable, bởi vì chính tình huống này có thể dẫn đến bugs. Nếu một phần code của chúng ta hoạt động dựa trên giả định rằng một giá trị sẽ không bao giờ thay đổi, nhưng một phần code khác lại thay đổi giá trị đó, thì phần đầu tiên có thể không hoạt động như thiết kế. Nguyên nhân của loại bug này có thể khó theo dõi sau khi đã xảy ra, đặc biệt khi phần code thứ hai chỉ _đôi khi_ thay đổi giá trị. Rust compiler đảm bảo rằng khi bạn khai báo một giá trị sẽ không thay đổi, nó thực sự sẽ không thay đổi, vì vậy bạn không phải tự theo dõi điều đó. Code của bạn nhờ vậy dễ suy luận hơn.

Nhưng mutability có thể rất hữu ích và làm cho việc viết code tiện lợi hơn. Mặc dù variables là immutable theo mặc định, bạn có thể làm chúng trở nên mutable bằng cách thêm `mut` trước tên variable như bạn đã làm trong [Chương 2][storing-values-with-variables]<!-- ignore -->. Thêm `mut` cũng truyền đạt ý định cho người đọc code trong tương lai bằng cách chỉ ra rằng các phần khác của code sẽ thay đổi giá trị của variable này.

Ví dụ, hãy thay đổi _src/main.rs_ thành như sau:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/src/main.rs}}
```

Khi chạy chương trình này, chúng ta nhận được:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/output.txt}}
```

Chúng ta được phép thay đổi giá trị gán cho `x` từ `5` thành `6` khi dùng `mut`. Cuối cùng, quyết định có dùng mutability hay không là tùy bạn và phụ thuộc vào điều bạn cho là rõ ràng nhất trong từng tình huống cụ thể.

<!-- Old headings. Do not remove or links may break. -->
<a id="constants"></a>

### Khai Báo Constants

Giống như immutable variables, _constants_ là các giá trị được gán cho một tên và không được phép thay đổi, nhưng có một số điểm khác biệt giữa constants và variables.

Thứ nhất, bạn không được phép dùng `mut` với constants. Constants không chỉ là immutable theo mặc định — chúng luôn luôn là immutable. Bạn khai báo constants bằng keyword `const` thay vì keyword `let`, và kiểu của giá trị _phải_ được annotate. Chúng ta sẽ đề cập đến types và type annotations trong phần tiếp theo, ["Data Types"][data-types]<!-- ignore -->, vì vậy đừng lo về chi tiết ngay bây giờ. Chỉ cần biết rằng bạn phải luôn annotate type.

Constants có thể được khai báo ở bất kỳ scope nào, kể cả global scope, điều này làm chúng hữu ích cho các giá trị mà nhiều phần của code cần biết đến.

Điểm khác biệt cuối cùng là constants chỉ có thể được gán bằng một biểu thức hằng số, không phải kết quả của một giá trị chỉ có thể được tính toán tại runtime.

Đây là một ví dụ khai báo constant:

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

Tên của constant là `THREE_HOURS_IN_SECONDS`, và giá trị của nó được đặt bằng kết quả của phép nhân 60 (số giây trong một phút) với 60 (số phút trong một giờ) với 3 (số giờ chúng ta muốn đếm trong chương trình). Quy ước đặt tên cho constants trong Rust là dùng toàn chữ hoa với dấu gạch dưới giữa các từ. Compiler có thể đánh giá một tập hợp hạn chế các phép toán tại compile time, điều này cho phép chúng ta chọn viết giá trị này theo cách dễ hiểu và xác minh hơn, thay vì đặt constant này bằng giá trị 10,800. Xem [phần về constant evaluation trong Rust Reference][const-eval] để biết thêm thông tin về những phép toán nào có thể được dùng khi khai báo constants.

Constants có hiệu lực trong toàn bộ thời gian chương trình chạy, trong phạm vi scope mà chúng được khai báo. Đặc tính này làm constants hữu ích cho các giá trị trong domain ứng dụng của bạn mà nhiều phần của chương trình có thể cần biết, chẳng hạn như điểm tối đa mà một người chơi trong game được phép kiếm, hoặc tốc độ ánh sáng.

Việc đặt tên các giá trị hardcoded được dùng xuyên suốt chương trình của bạn thành constants rất hữu ích trong việc truyền đạt ý nghĩa của giá trị đó cho những người bảo trì code trong tương lai. Nó cũng giúp có chỉ một nơi duy nhất trong code mà bạn cần thay đổi nếu giá trị hardcoded cần được cập nhật trong tương lai.

### Shadowing

Như bạn đã thấy trong tutorial guessing game ở [Chương 2][comparing-the-guess-to-the-secret-number]<!-- ignore -->, bạn có thể khai báo một variable mới với cùng tên như một variable trước đó. Các Rustacean nói rằng variable đầu tiên bị _shadowed_ bởi variable thứ hai, có nghĩa là variable thứ hai là thứ mà compiler sẽ thấy khi bạn dùng tên variable đó. Thực chất, variable thứ hai che khuất variable đầu tiên, chiếm tất cả các lần dùng tên variable đó cho đến khi chính nó bị shadow hoặc scope kết thúc. Chúng ta có thể shadow một variable bằng cách dùng cùng tên variable đó và lặp lại việc dùng keyword `let` như sau:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/src/main.rs}}
```

Chương trình này đầu tiên gán `x` với giá trị `5`. Sau đó, nó tạo một variable mới `x` bằng cách lặp lại `let x =`, lấy giá trị gốc và cộng thêm `1` nên giá trị của `x` là `6`. Sau đó, trong một inner scope được tạo bằng dấu ngoặc nhọn, câu lệnh `let` thứ ba cũng shadow `x` và tạo một variable mới, nhân giá trị trước đó với `2` để `x` có giá trị `12`. Khi scope đó kết thúc, inner shadowing kết thúc và `x` trở lại là `6`. Khi chạy chương trình này, nó sẽ xuất ra như sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/output.txt}}
```

Shadowing khác với việc đánh dấu một variable là `mut` bởi vì chúng ta sẽ nhận được compile-time error nếu vô tình cố gán lại cho variable này mà không dùng keyword `let`. Bằng cách dùng `let`, chúng ta có thể thực hiện một số biến đổi trên giá trị nhưng vẫn giữ variable là immutable sau khi các biến đổi đó hoàn thành.

Điểm khác biệt khác giữa `mut` và shadowing là vì chúng ta thực chất đang tạo một variable mới khi dùng lại keyword `let`, chúng ta có thể thay đổi kiểu của giá trị nhưng tái sử dụng cùng tên. Ví dụ, giả sử chương trình của chúng ta yêu cầu người dùng cho biết họ muốn bao nhiêu khoảng trắng giữa một số văn bản bằng cách nhập ký tự khoảng trắng, và sau đó chúng ta muốn lưu trữ input đó dưới dạng số:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-04-shadowing-can-change-types/src/main.rs:here}}
```

Variable `spaces` đầu tiên là kiểu string, và variable `spaces` thứ hai là kiểu số. Shadowing vì vậy giúp chúng ta không phải nghĩ ra các tên khác nhau, chẳng hạn như `spaces_str` và `spaces_num`; thay vào đó, chúng ta có thể tái sử dụng tên `spaces` đơn giản hơn. Tuy nhiên, nếu chúng ta cố dùng `mut` cho điều này, như ví dụ dưới đây, chúng ta sẽ nhận được compile-time error:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/src/main.rs:here}}
```

Lỗi cho biết chúng ta không được phép thay đổi kiểu của một variable:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/output.txt}}
```

Bây giờ chúng ta đã khám phá cách variables hoạt động, hãy cùng tìm hiểu thêm về các data types mà chúng có thể có.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[data-types]: ch03-02-data-types.html#data-types
[storing-values-with-variables]: ch02-00-guessing-game-tutorial.html#storing-values-with-variables
[const-eval]: ../reference/const_eval.html
