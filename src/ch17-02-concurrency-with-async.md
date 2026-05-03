<!-- Old headings. Do not remove or links may break. -->

<a id="concurrency-with-async"></a>

## Áp Dụng Concurrency Với Async

Trong phần này, chúng ta sẽ áp dụng async cho một số thách thức concurrency tương tự mà chúng ta đã giải quyết với threads trong Chương 16. Vì chúng ta đã nói về rất nhiều ý tưởng chính ở đó, trong phần này chúng ta sẽ tập trung vào những gì khác nhau giữa threads và futures.

Trong nhiều trường hợp, các APIs để làm việc với concurrency sử dụng async rất giống như những APIs để sử dụng threads. Trong những trường hợp khác, chúng kết thúc khác nhau khá. Thậm chí khi các APIs _look_ tương tự giữa threads và async, chúng thường có hành vi khác—và chúng gần như luôn có các đặc tính hiệu suất khác nhau.

<!-- Old headings. Do not remove or links may break. -->

<a id="counting"></a>

### Tạo Một Task Mới Với `spawn_task`

Hoạt động đầu tiên chúng ta giải quyết trong phần ["Creating a New Thread with `spawn`"][thread-spawn]<!-- ignore --> ở Chương 16 là đếm lên trên hai separate threads. Hãy làm điều tương tự sử dụng async. Crate `trpl` cung cấp một function `spawn_task` trông rất giống API `thread::spawn`, và một function `sleep` là một phiên bản async của API `thread::sleep`. Chúng ta có thể sử dụng những điều này cùng nhau để triển khai ví dụ đếm, như thể hiện trong Listing 17-6.

<Listing number="17-6" caption="Tạo một task mới để in một điều trong khi main task in cái gì khác" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-06/src/main.rs:all}}
```

</Listing>

Là điểm xuất phát của chúng ta, chúng ta thiết lập function `main` của chúng ta bằng `trpl::block_on` để function top-level của chúng ta có thể là async.

> Ghi chú: Từ thời điểm này trở đi trong chương, mỗi ví dụ sẽ bao gồm mã wrapping chính xác này với `trpl::block_on` trong `main`, vì vậy chúng ta thường sẽ bỏ qua nó giống như chúng ta làm với `main`. Hãy nhớ bao gồm nó trong mã của bạn!

Sau đó chúng ta viết hai loops trong block đó, mỗi loop chứa một lệnh gọi `trpl::sleep`, chờ nửa giây (500 milliseconds) trước khi gửi thông báo tiếp theo. Chúng ta đặt một loop trong phần thân của `trpl::spawn_task` và cái kia trong một top-level `for` loop. Chúng ta cũng thêm một `await` sau các lệnh gọi `sleep`.

Mã này hành xử tương tự như triển khai dựa trên thread—bao gồm cả thực tế là bạn có thể thấy các thông báo xuất hiện theo một thứ tự khác nhau trong terminal của riêng bạn khi bạn chạy nó:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
```

Phiên bản này dừng ngay khi `for` loop trong phần thân của main async block kết thúc, bởi vì task spawn bởi `spawn_task` bị tắt khi function `main` kết thúc. Nếu bạn muốn nó chạy cho đến khi hoàn thành task, bạn sẽ cần sử dụng một join handle để chờ task đầu tiên hoàn thành. Với threads, chúng ta sử dụng phương thức `join` để "block" cho đến khi thread chạy xong. Trong Listing 17-7, chúng ta có thể sử dụng `await` để làm điều tương tự, vì chính task handle là một future. Kiểu `Output` của nó là một `Result`, vì vậy chúng ta cũng unwrap nó sau khi awaiting nó.

