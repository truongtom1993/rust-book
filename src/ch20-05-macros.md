## Macros

Chúng ta đã sử dụng những macros như `println!` trong suốt quyển sách này, nhưng chúng ta không hoàn toàn khám phá những gì một macro là và cách nó hoạt động. Thuật ngữ _macro_ đề cập đến một gia đình của những tính năng trong Rust—những declarative macros với `macro_rules!` và ba loại của những procedural macros:

- Custom `#[derive]` macros mà chỉ định code được thêm vào với `derive` attribute được sử dụng trên structs và enums
- Attribute-like macros mà định nghĩa những custom attributes có thể sử dụng được trên bất kỳ item nào
- Function-like macros mà trông giống như những function calls nhưng hoạt động trên những tokens được chỉ định như argument của chúng

Chúng ta sẽ nói về mỗi loại này lần lượt, nhưng trước tiên, chúng ta hãy xem xét tại sao chúng ta thậm chí cần macros khi chúng ta đã có những functions.

### The Difference Between Macros and Functions

Về cơ bản, macros là một cách viết code mà viết code khác, mà được biết như _metaprogramming_. Trong Appendix C, chúng ta thảo luận `derive` attribute, mà tạo một triển khai của những traits khác nhau cho bạn. Chúng ta cũng đã sử dụng những macros `println!` và `vec!` trong suốt quyển sách. Tất cả những macros này _expand_ để tạo thêm code hơn code bạn đã viết thủ công.

Metaprogramming hữu ích để giảm lượng code bạn phải viết và duy trì, đó là cũng một trong những vai trò của functions. Tuy nhiên, macros có một số quyền hạn bổ sung mà functions không có.

Một function signature phải khai báo số và loại của parameters function có. Macros, mặt khác, có thể lấy một số variable của parameters: Chúng ta có thể gọi `println!("hello")` với một argument hoặc `println!("hello {}", name)` với hai arguments. Cũng, macros được mở rộng trước khi trình biên dịch diễn giải ý nghĩa của code, vì vậy một macro có thể, ví dụ, triển khai một trait trên một loại nhất định. Một function không thể, vì nó được gọi tại runtime và một trait cần được triển khai tại thời gian compile.

Những nhược điểm để triển khai một macro thay vì một function là những định nghĩa macro phức tạp hơn những định nghĩa function vì bạn đang viết Rust code mà viết Rust code. Do đó, định nghĩa macro nói chung khó đọc, hiểu, và duy trì hơn những định nghĩa function.

Một sự khác biệt quan trọng khác giữa macros và functions là bạn phải định nghĩa macros hoặc mang chúng vào scope _trước_ khi bạn gọi chúng trong một file, không giống những functions bạn có thể định nghĩa ở bất cứ đâu và gọi ở bất cứ đâu.

<!-- Old headings. Do not remove or links may break. -->

<a id="declarative-macros-with-macro_rules-for-general-metaprogramming"></a>

### Declarative Macros for General Metaprogramming

Loại macros được sử dụng rộng rãi nhất trong Rust là _declarative macro_. Những cái này cũng được đôi khi gọi là "macros bằng ví dụ," "`macro_rules!` macros," hoặc chỉ "macros" đơn giản. Tại lõi của chúng, những declarative macros cho phép bạn viết những cái gì tương tự như Rust `match` expression. Như được thảo luận trong Chương 6, những `match` expressions là những cấu trúc kiểm soát lấy một expression, so sánh những kết quả giá trị của expression với những patterns, và sau đó chạy code được liên kết với những pattern khớp. Macros cũng so sánh một giá trị với những patterns mà được liên kết với code cụ thể: Trong tình huống này, giá trị là những Rust source code literal được chuyển cho macro; những patterns được so sánh với cấu trúc của mã nguồn đó; và code được liên kết với mỗi pattern, khi matched, thay thế code được chuyển cho macro. Tất cả điều này xảy ra trong suốt quá trình compilation.

