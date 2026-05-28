## Lỗi Không Thể Phục Hồi với `panic!`

Đôi khi những điều xấu xảy ra trong mã của bạn, và bạn không thể làm gì cả. Trong những trường hợp này, Rust cung cấp macro `panic!`. Có hai cách để gây ra panic trong thực tế: thực hiện một hành động khiến mã của chúng ta panic (chẳng hạn như truy cập mảng vượt quá giới hạn) hoặc gọi macro `panic!` một cách rõ ràng. Trong cả hai trường hợp, chúng ta gây ra panic trong chương trình của mình. Mặc định, các panic này sẽ in ra một tin nhắn lỗi, giải nén stack, dọn dẹp stack, và thoát. Thông qua một biến môi trường, bạn cũng có thể yêu cầu Rust hiển thị call stack khi panic xảy ra để dễ dàng theo dõi nguồn gốc của panic.

> ### Giải Nén Stack hay Hủy Bỏ Khi Xảy Ra Panic
>
> Mặc định, khi panic xảy ra, chương trình bắt đầu _giải nén_ (unwinding), điều này có nghĩa là Rust sẽ lùi lại trên stack và dọn dẹp dữ liệu từ mỗi function mà nó gặp. Tuy nhiên, lùi lại và dọn dẹp là rất nhiều công việc. Vì vậy, Rust cho phép bạn chọn giải pháp thay thế là _hủy bỏ_ (abort) ngay lập tức, điều này sẽ kết thúc chương trình mà không cần dọn dẹp.
>
> Bộ nhớ mà chương trình đang sử dụng sẽ cần được dọn dẹp bởi hệ điều hành. Nếu trong dự án của bạn bạn cần làm cho binary kết quả nhỏ nhất có thể, bạn có thể chuyển từ unwinding sang abort khi panic bằng cách thêm `panic = ‘abort’` vào các phần `[profile]` thích hợp trong file _Cargo.toml_ của bạn. Ví dụ, nếu bạn muốn abort on panic ở release mode, hãy thêm cái này:
>
> ```toml
> [profile.release]
> panic = ‘abort’
> ```

Hãy thử gọi `panic!` trong một chương trình đơn giản:

<Listing file-name="src/main.rs">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-01-panic/src/main.rs}}
```

</Listing>

Khi bạn chạy chương trình, bạn sẽ thấy điều gì đó như thế này:

```console
{{#include ../listings/ch09-error-handling/no-listing-01-panic/output.txt}}
```

Lệnh gọi `panic!` gây ra thông báo lỗi được chứa trong hai dòng cuối cùng. Dòng đầu tiên hiển thị thông báo panic của chúng ta và nơi trong mã nguồn của chúng ta nơi panic xảy ra: _src/main.rs:2:5_ chỉ ra rằng đó là dòng thứ hai, ký tự thứ năm của file _src/main.rs_ của chúng ta.

Trong trường hợp này, dòng được chỉ định là một phần của mã của chúng ta, và nếu chúng ta đi đến dòng đó, chúng ta sẽ thấy lệnh gọi macro `panic!`. Trong những trường hợp khác, lệnh gọi `panic!` có thể nằm trong mã mà mã của chúng ta gọi, và tên file và số dòng được báo cáo bởi thông báo lỗi sẽ là mã của ai đó khác nơi macro `panic!` được gọi, không phải dòng mã của chúng ta dẫn đến lệnh gọi `panic!`.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-a-panic-backtrace"></a>

Chúng ta có thể sử dụng backtrace của các function mà lệnh gọi `panic!` đến từ để tìm ra phần mã của chúng ta gây ra sự cố. Để hiểu cách sử dụng backtrace `panic!`, hãy xem xét một ví dụ khác và xem điều gì sẽ xảy ra khi lệnh gọi `panic!` xuất phát từ một thư viện vì lỗi trong mã của chúng ta thay vì từ mã của chúng ta gọi macro trực tiếp. Listing 9-1 có một số mã cố gắng truy cập một chỉ mục trong vector vượt quá phạm vi của các chỉ mục hợp lệ.

<Listing number="9-1" file-name="src/main.rs" caption="Cố gắng truy cập một phần tử ngoài phạm vi của vector, điều này sẽ gây ra lệnh gọi `panic!`">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-01/src/main.rs}}
```

</Listing>

Ở đây, chúng ta đang cố gắng truy cập phần tử thứ 100 của vector của chúng ta (nằm ở chỉ mục 99 vì chỉ mục bắt đầu từ 0), nhưng vector chỉ có ba phần tử. Trong tình huống này, Rust sẽ panic. Sử dụng `[]` được cho là sẽ trả về một phần tử, nhưng nếu bạn truyền một chỉ mục không hợp lệ, sẽ không có phần tử nào mà Rust có thể trả về tại đây sẽ là đúng.

Trong C, cố gắng đọc vượt quá cuối của một cấu trúc dữ liệu là hành vi không xác định. Bạn có thể nhận được bất kỳ điều gì ở vị trí trong bộ nhớ sẽ tương ứng với phần tử đó trong cấu trúc dữ liệu, ngay cả khi bộ nhớ không thuộc về cấu trúc đó. Điều này được gọi là _buffer overread_ và có thể dẫn đến các lỗ hổng bảo mật nếu kẻ tấn công có khả năng thao tác chỉ mục theo cách để đọc dữ liệu mà họ không được phép đọc được lưu trữ sau cấu trúc dữ liệu.

Để bảo vệ chương trình của bạn khỏi loại lỗ hổng này, nếu bạn cố gắng đọc một phần tử ở một chỉ mục không tồn tại, Rust sẽ dừng thực thi và từ chối tiếp tục. Hãy thử nó và xem:

```console
{{#include ../listings/ch09-error-handling/listing-09-01/output.txt}}
```

Lỗi này chỉ vào dòng 4 của _main.rs_ của chúng ta nơi chúng ta cố gắng truy cập chỉ mục 99 của vector trong `v`.

Dòng `note:` cho chúng ta biết rằng chúng ta có thể đặt biến môi trường `RUST_BACKTRACE` để lấy backtrace của những gì chính xác đã xảy ra để gây ra lỗi. _Backtrace_ là danh sách tất cả các function đã được gọi để đến được điểm này. Backtrace trong Rust hoạt động như trong các ngôn ngữ khác: Chìa khóa để đọc backtrace là bắt đầu từ đầu và đọc cho đến khi bạn thấy các file bạn đã viết. Đó là nơi mà sự cố bắt nguồn. Các dòng ở trên nơi đó là mã mà mã của bạn đã gọi; các dòng ở dưới là mã gọi mã của bạn. Những dòng trước và sau này có thể bao gồm mã Rust cốt lõi, mã thư viện tiêu chuẩn, hoặc các crate bạn đang sử dụng. Hãy thử lấy backtrace bằng cách đặt biến môi trường `RUST_BACKTRACE` thành bất kỳ giá trị nào ngoài `0`. Listing 9-2 hiển thị đầu ra tương tự như những gì bạn sẽ thấy.

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

Đó là rất nhiều đầu ra! Đầu ra chính xác mà bạn thấy có thể khác nhau tùy thuộc vào hệ điều hành và phiên bản Rust của bạn. Để lấy backtrace với thông tin này, các ký hiệu gỡ lỗi phải được bật. Các ký hiệu gỡ lỗi được bật theo mặc định khi sử dụng `cargo build` hoặc `cargo run` mà không có cờ `--release`, như chúng tôi đang làm ở đây.

Trong đầu ra trong Listing 9-2, dòng 6 của backtrace chỉ vào dòng trong dự án của chúng ta gây ra sự cố: dòng 4 của _src/main.rs_. Nếu chúng ta không muốn chương trình panic, chúng ta nên bắt đầu cuộc điều tra của mình tại vị trí được chỉ bởi dòng đầu tiên đề cập đến một file chúng ta đã viết. Trong Listing 9-1, nơi chúng ta cố ý viết mã sẽ panic, cách để khắc phục panic là không yêu cầu một phần tử vượt quá phạm vi của các chỉ mục vector. Khi mã của bạn panic trong tương lai, bạn sẽ cần tìm ra hành động nào mà mã đang thực hiện với những giá trị nào để gây ra panic và mã nên làm gì thay vào đó.

Chúng ta sẽ quay lại `panic!` và khi chúng ta nên và không nên sử dụng `panic!` để xử lý các tình huống lỗi trong phần [“Có nên `panic!` hay Không”][to-panic-or-not-to-panic]<!-- ignore --> ở phía sau trong chương này. Tiếp theo, chúng ta sẽ xem cách phục hồi từ một lỗi bằng cách sử dụng `Result`.

[to-panic-or-not-to-panic]: ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic
