## Xây Dựng một Web Server Đơn Luồng

Chúng ta sẽ bắt đầu bằng cách giúp một web server đơn luồng hoạt động. Trước khi chúng ta bắt đầu, hãy xem một cái nhìn tổng quan nhanh chóng về các giao thức liên quan đến việc xây dựng web servers. Chi tiết của các giao thức này nằm ngoài phạm vi của cuốn sách này, nhưng một tổng quan ngắn gọn sẽ cung cấp cho bạn thông tin bạn cần.

Hai giao thức chính liên quan đến web servers là _Hypertext Transfer Protocol_ _(HTTP)_ và _Transmission Control Protocol_ _(TCP)_. Cả hai giao thức đều là các giao thức _request-response_, có nghĩa là một _client_ khởi tạo yêu cầu và một _server_ lắng nghe các yêu cầu và cung cấp một phản hồi cho client. Nội dung của những yêu cầu và phản hồi đó được định nghĩa bởi các giao thức.

TCP là giao thức cấp thấp hơn mô tả chi tiết về cách thông tin truyền từ một server đến server khác nhưng không xác định thông tin đó là gì. HTTP xây dựng trên TCP bằng cách xác định nội dung của các yêu cầu và phản hồi. Về mặt kỹ thuật, có thể sử dụng HTTP với các giao thức khác, nhưng trong phần lớn các trường hợp, HTTP gửi dữ liệu của nó qua TCP. Chúng ta sẽ làm việc với các byte thô của các yêu cầu và phản hồi TCP và HTTP.

### Lắng Nghe Kết Nối TCP

Web server của chúng ta cần lắng nghe một kết nối TCP, vì vậy đó là phần đầu tiên mà chúng ta sẽ làm việc. Thư viện tiêu chuẩn cung cấp một mô-đun `std::net` cho phép chúng ta thực hiện điều này. Hãy tạo một dự án mới theo cách thông thường:

```console
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

Bây giờ nhập mã trong Listing 21-1 vào _src/main.rs_ để bắt đầu. Mã này sẽ lắng nghe địa chỉ cục bộ `127.0.0.1:7878` cho các stream TCP đến. Khi nó nhận được một stream đến, nó sẽ in `Connection established!`.

<Listing number="21-1" file-name="src/main.rs" caption="Lắng nghe các stream đến và in một thông báo khi chúng ta nhận được một stream">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-01/src/main.rs}}
```

</Listing>

Sử dụng `TcpListener`, chúng ta có thể lắng nghe các kết nối TCP tại địa chỉ `127.0.0.1:7878`. Trong địa chỉ, phần trước dấu hai chấm là một địa chỉ IP đại diện cho máy tính của bạn (điều này giống nhau trên mọi máy tính và không đại diện cho máy tính của các tác giả cụ thể), và `7878` là cổng. Chúng ta đã chọn cổng này vì hai lý do: HTTP thường không được chấp nhận trên cổng này, vì vậy server của chúng ta không chắc sẽ xung đột với bất kỳ web server nào khác mà bạn có thể chạy trên máy của mình, và 7878 là _rust_ được gõ trên một chiếc điện thoại.

Hàm `bind` trong trường hợp này hoạt động giống như hàm `new` ở chỗ nó sẽ trả về một phiên bản `TcpListener` mới. Hàm được gọi là `bind` vì trong mạng, kết nối đến một cổng để lắng nghe được gọi là "binding to a port."

Hàm `bind` trả về một `Result<T, E>`, chỉ ra rằng có thể binding sẽ không thành công, ví dụ như, nếu chúng ta chạy hai phiên bản chương trình của mình và vì vậy có hai chương trình lắng nghe cùng một cổng. Vì chúng ta đang viết một server cơ bản chỉ cho mục đích học tập, chúng ta sẽ không lo lắng về việc xử lý các loại lỗi này; thay vào đó, chúng ta sử dụng `unwrap` để dừng chương trình nếu các lỗi xảy ra.

