## Lỗi Không Thể Phục Hồi với `panic!`

Đôi khi những điều xấu xảy ra trong code của bạn, và không có gì bạn có thể làm
về nó. Trong những trường hợp này, Rust có macro `panic!`. Có hai cách để gây ra
một panic trong thực hành: bằng cách thực hiện một hành động khiến code của chúng ta
panic (chẳng hạn như truy cập một mảng vượt quá phạm vi) hoặc bằng cách gọi một cách
rõ ràng macro `panic!`. Trong cả hai trường hợp, chúng ta gây ra một panic trong chương trình.
Theo mặc định, những panic này sẽ in một thông báo lỗi, unwind, dọn dẹp stack và thoát.
Thông qua một biến môi trường, bạn cũng có thể yêu cầu Rust hiển thị call stack khi
một panic xảy ra để dễ dàng theo dõi nguồn của panic.

> ### Unwinding Stack hoặc Aborting khi Panic Xảy Ra
>
> Theo mặc định, khi một panic xảy ra, chương trình bắt đầu _unwinding_, có nghĩa là
> Rust sẽ quay lại stack và dọn dẹp dữ liệu từ mỗi function mà nó gặp.
> Tuy nhiên, quay lại và dọn dẹp là một công việc rất nặng. Rust
> do đó cho phép bạn chọn cách khác là _aborting_ ngay lập tức,
> điều này kết thúc chương trình mà không dọn dẹp.
>
> Bộ nhớ mà chương trình đang sử dụng sẽ cần phải được dọn dẹp bởi
> hệ điều hành. Nếu trong dự án của bạn bạn cần làm cho binary kết quả càng nhỏ
> càng tốt, bạn có thể chuyển từ unwinding sang aborting khi panic xảy ra bằng cách
> thêm `panic = 'abort'` vào các `[profile]` sections thích hợp trong tệp
> _Cargo.toml_ của bạn. Ví dụ, nếu bạn muốn abort on panic ở release mode,
> hãy thêm cái này:
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

Let’s try calling `panic!` in a simple program:

<Listing file-name="src/main.rs">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-01-panic/src/main.rs}}
```

</Listing>

Khi bạn chạy chương trình, bạn sẽ thấy cái gì đó như thế này:

```console
{{#include ../listings/ch09-error-handling/no-listing-01-panic/output.txt}}
```

Lệnh gọi `panic!` gây ra thông báo lỗi có chứa hai dòng cuối cùng.
Dòng đầu tiên hiển thị thông báo panic của chúng ta và nơi trong source code của chúng ta
mà panic đã xảy ra: _src/main.rs:2:5_ chỉ ra rằng đó là dòng thứ hai,
ký tự thứ năm của tệp _src/main.rs_ của chúng ta.

Trong trường hợp này, dòng được chỉ ra là một phần của code của chúng ta, và nếu chúng ta
đi đến dòng đó, chúng ta sẽ thấy lệnh gọi macro `panic!`. Trong các trường hợp khác, lệnh gọi
`panic!` có thể nằm trong code mà code của chúng ta gọi, và tên file và số dòng được báo cáo
bởi thông báo lỗi sẽ là code của ai đó khác nơi macro `panic!` được gọi, không phải là
dòng code của chúng ta cuối cùng dẫn đến lệnh gọi `panic!`.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-a-panic-backtrace"></a>

Chúng ta có thể sử dụng backtrace của các function mà lệnh gọi `panic!` đến từ để hình dung
phần nào trong code của chúng ta đang gây ra vấn đề. Để hiểu cách sử dụng
một backtrace `panic!`, hãy xem xét một ví dụ khác và xem điều gì sẽ xảy ra khi
một lệnh gọi `panic!` đến từ một library vì có một bug trong code của chúng ta thay vì
từ code của chúng ta gọi macro trực tiếp. Listing 9-1 có một số code
cố gắng truy cập một index trong vector vượt quá phạm vi các index hợp lệ.

<Listing number="9-1" file-name="src/main.rs" caption="Attempting to access an element beyond the end of a vector, which will cause a call to `panic!`">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-01/src/main.rs}}
```

</Listing>

Ở đây, chúng ta đang cố gắng truy cập phần tử thứ 100 của vector của chúng ta (nó nằm ở
index 99 vì indexing bắt đầu từ zero), nhưng vector chỉ có ba
phần tử. Trong tình huống này, Rust sẽ panic. Sử dụng `[]` được cho là sẽ trả về
một phần tử, nhưng nếu bạn truyền một index không hợp lệ, không có phần tử nào mà Rust
có thể trả về ở đây sẽ là chính xác.

Trong C, cố gắng đọc vượt quá phạm vi kết thúc của một cấu trúc dữ liệu là undefined
behavior. Bạn có thể nhận được bất cứ gì ở vị trí trong bộ nhớ sẽ
tương ứng với phần tử đó trong cấu trúc dữ liệu, ngay cả khi bộ nhớ
không thuộc về cấu trúc đó. Đây được gọi là _buffer overread_ và có thể
dẫn đến các lỗ hổng bảo mật nếu kẻ tấn công có khả năng thao tác index
theo cách đó để đọc dữ liệu mà họ không được phép đọc được lưu trữ sau
cấu trúc dữ liệu.

Để bảo vệ chương trình của bạn khỏi loại lỗ hổng này, nếu bạn cố gắng đọc một
phần tử ở một index không tồn tại, Rust sẽ dừng thực thi và từ chối
tiếp tục. Hãy thử nó và xem:

```console
{{#include ../listings/ch09-error-handling/listing-09-01/output.txt}}
```

Lỗi này chỉ đến dòng 4 của _main.rs_ của chúng ta nơi chúng ta cố gắng truy cập index
99 của vector trong `v`.

Dòng `note:` cho chúng ta biết rằng chúng ta có thể đặt biến môi trường
`RUST_BACKTRACE` để lấy một backtrace của chính xác những gì đã xảy ra để gây ra lỗi. Một
_backtrace_ là một danh sách của tất cả các function đã được gọi để đến được
điểm này. Backtraces trong Rust hoạt động như chúng làm trong các ngôn ngữ khác: Chìa khóa để
đọc backtrace là bắt đầu từ đầu và đọc cho đến khi bạn thấy các file mà bạn
đã viết. Đó là chỗ nơi vấn đề bắt nguồn. Các dòng phía trên chỗ đó
là code mà code của bạn đã gọi; các dòng phía dưới là code đã gọi code của bạn.
Những dòng trước-và-sau này có thể bao gồm code Rust core, mã
thư viện chuẩn, hoặc các crates mà bạn đang sử dụng. Hãy cố gắng lấy một backtrace bằng cách
đặt biến môi trường `RUST_BACKTRACE` thành bất kỳ giá trị nào ngoại trừ `0`.
Listing 9-2 hiển thị output tương tự như cái bạn sẽ thấy.

<!-- manual-regeneration
cd listings/ch09-error-handling/listing-09-01
RUST_BACKTRACE=1 cargo run
copy the backtrace output below
check the backtrace number mentioned in the text below the listing
-->

<Listing number="9-2" caption="The backtrace generated by a call to `panic!` displayed when the environment variable `RUST_BACKTRACE` is set">

```console
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
stack backtrace:
   0: rust_begin_unwind
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/std/src/panicking.rs:692:5
   1: core::panicking::panic_fmt
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:75:14
   2: core::panicking::panic_bounds_check
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:273:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:274:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:16:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/alloc/src/vec/mod.rs:3361:9
   6: panic::main
             at ./src/main.rs:4:6
   7: core::ops::function::FnOnce::call_once
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

</Listing>

Đó là rất nhiều output! Output chính xác mà bạn thấy có thể khác nhau tùy thuộc
vào hệ điều hành và phiên bản Rust của bạn. Để lấy backtraces với
thông tin này, debug symbols phải được bật. Debug symbols được bật
theo mặc định khi sử dụng `cargo build` hoặc `cargo run` mà không có cờ `--release`,
như chúng ta ở đây.

Trong output ở Listing 9-2, dòng 6 của backtrace chỉ đến dòng trong
dự án của chúng ta đang gây ra vấn đề: dòng 4 của _src/main.rs_. Nếu chúng ta không muốn
chương trình của chúng ta panic, chúng ta nên bắt đầu điều tra của chúng ta tại vị trí được chỉ ra
bởi dòng đầu tiên đề cập đến một file mà chúng ta đã viết. Ở Listing 9-1, nơi chúng ta
có ý định viết code sẽ panic, cách để sửa panic là
không yêu cầu một phần tử ngoài phạm vi của các index vector. Khi code của bạn
panic trong tương lai, bạn sẽ cần phải tìm ra hành động nào code đang thực hiện
với những giá trị nào để gây ra panic và code nên làm gì thay vào đó.

Chúng ta sẽ quay lại với `panic!` và khi chúng ta nên và không nên sử dụng `panic!` để
xử lý các điều kiện lỗi trong phần [“To `panic!` or Not to
`panic!`”][to-panic-or-not-to-panic]<!-- ignore --> sau này trong
chương này. Tiếp theo, chúng ta sẽ xem cách phục hồi từ một lỗi sử dụng `Result`.

[to-panic-or-not-to-panic]: ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic
