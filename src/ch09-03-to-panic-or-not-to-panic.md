## Để `panic!` hay Không `panic!`

Vì vậy, bạn quyết định khi nào nên gọi `panic!` và khi nào nên trả về
`Result` như thế nào? Khi code panics, không có cách nào để phục hồi.
Bạn có thể gọi `panic!` cho bất kỳ tình huống lỗi nào, cho dù có hay không
có cách có thể phục hồi, nhưng sau đó bạn đang đưa ra quyết định rằng một tình huống
không thể phục hồi thay mặt cho calling code. Khi bạn chọn trả về một
giá trị `Result`, bạn cung cấp các tùy chọn cho calling code. Calling code có thể
chọn cố gắng phục hồi theo cách thích hợp cho tình huống của nó, hoặc nó có thể quyết định
rằng một giá trị `Err` trong trường hợp này không thể phục hồi, vì vậy nó có thể gọi
`panic!` và chuyển lỗi có thể phục hồi của bạn thành một lỗi không thể
phục hồi. Do đó, trả về `Result` là một lựa chọn mặc định tốt khi bạn đang
định nghĩa một function có thể thất bại.

Trong các tình huống như ví dụ, code nguyên mẫu, và tests, nó
thích hợp hơn để viết code panic thay vì trả về `Result`.
Hãy khám phá lý do tại sao, sau đó thảo luận các tình huống mà compiler không thể
nói được rằng thất bại là không thể, nhưng bạn như một con người có thể.
Chương sẽ kết thúc với một số hướng dẫn chung về cách quyết định
liệu có nên panic trong library code.

### Ví Dụ, Code Nguyên Mẫu, và Tests

Khi bạn đang viết một ví dụ để minh họa một khái niệm nào đó, cũng bao gồm
code error-handling mạnh mẽ có thể làm cho ví dụ kém rõ ràng hơn. Ở các ví dụ,
nó được hiểu rằng một lệnh gọi tới một method như `unwrap` có thể panic
được dự định như một placeholder cho cách bạn muốn ứng dụng của bạn
xử lý lỗi, có thể khác nhau dựa trên những gì phần còn lại của code của bạn đang làm.

Tương tự, các methods `unwrap` và `expect` rất tiện lợi khi bạn đang
tạo nguyên mẫu và bạn chưa sẵn sàng quyết định cách xử lý lỗi. Chúng để lại
các markers rõ ràng trong code của bạn để khi bạn sẵn sàng làm cho
chương trình của bạn mạnh mẽ hơn.

Nếu một lệnh gọi method thất bại trong một test, bạn muốn toàn bộ test
thất bại, ngay cả khi method đó không phải là chức năng được kiểm tra. Vì
`panic!` là cách một test được đánh dấu là thất bại, gọi `unwrap` hoặc
`expect` chính xác là những gì nên xảy ra.

<!-- Old headings. Do not remove or links may break. -->

<a id="cases-in-which-you-have-more-information-than-the-compiler"></a>

### Khi Bạn Có Thêm Thông Tin Hơn Compiler

Nó cũng sẽ thích hợp để gọi `expect` khi bạn có một số logic khác
đảm bảo rằng `Result` sẽ có một giá trị `Ok`, nhưng logic đó không
là thứ compiler hiểu. Bạn vẫn sẽ có một giá trị `Result` mà bạn
cần xử lý: Bất kỳ thao tác nào bạn đang gọi vẫn có khả năng thất bại
nói chung, ngay cả khi nó logic không thể trong tình huống cụ thể của bạn.
Nếu bạn có thể đảm bảo bằng cách kiểm tra code thủ công rằng bạn sẽ không bao giờ
có một variant `Err`, nó là hoàn toàn chấp nhận được để gọi `expect` và
document lý do tại sao bạn nghĩ bạn sẽ không bao giờ có một variant `Err`
trong argument text. Đây là một ví dụ:

