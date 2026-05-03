## Tổ chức Test

Như đã đề cập ở đầu chương, testing là một lĩnh vực phức tạp, và mọi người sử dụng các thuật ngữ và cách tổ chức khác nhau. Cộng đồng Rust nghĩ về các test theo hai loại chính: unit tests và integration tests. _Unit tests_ nhỏ và tập trung hơn, test một module một cách cô lập, và có thể test các private interfaces. _Integration tests_ hoàn toàn bên ngoài library của bạn và sử dụng code của bạn theo cách mà bất kỳ code bên ngoài nào khác sẽ sử dụng, chỉ dùng public interface và có thể thực thi nhiều module trong một test. 

Việc viết cả hai loại test là quan trọng để đảm bảo rằng các phần của library hoạt động như bạn mong đợi, riêng lẻ và cùng nhau. 

### Unit Tests

Mục đích của unit tests là test từng unit code một cách cô lập với phần còn lại của code để nhanh chóng xác định nơi code hoạt động đúng và không đúng như mong đợi. Bạn sẽ đặt unit tests trong thư mục _src_ ở mỗi file chứa code mà chúng test. Quy ước là tạo một module tên `tests` trong mỗi file để chứa các test functions và annotate module với `cfg(test)`. 

#### The `tests` Module và `#[cfg(test)]`

Annotation `#[cfg(test)]` trên module `tests` bảo Rust chỉ compile và run test code khi bạn chạy `cargo test`, không phải khi chạy `cargo build`. Điều này tiết kiệm thời gian compile khi bạn chỉ muốn build library và tiết kiệm không gian trong artifact đã compile vì tests không được include. Bạn sẽ thấy rằng vì integration tests nằm ở thư mục khác, chúng không cần annotation `#[cfg(test)]`. Tuy nhiên, vì unit tests nằm cùng file với code, bạn sẽ dùng `#[cfg(test)]` để chỉ định rằng chúng không nên được include trong kết quả compile. 

Hãy nhớ rằng khi chúng ta generate project `adder` mới ở phần đầu chương này, Cargo đã generate code này cho chúng ta:

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

Trên module `tests` được generate tự động, attribute `cfg` viết tắt của _configuration_ và bảo Rust rằng item tiếp theo chỉ nên được include nếu có một configuration option nhất định. Trong trường hợp này, configuration option là `test`, được Rust cung cấp để compile và run tests. Bằng cách dùng attribute `cfg`, Cargo chỉ compile test code của chúng ta nếu chúng ta chạy tests với `cargo test`. Điều này bao gồm bất kỳ helper functions nào có thể nằm trong module này, ngoài các functions được annotate với `#[test]`. 

<!-- Old headings. Do not remove or links may break. -->

<a id=\"testing-private-functions\"></a>

#### Private Function Tests

Có tranh luận trong cộng đồng testing về việc có nên test trực tiếp private functions hay không, và các ngôn ngữ khác làm khó khăn hoặc không thể test private functions. Bất kể bạn theo trường phái testing nào, privacy rules của Rust cho phép bạn test private functions. Hãy xem code trong Listing 11-12 với private function `internal_adder`. 

<Listing number=\"11-12\" file-name=\"src/lib.rs\" caption=\"Testing a private function\">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-12/src/lib.rs}}
```

</Listing>

Lưu ý rằng function `internal_adder` không được mark là `pub`. Tests chỉ là Rust code thông thường, và module `tests` chỉ là một module khác. Như chúng ta đã thảo luận trong [“Paths for Referring to an Item in the Module Tree”][paths]<!-- ignore -->, items trong child modules có thể sử dụng items trong ancestor modules. Trong test này, chúng ta đưa tất cả items thuộc parent của module `tests` vào scope với `use super::*`, rồi test có thể gọi `internal_adder`. Nếu bạn không nghĩ private functions nên được test, không có gì trong Rust bắt buộc bạn làm vậy. 

### Integration Tests

Trong Rust, integration tests hoàn toàn bên ngoài library của bạn. Chúng sử dụng library theo cách mà bất kỳ code nào khác sẽ sử dụng, nghĩa là chúng chỉ có thể gọi functions thuộc public API của library. Mục đích của chúng là test xem nhiều phần của library có hoạt động đúng cùng nhau không. Các units code hoạt động đúng riêng lẻ có thể có vấn đề khi integrate, nên test coverage cho integrated code cũng quan trọng. Để tạo integration tests, trước tiên bạn cần một thư mục _tests_. 

#### The _tests_ Directory

Chúng ta tạo thư mục _tests_ ở top level của project directory, cạnh _src_. Cargo biết tìm integration test files trong thư mục này. Chúng ta có thể tạo bao nhiêu test files tùy ý, và Cargo sẽ compile mỗi file như một individual crate. 

Hãy tạo một integration test. Với code trong Listing 11-12 vẫn ở file _src/lib.rs_, tạo thư mục _tests_, và tạo file mới tên _tests/integration_test.rs_. Cấu trúc directory của bạn nên như thế này:

```text
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

Nhập code trong Listing 11-13 vào file _tests/integration_test.rs_.

<Listing number=\"11-13\" file-name=\"tests/integration_test.rs\" caption=\"An integration test of a function in the `adder` crate\">

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-13/tests/integration_test.rs}}
```

</Listing>

Mỗi file trong thư mục _tests_ là một separate crate, nên chúng ta cần đưa library vào scope của mỗi test crate. Vì lý do đó, chúng ta thêm `use adder::add_two;` ở đầu code, điều mà chúng ta không cần trong unit tests. 

Chúng ta không cần annotate bất kỳ code nào trong _tests/integration_test.rs_ với `#[cfg(test)]`. Cargo đối xử đặc biệt với thư mục _tests_ và chỉ compile files trong thư mục này khi chạy `cargo test`. Chạy `cargo test` bây giờ:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-13/output.txt}}
```

Ba phần output bao gồm unit tests, integration test, và doc tests. Lưu ý rằng nếu bất kỳ test nào trong một section fail, các section sau sẽ không run. Ví dụ, nếu unit test fail, sẽ không có output cho integration và