<!-- Old headings. Do not remove or links may break. -->

<a id="extensible-concurrency-with-the-sync-and-send-traits"></a>
<a id="extensible-concurrency-with-the-send-and-sync-traits"></a>

## Lập trình Concurrent Có Mở Rộng với `Send` và `Sync`

Điều thú vị là hầu hết các tính năng concurrency mà chúng ta đã nói đến cho đến
nay trong chương này là một phần của thư viện chuẩn, không phải của ngôn ngữ.
Các lựa chọn của bạn để xử lý concurrency không bị giới hạn ở ngôn ngữ hoặc
thư viện chuẩn; bạn có thể viết các tính năng concurrency của riêng mình hoặc
sử dụng những tính năng được viết bởi những người khác.

Tuy nhiên, trong số các khái niệm concurrency chính được nhúng trong ngôn ngữ
thay vì thư viện chuẩn là các trait `std::marker` có tên `Send` và `Sync`.

<!-- Old headings. Do not remove or links may break. -->

<a id="allowing-transference-of-ownership-between-threads-with-send"></a>

### Chuyển giao Ownership giữa các Thread

Trait `Send` marker cho biết rằng ownership của các giá trị của kiểu triển khai
`Send` có thể được chuyển giao giữa các thread. Hầu hết mọi kiểu Rust triển khai
`Send`, nhưng có một số ngoại lệ, bao gồm `Rc<T>`: Điều này không thể triển khai
`Send` vì nếu bạn nhân bản một giá trị `Rc<T>` và cố gắng chuyển giao ownership
của bản nhân này sang một thread khác, cả hai thread có thể cập nhật số lượng
reference cùng một lúc. Vì lý do này, `Rc<T>` được triển khai để sử dụng trong
các tình huống single-threaded nơi bạn không muốn phải trả giá về hiệu suất
thread-safe.

Do đó, hệ thống kiểu của Rust và trait bounds đảm bảo rằng bạn không bao giờ
có thể vô tình gửi một giá trị `Rc<T>` giữa các thread một cách không an toàn.
Khi chúng ta cố gắng làm điều này trong Listing 16-14, chúng ta gặp lỗi
`` the trait `Send` is not implemented for `Rc<Mutex<i32>>` ``. Khi chúng ta
chuyển sang `Arc<T>`, thứ mà triển khai `Send`, mã được biên dịch.

Bất kỳ kiểu nào được tạo thành hoàn toàn bởi các kiểu `Send` cũng tự động được
đánh dấu là `Send`. Hầu hết tất cả các kiểu primitive là `Send`, ngoại trừ raw
pointers, mà chúng ta sẽ thảo luận trong Chương 20.

<!-- Old headings. Do not remove or links may break. -->

<a id="allowing-access-from-multiple-threads-with-sync"></a>

### Truy cập từ nhiều Thread

Trait `Sync` marker cho biết rằng an toàn cho kiểu triển khai `Sync` được tham
chiếu từ nhiều thread. Nói cách khác, bất kỳ kiểu `T` nào triển khai `Sync` nếu
`&T` (một reference bất biến đến `T`) triển khai `Send`, có nghĩa là reference
có thể được gửi an toàn đến một thread khác. Tương tự như `Send`, tất cả các
kiểu primitive triển khai `Sync`, và các kiểu được tạo thành hoàn toàn bởi các
kiểu triển khai `Sync` cũng triển khai `Sync`.

Smart pointer `Rc<T>` cũng không triển khai `Sync` vì những lý do tương tự mà nó
không triển khai `Send`. Kiểu `RefCell<T>` (mà chúng ta đã nói đến trong
Chương 15) và họ các kiểu `Cell<T>` liên quan không triển khai `Sync`. Việc
triển khai borrow checking mà `RefCell<T>` thực hiện tại thời gian chạy không
thread-safe. Smart pointer `Mutex<T>` triển khai `Sync` và có thể được sử dụng
để chia sẻ quyền truy cập với nhiều thread, như bạn đã thấy trong [“Truy cập
Chia sẻ đến `Mutex<T>`”][shared-access]<!-- ignore -->.

### Triển khai `Send` và `Sync` Thủ công là Không An toàn

Vì các kiểu được tạo thành hoàn toàn bởi các kiểu khác triển khai các trait
`Send` và `Sync` cũng tự động triển khai `Send` và `Sync`, chúng ta không phải
triển khai những trait này thủ công. Là marker traits, chúng thậm chí không có
bất kỳ phương thức nào để triển khai. Chúng chỉ hữu ích cho việc thực thi các
bất biến liên quan đến concurrency.

Triển khai thủ công các trait này liên quan đến việc triển khai mã Rust không
an toàn. Chúng ta sẽ nói về việc sử dụng mã Rust không an toàn trong Chương 20;
bây giờ, thông tin quan trọng là xây dựng các kiểu concurrent mới không được
tạo thành từ các phần `Send` và `Sync` đòi hỏi suy nghĩ cẩn thận để duy trì
các đảm bảo an toàn. [“The Rustonomicon”][nomicon] có thêm thông tin về những
đảm bảo này và cách duy trì chúng.

## Tóm tắt

Đây không phải là lần cuối cùng bạn sẽ thấy concurrency trong cuốn sách này:
Chương tiếp theo tập trung vào lập trình async, và dự án trong Chương 21 sẽ sử
dụng các khái niệm trong chương này trong một tình huống thực tế hơn so với các
ví dụ nhỏ hơn được thảo luận ở đây.

Như đã đề cập trước đó, vì rất ít về cách Rust xử lý concurrency là một phần của
ngôn ngữ, nhiều giải pháp concurrency được triển khai dưới dạng crates. Chúng
phát triển nhanh hơn thư viện chuẩn, vì vậy hãy chắc chắn tìm kiếm trực tuyến
để tìm các crates tân tiến hiện tại để sử dụng trong các tình huống multithreaded.

Thư viện chuẩn Rust cung cấp các channels cho việc truyền tin nhắn và các kiểu
smart pointer, như `Mutex<T>` và `Arc<T>`, an toàn để sử dụng trong các ngữ cảnh
concurrent. Hệ thống kiểu và borrow checker đảm bảo rằng mã sử dụng những giải
pháp này sẽ không kết thúc với data races hoặc invalid references. Khi bạn
biên dịch thành công mã của mình, bạn có thể yên tâm rằng nó sẽ chạy vui vẻ
trên nhiều thread mà không có loại lỗi khó theo dõi thường gặp trong các ngôn
ngữ khác. Lập trình concurrent không còn là một khái niệm đáng sợ nữa: Hãy tiến
tới và làm cho các chương trình của bạn concurrent, một cách táo bạo!

[shared-access]: ch16-03-shared-state.html#shared-access-to-mutext
[nomicon]: ../nomicon/index.html