Phương thức `incoming` trên `TcpListener` trả về một iterator cung cấp cho chúng ta một chuỗi các stream (cụ thể hơn, các stream của loại `TcpStream`). Một _stream_ duy nhất đại diện cho một kết nối mở giữa client và server. _Connection_ là tên của toàn bộ quy trình yêu cầu và phản hồi trong đó client kết nối đến server, server tạo ra phản hồi, và server đóng kết nối. Do đó, chúng ta sẽ đọc từ `TcpStream` để xem client đã gửi gì và sau đó viết phản hồi của chúng ta vào stream để gửi dữ liệu lại cho client. Nói chung, vòng lặp `for` này sẽ xử lý từng kết nối theo lượt và tạo ra một chuỗi các stream để chúng ta xử lý.

Hiện tại, cách xử lý stream của chúng ta bao gồm gọi `unwrap` để kết thúc chương trình của chúng ta nếu stream có bất kỳ lỗi nào; nếu không có lỗi nào, chương trình sẽ in một thông báo. Chúng ta sẽ thêm nhiều chức năng hơn cho trường hợp thành công trong listing tiếp theo. Lý do tại sao chúng ta có thể nhận được lỗi từ phương thức `incoming` khi một client kết nối đến server là vì chúng ta không thực sự lặp lại các kết nối. Thay vào đó, chúng ta đang lặp lại _connection attempts_. Kết nối có thể không thành công vì một số lý do, nhiều trong số đó là cụ thể của hệ điều hành. Ví dụ, nhiều hệ điều hành có giới hạn về số lượng kết nối mở đồng thời mà chúng có thể hỗ trợ; những nỗ lực kết nối mới vượt quá con số đó sẽ tạo ra lỗi cho đến khi một số kết nối mở được đóng lại.

Hãy thử chạy mã này! Gọi `cargo run` trong terminal và sau đó tải _127.0.0.1:7878_ vào một trình duyệt web. Trình duyệt sẽ hiển thị một thông báo lỗi giống như "Connection reset" vì server hiện không gửi lại bất kỳ dữ liệu nào. Nhưng khi bạn nhìn vào terminal của mình, bạn sẽ thấy một số thông báo được in khi trình duyệt kết nối đến server!

```text
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
```

Đôi khi bạn sẽ thấy nhiều thông báo được in cho một yêu cầu trình duyệt; lý do có thể là trình duyệt đang tạo một yêu cầu cho trang cũng như một yêu cầu cho các tài nguyên khác, như biểu tượng _favicon.ico_ xuất hiện trong tab trình duyệt.

Nó cũng có thể là trình duyệt đang cố gắng kết nối đến server nhiều lần vì server không phản hồi bằng bất kỳ dữ liệu nào. Khi `stream` vượt ra ngoài phạm vi và bị drop ở cuối vòng lặp, kết nối được đóng lại như một phần của triển khai `drop`. Trình duyệt đôi khi xử lý các kết nối đóng bằng cách thử lại, vì vấn đề có thể là tạm thời.

Các trình duyệt cũng đôi khi mở nhiều kết nối đến server mà không gửi bất kỳ yêu cầu nào để nếu chúng *có thể* gửi yêu cầu sau này, những yêu cầu đó có thể xảy ra nhanh hơn. Khi điều này xảy ra, server của chúng ta sẽ thấy mỗi kết nối, bất kể liệu có bất kỳ yêu cầu nào qua kết nối đó hay không. Nhiều phiên bản trình duyệt dựa trên Chrome làm điều này, ví dụ; bạn có thể vô hiệu hóa tối ưu hóa đó bằng cách sử dụng chế độ duyệt riêng tư hoặc sử dụng một trình duyệt khác.

Yếu tố quan trọng là chúng ta đã thành công trong việc lấy một handle cho một kết nối TCP!

Hãy nhớ dừng chương trình bằng cách nhấn <kbd>ctrl</kbd>-<kbd>C</kbd> khi bạn hoàn thành việc chạy một phiên bản cụ thể của mã. Sau đó, khởi động lại chương trình bằng cách gọi lệnh `cargo run` sau khi bạn thực hiện từng bộ thay đổi mã để đảm bảo bạn đang chạy mã mới nhất.

### Đọc Yêu Cầu

Hãy triển khai chức năng để đọc yêu cầu từ trình duyệt! Để tách biệt các mối quan tâm của việc trước tiên lấy một kết nối và sau đó thực hiện một hành động nào đó với kết nối, chúng ta sẽ bắt đầu một hàm mới để xử lý các kết nối. Trong hàm `handle_connection` mới này, chúng ta sẽ đọc dữ liệu từ luồng TCP và in nó để chúng ta có thể thấy dữ liệu được gửi từ trình duyệt. Thay đổi mã để trông giống như Listing 21-2.

<Listing number="21-2" file-name="src/main.rs" caption="Đọc từ `TcpStream` và in dữ liệu">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-02/src/main.rs}}
```

</Listing>

Chúng ta đưa `std::io::BufReader` và `std::io::prelude` vào phạm vi để có quyền truy cập vào các traits và loại cho phép chúng ta đọc từ và viết vào stream. Trong vòng lặp `for` trong hàm `main`, thay vì in một thông báo nói rằng chúng ta đã tạo một kết nối, chúng ta bây giờ gọi hàm `handle_connection` mới và truyền `stream` cho nó.

Trong hàm `handle_connection`, chúng ta tạo một phiên bản `BufReader` mới bao bọc một tham chiếu đến `stream`. `BufReader` thêm buffering bằng cách quản lý các cuộc gọi tới các phương thức trait `std::io::Read` cho chúng ta.

Chúng ta tạo một biến được đặt tên là `http_request` để thu thập các dòng của yêu cầu mà trình duyệt gửi đến server của chúng ta. Chúng ta chỉ ra rằng chúng ta muốn thu thập những dòng này trong một vector bằng cách thêm chú thích loại `Vec<_>`.

`BufReader` triển khai trait `std::io::BufRead`, cung cấp phương thức `lines`. Phương thức `lines` trả về một iterator của `Result<String, std::io::Error>` bằng cách chia nhỏ luồng dữ liệu bất cứ khi nào nó thấy một byte dòng mới. Để nhận từng `String`, chúng ta `map` và `unwrap` từng `Result`. `Result` có thể là một lỗi nếu dữ liệu không phải là UTF-8 hợp lệ hoặc nếu có vấn đề đọc từ stream. Một lần nữa, một chương trình sản xuất nên xử lý các lỗi này một cách duyên dáng hơn, nhưng chúng ta chọn để dừng chương trình trong trường hợp lỗi để đơn giản.

Trình duyệt báo hiệu kết thúc một yêu cầu HTTP bằng cách gửi hai ký tự dòng mới liên tiếp, vì vậy để lấy một yêu cầu từ stream, chúng ta lấy các dòng cho đến khi chúng ta nhận được một dòng là chuỗi trống. Khi chúng ta đã thu thập các dòng vào vector, chúng ta in chúng bằng cách sử dụng định dạng gỡ lỗi đẹp để chúng ta có thể xem xét các hướng dẫn mà trình duyệt web gửi đến server của chúng ta.

Hãy thử mã này! Bắt đầu chương trình và tạo một yêu cầu trong một trình duyệt web lần nữa. Lưu ý rằng chúng ta vẫn sẽ gặp một trang lỗi trong trình duyệt, nhưng đầu ra của chương trình của chúng ta trong terminal sẽ bây giờ trông tương tự như thế này:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-02
cargo run
make a request to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "DNT: 1",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
    "Cache-Control: max-age=0",
]
```