```rust
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-08-unwrap-that-cant-fail/src/main.rs:here}}
```
Chúng ta đang tạo một instance `IpAddr` bằng cách parse một string được hardcode.  
Chúng ta có thể thấy rằng `127.0.0.1` là một IP address hợp lệ, vì vậy việc dùng `expect` ở đây là chấp nhận được.  
Tuy nhiên, bản thân việc string này được hardcode và hợp lệ cũng không làm thay đổi kiểu trả về của method `parse`:  
Chúng ta vẫn nhận về một giá trị `Result`, và compiler vẫn sẽ buộc chúng ta xử lý `Result` như thể variant `Err` là có thể xảy ra, vì compiler không đủ thông minh để biết rằng string này **luôn luôn** là một IP address hợp lệ.  
Nếu string IP address đến từ người dùng thay vì được hardcode trong chương trình và do đó _có_ khả năng thất bại, khi đó chúng ta chắc chắn muốn xử lý `Result` theo một cách robust hơn.  
Việc ghi chú rõ giả định rằng IP address này được hardcode sẽ giúp nhắc chúng ta đổi `expect` sang code error-handling tốt hơn nếu sau này cần lấy IP address từ một nguồn khác.

### Các guideline cho Error Handling

Nên để code của bạn panic trong những trường hợp code có thể rơi vào một trạng thái xấu.  
Trong ngữ cảnh này, _trạng thái xấu_ là khi một assumption, guarantee, contract, hoặc invariant nào đó bị phá vỡ, ví dụ khi những giá trị không hợp lệ, giá trị mâu thuẫn, hoặc giá trị bị thiếu được truyền vào code của bạn—và kèm theo một hoặc nhiều điều sau:

- Trạng thái xấu này là điều không mong đợi, trái ngược với những thứ có thể xảy ra thỉnh thoảng, như việc user nhập data sai format.  
- Code của bạn ở những bước sau **cần** dựa vào assumption rằng nó không ở trong trạng thái xấu này, thay vì phải kiểm tra lỗi ở mọi bước.  
- Không có cách tốt để encode thông tin này vào các type bạn dùng. Chúng ta sẽ đi qua một ví dụ về ý này trong phần [“Encoding States and Behavior as Types”][encoding]<!-- ignore --> ở Chapter 18.

Nếu ai đó gọi code của bạn và truyền vào các giá trị vô nghĩa, tốt nhất là bạn nên trả về một error nếu có thể, để user của library có thể tự quyết định họ muốn xử lý tình huống đó như thế nào.  
Tuy vậy, trong những trường hợp tiếp tục chạy có thể gây mất an toàn hoặc gây hại, lựa chọn tốt nhất có thể là gọi `panic!` và cảnh báo người đang dùng library về bug trong code của họ để họ sửa trong quá trình development.  
Tương tự, `panic!` thường phù hợp khi bạn gọi external code nằm ngoài tầm kiểm soát của bạn và nó trả về một trạng thái không hợp lệ mà bạn không có cách nào sửa được.

Tuy nhiên, khi failure là điều được dự đoán trước, việc trả về một `Result` sẽ phù hợp hơn là gọi `panic!`.  
Ví dụ gồm có parser nhận phải dữ liệu malformed, hoặc một HTTP request trả về status cho biết bạn đã chạm rate limit.  
Trong các trường hợp này, trả về `Result` biểu đạt rằng failure là một khả năng được mong đợi, và code gọi hàm phải tự quyết định cách xử lý.

Khi code của bạn thực hiện một thao tác có thể khiến người dùng gặp rủi ro nếu được gọi với các giá trị không hợp lệ, code của bạn nên verify các giá trị đó trước và panic nếu chúng không hợp lệ.  
Việc này chủ yếu vì lý do an toàn: Thử vận hành trên dữ liệu không hợp lệ có thể khiến code của bạn lộ ra các lỗ hổng.  
Đây là lý do chính khiến standard library sẽ gọi `panic!` nếu bạn cố truy cập vùng nhớ out-of-bounds: Cố truy cập vùng nhớ không thuộc về cấu trúc dữ liệu hiện tại là một vấn đề bảo mật rất phổ biến.  
Các function thường có _contract_: Hành vi của chúng chỉ được đảm bảo nếu input thỏa mãn những điều kiện nhất định.  
Panicking khi contract bị vi phạm là hợp lý bởi vì việc vi phạm contract luôn biểu thị một bug phía caller, và đây không phải kiểu lỗi mà bạn muốn code gọi hàm phải explicit handle.  
Thực tế là không có cách hợp lý nào để code gọi hàm có thể recover; chính _lập trình viên_ gọi hàm mới là người cần sửa code.  
Các contract cho một function, đặc biệt là khi vi phạm nó sẽ gây panic, nên được giải thích trong API documentation của function đó.

Tuy nhiên, nếu thêm quá nhiều check error vào mọi function thì sẽ rất dài dòng và khó chịu.  
May mắn là bạn có thể dùng type system của Rust (và vì thế là compile-time type checking) để làm giúp rất nhiều check.  
Nếu function của bạn nhận một parameter có type cụ thể, bạn có thể tiếp tục logic code với niềm tin rằng compiler đã đảm bảo bạn có một giá trị hợp lệ.  
Ví dụ, nếu bạn dùng một type thay vì `Option`, program của bạn kỳ vọng **luôn có** giá trị thay vì có thể _không có_ gì.  
Code của bạn khi đó không cần handle hai case `Some` và `None`: Nó chỉ cần handle case chắc chắn có value.  
Code cố gắng truyền “nothing” vào function của bạn thậm chí sẽ không compile, nên function của bạn không cần check case đó lúc runtime.  
Một ví dụ khác là dùng unsigned integer như `u32`, vốn đảm bảo rằng parameter không bao giờ âm.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-custom-types-for-validation"></a>

### Custom Types cho việc Validation

Hãy đi xa hơn một bước với ý tưởng dùng type system của Rust để đảm bảo chúng ta luôn có một giá trị hợp lệ bằng cách tạo một custom type cho việc validation.  
Nhớ lại game đoán số ở Chapter 2, nơi code của chúng ta yêu cầu user đoán một số từ 1 đến 100.  
Chúng ta chưa bao giờ validate rằng guess của user nằm trong khoảng các số đó trước khi so sánh nó với secret number; chúng ta chỉ validate rằng guess là số dương.  
Trong trường hợp này, hậu quả không quá nghiêm trọng: Output “Too high” hoặc “Too low” vẫn sẽ đúng.  
Tuy nhiên, việc cải tiến để hướng user tới các guess hợp lệ và có behavior khác nhau khi user đoán một số out-of-range so với khi user nhập, chẳng hạn, chữ cái, sẽ là một enhancement hữu ích.

Một cách để làm điều này là parse guess dưới dạng `i32` thay vì chỉ `u32` để cho phép số âm, rồi thêm một check xem số đó có nằm trong khoảng hay không, như sau:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-09-guess-out-of-range/src/main.rs:here}}
```

</Listing>

Biểu thức `if` sẽ check xem giá trị của chúng ta có nằm ngoài range không, thông báo cho user về vấn đề, và gọi `continue` để bắt đầu iteration tiếp theo của loop và hỏi một guess khác.  
Sau biểu thức `if`, chúng ta có thể tiếp tục so sánh `guess` với secret number với niềm tin rằng `guess` đang nằm trong khoảng từ 1 đến 100.

Tuy nhiên, đây không phải giải pháp lý tưởng: Nếu việc program chỉ được phép hoạt động trên các giá trị từ 1 đến 100 là cực kỳ quan trọng, và có nhiều function đều yêu cầu điều này, thì việc lặp lại check như vậy ở mọi function sẽ rất tẻ nhạt (và có thể ảnh hưởng performance).

Thay vào đó, chúng ta có thể tạo một type mới trong một module riêng và đặt logic validation vào một function dùng để tạo instance của type đó, thay vì lặp lại validation ở khắp nơi.  
Nhờ đó, các function có thể dùng type mới này trong signature của chúng và tự tin dùng các giá trị nhận được.  
Listing 9-13 cho thấy một cách định nghĩa type `Guess` chỉ tạo instance `Guess` nếu function `new` nhận vào một giá trị nằm trong khoảng từ 1 đến 100.

<Listing number="9-13" caption="Type `Guess` chỉ tiếp tục với các giá trị từ 1 đến 100" file-name="src/guessing_game.rs">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-13/src/guessing_game.rs}}
```