Để định nghĩa một macro, bạn sử dụng những `macro_rules!` construct. Chúng ta hãy khám phá cách sử dụng `macro_rules!` bằng cách xem xét cách những `vec!` macro được định nghĩa. Chương 8 đề cập đến cách chúng ta có thể sử dụng macro `vec!` để tạo một vector mới với những giá trị cụ thể. Ví dụ, những macro sau tạo một vector mới chứa ba integers:

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

Chúng ta có thể cũng sử dụng macro `vec!` để tạo một vector của hai integers hoặc một vector của năm string slices. Chúng ta sẽ không thể sử dụng một function để làm được những cái gì tương tự vì chúng ta sẽ không biết số hoặc loại những giá trị trước.

Listing 20-35 cho thấy một định nghĩa hơi đơn giản hóa của macro `vec!`.

<Listing number="20-35" file-name="src/lib.rs" caption="A simplified version of the `vec!` macro definition">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-35/src/lib.rs}}
```

</Listing>

> Lưu ý: Những định nghĩa thực tế của macro `vec!` trong thư viện chuẩn bao gồm code để pre-allocate lượng đúng của bộ nhớ trước. Mã này là một tối ưu mà chúng ta không bao gồm ở đây, để làm ví dụ đơn giản hơn.

Annotation `#[macro_export]` chỉ ra rằng macro này sẽ được làm sẵn sàng bất cứ khi nào crate mà macro được định nghĩa trong được mang vào scope. Không có annotation này, macro không thể được mang vào scope.

Chúng ta sau đó bắt đầu định nghĩa macro với `macro_rules!` và tên của macro chúng ta định nghĩa _mà không có_ những dấu chấm than. Tên, trong trường hợp này `vec`, được theo sau bởi những dấu ngoặc nhọn biểu thị những phần thân của những định nghĩa macro.

Cấu trúc trong những phần thân `vec!` tương tự như cấu trúc của một `match` expression. Ở đây chúng ta có một arm với những pattern `( $( $x:expr ),* )`, được theo sau bởi `=>` và những block của code được liên kết với những patterns này. Nếu pattern khớp, những associated block của code sẽ được phát ra. Với mục đích rằng đây là những pattern duy nhất trong những macro này, có chỉ một cách hợp lệ để match; bất kỳ pattern khác sẽ kết quả trong một error. Những macros phức tạp hơn sẽ có thêm hơn một arm.

Syntax pattern hợp lệ trong những định nghĩa macro khác với những pattern syntax được đề cập trong Chương 19 vì những macro patterns được matched chống lại cấu trúc Rust code chứ không những giá trị. Hãy đi qua những pattern pieces trong Listing 20-29 có nghĩa là gì; với những macro pattern syntax đầy đủ, xem ["Rust Reference"][ref].

Đầu tiên, chúng ta sử dụng một tập hợp những dấu ngoặc đơn để bao quanh tất cả những pattern. Chúng ta sử dụng một dấu dollar (`$`) để khai báo một biến trong hệ thống macro mà sẽ chứa những Rust code khớp với những pattern. Những dấu dollar làm nó rõ ràng rằng đây là một biến macro đối với một biến Rust thường xuyên. Tiếp theo là một tập hợp những dấu ngoặc đơn mà captures những giá trị mà khớp với những pattern trong những dấu ngoặc đơn để sử dụng trong code thay thế. Trong `$()` là `$x:expr`, mà khớp bất kỳ Rust expression nào và cung cấp expression những tên `$x`.

Những dấu phẩy sau `$()` chỉ ra rằng một ký tự dấu phẩy phân tách literal phải xuất hiện giữa mỗi instance của code mà khớp với code trong `$()`. `*` chỉ ra rằng những pattern khớp không hoặc hơn của bất kỳ những gì trước đó `*`.

Khi chúng ta gọi những macro này với `vec![1, 2, 3];`, `$x` pattern khớp ba lần với ba expressions `1`, `2`, và `3`.

Bây giờ chúng ta hãy xem xét những pattern trong những phần thân của code được liên kết với những arm này: `temp_vec.push()` trong `$()*` được tạo cho mỗi phần mà khớp `$()` trong những pattern không hoặc hơn lần tùy thuộc vào bao nhiêu lần pattern khớp. `$x` được thay thế với mỗi expression matched. Khi chúng ta gọi những macro này với `vec![1, 2, 3];`, code được tạo mà thay thế lệnh gọi macro này sẽ là những sau:

```rust,ignore
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

Chúng ta đã định nghĩa một macro mà có thể lấy bất kỳ số arguments của bất kỳ loại nào và có thể tạo code để tạo một vector chứa những elements được chỉ định.

Để học về cách viết macros thêm, hãy tham khảo những tài liệu trực tuyến hoặc những tài nguyên khác, chẳng hạn như ["The Little Book of Rust Macros"][tlborm] được bắt đầu bởi Daniel Keep và tiếp tục bởi Lukas Wirth.

### Procedural Macros for Generating Code from Attributes

Những hình thức thứ hai của macros là những procedural macro, mà hoạt động hơn như một function (và là một loại những thủ tục). _Procedural macros_ chấp nhận một số code như một đầu vào, hoạt động trên code đó, và tạo một số code như một đầu ra thay vì matching chống lại những patterns và thay thế code với code khác như những declarative macros làm. Những ba loại những procedural macros là custom `derive`, attribute-like, và function-like, và tất cả công việc trong một cách tương tự.

Khi tạo những procedural macros, những định nghĩa phải nằm trong những crate của riêng chúng với một loại crate đặc biệt. Điều này là cho những lý do kỹ thuật phức tạp mà chúng ta hy vọng để loại bỏ trong những tương lai. Trong Listing 20-36, chúng ta chỉ ra cách để định nghĩa một procedural macro, nơi `some_attribute` là một placeholder cho sử dụng một lựa chọn macro cụ thể.

<Listing number="20-36" file-name="src/lib.rs" caption="An example of defining a procedural macro">

```rust,ignore
use proc_macro::TokenStream;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

</Listing>

Function mà định nghĩa một procedural macro lấy một `TokenStream` như một đầu vào và tạo một `TokenStream` như một đầu ra. Loại `TokenStream` được định nghĩa bởi crate `proc_macro` mà được bao gồm với Rust và đại diện cho một chuỗi của những tokens. Đây là những lõi của macro: Những source code mà macro hoạt động trên tạo những input `TokenStream`, và code mà macro tạo là những output `TokenStream`. Function cũng có một attribute được đính kèm với nó mà chỉ định những loại nào của procedural macro chúng ta đang tạo. Chúng ta có thể có những loại nhiều của những procedural macros trong crate tương tự.

Chúng ta hãy xem xét những loại khác nhau của những procedural macros. Chúng ta sẽ bắt đầu với một macro `derive` tùy chỉnh và sau đó giải thích những sự khác biệt nhỏ mà làm những hình thức khác khác biệt.

<!-- Old headings. Do not remove or links may break. -->

<a id="how-to-write-a-custom-derive-macro"></a>

### Custom `derive` Macros

Chúng ta hãy tạo một crate được đặt tên là `hello_macro` mà định nghĩa một trait được đặt tên là `HelloMacro` với một associated function được đặt tên là `hello_macro`. Thay vì làm những người dùng của chúng ta triển khai trait `HelloMacro` cho mỗi loại của chúng, chúng ta sẽ cung cấp một procedural macro để những người dùng có thể chú thích loại của chúng với `#[derive(HelloMacro)]` để có một triển khai mặc định của function `hello_macro`. Những triển khai mặc định sẽ in `Hello, Macro! My name is TypeName!` nơi `TypeName` là tên của những loại trên đó trait này được định nghĩa. Nói cách khác, chúng ta sẽ viết một crate mà cho phép một lập trình viên khác để viết code như Listing 20-37 sử dụng crate của chúng ta.

<Listing number="20-37" file-name="src/main.rs" caption="The code a user of our crate will be able to write when using our procedural macro">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-37/src/main.rs}}
```

</Listing>

Code này sẽ in `Hello, Macro! My name is Pancakes!` khi chúng ta hoàn tất. Những bước đầu tiên là để tạo một crate thư viện mới, như thế này:

```console
$ cargo new hello_macro --lib
```

Tiếp theo, trong Listing 20-38, chúng ta sẽ định nghĩa `HelloMacro` trait và associated function của nó.

<Listing file-name="src/lib.rs" number="20-38" caption="A simple trait that we will use with the `derive` macro">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-38/hello_macro/src/lib.rs}}
```

</Listing>

Chúng ta có một trait và function của nó. Tại điểm này, những người dùng crate của chúng ta có thể triển khai trait để đạt được chức năng mong muốn, như trong Listing 20-39.

<Listing number="20-39" file-name="src/main.rs" caption="How it would look if users wrote a manual implementation of the `HelloMacro` trait">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-39/pancakes/src/main.rs}}
```

</Listing>

Tuy nhiên, họ sẽ cần để viết những implementation block cho mỗi loại họ muốn để sử dụng với `hello_macro`; chúng ta muốn để tiết kiệm cho họ từ việc phải làm công việc này.

Ngoài ra, chúng ta không thể nhưng cung cấp function `hello_macro` với triển khai mặc định mà sẽ in những tên của những loại mà trait được triển khai trên: Rust không có những khả năng reflection, vì vậy nó không thể xem lên tên của những loại tại runtime. Chúng ta cần một macro để tạo code tại thời gian compile.

Những bước tiếp theo là để định nghĩa những procedural macro. Tại thời điểm của bài viết này, những procedural macros cần phải nằm trong crate của riêng chúng. Cuối cùng, những hạn chế này có thể được nâng lên. Những quy ước cho cách cấu trúc những crates và những crates macro như sau: Cho một crate được đặt tên là `foo`, một crate procedural macro `derive` tùy chỉnh được gọi là `foo_derive`. Chúng ta hãy bắt đầu một crate mới được gọi là `hello_macro_derive` trong dự án `hello_macro` của chúng ta:

```console
$ cargo new hello_macro_derive --lib
```

Hai crates của chúng ta được có liên quan chặt chẽ, vì vậy chúng ta tạo những crate procedural macro trong những thư mục của crate `hello_macro` của chúng ta. Nếu chúng ta thay đổi những định nghĩa trait trong `hello_macro`, chúng ta sẽ phải thay đổi những triển khai của những procedural macro trong `hello_macro_derive` cũng. Hai crates sẽ cần để được xuất bản riêng biệt, và những lập trình viên sử dụng những crates này sẽ cần để thêm cả hai như những dependencies và mang cả hai vào scope. Chúng ta có thể thay vào đó có crate `hello_macro` sử dụng `hello_macro_derive` như một dependency và re-export những code procedural macro. Tuy nhiên, những cách chúng ta đã cấu trúc những dự án làm nó có thể cho những lập trình viên để sử dụng `hello_macro` thậm chí nếu chúng họ không muốn những chức năng `derive`.

Chúng ta cần để khai báo crate `hello_macro_derive` như một procedural macro crate. Chúng ta cũng sẽ cần chức năng từ những crates `syn` và `quote`, như bạn sẽ thấy trong một chút, vì vậy chúng ta cần để thêm chúng như những dependencies. Thêm những sau vào những _Cargo.toml_ file cho `hello_macro_derive`:

<Listing file-name="hello_macro_derive/Cargo.toml">

```toml
{{#include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/Cargo.toml:6:12}}
```

</Listing>

Để bắt đầu định nghĩa những procedural macro, đặt code trong Listing 20-40 vào những _src/lib.rs_ file của bạn cho crate `hello_macro_derive`. Lưu ý rằng code này sẽ không compile cho đến khi chúng ta thêm một định nghĩa cho những `impl_hello_macro` function.

<Listing number="20-40" file-name="hello_macro_derive/src/lib.rs" caption="Code that most procedural macro crates will require in order to process Rust code">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/src/lib.rs}}
```

</Listing>

Thông báo rằng chúng ta đã chia code thành những `hello_macro_derive` function, mà chịu trách nhiệm cho việc phân tích cú pháp những `TokenStream`, và những `impl_hello_macro` function, mà chịu trách nhiệm cho việc biến đổi những cây cú pháp: Điều này làm viết một procedural macro thuận tiện hơn. Code trong những outer function (`hello_macro_derive` trong trường hợp này) sẽ là những tương tự cho gần như mỗi procedural macro crate bạn thấy hoặc tạo. Code bạn chỉ định trong những phần thân của những inner function (`impl_hello_macro` trong trường hợp này) sẽ là khác nhau tùy thuộc vào mục đích của những procedural macro của bạn.

Chúng ta đã giới thiệu ba crates mới: `proc_macro`, [`syn`][syn]<!-- ignore -->, và [`quote`][quote]<!-- ignore -->. Crate `proc_macro` đi kèm với Rust, vì vậy chúng ta không cần phải thêm những điều đó vào những dependencies trong _Cargo.toml_. Crate `proc_macro` là những API của trình biên dịch mà cho phép chúng ta để đọc và thao tác Rust code từ code của chúng ta.

Crate `syn` phân tích cú pháp Rust code từ một string vào một cấu trúc dữ liệu mà chúng ta có thể thực hiện những hoạt động trên. Crate `quote` bật những cấu trúc dữ liệu `syn` quay trở lại vào Rust code. Những crates này làm nó đơn giản hơn nhiều để phân tích cú pháp bất kỳ số loại Rust code chúng ta có thể muốn để xử lý: Viết một parser đầy đủ cho Rust code không phải là một tác vụ đơn giản.

Function `hello_macro_derive` sẽ được gọi khi những người dùng của thư viện của chúng ta chỉ định `#[derive(HelloMacro)]` trên một loại. Điều này là có thể vì chúng ta đã chú thích function `hello_macro_derive` ở đây với `proc_macro_derive` và chỉ định những tên `HelloMacro`, mà khớp tên trait của chúng ta; đây là những quy ước mà hầu hết những procedural macros theo.

Function `hello_macro_derive` đầu tiên chuyển đổi những `input` từ một `TokenStream` để cấu trúc dữ liệu mà chúng ta có thể sau đó diễn giải và thực hiện những hoạt động trên. Đây là nơi `syn` đi vào. Function `parse` trong `syn` lấy một `TokenStream` và trả về một struct `DeriveInput` đại diện cho những code Rust được phân tích cú pháp. Listing 20-41 cho thấy những phần có liên quan của struct `DeriveInput` chúng ta nhận được từ việc phân tích cú pháp những string `struct Pancakes;`.

<Listing number="20-41" caption="The `DeriveInput` instance we get when parsing the code that has the macro's attribute in Listing 20-37">

```rust,ignore
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

</Listing>

Những fields của struct này chỉ ra rằng Rust code chúng ta đã phân tích cú pháp là một unit struct với những `ident` (_identifier_, có nghĩa là tên) của `Pancakes`. Có hơn những fields trên struct này cho việc mô tả tất cả các loại Rust code; kiểm tra những [`syn` documentation cho `DeriveInput`][syn-docs] để nhiều thông tin.

Sớm chúng ta sẽ định nghĩa function `impl_hello_macro`, mà là nơi chúng ta sẽ xây dựng Rust code mới chúng ta muốn để bao gồm. Nhưng trước khi chúng ta làm, lưu ý rằng những output cho macro `derive` của chúng ta cũng là một `TokenStream`. `TokenStream` được trả về được thêm vào code mà những người dùng crate của chúng ta viết, vì vậy khi chúng họ compile crate của chúng, họ sẽ nhận được những chức năng bổ sung mà chúng ta cung cấp trong những `TokenStream` được sửa đổi.

Bạn có thể đã chú ý rằng chúng ta đang gọi `unwrap` để gây ra function `hello_macro_derive` để panic nếu lệnh gọi cho function `syn::parse` không thành công ở đây. Nó là cần thiết cho procedural macro của chúng ta để panic trên errors vì những `proc_macro_derive` functions phải trả về `TokenStream` chứ không những `Result` để phù hợp với những API procedural macro. Chúng ta đã đơn giản hóa ví dụ này bằng cách sử dụng `unwrap`; trong code sản xuất, bạn nên cung cấp những thông báo error cụ thể hơn về những gì đã đi sai bằng cách sử dụng `panic!` hoặc `expect`.

Bây giờ chúng ta có code để bật những code Rust được chú thích từ một `TokenStream` vào một instance `DeriveInput`, chúng ta hãy tạo code mà triển khai trait `HelloMacro` trên loại được chú thích, như được hiển thị trong Listing 20-42.

<Listing number="20-42" file-name="hello_macro_derive/src/lib.rs" caption="Implementing the `HelloMacro` trait using the parsed Rust code">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-42/hello_macro/hello_macro_derive/src/lib.rs:here}}
```

</Listing>

Chúng ta nhận được một instance struct `Ident` chứa những tên (identifier) của loại được chú thích bằng cách sử dụng `ast.ident`. Struct trong Listing 20-41 cho thấy rằng khi chúng ta chạy function `impl_hello_macro` trên code trong Listing 20-37, `ident` chúng ta nhận được sẽ có những `ident` field với một giá trị của `"Pancakes"`. Do đó, biến `name` trong Listing 20-42 sẽ chứa một instance struct `Ident` mà, khi in, sẽ là những string `"Pancakes"`, những tên của struct trong Listing 20-37.

Macro `quote!` cho phép chúng ta định nghĩa Rust code mà chúng ta muốn để trả về. Trình biên dịch mong đợi cái gì khác từ những kết quả trực tiếp của sự thi hành macro `quote!`, vì vậy chúng ta cần để chuyển đổi nó để `TokenStream`. Chúng ta làm điều này bằng cách gọi `into` method, mà tiêu thụ những đại diện trung gian này và trả về một giá trị của loại `TokenStream` được yêu cầu.

Macro `quote!` cũng cung cấp một số cơ chế mẫu rất mát mẻ: Chúng ta có thể nhập `#name`, và `quote!` sẽ thay thế nó với giá trị trong biến `name`. Bạn thậm chí có thể làm một số lặp lại tương tự như cách những macros thông thường hoạt động. Kiểm tra những [`quote` crate's docs][quote-docs] cho một giới thiệu triệt để.

Chúng ta muốn những procedural macro của chúng ta để tạo một triển khai của trait `HelloMacro` của chúng ta cho loại mà người dùng chú thích, chúng ta có thể nhận được bằng cách sử dụng `#name`. Những triển khai trait có một function `hello_macro`, có phần thân chứa những chức năng chúng ta muốn để cung cấp: in `Hello, Macro! My name is` và sau đó những tên của loại được chú thích.

Macro `stringify!` được sử dụng ở đây là xây dựng trong Rust. Nó lấy một Rust expression, chẳng hạn như `1 + 2`, và tại thời gian compile bật expression thành một string literal, chẳng hạn như `"1 + 2"`. Điều này khác với `format!` hoặc `println!`, mà là những macros mà đánh giá expression và sau đó bật những kết quả vào một `String`. Có một khả năng mà những `#name` input có thể là một expression để in theo nghĩa đen, vì vậy chúng ta sử dụng `stringify!`. Sử dụng `stringify!` cũng tiết kiệm một sự phân bổ bằng cách chuyển đổi `#name` để một string literal tại thời gian compile.

Tại điểm này, `cargo build` nên hoàn thành một cách thành công trong cả hai `hello_macro` và `hello_macro_derive`. Chúng ta hãy kết nối những crates này cho code trong Listing 20-37 để thấy những procedural macro được hoạt động! Tạo một binary dự án mới trong thư mục _projects_ của bạn bằng cách sử dụng `cargo new pancakes`. Chúng ta cần để thêm `hello_macro` và `hello_macro_derive` như những dependencies trong `pancakes` crate's _Cargo.toml_. Nếu bạn đang xuất bản những phiên bản của `hello_macro` và `hello_macro_derive` để [crates.io](https://crates.io/)<!-- ignore -->, chúng sẽ là những regular dependencies; nếu không, bạn có thể chỉ định chúng như `path` dependencies như sau:

```toml
{{#include ../listings/ch20-advanced-features/no-listing-21-pancakes/pancakes/Cargo.toml:6:8}}
```

Đặt code trong Listing 20-37 vào _src/main.rs_, và chạy `cargo run`: Nó nên in `Hello, Macro! My name is Pancakes!`. Những triển khai của trait `HelloMacro` từ những procedural macro được bao gồm mà không cần crate `pancakes` để triển khai nó; những `#[derive(HelloMacro)]` thêm những triển khai trait.

Tiếp theo, chúng ta hãy khám phá cách những loại khác của những procedural macros khác từ những macros `derive` tùy chỉnh.

### Attribute-Like Macros

Attribute-like macros tương tự như những macros `derive` tùy chỉnh, nhưng thay vì tạo code cho `derive` attribute, họ cho phép bạn để tạo những attributes mới. Chúng cũng linh hoạt hơn: `derive` chỉ hoạt động cho structs và enums; những attributes có thể được áp dụng cho những items khác cũng như, chẳng hạn như những functions. Ở đây là một ví dụ của sử dụng một attribute-like macro. Nói rằng bạn có một attribute được đặt tên là `route` mà chú thích những functions khi sử dụng một framework ứng dụng web:

```rust,ignore
#[route(GET, "/")]
fn index() {
```

Attribute `#[route]` này sẽ được định nghĩa bởi những framework như một procedural macro. Những signature của những macro định nghĩa function sẽ trông giống như những cái này:

```rust,ignore
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

Ở đây, chúng ta có hai parameters của loại `TokenStream`. Những đầu tiên là dành cho những nội dung của attribute: `GET, "/"` phần. Những thứ hai là những phần thân của item những attribute được đính kèm đến: trong trường hợp này, `fn index() {}` và những còn lại của những phần thân function.

Khác hơn những điều đó, attribute-like macros hoạt động cùng cách như những macros `derive` tùy chỉnh: Bạn tạo một crate với loại crate `proc-macro` và triển khai một function mà tạo code bạn muốn!

### Function-Like Macros

Function-like macros định nghĩa những macros mà trông giống những function calls. Tương tự để `macro_rules!` macros, chúng linh hoạt hơn những functions; ví dụ, chúng có thể lấy một số unknown của arguments. Tuy nhiên, `macro_rules!` macros có thể chỉ được định nghĩa bằng cách sử dụng những match-like syntax chúng ta thảo luận trong phần ["Declarative Macros for General Metaprogramming"][decl]<!-- ignore --> trước. Function-like macros lấy một `TokenStream` parameter, và những định nghĩa của chúng thao tác những `TokenStream` bằng cách sử dụng Rust code như những loại khác của những procedural macros làm. Một ví dụ của một function-like macro là một macro `sql!` mà có thể được gọi như vậy:

```rust,ignore
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

Macro này sẽ phân tích cú pháp những SQL statement bên trong nó và kiểm tra rằng nó đúng cú pháp, mà là xử lý phức tạp hơn nhiều mà một `macro_rules!` macro có thể làm. Macro `sql!` sẽ được định nghĩa như thế này:

```rust,ignore
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

Những định nghĩa này tương tự như những signature của macro `derive` tùy chỉnh: Chúng ta nhận được những tokens mà bên trong những dấu ngoặc đơn và trả về code chúng ta muốn để tạo.

## Summary

Whew! Bây giờ bạn có một số tính năng Rust trong hộp công cụ của bạn mà bạn có lẽ sẽ không sử dụng thường xuyên, nhưng bạn sẽ biết chúng có sẵn trong những hoàn cảnh rất cụ thể. Chúng ta đã giới thiệu một vài những chủ đề phức tạp vì vậy khi bạn gặp phải chúng trong những suggestions thông báo lỗi hoặc trong code của những người khác, bạn sẽ có thể để nhận ra những khái niệm và syntax này. Sử dụng chương này như những tài liệu tham khảo để hướng dẫn bạn để các giải pháp.

Tiếp theo, chúng ta sẽ đặt mọi thứ chúng ta đã thảo luận trong suốt quyển sách vào thực hành và làm thêm một dự án!

[ref]: ../reference/macros-by-example.html
[tlborm]: https://veykril.github.io/tlborm/
[syn]: https://crates.io/crates/syn
[quote]: https://crates.io/crates/quote
[syn-docs]: https://docs.rs/syn/2.0/syn/struct.DeriveInput.html
[quote-docs]: https://docs.rs/quote
[decl]: #declarative-macros-with-macro_rules-for-general-metaprogramming
