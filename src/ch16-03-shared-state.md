## Đồng Thời Trạng Thái Được Chia Sẻ

Message passing là một cách tốt để xử lý tính đồng thời, nhưng nó không phải
là cách duy nhất. Một phương pháp khác sẽ là nhiều thread truy cập cùng một
dữ liệu được chia sẻ. Hãy xem lại phần này của câu nói từ tài liệu ngôn ngữ
Go: “Không giao tiếp bằng cách chia sẻ bộ nhớ.”

Giao tiếp bằng cách chia sẻ bộ nhớ sẽ trông như thế nào? Ngoài ra, tại sao
những người ưa chuộng message-passing lại cảnh báo không sử dụng chia sẻ bộ
nhớ?

Theo một cách nào đó, channel trong bất kỳ ngôn ngữ lập trình nào cũng tương
tự như ownership duy nhất vì khi bạn chuyển một giá trị xuống channel, bạn
không nên sử dụng giá trị đó nữa. Đồng thời chia sẻ bộ nhớ giống như multiple
ownership: Nhiều thread có thể truy cập cùng một vị trí bộ nhớ tại cùng một
thời điểm. Như bạn thấy trong Chương 15, nơi smart pointer tạo ra multiple
ownership có thể xảy ra, multiple ownership có thể thêm độ phức tạp vì những
chủ sở hữu khác nhau này cần phải quản lý. Hệ thống loại và quy tắc ownership
của Rust rất hữu ích trong việc quản lý này một cách chính xác. Để làm ví dụ,
hãy xem mutex, một trong những primitive đồng thời phổ biến nhất cho bộ nhớ
được chia sẻ.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-mutexes-to-allow-access-to-data-from-one-thread-at-a-time"></a>

### Kiểm Soát Truy Cập Bằng Mutex

_Mutex_ là viết tắt của _mutual exclusion_, theo cách mà mutex chỉ cho phép
một thread truy cập một số dữ liệu tại bất kỳ thời điểm nào. Để truy cập dữ
liệu trong mutex, một thread trước tiên phải báo hiệu rằng nó muốn truy cập
bằng cách yêu cầu có được lock của mutex. _Lock_ là một cấu trúc dữ liệu là
một phần của mutex theo dõi ai hiện có quyền truy cập độc quyền vào dữ liệu.
Do đó, mutex được mô tả là _guarding_ dữ liệu nó giữ thông qua hệ thống lock.

Mutex có danh tiếng khó sử dụng vì bạn phải nhớ hai quy tắc:

1. Bạn phải cố gắng có được lock trước khi sử dụng dữ liệu.
2. Khi bạn xong với dữ liệu mà mutex guards, bạn phải unlock dữ liệu để
   các thread khác có thể có được lock.

Để có một phép ẩu dụ thế giới thực cho một mutex, hãy tưởng tượng một cuộc
thảo luận bảng tại một hội nghị chỉ có một microphone. Trước khi một thành
viên bảng có thể nói, họ phải yêu cầu hoặc báo hiệu rằng họ muốn sử dụng
microphone. Khi họ nhận được microphone, họ có thể nói miễn là họ muốn và sau
đó trao microphone cho thành viên bảng tiếp theo yêu cầu nói. Nếu một thành
viên bảng quên trao microphone khi họ xong với nó, không ai khác có thể nói.
Nếu quản lý microphone được chia sẻ diễn ra sai, bảng sẽ không hoạt động như
dự kiến!

Quản lý mutex có thể vô cùng khó để xử lý đúng, đó là lý do tại sao rất nhiều
người nhiệt tình về channel. Tuy nhiên, nhờ hệ thống loại và quy tắc ownership
của Rust, bạn không thể locking và unlocking sai.

#### API của `Mutex<T>`

Để làm ví dụ về cách sử dụng mutex, hãy bắt đầu bằng cách sử dụng mutex trong
ngữ cảnh single-threaded, như được hiển thị trong Listing 16-12.

