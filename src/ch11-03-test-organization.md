## Tổ chức Test

Như đã đề cập ở đầu chương, testing là một lĩnh vực phức tạp, và mọi người sử dụng các thuật ngữ và cách tổ chức khác nhau. Cộng đồng Rust nghĩ về các test theo hai loại chính: unit tests và integration tests. _Unit tests_ nhỏ và tập trung hơn, test một module một cách cô lập, và có thể test các private interfaces. _Integration tests_ hoàn toàn bên ngoài library của bạn và sử dụng code của bạn theo cách giống như bất kỳ external code nào khác, chỉ sử dụng public interface và có thể thực thi nhiều module trong một test.

Việc viết cả hai loại test đều quan trọng để đảm bảo rằng các phần của library hoạt động như bạn mong đợi, riêng lẻ và cùng nhau.

### Unit Tests

Mục đích của unit tests là test từng unit code một cách cô lập với phần còn lại của code để nhanh chóng xác định nơi code hoạt động và không hoạt động như mong đợi. Bạn sẽ đặt unit tests trong thư mục _src_ ở mỗi file chứa code mà chúng đang test. Quy ước là tạo một module tên `tests` trong mỗi file để chứa các test functions và annotate module với `cfg(test)`.

#### The `tests` Module và `#[cfg(test)]`

Annotation `#[cfg(test)]` trên module `tests` bảo Rust compile và run test code chỉ khi bạn run `cargo test`, không phải khi run `cargo build`. Điều này tiết kiệm thời gian compile khi bạn chỉ muốn build library và tiết kiệm không gian trong compiled artifact vì tests không được include. Bạn sẽ thấy rằng vì integration tests nằm ở directory khác, chúng không cần annotation `#[cfg(test)]`. Tuy nhiên, vì unit tests nằm cùng file với code, bạn sẽ sử dụng `#[cfg(test)]` để chỉ định rằng chúng không nên được include trong compiled result.

Hãy nhớ rằng khi chúng ta generate project `adder` mới ở phần đầu chương này, Cargo đã generate code này cho chúng ta:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

Trên module `tests` được generate tự động, attribute `cfg` đứng cho _configuration_ và bảo Rust rằng item tiếp theo chỉ nên được include nếu có một configuration option nhất định. Trong trường hợp này, configuration option là `test`, được cung cấp bởi Rust để compile và run tests. Bằng cách sử dụng attribute `cfg`, Cargo chỉ compile test code của chúng ta nếu chúng ta actively run tests với `cargo test`. Điều này bao gồm bất kỳ helper functions nào có thể nằm trong module này, ngoài các functions được annotate với `#[test]`.

<!-- Old headings. Do not remove or links may break. -->

<a id="testing-private-functions"></a>

#### Private Function Tests

Có tranh luận trong cộng đồng testing về việc có nên test trực tiếp private functions hay không, và các ngôn ngữ khác làm cho việc test private functions khó khăn hoặc không thể. Bất kể bạn theo testing ideology nào, privacy rules của Rust cho phép bạn test private functions. Hãy xem code ở Listing 11-12 với private function `internal_adder`.

<Listing number="11-12" file-name="src/lib.rs" caption="Testing a private function">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-12/src/lib.rs}}
```

</Listing>

Lưu ý rằng function `internal_adder` không được mark là `pub`. Tests chỉ là Rust code, và module `tests` chỉ là một module khác. Như chúng ta đã thảo luận ở [“Paths for Referring to an Item in the Module Tree”][paths]<!-- ignore -->, items trong child modules có thể sử dụng items trong ancestor modules. Trong test này, chúng ta bring tất cả items thuộc parent của module `tests` vào scope với `use super::*`, và sau đó test có thể gọi `internal_adder`. Nếu bạn nghĩ private functions không nên được test, không có gì trong Rust ép bạn làm vậy.

### Integration Tests

Trong Rust, integration tests hoàn toàn external với library của bạn. Chúng sử dụng library theo cách giống như bất kỳ code nào khác, nghĩa là chúng chỉ có thể gọi functions thuộc public API của library. Mục đích của chúng là test xem nhiều phần của library có hoạt động cùng nhau đúng không. Các units code hoạt động đúng riêng lẻ có thể có vấn đề khi được integrate, nên test coverage cho integrated code cũng quan trọng. Để tạo integration tests, trước tiên bạn cần một thư mục _tests_.

#### The _tests_ Directory

Chúng ta tạo thư mục _tests_ ở top level của project directory, next to _src_. Cargo biết tìm integration test files trong directory này. Chúng ta có thể tạo bao nhiêu test files tùy thích, và Cargo sẽ compile mỗi file như một individual crate.

Hãy tạo một integration test. Với code ở Listing 11-12 vẫn trong file _src/lib.rs_, tạo thư mục _tests_, và tạo file mới tên _tests/integration_test.rs_. Cấu trúc directory của bạn nên như thế này:

```text
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

Nhập code ở Listing 11-13 vào file _tests/integration_test.rs_.