</Listing>

Lưu ý rằng code trong *src/guessing_game.rs* phụ thuộc vào việc thêm một module declaration `mod guessing_game;` trong *src/lib.rs* mà chúng ta không show ở đây.  
Bên trong file của module mới này, chúng ta định nghĩa một struct tên `Guess` với một field tên `value` giữ một `i32`.  
Đây là nơi number sẽ được lưu.

Tiếp theo, chúng ta implement một associated function tên `new` trên `Guess` để tạo các instance `Guess`.  
Function `new` được định nghĩa nhận một parameter `value` kiểu `i32` và trả về một `Guess`.  
Code trong body của `new` sẽ test `value` để chắc chắn rằng nó nằm trong khoảng từ 1 đến 100.  
Nếu `value` không qua được test, chúng ta gọi `panic!`, điều này sẽ cảnh báo lập trình viên đang viết code gọi hàm rằng họ có một bug cần phải sửa, vì tạo một `Guess` với `value` nằm ngoài khoảng này sẽ vi phạm contract mà `Guess::new` đang dựa vào.  
Các điều kiện mà trong đó `Guess::new` có thể panic nên được đề cập trong public API documentation; chúng ta sẽ bàn về convention document chỉ ra khả năng `panic!` trong API mà bạn tạo ở Chapter 14.  
Nếu `value` qua được test, chúng ta tạo một `Guess` mới với field `value` được set bằng parameter `value` và trả về `Guess` đó.

Tiếp theo, chúng ta implement một method tên `value` nhận `&self`, không có parameter nào khác, và trả về một `i32`.  
Kiểu method này đôi khi được gọi là _getter_ vì mục đích của nó là lấy data từ các field và trả về.  
Public method này là cần thiết vì field `value` của struct `Guess` là private.  
Việc `value` là private rất quan trọng để code sử dụng `Guess` không thể tự ý set `value` trực tiếp: Code bên ngoài module `guessing_game` _bắt buộc_ phải dùng function `Guess::new` để tạo instance `Guess`, nhờ đó đảm bảo không có `Guess` nào có `value` mà chưa được check bởi các điều kiện trong `Guess::new`.

Một function có parameter hoặc return value chỉ là những số từ 1 đến 100 khi đó có thể khai báo trong signature rằng nó nhận hoặc trả về `Guess` thay vì `i32` và sẽ không cần thêm bất kỳ check nào nữa trong body.

## Tổng kết

Các tính năng error-handling của Rust được thiết kế để giúp bạn viết code robust hơn.  
Macro `panic!` báo hiệu rằng program của bạn đang ở một trạng thái không thể handle được và cho phép bạn dừng process thay vì cố tiếp tục với các giá trị invalid hoặc sai.  
Enum `Result` dùng type system của Rust để chỉ ra rằng một operation có thể fail theo một cách mà code của bạn có thể recover.  
Bạn có thể dùng `Result` để báo cho code gọi hàm biết rằng nó cần handle cả khả năng success và failure.  
Dùng `panic!` và `Result` đúng chỗ sẽ giúp code của bạn reliable hơn trước những vấn đề tất yếu xảy ra.

Giờ khi bạn đã thấy những cách hữu ích mà standard library dùng generics với enum `Option` và `Result`, chúng ta sẽ nói về cách generics hoạt động và làm sao bạn có thể dùng chúng trong code của mình.

[encoding]: ch18-03-oo-design-patterns.html#encoding-states-and-behavior-as-types
