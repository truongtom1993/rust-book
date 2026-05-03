<!-- Old headings. Do not remove or links may break. -->

<a id="using-message-passing-to-transfer-data-between-threads"></a>

## Truyền Dữ Liệu Giữa Các Thread Bằng Message Passing

Một phương pháp ngày càng phổ biến để đảm bảo tính đồng thời an toàn là message
passing, nơi các thread hoặc actor giao tiếp với nhau bằng cách gửi các tin nhắn
chứa dữ liệu. Dưới đây là ý tưởng này dưới dạng một câu nói từ [tài liệu ngôn ngữ Go](https://golang.org/doc/effective_go.html#concurrency):
“Không giao tiếp bằng cách chia sẻ bộ nhớ; thay vào đó, hãy chia sẻ bộ nhớ bằng cách giao tiếp.”

Để thực hiện tính đồng thời gửi tin nhắn, thư viện chuẩn của Rust cung cấp một
triển khai của các channel. _Channel_ là một khái niệm lập trình chung theo
đó dữ liệu được gửi từ một thread đến thread khác.

Bạn có thể tưởng tượng một channel trong lập trình giống như một kênh nước có
hướng, chẳng hạn như một suối hoặc một dòng sông. Nếu bạn đặt một cái gì đó
như một chú vịt cao su vào sông, nó sẽ chảy xuống dòng đến cuối cùng của
đường thủy.

Một channel có hai phần: một transmitter và một receiver. Nửa transmitter là
vị trí phía thượng nguồn nơi bạn đặt chú vịt cao su vào sông, và nửa receiver
là nơi chú vịt cao su kết thúc hạ lưu. Một phần của mã của bạn gọi các phương
thức trên transmitter với dữ liệu bạn muốn gửi, và một phần khác kiểm tra đầu
cuối nhận để tìm các tin nhắn đến. Một channel được gọi là _closed_ nếu một
trong hai nửa transmitter hoặc receiver bị drop.

Ở đây, chúng ta sẽ xây dựng một chương trình có một thread để tạo ra các giá
trị và gửi chúng xuống một channel, và một thread khác sẽ nhận các giá trị đó
và in chúng ra. Chúng ta sẽ gửi các giá trị đơn giản giữa các thread bằng
cách sử dụng channel để minh họa tính năng. Khi bạn quen thuộc với kỹ thuật,
bạn có thể sử dụng channel cho bất kỳ thread nào cần giao tiếp với nhau, chẳng
hạn như một hệ thống trò chuyện hoặc một hệ thống nơi nhiều thread thực hiện
các phần của một tính toán và gửi các phần đó đến một thread tập hợp các kết
quả.

Đầu tiên, trong Listing 16-6, chúng ta sẽ tạo một channel nhưng không làm gì
với nó. Lưu ý rằng điều này sẽ không biên dịch vì Rust không thể xác định
loại giá trị nào chúng ta muốn gửi qua channel.

<Listing number="16-6" file-name="src/main.rs" caption="Tạo một channel và gán hai phần cho `tx` và `rx`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-06/src/main.rs}}
```

</Listing>

Chúng ta tạo một channel mới bằng cách sử dụng hàm `mpsc::channel`; `mpsc` là
viết tắt của _multiple producer, single consumer_. Nói cách khác, cách thư viện
chuẩn Rust triển khai channel có nghĩa là một channel có thể có nhiều đầu
_sending_ tạo ra các giá trị nhưng chỉ một đầu _receiving_ tiêu thụ những giá
trị đó. Hãy tưởng tượng nhiều dòng chảy cùng nhau thành một dòng sông lớn:
Mọi thứ được gửi xuống bất kỳ dòng nào cũng sẽ kết thúc ở một dòng sông cuối
cùng. Chúng ta sẽ bắt đầu với một single producer bây giờ, nhưng chúng ta sẽ
thêm nhiều producer khi làm cho ví dụ này hoạt động.

Hàm `mpsc::channel` trả về một tuple, phần tử đầu tiên là đầu gửi—transmitter
—và phần tử thứ hai là đầu nhận—receiver. Các chữ viết tắt `tx` và `rx` được
sử dụng theo truyền thống trong nhiều lĩnh vực cho _transmitter_ và _receiver_,
tương ứng, vì vậy chúng ta đặt tên các biến như vậy để chỉ ra từng đầu. Chúng
ta đang sử dụng một câu lệnh `let` với một mẫu phân tách tuple; chúng ta sẽ
thảo luận về việc sử dụng các mẫu trong các câu lệnh `let` và phân tách trong
Chương 19. Bây giờ, hãy biết rằng sử dụng một câu lệnh `let` theo cách này là
một cách tiện lợi để trích xuất các phần của tuple được trả về bởi `mpsc::channel`.

Hãy di chuyển đầu truyền vào một thread được tạo ra và có nó gửi một
chuỗi sao cho thread được tạo ra đang giao tiếp với thread chính, như
được hiển thị trong Listing 16-7. Đây giống như việc đặt một chú vịt cao
su vào sông phía thượng nguồn hoặc gửi một tin nhắn trò chuyện từ một
thread này sang thread khác.

<Listing number="16-7" file-name="src/main.rs" caption="Di chuyển `tx` đến một thread được tạo ra và gửi `&quot;hi&quot;`">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-07/src/main.rs}}
```

</Listing>

Một lần nữa, chúng ta đang sử dụng `thread::spawn` để tạo một thread mới
và sau đó sử dụng `move` để di chuyển `tx` vào closure sao cho thread
được tạo ra sở hữu `tx`. Thread được tạo ra cần sở hữu transmitter để
có thể gửi tin nhắn qua channel.

Transmitter có một phương thức `send` lấy giá trị chúng ta muốn gửi.
Phương thức `send` trả về một loại `Result<T, E>`, vì vậy nếu receiver
đã được drop và không có nơi nào để gửi giá trị, thao tác send sẽ
trả về một lỗi. Trong ví dụ này, chúng ta gọi `unwrap` để panic trong
trường hợp có lỗi. Nhưng trong một ứng dụng thực tế, chúng ta sẽ xử lý
nó đúng cách: Quay lại Chương 9 để xem lại các chiến lược xử lý lỗi
thích hợp.

Trong Listing 16-8, chúng ta sẽ lấy giá trị từ receiver trong thread chính.
Đây giống như việc lấy chú vịt cao su từ nước ở cuối sông hoặc nhận một
tin nhắn trò chuyện.

<Listing number="16-8" file-name="src/main.rs" caption="Nhận giá trị `&quot;hi&quot;` trong thread chính và in nó ra">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-08/src/main.rs}}
```

</Listing>

Receiver có hai phương thức hữu ích: `recv` và `try_recv`. Chúng ta đang
sử dụng `recv`, viết tắt của _receive_, sẽ block việc thực hiện của thread
chính và chờ cho đến khi một giá trị được gửi xuống channel. Khi một giá
trị được gửi, `recv` sẽ trả về nó trong một `Result<T, E>`. Khi transmitter
đóng, `recv` sẽ trả về một lỗi để báo hiệu rằng sẽ không có giá trị nào
khác sắp tới.

Phương thức `try_recv` không block, nhưng thay vào đó sẽ trả về một
`Result<T, E>` ngay lập tức: một giá trị `Ok` giữ một tin nhắn nếu một
tin nhắn có sẵn và một giá trị `Err` nếu không có tin nhắn nào lúc này.
Sử dụng `try_recv` rất hữu ích nếu thread này có công việc khác để làm
trong khi chờ tin nhắn: Chúng ta có thể viết một vòng lặp gọi `try_recv`
từng lúc, xử lý một tin nhắn nếu có sẵn, ngoài ra làm công việc khác
trong một lúc cho đến khi kiểm tra lại.

Chúng ta đã sử dụng `recv` trong ví dụ này để đơn giản; chúng ta không
có công việc nào khác cho thread chính để làm ngoài việc chờ tin nhắn,
vì vậy block thread chính là thích hợp.

Khi chúng ta chạy mã trong Listing 16-8, chúng ta sẽ thấy giá trị được in
từ thread chính:

<!-- Not extracting output because changes to this output aren’t significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
```

Hoàn hảo!

<!-- Old headings. Do not remove or links may break. -->

<a id="channels-and-ownership-transference"></a>

### Truyền Ownership Qua Các Channel

Các quy tắc ownership đóng một vai trò quan trọng trong việc gửi tin nhắn
vì chúng giúp bạn viết mã đồng thời an toàn. Ngăn chặn các lỗi trong lập
trình đồng thời là lợi thế của việc suy nghĩ về ownership trong các chương
trình Rust của bạn. Hãy làm một thí nghiệm để chỉ ra cách channel và ownership
hoạt động cùng nhau để ngăn chặn các vấn đề: Chúng ta sẽ cố gắng sử dụng
giá trị `val` trong thread được tạo ra _sau khi_ chúng ta đã gửi nó xuống
channel. Hãy cố gắng biên dịch mã trong Listing 16-9 để xem tại sao mã này
không được phép.

<Listing number="16-9" file-name="src/main.rs" caption="Cố gắng sử dụng `val` sau khi chúng ta đã gửi nó xuống channel">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-09/src/main.rs}}
```

</Listing>

Ở đây, chúng ta cố gắng in `val` sau khi chúng ta đã gửi nó xuống channel
thông qua `tx.send`. Cho phép điều này sẽ là một ý tưởng tệ: Khi giá trị
đã được gửi đến một thread khác, thread đó có thể sửa đổi hoặc drop nó
trước khi chúng ta cố gắng sử dụng giá trị đó lại. Có khả năng, các sửa
đổi của thread khác có thể gây ra lỗi hoặc kết quả không mong muốn do dữ
liệu không nhất quán hoặc không tồn tại. Tuy nhiên, Rust cung cấp cho chúng
ta một lỗi nếu chúng ta cố gắng biên dịch mã trong Listing 16-9:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-09/output.txt}}
```

