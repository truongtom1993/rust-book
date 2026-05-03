## Cargo Workspaces

Trong Chương 12, chúng ta đã xây dựng một package bao gồm một binary crate và một
library crate. Khi dự án của bạn phát triển, bạn có thể thấy rằng library crate
tiếp tục trở nên lớn hơn và bạn muốn chia nhỏ package của bạn hơn nữa thành các
library crates. Cargo cung cấp một feature được gọi là _workspaces_ có thể giúp
quản lý các packages liên quan được phát triển cùng lúc.

### Tạo Workspace

Một _workspace_ là một tập hợp các packages chia sẻ cùng một _Cargo.lock_ và output
directory. Hãy tạo một dự án bằng workspace—chúng ta sẽ sử dụng mã code tầm thường để
chúng ta có thể tập trung vào cấu trúc của workspace. Có nhiều cách để cấu trúc một
workspace, vì vậy chúng ta sẽ chỉ hiển thị một cách phổ biến. Chúng ta sẽ có một
workspace chứa một binary và hai libraries. Binary, sẽ cung cấp chức năng chính, sẽ
phụ thuộc vào hai libraries. Một library sẽ cung cấp hàm `add_one` và library khác
hàm `add_two`. Ba crates này sẽ là một phần của cùng một workspace. Chúng ta sẽ bắt
đầu bằng cách tạo một directory mới cho workspace:

```console
$ mkdir add
$ cd add
```

Tiếp theo, trong directory _add_, chúng ta tạo file _Cargo.toml_ sẽ configure toàn
bộ workspace. File này sẽ không có một section `[package]`. Thay vào đó, nó sẽ bắt
đầu với section `[workspace]` sẽ cho phép chúng ta thêm members vào workspace. Chúng
ta cũng muốn sử dụng phiên bản mới nhất và tuyệt vời nhất của Cargo's resolver
algorithm trong workspace bằng cách đặt giá trị `resolver` thành `"3"`:

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-01-workspace/add/Cargo.toml}}
```

Tiếp theo, chúng ta sẽ tạo binary crate `adder` bằng cách chạy `cargo new` trong
directory _add_:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-01-adder-crate/add
remove `members = ["adder"]` from Cargo.toml
rm -rf adder
cargo new adder
copy output below
-->

```console
$ cargo new adder
     Created binary (application) `adder` package
      Adding `adder` as member of workspace at `file:///projects/add`
```

Chạy `cargo new` bên trong một workspace cũng tự động thêm newly created package
vào key `members` trong definition `[workspace]` trong workspace _Cargo.toml_, như
thế này:

```toml
{{#include ../listings/ch14-more-about-cargo/output-only-01-adder-crate/add/Cargo.toml}}
```

Tại điểm này, chúng ta có thể build workspace bằng cách chạy `cargo build`. Các files
trong directory _add_ của bạn sẽ trông như thế này:

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

Workspace có một _target_ directory ở top level mà các compiled artifacts sẽ được
đặt vào; package `adder` không có target directory riêng của nó. Ngay cả khi chúng
ta chạy `cargo build` từ bên trong directory _adder_, các compiled artifacts sẽ vẫn
kết thúc ở _add/target_ thay vì _add/adder/target_. Cargo cấu trúc _target_ directory
trong workspace như thế này vì các crates trong workspace được cho là phụ thuộc vào
nhau. Nếu mỗi crate có target directory riêng của nó, mỗi crate sẽ phải recompile
mỗi crate khác trong workspace để đặt artifacts vào target directory riêng của nó.
Bằng cách chia sẻ một _target_ directory, các crates có thể tránh unnecessary
rebuilding.

### Tạo Package Thứ Hai Trong Workspace

Tiếp theo, hãy tạo một member package khác trong workspace và gọi nó là `add_one`.
Tạo một library crate mới có tên `add_one`:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-02-add-one/add
remove `"add_one"` from `members` list in Cargo.toml
rm -rf add_one
cargo new add_one --lib
copy output below
-->

```console
$ cargo new add_one --lib
     Created library `add_one` package
      Adding `add_one` as member of workspace at `file:///projects/add`
```

Top-level _Cargo.toml_ bây giờ sẽ include đường dẫn _add_one_ trong list `members`:

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/Cargo.toml}}
```

Directory _add_ của bạn bây giờ nên có các directories và files này:

```text
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

Trong file _add_one/src/lib.rs_, hãy thêm hàm `add_one`:

<span class="filename">Filename: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/add_one/src/lib.rs}}
```

Bây giờ chúng ta có thể có package `adder` với binary của chúng ta phụ thuộc vào
package `add_one` chứa library của chúng ta. Đầu tiên, chúng ta sẽ cần thêm path
dependency trên `add_one` vào _adder/Cargo.toml_.

<span class="filename">Filename: adder/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/adder/Cargo.toml:6:7}}
```

Cargo không giả định rằng các crates trong một workspace sẽ phụ thuộc vào nhau, vì
vậy chúng ta cần phải rõ ràng về các dependency relationships.

Tiếp theo, hãy sử dụng hàm `add_one` (từ crate `add_one`) trong crate `adder`. Mở
file _adder/src/main.rs_ và thay đổi hàm `main` để gọi hàm `add_one`, như trong
Listing 14-7.

<Listing number="14-7" file-name="adder/src/main.rs" caption="Sử dụng library crate `add_one` từ crate `adder`">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-07/add/adder/src/main.rs}}
```

</Listing>

