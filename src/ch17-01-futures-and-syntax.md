## Futures và Cú pháp Async

Các yếu tố chính của lập trình không đồng bộ trong Rust là _futures_ và các từ khóa `async` và `await` của Rust.

Một _future_ là một giá trị có thể chưa sẵn sàng bây giờ nhưng sẽ sẵn sàng tại một thời điểm nào đó trong tương lai. (Khái niệm tương tự này xuất hiện trong nhiều ngôn ngữ, đôi khi dưới những tên khác như _task_ hoặc _promise_.) Rust cung cấp một trait `Future` như một khối xây dựng để các hoạt động async khác nhau có thể được triển khai với các cấu trúc dữ liệu khác nhau nhưng với một giao diện chung. Trong Rust, futures là các kiểu dữ liệu triển khai trait `Future`. Mỗi future giữ thông tin của riêng nó về tiến trình đã được thực hiện và ý nghĩa của "sẵn sàng".

Bạn có thể áp dụng từ khóa `async` cho các block và function để chỉ định rằng chúng có thể bị gián đoạn và tiếp tục. Trong một async block hoặc async function, bạn có thể sử dụng từ khóa `await` để _await một future_ (tức là chờ nó trở thành sẵn sàng). Bất kỳ điểm nào bạn await một future trong một async block hoặc function là một vị trí tiềm năng cho block hoặc function đó để tạm dừng và tiếp tục. Quá trình kiểm tra với một future để xem giá trị của nó có sẵn sàng hay không được gọi là _polling_.

Một số ngôn ngữ khác, chẳng hạn như C# và JavaScript, cũng sử dụng các từ khóa `async` và `await` cho lập trình async. Nếu bạn quen thuộc với những ngôn ngữ đó, bạn có thể nhận thấy một số khác biệt đáng kể về cách Rust xử lý cú pháp. Có lý do chính đáng cho điều này, như chúng ta sẽ thấy!

Khi viết async Rust, chúng ta sử dụng các từ khóa `async` và `await` hầu hết thời gian. Rust biên dịch chúng thành mã tương đương sử dụng trait `Future`, giống như cách nó biên dịch các vòng lặp `for` thành mã tương đương sử dụng trait `Iterator`. Tuy nhiên, vì Rust cung cấp trait `Future`, bạn cũng có thể triển khai nó cho các kiểu dữ liệu của riêng bạn khi cần. Nhiều hàm mà chúng ta sẽ thấy trong suốt chương này trả về các kiểu có triển khai riêng của `Future`. Chúng ta sẽ quay trở lại định nghĩa của trait ở cuối chương và đi sâu hơn vào cách nó hoạt động, nhưng đây là đủ chi tiết để chúng ta tiếp tục.

Tất cả điều này có thể cảm thấy hơi trừu tượng, vì vậy hãy viết chương trình async đầu tiên của chúng ta: một web scraper nhỏ. Chúng ta sẽ truyền vào hai URL từ dòng lệnh, tìm nạp cả hai đồng thời, và trả về kết quả của bất kỳ cái nào kết thúc trước. Ví dụ này sẽ có khá nhiều cú pháp mới, nhưng đừng lo—chúng ta sẽ giải thích mọi thứ bạn cần biết khi chúng ta tiếp tục.

## Chương Trình Async Đầu Tiên Của Chúng Ta

Để giữ trọng tâm của chương này vào việc học async thay vì xử lý các phần của hệ sinh thái, chúng ta đã tạo crate `trpl` (viết tắt của "The Rust Programming Language"). Nó tái xuất tất cả các kiểu, trait và function mà bạn cần, chủ yếu từ các crate [`futures`][futures-crate]<!-- ignore --> và [`tokio`][tokio]<!-- ignore -->. Crate `futures` là một nhà cung cấp chính thức cho thử nghiệm Rust cho mã async, và nó thực sự là nơi trait `Future` được thiết kế ban đầu. Tokio là async runtime được sử dụng rộng rãi nhất trong Rust ngày nay, đặc biệt là cho các ứng dụng web. Có những runtime tuyệt vời khác, và chúng có thể phù hợp hơn với mục đích của bạn. Chúng ta sử dụng crate `tokio` dưới cùng cho `trpl` vì nó được kiểm tra kỹ lưỡng và được sử dụng rộng rãi.

Trong một số trường hợp, `trpl` cũng đổi tên hoặc bao bọc các API gốc để bạn tập trung vào các chi tiết liên quan đến chương này. Nếu bạn muốn hiểu crate làm gì, chúng tôi khuyến khích bạn kiểm tra [mã nguồn của nó][crate-source]. Bạn sẽ có thể thấy crate nào mỗi tái xuất đến, và chúng tôi đã để lại các nhận xét mở rộng giải thích crate làm gì.

Tạo một dự án binary mới có tên `hello-async` và thêm crate `trpl` làm dependency:

```console
$ cargo new hello-async
$ cd hello-async
$ cargo add trpl
```

Bây giờ chúng ta có thể sử dụng các phần khác nhau được cung cấp bởi `trpl` để viết chương trình async đầu tiên của chúng ta. Chúng ta sẽ xây dựng một công cụ dòng lệnh nhỏ để tìm nạp hai trang web, lấy phần tử `<title>` từ mỗi trang, và in ra tiêu đề của trang hoàn tất quá trình đó trước tiên.

### Định Nghĩa Function page_title

Hãy bắt đầu bằng cách viết một function nhận một URL trang làm parameter, thực hiện một yêu cầu đến nó, và trả về văn bản của phần tử `<title>` (xem Listing 17-1).

<Listing number="17-1" file-name="src/main.rs" caption="Định nghĩa một async function để lấy phần tử title từ một trang HTML">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-01/src/main.rs:all}}
```

</Listing>

Đầu tiên, chúng ta định nghĩa một function được gọi là `page_title` và đánh dấu nó bằng từ khóa `async`. Sau đó chúng ta sử dụng function `trpl::get` để tìm nạp bất kỳ URL nào được truyền vào và thêm từ khóa `await` để await response. Để có được văn bản của `response`, chúng ta gọi phương thức `text` của nó và một lần nữa await nó bằng từ khóa `await`. Cả hai bước này đều không đồng bộ. Đối với function `get`, chúng ta phải chờ máy chủ gửi lại phần đầu tiên của response của nó, sẽ bao gồm HTTP headers, cookies, v.v. và có thể được gửi riêng biệt từ phần body của response. Đặc biệt nếu body rất lớn, có thể mất một thời gian để tất cả đến. Vì chúng ta phải chờ _toàn bộ_ response đến, phương thức `text` cũng không đồng bộ.

Chúng ta phải explicit await cả hai futures này, vì futures trong Rust là _lazy_: chúng không làm gì cho đến khi bạn yêu cầu chúng bằng từ khóa `await`. (Trên thực tế, Rust sẽ hiển thị một cảnh báo trình biên dịch nếu bạn không sử dụng một future.) Điều này có thể nhắc bạn về cuộc thảo luận về iterators trong phần ["Processing a Series of Items with Iterators"][iterators-lazy]<!-- ignore --> ở Chương 13. Iterators không làm gì trừ khi bạn gọi phương thức `next` của chúng—dù trực tiếp hay bằng cách sử dụng các vòng lặp `for` hoặc các phương thức như `map` sử dụng `next` dưới cùng. Tương tự như vậy, futures không làm gì trừ khi bạn rõ ràng yêu cầu chúng. Sự lazy này cho phép Rust tránh chạy mã async cho đến khi nó thực sự cần thiết.

> Ghi chú: Điều này khác với hành vi chúng ta thấy khi sử dụng `thread::spawn` trong phần ["Creating a New Thread with spawn"][thread-spawn]<!-- ignore --> ở Chương 16, nơi closure chúng ta truyền đến một thread khác bắt đầu chạy ngay lập tức. Nó cũng khác với cách nhiều ngôn ngữ khác tiếp cận async. Nhưng điều quan trọng là Rust phải có thể cung cấp các bảo đảm về hiệu suất của nó, giống như với iterators.

Khi chúng ta có `response_text`, chúng ta có thể phân tích cú pháp nó thành một instance của kiểu `Html` bằng cách sử dụng `Html::parse`. Thay vì một chuỗi thô, chúng ta bây giờ có một kiểu dữ liệu mà chúng ta có thể sử dụng để làm việc với HTML như một cấu trúc dữ liệu phong phú hơn. Đặc biệt, chúng ta có thể sử dụng phương thức `select_first` để tìm instance đầu tiên của một bộ chọn CSS nhất định. Bằng cách truyền chuỗi `"title"`, chúng ta sẽ nhận được phần tử `<title>` đầu tiên trong tài liệu, nếu có. Vì có thể không có phần tử nào phù hợp, `select_first` trả về một `Option<ElementRef>`. Cuối cùng, chúng ta sử dụng phương thức `Option::map`, cho phép chúng ta làm việc với item trong `Option` nếu nó có, và không làm gì nếu nó không. (Chúng ta cũng có thể sử dụng một biểu thức `match` ở đây, nhưng `map` là idiomatic hơn.) Trong phần thân của function chúng ta cung cấp cho `map`, chúng ta gọi `inner_html` trên `title` để lấy nội dung của nó, đó là một `String`. Khi tất cả xong, chúng ta có một `Option<String>`.

Lưu ý rằng từ khóa `await` của Rust đi _sau_ biểu thức bạn đang await, không phải trước nó. Tức là, nó là một từ khóa _postfix_. Điều này có thể khác với những gì bạn quen thuộc nếu bạn đã sử dụng `async` trong các ngôn ngữ khác, nhưng trong Rust nó làm cho các chuỗi phương thức dễ làm việc hơn nhiều. Kết quả là, chúng ta có thể thay đổi phần thân của `page_title` để chuỗi các lệnh gọi function `trpl::get` và `text` với `await` giữa chúng, như thể hiện trong Listing 17-2.

<Listing number="17-2" file-name="src/main.rs" caption="Chaining với từ khóa `await`">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-02/src/main.rs:chaining}}
```

</Listing>

Với điều đó, chúng ta đã successfully viết function async đầu tiên của chúng ta! Trước khi thêm một số mã trong `main` để gọi nó, hãy nói thêm một chút về những gì chúng ta đã viết và ý nghĩa của nó.

Khi Rust thấy một _block_ được đánh dấu bằng từ khóa `async`, nó biên dịch nó thành một kiểu dữ liệu ẩn danh duy nhất triển khai trait `Future`. Khi Rust thấy một _function_ được đánh dấu bằng `async`, nó biên dịch nó thành một non-async function có phần thân là một async block. Kiểu trả về của async function là kiểu dữ liệu ẩn danh mà trình biên dịch tạo cho async block đó.

Do đó, viết `async fn` tương đương với viết một function trả về một _future_ của kiểu trả về. Đối với trình biên dịch, một định nghĩa function như `async fn page_title` trong Listing 17-1 được nội bộ tương đương với một non-async function được định nghĩa như thế này:

```rust
# extern crate trpl; // required for mdbook test
use std::future::Future;
use trpl::Html;

fn page_title(url: &str) -> impl Future<Output = Option<String>> {
    async move {
        let text = trpl::get(url).await.text().await;
        Html::parse(&text)
            .select_first("title")
            .map(|title| title.inner_html())
    }
}
```

Hãy xem qua từng phần của phiên bản được biến đổi:

- Nó sử dụng cú pháp `impl Trait` mà chúng ta đã thảo luận ở Chương 10 trong phần ["Traits as Parameters"][impl-trait]<!-- ignore -->.
- Giá trị trả về triển khai trait `Future` với một kiểu liên kết là `Output`. Lưu ý rằng kiểu `Output` là `Option<String>`, giống như kiểu trả về ban đầu từ phiên bản `async fn` của `page_title`.
- Tất cả mã được gọi trong phần thân của function ban đầu được bao bọc trong một block `async move`. Hãy nhớ rằng blocks là expressions. Toàn bộ block này là expression được trả về từ function.
- Async block này tạo ra một giá trị có kiểu `Option<String>`, như vừa được mô tả. Giá trị đó phù hợp với kiểu `Output` trong kiểu trả về. Điều này giống như các blocks khác mà bạn đã thấy.
- Phần thân function mới là một block `async move` vì cách nó sử dụng parameter `url`. (Chúng ta sẽ nói nhiều hơn về `async` so với `async move` sau trong chương.)

Bây giờ chúng ta có thể gọi `page_title` trong `main`.

<!-- Old headings. Do not remove or links may break. -->

<a id ="determining-a-single-pages-title"></a>

### Thực Hiện Một Async Function Với Một Runtime

Để bắt đầu, chúng ta sẽ lấy tiêu đề cho một single page, được thể hiện trong Listing 17-3. Thật không may, mã này chưa biên dịch.

<Listing number="17-3" file-name="src/main.rs" caption="Gọi function `page_title` từ `main` với một user-supplied argument">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-03/src/main.rs:main}}
```

</Listing>

Chúng ta theo dõi cùng một pattern chúng ta đã sử dụng để lấy command line arguments trong phần ["Accepting Command Line Arguments"][cli-args]<!-- ignore --> ở Chương 12. Sau đó chúng ta truyền URL argument đến `page_title` và await kết quả. Vì giá trị được tạo ra bởi future là một `Option<String>`, chúng ta sử dụng một biểu thức `match` để in các thông báo khác nhau để tính đến việc liệu trang có `<title>` hay không.

Địa điểm duy nhất chúng ta có thể sử dụng từ khóa `await` là trong các async functions hoặc blocks, và Rust sẽ không cho phép chúng ta đánh dấu special function `main` làm `async`.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-03
cargo build
copy just the compiler error
-->

```text
error[E0752]: `main` function is not allowed to be `async`
 --> src/main.rs:6:1
  |
6 | async fn main() {
  | ^^^^^^^^^^^^^^^ `main` function is not allowed to be `async`
```

Lý do `main` không thể được đánh dấu `async` là mã async cần một _runtime_: một Rust crate quản lý các chi tiết của việc thực hiện mã không đồng bộ. Một function `main` của chương trình có thể _khởi tạo_ một runtime, nhưng nó không phải là một runtime _itself_. (Chúng ta sẽ thấy thêm về lý do tại sao điều này lại là như vậy trong chút nữa.) Mỗi chương trình Rust thực hiện mã async có ít nhất một vị trí mà nó thiết lập một runtime thực hiện futures.

Hầu hết các ngôn ngữ hỗ trợ async đi kèm với một runtime, nhưng Rust thì không. Thay vào đó, có nhiều async runtimes khác nhau có sẵn, mỗi cách tạo ra những tradeoff khác nhau phù hợp với trường hợp sử dụng nó nhắm đến. Ví dụ, một máy chủ web có throughput cao với nhiều CPU cores và một lượng RAM lớn có những nhu cầu rất khác nhau so với một microcontroller với một single core, một lượng RAM nhỏ, và không có khả năng heap allocation. Các crates cung cấp những runtimes đó cũng thường cung cấp các phiên bản async của chức năng phổ biến như file hoặc network I/O.

Ở đây, và trong phần còn lại của chương này, chúng ta sẽ sử dụng function `block_on` từ crate `trpl`, nhận một future làm argument và chặn thread hiện tại cho đến khi future này chạy hoàn thành. Đằng sau cảnh, gọi `block_on` thiết lập một runtime sử dụng crate `tokio` được sử dụng để chạy future được truyền vào (hành vi `block_on` của crate `trpl` tương tự như các function `block_on` của các crates runtime khác). Khi future hoàn thành, `block_on` trả về bất kỳ giá trị nào mà future tạo ra.

Chúng ta có thể truyền future được trả về bởi `page_title` trực tiếp đến `block_on` và, khi nó hoàn thành, chúng ta có thể match trên `Option<String>` kết quả như chúng ta cố gắng làm trong Listing 17-3. Tuy nhiên, đối với hầu hết các ví dụ trong chương (và hầu hết mã async trong thế giới thực), chúng ta sẽ làm nhiều hơn chỉ một lệnh gọi async function, vì vậy thay vào đó chúng ta sẽ truyền một block `async` và rõ ràng await kết quả của lệnh gọi `page_title`, như trong Listing 17-4.

<Listing number="17-4" caption="Awaiting một async block với `trpl::block_on`" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook test does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-04/src/main.rs:run}}
```

</Listing>

Khi chúng ta chạy mã này, chúng ta nhận được hành vi mà chúng ta dự kiến ban đầu:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-04
cargo build # skip all the build noise
cargo run -- "https://www.rust-lang.org"
# copy the output here
-->

```console
$ cargo run -- "https://www.rust-lang.org"
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/async_await 'https://www.rust-lang.org'`
The title for https://www.rust-lang.org was
            Rust Programming Language
```

Phew—chúng ta cuối cùng đã có một số mã async hoạt động! Nhưng trước khi thêm mã để race hai sites lại với nhau, hãy tạm thời quay lại chú ý của chúng ta vào cách hoạt động của futures.

Mỗi _await point_—tức là, mỗi vị trí mà mã sử dụng từ khóa `await`—đại diện cho một nơi mà điều khiển được chuyển lại cho runtime. Để làm điều đó hoạt động, Rust cần theo dõi trạng thái liên quan trong async block để runtime có thể thực hiện một số công việc khác và sau đó quay lại khi nó sẵn sàng cố gắng tiến bộ cái đầu tiên một lần nữa. Đây là một state machine vô hình, như thể bạn đã viết một enum như thế này để lưu trạng thái hiện tại ở mỗi await point:

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-state-machine/src/lib.rs:enum}}
```

Viết mã để chuyển tiếp giữa mỗi trạng thái bằng tay sẽ là tedious và error-prone, tuy nhiên, đặc biệt khi bạn cần thêm nhiều chức năng hơn và nhiều trạng thái hơn cho mã sau này. May mắn thay, trình biên dịch Rust tạo và quản lý các cấu trúc dữ liệu state machine cho mã async tự động. Các quy tắc normal borrowing và ownership xung quanh các cấu trúc dữ liệu vẫn áp dụng, và may mắn thay, trình biên dịch cũng xử lý việc kiểm tra các quy tắc đó cho chúng ta và cung cấp các thông báo lỗi hữu ích. Chúng ta sẽ làm việc thông qua một số trong số đó sau trong chương.

Cuối cùng, cái gì đó phải thực hiện state machine này, và cái gì đó đó là một runtime. (Đây là lý do tại sao bạn có thể gặp các thông báo về _executors_ khi tìm hiểu về runtimes: một executor là phần của runtime chịu trách nhiệm thực hiện mã async.)

Bây giờ bạn có thể thấy tại sao trình biên dịch đã dừng chúng ta khỏi tạo `main` thành một async function ở Listing 17-3. Nếu `main` là một async function, cái gì khác sẽ cần quản lý state machine cho bất kỳ future nào `main` trả về, nhưng `main` là điểm khởi đầu cho chương trình! Thay vào đó, chúng ta gọi function `trpl::block_on` trong `main` để thiết lập một runtime và chạy future được trả về bởi block `async` cho đến khi nó xong.

> Ghi chú: Một số runtimes cung cấp macros để bạn _can_ viết một async function `main`. Những macros đó viết lại `async fn main() { ... }` thành một normal `fn main`, làm điều tương tự như chúng ta đã làm bằng tay trong Listing 17-4: gọi một function chạy một future hoàn thành theo cách `trpl::block_on` làm.

Bây giờ hãy kết hợp những mảnh ghép này lại và xem cách chúng ta có thể viết mã concurrent.

<!-- Old headings. Do not remove or links may break. -->

<a id="racing-our-two-urls-against-each-other"></a>

### Racing Hai URLs Lại Với Nhau Đồng Thời

Trong Listing 17-5, chúng ta gọi `page_title` với hai URLs khác nhau được truyền vào từ dòng lệnh và race chúng bằng cách chọn bất kỳ future nào kết thúc trước tiên.

<Listing number="17-5" caption="Gọi `page_title` cho hai URLs để xem cái nào return trước tiên" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-05/src/main.rs:all}}
```

</Listing>

Chúng ta bắt đầu bằng cách gọi `page_title` cho mỗi user-supplied URLs. Chúng ta lưu futures kết quả làm `title_fut_1` và `title_fut_2`. Hãy nhớ rằng, chúng thực sự không làm gì, vì futures là lazy và chúng ta chưa await chúng. Sau đó chúng ta truyền futures đến `trpl::select`, trả về một giá trị để chỉ ra futures nào được truyền vào để nó kết thúc trước tiên.

> Ghi chú: Dưới cùng, `trpl::select` được xây dựng trên một function `select` tổng quát hơn được định nghĩa trong crate `futures`. Function `select` của crate `futures` có thể làm rất nhiều thứ mà function `trpl::select` không thể, nhưng nó cũng có thêm một số phức tạp mà chúng ta có thể bỏ qua bây giờ.

Bất kỳ future nào cũng có thể hợp pháp "thắng", vì vậy nó không có ý nghĩa để trả về một `Result`. Thay vào đó, `trpl::select` trả về một kiểu chúng ta chưa thấy trước đó, `trpl::Either`. Kiểu `Either` hơi giống một `Result` ở chỗ nó có hai cases. Không giống như `Result`, tuy nhiên, không có khái niệm về success hoặc failure baked vào `Either`. Thay vào đó, nó sử dụng `Left` và `Right` để chỉ "cái này hoặc cái kia":

```rust
enum Either<A, B> {
    Left(A),
    Right(B),
}
```

Function `select` trả về `Left` với output của future đó nếu argument đầu tiên thắng, và `Right` với output của future argument thứ hai nếu _cái đó_ thắng. Điều này khớp với thứ tự các arguments xuất hiện trong khi gọi function: argument đầu tiên ở bên trái của argument thứ hai.

Chúng ta cũng cập nhật `page_title` để trả về cùng một URL được truyền vào. Bằng cách đó, nếu trang kết thúc trước tiên không có `<title>` mà chúng ta có thể resolve, chúng ta vẫn có thể in một thông báo có ý nghĩa. Với thông tin đó có sẵn, chúng ta kết thúc bằng cách cập nhật output `println!` để chỉ ra cả URL nào kết thúc trước tiên và cái gì, nếu có, `<title>` cho trang web tại URL đó.

Bây giờ bạn đã xây dựng một small working web scraper! Chọn một số URLs và chạy công cụ dòng lệnh. Bạn có thể khám phá rằng một số sites consistently nhanh hơn những sites khác, trong khi trong những trường hợp khác site nhanh hơn thay đổi từ run này sang run khác. Quan trọng hơn là, bạn đã học được những điều cơ bản về làm việc với futures, vì vậy bây giờ chúng ta có thể đào sâu hơn vào những gì chúng ta có thể làm với async.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
[iterators-lazy]: ch13-02-iterators.html
[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[cli-args]: ch12-01-accepting-command-line-arguments.html

<!-- TODO: map source link version to version of Rust? -->

[crate-source]: https://github.com/rust-lang/book/tree/main/packages/trpl
[futures-crate]: https://crates.io/crates/futures
[tokio]: https://tokio.rs
