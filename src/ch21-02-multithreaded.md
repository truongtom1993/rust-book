<!-- Old headings. Do not remove or links may break. -->

<a id="turning-our-single-threaded-server-into-a-multithreaded-server"></a>
<a id="from-single-threaded-to-multithreaded-server"></a>

## Từ một Web Server Đơn Luồng đến Web Server Đa Luồng

Ngay bây giờ, server sẽ xử lý mỗi yêu cầu theo lượt, có nghĩa là nó sẽ không xử lý kết nối thứ hai cho đến khi kết nối đầu tiên hoàn thành xử lý. Nếu server nhận được ngày càng nhiều yêu cầu, quá trình thực thi nối tiếp này sẽ kém tối ưu hơn. Nếu server nhận được một yêu cầu mất nhiều thời gian để xử lý, các yêu cầu tiếp theo sẽ phải chờ cho đến khi yêu cầu dài hoàn thành, ngay cả khi các yêu cầu mới có thể được xử lý nhanh chóng. Chúng ta sẽ cần phải khắc phục điều này, nhưng trước tiên chúng ta sẽ xem xét vấn đề đang hoạt động.

<!-- Old headings. Do not remove or links may break. -->

<a id="simulating-a-slow-request-in-the-current-server-implementation"></a>

### Mô Phỏng một Yêu Cầu Chậm

Chúng ta sẽ xem xét cách một yêu cầu xử lý chậm có thể ảnh hưởng đến các yêu cầu khác được tạo cho triển khai server hiện tại của chúng ta. Listing 21-10 triển khai xử lý một yêu cầu đến _/sleep_ với một phản hồi chậm mô phỏng sẽ khiến server ngủ trong năm giây trước khi phản hồi.

<Listing number="21-10" file-name="src/main.rs" caption="Mô phỏng một yêu cầu chậm bằng cách ngủ trong năm giây">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-10/src/main.rs:here}}
```

</Listing>

Chúng ta đã chuyển từ `if` sang `match` bây giờ chúng ta có ba trường hợp. Chúng ta cần khớp một cách rõ ràng trên một lát của `request_line` để pattern-match với các giá trị chuỗi nghĩa đen; `match` không tự động referencing và dereferencing như phương thức bằng nhau làm.

Cánh tay đầu tiên giống như khối `if` từ Listing 21-9. Cánh tay thứ hai khớp một yêu cầu để _/sleep_. Khi yêu cầu đó được nhận, server sẽ ngủ trong năm giây trước khi hiển thị trang HTML thành công. Cánh tay thứ ba giống như khối `else` từ Listing 21-9.

Bạn có thể thấy server của chúng ta như thế nào: Thư viện thực sẽ xử lý việc công nhận nhiều yêu cầu theo một cách ít dài dòng hơn nhiều!

Bắt đầu server sử dụng `cargo run`. Sau đó, mở hai cửa sổ trình duyệt: một cho _http://127.0.0.1:7878_ và một cho _http://127.0.0.1:7878/sleep_. Nếu bạn nhập URI _/_ một vài lần, như trước đó, bạn sẽ thấy nó phản hồi nhanh chóng. Nhưng nếu bạn nhập _/sleep_ và sau đó tải _/_, bạn sẽ thấy rằng _/_ chờ cho đến khi `sleep` đã ngủ trong năm giây đầy đủ trước khi tải.

Có nhiều kỹ thuật chúng ta có thể sử dụng để tránh các yêu cầu sao lưu phía sau một yêu cầu chậm, bao gồm việc sử dụng async như chúng ta đã làm Chương 17; cái mà chúng ta sẽ triển khai là một thread pool.

### Cải Thiện Thông Lượng với một Thread Pool

Một _thread pool_ là một nhóm các luồng được sinh ra sẵn sàng và chờ đợi để xử lý một nhiệm vụ. Khi chương trình nhận được một nhiệm vụ mới, nó gán một trong các luồng trong nhóm cho nhiệm vụ, và luồng đó sẽ xử lý nhiệm vụ. Các luồng còn lại trong nhóm có sẵn để xử lý bất kỳ nhiệm vụ khác nào xuất hiện trong khi luồng đầu tiên đang xử lý. Khi luồng đầu tiên xong xử lý nhiệm vụ của nó, nó được trả lại nhóm các luồng nhàn rỗi, sẵn sàng để xử lý một nhiệm vụ mới. Một thread pool cho phép bạn xử lý các kết nối một cách đồng thời, tăng thông lượng của server của bạn.

Chúng ta sẽ giới hạn số lượng luồng trong nhóm thành một số nhỏ để bảo vệ chúng ta khỏi các cuộc tấn công DoS; nếu chúng ta để chương trình của mình tạo một luồng mới cho mỗi yêu cầu khi nó đến, ai đó tạo 10 triệu yêu cầu cho server của chúng ta có thể gây ra hỗn loạn bằng cách sử dụng hết tất cả các tài nguyên của server của chúng ta và làm cho việc xử lý các yêu cầu trở nên chậm.

Thay vì sinh ra các luồng không giới hạn, do đó, chúng ta sẽ có một số lượng luồng cố định chờ trong nhóm. Các yêu cầu xuất hiện được gửi đến nhóm để xử lý. Nhóm sẽ duy trì một hàng đợi các yêu cầu đến. Mỗi một trong các luồng trong nhóm sẽ bật một yêu cầu từ hàng đợi này, xử lý yêu cầu, và sau đó yêu cầu hàng đợi cho một yêu cầu khác. Với thiết kế này, chúng ta có thể xử lý tối đa _`N`_ yêu cầu đồng thời, trong đó _`N`_ là số lượng luồng. Nếu mỗi luồng đang phản hồi một yêu cầu chạy lâu dài, các yêu cầu tiếp theo vẫn có thể sao lưu trong hàng đợi, nhưng chúng ta đã tăng số lượng yêu cầu chạy lâu dài mà chúng ta có thể xử lý trước khi đạt đến điểm đó.

Kỹ thuật này chỉ là một trong nhiều cách để cải thiện thông lượng của một web server. Các tùy chọn khác mà bạn có thể khám phá là mô hình fork/join, mô hình async I/O đơn luồng, và mô hình async I/O đa luồng. Nếu bạn quan tâm đến chủ đề này, bạn có thể đọc thêm về các giải pháp khác và cố gắng triển khai chúng; với một ngôn ngữ cấp thấp như Rust, tất cả các tùy chọn này đều có thể.

Trước khi chúng ta bắt đầu triển khai một thread pool, hãy nói về những gì sử dụng nhóm sẽ trông như thế nào. Khi bạn đang cố gắng thiết kế mã, viết giao diện máy khách trước tiên có thể giúp hướng dẫn thiết kế của bạn. Viết API của mã để nó được cấu trúc theo cách bạn muốn gọi nó; sau đó, triển khai chức năng trong cấu trúc đó thay vì triển khai chức năng và sau đó thiết kế API công khai.

Tương tự như cách chúng ta đã sử dụng phát triển hướng đơn vị trong dự án trong Chương 12, chúng ta sẽ sử dụng phát triển được hướng dẫn bởi trình biên dịch ở đây. Chúng ta sẽ viết mã gọi các hàm mà chúng ta muốn, và sau đó chúng ta sẽ xem các lỗi từ trình biên dịch để xác định những gì chúng ta nên thay đổi tiếp theo để mã hoạt động. Trước khi chúng ta làm điều đó, tuy nhiên, chúng ta sẽ khám phá kỹ thuật mà chúng ta không sẽ sử dụng như một điểm bắt đầu.

<!-- Old headings. Do not remove or links may break. -->

<a id="code-structure-if-we-could-spawn-a-thread-for-each-request"></a>

#### Sinh một Luồng cho Mỗi Yêu Cầu

Đầu tiên, hãy khám phá cách mã của chúng ta có thể trông nếu nó thực sự tạo một luồng mới cho mỗi kết nối. Như được đề cập trước đó, đây không phải là kế hoạch cuối cùng của chúng ta do các vấn đề với việc có khả năng sinh ra một số lượng vô hạn luồng, nhưng nó là một điểm bắt đầu để có được một web server đa luồng hoạt động trước. Sau đó, chúng ta sẽ thêm thread pool như một cải tiến, và so sánh hai giải pháp sẽ dễ dàng hơn.

Listing 21-11 cho thấy các thay đổi để tạo cho `main` để sinh một luồng mới để xử lý mỗi stream trong vòng lặp `for`.

<Listing number="21-11" file-name="src/main.rs" caption="Sinh một luồng mới cho mỗi stream">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-11/src/main.rs:here}}
```

</Listing>

Như bạn đã tìm hiểu trong Chương 16, `thread::spawn` sẽ tạo một luồng mới và sau đó chạy mã trong bao đóng trong luồng mới. Nếu bạn chạy mã này và tải _/sleep_ trong trình duyệt của bạn, sau đó _/_ trong hai thẻ trình duyệt khác nữa, bạn sẽ thực sự thấy rằng các yêu cầu cho _/_ không phải chờ để _/sleep_ kết thúc. Tuy nhiên, như chúng ta đã đề cập, điều này sẽ cuối cùng làm choáng ngợp hệ thống vì bạn sẽ tạo các luồng mới mà không có bất kỳ giới hạn nào.

Bạn cũng có thể nhớ lại từ Chương 17 rằng đây chính xác là loại tình huống mà async và await thực sự tỏa sáng! Hãy nhớ điều đó khi chúng ta xây dựng thread pool và suy nghĩ về cách mọi thứ sẽ trông khác nhau hoặc giống với async.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-similar-interface-for-a-finite-number-of-threads"></a>

#### Tạo một Số Lượng Luồng Hữu Hạn

Chúng ta muốn thread pool của chúng ta hoạt động theo cách tương tự, quen thuộc để chuyển đổi từ các luồng sang một thread pool không yêu cầu các thay đổi lớn đối với mã sử dụng API của chúng ta. Listing 21-12 hiển thị giao diện giả thuyết cho một struct `ThreadPool` mà chúng ta muốn sử dụng thay vì `thread::spawn`.

