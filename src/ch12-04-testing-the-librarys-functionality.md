<!-- Old headings. Do not remove or links may break. -->
<a id="developing-the-librarys-functionality-with-test-driven-development"></a>
## Bổ sung chức năng với phát triển hướng kiểm thử (Test-Driven Development)

Giờ đây, vì đã tách logic tìm kiếm trong _src/lib.rs_ ra khỏi hàm `main`, việc viết kiểm thử cho chức năng lõi của mã nguồn trở nên dễ dàng hơn nhiều. Chúng ta có thể gọi trực tiếp các hàm với nhiều tham số khác nhau và kiểm tra giá trị trả về mà không cần gọi chương trình nhị phân từ dòng lệnh.

Trong phần này, chúng ta sẽ bổ sung logic tìm kiếm vào chương trình `minigrep` bằng quy trình phát triển hướng kiểm thử (TDD) với các bước sau:

1. Viết một kiểm thử bị thất bại và chạy nó để bảo đảm nó thất bại vì đúng lý do bạn mong đợi.
2. Viết hoặc chỉnh sửa vừa đủ mã để làm cho kiểm thử mới vượt qua.
3. Tái cấu trúc (refactor) đoạn mã bạn vừa thêm hoặc thay đổi và bảo đảm toàn bộ kiểm thử vẫn vượt qua.
4. Lặp lại từ bước 1!

Mặc dù chỉ là một trong nhiều phương pháp phát triển phần mềm, TDD có thể hỗ trợ dẫn dắt thiết kế mã nguồn. Việc viết kiểm thử trước khi viết mã làm cho kiểm thử vượt qua giúp duy trì mức độ bao phủ kiểm thử cao trong suốt quá trình phát triển.

Chúng ta sẽ áp dụng TDD cho việc hiện thực chức năng thực sự thực hiện tìm kiếm chuỗi truy vấn trong nội dung tệp và tạo ra danh sách các dòng khớp với truy vấn. Chúng ta sẽ bổ sung chức năng này trong một hàm tên là `search`.

### Viết một kiểm thử thất bại

Trong _src/lib.rs_, chúng ta sẽ thêm một mô-đun `tests` cùng với một hàm kiểm thử, giống như đã thực hiện trong [Chương 11][ch11-anatomy]<!-- ignore -->. Hàm kiểm thử này đặc tả hành vi mong muốn của hàm `search`: nó sẽ nhận một chuỗi truy vấn và đoạn văn bản cần tìm kiếm, sau đó trả về chỉ những dòng trong văn bản có chứa chuỗi truy vấn. Liệt kê 12-15 cho thấy kiểm thử này.

<Listing number="12-15" file-name="src/lib.rs" caption="Tạo một kiểm thử thất bại cho hàm `search` cho phần chức năng mà chúng ta mong muốn có">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-15/src/lib.rs:here}}
```

</Listing>

Kiểm thử này tìm kiếm chuỗi "`duct`". Văn bản cần tìm kiếm gồm ba dòng, và chỉ có một dòng chứa "`duct`" (lưu ý dấu gạch chéo ngược ngay sau dấu ngoặc kép mở báo cho Rust không chèn ký tự xuống dòng ở đầu nội dung literal chuỗi này). Chúng ta khẳng định rằng giá trị trả về từ hàm `search` chỉ chứa đúng dòng mà ta kỳ vọng.

Nếu chạy kiểm thử này ngay bây giờ, nó sẽ thất bại vì macro `unimplemented!` sẽ panic với thông báo "not implemented". Theo đúng nguyên tắc TDD, chúng ta sẽ thực hiện một bước nhỏ là thêm vừa đủ mã để làm cho lời gọi hàm không còn panic: định nghĩa hàm `search` luôn trả về một vector rỗng, như minh họa trong Liệt kê 12-16. Lúc đó, kiểm thử sẽ biên dịch được nhưng thất bại vì một vector rỗng không khớp với vector chứa dòng "`safe, fast, productive.`".

<Listing number="12-16" file-name="src/lib.rs" caption="Định nghĩa vừa đủ hàm `search` để việc gọi nó không bị panic">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-16/src/lib.rs:here}}
```

</Listing>

Bây giờ, hãy thảo luận lý do tại sao chúng ta cần khai báo rõ ràng vòng đời `'a` trong chữ ký của hàm `search` và sử dụng vòng đời đó cho cả tham số `contents` và giá trị trả về. Nhắc lại trong [Chương 10][ch10-lifetimes]<!-- ignore --> rằng các tham số vòng đời chỉ ra vòng đời của tham số nào được liên kết với vòng đời của giá trị trả về. Trong trường hợp này, chúng ta chỉ ra rằng vector trả về phải chứa các lát cắt chuỗi tham chiếu đến các lát cắt của tham số `contents` (thay vì tham số `query`).

Nói cách khác, chúng ta nói cho Rust biết dữ liệu được trả về bởi hàm `search` sẽ sống lâu bằng dữ liệu được truyền vào hàm `search` qua tham số `contents`. Điều này rất quan trọng! Dữ liệu mà một slice tham chiếu tới phải còn hợp lệ để tham chiếu đó hợp lệ; nếu trình biên dịch giả định rằng chúng ta tạo các lát cắt chuỗi từ `query` thay vì từ `contents`, nó sẽ thực hiện kiểm tra an toàn sai cách.

Nếu chúng ta quên chú thích vòng đời và cố gắng biên dịch hàm này, ta sẽ nhận được lỗi sau:

```console
{{#include ../listings/ch12-an-io-project/output-only-02-missing-lifetimes/output.txt}}
```

Rust không thể biết tham số nào trong hai tham số cần được dùng cho kiểu trả về, vì vậy chúng ta phải chỉ ra một cách tường minh. Lưu ý rằng phần trợ giúp gợi ý chỉ định cùng một tham số vòng đời cho tất cả tham số và kiểu trả về, nhưng gợi ý đó là không chính xác! Vì `contents` là tham số chứa toàn bộ văn bản và chúng ta muốn trả về những phần của văn bản đó khớp với truy vấn, nên chúng ta biết rằng `contents` là tham số duy nhất cần được liên kết với giá trị trả về thông qua cú pháp vòng đời.

Các ngôn ngữ lập trình khác không yêu cầu bạn phải liên kết tham số với giá trị trả về ngay trong chữ ký hàm, nhưng việc này sẽ dần trở nên quen thuộc theo thời gian. Bạn có thể muốn so sánh ví dụ này với các ví dụ trong phần [“Xác thực tham chiếu với vòng đời”][validating-references-with-lifetimes]<!-- ignore --> ở Chương 10.

### Viết mã để vượt qua bài kiểm thử

Hiện tại, bài kiểm thử của chúng ta đang thất bại vì hàm luôn trả về một vector rỗng. Để khắc phục điều đó và hiện thực hóa `search`, chương trình cần thực hiện các bước sau:

1. Lặp qua từng dòng trong nội dung (`contents`).
2. Kiểm tra xem dòng đó có chứa chuỗi truy vấn (`query`) hay không.
3. Nếu có, thêm dòng đó vào danh sách các giá trị sẽ được trả về.
4. Nếu không, không làm gì cả.
5. Trả về danh sách các kết quả khớp.

Hãy thực hiện lần lượt từng bước, bắt đầu với việc lặp qua các dòng.

#### Lặp qua các dòng với phương thức `lines`

Rust cung cấp một phương thức hữu ích để xử lý việc lặp qua chuỗi theo từng dòng, thuận tiện được đặt tên là `lines`, hoạt động như trong Liệt kê 12-17. Lưu ý rằng đoạn mã này hiện vẫn chưa biên dịch được.

<Listing number="12-17" file-name="src/lib.rs" caption="Lặp qua từng dòng trong `contents`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-17/src/lib.rs:here}}
```

</Listing>

Phương thức `lines` trả về một iterator. Chúng ta sẽ bàn chi tiết về iterator trong [Chương 13][ch13-iterators]<!-- ignore -->. Tuy nhiên, hãy nhớ rằng bạn đã thấy cách sử dụng iterator này trong [Liệt kê 3-5][ch3-iter]<!-- ignore -->, nơi chúng ta dùng vòng lặp `for` với một iterator để chạy một đoạn mã trên từng phần tử trong một tập hợp.

#### Tìm kiếm chuỗi truy vấn trong từng dòng

Tiếp theo, chúng ta sẽ kiểm tra xem dòng hiện tại có chứa chuỗi truy vấn hay không. May mắn là kiểu chuỗi có một phương thức hữu ích tên là `contains` thực hiện việc này cho chúng ta. Hãy thêm một lời gọi tới phương thức `contains` trong hàm `search`, như trong Liệt kê 12-18. Lưu ý rằng đoạn mã này vẫn chưa biên dịch được.

<Listing number="12-18" file-name="src/lib.rs" caption="Bổ sung chức năng để kiểm tra xem dòng có chứa chuỗi trong `query` hay không">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-18/src/lib.rs:here}}
```

</Listing>

Tại thời điểm này, chúng ta đang từng bước xây dựng chức năng. Để mã biên dịch được, chúng ta cần trả về một giá trị từ thân hàm như đã khai báo trong chữ ký hàm.

#### Lưu trữ các dòng khớp

Để hoàn thiện hàm này, chúng ta cần một cách lưu trữ các dòng khớp mà ta muốn trả về. Để làm điều đó, ta có thể tạo một vector có thể thay đổi (mutable) trước vòng lặp `for` và gọi phương thức `push` để lưu một `line` vào vector. Sau vòng lặp `for`, chúng ta trả về vector đó, như trong Liệt kê 12-19.

<Listing number="12-19" file-name="src/lib.rs" caption="Lưu trữ các dòng khớp để có thể trả về chúng">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:here}}
```

</Listing>

Bây giờ hàm `search` sẽ chỉ trả về các dòng có chứa `query`, và bài kiểm thử của chúng ta sẽ vượt qua. Hãy chạy bài kiểm thử:

```console
{{#include ../listings/ch12-an-io-project/listing-12-19/output.txt}}
```

Bài kiểm thử đã vượt qua, nên chúng ta biết rằng nó hoạt động đúng.

Tại thời điểm này, chúng ta có thể xem xét các cơ hội để tái cấu trúc (refactor) phần hiện thực của hàm tìm kiếm, đồng thời vẫn giữ cho các bài kiểm thử vượt qua để đảm bảo chức năng không thay đổi. Mã trong hàm `search` không quá tệ, nhưng chưa tận dụng được một số đặc tính hữu ích của iterator. Chúng ta sẽ quay lại ví dụ này trong [Chương 13][ch13-iterators]<!-- ignore -->, nơi ta sẽ khám phá iterator chi tiết hơn, và xem cách cải thiện nó.

Giờ thì toàn bộ chương trình phải hoạt động! Hãy thử với một từ mà kết quả phải trả về chính xác một dòng trong bài thơ của Emily Dickinson: _frog_.

```console
{{#include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/output.txt}}
```

Tốt! Bây giờ hãy thử một từ sẽ khớp với nhiều dòng, chẳng hạn _body_:

```console
{{#include ../listings/ch12-an-io-project/output-only-03-multiple-matches/output.txt}}
```

Và cuối cùng, hãy đảm bảo rằng chúng ta không nhận được dòng nào khi tìm kiếm một từ không hề xuất hiện trong bài thơ, chẳng hạn _monomorphization_:

```console
{{#include ../listings/ch12-an-io-project/output-only-04-no-matches/output.txt}}
```

Rất tốt! Chúng ta đã xây dựng được một phiên bản thu nhỏ của một công cụ kinh điển và học được rất nhiều về cách cấu trúc ứng dụng. Chúng ta cũng đã tìm hiểu đôi chút về nhập/xuất file, lifetime, testing và phân tích tham số dòng lệnh.

Để hoàn thiện dự án này, chúng ta sẽ nhanh chóng minh họa cách làm việc với biến môi trường (environment variables) và cách ghi ra chuẩn lỗi (standard error), cả hai đều hữu ích khi bạn viết các chương trình dòng lệnh.

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[ch11-anatomy]: ch11-01-writing-tests.html#the-anatomy-of-a-test-function
[ch10-lifetimes]: ch10-03-lifetime-syntax.html
[ch3-iter]: ch03-05-control-flow.html#looping-through-a-collection-with-for
[ch13-iterators]: ch13-02-iterators.html