Lỗi đồng thời của chúng ta đã gây ra một lỗi biên dịch. Hàm `send` sở hữu
tham số của nó, và khi giá trị được di chuyển, receiver sở hữu nó. Điều này
ngăn chúng ta sử dụng giá trị lại sau khi gửi nó; hệ thống ownership kiểm
tra rằng mọi thứ đều ổn.

<!-- Old headings. Do not remove or links may break. -->

<a id="sending-multiple-values-and-seeing-the-receiver-waiting"></a>

### Gửi Nhiều Giá Trị

Mã trong Listing 16-8 đã biên dịch và chạy, nhưng nó không rõ ràng cho chúng
ta thấy rằng hai thread riêng biệt đang nói chuyện với nhau qua channel.

Trong Listing 16-10, chúng ta đã thực hiện một số sửa đổi sẽ chứng minh rằng
mã trong Listing 16-8 đang chạy đồng thời: Thread được tạo ra bây giờ sẽ gửi
nhiều tin nhắn và tạm dừng trong một giây giữa mỗi tin nhắn.

<Listing number="16-10" file-name="src/main.rs" caption="Gửi nhiều tin nhắn và tạm dừng giữa mỗi cái">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-10/src/main.rs}}
```

</Listing>

Lần này, thread được tạo ra có một vector chuỗi mà chúng ta muốn gửi đến
thread chính. Chúng ta lặp lại trên chúng, gửi từng cái riêng biệt, và tạm
dừng giữa mỗi cái bằng cách gọi hàm `thread::sleep` với giá trị `Duration`
một giây.

Trong thread chính, chúng ta không gọi hàm `recv` một cách rõ ràng nữa:
Thay vào đó, chúng ta đang coi `rx` như một iterator. Đối với mỗi giá trị
được nhận, chúng ta đang in nó. Khi channel được đóng, lặp sẽ kết thúc.

Khi chạy mã trong Listing 16-10, bạn sẽ thấy kết quả sau với tạm dừng một
giây giữa mỗi dòng:

<!-- Not extracting output because changes to this output aren’t significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: from
Got: the
Got: thread
```

Vì chúng ta không có bất kỳ mã nào tạm dừng hoặc trì hoãn trong vòng lặp `for`
trong thread chính, chúng ta có thể nói rằng thread chính đang chờ để nhận các
giá trị từ thread được tạo ra.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-multiple-producers-by-cloning-the-transmitter"></a>

### Tạo Nhiều Producer

Trước đó chúng ta đã đề cập rằng `mpsc` là viết tắt của _multiple producer,
single consumer_. Hãy sử dụng `mpsc` và mở rộng mã trong Listing 16-10 để tạo
nhiều thread đều gửi giá trị đến cùng một receiver. Chúng ta có thể làm điều
đó bằng cách clone transmitter, như được hiển thị trong Listing 16-11.

<Listing number="16-11" file-name="src/main.rs" caption="Gửi nhiều tin nhắn từ nhiều producer">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-11/src/main.rs:here}}
```

</Listing>

Lần này, trước khi chúng ta tạo thread được tạo ra đầu tiên, chúng ta gọi
`clone` trên transmitter. Điều này sẽ cung cấp cho chúng ta một transmitter
mới mà chúng ta có thể chuyển đến thread được tạo ra đầu tiên. Chúng ta chuyển
transmitter gốc đến thread được tạo ra thứ hai. Điều này cung cấp cho chúng ta
hai thread, mỗi cái gửi các tin nhắn khác nhau đến receiver duy nhất.

Khi bạn chạy mã, kết quả của bạn sẽ trông giống như thế này:

<!-- Not extracting output because changes to this output aren’t significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

Bạn có thể thấy các giá trị theo một thứ tự khác, tùy thuộc vào hệ thống của bạn.
Đây là điều làm cho tính đồng thời thú vị cũng như khó khăn. Nếu bạn thử
`thread::sleep`, cung cấp các giá trị khác nhau trong các thread khác nhau,
mỗi lần chạy sẽ không xác định hơn và tạo kết quả khác nhau mỗi lần.

Bây giờ chúng ta đã xem cách channel hoạt động, hãy xem một phương pháp
đồng thời khác.