<Listing number="16-12" file-name="src/main.rs" caption="Khám phá API của `Mutex<T>` trong ngữ cảnh single-threaded để đơn giản">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-12/src/main.rs}}
```

</Listing>

Như với nhiều loại, chúng ta tạo `Mutex<T>` bằng cách sử dụng hàm liên kết `new`.
Để truy cập dữ liệu bên trong mutex, chúng ta sử dụng phương thức `lock` để có
được lock. Cuộc gọi này sẽ block thread hiện tại để nó không thể làm bất kỳ
công việc nào cho đến khi đến lượt chúng ta có được lock.

Cuộc gọi `lock` sẽ thất bại nếu thread khác giữ lock panic. Trong trường hợp
đó, không ai bao giờ có thể có được lock, vì vậy chúng ta đã chọn `unwrap`
và có thread này panic nếu chúng ta ở trong tình huống đó.

Sau khi chúng ta có được lock, chúng ta có thể coi giá trị trả về, được gọi là
`num` trong trường hợp này, như một reference có thể thay đổi đến dữ liệu bên
trong. Hệ thống loại đảm bảo rằng chúng ta có được lock trước khi sử dụng giá
trị trong `m`. Loại của `m` là `Mutex<i32>`, không phải `i32`, vì vậy chúng ta
_must_ gọi `lock` để có thể sử dụng giá trị `i32`. Chúng ta không thể quên; hệ
thống loại sẽ không cho phép chúng ta truy cập `i32` bên trong nếu không.

Cuộc gọi `lock` trả về một loại gọi là `MutexGuard`, được bao bọc trong một
`LockResult` mà chúng ta xử lý bằng cuộc gọi `unwrap`. Loại `MutexGuard`
triển khai `Deref` để trỏ đến dữ liệu bên trong của chúng ta; loại cũng có
triển khai `Drop` phát hành lock tự động khi `MutexGuard` vượt ra ngoài scope,
điều này xảy ra ở cuối scope bên trong. Do đó, chúng ta không có nguy hiểm
quên phát hành lock và chặn mutex khỏi được sử dụng bởi các thread khác vì
phát hành lock xảy ra tự động.

Sau khi drop lock, chúng ta có thể in giá trị mutex và thấy rằng chúng ta đã
có thể thay đổi `i32` bên trong thành `6`.

<!-- Old headings. Do not remove or links may break. -->

<a id="sharing-a-mutext-between-multiple-threads"></a>

#### Truy Cập Được Chia Sẻ đến `Mutex<T>`

Bây giờ hãy cố gắng chia sẻ một giá trị giữa nhiều thread bằng cách sử dụng
`Mutex<T>`. Chúng ta sẽ quay lên 10 thread và có mỗi cái tăng giá trị counter
lên 1, vì vậy counter sẽ đi từ 0 đến 10. Ví dụ trong Listing 16-13 sẽ có một
lỗi compiler, và chúng ta sẽ sử dụng lỗi đó để tìm hiểu thêm về sử dụng
`Mutex<T>` và cách Rust giúp chúng ta sử dụng nó một cách chính xác.

<Listing number="16-13" file-name="src/main.rs" caption="Mười thread, mỗi cái tăng counter được guard bởi một `Mutex<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-13/src/main.rs}}
```

</Listing>

Chúng ta tạo biến `counter` để giữ `i32` bên trong `Mutex<T>`, như chúng ta
đã làm trong Listing 16-12. Tiếp theo, chúng ta tạo 10 thread bằng cách lặp
qua một loạt các số. Chúng ta sử dụng `thread::spawn` và cung cấp tất cả các
thread cùng một closure: một cái di chuyển counter vào thread, có được lock
trên `Mutex<T>` bằng cách gọi phương thức `lock`, và sau đó thêm 1 vào giá
trị trong mutex. Khi một thread hoàn thành chạy closure của nó, `num` sẽ vượt
ra ngoài scope và phát hành lock để một thread khác có thể có được nó.

Trong thread chính, chúng ta thu thập tất cả các join handle. Sau đó, như
chúng ta đã làm trong Listing 16-2, chúng ta gọi `join` trên mỗi handle để
đảm bảo tất cả các thread hoàn thành. Tại thời điểm đó, thread chính sẽ có
được lock và in kết quả của chương trình này.

Chúng ta đã gợi ý rằng ví dụ này sẽ không biên dịch. Bây giờ hãy tìm ra lý do!

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-13/output.txt}}
```

Thông báo lỗi nói rằng giá trị `counter` đã được di chuyển trong lần lặp
trước của vòng lặp. Rust đang cho chúng ta biết rằng chúng ta không thể di
chuyển ownership của lock `counter` vào nhiều thread. Hãy sửa lỗi compiler
bằng phương thức multiple-ownership mà chúng ta đã thảo luận trong Chương 15.

#### Multiple Ownership với Multiple Threads

Trong Chương 15, chúng ta đã cung cấp một giá trị cho nhiều chủ sở hữu bằng
cách sử dụng smart pointer `Rc<T>` để tạo một giá trị reference-counted. Hãy
làm điều tương tự ở đây và xem điều gì sẽ xảy ra. Chúng ta sẽ bao bọc `Mutex<T>`
trong `Rc<T>` trong Listing 16-14 và clone `Rc<T>` trước khi di chuyển ownership
đến thread.

<Listing number="16-14" file-name="src/main.rs" caption="Cố gắng sử dụng `Rc<T>` để cho phép nhiều thread sở hữu `Mutex<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-14/src/main.rs}}
```

</Listing>

Một lần nữa, chúng ta biên dịch và nhận... các lỗi khác nhau! Compiler đang dạy
chúng ta rất nhiều:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-14/output.txt}}
```

Wow, thông báo lỗi đó rất dài dòng! Đây là phần quan trọng để tập trung:
`` `Rc<Mutex<i32>>` cannot be sent between threads safely ``. Compiler cũng
đang cho chúng ta biết lý do tại sao: `` the trait `Send` is not implemented
for `Rc<Mutex<i32>>` ``. Chúng ta sẽ nói về `Send` trong phần tiếp theo: Đây
là một trong những trait đảm bảo rằng các loại chúng ta sử dụng với thread
được dùng trong các tình huống đồng thời.

Thật không may, `Rc<T>` không an toàn để chia sẻ qua các thread. Khi `Rc<T>`
quản lý reference count, nó thêm vào count cho mỗi cuộc gọi `clone` và trừ
đi khỏi count khi mỗi clone bị drop. Nhưng nó không sử dụng bất kỳ primitive
đồng thời nào để đảm bảo rằng các thay đổi đối với count không thể bị gián
đoạn bởi thread khác. Điều này có thể dẫn đến đếm sai—các lỗi tinh tế có thể
dẫn đến rò rỉ bộ nhớ hoặc một giá trị bị drop trước khi chúng ta xong với nó.
Những gì chúng ta cần là một loại chính xác như `Rc<T>`, nhưng thực hiện các
thay đổi đối với reference count theo cách thread-safe.

#### Atomic Reference Counting với `Arc<T>`

May mắn thay, `Arc<T>` _is_ một loại như `Rc<T>` mà an toàn để sử dụng trong
các tình huống đồng thời. _a_ là viết tắt của _atomic_, nghĩa là đó là một
loại _atomically reference-counted_. Atomics là một loại primitive đồng thời
bổ sung mà chúng ta sẽ không đề cập chi tiết ở đây: Xem tài liệu thư viện
chuẩn cho [`std::sync::atomic`][atomic]<!-- ignore --> để biết thêm chi tiết.
Tại thời điểm này, bạn chỉ cần biết rằng atomics hoạt động giống như loại
primitive nhưng an toàn để chia sẻ qua các thread.

Bạn có thể sau đó tự hỏi tại sao tất cả các loại primitive không phải là
atomic và tại sao các loại thư viện chuẩn không được triển khai để sử dụng
`Arc<T>` theo mặc định. Lý do là thread safety đi kèm với một hình phạt
hiệu suất mà bạn chỉ muốn trả tiền khi bạn thực sự cần. Nếu bạn chỉ thực
hiện các hoạt động trên các giá trị trong một thread duy nhất, mã của bạn
có thể chạy nhanh hơn nếu nó không phải thực thi các bảo đảm mà atomics cung
cấp.