<Listing number="17-7" caption="Sử dụng `await` với một join handle để chạy một task hoàn thành" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-07/src/main.rs:handle}}
```

</Listing>

Phiên bản cập nhật này chạy cho đến khi _cả hai_ loops kết thúc:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

Cho đến nay, nó trông giống như async và threads cho chúng ta kết quả tương tự, chỉ với cú pháp khác: sử dụng `await` thay vì gọi `join` trên join handle, và awaiting các lệnh gọi `sleep`.

Sự khác biệt lớn hơn là chúng ta không cần spawn một operating system thread khác để làm điều này. Trên thực tế, chúng ta thậm chí không cần spawn một task ở đây. Vì async blocks biên dịch thành anonymous futures, chúng ta có thể đặt mỗi loop trong một async block và có runtime chạy cả hai để hoàn thành sử dụng function `trpl::join`.

Trong phần ["Waiting for All Threads to Finish"][join-handles]<!-- ignore --> ở Chương 16, chúng ta đã chỉ ra cách sử dụng phương thức `join` trên kiểu `JoinHandle` được trả về khi bạn gọi `std::thread::spawn`. Function `trpl::join` tương tự, nhưng cho futures. Khi bạn cung cấp nó hai futures, nó tạo ra một future mới duy nhất có output là một tuple chứa output của mỗi future bạn truyền vào một khi _cả hai_ hoàn thành. Do đó, trong Listing 17-8, chúng ta sử dụng `trpl::join` để chờ cả `fut1` và `fut2` kết thúc. Chúng ta _không_ await `fut1` và `fut2` nhưng thay vào đó future mới được tạo ra bởi `trpl::join`. Chúng ta ignore output, vì nó chỉ là một tuple chứa hai unit values.

<Listing number="17-8" caption="Sử dụng `trpl::join` để await hai anonymous futures" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-08/src/main.rs:join}}
```

</Listing>

Khi chúng ta chạy cái này, chúng ta thấy cả hai futures chạy hoàn thành:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the first task!
hi number 1 from the second task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

Bây giờ, bạn sẽ thấy cùng một thứ tự mỗi lần, đó là rất khác với những gì chúng ta thấy với threads và với `trpl::spawn_task` trong Listing 17-7. Đó là bởi vì function `trpl::join` là _fair_, có nghĩa là nó kiểm tra mỗi future như nhau, xen kẽ giữa chúng, và không bao giờ cho phép cái này race ahead nếu cái kia sẵn sàng. Với threads, operating system quyết định thread nào để kiểm tra và bao lâu để cho nó chạy. Với async Rust, runtime quyết định task nào để kiểm tra. (Trên thực tế, các chi tiết trở nên phức tạp vì một async runtime có thể sử dụng operating system threads dưới cùng như một phần của cách nó quản lý concurrency, vì vậy đảm bảo công bằng có thể là nhiều công việc cho runtime—nhưng nó vẫn là khả thi!) Runtimes không phải đảm bảo công bằng cho bất kỳ hoạt động nào, và chúng thường cung cấp các APIs khác nhau để cho phép bạn chọn liệu bạn có muốn công bằng hay không.

Hãy thử một số biến thể trên awaiting futures và xem những gì họ làm:

- Loại bỏ async block từ xung quanh một hoặc cả hai loops.
- Await mỗi async block ngay lập tức sau khi xác định nó.
- Wrap chỉ loop đầu tiên trong một async block, và await future kết quả sau phần thân loop thứ hai.

Để có một thách thức thêm, hãy xem liệu bạn có thể hình dung ra output sẽ là gì trong mỗi case _trước_ chạy mã!

<!-- Old headings. Do not remove or links may break. -->

<a id="message-passing"></a>
<a id="counting-up-on-two-tasks-using-message-passing"></a>

### Gửi Dữ Liệu Giữa Hai Tasks Sử Dụng Message Passing

Chia sẻ dữ liệu giữa futures cũng sẽ quen thuộc: chúng ta sẽ sử dụng message passing lại, nhưng lần này với các phiên bản async của các kiểu và functions. Chúng ta sẽ thực hiện một con đường hơi khác so với chúng ta đã làm trong phần ["Transfer Data Between Threads with Message Passing"][message-passing-threads]<!-- ignore --> ở Chương 16 để minh họa một số khác biệt chính giữa concurrency dựa trên thread và dựa trên futures. Trong Listing 17-9, chúng ta sẽ bắt đầu với chỉ một async block—_không_ spawn một separate task như chúng ta đã spawn một separate thread.

<Listing number="17-9" caption="Tạo một async channel và gán hai halves cho `tx` và `rx`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-09/src/main.rs:channel}}
```

</Listing>

Ở đây, chúng ta sử dụng `trpl::channel`, một phiên bản async của multiple-producer, single-consumer channel API chúng ta đã sử dụng với threads ở Chương 16. Phiên bản async của API chỉ hơi khác so với phiên bản dựa trên thread: nó sử dụng một mutable thay vì một immutable receiver `rx`, và phương thức `recv` của nó tạo ra một future chúng ta cần await thay vì tạo ra giá trị trực tiếp. Bây giờ chúng ta có thể gửi các thông báo từ sender đến receiver. Lưu ý rằng chúng ta không phải spawn một separate thread hoặc thậm chí một task; chúng ta chỉ cần await lệnh gọi `rx.recv`.

Phương thức `Receiver::recv` đồng bộ trong `std::mpsc::channel` chặn cho đến khi nó nhận được một thông báo. Phương thức `trpl::Receiver::recv` không, bởi vì nó là async. Thay vì chặn, nó chuyển điều khiển trở lại runtime cho đến khi một thông báo được nhận hoặc bên gửi của channel đóng. Trong contrast, chúng ta không await lệnh gọi `send`, bởi vì nó không chặn. Nó không cần, bởi vì channel chúng ta đang gửi vào nó là unbounded.

> Ghi chú: Vì tất cả mã async này chạy trong một async block trong một lệnh gọi `trpl::block_on`, mọi thứ bên trong nó có thể tránh chặn. Tuy nhiên, mã _outside_ nó sẽ chặn trên function `block_on` returning. Đó là toàn bộ điểm của function `trpl::block_on`: nó cho phép bạn _chọn_ nơi để chặn trên một số mã async, và do đó nơi để chuyển tiếp giữa mã async và đồng bộ.

Lưu ý hai điều về ví dụ này. Thứ nhất, thông báo sẽ tới ngay lập tức. Thứ hai, mặc dù chúng ta sử dụng một future ở đây, không có concurrency nhưng. Mọi thứ trong listing xảy ra một cách tuần tự, giống như thể nếu không có futures liên quan.

Hãy giải quyết phần đầu tiên bằng cách gửi một loạt các thông báo và ngủ giữa chúng, như thể hiện trong Listing 17-10.

<!-- We cannot test this one because it never stops! -->

<Listing number="17-10" caption="Gửi và nhận các thông báo multi trên async channel và ngủ với một `await` giữa mỗi thông báo" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-10/src/main.rs:many-messages}}
```

</Listing>

Ngoài việc gửi các thông báo, chúng ta cần nhận chúng. Trong trường hợp này, vì chúng ta biết bao nhiêu thông báo đang đến, chúng ta có thể làm điều đó theo cách thủ công bằng cách gọi `rx.recv().await` bốn lần. Trong thế giới thực, tuy nhiên, chúng ta sẽ thường chờ đợi trên một _unknown_ số thông báo, vì vậy chúng ta cần tiếp tục chờ đợi cho đến khi chúng ta xác định rằng không có thêm thông báo nào.

Trong Listing 16-10, chúng ta đã sử dụng một `for` loop để xử lý tất cả các items nhận được từ một channel đồng bộ. Rust chưa có cách để sử dụng `for` loop với một _asynchronously produced_ chuỗi items, tuy nhiên, vì vậy chúng ta cần sử dụng một loop chúng ta chưa thấy trước: the `while let` conditional loop. Đây là phiên bản loop của cấu trúc `if let` chúng ta đã thấy ở phần ["Concise Control Flow with `if let` and `let...else`"][if-let]<!-- ignore --> ở Chương 6. Loop sẽ tiếp tục thực hiện miễn là pattern nó chỉ định tiếp tục khớp với giá trị.

Lệnh gọi `rx.recv` tạo ra một future, mà chúng ta await. Runtime sẽ tạm dừng future cho đến khi nó sẵn sàng. Một khi một thông báo tới, future sẽ resolve thành `Some(message)` nhiều lần như một thông báo tới. Khi channel đóng, bất kể liệu _any_ thông báo đã tới, future sẽ thay vào đó resolve thành `None` để chỉ ra rằng không có thêm các giá trị nào và do đó chúng ta nên dừng polling—tức là, dừng awaiting.

