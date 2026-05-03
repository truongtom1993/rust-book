## Tắt Graceful và Dọn Dẹp

Mã trong Listing 21-20 đang phản hồi các yêu cầu không đồng bộ thông qua việc sử dụng một thread pool, như chúng ta dự định. Chúng ta nhận được một số cảnh báo về các trường `workers`, `id`, và `thread` mà chúng ta không sử dụng theo cách trực tiếp giúp chúng ta nhớ rằng chúng ta không dọn dẹp bất cứ điều gì. Khi chúng ta sử dụng phương pháp ít thanh lịch hơn <kbd>ctrl</kbd>-<kbd>C</kbd> để dừng luồng chính, tất cả các luồng khác bị dừng ngay lập tức cũng vậy, ngay cả khi chúng đang ở giữa phục vụ một yêu cầu.

Tiếp theo, chúng ta sẽ triển khai trait `Drop` để gọi `join` trên mỗi luồng trong nhóm để chúng có thể kết thúc các yêu cầu mà chúng đang làm việc trước khi đóng. Sau đó, chúng ta sẽ triển khai một cách để nói với các luồng mà chúng nên ngừng chấp nhận yêu cầu mới và tắt. Để xem mã này hoạt động, chúng ta sẽ sửa đổi server của chúng ta để chấp nhận chỉ hai yêu cầu trước khi gracefully tắt thread pool của nó.

Một điều cần lưu ý khi chúng ta đi: Không có phần nào của mã xử lý thực thi các bao đóng bị ảnh hưởng bởi các phần này, vì vậy mọi thứ ở đây sẽ giống nhau nếu chúng ta đang sử dụng một thread pool cho một async runtime.

### Triển Khai Trait `Drop` trên `ThreadPool`

Hãy bắt đầu với việc triển khai `Drop` trên thread pool của chúng ta. Khi nhóm được drop, các luồng của chúng ta đều nên join để chắc chắn rằng chúng kết thúc công việc của chúng. Listing 21-22 cho thấy một nỗ lực đầu tiên ở một triển khai `Drop`; mã này sẽ chưa hoạt động hoàn toàn.