Hãy build workspace bằng cách chạy `cargo build` trong top-level directory _add_!

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.22s
```

Để chạy binary crate từ directory _add_, chúng ta có thể chỉ định package nào trong
workspace chúng ta muốn chạy bằng cách sử dụng argument `-p` và package name với
`cargo run`:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo run -p adder
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo run -p adder
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

Điều này chạy mã code trong _adder/src/main.rs_, phụ thuộc vào crate `add_one`.

<!-- Old headings. Do not remove or links may break. -->

<a id="depending-on-an-external-package-in-a-workspace"></a>

### Phụ Thuộc Trên Một External Package

Lưu ý rằng workspace có chỉ một file _Cargo.lock_ ở top level, thay vì có một
_Cargo.lock_ trong directory của mỗi crate. Điều này đảm bảo rằng tất cả các crates
đang sử dụng cùng một phiên bản của tất cả dependencies. Nếu chúng ta thêm package
`rand` vào files _adder/Cargo.toml_ và _add_one/Cargo.toml_, Cargo sẽ resolve cả
hai thành một phiên bản của `rand` và record điều đó trong one _Cargo.lock_. Làm
cho tất cả các crates trong workspace sử dụng cùng dependencies có nghĩa là các
crates sẽ luôn tương thích với nhau. Hãy thêm crate `rand` vào section
`[dependencies]` trong file _add_one/Cargo.toml_ để chúng ta có thể sử dụng crate
`rand` trong crate `add_one`:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch02-00-guessing-game-tutorial.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
-->

<span class="filename">Filename: add_one/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add/add_one/Cargo.toml:6:7}}
```

Bây giờ chúng ta có thể thêm `use rand;` vào file _add_one/src/lib.rs_, và building
toàn bộ workspace bằng cách chạy `cargo build` trong directory _add_ sẽ bring in
và compile crate `rand`. Chúng ta sẽ nhận được một warning vì chúng ta không refer
tới `rand` mà chúng ta đã bring vào scope:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
   --snip--
   Compiling rand v0.8.5
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

warning: `add_one` (lib) generated 1 warning (run `cargo fix --lib -p add_one` to apply 1 suggestion)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.95s
```

Top-level _Cargo.lock_ bây giờ chứa thông tin về dependency của `add_one` trên
`rand`. Tuy nhiên, mặc dù `rand` được sử dụng ở đâu đó trong workspace, chúng ta
không thể sử dụng nó trong các crates khác trong workspace trừ khi chúng ta cũng
thêm `rand` vào files _Cargo.toml_ của chúng. Ví dụ, nếu chúng ta thêm `use rand;`
vào file _adder/src/main.rs_ cho package `adder`, chúng ta sẽ nhận được một error:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-03-use-rand/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`
```

Để sửa điều này, edit file _Cargo.toml_ cho package `adder` và indicate rằng
`rand` cũng là dependency cho nó. Building package `adder` sẽ thêm `rand` vào danh
sách dependencies cho `adder` trong _Cargo.lock_, nhưng sẽ không có additional
copies của `rand` được download. Cargo sẽ đảm bảo rằng mỗi crate trong mỗi package
trong workspace sử dụng package `rand` sẽ sử dụng phiên bản giống nhau miễn là
chúng specify compatible versions của `rand`, tiết kiệm không gian và đảm bảo rằng
các crates trong workspace sẽ tương thích với nhau.

Nếu các crates trong workspace specify incompatible versions của cùng một dependency,
Cargo sẽ resolve từng crate nhưng vẫn sẽ cố gắng resolve càng ít versions càng tốt.

### Thêm Test Vào Workspace

Để có một enhancement khác, hãy thêm một test của hàm `add_one::add_one` trong crate
`add_one`:

<span class="filename">Filename: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add/add_one/src/lib.rs}}
```

Bây giờ chạy `cargo test` trong top-level directory _add_. Chạy `cargo test` trong
một workspace cấu trúc như cái này sẽ chạy tests cho tất cả các crates trong
workspace:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test
copy output below; the output updating script doesn't handle subdirectories in
paths properly
-->

```console
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.20s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-3a47283c568d2b6a)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Phần đầu tiên của output cho thấy test `it_works` trong crate `add_one` passed.
Phần tiếp theo cho thấy zero tests được tìm thấy trong crate `adder`, và sau đó
phần cuối cùng cho thấy zero documentation tests được tìm thấy trong crate
`add_one`.

Chúng ta cũng có thể chạy tests cho một crate cụ thể trong workspace từ top-level
directory bằng cách sử dụng flag `-p` và specify tên của crate mà chúng ta muốn
test:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test -p add_one
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo test -p add_one
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Output này cho thấy `cargo test` chỉ chạy tests cho crate `add_one` và không chạy
tests của crate `adder`.

Nếu bạn publish các crates trong workspace tới
[crates.io](https://crates.io/)<!-- ignore -->, mỗi crate trong workspace sẽ cần
phải được published riêng biệt. Giống như `cargo test`, chúng ta có thể publish
một crate cụ thể trong workspace của chúng ta bằng cách sử dụng flag `-p` và
specify tên của crate mà chúng ta muốn publish.

Để luyện tập thêm, hãy thêm một crate `add_two` vào workspace này theo cách tương
tự như crate `add_one`!

Khi dự án của bạn phát triển, hãy xem xét sử dụng workspace: Nó cho phép bạn làm
việc với các thành phần nhỏ hơn, dễ hiểu hơn so với một đốm lớn của code. Hơn nữa,
giữ các crates trong một workspace có thể làm cho sự phối hợp giữa các crates dễ
dàng hơn nếu chúng thường xuyên bị thay đổi cùng một lúc.