Tùy thuộc vào trình duyệt của bạn, bạn có thể nhận được đầu ra hơi khác. Bây giờ chúng ta in dữ liệu yêu cầu, chúng ta có thể thấy lý do tại sao chúng ta nhận được nhiều kết nối từ một yêu cầu trình duyệt bằng cách xem xét đường dẫn sau `GET` trong dòng đầu tiên của yêu cầu. Nếu các kết nối lặp lại đều yêu cầu _/_, chúng ta biết rằng trình duyệt đang cố gắng tìm nạp _/_ nhiều lần vì nó không nhận được phản hồi từ chương trình của chúng ta.

Hãy chia nhỏ dữ liệu yêu cầu này để hiểu điều mà trình duyệt đang yêu cầu chương trình của chúng ta.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-closer-look-at-an-http-request"></a>
<a id="looking-closer-at-an-http-request"></a>

### Xem Xét Kỹ Lưỡng Hơn một Yêu Cầu HTTP

HTTP là một giao thức dựa trên văn bản, và một yêu cầu có định dạng này:

```text
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

Dòng đầu tiên là _request line_ chứa thông tin về những gì client đang yêu cầu. Phần đầu tiên của dòng yêu cầu chỉ ra phương thức được sử dụng, chẳng hạn như `GET` hoặc `POST`, mô tả cách client tạo yêu cầu này. Client của chúng ta đã sử dụng một yêu cầu `GET`, có nghĩa là nó đang yêu cầu thông tin.

Phần tiếp theo của dòng yêu cầu là _/_, chỉ ra _uniform resource identifier_ _(URI)_ mà client yêu cầu: Một URI gần như, nhưng không hoàn toàn giống, _uniform resource locator_ _(URL)_. Sự khác biệt giữa các URI và URL không quan trọng cho mục đích của chúng ta trong chương này, nhưng spec HTTP sử dụng thuật ngữ _URI_, vì vậy chúng ta có thể chỉ cần thay thế tinh thần _URL_ cho _URI_ ở đây.

Phần cuối cùng là phiên bản HTTP mà client sử dụng, và sau đó dòng yêu cầu kết thúc trong chuỗi CRLF. (_CRLF_ là viết tắt của _carriage return_ và _line feed_, những thuật ngữ từ những ngày của máy đánh chữ!) Chuỗi CRLF cũng có thể được viết là `\r\n`, trong đó `\r` là dòng quay lại và `\n` là nguồn cấp dữ liệu dòng. Chuỗi _CRLF_ tách dòng yêu cầu khỏi phần còn lại của dữ liệu yêu cầu. Lưu ý rằng khi CRLF được in, chúng ta thấy một dòng mới bắt đầu chứ không phải `\r\n`.

Xem xét dữ liệu dòng yêu cầu chúng ta nhận được từ chạy chương trình của chúng ta cho đến nay, chúng ta thấy rằng `GET` là phương thức, _/_ là URI yêu cầu, và `HTTP/1.1` là phiên bản.

Sau dòng yêu cầu, các dòng còn lại bắt đầu từ `Host:` trở đi là các headers. Các yêu cầu `GET` không có body.

Hãy thử tạo một yêu cầu từ một trình duyệt khác hoặc yêu cầu một địa chỉ khác, chẳng hạn như _127.0.0.1:7878/test_, để xem cách dữ liệu yêu cầu thay đổi.

Bây giờ chúng ta biết những gì trình duyệt đang yêu cầu, hãy gửi lại một số dữ liệu!

### Viết một Phản Hồi

Chúng ta sắp triển khai việc gửi dữ liệu để phản hồi một yêu cầu của client. Các phản hồi có định dạng sau:

```text
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

Dòng đầu tiên là một _status line_ chứa phiên bản HTTP được sử dụng trong phản hồi, một mã trạng thái số tóm tắt kết quả của yêu cầu, và một cụm từ lý do cung cấp một mô tả văn bản của mã trạng thái. Sau chuỗi CRLF là bất kỳ headers nào, một chuỗi CRLF khác, và body của phản hồi.

Dưới đây là một phản hồi ví dụ sử dụng phiên bản HTTP 1.1 và có mã trạng thái 200, một cụm từ lý do OK, không có headers và không có body:

```text
HTTP/1.1 200 OK\r\n\r\n
```

Mã trạng thái 200 là phản hồi thành công tiêu chuẩn. Văn bản là một phản hồi HTTP thành công rất nhỏ. Hãy viết cái này vào stream như phản hồi của chúng ta cho một yêu cầu thành công! Từ hàm `handle_connection`, hãy loại bỏ `println!` đang in dữ liệu yêu cầu và thay thế nó bằng mã trong Listing 21-3.

<Listing number="21-3" file-name="src/main.rs" caption="Viết một phản hồi HTTP thành công nhỏ vào stream">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-03/src/main.rs:here}}
```

</Listing>

Dòng đầu tiên mới định nghĩa biến `response` chứa dữ liệu của thông báo thành công. Sau đó, chúng ta gọi `as_bytes` trên `response` của chúng ta để chuyển đổi dữ liệu chuỗi thành byte. Phương thức `write_all` trên `stream` nhận một `&[u8]` và gửi những byte đó trực tiếp xuống kết nối. Vì hoạt động `write_all` có thể thất bại, chúng ta sử dụng `unwrap` trên bất kỳ kết quả lỗi nào như trước. Một lần nữa, trong một ứng dụng thực, bạn sẽ thêm xử lý lỗi ở đây.

Với những thay đổi này, hãy chạy mã của chúng ta và tạo một yêu cầu. Chúng ta không còn in bất kỳ dữ liệu nào vào terminal, vì vậy chúng ta sẽ không thấy bất kỳ đầu ra nào khác ngoài đầu ra từ Cargo. Khi bạn tải _127.0.0.1:7878_ vào một trình duyệt web, bạn sẽ nhận được một trang trống thay vì một lỗi. Bạn vừa handcoded nhận một yêu cầu HTTP và gửi một phản hồi!

### Trả Về HTML Thực

Hãy triển khai chức năng để trả về nhiều hơn một trang trống. Tạo tệp mới _hello.html_ trong thư mục gốc của dự án của bạn, không phải trong thư mục _src_. Bạn có thể nhập bất kỳ HTML nào bạn muốn; Listing 21-4 hiển thị một khả năng.

<Listing number="21-4" file-name="hello.html" caption="Một tệp HTML mẫu để trả về trong một phản hồi">

```html
{{#include ../listings/ch21-web-server/listing-21-05/hello.html}}
```

</Listing>

Đây là một tài liệu HTML5 tối thiểu với một tiêu đề và một số văn bản. Để trả về điều này từ server khi một yêu cầu được nhận, chúng ta sẽ sửa đổi `handle_connection` như được hiển thị trong Listing 21-5 để đọc tệp HTML, thêm nó vào phản hồi như một body, và gửi nó.

<Listing number="21-5" file-name="src/main.rs" caption="Gửi nội dung của *hello.html* như body của phản hồi">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-05/src/main.rs:here}}
```

</Listing>

Chúng ta đã thêm `fs` vào câu lệnh `use` để đưa mô-đun hệ thống tệp của thư viện tiêu chuẩn vào phạm vi. Mã để đọc nội dung của một tệp vào một chuỗi sẽ trông quen thuộc; chúng ta đã sử dụng nó khi chúng ta đọc nội dung của một tệp cho dự án I/O của chúng ta trong Listing 12-4.

Tiếp theo, chúng ta sử dụng `format!` để thêm nội dung của tệp như body của phản hồi thành công. Để đảm bảo một phản hồi HTTP hợp lệ, chúng ta thêm header `Content-Length`, được đặt thành kích thước của body phản hồi của chúng ta—trong trường hợp này, kích thước của `hello.html`.

Chạy mã này với `cargo run` và tải _127.0.0.1:7878_ vào trình duyệt của bạn; bạn sẽ thấy HTML của bạn được hiển thị!

Hiện tại, chúng ta đang bỏ qua dữ liệu yêu cầu trong `http_request` và chỉ gửi lại nội dung của tệp HTML một cách vô điều kiện. Điều đó có nghĩa là nếu bạn cố gắng yêu cầu _127.0.0.1:7878/something-else_ trong trình duyệt của bạn, bạn vẫn sẽ nhận được lại phản hồi HTML tương tự này. Tại thời điểm này, server của chúng ta rất hạn chế và không làm những gì hầu hết các web servers làm. Chúng ta muốn tùy chỉnh các phản hồi của chúng ta tùy thuộc vào yêu cầu và chỉ gửi lại tệp HTML cho một yêu cầu được tạo thành tốt để _/_.

### Xác Thực Yêu Cầu và Phản Hồi Có Chọn Lọc

Hiện tại, web server của chúng ta sẽ trả về HTML trong tệp bất kể client yêu cầu gì. Hãy thêm chức năng để kiểm tra xem trình duyệt có yêu cầu _/_ trước khi trả về tệp HTML hay không và để trả về một lỗi nếu trình duyệt yêu cầu bất cứ thứ gì khác. Để làm điều này, chúng ta cần sửa đổi `handle_connection`, như được hiển thị trong Listing 21-6. Mã mới này kiểm tra nội dung của yêu cầu được nhận dựa trên những gì chúng ta biết là một yêu cầu cho _/_ trông như thế nào và thêm các khối `if` và `else` để xử lý các yêu cầu khác nhau.

<Listing number="21-6" file-name="src/main.rs" caption="Xử lý các yêu cầu đến */* khác với các yêu cầu khác">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-06/src/main.rs:here}}
```

</Listing>

Chúng ta chỉ sắp xem xét dòng đầu tiên của yêu cầu HTTP, vì vậy thay vì đọc toàn bộ yêu cầu vào một vector, chúng ta đang gọi `next` để lấy mục đầu tiên từ iterator. `unwrap` đầu tiên sẽ chăm sóc `Option` và dừng chương trình nếu iterator không có mục nào. `unwrap` thứ hai xử lý `Result` và có tác dụng giống như `unwrap` nằm trong `map` được thêm vào Listing 21-2.

Tiếp theo, chúng ta kiểm tra `request_line` để xem nó có bằng dòng yêu cầu của một yêu cầu GET cho đường dẫn _/_ hay không. Nếu đó là vậy, khối `if` trả về nội dung của tệp HTML của chúng ta.

Nếu `request_line` _không_ bằng yêu cầu GET cho đường dẫn _/_, điều đó có nghĩa là chúng ta đã nhận được một yêu cầu nào đó khác. Chúng ta sẽ thêm mã vào khối `else` trong một lúc để phản hồi tất cả các yêu cầu khác.

Chạy mã này bây giờ và yêu cầu _127.0.0.1:7878_; bạn sẽ nhận được HTML trong _hello.html_. Nếu bạn tạo bất kỳ yêu cầu nào khác, chẳng hạn như _127.0.0.1:7878/something-else_, bạn sẽ gặp một lỗi kết nối như những yêu cầu bạn thấy khi chạy mã trong Listing 21-1 và Listing 21-2.

Bây giờ hãy thêm mã trong Listing 21-7 vào khối `else` để trả về một phản hồi với mã trạng thái 404, chỉ ra rằng nội dung cho yêu cầu không được tìm thấy. Chúng ta cũng sẽ trả về một số HTML cho một trang để hiển thị trong trình duyệt chỉ ra phản hồi cho người dùng cuối.

<Listing number="21-7" file-name="src/main.rs" caption="Phản hồi với mã trạng thái 404 và một trang lỗi nếu bất cứ thứ gì khác ngoài */* được yêu cầu">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-07/src/main.rs:here}}
```

</Listing>

Ở đây, phản hồi của chúng ta có một dòng trạng thái với mã trạng thái 404 và cụm từ lý do `NOT FOUND`. Body của phản hồi sẽ là HTML trong tệp _404.html_. Bạn sẽ cần tạo một tệp _404.html_ tiếp theo _hello.html_ cho trang lỗi; một lần nữa, vui lòng sử dụng bất kỳ HTML nào bạn muốn, hoặc sử dụng ví dụ HTML trong Listing 21-8.

<Listing number="21-8" file-name="404.html" caption="Nội dung mẫu cho trang để gửi lại bằng bất kỳ phản hồi 404 nào">

```html
{{#include ../listings/ch21-web-server/listing-21-07/404.html}}
```

</Listing>

Với những thay đổi này, hãy chạy server của bạn lại. Yêu cầu _127.0.0.1:7878_ sẽ trả về nội dung của _hello.html_, và bất kỳ yêu cầu nào khác, như _127.0.0.1:7878/foo_, sẽ trả về HTML lỗi từ _404.html_.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-touch-of-refactoring"></a>

### Tái Cấu Trúc

Tại thời điểm này, các khối `if` và `else` có rất nhiều sự lặp lại: Chúng đều đang đọc các tệp và viết nội dung của các tệp vào stream. Sự khác biệt duy nhất là dòng trạng thái và tên tệp. Hãy làm cho mã ngắn gọn hơn bằng cách kéo ra những sự khác biệt đó thành các dòng `if` và `else` riêng biệt sẽ gán các giá trị của dòng trạng thái và tên tệp cho các biến; chúng ta có thể sử dụng những biến đó một cách vô điều kiện trong mã để đọc tệp và viết phản hồi. Listing 21-9 hiển thị mã kết quả sau khi thay thế các khối `if` và `else` lớn.

<Listing number="21-9" file-name="src/main.rs" caption="Tái cấu trúc các khối `if` và `else` để chỉ chứa mã khác nhau giữa hai trường hợp">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-09/src/main.rs:here}}
```

</Listing>

Bây giờ các khối `if` và `else` chỉ trả về các giá trị thích hợp cho dòng trạng thái và tên tệp trong một tuple; chúng ta sau đó sử dụng destructuring để gán hai giá trị này cho `status_line` và `filename` bằng cách sử dụng một pattern trong câu lệnh `let`, như được thảo luận trong Chương 19.

Mã được lặp lại trước đây nằm ngoài các khối `if` và `else` và sử dụng các biến `status_line` và `filename`. Điều này giúp dễ dàng hơn để xem sự khác biệt giữa hai trường hợp, và nó có nghĩa là chúng ta chỉ có một nơi để cập nhật mã nếu chúng ta muốn thay đổi cách đọc tệp và viết phản hồi hoạt động. Hành vi của mã trong Listing 21-9 sẽ giống với Listing 21-7.

Tuyệt vời! Chúng ta bây giờ có một web server đơn giản trong xấp xỉ 40 dòng mã Rust phản hồi một yêu cầu bằng một trang nội dung và phản hồi tất cả các yêu cầu khác bằng phản hồi 404.

Hiện tại, server của chúng ta chạy trong một luồng duy nhất, có nghĩa là nó chỉ có thể phục vụ một yêu cầu tại một thời điểm. Hãy xem xét cách điều đó có thể là một vấn đề bằng cách mô phỏng một số yêu cầu chậm. Sau đó, chúng ta sẽ khắc phục nó để server của chúng ta có thể xử lý nhiều yêu cầu cùng một lúc.
