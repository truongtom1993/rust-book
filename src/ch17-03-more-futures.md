
<!-- Old headings. Do not remove or links may break. -->

<a id="yielding"></a>

### Nhượng Quyền Kiểm Soát Cho Runtime

Nhớ lại từ phần ["Our First Async Program"][async-program]<!-- ignore --> rằng tại mỗi await point, Rust cung cấp cho runtime một cơ hội để tạm dừng task và chuyển sang cái khác nếu future được awaited không sẵn sàng. Ngược lại cũng đúng: Rust _only_ tạm dừng async blocks và chuyển điều khiển trở lại runtime tại một await point. Mọi thứ giữa await points là đồng bộ.

Điều đó có nghĩa là nếu bạn làm rất nhiều công việc trong một async block mà không có một await point, future đó sẽ chặn bất kỳ futures khác nào từ việc tạo progress. Bạn có thể đôi khi nghe điều này được gọi là một future _starving_ futures khác. Trong một số trường hợp, điều đó có thể không phải là một big deal. Tuy nhiên, nếu bạn đang làm một loại expensive setup hoặc long-running work, hoặc nếu bạn có một future sẽ tiếp tục làm một số task cụ thể vô định, bạn sẽ cần suy nghĩ về khi nào và nơi nào để tay kiểm soát trở lại runtime.

Hãy mô phỏng một long-running operation để minh họa vấn đề starvation, sau đó khám phá cách để giải quyết nó. Listing 17-14 giới thiệu một function `slow`.

<Listing number="17-14" caption="Sử dụng `thread::sleep` để mô phỏng slow operations" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-14/src/main.rs:slow}}
```

</Listing>

Mã này sử dụng `std::thread::sleep` thay vì `trpl::sleep` vì vậy gọi `slow` sẽ chặn thread hiện tại cho một số milliseconds. Chúng ta có thể sử dụng `slow` để đứng cho các real-world operations mà vừa long-running vừa blocking.

Trong Listing 17-15, chúng ta sử dụng `slow` để giả lập làm loại CPU-bound work này trong một cặp futures.

<Listing number="17-15" caption="Gọi function `slow` để mô phỏng slow operations" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-15/src/main.rs:slow-futures}}
```

</Listing>

Mỗi future chuyển điều khiển trở lại runtime chỉ _sau_ thực hiện một bunch của slow operations. Nếu bạn chạy mã này, bạn sẽ thấy output này:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-15/
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'a' ran for 10ms
'a' ran for 20ms
'b' started.
'b' ran for 75ms
'b' ran for 10ms
'b' ran for 15ms
'b' ran for 350ms
'a' finished.
```

Như với Listing 17-5 nơi chúng ta sử dụng `trpl::select` để race futures fetching hai URLs, `select` vẫn kết thúc ngay khi `a` xong. Không có interleaving giữa các lệnh gọi `slow` trong hai futures, tuy nhiên. Tương lai `a` làm tất cả công việc của nó cho đến khi lệnh gọi `trpl::sleep` được awaited, sau đó tương lai `b` làm tất cả công việc của nó cho đến khi lệnh gọi `trpl::sleep` của riêng nó được awaited, và cuối cùng tương lai `a` hoàn thành. Để cho phép cả hai futures tạo progress giữa các slow tasks của chúng, chúng ta cần await points vì vậy chúng ta có thể tay kiểm soát trở lại runtime. Điều đó có nghĩa là chúng ta cần một cái gì đó chúng ta có thể await!

Chúng ta đã có thể thấy loại handoff này xảy ra trong Listing 17-15: nếu chúng ta loại bỏ `trpl::sleep` tại cuối tương lai `a`, nó sẽ hoàn thành mà không có tương lai `b` chạy _at all_. Hãy thử sử dụng function `trpl::sleep` như một điểm bắt đầu cho phép operations chuyển off making progress, như thể hiện trong Listing 17-16.

<Listing number="17-16" caption="Sử dụng `trpl::sleep` để cho phép operations chuyển off making progress" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-16/src/main.rs:here}}
```

</Listing>

Chúng ta đã thêm `trpl::sleep` calls với await points giữa mỗi lệnh gọi `slow`. Bây giờ công việc của hai futures được interleaved:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-16
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'b' started.
'b' ran for 75ms
'a' ran for 10ms
'b' ran for 10ms
'a' ran for 20ms
'b' ran for 15ms
'a' finished.
```

Tương lai `a` vẫn chạy cho một bit trước khi tay kiểm soát sang `b`, vì nó gọi `slow` trước khi bao giờ gọi `trpl::sleep`, nhưng sau đó futures swap quay lại và quay lại mỗi lần một trong số chúng hit một await point. Trong trường hợp này, chúng ta đã làm điều đó sau mỗi lệnh gọi `slow`, nhưng chúng ta có thể phá vỡ công việc theo cách bất kỳ có ý nghĩa cho chúng ta.

Chúng ta thực sự không muốn _sleep_ ở đây, tuy nhiên: chúng ta muốn tạo progress như nhanh như chúng ta có thể. Chúng ta chỉ cần tay kiểm soát trở lại runtime. Chúng ta có thể làm điều đó trực tiếp, sử dụng function `trpl::yield_now`. Trong Listing 17-17, chúng ta thay thế tất cả những `trpl::sleep` calls với `trpl::yield_now`.

<Listing number="17-17" caption="Sử dụng `yield_now` để cho phép operations chuyển off making progress" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-17/src/main.rs:yields}}
```

</Listing>

Mã này vừa rõ ràng hơn về actual intent và có thể significantly nhanh hơn so với sử dụng `sleep`, vì timers như cái được sử dụng bởi `sleep` thường có limits trên cách granular chúng có thể. Phiên bản của `sleep` chúng ta đang sử dụng, ví dụ, sẽ luôn ngủ cho ít nhất một millisecond, thậm chí nếu chúng ta truyền nó một `Duration` của một nanosecond. Lại nữa, modern computers là _fast_: chúng có thể làm rất nhiều trong một millisecond!

Điều này có nghĩa là async có thể hữu ích thậm chí cho compute-bound tasks, tùy thuộc vào những gì khác chương trình của bạn đang làm, vì nó cung cấp một useful tool cho việc cấu trúc các mối quan hệ giữa các phần khác nhau của chương trình (nhưng với một cost của overhead của async state machine). Đây là một hình thức _cooperative multitasking_, nơi mỗi future có quyền để xác định khi nó tay kiểm soát qua await points. Mỗi future do đó cũng có responsibility để tránh blocking cho quá lâu. Trong một số Rust-based embedded operating systems, đây là _only_ loại multitasking!

Trong real-world code, bạn sẽ không thường xuyên xen kẽ function calls với await points trên mỗi line, tất nhiên. Trong khi nhượng quyền kiểm soát theo cách này là relatively inexpensive, nó không phải là free. Trong nhiều trường hợp, cố gắng phá vỡ một compute-bound task có thể làm cho nó significantly chậm hơn, vì vậy đôi khi nó tốt hơn cho _overall_ performance để cho một operation block briefly. Luôn luôn measure để xem performance bottlenecks thực tế của mã của bạn là gì. Underlying dynamic vẫn quan trọng để có trong tâm trí, tuy nhiên, nếu bạn _are_ thấy rất nhiều công việc xảy ra trong serial mà bạn dự kiến sẽ xảy ra đồng thời!

### Xây Dựng Các Async Abstractions Của Chúng Ta Riêng

Chúng ta cũng có thể soạn futures với nhau để tạo các patterns mới. Ví dụ, chúng ta có thể xây dựng một function `timeout` với async building blocks chúng ta đã có. Khi chúng ta xong, kết quả sẽ là một building block khác chúng ta có thể sử dụng để tạo vẫn nhiều async abstractions hơn.

Listing 17-18 hiển thị cách chúng ta sẽ expect `timeout` này để hoạt động với một slow future.

<Listing number="17-18" caption="Sử dụng imagine `timeout` của chúng ta để chạy một slow operation với một time limit" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-18/src/main.rs:here}}
```

</Listing>

Hãy triển khai điều này! Để bắt đầu, hãy suy nghĩ về API cho `timeout`:

- Nó cần phải là một async function chính nó để chúng ta có thể await nó.
- Tham số đầu tiên của nó sẽ là một future để chạy. Chúng ta có thể tạo nó generic để cho phép nó hoạt động với bất kỳ future nào.
- Tham số thứ hai của nó sẽ là maximum time để chờ. Nếu chúng ta sử dụng một `Duration`, sẽ làm cho nó dễ dàng để truyền dọc theo `trpl::sleep`.
- Nó sẽ trả về một `Result`. Nếu future hoàn thành thành công, `Result` sẽ là `Ok` với giá trị được tạo ra bởi future. Nếu timeout elapses đầu tiên, `Result` sẽ là `Err` với duration mà timeout chờ đợi cho.

Listing 17-19 hiển thị declaration này.

<!-- This is not tested because it intentionally does not compile. -->

<Listing number="17-19" caption="Định Nghĩa Signature Của `timeout`" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-19/src/main.rs:declaration}}
```

</Listing>

Đó là thỏa mãn các mục tiêu của chúng ta cho các loại. Bây giờ hãy suy nghĩ về _behavior_ chúng ta cần: chúng ta muốn race future được truyền vào chống lại duration. Chúng ta có thể sử dụng `trpl::sleep` để tạo một timer future từ duration, và sử dụng `trpl::select` để chạy timer đó với future người gọi truyền vào.

Trong Listing 17-20, chúng ta triển khai `timeout` bằng cách matching trên kết quả của awaiting `trpl::select`.

<Listing number="17-20" caption="Định Nghĩa `timeout` Với `select` Và `sleep`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-20/src/main.rs:implementation}}
```

</Listing>

Triển khai của `trpl::select` không phải fair: nó luôn polls arguments trong thứ tự trong đó chúng được truyền (các `select` triển khai khác sẽ randomly chọn argument nào để poll đầu tiên). Do đó, chúng ta truyền `future_to_try` đến `select` đầu tiên vì vậy nó nhận được một cơ hội để hoàn thành ngay cả khi `max_time` là một very short duration. Nếu `future_to_try` kết thúc đầu tiên, `select` sẽ return `Left` với output từ `future_to_try`. Nếu `timer` kết thúc đầu tiên, `select` sẽ return `Right` với output của timer là `()`.

Nếu `future_to_try` thành công và chúng ta nhận được `Left(output)`, chúng ta return `Ok(output)`. Nếu sleep timer elapses thay vào đó và chúng ta nhận được `Right(())`, chúng ta ignore `()` với `_` và return `Err(max_time)` thay vào đó.

Với điều đó, chúng ta có một working `timeout` xây dựng từ hai async helpers khác. Nếu chúng ta chạy mã của chúng ta, nó sẽ print failure mode sau timeout:

```text
Failed after 2 seconds
```

Bởi vì futures soạn với các futures khác, bạn có thể xây dựng thực sự powerful tools sử dụng smaller async building blocks. Ví dụ, bạn có thể sử dụng cùng approach để kết hợp timeouts với retries, và turn sử dụng những với operations như network calls (như những trong Listing 17-5).

Trong thực tế, bạn sẽ thường xuyên làm việc trực tiếp với `async` và `await`, và secondarily với functions như `select` và macros như `join!` macro để kiểm soát cách futures outermost được thực hiện.

Chúng ta đã bây giờ thấy một số cách để làm việc với nhiều futures tại cùng một thời gian. Tiếp theo, chúng ta sẽ xem làm cách nào chúng ta có thể làm việc với nhiều futures trong một sequence qua thời gian với _streams_.

[async-program]: ch17-01-futures-and-syntax.html#our-first-async-program
