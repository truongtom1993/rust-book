## Chấp nhận Command Line Arguments

Hãy tạo một project mới bằng `cargo new` như thường lệ. Chúng ta sẽ đặt tên project là
`minigrep` để phân biệt với tool `grep` mà bạn có thể đã có sẵn trên hệ thống của mình. 

```console
$ cargo new minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

Nhiệm vụ đầu tiên là làm cho `minigrep` nhận hai command line argument: đường dẫn
file và chuỗi cần tìm. Nghĩa là, chúng ta muốn có thể chạy chương trình bằng
`cargo run`, hai dấu gạch ngang để chỉ ra rằng các argument phía sau là dành cho
chương trình của chúng ta chứ không phải cho `cargo`, một chuỗi cần tìm, và đường
dẫn tới file cần tìm kiếm, như sau: 

```console
$ cargo run -- searchstring example-filename.txt
```

Hiện tại, chương trình được tạo bởi `cargo new` chưa thể xử lý các argument mà
chúng ta truyền vào. Có một số library sẵn có trên [crates.io](https://crates.io/)
có thể hỗ trợ viết chương trình nhận command line arguments, nhưng vì bạn mới học
concept này, chúng ta sẽ tự implement khả năng đó. 

### Đọc giá trị argument

Để `minigrep` có thể đọc các giá trị command line arguments được truyền vào, chúng ta
cần dùng function `std::env::args` do standard library của Rust cung cấp. Function này
trả về một iterator của các command line arguments được truyền cho `minigrep`. 

Chúng ta sẽ học kỹ hơn về iterator ở [Chapter 13][ch13]. Tạm thời, bạn chỉ cần biết hai
điểm về iterator: iterator tạo ra một chuỗi các giá trị, và chúng ta có thể gọi method
`collect` trên iterator để biến nó thành một collection, chẳng hạn như vector, chứa tất
cả phần tử mà iterator tạo ra. 

Đoạn code trong Listing 12-1 cho phép chương trình `minigrep` đọc mọi command line
arguments được truyền vào, sau đó collect các giá trị đó vào một vector. 

<Listing number="12-1" file-name="src/main.rs" caption="Collect các command line arguments vào một vector và in chúng ra">

```rust
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-01/src/main.rs}}
```

</Listing>

Đầu tiên, chúng ta đưa module `std::env` vào scope bằng một `use` statement để có thể
dùng function `args` của nó. Lưu ý rằng function `std::env::args` nằm lồng trong hai
cấp module. 

Như đã thảo luận ở [Chapter 7][ch7-idiomatic-use], trong những trường hợp function mong
muốn nằm trong nhiều hơn một module, chúng ta chọn đưa parent module vào scope thay vì
đưa trực tiếp function vào. Làm như vậy giúp ta dễ dàng dùng các function khác từ
`std::env`, đồng thời cũng bớt mơ hồ hơn so với việc viết `use std::env::args` rồi chỉ
gọi function là `args`, vì `args` rất dễ bị nhầm với một function được định nghĩa trong
module hiện tại. 

> ### Function `args` và Unicode không hợp lệ
>
> Lưu ý rằng `std::env::args` sẽ panic nếu có bất kỳ argument nào chứa Unicode không hợp lệ. 
>
> Nếu chương trình của bạn cần chấp nhận các argument chứa Unicode không hợp lệ, hãy dùng
> `std::env::args_os` thay thế. Function đó trả về một iterator tạo ra các giá trị
> `OsString` thay vì `String`. [doc.rust-lang](https://doc.rust-lang.org/std/env/fn.args_os.html)
>
> Ở đây chúng ta chọn dùng `std::env::args` cho đơn giản, vì `OsString` khác nhau tùy
> platform và làm việc với nó cũng phức tạp hơn `String`. 

Ở dòng đầu tiên của `main`, chúng ta gọi `env::args`, rồi ngay lập tức dùng `collect`
để biến iterator thành một vector chứa tất cả các giá trị do iterator tạo ra. 

Chúng ta có thể dùng function `collect` để tạo ra nhiều loại collection khác nhau, nên
chúng ta annotate type của `args` một cách tường minh để chỉ rõ rằng mình muốn một
vector chuỗi. Dù trong Rust bạn rất hiếm khi phải annotate type, `collect` là một trong
những trường hợp mà bạn thường cần làm vậy, vì Rust không thể suy luận được loại
collection mà bạn muốn. 

Cuối cùng, chúng ta in vector ra bằng debug macro. Hãy thử chạy đoạn code trước với
không có argument nào, rồi sau đó với hai argument. 

```console
{{#include ../listings/ch12-an-io-project/listing-12-01/output.txt}}
```

```console
{{#include ../listings/ch12-an-io-project/output-only-01-with-args/output.txt}}
```

Lưu ý rằng giá trị đầu tiên trong vector là `"target/debug/minigrep"`, chính là tên của
binary. Điều này giống với cách danh sách arguments hoạt động trong C, cho phép chương
trình dùng tên mà nó được gọi tới trong quá trình thực thi. 

Việc có quyền truy cập vào tên chương trình thường khá tiện, ví dụ khi bạn muốn in nó ra
trong message hoặc thay đổi hành vi chương trình tùy theo command line alias nào đã được
dùng để gọi chương trình. Nhưng trong phạm vi chương này, chúng ta sẽ bỏ qua nó và chỉ
lưu lại hai argument mà mình cần. 

### Lưu giá trị argument vào variable

Hiện tại chương trình đã có thể truy cập các giá trị được truyền qua command line
arguments. Bây giờ chúng ta cần lưu hai giá trị đó vào variable để có thể dùng chúng
xuyên suốt phần còn lại của chương trình. Chúng ta sẽ làm điều đó trong Listing 12-2. 

<Listing number="12-2" file-name="src/main.rs" caption="Tạo các variable để giữ query argument và file path argument">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-02/src/main.rs}}
```

</Listing>

Như chúng ta đã thấy khi in vector ra, tên chương trình chiếm giá trị đầu tiên trong
vector tại `args[0]`, nên chúng ta sẽ bắt đầu lấy các argument từ index 1. 

Argument đầu tiên mà `minigrep` nhận là chuỗi cần tìm, nên chúng ta đặt một reference tới
argument đầu tiên vào variable `query`. Argument thứ hai sẽ là đường dẫn file, nên chúng
ta đặt một reference tới argument thứ hai vào variable `file_path`. 

Chúng ta tạm thời in giá trị của các variable này ra để chứng minh rằng code đang hoạt
động đúng như mong muốn. Hãy chạy lại chương trình với các argument `test` và
`sample.txt`. 

```console
{{#include ../listings/ch12-an-io-project/listing-12-02/output.txt}}
```

Tuyệt vời, chương trình đã hoạt động! Các giá trị argument mà chúng ta cần đã được lưu
vào đúng variable. Ở phần sau, chúng ta sẽ thêm error handling để xử lý một số tình huống
có thể xảy ra, chẳng hạn khi user không truyền argument nào; còn bây giờ, chúng ta sẽ tạm
bỏ qua trường hợp đó và tiếp tục thêm khả năng đọc file. 

[ch13]: ch13-00-functional-features.html
[ch7-idiomatic-use]: ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths