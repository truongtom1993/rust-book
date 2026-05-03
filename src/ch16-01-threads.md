## Sử dụng Threads để Chạy Code Đồng thời

Trong hầu hết các hệ điều hành hiện tại, code của một chương trình được chạy trong một
_process_, và hệ điều hành sẽ quản lý nhiều process cùng một lúc.
Trong một chương trình, bạn cũng có thể có những phần độc lập chạy đồng thời.
Các tính năng chạy những phần độc lập này được gọi là _threads_. Ví dụ,
một web server có thể có nhiều threads để nó có thể phản hồi
nhiều hơn một request cùng một lúc.

Chia tính toán trong chương trình của bạn thành nhiều threads để chạy nhiều
task cùng một lúc có thể cải thiện hiệu suất, nhưng nó cũng thêm độ phức tạp.
Vì các threads có thể chạy đồng thời, không có bảo đảm nào về
thứ tự mà những phần code của bạn trên các threads khác nhau sẽ chạy. Điều này có thể dẫn
đến các vấn đề, chẳng hạn như:

- Race conditions, trong đó các threads đang truy cập dữ liệu hoặc tài nguyên
  theo thứ tự không nhất quán
- Deadlocks, trong đó hai threads đang chờ nhau, ngăn cả hai
  threads tiếp tục
- Bugs chỉ xảy ra trong những tình huống nhất định và khó tái tạo và sửa
  một cách đáng tin cậy

Rust cố gắng giảm thiểu các tác động tiêu cực của việc sử dụng threads, nhưng
lập trình trong bối cảnh multithreaded vẫn cần suy tư cẩn thận và yêu cầu
một cấu trúc code khác với các chương trình chạy trong một
thread duy nhất.

Các ngôn ngữ lập trình triển khai threads theo một vài cách khác nhau, và nhiều
hệ điều hành cung cấp một API mà ngôn ngữ lập trình có thể gọi để tạo
các threads mới. Thư viện tiêu chuẩn Rust sử dụng mô hình _1:1_ của thread
implementation, theo đó một chương trình sử dụng một operating system thread cho mỗi
language thread. Có các crates triển khai các mô hình threading khác
tạo ra những trade-off khác nhau với mô hình 1:1. (Hệ thống async của Rust, mà chúng ta sẽ
thấy trong chương tiếp theo, cũng cung cấp một cách tiếp cận khác để xử lý concurrency.)

### Tạo một Thread Mới với `spawn`

Để tạo một thread mới, chúng ta gọi hàm `thread::spawn` và truyền cho nó một
closure (chúng ta đã nói về closures trong Chapter 13) chứa code mà chúng ta muốn
chạy trong thread mới. Ví dụ trong Listing 16-1 in một số văn bản từ một main
thread và văn bản khác từ một thread mới.

<Listing number="16-1" file-name="src/main.rs" caption="Tạo một thread mới để in một thứ gì đó trong khi main thread in thứ khác">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-01/src/main.rs}}
```

</Listing>

Lưu ý rằng khi main thread của một chương trình Rust hoàn thành, tất cả các spawned threads
bị tắt, bất kể chúng đã chạy xong hay chưa. Output từ
chương trình này có thể hơi khác mỗi lần, nhưng nó sẽ trông tương tự như
sau:

<!-- Not extracting output because changes to this output aren’t significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

Các gọi đến `thread::sleep` buộc một thread phải dừng lại thực thi của nó trong một khoảng thời gian
ngắn, cho phép một thread khác chạy. Các threads có thể sẽ
lần lượt, nhưng điều đó không được bảo đảm: Nó phụ thuộc vào cách hệ điều hành của bạn
lên lịch các threads. Trong lần chạy này, main thread in trước, mặc dù
lệnh print từ spawned thread xuất hiện trước trong code. Và mặc dù
chúng ta yêu cầu spawned thread in cho đến khi `i` là `9`, nó chỉ được đến `5`
trước khi main thread tắt.

Nếu bạn chạy code này và chỉ thấy output từ main thread, hoặc không thấy bất kỳ
chồng chéo nào, hãy thử tăng các số trong các ranges để tạo thêm cơ hội
cho hệ điều hành chuyển đổi giữa các threads.

<!-- Old headings. Do not remove or links may break. -->

<a id="waiting-for-all-threads-to-finish-using-join-handles"></a>

### Chờ Tất cả Threads Hoàn thành

Code trong Listing 16-1 không chỉ dừng spawned thread sớm hơn dự kiến hầu hết
thời gian vì main thread kết thúc, mà vì không có bảo đảm về
thứ tự mà các threads chạy, chúng ta cũng không thể bảo đảm rằng spawned thread
sẽ được chạy!

Chúng ta có thể sửa vấn đề của spawned thread không chạy hoặc kết thúc
sớm hơn dự kiến bằng cách lưu giá trị trả về của `thread::spawn` trong một biến. Kiểu
trả về của `thread::spawn` là `JoinHandle<T>`. `JoinHandle<T>` là một
owned value mà, khi chúng ta gọi phương thức `join` trên nó, sẽ chờ
thread của nó kết thúc. Listing 16-2 cho thấy cách sử dụng `JoinHandle<T>` của
thread mà chúng ta tạo trong Listing 16-1 và cách gọi `join` để chắc chắn
spawned thread kết thúc trước khi `main` thoát.

<Listing number="16-2" file-name="src/main.rs" caption="Lưu `JoinHandle&lt;T&gt;` từ `thread::spawn` để bảo đảm thread được chạy cho đến hoàn thành">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-02/src/main.rs}}
```

</Listing>

Gọi `join` trên handle sẽ chặn thread đang chạy cho đến khi
thread được đại diện bởi handle kết thúc. _Blocking_ một thread có nghĩa là
thread bị ngăn không thể thực hiện công việc hoặc thoát. Vì chúng ta đã đặt lệnh gọi
đến `join` sau vòng `for` của main thread, chạy Listing 16-2 sẽ
tạo ra output tương tự như sau:

<!-- Not extracting output because changes to this output aren’t significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

Hai threads tiếp tục thay phiên nhau, nhưng main thread chờ vì
lệnh gọi `handle.join()` và không kết thúc cho đến khi spawned thread hoàn thành.

Nhưng hãy xem chuyện gì sẽ xảy ra khi chúng ta thay vào đó di chuyển `handle.join()` trước
vòng `for` trong `main`, như thế này:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/no-listing-01-join-too-early/src/main.rs}}
```

</Listing>

Main thread sẽ chờ spawned thread kết thúc và sau đó chạy
vòng `for` của nó, vì vậy output sẽ không bị xen kẽ nữa, như thể hiện ở đây:

<!-- Not extracting output because changes to this output aren’t significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

Những chi tiết nhỏ, chẳng hạn như vị trí gọi `join`, có thể ảnh hưởng đến việc
threads của bạn có chạy cùng một lúc hay không.

### Sử dụng `move` Closures với Threads

Chúng ta thường sẽ sử dụng từ khóa `move` với closures được truyền đến `thread::spawn`
vì closure sẽ nhận ownership của các giá trị mà nó sử dụng từ
environment, do đó chuyển ownership của các giá trị đó từ một thread sang
thread khác. Trong [“Capturing References or Moving Ownership”][capture]<!-- ignore
--> trong Chapter 13, chúng ta đã thảo luận về `move` trong bối cảnh closures. Bây giờ chúng ta sẽ
tập trung hơn vào sự tương tác giữa `move` và `thread::spawn`.

Lưu ý trong Listing 16-1 rằng closure mà chúng ta truyền đến `thread::spawn` không lấy
arguments: Chúng ta không sử dụng bất kỳ dữ liệu nào từ main thread trong
code của spawned thread. Để sử dụng dữ liệu từ main thread trong spawned thread,
closure của spawned thread phải capture các giá trị mà nó cần. Listing 16-3 cho thấy
một nỗ lực để tạo một vector trong main thread và sử dụng nó trong spawned
thread. Tuy nhiên, điều này sẽ không hoạt động được, như bạn sẽ thấy trong một chút.

<Listing number="16-3" file-name="src/main.rs" caption="Cố gắng sử dụng một vector được tạo bởi main thread trong một thread khác">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-03/src/main.rs}}
```

</Listing>

Closure sử dụng `v`, vì vậy nó sẽ capture `v` và làm cho nó trở thành một phần của
environment của closure. Vì `thread::spawn` chạy closure này trong một thread mới, chúng ta
nên có thể truy cập `v` bên trong thread mới đó. Nhưng khi chúng ta compile
ví dụ này, chúng ta gặp lỗi sau:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-03/output.txt}}
```

Rust _infers_ cách capture `v`, và vì `println!` chỉ cần một reference
đến `v`, closure cố gắng borrow `v`. Tuy nhiên, có một vấn đề: Rust không thể
biết spawned thread sẽ chạy bao lâu, vì vậy nó không biết liệu
reference đến `v` sẽ luôn hợp lệ hay không.

Listing 16-4 cung cấp một tình huống có nhiều khả năng có một reference đến `v`
không hợp lệ.

<Listing number="16-4" file-name="src/main.rs" caption="Một thread với một closure cố gắng capture một reference đến `v` từ một main thread mà drops `v`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-04/src/main.rs}}
```

</Listing>

Nếu Rust cho phép chúng ta chạy code này, có khả năng spawned
thread sẽ ngay lập tức được đặt vào nền mà không chạy chút nào. Spawned
thread có một reference đến `v` bên trong, nhưng main thread ngay lập tức
drops `v`, sử dụng hàm `drop` mà chúng ta đã thảo luận trong Chapter 15. Sau đó, khi
spawned thread bắt đầu thực thi, `v` không còn hợp lệ nữa, vì vậy một reference đến nó
cũng không hợp lệ. Ôi không!

Để sửa lỗi compiler trong Listing 16-3, chúng ta có thể sử dụng lời khuyên của error message:

<!-- manual-regeneration
after automatic regeneration, look at listings/ch16-fearless-concurrency/listing-16-03/output.txt and copy the relevant part
-->

```text
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

Bằng cách thêm từ khóa `move` trước closure, chúng ta buộc closure phải lấy
ownership của các giá trị mà nó đang sử dụng thay vì cho phép Rust suy luận rằng nó
nên borrow các giá trị. Sửa đổi Listing 16-3 được hiển thị trong Listing
16-5 sẽ compile và chạy như chúng ta dự định.

<Listing number="16-5" file-name="src/main.rs" caption="Sử dụng từ khóa `move` để buộc closure lấy ownership của các giá trị mà nó sử dụng">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-05/src/main.rs}}
```

</Listing>

Chúng ta có thể bị cứ gạt để thử điều tương tự để sửa code trong Listing 16-4, nơi
main thread gọi `drop` bằng cách sử dụng closure `move`. Tuy nhiên, sửa này sẽ
không hoạt động vì những gì Listing 16-4 đang cố gắng làm bị không cho phép vì một
lý do khác. Nếu chúng ta thêm `move` vào closure, chúng ta sẽ di chuyển `v` vào
environment của closure, và chúng ta không còn có thể gọi `drop` trên nó trong main
thread. Thay vào đó, chúng ta sẽ gặp lỗi compiler này:

```console
{{#include ../listings/ch16-fearless-concurrency/output-only-01-move-drop/output.txt}}
```

Quy tắc ownership của Rust đã cứu chúng ta lần nữa! Chúng ta gặp lỗi từ code trong
Listing 16-3 vì Rust đang bảo thủ và chỉ borrow `v` cho
thread, điều đó có nghĩa là main thread có thể về mặt lý thuyết vô hiệu hóa reference của
spawned thread. Bằng cách nói với Rust di chuyển ownership của `v` đến spawned
thread, chúng ta đang bảo đảm cho Rust rằng main thread sẽ không sử dụng `v` nữa.
Nếu chúng ta thay đổi Listing 16-4 theo cách tương tự, chúng ta sẽ vi phạm các quy tắc
ownership khi chúng ta cố gắng sử dụng `v` trong main thread. Từ khóa `move` ghi đè
mặc định bảo thủ của Rust là borrowing; nó không cho phép chúng ta vi phạm các
quy tắc ownership.

Bây giờ chúng ta đã đề cập đến những gì là threads và những phương thức được cung cấp bởi thread
API, hãy xem xét một số tình huống trong đó chúng ta có thể sử dụng threads.

[capture]: ch13-01-closures.html#capturing-references-or-moving-ownership