<Listing number="11-13" file-name="tests/integration_test.rs" caption="An integration test of a function in the `adder` crate">

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-13/tests/integration_test.rs}}
```

</Listing>

Mỗi file trong thư mục _tests_ là một separate crate, nên chúng ta cần bring library vào scope của mỗi test crate. Vì lý do đó, chúng ta thêm `use adder::add_two;` ở đầu code, điều mà chúng ta không cần trong unit tests.

Chúng ta không cần annotate bất kỳ code nào trong _tests/integration_test.rs_ với `#[cfg(test)]`. Cargo treats thư mục _tests_ specially và chỉ compile files trong directory này khi chúng ta run `cargo test`. Chạy `cargo test` ngay bây giờ:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-13/output.txt}}
```

Ba phần output bao gồm unit tests, integration test, và doc tests. Lưu ý rằng nếu bất kỳ test nào trong một section fail, các section sau sẽ không được run. Ví dụ, nếu unit test fail, sẽ không có output cho integration và doc tests, vì những tests đó chỉ được run nếu tất cả unit tests pass.

Phần đầu tiên cho unit tests giống như chúng ta đã thấy: một line cho mỗi unit test (một tên `internal` mà chúng ta thêm ở Listing 11-12) và sau đó một summary line cho unit tests.

Phần integration tests bắt đầu với line `Running tests/integration_test.rs`. Tiếp theo, có line cho mỗi test function trong integration test đó và một summary line cho kết quả của integration test ngay trước khi section `Doc-tests adder` bắt đầu.

Mỗi integration test file có section riêng, nên nếu chúng ta thêm nhiều files hơn trong thư mục _tests_, sẽ có nhiều integration test sections hơn.

Chúng ta vẫn có thể run một particular integration test function bằng cách chỉ định tên test function như argument cho `cargo test`. Để run tất cả tests trong một particular integration test file, sử dụng argument `--test` của `cargo test` theo sau tên file:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-05-single-integration/output.txt}}
```

Lệnh này chỉ run tests trong file _tests/integration_test.rs_.

#### Submodules in Integration Tests

Khi bạn thêm nhiều integration tests hơn, bạn có thể muốn tạo nhiều files hơn trong thư mục _tests_ để giúp tổ chức chúng; ví dụ, bạn có thể group test functions theo functionality chúng đang test. Như đã đề cập trước, mỗi file trong thư mục _tests_ được compile như separate crate, hữu ích để tạo separate scopes gần giống cách end users sẽ sử dụng crate của bạn. Tuy nhiên, điều này nghĩa là files trong thư mục _tests_ không share behavior giống files trong _src_, như bạn đã học ở Chapter 7 về cách separate code vào modules và files.

Behavior khác biệt của _tests_ directory files rõ nhất khi bạn có set helper functions để sử dụng trong multiple integration test files, và bạn thử follow các bước ở [“Separating Modules into Different Files”][separating-modules-into-files]<!-- ignore --> section của Chapter 7 để extract chúng vào common module. Ví dụ, nếu chúng ta tạo _tests/common.rs_ và đặt function tên `setup` trong đó, chúng ta có thể thêm code vào `setup` mà chúng ta muốn gọi từ multiple test functions trong multiple test files:

<span class="filename">Filename: tests/common.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/tests/common.rs}}
```

Khi chúng ta run tests lại, chúng ta sẽ thấy một section mới trong test output cho file _common.rs_, dù file này không chứa test functions nào và chúng ta không gọi `setup` function từ đâu cả:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/output.txt}}
```

Việc `common` xuất hiện trong test results với `running 0 tests` không phải là điều chúng ta muốn. Chúng ta chỉ muốn share code với các integration test files khác. Để tránh `common` xuất hiện trong test output, thay vì tạo _tests/common.rs_, chúng ta sẽ tạo _tests/common/mod.rs_. Project directory bây giờ như thế này:

```text
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

Đây là older naming convention mà Rust cũng hiểu, được đề cập ở [“Alternate File Paths”][alt-paths]<!-- ignore --> ở Chapter 7. Naming file theo cách này bảo Rust không treat module `common` như integration test file. Khi chúng ta move code function `setup` vào _tests/common/mod.rs_ và delete file _tests/common.rs_, section trong test output sẽ không còn xuất hiện. Files trong subdirectories của thư mục _tests_ không được compile như separate crates hoặc có sections trong test output.

Sau khi tạo _tests/common/mod.rs_, chúng ta có thể sử dụng nó từ bất kỳ integration test files nào như một module. Đây là ví dụ gọi function `setup` từ test `it_adds_two` trong _tests/integration_test.rs_:

<span class="filename">Filename: tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-13-fix-shared-test-code-problem/tests/integration_test.rs}}
```

Lưu ý rằng declaration `mod common;` giống như module declaration chúng ta demo ở Listing 7-21. Sau đó, trong test function, chúng ta có thể gọi function `common::setup()`.

#### Integration Tests for Binary Crates

Nếu project của chúng ta là binary crate chỉ chứa file _src/main.rs_ và không có file _src/lib.rs_, chúng ta không thể tạo integration tests trong thư mục _tests_ và bring functions định nghĩa trong file _src/main.rs_ vào scope với `use` statement. Chỉ library crates expose functions mà crates khác có thể sử dụng; binary crates được meant để run on their own.

Đây là một lý do Rust projects cung cấp binary có straightforward file _src/main.rs_ gọi logic nằm trong file _src/lib.rs_. Sử dụng structure đó, integration tests _có thể_ test library crate với `use` để make important functionality available. Nếu important functionality hoạt động, small amount code trong file _src/main.rs_ cũng sẽ hoạt động, và small amount code đó không cần test.

## Tóm tắt

Testing features của Rust cung cấp cách specify cách code nên function để đảm bảo nó tiếp tục hoạt động như bạn expect, ngay cả khi bạn make changes. Unit tests exercise các phần khác nhau của library riêng lẻ và có thể test private implementation details. Integration tests check nhiều phần của library hoạt động cùng nhau đúng, và chúng sử dụng public API của library để test code theo cách external code sẽ sử dụng. Dù type system và ownership rules của Rust giúp prevent một số loại bugs, tests vẫn quan trọng để reduce logic bugs liên quan đến cách code expected to behave.

Hãy combine kiến thức bạn học ở chapter này và các chapter trước để work on a project!

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[separating-modules-into-files]: ch07-05-separating-modules-into-different-files.html
[alt-paths]: ch07-05-separating-modules-into-different-files.html#alternate-file-paths