<Listing number="21-22" file-name="src/lib.rs" caption="Joining mỗi luồng khi thread pool vượt ra ngoài phạm vi">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-22/src/lib.rs:here}}
```

</Listing>

Đầu tiên, chúng ta lặp qua mỗi `workers` của thread pool. Chúng ta sử dụng `&mut` cho điều này vì `self` là một tham chiếu có thể biến đổi, và chúng ta cũng cần có thể biến đổi `worker`. Đối với mỗi `worker`, chúng ta in một thông báo nói rằng phiên bản `Worker` cụ thể này đang tắt, và sau đó chúng ta gọi `join` trên luồng của phiên bản `Worker` đó. Nếu cuộc gọi `join` thất bại, chúng ta sử dụng `unwrap` để làm Rust panic và đi vào một tắt không graceful.

Đây là lỗi chúng ta nhận được khi chúng ta biên dịch mã này:

```console
{{#include ../listings/ch21-web-server/listing-21-22/output.txt}}
```

Lỗi cho chúng ta biết chúng ta không thể gọi `join` vì chúng ta chỉ có một mượn có thể biến đổi của mỗi `worker` và `join` nhận quyền sở hữu của đối số của nó. Để giải quyết vấn đề này, chúng ta cần chuyển luồng ra khỏi phiên bản `Worker` sở hữu `thread` để `join` có thể tiêu thụ luồng. Một cách để làm điều này là thực hiện cùng một cách tiếp cận mà chúng ta đã thực hiện trong Listing 18-15. Nếu `Worker` giữ một `Option<thread::JoinHandle<()>>`, chúng ta có thể gọi phương thức `take` trên `Option` để chuyển giá trị ra khỏi biến thể `Some` và để lại một biến thể `None` tại chỗ của nó. Nói cách khác, một `Worker` đang chạy sẽ có một biến thể `Some` trong `thread`, và khi chúng ta muốn dọn dẹp một `Worker`, chúng ta sẽ thay thế `Some` bằng `None` để `Worker` sẽ không có một luồng để chạy.

Tuy nhiên, _chỉ_ lần này sẽ xuất hiện sẽ là khi dropping `Worker`. Để đổi lấy, chúng ta sẽ phải đối phó với một `Option<thread::JoinHandle<()>>` ở bất cứ nơi nào chúng ta truy cập `worker.thread`. Rust idiom sử dụng `Option` khá nhiều, nhưng khi bạn thấy mình bao bọc thứ gì đó bạn biết sẽ luôn có trong một `Option` như một giải pháp thay thế như thế này, nó là một ý tưởng tốt để tìm kiếm các cách tiếp cận thay thế để làm cho mã của bạn sạch hơn và ít lỗi hơn.

Trong trường hợp này, một thay thế tốt hơn tồn tại: Phương thức `Vec::drain`. Nó chấp nhận một tham số phạm vi để chỉ định mục nào sẽ bị xóa khỏi vector và trả về một iterator của những mục đó. Truyền cú pháp phạm vi `..` sẽ xóa *mọi* giá trị từ vector.

Vì vậy, chúng ta cần cập nhật triển khai `drop` của `ThreadPool` như thế này:

<Listing file-name="src/lib.rs">

```rust
{{#rustdoc_include ../listings/ch21-web-server/no-listing-04-update-drop-definition/src/lib.rs:here}}
```

</Listing>

Điều này giải quyết lỗi trình biên dịch và không yêu cầu bất kỳ thay đổi nào khác đối với mã của chúng ta. Lưu ý rằng, bởi vì drop có thể được gọi khi panic, unwrap cũng có thể panic và gây ra một panic kép, điều này ngay lập tức làm sập chương trình và kết thúc bất kỳ dọn dẹp nào đang tiến hành. Điều này là tốt cho một chương trình ví dụ, nhưng nó không được khuyến khích cho mã sản xuất.

### Báo Hiệu cho Các Luồng để Ngừng Lắng Nghe các Công Việc

Với tất cả các thay đổi chúng ta đã thực hiện, mã của chúng ta biên dịch mà không có bất kỳ cảnh báo nào. Tuy nhiên, tin xấu là mã này không hoạt động theo cách mà chúng ta muốn nó chưa. Chìa khóa là logic trong các bao đóng chạy bởi các luồng của các phiên bản `Worker`: Tại thời điểm này, chúng ta gọi `join`, nhưng điều đó sẽ không tắt các luồng, bởi vì chúng `loop` mãi mãi tìm kiếm công việc. Nếu chúng ta cố gắng drop `ThreadPool` của chúng ta với triển khai `drop` hiện tại của chúng ta, luồng chính sẽ chặn mãi mãi, chờ đợi luồng đầu tiên kết thúc.

Để khắc phục vấn đề này, chúng ta sẽ cần một thay đổi trong triển khai `drop` của `ThreadPool` và sau đó một thay đổi trong vòng lặp `Worker`.

Đầu tiên, chúng ta sẽ thay đổi triển khai `drop` của `ThreadPool` để rõ ràng drop `sender` trước khi chờ các luồng kết thúc. Listing 21-23 cho thấy các thay đổi để `ThreadPool` để rõ ràng drop `sender`. Không giống như với luồng, ở đây chúng ta _do_ cần sử dụng một `Option` để có thể chuyển `sender` ra khỏi `ThreadPool` bằng `Option::take`.

<Listing number="21-23" file-name="src/lib.rs" caption="Rõ ràng dropping `sender` trước khi joining các luồng `Worker`">

```rust,noplayground,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-23/src/lib.rs:here}}
```

</Listing>

Dropping `sender` đóng channel, chỉ ra rằng không có tin nhắn nào khác sẽ được gửi. Khi điều đó xảy ra, tất cả các cuộc gọi `recv` mà các phiên bản `Worker` thực hiện trong vòng lặp vô hạn sẽ trả về một lỗi. Trong Listing 21-24, chúng ta thay đổi vòng lặp `Worker` để gracefully thoát khỏi vòng lặp trong trường hợp đó, có nghĩa là các luồng sẽ kết thúc khi triển khai `drop` của `ThreadPool` gọi `join` trên chúng.

<Listing number="21-24" file-name="src/lib.rs" caption="Rõ ràng thoát khỏi vòng lặp khi `recv` trả về một lỗi">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-24/src/lib.rs:here}}
```

</Listing>

Để xem mã này hoạt động, hãy sửa đổi `main` để chấp nhận chỉ hai yêu cầu trước khi gracefully tắt server, như được hiển thị trong Listing 21-25.

<Listing number="21-25" file-name="src/main.rs" caption="Tắt server sau khi phục vụ hai yêu cầu bằng cách thoát khỏi vòng lặp">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/listing-21-25/src/main.rs:here}}
```

</Listing>

Bạn sẽ không muốn một web server thế giới thực tế để tắt sau khi phục vụ chỉ hai yêu cầu. Mã này chỉ chứng minh rằng graceful shutdown và cleanup đang hoạt động.

Phương thức `take` được định nghĩa trong trait `Iterator` và giới hạn sự lặp lại đến hai mục tối đa. `ThreadPool` sẽ vượt ra ngoài phạm vi ở cuối `main`, và triển khai `drop` sẽ chạy.

Bắt đầu server với `cargo run` và tạo ba yêu cầu. Yêu cầu thứ ba sẽ lỗi, và trong terminal của bạn, bạn sẽ thấy đầu ra tương tự như thế này:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-25
cargo run
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
third request will error because server will have shut down
copy output below
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.41s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

Bạn có thể thấy một thứ tự khác nhau của các `Worker` ID và tin nhắn được in. Chúng ta có thể thấy cách mã này hoạt động từ các tin nhắn: các phiên bản `Worker` 0 và 3 nhận được hai yêu cầu đầu tiên. Server dừng chấp nhận các kết nối sau kết nối thứ hai, và triển khai `Drop` trên `ThreadPool` bắt đầu thực thi trước khi `Worker` 3 thậm chí bắt đầu công việc của nó. Dropping `sender` ngắt tất cả các phiên bản `Worker` và nói với chúng để tắt. Các phiên bản `Worker` mỗi in một thông báo khi chúng ngắt, và sau đó thread pool gọi `join` để chờ mỗi luồng `Worker` kết thúc.

Chú ý một khía cạnh thú vị của việc thực thi cụ thể này: `ThreadPool` đã drop `sender`, và trước khi bất kỳ `Worker` nào nhận được một lỗi, chúng ta đã cố gắng join `Worker` 0. `Worker` 0 chưa nhận được một lỗi từ `recv`, vì vậy luồng chính bị chặn, chờ `Worker` 0 kết thúc. Trong khi đó, `Worker` 3 nhận được một công việc và sau đó tất cả các luồng nhận được một lỗi. Khi `Worker` 0 hoàn thành, luồng chính chờ các phiên bản `Worker` còn lại kết thúc. Tại thời điểm đó, chúng đã thoát khỏi các vòng lặp của chúng và dừng lại.

Chúc mừng! Chúng ta bây giờ đã hoàn thành dự án của chúng ta; chúng ta có một web server cơ bản sử dụng một thread pool để phản hồi không đồng bộ. Chúng ta có thể thực hiện một graceful shutdown của server, có thể dọn dẹp tất cả các luồng trong nhóm.

Đây là mã đầy đủ cho các tài liệu tham khảo:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/main.rs}}
```