`while let` loop kéo tất cả điều này lại với nhau. Nếu kết quả của lệnh gọi `rx.recv().await` là `Some(message)`, chúng ta nhận được access đến thông báo và chúng ta có thể sử dụng nó trong phần thân loop, giống như chúng ta có thể với `if let`. Nếu kết quả là `None`, loop kết thúc. Mỗi lần loop hoàn thành, nó đạt đến await point lại, vì vậy runtime tạm dừng nó lại cho đến khi một thông báo khác tới.

Mã bây giờ successfully gửi và nhận tất cả các thông báo. Thật không may, có một vài vấn đề. Cho một điều, các thông báo không tới tại các khoảng thời gian nửa giây. Chúng tới tất cả cùng lúc, 2 giây (2,000 milliseconds) sau khi chúng ta bắt đầu chương trình. Để khác, chương trình này cũng không bao giờ thoát! Thay vào đó, nó chờ đợi mãi mãi cho các thông báo mới. Bạn sẽ cần tắt nó sử dụng <kbd>ctrl</kbd>-<kbd>C</kbd>.

#### Mã Trong Một Async Block Thực Hiện Tuyến Tính

Hãy bắt đầu bằng cách kiểm tra tại sao các thông báo đến tất cả cùng lúc sau toàn bộ delay, thay vì đến với delays giữa mỗi cái. Trong một async block nhất định, thứ tự mà các từ khóa `await` xuất hiện trong mã cũng là thứ tự trong đó chúng được thực hiện khi chương trình chạy.

Có chỉ một async block trong Listing 17-10, vì vậy mọi thứ trong nó chạy tuyến tính. Vẫn còn không có concurrency. Tất cả các lệnh gọi `tx.send` xảy ra, xen kẽ với tất cả các lệnh gọi `trpl::sleep` và các await points liên kết của chúng. Chỉ sau đó `while let` loop get để đi qua bất kỳ await points nào trên các lệnh gọi `recv`.

Để có được hành vi chúng ta muốn, nơi delay sleep xảy ra giữa mỗi thông báo, chúng ta cần đặt các hoạt động `tx` và `rx` của riêng chúng trong async blocks, như thể hiện trong Listing 17-11. Sau đó runtime có thể thực hiện mỗi cái riêng biệt sử dụng `trpl::join`, giống như trong Listing 17-8. Một lần nữa, chúng ta await kết quả của lệnh gọi `trpl::join`, không phải individual futures. Nếu chúng ta đã await individual futures một cách tuần tự, chúng ta sẽ chỉ kết thúc quay lại trong một tuyến tính flow—chính xác những gì chúng ta đang cố gắng _không_ làm.

<!-- We cannot test this one because it never stops! -->

<Listing number="17-11" caption="Tách `send` và `recv` vào async blocks của riêng chúng và await futures cho những blocks đó" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-11/src/main.rs:futures}}
```

</Listing>

Với mã cập nhật trong Listing 17-11, các thông báo được in tại các khoảng thời gian 500-millisecond, thay vì tất cả trong một sự vội vã sau 2 giây.

#### Di Chuyển Ownership Vào Một Async Block

Chương trình vẫn không bao giờ thoát, tuy nhiên, vì cách `while let` loop tương tác với `trpl::join`:

- Future được trả về từ `trpl::join` hoàn thành chỉ một lần _cả hai_ futures được truyền vào hoàn thành.
- Tương lai `tx_fut` hoàn thành khi nó kết thúc ngủ sau khi gửi thông báo cuối cùng trong `vals`.
- Tương lai `rx_fut` sẽ không hoàn thành cho đến khi `while let` loop kết thúc.
- `while let` loop sẽ không kết thúc cho đến khi awaiting `rx.recv` tạo ra `None`.
- Awaiting `rx.recv` sẽ return `None` chỉ một khi đầu bên kia của channel được đóng.
- Channel sẽ đóng chỉ nếu chúng ta gọi `rx.close` hoặc khi bên gửi, `tx`, bị drop.
- Chúng ta không gọi `rx.close` bất cứ nơi nào, và `tx` sẽ không được drop cho đến khi outermost async block được truyền vào `trpl::block_on` kết thúc.
- Block không thể kết thúc vì nó được chặn trên `trpl::join` hoàn thành, mà đưa chúng ta trở lại đầu danh sách này.

Ngay bây giờ, async block nơi chúng ta gửi các thông báo chỉ _borrows_ `tx` vì gửi một thông báo không yêu cầu ownership, nhưng nếu chúng ta có thể _di chuyển_ `tx` vào async block đó, nó sẽ được drop khi block đó kết thúc. Trong phần ["Capturing References or Moving Ownership"][capture-or-move]<!-- ignore --> ở Chương 13, bạn đã học cách sử dụng từ khóa `move` với closures, và, như thảo luận trong phần ["Using `move` Closures with Threads"][move-threads]<!-- ignore --> ở Chương 16, chúng ta thường cần di chuyển dữ liệu vào closures khi làm việc với threads. Các động lực cơ bản tương tự áp dụng cho async blocks, vì vậy từ khóa `move` hoạt động với async blocks giống như nó hoạt động với closures.

Trong Listing 17-12, chúng ta thay đổi block được sử dụng để gửi các thông báo từ `async` thành `async move`.

<Listing number="17-12" caption="Một sửa đổi của mã từ Listing 17-11 mà correctly tắt khi hoàn thành" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-12/src/main.rs:with-move}}
```

