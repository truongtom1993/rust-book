## Xuất bản Crate lên Crates.io

Chúng ta đã sử dụng các package từ [crates.io](https://crates.io/)<!-- ignore --> như là dependency của project, nhưng bạn cũng có thể chia sẻ code của mình với người khác bằng cách xuất bản các package của riêng bạn. Registry crate tại [crates.io](https://crates.io/)<!-- ignore --> phân phối source code của các package của bạn, vì vậy nó chủ yếu host code mã nguồn mở. [doc.rust-lang](https://doc.rust-lang.org/cargo/reference/publishing.html)

Rust và Cargo có các tính năng giúp package đã xuất bản của bạn dễ dàng hơn để mọi người tìm thấy và sử dụng. Chúng ta sẽ nói về một số tính năng này tiếp theo và sau đó giải thích cách xuất bản một package. [doc.rust-lang](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html)

### Tạo Documentation Comment Hữu Ích

Việc document package của bạn một cách chính xác sẽ giúp người dùng khác biết cách và khi nào sử dụng chúng, vì vậy rất đáng để đầu tư thời gian viết documentation. Trong Chương 3, chúng ta đã thảo luận về cách comment code Rust bằng hai dấu gạch chéo, `//`. Rust cũng có một loại comment đặc biệt cho documentation, được gọi một cách thuận tiện là _documentation comment_, sẽ tạo ra HTML documentation. HTML hiển thị nội dung của documentation comment cho các public API item dành cho lập trình viên quan tâm đến việc biết cách _sử dụng_ crate của bạn chứ không phải cách crate của bạn được _triển khai_. [doc.rust-lang](https://doc.rust-lang.org/rust-by-example/meta/doc.html)

Documentation comment sử dụng ba dấu gạch chéo, `///`, thay vì hai và hỗ trợ ký hiệu Markdown để định dạng văn bản. Đặt documentation comment ngay trước item mà chúng đang document. Listing 14-1 cho thấy documentation comment cho function `add_one` trong một crate có tên `my_crate`. [doc.rust-lang](https://doc.rust-lang.org/rust-by-example/meta/doc.html)

<Listing number="14-1" file-name="src/lib.rs" caption="Một documentation comment cho function">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-01/src/lib.rs}}
```

</Listing>

Ở đây, chúng ta đưa ra mô tả về những gì function `add_one` làm, bắt đầu một phần với heading `Examples`, và sau đó cung cấp code minh họa cách sử dụng function `add_one`. Chúng ta có thể tạo HTML documentation từ documentation comment này bằng cách chạy `cargo doc`. Command này chạy tool `rustdoc` được phân phối với Rust và đặt HTML documentation đã tạo vào thư mục _target/doc_. [doc.rust-lang](https://doc.rust-lang.org/rust-by-example/meta/doc.html)

Để thuận tiện, việc chạy `cargo doc --open` sẽ build HTML cho documentation của crate hiện tại (cũng như documentation cho tất cả các dependency của crate) và mở kết quả trong trình duyệt web. Điều hướng đến function `add_one` và bạn sẽ thấy cách văn bản trong documentation comment được render, như trong Hình 14-1. [doc.rust-lang](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html)

<img alt="Rendered HTML documentation for the `add_one` function of `my_crate`" src="img/trpl14-01.png" class="center" />

<span class="caption">Hình 14-1: HTML documentation cho function `add_one`</span>

#### Các Section Thường Được Sử Dụng

Chúng ta đã sử dụng Markdown heading `# Examples` trong Listing 14-1 để tạo một section trong HTML với tiêu đề "Examples". Dưới đây là một số section khác mà tác giả crate thường sử dụng trong documentation của họ: [doc.rust-lang](https://doc.rust-lang.org/rustdoc/how-to-write-documentation.html)

- **Panics**: Đây là các tình huống mà function đang được document có thể panic. Những người gọi function không muốn chương trình của họ panic nên đảm bảo họ không gọi function trong những tình huống này.
- **Errors**: Nếu function trả về `Result`, việc mô tả các loại error có thể xảy ra và điều kiện nào có thể khiến những error đó được trả về có thể hữu ích cho người gọi để họ có thể viết code xử lý các loại error khác nhau theo những cách khác nhau.
- **Safety**: Nếu function `unsafe` để gọi (chúng ta thảo luận về unsafety trong Chương 20), nên có một section giải thích tại sao function là unsafe và bao gồm các invariant mà function mong đợi người gọi tuân thủ.

Hầu hết các documentation comment không cần tất cả các section này, nhưng đây là một checklist tốt để nhắc nhở bạn về các khía cạnh của code mà người dùng sẽ quan tâm biết. [doc.rust-lang](https://doc.rust-lang.org/rustdoc/how-to-write-documentation.html)

#### Documentation Comment như Test

Việc thêm code block ví dụ trong documentation comment của bạn có thể giúp minh họa cách sử dụng library của bạn và có một bonus bổ sung: Chạy `cargo test` sẽ chạy các code example trong documentation của bạn như test! Không có gì tốt hơn documentation với các ví dụ. Nhưng không có gì tồi tệ hơn các ví dụ không hoạt động vì code đã thay đổi kể từ khi documentation được viết. Nếu chúng ta chạy `cargo test` với documentation cho function `add_one` từ Listing 14-1, chúng ta sẽ thấy một section trong kết quả test trông như thế này: [doc.rust-lang](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html)

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-01/
cargo test
copy just the doc-tests section below
-->

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
```

Bây giờ, nếu chúng ta thay đổi function hoặc ví dụ sao cho `assert_eq!` trong ví dụ panic, và chạy `cargo test` lại, chúng ta sẽ thấy rằng doc test phát hiện ra rằng ví dụ và code không đồng bộ với nhau ! [doc.rust-lang](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html)

<!-- Old headings. Do not remove or links may break. -->

<a id="commenting-contained-items"></a>

#### Contained Item Comment

Kiểu doc comment `//!` thêm documentation vào item mà *chứa* các comment thay vì các item *theo sau* các comment. Chúng ta thường sử dụng các doc comment này bên trong crate root file (_src/lib.rs_ theo quy ước) hoặc bên trong một module để document crate hoặc module như một tổng thể. [doc.rust-lang](https://doc.rust-lang.org/reference/comments.html)

Ví dụ, để thêm documentation mô tả mục đích của crate `my_crate` chứa function `add_one`, chúng ta thêm documentation comment bắt đầu bằng `//!` vào đầu file _src/lib.rs_, như trong Listing 14-2.

<Listing number="14-2" file-name="src/lib.rs" caption="Documentation cho crate `my_crate` như một tổng thể">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-02/src/lib.rs:here}}
```

</Listing>

Lưu ý không có bất kỳ code nào sau dòng cuối cùng bắt đầu bằng `//!`. Bởi vì chúng ta bắt đầu comment bằng `//!` thay vì `///`, chúng ta đang document item chứa comment này thay vì một item theo sau comment này. Trong trường hợp này, item đó là file _src/lib.rs_, đó là crate root. Những comment này mô tả toàn bộ crate. [doc.rust-lang](https://doc.rust-lang.org/reference/comments.html)

Khi chúng ta chạy `cargo doc --open`, các comment này sẽ hiển thị trên trang đầu của documentation cho `my_crate` phía trên danh sách các public item trong crate, như trong Hình 14-2.

Documentation comment bên trong item rất hữu ích để mô tả crate và module đặc biệt. Sử dụng chúng để giải thích mục đích tổng thể của container để giúp người dùng của bạn hiểu tổ chức của crate. [doc.rust-lang](https://doc.rust-lang.org/rustdoc/how-to-write-documentation.html)

<img alt="Rendered HTML documentation with a comment for the crate as a whole" src="img/trpl14-02.png" class="center" />

<span class="caption">Hình 14-2: Documentation được render cho `my_crate`, bao gồm comment mô tả crate như một tổng thể</span>

<!-- Old headings. Do not remove or links may break. -->

<a id="exporting-a-convenient-public-api-with-pub-use"></a>

### Export Public API Thuận Tiện

Cấu trúc của public API của bạn là một cân nhắc quan trọng khi xuất bản một crate. Những người sử dụng crate của bạn ít quen thuộc với cấu trúc hơn bạn và có thể gặp khó khăn trong việc tìm các phần họ muốn sử dụng nếu crate của bạn có một module hierarchy lớn. [doc.rust-lang](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html)

Trong Chương 7, chúng ta đã đề cập đến cách làm cho item public bằng cách sử dụng keyword `pub`, và cách đưa item vào scope với keyword `use`. Tuy nhiên, cấu trúc có ý nghĩa với bạn trong khi bạn đang phát triển một crate có thể không thuận tiện lắm cho người dùng của bạn. Bạn có thể muốn tổ chức struct của mình trong một hierarchy chứa nhiều level, nhưng sau đó những người muốn sử dụng type bạn đã định nghĩa sâu trong hierarchy có thể gặp khó khăn trong việc tìm ra rằng type đó tồn tại. Họ cũng có thể khó chịu khi phải nhập `use my_crate::some_module::another_module::UsefulType;` thay vì `use my_crate::UsefulType;`. [doc.rust-lang](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html)

Tin tốt là nếu cấu trúc _không_ thuận tiện cho người khác sử dụng từ library khác, bạn không phải sắp xếp lại tổ chức nội bộ của mình: Thay vào đó, bạn có thể re-export item để tạo một cấu trúc public khác với cấu trúc private của bạn bằng cách sử dụng `pub use`. *Re-exporting* lấy một public item ở một vị trí và làm cho nó public ở một vị trí khác, như thể nó được định nghĩa ở vị trí khác. [effective-rust](https://effective-rust.com/re-export.html)

Ví dụ, giả sử chúng ta tạo một library có tên `art` để mô hình hóa các khái niệm nghệ thuật. Trong library này có hai module: một module `kinds` chứa hai enum có tên `PrimaryColor` và `SecondaryColor` và một module `utils` chứa một function có tên `mix`, như trong Listing 14-3.

<Listing number="14-3" file-name="src/lib.rs" caption="Một library `art` với các item được tổ chức thành module `kinds` và `utils`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-03/src/lib.rs:here}}
```

</Listing>

Hình 14-3 cho thấy trang đầu của documentation cho crate này được tạo bởi `cargo doc` sẽ trông như thế nào.

<img alt="Rendered documentation for the `art` crate that lists the `kinds` and `utils` modules" src="img/trpl14-03.png" class="center" />

<span class="caption">Hình 14-3: Trang đầu của documentation cho `art` liệt kê module `kinds` và `utils`</span>

Lưu ý rằng type `PrimaryColor` và `SecondaryColor` không được liệt kê trên trang đầu, cũng như function `mix`. Chúng ta phải click vào `kinds` và `utils` để xem chúng.

Một crate khác phụ thuộc vào library này sẽ cần statement `use` đưa các item từ `art` vào scope, chỉ định cấu trúc module hiện đang được định nghĩa. Listing 14-4 cho thấy một ví dụ về một crate sử dụng item `PrimaryColor` và `mix` từ crate `art`.

<Listing number="14-4" file-name="src/main.rs" caption="Một crate sử dụng các item của crate `art` với cấu trúc nội bộ của nó được export">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-04/src/main.rs}}
```

</Listing>

Tác giả của code trong Listing 14-4, sử dụng crate `art`, phải tìm ra rằng `PrimaryColor` nằm trong module `kinds` và `mix` nằm trong module `utils`. Cấu trúc module của crate `art` liên quan nhiều hơn đến developer làm việc trên crate `art` hơn là những người sử dụng nó. Cấu trúc nội bộ không chứa bất kỳ thông tin hữu ích nào cho ai đó đang cố gắng hiểu cách sử dụng crate `art`, mà thay vào đó gây nhầm lẫn vì developer sử dụng nó phải tìm ra nơi cần xem, và phải chỉ định tên module trong statement `use`. [doc.rust-lang](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html)

Để loại bỏ tổ chức nội bộ khỏi public API, chúng ta có thể sửa đổi code crate `art` trong Listing 14-3 để thêm statement `pub use` để re-export các item ở level cao nhất, như trong Listing 14-5.

<Listing number="14-5" file-name="src/lib.rs" caption="Thêm statement `pub use` để re-export item">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-05/src/lib.rs:here}}
```

</Listing>

API documentation mà `cargo doc` tạo cho crate này bây giờ sẽ liệt kê và liên kết các re-export trên trang đầu, như trong Hình 14-4, làm cho type `PrimaryColor` và `SecondaryColor` và function `mix` dễ tìm hơn. [doc.rust-lang](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html)

<img alt="Rendered documentation for the `art` crate with the re-exports on the front page" src="img/trpl14-04.png" class="center" />

<span class="caption">Hình 14-4: Trang đầu của documentation cho `art` liệt kê các re-export</span>

Người dùng crate `art` vẫn có thể xem và sử dụng cấu trúc nội bộ từ Listing 14-3 như được minh họa trong Listing 14-4, hoặc họ có thể sử dụng cấu trúc thuận tiện hơn trong Listing 14-5, như trong Listing 14-6.

<Listing number="14-6" file-name="src/main.rs" caption="Một chương trình sử dụng các item được re-export từ crate `art`">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-06/src/main.rs:here}}
```

</Listing>

Trong trường hợp có nhiều nested module, việc re-export các type ở level cao nhất với `pub use` có thể tạo ra sự khác biệt đáng kể trong trải nghiệm của những người sử dụng crate. Một use case phổ biến khác của `pub use` là re-export định nghĩa của một dependency trong crate hiện tại để làm cho định nghĩa của crate đó trở thành một phần của public API của crate của bạn. [effective-rust](https://effective-rust.com/re-export.html)

Việc tạo một cấu trúc public API hữu ích là một nghệ thuật hơn là khoa học, và bạn có thể lặp lại để tìm API hoạt động tốt nhất cho người dùng của bạn. Việc chọn `pub use` mang lại cho bạn sự linh hoạt trong cách bạn cấu trúc crate của mình nội bộ và tách cấu trúc nội bộ đó khỏi những gì bạn trình bày cho người dùng của bạn. Hãy xem một số code của crate bạn đã cài đặt để xem liệu cấu trúc nội bộ của chúng có khác với public API của chúng không. [doc.rust-lang](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html)

### Thiết Lập Tài Khoản Crates.io

Trước khi bạn có thể xuất bản bất kỳ crate nào, bạn cần tạo một tài khoản trên [crates.io](https://crates.io/)<!-- ignore --> và nhận API token. Để làm như vậy, hãy truy cập trang chủ tại [crates.io](https://crates.io/)<!-- ignore --> và đăng nhập qua tài khoản GitHub. (Tài khoản GitHub hiện là một yêu cầu, nhưng trang web có thể hỗ trợ các cách khác để tạo tài khoản trong tương lai.) Sau khi bạn đã đăng nhập, hãy truy cập cài đặt tài khoản của bạn tại [https://crates.io/me/](https://crates.io/me/)<!-- ignore --> và lấy API key của bạn. Sau đó, chạy command `cargo login` và paste API key của bạn khi được nhắc, như thế này: [doc.rust-lang](https://doc.rust-lang.org/cargo/reference/publishing.html)

```console
$ cargo login
abcdefghijklmnopqrstuvwxyz012345
```

Command này sẽ thông báo cho Cargo về API token của bạn và lưu trữ nó cục bộ trong _~/.cargo/credentials.toml_. Lưu ý rằng token này là một bí mật: Đừng chia sẻ nó với bất kỳ ai khác. Nếu bạn chia sẻ nó với bất kỳ ai vì bất kỳ lý do gì, bạn nên thu hồi nó và tạo một token mới trên [crates.io](https://crates.io/)<!-- ignore -->. [doc.rust-lang](https://doc.rust-lang.org/cargo/reference/publishing.html)

### Thêm Metadata vào Crate Mới

Giả sử bạn có một crate bạn muốn xuất bản. Trước khi xuất bản, bạn cần thêm một số metadata trong section `[package]` của file _Cargo.toml_ của crate. [doc.rust-lang](https://doc.rust-lang.org/cargo/reference/publishing.html)

Crate của bạn sẽ cần một tên duy nhất. Trong khi bạn đang làm việc trên một crate cục bộ, bạn có thể đặt tên crate bất cứ thứ gì bạn muốn. Tuy nhiên, tên crate trên [crates.io](https://crates.io/)<!-- ignore --> được phân bổ theo nguyên tắc ai đến trước được phục vụ trước. Khi một tên crate đã được sử dụng, không ai khác có thể xuất bản một crate với tên đó. Trước khi cố gắng xuất bản một crate, hãy tìm kiếm tên bạn muốn sử dụng. Nếu tên đã được sử dụng, bạn sẽ cần tìm một tên khác và chỉnh sửa field `name` trong file _Cargo.toml_ dưới section `[package]` để sử dụng tên mới để xuất bản, như sau: [doc.rust-lang](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html)

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
```

Ngay cả khi bạn đã chọn một tên duy nhất, khi bạn chạy `cargo publish` để xuất bản crate tại thời điểm này, bạn sẽ nhận được một cảnh báo và sau đó là một error: [doc.rust-lang](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html)

<!-- manual-regeneration
Create a new package with an unregistered name, making no further modifications
  to the generated package, so it is missing the description and license fields.
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
warning: manifest has no description, license, license-file, documentation, homepage or repository.
See [https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata](https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata) for more info.
--snip--
error: failed to publish to registry at [https://crates.io](https://crates.io)

Caused by:
  the remote server responded with an error (status 400 Bad Request): missing or empty metadata fields: description, license. Please see [https://doc.rust-lang.org/cargo/reference/manifest.html](https://doc.rust-lang.org/cargo/reference/manifest.html) for more information on configuring these fields
```

Điều này dẫn đến một error vì bạn thiếu một số thông tin quan trọng: Một mô tả và license là bắt buộc để mọi người biết crate của bạn làm gì và theo điều khoản nào họ có thể sử dụng nó. Trong _Cargo.toml_, thêm một mô tả chỉ là một hoặc hai câu, vì nó sẽ xuất hiện với crate của bạn trong kết quả tìm kiếm. Đối với field `license`, bạn cần cung cấp một _license identifier value_. [Linux Foundation's Software Package Data Exchange (SPDX)][spdx] liệt kê các identifier bạn có thể sử dụng cho giá trị này. Ví dụ, để chỉ định rằng bạn đã cấp phép crate của mình bằng MIT License, hãy thêm identifier `MIT`: [doc.rust-lang](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html)

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
license = "MIT"
```

Nếu bạn muốn sử dụng một license không xuất hiện trong SPDX, bạn cần đặt văn bản của license đó vào một file, bao gồm file trong project của bạn, và sau đó sử dụng `license-file` để chỉ định tên của file đó thay vì sử dụng key `license`. [doc.rust-lang](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html)

Hướng dẫn về license nào phù hợp với project của bạn nằm ngoài phạm vi của cuốn sách này. Nhiều người trong cộng đồng Rust cấp phép project của họ theo cách tương tự như Rust bằng cách sử dụng license kép `MIT OR Apache-2.0`. Thực hành này cho thấy rằng bạn cũng có thể chỉ định nhiều license identifier được phân tách bằng `OR` để có nhiều license cho project của bạn. [doc.rust-lang](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html)

Với một tên duy nhất, version, mô tả của bạn và một license được thêm vào, file _Cargo.toml_ cho một project sẵn sàng xuất bản có thể trông như thế này:

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2024"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```

[Documentation của Cargo](https://doc.rust-lang.org/cargo/) mô tả metadata khác bạn có thể chỉ định để đảm bảo rằng người khác có thể khám phá và sử dụng crate của bạn dễ dàng hơn.

### Xuất Bản lên Crates.io

Bây giờ bạn đã tạo một tài khoản, lưu API token của bạn, chọn một tên cho crate của bạn và chỉ định metadata bắt buộc, bạn đã sẵn sàng để xuất bản ! Xuất bản một crate tải lên một version cụ thể lên [crates.io](https://crates.io/)<!-- ignore --> để người khác sử dụng. [doc.rust-lang](https://doc.rust-lang.org/cargo/reference/publishing.html)

Hãy cẩn thận, vì việc xuất bản là _vĩnh viễn_. Version không bao giờ có thể bị ghi đè, và code không thể bị xóa ngoại trừ trong một số trường hợp nhất định. Một mục tiêu chính của Crates.io là hoạt động như một kho lưu trữ vĩnh viễn code để các build của tất cả các project phụ thuộc vào crate từ [crates.io](https://crates.io/)<!-- ignore --> sẽ tiếp tục hoạt động. Cho phép xóa version sẽ làm cho việc thực hiện mục tiêu đó là không thể. Tuy nhiên, không có giới hạn về số lượng version crate bạn có thể xuất bản. [doc.rust-lang](https://doc.rust-lang.org/cargo/reference/publishing.html)

Chạy command `cargo publish` một lần nữa. Bây giờ nó sẽ thành công: [doc.rust-lang](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html)

<!-- manual-regeneration
go to some valid crate, publish a new version
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
    Packaged 6 files, 1.2KiB (895.0B compressed)
   Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
   Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
    Uploaded guessing_game v0.1.0 to registry `crates-io`
note: waiting for `guessing_game v0.1.0` to be available at registry
`crates-io`.
You may press ctrl-c to skip waiting; the crate should be available shortly.
   Published guessing_game v0.1.0 at registry `crates-io`
```

Chúc mừng! Bạn đã chia sẻ code của mình với cộng đồng Rust, và bất kỳ ai cũng có thể dễ dàng thêm crate của bạn làm dependency của project của họ. [doc.rust-lang](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html)

### Xuất Bản Version Mới của Crate Hiện Có

Khi bạn đã thực hiện thay đổi đối với crate của mình và sẵn sàng phát hành một version mới, bạn thay đổi giá trị `version` được chỉ định trong file _Cargo.toml_ của bạn và xuất bản lại. Sử dụng [quy tắc Semantic Versioning][semver] để quyết định số version tiếp theo thích hợp là gì, dựa trên các loại thay đổi bạn đã thực hiện. Sau đó, chạy `cargo publish` để tải lên version mới. [doc.rust-lang](https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html)

<!-- Old headings. Do not remove or links may break. -->

<a id="removing-versions-from-cratesio-with-cargo-yank"></a>
<a id="deprecating-versions-from-cratesio-with-cargo-yank"></a>

### Deprecate Version từ Crates.io

Mặc dù bạn không thể xóa các version trước của một crate, bạn có thể ngăn bất kỳ project nào trong tương lai thêm chúng làm dependency mới. Điều này hữu ích khi một version crate bị hỏng vì một lý do hoặc lý do khác. Trong những tình huống như vậy, Cargo hỗ trợ yanking một version crate.

_Yanking_ một version ngăn các project mới phụ thuộc vào version đó trong khi cho phép tất cả các project hiện có phụ thuộc vào nó tiếp tục. Về cơ bản, một yank có nghĩa là tất cả các project với _Cargo.lock_ sẽ không bị hỏng, và bất kỳ file _Cargo.lock_ nào trong tương lai được tạo sẽ không sử dụng version đã yank.

Để yank một version của một crate, trong thư mục của crate mà bạn đã xuất bản trước đó, hãy chạy `cargo yank` và chỉ định version bạn muốn yank. Ví dụ, nếu chúng ta đã xuất bản một crate có tên `guessing_game` version 1.0.1 và chúng ta muốn yank nó, thì chúng ta sẽ chạy như sau trong thư mục project cho `guessing_game`:

<!-- manual-regeneration:
cargo yank carol-test --version 2.1.0
cargo yank carol-test --version 2.1.0 --undo
-->

```console
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1
```

Bằng cách thêm `--undo` vào command, bạn cũng có thể hoàn tác một yank và cho phép các project bắt đầu phụ thuộc vào một version một lần nữa:

```console
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1
```

Một yank _không_ xóa bất kỳ code nào. Ví dụ, nó không thể xóa các secret được tải lên vô tình. Nếu điều đó xảy ra, bạn phải reset những secret đó ngay lập tức.

[spdx]: [https://spdx.org/licenses/](https://spdx.org/licenses/)
[semver]: [https://semver.org/](https://semver.org/)