## Cải Thiện Dự Án I/O Của Chúng Ta

Với kiến thức mới về iterators, chúng ta có thể cải thiện dự án I/O trong
Chương 12 bằng cách sử dụng iterators để làm cho các phần mã code rõ ràng hơn và
ngắn gọn hơn. Hãy xem cách iterators có thể cải thiện cách triển khai hàm
`Config::build` và hàm `search` của chúng ta.

### Loại Bỏ `clone` Bằng Cách Sử Dụng Iterator

Trong Listing 12-6, chúng tôi đã thêm mã code lấy một slice của các giá trị
`String` và tạo một instance của struct `Config` bằng cách indexing vào slice
và cloning các giá trị, cho phép struct `Config` sở hữu những giá trị đó. Trong
Listing 13-17, chúng tôi đã tái tạo lại cách triển khai hàm `Config::build` như
nó đã từng ở Listing 12-23.

<Listing number="13-17" file-name="src/main.rs" caption="Tái tạo lại hàm `Config::build` từ Listing 12-23">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-23-reproduced/src/main.rs:ch13}}
```

</Listing>

Vào thời điểm đó, chúng tôi nói không lo lắng về các lệnh gọi `clone` không
hiệu quả vì chúng tôi sẽ loại bỏ chúng trong tương lai. Vâng, thời điểm đó đã
đến rồi!

Chúng ta cần `clone` ở đây vì chúng ta có một slice với các phần tử `String`
trong tham số `args`, nhưng hàm `build` không sở hữu `args`. Để trả lại quyền
sở hữu một instance `Config`, chúng ta phải clone các giá trị từ các trường
`query` và `file_path` của `Config` để instance `Config` có thể sở hữu các giá
trị của nó.

Với kiến thức mới của chúng ta về iterators, chúng ta có thể thay đổi hàm
`build` để nhận quyền sở hữu một iterator làm tham số của nó thay vì borrowing
một slice. Chúng tôi sẽ sử dụng chức năng iterator thay vì mã code kiểm tra độ
dài của slice và indexing vào các vị trí cụ thể. Điều này sẽ làm rõ ràng hơn
những gì hàm `Config::build` đang làm vì iterator sẽ truy cập các giá trị.

Khi `Config::build` nhận quyền sở hữu của iterator và ngừng sử dụng các
operation indexing mà borrow, chúng ta có thể di chuyển các giá trị `String`
từ iterator vào `Config` thay vì gọi `clone` và tạo một allocation mới.

#### Sử Dụng Iterator Được Trả Về Trực Tiếp

Mở file _src/main.rs_ của dự án I/O của bạn, file này sẽ trông như thế này:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-24-reproduced/src/main.rs:ch13}}
```

Đầu tiên, chúng ta sẽ thay đổi phần bắt đầu của hàm `main` mà chúng ta có ở
Listing 12-24 thành mã code trong Listing 13-18, lần này sử dụng một iterator.
Điều này sẽ không compile cho đến khi chúng ta cũng cập nhật `Config::build`.

<Listing number="13-18" file-name="src/main.rs" caption="Truyền giá trị được trả về của `env::args` tới `Config::build`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-18/src/main.rs:here}}
```

</Listing>

Hàm `env::args` trả về một iterator! Thay vì collect các giá trị của iterator
vào một vector và sau đó truyền một slice tới `Config::build`, giờ đây chúng ta
truyền quyền sở hữu của iterator được trả về từ `env::args` tới `Config::build`
trực tiếp.

Tiếp theo, chúng ta cần cập nhật định nghĩa của `Config::build`. Hãy thay đổi
signature của `Config::build` để trông giống như Listing 13-19. Điều này vẫn sẽ
không compile, vì chúng ta cần cập nhật body của hàm.

<Listing number="13-19" file-name="src/main.rs" caption="Cập nhật signature của `Config::build` để mong đợi một iterator">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-19/src/main.rs:here}}
```

</Listing>

Tài liệu của thư viện tiêu chuẩn cho hàm `env::args` cho thấy rằng kiểu của
iterator mà nó trả về là `std::env::Args`, và kiểu đó triển khai trait `Iterator`
và trả về các giá trị `String`.

Chúng tôi đã cập nhật signature của hàm `Config::build` sao cho tham số `args`
có kiểu generic với trait bounds `impl Iterator<Item = String>` thay vì
`&[String]`. Cách sử dụng syntax `impl Trait` này mà chúng ta đã thảo luận trong
phần ["Sử dụng Traits làm Tham Số"][impl-trait]<!-- ignore --> của Chương 10
có nghĩa là `args` có thể là bất kỳ kiểu nào triển khai trait `Iterator` và trả
về các item `String`.

Vì chúng ta đang nhận quyền sở hữu của `args` và chúng ta sẽ mutate `args` bằng
cách iterating trên nó, chúng ta có thể thêm keyword `mut` vào specification
của tham số `args` để làm nó mutable.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-iterator-trait-methods-instead-of-indexing"></a>

#### Sử Dụng Các Phương Thức Của Trait `Iterator`

Tiếp theo, chúng ta sẽ sửa body của `Config::build`. Vì `args` triển khai trait
`Iterator`, chúng ta biết rằng chúng ta có thể gọi phương thức `next` trên nó!
Listing 13-20 cập nhật mã code từ Listing 12-23 để sử dụng phương thức `next`.

<Listing number="13-20" file-name="src/main.rs" caption="Thay đổi body của `Config::build` để sử dụng iterator methods">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-20/src/main.rs:here}}
```

</Listing>

Hãy nhớ rằng giá trị đầu tiên trong giá trị được trả về của `env::args` là tên
của chương trình. Chúng ta muốn bỏ qua điều đó và đi tới giá trị tiếp theo, vì
vậy trước tiên chúng ta gọi `next` và không làm gì với giá trị được trả về. Sau
đó, chúng ta gọi `next` để lấy giá trị mà chúng ta muốn đặt vào trường `query`
của `Config`. Nếu `next` trả về `Some`, chúng ta sử dụng `match` để extract giá
trị. Nếu nó trả về `None`, có nghĩa là không đủ arguments được cung cấp, và chúng
ta return sớm với một giá trị `Err`. Chúng ta làm điều tương tự cho giá trị
`file_path`.

<!-- Old headings. Do not remove or links may break. -->

<a id="making-code-clearer-with-iterator-adapters"></a>

### Làm Rõ Ràng Code Bằng Iterator Adapters

Chúng ta cũng có thể tận dụng iterators trong hàm `search` trong dự án I/O của
chúng ta, được tái tạo ở đây trong Listing 13-21 như nó đã ở Listing 12-19.

<Listing number="13-21" file-name="src/lib.rs" caption="Cách triển khai hàm `search` từ Listing 12-19">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:ch13}}
```

</Listing>

Chúng ta có thể viết mã code này theo cách ngắn gọn hơn bằng cách sử dụng các
phương thức iterator adapter. Làm như vậy cũng cho phép chúng ta tránh có một
vector `results` intermediate mutable. Phong cách lập trình hàm thích minimize
lượng mutable state để làm cho code rõ ràng hơn. Loại bỏ mutable state có thể
cho phép một enhancement trong tương lai để làm cho việc searching xảy ra song
song vì chúng ta sẽ không phải quản lý concurrent access tới vector `results`.
Listing 13-22 cho thấy sự thay đổi này.

<Listing number="13-22" file-name="src/lib.rs" caption="Sử dụng iterator adapter methods trong cách triển khai hàm `search`">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-22/src/lib.rs:here}}
```

</Listing>

Hãy nhớ lại rằng mục đích của hàm `search` là trả về tất cả các dòng trong
`contents` chứa `query`. Tương tự như ví dụ `filter` trong Listing 13-16, mã
code này sử dụng adapter `filter` để giữ lại chỉ các dòng mà `line.contains(query)`
trả về `true`. Sau đó, chúng ta collect các dòng khớp vào một vector khác bằng
`collect`. Đơn giản hơn nhiều! Hãy thoải mái thực hiện cùng một thay đổi để sử
dụng iterator methods trong hàm `search_case_insensitive` cũng vậy.

Để cải thiện thêm, hãy return một iterator từ hàm `search` bằng cách loại bỏ lệnh
gọi `collect` và thay đổi return type thành `impl Iterator<Item = &'a str>` để
hàm trở thành một iterator adapter. Lưu ý rằng bạn cũng sẽ cần phải cập nhật
các tests! Tìm kiếm qua một file lớn bằng cách sử dụng tool `minigrep` của bạn
trước và sau khi thực hiện thay đổi này để quan sát sự khác biệt trong hành vi.
Trước thay đổi này, chương trình sẽ không in bất kỳ kết quả nào cho đến khi nó
đã collect tất cả các kết quả, nhưng sau thay đổi, các kết quả sẽ được in khi
mỗi dòng khớp được tìm thấy vì vòng lặp `for` trong hàm `run` có khả năng tận
dụng tính biếng của iterator.

<!-- Old headings. Do not remove or links may break. -->

<a id="choosing-between-loops-or-iterators"></a>

### Chọn Giữa Loops và Iterators

Câu hỏi logic tiếp theo là bạn nên chọn phong cách nào trong mã code của riêng
bạn và tại sao: cách triển khai ban đầu trong Listing 13-21 hoặc phiên bản sử
dụng iterators trong Listing 13-22 (giả sử chúng ta đang collect tất cả các kết
quả trước khi trả về chúng thay vì return iterator). Hầu hết các lập trình viên
Rust thích sử dụng phong cách iterator. Nó khó nắm bắt hơn một chút lúc đầu,
nhưng khi bạn cảm nhận được các iterator adapters khác nhau và những gì chúng làm,
iterators có thể dễ hiểu hơn. Thay vì điều chỉnh các bits khác nhau của looping
và building vectors mới, code tập trung vào mục tiêu cấp cao của vòng lặp. Điều
này tóm tắt một số code phổ biến để dễ dàng hơn để thấy các khái niệm độc đáo
cho code này, chẳng hạn như điều kiện filtering mà mỗi phần tử trong iterator
phải đáp ứng.

Nhưng hai cách triển khai này có thực sự tương đương không? Giả định trực quan
có thể là vòng lặp cấp thấp hơn sẽ nhanh hơn. Hãy nói về performance.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