</Listing>

Khi chúng ta chạy _this_ phiên bản của mã, nó tắt gracefully sau khi thông báo cuối cùng được gửi và nhận. Tiếp theo, hãy xem những gì sẽ cần thay đổi để gửi dữ liệu từ hơn một future.

#### Joining Một Số Futures Với `join!` Macro

Async channel này cũng là một multiple-producer channel, vì vậy chúng ta có thể gọi `clone` trên `tx` nếu chúng ta muốn gửi các thông báo từ nhiều futures, như thể hiện trong Listing 17-13.

<Listing number="17-13" caption="Sử dụng multiple producers với async blocks" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-13/src/main.rs:here}}
```

</Listing>

Đầu tiên, chúng ta clone `tx`, tạo `tx1` bên ngoài async block đầu tiên. Chúng ta di chuyển `tx1` vào block đó giống như chúng ta đã làm trước với `tx`. Sau đó, sau, chúng ta di chuyển `tx` ban đầu vào một _new_ async block, nơi chúng ta gửi thêm các thông báo trên một delay hơi chậm. Chúng ta xảy ra để đặt async block mới này sau async block để nhận các thông báo, nhưng nó có thể đi trước nó giống nhau. Điều chính là thứ tự mà futures được awaited, không phải thứ tự trong đó chúng được tạo.

Cả hai async blocks để gửi các thông báo cần phải là `async move` blocks để cả `tx` và `tx1` được drop khi những blocks đó kết thúc. Nếu không, chúng ta sẽ kết thúc quay lại trong infinite loop tương tự mà chúng ta bắt đầu trong.

Cuối cùng, chúng ta chuyển từ `trpl::join` đến `trpl::join!` để xử lý future bổ sung: `join!` macro awaits một số arbitrary futures nơi chúng ta biết số futures tại compile time. Chúng ta sẽ thảo luận về awaiting một collection của một unknown số futures sau trong chương này.

Bây giờ chúng ta thấy tất cả các thông báo từ cả hai gửi futures, và vì gửi futures sử dụng slightly delays khác nhau sau gửi, các thông báo cũng được nhận tại những khoảng thời gian khác nhau:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
received 'hi'
received 'more'
received 'from'
received 'the'
received 'messages'
received 'future'
received 'for'
received 'you'
```

Chúng ta đã khám phá cách sử dụng message passing để gửi dữ liệu giữa futures, cách mã trong một async block chạy tuần tự, cách di chuyển ownership vào một async block, và cách join nhiều futures. Tiếp theo, hãy thảo luận về cách và tại sao để cho runtime biết nó có thể chuyển sang task khác.

[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[join-handles]: ch16-01-threads.html#waiting-for-all-threads-to-finish
[message-passing-threads]: ch16-02-message-passing.html
[if-let]: ch06-03-if-let.html
[capture-or-move]: ch13-01-closures.html#capturing-references-or-moving-ownership
[move-threads]: ch16-01-threads.html#using-move-closures-with-threads