Hãy quay lại ví dụ của chúng ta: `Arc<T>` và `Rc<T>` có cùng một API, vì vậy
chúng ta sửa chương trình của chúng ta bằng cách thay đổi dòng `use`, cuộc gọi
`new`, và cuộc gọi `clone`. Mã trong Listing 16-15 cuối cùng sẽ biên dịch và
chạy.

<Listing number="16-15" file-name="src/main.rs" caption="Sử dụng `Arc<T>` để bao bọc `Mutex<T>` để có thể chia sẻ ownership trên nhiều thread">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-15/src/main.rs}}
```

</Listing>

Mã này sẽ in như sau:

<!-- Not extracting output because changes to this output aren’t significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Result: 10
```

Chúng ta đã làm nó! Chúng ta đã đếm từ 0 đến 10, điều này có vẻ không ấn tượng
lắm, nhưng nó đã dạy chúng ta rất nhiều về `Mutex<T>` và thread safety. Bạn cũng
có thể sử dụng cấu trúc của chương trình này để thực hiện các hoạt động phức tạp
hơn là chỉ tăng một counter. Sử dụng chiến lược này, bạn có thể chia một phép
tính thành các phần độc lập, chia các phần đó trên các thread, và sau đó sử dụng
`Mutex<T>` để có mỗi thread cập nhật kết quả cuối cùng với phần của nó.

Lưu ý rằng nếu bạn đang thực hiện các hoạt động số đơn giản, có các loại đơn
giản hơn các loại `Mutex<T>` được cung cấp bởi [mô-đun `std::sync::atomic` của
thư viện chuẩn][atomic]<!-- ignore -->. Các loại này cung cấp truy cập atomic,
đồng thời an toàn đến các loại primitive. Chúng ta chọn sử dụng `Mutex<T>` với
một loại primitive cho ví dụ này để chúng ta có thể tập trung vào cách `Mutex<T>`
hoạt động.

<!-- Old headings. Do not remove or links may break. -->

<a id="similarities-between-refcelltrct-and-mutextarct"></a>

### So Sánh `RefCell<T>`/`Rc<T>` và `Mutex<T>`/`Arc<T>`

Bạn có thể nhận thấy rằng `counter` là immutable nhưng chúng ta có thể có một
reference có thể thay đổi đến giá trị bên trong nó; điều này có nghĩa `Mutex<T>`
cung cấp interior mutability, như họ `Cell` làm. Theo cách tương tự chúng ta sử
dụng `RefCell<T>` trong Chương 15 để cho phép chúng ta mutate nội dung bên trong
`Rc<T>`, chúng ta sử dụng `Mutex<T>` để mutate nội dung bên trong `Arc<T>`.

Chi tiết khác để lưu ý là Rust không thể bảo vệ bạn khỏi tất cả các loại lỗi
logic khi bạn sử dụng `Mutex<T>`. Nhớ lại từ Chương 15 rằng sử dụng `Rc<T>`
đi kèm với rủi ro tạo ra reference cycles, nơi hai giá trị `Rc<T>` đề cập đến
nhau, gây ra rò rỉ bộ nhớ. Tương tự, `Mutex<T>` đi kèm với rủi ro tạo ra
_deadlocks_. Những điều này xảy ra khi một hoạt động cần khóa hai tài nguyên
và hai thread mỗi cái đã có được một trong các lock, khiến chúng chờ đợi nhau
mãi mãi. Nếu bạn quan tâm đến deadlock, hãy cố gắng tạo một chương trình Rust
có deadlock; sau đó, nghiên cứu các chiến lược giảm thiểu deadlock cho mutex
trong bất kỳ ngôn ngữ nào và cố gắng triển khai chúng trong Rust. Tài liệu API
thư viện chuẩn cho `Mutex<T>` và `MutexGuard` cung cấp thông tin hữu ích.

Chúng ta sẽ kết thúc chương này bằng cách nói về các trait `Send` và `Sync` và
cách chúng ta có thể sử dụng chúng với các loại tùy chỉnh.

[atomic]: ../std/sync/atomic/index.html
