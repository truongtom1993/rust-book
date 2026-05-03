## Làm việc với Environment Variables

Chúng ta sẽ cải thiện binary `minigrep` bằng cách thêm một tính năng bổ sung: một tùy chọn cho tìm kiếm không phân biệt hoa thường mà người dùng có thể bật thông qua environment variable. Chúng ta có thể làm cho tính năng này thành một tùy chọn command line và yêu cầu người dùng nhập nó mỗi lần họ muốn áp dụng, nhưng thay vào đó bằng cách biến nó thành một environment variable, chúng ta cho phép người dùng thiết lập environment variable một lần và tất cả các tìm kiếm của họ sẽ không phân biệt hoa thường trong phiên terminal đó.

<!-- Old headings. Do not remove or links may break. -->
<a id="writing-a-failing-test-for-the-case-insensitive-search-function"></a>

### Viết một Test Thất bại cho Case-Insensitive Search

Đầu tiên chúng ta thêm một function mới `search_case_insensitive` vào library `minigrep` sẽ được gọi khi environment variable có giá trị. Chúng ta sẽ tiếp tục theo quy trình TDD, vì vậy bước đầu tiên lại là viết một test thất bại. Chúng ta sẽ thêm một test mới cho function `search_case_insensitive` mới và đổi tên test cũ của chúng ta từ `one_result` thành `case_sensitive` để làm rõ sự khác biệt giữa hai test, như được hiển thị trong Listing 12-20.

<Listing number="12-20" file-name="src/lib.rs" caption="Thêm một test thất bại mới cho function case-insensitive mà chúng ta sắp thêm">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-20/src/lib.rs:here}}
```

</Listing>

Lưu ý rằng chúng ta cũng đã chỉnh sửa `contents` của test cũ. Chúng ta đã thêm một dòng mới với văn bản `"Duct tape."` sử dụng chữ _D_ viết hoa không nên khớp với query `"duct"` khi chúng ta tìm kiếm theo cách phân biệt hoa thường. Thay đổi test cũ theo cách này giúp đảm bảo rằng chúng ta không vô tình phá vỡ chức năng tìm kiếm phân biệt hoa thường mà chúng ta đã implement. Test này sẽ pass ngay bây giờ và sẽ tiếp tục pass khi chúng ta làm việc trên tìm kiếm không phân biệt hoa thường.

Test mới cho tìm kiếm không phân biệt hoa thường sử dụng `"rUsT"` làm query của nó. Trong function `search_case_insensitive` mà chúng ta sắp thêm, query `"rUsT"` sẽ khớp với dòng chứa `"Rust:"` với chữ _R_ viết hoa và khớp với dòng `"Trust me."` mặc dù cả hai đều có cách viết hoa khác với query. Đây là test thất bại của chúng ta, và nó sẽ thất bại khi compile vì chúng ta chưa định nghĩa function `search_case_insensitive`. Bạn có thể thoải mái thêm một skeleton implementation luôn trả về một vector rỗng, tương tự như cách chúng ta đã làm cho function `search` trong Listing 12-16 để thấy test compile và fail.

### Implementing function `search_case_insensitive`

Function `search_case_insensitive`, được hiển thị trong Listing 12-21, sẽ gần giống với function `search`. Sự khác biệt duy nhất là chúng ta sẽ chuyển `query` và mỗi `line` thành chữ thường để cho dù trường hợp nào của các input argument, chúng sẽ có cùng kiểu chữ khi chúng ta kiểm tra xem dòng có chứa query hay không.

<Listing number="12-21" file-name="src/lib.rs" caption="Định nghĩa function `search_case_insensitive` để chuyển query và line thành chữ thường trước khi so sánh chúng">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-21/src/lib.rs:here}}
```

</Listing>

Đầu tiên, chúng ta chuyển chuỗi `query` thành chữ thường và lưu trữ nó trong một variable mới với cùng tên, shadowing `query` ban đầu. Gọi `to_lowercase` trên query là cần thiết để cho dù query của người dùng là `"rust"`, `"RUST"`, `"Rust"`, hay `"rUsT"`, chúng ta sẽ xử lý query như thể nó là `"rust"` và không phân biệt hoa thường. Mặc dù `to_lowercase` sẽ xử lý Unicode cơ bản, nó sẽ không chính xác 100 phần trăm. Nếu chúng ta đang viết một ứng dụng thực tế, chúng ta muốn làm thêm một chút công việc ở đây, nhưng phần này là về environment variables, không phải Unicode, vì vậy chúng ta sẽ để nó như vậy ở đây.

Lưu ý rằng `query` bây giờ là một `String` thay vì một string slice vì gọi `to_lowercase` tạo ra dữ liệu mới thay vì tham chiếu dữ liệu hiện có. Giả sử query là `"rUsT"`, ví dụ: String slice đó không chứa chữ thường `u` hoặc `t` để chúng ta sử dụng, vì vậy chúng ta phải cấp phát một `String` mới chứa `"rust"`. Khi chúng ta truyền `query` làm argument cho method `contains` bây giờ, chúng ta cần thêm một ampersand vì signature của `contains` được định nghĩa để nhận một string slice.

Tiếp theo, chúng ta thêm một lời gọi đến `to_lowercase` trên mỗi `line` để chuyển tất cả các ký tự thành chữ thường. Bây giờ chúng ta đã chuyển đổi `line` và `query` thành chữ thường, chúng ta sẽ tìm thấy các kết quả khớp bất kể trường hợp nào của query.

Hãy xem implementation này có pass các test không:

```console
{{#include ../listings/ch12-an-io-project/listing-12-21/output.txt}}
```

Tuyệt vời! Chúng đã pass. Bây giờ hãy gọi function `search_case_insensitive` mới từ function `run`. Đầu tiên, chúng ta sẽ thêm một configuration option vào struct `Config` để chuyển đổi giữa tìm kiếm phân biệt hoa thường và không phân biệt hoa thường. Thêm field này sẽ gây ra compiler errors vì chúng ta chưa khởi tạo field này ở bất cứ đâu:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:here}}
```

Chúng ta đã thêm field `ignore_case` chứa một Boolean. Tiếp theo, chúng ta cần function `run` kiểm tra giá trị của field `ignore_case` và sử dụng nó để quyết định gọi function `search` hay function `search_case_insensitive`, như được hiển thị trong Listing 12-22. Điều này vẫn chưa compile được.

<Listing number="12-22" file-name="src/main.rs" caption="Gọi `search` hoặc `search_case_insensitive` dựa trên giá trị trong `config.ignore_case`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:there}}
```

</Listing>

Cuối cùng, chúng ta cần kiểm tra environment variable. Các function để làm việc với environment variables nằm trong module `env` trong standard library, đã có trong scope ở đầu _src/main.rs_. Chúng ta sẽ sử dụng function `var` từ module `env` để kiểm tra xem có giá trị nào được thiết lập cho một environment variable có tên `IGNORE_CASE` hay không, như được hiển thị trong Listing 12-23.

<Listing number="12-23" file-name="src/main.rs" caption="Kiểm tra bất kỳ giá trị nào trong environment variable có tên `IGNORE_CASE`">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-23/src/main.rs:here}}
```

</Listing>

Ở đây, chúng ta tạo một variable mới, `ignore_case`. Để thiết lập giá trị của nó, chúng ta gọi function `env::var` và truyền cho nó tên của environment variable `IGNORE_CASE`. Function `env::var` trả về một `Result` sẽ là variant `Ok` thành công chứa giá trị của environment variable nếu environment variable được thiết lập thành bất kỳ giá trị nào. Nó sẽ trả về variant `Err` nếu environment variable không được thiết lập.

Chúng ta đang sử dụng method `is_ok` trên `Result` để kiểm tra xem environment variable có được thiết lập hay không, điều đó có nghĩa là chương trình nên thực hiện tìm kiếm không phân biệt hoa thường. Nếu environment variable `IGNORE_CASE` không được thiết lập thành bất cứ thứ gì, `is_ok` sẽ trả về `false` và chương trình sẽ thực hiện tìm kiếm phân biệt hoa thường. Chúng ta không quan tâm đến _giá trị_ của environment variable, chỉ quan tâm xem nó có được thiết lập hay không, vì vậy chúng ta đang kiểm tra `is_ok` thay vì sử dụng `unwrap`, `expect`, hoặc bất kỳ method nào khác mà chúng ta đã thấy trên `Result`.

Chúng ta truyền giá trị trong variable `ignore_case` cho instance `Config` để function `run` có thể đọc giá trị đó và quyết định gọi `search_case_insensitive` hay `search`, như chúng ta đã implement trong Listing 12-22.

Hãy thử nó! Đầu tiên, chúng ta sẽ chạy chương trình của mình mà không thiết lập environment variable và với query `to`, sẽ khớp với bất kỳ dòng nào chứa từ _to_ viết thường:

```console
{{#include ../listings/ch12-an-io-project/listing-12-23/output.txt}}
```

Có vẻ như vẫn hoạt động! Bây giờ hãy chạy chương trình với `IGNORE_CASE` được thiết lập thành `1` nhưng với cùng query `to`:

```console
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

Nếu bạn đang sử dụng PowerShell, bạn sẽ cần thiết lập environment variable và chạy chương trình như các lệnh riêng biệt:

```console
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

Điều này sẽ làm cho `IGNORE_CASE` tồn tại trong phần còn lại của phiên shell của bạn. Nó có thể được unset bằng cmdlet `Remove-Item`:

```console
PS> Remove-Item Env:IGNORE_CASE
```

Chúng ta sẽ nhận được các dòng chứa _to_ có thể có chữ viết hoa:

<!-- manual-regeneration
cd listings/ch12-an-io-project/listing-12-23
IGNORE_CASE=1 cargo run -- to poem.txt
can't extract because of the environment variable
-->

```console
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

Tuyệt vời, chúng ta cũng nhận được các dòng chứa _To_! Chương trình `minigrep` của chúng ta bây giờ có thể thực hiện tìm kiếm không phân biệt hoa thường được kiểm soát bởi một environment variable. Bây giờ bạn biết cách quản lý các tùy chọn được thiết lập bằng cách sử dụng command line arguments hoặc environment variables.

Một số chương trình cho phép arguments _và_ environment variables cho cùng một configuration. Trong những trường hợp đó, các chương trình quyết định rằng một hoặc cái kia được ưu tiên. Đối với một bài tập khác cho riêng bạn, hãy thử kiểm soát case sensitivity thông qua command line argument hoặc environment variable. Quyết định xem command line argument hay environment variable nên được ưu tiên nếu chương trình được chạy với một cái được thiết lập thành case sensitive và một cái được thiết lập thành ignore case.

Module `std::env` chứa nhiều tính năng hữu ích hơn để xử lý environment variables: Kiểm tra documentation của nó để xem những gì có sẵn.