</Listing>

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/lib.rs}}
```

</Listing>

Chúng ta có thể làm nhiều hơn ở đây! Nếu bạn muốn tiếp tục nâng cao dự án này, dưới đây là một số ý tưởng:

- Thêm thêm tài liệu vào `ThreadPool` và các phương thức công khai của nó.
- Thêm các bài kiểm tra của chức năng thư viện.
- Thay đổi các cuộc gọi đến `unwrap` để xử lý lỗi mạnh mẽ hơn.
- Sử dụng `ThreadPool` để thực hiện một số nhiệm vụ khác ngoài phục vụ các yêu cầu web.
- Tìm một crate thread pool trên [crates.io](https://crates.io/) và triển khai một web server tương tự bằng cách sử dụng crate thay vì. Sau đó, so sánh API và độ bền của nó với thread pool chúng ta đã triển khai.

## Tóm Lược

Làm tốt! Bạn đã đạt tới cuối sách! Chúng tôi muốn cảm ơn bạn vì đã tham gia cùng chúng tôi trong chuyến tham quan Rust này. Bây giờ bạn đã sẵn sàng để triển khai các dự án Rust của riêng bạn và giúp đỡ các dự án của những người khác. Hãy nhớ rằng có một cộng đồng Rustaceans chào đón những người sẽ yêu thích giúp bạn với bất kỳ thách thức nào bạn gặp phải trên hành trình Rust của bạn.