<Listing number="21-12" file-name="src/main.rs" caption="Giao diện `ThreadPool` lý tưởng của chúng ta">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-12/src/main.rs:here}}
```

</Listing>

Chúng ta sử dụng `ThreadPool::new` để tạo một thread pool mới với một số lượng luồng có thể cấu hình, trong trường hợp này là bốn. Sau đó, trong vòng lặp `for`, `pool.execute` có một giao diện tương tự như `thread::spawn` ở chỗ nó nhận một bao đóng mà nhóm sẽ chạy cho mỗi stream. Chúng ta cần triển khai `pool.execute` để nó nhận bao đóng và cung cấp nó cho một luồng trong nhóm để chạy. Mã này chưa sẽ biên dịch, nhưng chúng ta sẽ thử để trình biên dịch có thể hướng dẫn chúng ta cách khắc phục nó.

<!-- Old headings. Do not remove or links may break. -->

<a id="building-the-threadpool-struct-using-compiler-driven-development"></a>

#### Xây Dựng `ThreadPool` Sử Dụng Phát Triển Hướng Dẫn Bởi Trình Biên Dịch

Tạo các thay đổi trong Listing 21-12 để _src/main.rs_, và sau đó hãy sử dụng các lỗi trình biên dịch từ `cargo check` để hướng dẫn phát triển của chúng ta. Dưới đây là lỗi đầu tiên chúng ta nhận được:

```console
{{#include ../listings/ch21-web-server/listing-21-12/output.txt}}
```

Tuyệt vời! Lỗi này cho chúng ta biết chúng ta cần một loại `ThreadPool` hoặc mô-đun, vì vậy chúng ta sẽ xây dựng một ngay bây giờ. Triển khai `ThreadPool` của chúng ta sẽ độc lập với loại công việc mà web server của chúng ta đang làm. Vì vậy, hãy chuyển `hello` crate từ một binary crate thành một library crate để giữ triển khai `ThreadPool` của chúng ta. Sau khi chúng ta thay đổi thành một library crate, chúng ta cũng có thể sử dụng thư viện thread pool riêng biệt cho bất kỳ công việc nào chúng ta muốn làm bằng cách sử dụng một thread pool, không chỉ để phục vụ các yêu cầu web.

Tạo một tệp _src/lib.rs_ chứa những nội dung sau, đó là định nghĩa đơn giản nhất của một struct `ThreadPool` mà chúng ta có thể có cho bây giờ:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/lib.rs}}
```

</Listing>


Sau đó, chỉnh sửa tệp _main.rs_ để đưa `ThreadPool` vào phạm vi từ library crate bằng cách thêm mã sau vào đầu _src/main.rs_:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/main.rs:here}}
```

</Listing>

Mã này vẫn sẽ không hoạt động, nhưng hãy kiểm tra nó lại để nhận lỗi tiếp theo mà chúng ta cần giải quyết:

```console
{{#include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/output.txt}}
```

Lỗi này cho thấy tiếp theo chúng ta cần tạo một hàm liên kết được đặt tên là `new` cho `ThreadPool`. Chúng ta cũng biết rằng `new` cần có một tham số có thể chấp nhận `4` làm đối số và sẽ trả về một phiên bản `ThreadPool`. Hãy triển khai hàm `new` đơn giản nhất sẽ có những đặc điểm đó:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/src/lib.rs}}
```

</Listing>

Chúng ta đã chọn `usize` làm loại của tham số `size` vì chúng ta biết rằng một số lượng luồng âm không có ý nghĩa. Chúng ta cũng biết chúng ta sẽ sử dụng `4` này làm số phần tử trong một bộ sưu tập các luồng, đó là những gì loại `usize` dành cho, như được thảo luận trong phần ["Integer Types"][integer-types]<!-- ignore --> trong Chương 3.

Hãy kiểm tra mã lại:

```console
{{#include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/output.txt}}
```

Bây giờ lỗi xảy ra vì chúng ta không có phương thức `execute` trên `ThreadPool`. Nhớ lại từ phần ["Creating a Finite Number of Threads"](#creating-a-finite-number-of-threads)<!-- ignore --> rằng chúng ta đã quyết định thread pool của chúng ta nên có giao diện tương tự như `thread::spawn`. Ngoài ra, chúng ta sẽ triển khai hàm `execute` để nó nhận bao đóng được cung cấp cho nó và cung cấp nó cho một luồng nhàn rỗi trong nhóm để chạy.

Chúng ta sẽ xác định phương thức `execute` trên `ThreadPool` để nhận một bao đóng như một tham số. Nhớ lại từ phần ["Moving Captured Values Out of Closures"][moving-out-of-closures]<!-- ignore --> trong Chương 13 rằng chúng ta có thể nhận các bao đóng như các tham số với ba traits khác nhau: `Fn`, `FnMut`, và `FnOnce`. Chúng ta cần quyết định loại bao đóng nào sử dụng ở đây. Chúng ta biết chúng ta sẽ kết thúc bằng việc làm cái gì đó tương tự như triển khai thư viện tiêu chuẩn `thread::spawn`, vì vậy chúng ta có thể xem những gì ràng buộc mà chữ ký `thread::spawn` có trên tham số của nó. Tài liệu cho chúng ta thấy những điều sau:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

Loại tham số `F` là cái mà chúng ta quan tâm ở đây; loại tham số `T` liên quan đến giá trị trả về, và chúng ta không quan tâm đến điều đó. Chúng ta có thể thấy rằng `spawn` sử dụng `FnOnce` làm ràng buộc trait trên `F`. Điều này có lẽ là những gì chúng ta muốn cũng như, bởi vì chúng ta sẽ cuối cùng truyền đối số chúng ta nhận được trong `execute` để `spawn`. Chúng ta có thể tự tin hơn nữa rằng `FnOnce` là trait mà chúng ta muốn sử dụng bởi vì luồng để chạy một yêu cầu sẽ chỉ thực thi bao đóng của yêu cầu đó một lần, điều này phù hợp với `Once` trong `FnOnce`.

Loại tham số `F` cũng có ràng buộc trait `Send` và ràng buộc lifetime `'static`, những cái hữu ích trong tình huống của chúng ta: Chúng ta cần `Send` để chuyển bao đóng từ luồng này sang luồng khác và `'static` vì chúng ta không biết luồng sẽ mất bao lâu để thực thi. Hãy tạo một phương thức `execute` trên `ThreadPool` sẽ nhận một tham số generic của loại `F` với những ràng buộc này:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-03-define-execute/src/lib.rs:here}}
```

</Listing>

Chúng ta vẫn sử dụng `()` sau `FnOnce` vì `FnOnce` này đại diện cho một bao đóng không nhận tham số và trả về loại đơn vị `()`. Giống như các định nghĩa hàm, loại trả về có thể bị bỏ qua từ chữ ký, nhưng ngay cả khi chúng ta không có tham số, chúng ta vẫn cần dấu ngoặc.

Một lần nữa, đây là triển khai đơn giản nhất của phương thức `execute`: Nó không làm gì cả, nhưng chúng ta chỉ cố gắng làm cho mã biên dịch. Hãy kiểm tra nó lại:

```console
{{#include ../listings/ch21-web-server/no-listing-03-define-execute/output.txt}}
```

Nó biên dịch! Nhưng lưu ý rằng nếu bạn cố gắng `cargo run` và tạo một yêu cầu trong trình duyệt, bạn sẽ thấy các lỗi trong trình duyệt mà chúng ta thấy ở đầu chương. Thư viện của chúng ta không thực sự gọi bao đóng được truyền đến `execute` chưa!

> Lưu ý: Một câu nói bạn có thể nghe được về các ngôn ngữ có trình biên dịch nghiêm ngặt, chẳng hạn như Haskell và Rust, là "Nếu mã biên dịch, nó hoạt động." Nhưng câu nói này không phổ biến đúng. Dự án của chúng ta biên dịch, nhưng nó hoàn toàn không làm gì cả! Nếu chúng ta đang xây dựng một dự án thực, hoàn chỉnh, đây sẽ là một lúc tốt để bắt đầu viết các bài kiểm tra đơn vị để kiểm tra rằng mã biên dịch _và_ có hành vi mà chúng ta muốn.

Hãy xem xét: Điều gì sẽ khác ở đây nếu chúng ta sắp thực thi một tương lai thay vì một bao đóng?

#### Xác Thực Số Lượng Luồng trong `new`

Chúng ta không đang làm bất cứ điều gì với các tham số để `new` và `execute`. Hãy triển khai các thân của các hàm này với hành vi mà chúng ta muốn. Để bắt đầu, hãy suy nghĩ về `new`. Trước đó chúng ta đã chọn một loại không ký cho tham số `size` vì một nhóm với một số lượng luồng âm không có ý nghĩa. Tuy nhiên, một nhóm với luồng số 0 cũng không có ý nghĩa, nhưng 0 là một `usize` hoàn toàn hợp lệ. Chúng ta sẽ thêm mã để kiểm tra rằng `size` lớn hơn 0 trước khi chúng ta trả về một phiên bản `ThreadPool`, và chúng ta sẽ có chương trình panic nếu nó nhận 0 bằng cách sử dụng macro `assert!`, như được hiển thị trong Listing 21-13.

<Listing number="21-13" file-name="src/lib.rs" caption="Triển khai `ThreadPool::new` để panic nếu `size` là 0">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-13/src/lib.rs:here}}
```

</Listing>

Chúng ta cũng đã thêm một số tài liệu cho `ThreadPool` với các nhận xét doc. Lưu ý rằng chúng ta đã tuân theo các thực tiễn tốt về tài liệu bằng cách thêm một phần gọi ra các tình huống trong đó hàm của chúng ta có thể panic, như được thảo luận trong Chương 14. Hãy thử chạy `cargo doc --open` và nhấp vào struct `ThreadPool` để xem tài liệu được tạo cho `new` trông như thế nào!

Thay vì thêm macro `assert!` như chúng ta đã làm ở đây, chúng ta có thể thay đổi `new` thành `build` và trả về `Result` như chúng ta đã làm với `Config::build` trong dự án I/O trong Listing 12-9. Nhưng chúng ta đã quyết định trong trường hợp này rằng việc cố gắng tạo một thread pool mà không có bất kỳ luồng nào sẽ là một lỗi không thể phục hồi được. Nếu bạn cảm thấy tham vọng, hãy thử viết một hàm được đặt tên `build` với chữ ký sau để so sánh với hàm `new`:

```rust,ignore
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### Tạo Không Gian để Lưu Trữ các Luồng

Bây giờ chúng ta có một cách để biết chúng ta có một số lượng luồng hợp lệ để lưu trữ trong nhóm, chúng ta có thể tạo những luồng đó và lưu trữ chúng trong struct `ThreadPool` trước khi trả về struct. Nhưng làm thế nào chúng ta "lưu trữ" một luồng? Hãy xem lại chữ ký `thread::spawn`:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

Hàm `spawn` trả về một `JoinHandle<T>`, trong đó `T` là loại mà bao đóng trả về. Hãy thử sử dụng `JoinHandle` quá và xem điều gì xảy ra. Trong trường hợp của chúng ta, các bao đóng mà chúng ta truyền cho thread pool sẽ xử lý kết nối và không trả về bất cứ điều gì, vì vậy `T` sẽ là loại đơn vị `()`.

Mã trong Listing 21-14 sẽ biên dịch, nhưng nó không tạo bất kỳ luồng nào chưa. Chúng ta đã thay đổi định nghĩa của `ThreadPool` để giữ một vector của các phiên bản `thread::JoinHandle<()>`, khởi tạo vector với sức chứa của `size`, thiết lập một vòng lặp `for` sẽ chạy một số mã để tạo các luồng, và trả về một phiên bản `ThreadPool` chứa chúng.

<Listing number="21-14" file-name="src/lib.rs" caption="Tạo một vector cho `ThreadPool` để giữ các luồng">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-14/src/lib.rs:here}}
```

</Listing>

Chúng ta đã đưa `std::thread` vào phạm vi trong library crate vì chúng ta đang sử dụng `thread::JoinHandle` làm loại của các mục trong vector trong `ThreadPool`.

Khi một kích thước hợp lệ được nhận, `ThreadPool` của chúng ta tạo một vector mới có thể giữ các mục `size`. Hàm `with_capacity` thực hiện cùng một tác vụ như `Vec::new` nhưng có một sự khác biệt quan trọng: Nó pre-cấp phát không gian trong vector. Bởi vì chúng ta biết chúng ta cần lưu trữ các phần tử `size` trong vector, thực hiện cấp phát này phía trước là hiệu quả hơn một chút so với sử dụng `Vec::new`, có kích thước lại khi các phần tử được chèn.

Khi bạn chạy `cargo check` lại, nó sẽ thành công.

<!-- Old headings. Do not remove or links may break. -->
<a id ="a-worker-struct-responsible-for-sending-code-from-the-threadpool-to-a-thread"></a>

#### Gửi Mã từ `ThreadPool` tới một Luồng

Chúng ta để lại một nhận xét trong vòng lặp `for` trong Listing 21-14 liên quan đến việc tạo các luồng. Ở đây, chúng ta sẽ xem xét cách chúng ta thực sự tạo các luồng. Thư viện tiêu chuẩn cung cấp `thread::spawn` như một cách để tạo các luồng, và `thread::spawn` mong muốn nhận một số mã mà luồng sẽ chạy ngay khi luồng được tạo. Tuy nhiên, trong trường hợp của chúng ta, chúng ta muốn tạo các luồng và để chúng _chờ_ cho mã mà chúng ta sẽ gửi sau. Thư viện tiêu chuẩn triển khai threads không bao gồm bất kỳ cách nào để làm điều đó; chúng ta phải triển khai nó theo cách thủ công.

Chúng ta sẽ triển khai hành vi này bằng cách giới thiệu một cấu trúc dữ liệu mới giữa `ThreadPool` và các luồng sẽ quản lý hành vi mới này. Chúng ta sẽ gọi cấu trúc dữ liệu này _Worker_, đây là một thuật ngữ phổ biến trong các triển khai pooling. `Worker` nhận mã cần chạy và chạy mã trong luồng của nó.

Hãy nghĩ về những người làm việc trong bếp của một nhà hàng: Những người lao động chờ cho đến khi các đơn đặt hàng đến từ khách hàng, và sau đó chúng chịu trách nhiệm lấy những đơn đặt hàng đó và điền chúng.

Thay vì lưu trữ một vector của các phiên bản `JoinHandle<()>` trong thread pool, chúng ta sẽ lưu trữ các phiên bản của struct `Worker`. Mỗi `Worker` sẽ lưu trữ một phiên bản `JoinHandle<()>` duy nhất. Sau đó, chúng ta sẽ triển khai một phương thức trên `Worker` sẽ nhận một bao đóng của mã để chạy và gửi nó cho luồng đang chạy để thực thi. Chúng ta cũng sẽ cung cấp cho mỗi `Worker` một `id` để chúng ta có thể phân biệt giữa các phiên bản khác nhau của `Worker` trong nhóm khi ghi nhật ký hoặc gỡ lỗi.

Dưới đây là quy trình mới sẽ xảy ra khi chúng ta tạo một `ThreadPool`. Chúng ta sẽ triển khai mã gửi bao đóng cho luồng sau khi chúng ta có `Worker` được thiết lập theo cách này:

1. Xác định một struct `Worker` giữ một `id` và một `JoinHandle<()>`.
2. Thay đổi `ThreadPool` để giữ một vector của các phiên bản `Worker`.
3. Xác định một hàm `Worker::new` nhận một số `id` và trả về một phiên bản `Worker` giữ `id` và một luồng sinh với một bao đóng trống.
4. Trong `ThreadPool::new`, sử dụng bộ đếm vòng lặp `for` để tạo một `id`, tạo một `Worker` mới với `id` đó, và lưu trữ `Worker` trong vector.

Nếu bạn sẵn sàng cho một thách thức, hãy thử triển khai những thay đổi này của riêng bạn trước khi xem mã trong Listing 21-15.

Sẵn sàng? Đây là Listing 21-15 có một cách để tạo những sửa đổi trước đó.

<Listing number="21-15" file-name="src/lib.rs" caption="Sửa đổi `ThreadPool` để giữ các phiên bản `Worker` thay vì giữ các luồng trực tiếp">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-15/src/lib.rs:here}}
```

</Listing>

Chúng ta đã thay đổi tên của trường trên `ThreadPool` từ `threads` thành `workers` vì nó bây giờ giữ các phiên bản `Worker` thay vì các phiên bản `JoinHandle<()>`. Chúng ta sử dụng bộ đếm trong vòng lặp `for` làm đối số cho `Worker::new`, và chúng ta lưu trữ mỗi `Worker` mới trong vector được đặt tên là `workers`.

Mã bên ngoài (như server của chúng ta trong _src/main.rs_) không cần biết các chi tiết triển khai liên quan đến việc sử dụng một struct `Worker` trong `ThreadPool`, vì vậy chúng ta làm cho struct `Worker` và hàm `new` của nó riêng tư. Hàm `Worker::new` sử dụng `id` chúng ta cung cấp cho nó và lưu trữ một phiên bản `JoinHandle<()>` được tạo bằng cách sinh một luồng mới bằng cách sử dụng một bao đóng trống.

> Lưu ý: Nếu hệ điều hành không thể tạo một luồng vì không có đủ tài nguyên hệ thống, `thread::spawn` sẽ panic. Điều đó sẽ khiến toàn bộ server của chúng ta panic, ngay cả khi việc tạo một số luồng có thể thành công. Vì lợi ích của đơn giản, hành vi này là tốt, nhưng trong một triển khai thread pool sản xuất, bạn có thể muốn sử dụng [`std::thread::Builder`][builder]<!-- ignore --> và [`spawn`][builder-spawn]<!-- ignore --> phương thức của nó trả về `Result` thay vào đó.

Mã này sẽ biên dịch và sẽ lưu trữ số lượng phiên bản `Worker` chúng ta đã chỉ định làm đối số cho `ThreadPool::new`. Nhưng chúng ta _vẫn_ không xử lý bao đóng mà chúng ta nhận được trong `execute`. Hãy xem cách thực hiện điều đó tiếp theo.

#### Gửi Yêu Cầu tới Các Luồng thông qua Channels

Vấn đề tiếp theo chúng ta sẽ giải quyết là các bao đóng được cung cấp cho `thread::spawn` không làm gì cả. Hiện tại, chúng ta nhận được bao đóng mà chúng ta muốn thực thi trong phương thức `execute`. Nhưng chúng ta cần cung cấp cho `thread::spawn` một bao đóng để chạy khi chúng ta tạo mỗi `Worker` trong quá trình tạo `ThreadPool`.

Chúng ta muốn structs `Worker` mà chúng ta vừa tạo để tìm nạp mã để chạy từ một hàng đợi được giữ trong `ThreadPool` và gửi mã đó cho luồng của nó để chạy.

Các channels chúng ta đã tìm hiểu về Chương 16—một cách đơn giản để giao tiếp giữa hai luồng—sẽ hoàn hảo cho trường hợp sử dụng này. Chúng ta sẽ sử dụng một channel để hoạt động như hàng đợi của các công việc, và `execute` sẽ gửi một công việc từ `ThreadPool` tới các phiên bản `Worker`, những phiên bản sẽ gửi công việc cho luồng của nó. Dưới đây là kế hoạch:

1. `ThreadPool` sẽ tạo một channel và giữ trên người gửi.
2. Mỗi `Worker` sẽ giữ trên người nhận.
3. Chúng ta sẽ tạo một struct `Job` mới sẽ giữ các bao đóng mà chúng ta muốn gửi xuống channel.
4. Phương thức `execute` sẽ gửi công việc mà nó muốn thực thi thông qua người gửi.
5. Trong luồng của nó, `Worker` sẽ lặp qua người nhận của nó và thực thi các bao đóng của bất kỳ công việc nào mà nó nhận được.

Hãy bắt đầu bằng cách tạo một channel trong `ThreadPool::new` và giữ người gửi trong phiên bản `ThreadPool`, như được hiển thị trong Listing 21-16. Struct `Job` không giữ bất cứ điều gì vào lúc này nhưng sẽ là loại của mục chúng ta gửi xuống channel.

<Listing number="21-16" file-name="src/lib.rs" caption="Sửa đổi `ThreadPool` để lưu trữ người gửi của một channel truyền các phiên bản `Job`">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-16/src/lib.rs:here}}
```

</Listing>

Trong `ThreadPool::new`, chúng ta tạo channel mới và có nhóm giữ người gửi. Điều này sẽ biên dịch thành công.

Hãy thử truyền một người nhận của channel vào mỗi `Worker` khi thread pool tạo channel. Chúng ta biết chúng ta muốn sử dụng người nhận trong luồng mà các phiên bản `Worker` sinh ra, vì vậy chúng ta sẽ tham chiếu tham số `receiver` trong bao đóng. Mã trong Listing 21-17 sẽ chưa biên dịch.

<Listing number="21-17" file-name="src/lib.rs" caption="Truyền người nhận cho mỗi `Worker`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-17/src/lib.rs:here}}
```

</Listing>

Chúng ta đã tạo một số thay đổi nhỏ và dễ dàng: Chúng ta truyền người nhận vào `Worker::new`, và sau đó chúng ta sử dụng nó bên trong bao đóng.

Khi chúng ta cố gắng kiểm tra mã này, chúng ta nhận được lỗi này:

```console
{{#include ../listings/ch21-web-server/listing-21-17/output.txt}}
```

Mã đang cố gắng truyền `receiver` cho nhiều phiên bản `Worker`. Điều này sẽ không hoạt động, như bạn sẽ nhớ lại từ Chương 16: Triển khai channel mà Rust cung cấp là nhiều _producer_, người tiêu dùng duy nhất. Điều này có nghĩa là chúng ta không thể chỉ sao chép phần tiêu thụ của channel để sửa mã này. Chúng ta cũng không muốn gửi một thông báo nhiều lần cho nhiều người tiêu dùng; chúng ta muốn một danh sách tin nhắn với nhiều phiên bản `Worker` sao cho mỗi thông báo được xử lý một lần.

Ngoài ra, lấy một công việc từ hàng đợi channel liên quan đến việc biến `receiver`, vì vậy các luồng cần một cách an toàn để chia sẻ và sửa đổi `receiver`; nếu không, chúng ta có thể gặp các điều kiện chạy (như được đề cập trong Chương 16).

Nhớ lại các con trỏ thông minh an toàn theo luồng được thảo luận trong Chương 16: Để chia sẻ quyền sở hữu trên nhiều luồng và cho phép các luồng biến đổi giá trị, chúng ta cần sử dụng `Arc<Mutex<T>>`. Loại `Arc` sẽ cho phép các phiên bản `Worker` sở hữu người nhận, và `Mutex` sẽ đảm bảo rằng chỉ có một `Worker` nhận một công việc từ người nhận tại một thời điểm. Listing 21-18 cho thấy những thay đổi chúng ta cần thực hiện.

<Listing number="21-18" file-name="src/lib.rs" caption="Chia sẻ người nhận trong các phiên bản `Worker` bằng cách sử dụng `Arc` và `Mutex`">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-18/src/lib.rs:here}}
```

</Listing>

Trong `ThreadPool::new`, chúng ta đặt người nhận vào một `Arc` và một `Mutex`. Đối với mỗi `Worker` mới, chúng ta sao chép `Arc` để tăng số tham chiếu để các phiên bản `Worker` có thể chia sẻ quyền sở hữu của người nhận.

Với những thay đổi này, mã biên dịch! Chúng ta đang tiến bộ!

#### Triển Khai Phương Thức `execute`

Hãy cuối cùng triển khai phương thức `execute` trên `ThreadPool`. Chúng ta cũng sẽ thay đổi `Job` từ một struct thành một bí danh loại cho một đối tượng trait giữ loại bao đóng mà `execute` nhận. Như được thảo luận trong phần ["Type Synonyms and Type Aliases"][type-aliases]<!-- ignore --> trong Chương 20, các bí danh loại cho phép chúng ta làm cho các loại dài hơn ngắn hơn để dễ sử dụng. Xem Listing 21-19.

<Listing number="21-19" file-name="src/lib.rs" caption="Tạo bí danh loại `Job` cho một `Box` giữ mỗi bao đóng và sau đó gửi công việc xuống channel">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-19/src/lib.rs:here}}
```

</Listing>

Sau khi tạo một phiên bản `Job` mới bằng cách sử dụng bao đóng mà chúng ta nhận được trong `execute`, chúng ta gửi công việc đó xuống cuối người gửi của channel. Chúng ta đang gọi `unwrap` trên `send` cho trường hợp mà việc gửi thất bại. Điều này có thể xảy ra nếu, ví dụ, chúng ta dừng tất cả các luồng của chúng ta khỏi việc thực thi, có nghĩa là cuối người nhận đã ngừng nhận tin nhắn mới. Tại thời điểm này, chúng ta không thể dừng các luồng của chúng ta khỏi thực thi: Các luồng của chúng ta tiếp tục thực thi miễn là nhóm tồn tại. Lý do chúng ta sử dụng `unwrap` là chúng ta biết trường hợp thất bại sẽ không xảy ra, nhưng trình biên dịch không biết điều đó.

Nhưng chúng ta vẫn chưa hoàn thành! Trong `Worker`, bao đóng của chúng ta được truyền đến `thread::spawn` vẫn chỉ _tham chiếu_ cuối người nhận của channel. Thay vào đó, chúng ta cần bao đóng để lặp lại mãi mãi, yêu cầu cuối người nhận của channel để có một công việc và chạy công việc khi nó nhận được một. Hãy thực hiện thay đổi được hiển thị trong Listing 21-20 để `Worker::new`.

<Listing number="21-20" file-name="src/lib.rs" caption="Nhận và thực thi các công việc trong luồng phiên bản `Worker`">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-20/src/lib.rs:here}}
```

</Listing>

Ở đây, chúng ta trước tiên gọi `lock` trên `receiver` để có được mutex, và sau đó chúng ta gọi `unwrap` để panic trên bất kỳ lỗi nào. Có được một khóa có thể thất bại nếu mutex ở trạng thái _poisoned_, điều này có thể xảy ra nếu một luồng khác panic trong khi giữ khóa thay vì giải phóng khóa. Trong tình huống này, gọi `unwrap` để có luồng này panic là hành động chính xác để thực hiện. Vui lòng cảm thấy tự do thay đổi `unwrap` này thành `expect` với một thông báo lỗi có ý nghĩa đối với bạn.

Nếu chúng ta nhận được khóa trên mutex, chúng ta gọi `recv` để nhận một `Job` từ channel. Một `unwrap` cuối cùng chuyển qua bất kỳ lỗi nào ở đây cũng vậy, những lỗi có thể xảy ra nếu luồng giữ người gửi đã tắt, tương tự như cách phương thức `send` trả về `Err` nếu người nhận tắt.

Cuộc gọi `recv` chặn, vì vậy nếu không có công việc nào chưa, luồng hiện tại sẽ chờ cho đến khi một công việc có sẵn. `Mutex<T>` đảm bảo rằng chỉ một luồng `Worker` tại một thời điểm đang cố gắng yêu cầu một công việc.

Thread pool của chúng ta bây giờ ở trạng thái hoạt động! Cung cấp nó một `cargo run` và tạo một số yêu cầu:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-20
cargo run
make some requests to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field `workers` is never read
 --> src/lib.rs:7:5
  |
6 | pub struct ThreadPool {
  |            ---------- field in this struct
7 |     workers: Vec<Worker>,
  |     ^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: fields `id` and `thread` are never read
  --> src/lib.rs:48:5
   |
47 | struct Worker {
   |        ------ fields in this struct
48 |     id: usize,
   |     ^^
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^

warning: `hello` (lib) generated 2 warnings
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 4.91s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
```

Thành công! Chúng ta bây giờ có một thread pool thực thi các kết nối không đồng bộ. Không bao giờ có nhiều hơn bốn luồng được tạo, vì vậy hệ thống của chúng ta sẽ không bị quá tải nếu server nhận được rất nhiều yêu cầu. Nếu chúng ta tạo một yêu cầu tới _/sleep_, server sẽ có thể phục vụ các yêu cầu khác bằng cách có một luồng khác chạy chúng.

> Lưu ý: Nếu bạn mở _/sleep_ trong nhiều cửa sổ trình duyệt cùng một lúc, chúng có thể tải từng cái một theo khoảng thời gian năm giây. Một số trình duyệt web thực thi nhiều phiên bản của yêu cầu tương tự theo trình tự vì lý do bộ nhớ cache. Hạn chế này không phải do web server của chúng ta gây ra.

Đây là một lúc tốt để tạm dừng và xem xét cách mã trong Listings 21-18, 21-19, và 21-20 sẽ khác nhau nếu chúng ta đang sử dụng futures thay vì một bao đóng cho công việc được thực hiện. Các loại sẽ thay đổi như thế nào? Các chữ ký phương thức sẽ khác nhau như thế nào, nếu có? Các phần mã nào sẽ vẫn giống nhau?

Sau khi tìm hiểu về vòng lặp `while let` trong Chương 17 và Chương 19, bạn có thể tự hỏi tại sao chúng ta không viết mã luồng `Worker` như được hiển thị trong Listing 21-21.

<Listing number="21-21" file-name="src/lib.rs" caption="Một triển khai thay thế của `Worker::new` sử dụng `while let`">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-21/src/lib.rs:here}}
```

</Listing>

Mã này biên dịch và chạy nhưng không dẫn đến hành vi luồng mong muốn: Một yêu cầu chậm vẫn sẽ khiến các yêu cầu khác phải chờ để được xử lý. Lý do là hơi tinh tế: Struct `Mutex` không có phương thức `unlock` công khai vì quyền sở hữu của khóa dựa trên vòng đời của `MutexGuard<T>` trong `LockResult<MutexGuard<T>>` mà phương thức `lock` trả về. Tại thời gian biên dịch, borrow checker có thể thực thi quy tắc rằng một tài nguyên được bảo vệ bởi `Mutex` không thể được truy cập trừ khi chúng ta giữ khóa. Tuy nhiên, triển khai này cũng có thể dẫn đến khóa được giữ lâu hơn dự kiến nếu chúng ta không cẩn thận về vòng đời của `MutexGuard<T>`.

Mã trong Listing 21-20 sử dụng `let job = receiver.lock().unwrap().recv().unwrap();` hoạt động bởi vì với `let`, bất kỳ giá trị tạm thời nào được sử dụng trong biểu thức ở bên phải dấu bằng sẽ bị drop ngay khi câu lệnh `let` kết thúc. Tuy nhiên, `while let` (và `if let` và `match`) không drop giá trị tạm thời cho đến khi kết thúc của khối liên kết. Trong Listing 21-21, khóa vẫn được giữ trong thời gian gọi `job()`, có nghĩa là các phiên bản `Worker` khác không thể nhận công việc.

[type-aliases]: ch20-03-advanced-types.html#type-synonyms-and-type-aliases
[integer-types]: ch03-02-data-types.html#integer-types
[moving-out-of-closures]: ch13-01-closures.html#moving-captured-values-out-of-closures
[builder]: ../std/thread/struct.Builder.html
[builder-spawn]: ../std/thread/struct.Builder.html#method.spawn
