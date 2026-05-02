## Lưu Trữ Text Được Mã Hóa UTF-8 với Strings

Chúng ta đã nói về strings trong Chapter 4, nhưng bây giờ chúng ta sẽ xem xét chúng sâu hơn.
Các Rustaceans mới thường gặp khó khăn với strings vì sự kết hợp của ba
lý do: Rust có xu hướng công khai các lỗi có thể xảy ra, strings là một
cấu trúc dữ liệu phức tạp hơn so với những gì nhiều lập trình viên cho rằng, và
UTF-8. Những yếu tố này kết hợp lại theo cách có vẻ khó khăn khi bạn
đến từ các ngôn ngữ lập trình khác.

Chúng ta thảo luận về strings trong bối cảnh collections vì strings được
triển khai dưới dạng collection của bytes, cộng với một số methods để cung cấp
functionality hữu ích khi những bytes đó được diễn giải dưới dạng text. Trong phần này, chúng ta sẽ
nói về các hoạt động trên `String` mà mọi loại collection đều có, chẳng hạn như
tạo, cập nhật và đọc. Chúng ta cũng sẽ thảo luận về những cách mà `String`
khác biệt với các collections khác, cụ thể là cách indexing vào `String` là
phức tạp do sự khác biệt giữa cách mà con người và máy tính diễn giải
dữ liệu `String`.

<!-- Old headings. Do not remove or links may break. -->

<a id=”what-is-a-string”></a>

### Định Nghĩa Strings

Trước tiên, chúng ta sẽ định nghĩa những gì chúng ta muốn nói bằng thuật ngữ _string_. Rust chỉ có một kiểu string
trong ngôn ngữ cốt lõi, đó là string slice `str` thường được
nhìn thấy dưới dạng borrowed, `&str`. Trong Chapter 4, chúng ta đã nói về string slices,
đó là tham chiếu đến một số dữ liệu string được mã hóa UTF-8 được lưu trữ ở nơi khác. String
literals, ví dụ, được lưu trữ trong binary của chương trình và do đó là
string slices.

Kiểu `String`, được cung cấp bởi thư viện chuẩn của Rust thay vì
được mã hóa vào ngôn ngữ cốt lõi, là một kiểu string
có thể phát triển, mutable, owned, được mã hóa UTF-8.
Khi Rustaceans nói đến “strings” trong Rust, họ có thể
đang tham chiếu đến loại `String` hoặc string slice `&str`, không phải chỉ một
trong những loại đó. Mặc dù phần này chủ yếu là về `String`, cả hai loại đều
được sử dụng rộng rãi trong thư viện chuẩn của Rust, và cả `String` và string slices
đều được mã hóa UTF-8.

### Tạo một String Mới

Nhiều hoạt động giống nhau có sẵn với `Vec<T>` cũng có sẵn với `String`
vì `String` thực sự được triển khai như một wrapper xung quanh một vector
của bytes với một số guarantees, restrictions, và capabilities bổ sung. Một ví dụ
của một hàm hoạt động theo cách tương tự với `Vec<T>` và `String` là hàm `new`
để tạo một instance, được hiển thị trong Listing 8-11.

<Listing number="8-11" caption="Tạo một `String` mới rỗng">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-11/src/main.rs:here}}
```

</Listing>

Dòng này tạo một string mới rỗng gọi là `s`, vào đó chúng ta có thể tải
dữ liệu. Thường xuyên, chúng ta sẽ có một số dữ liệu ban đầu mà chúng ta muốn bắt đầu
string. Để làm điều đó, chúng ta sử dụng method `to_string`, có sẵn trên bất kỳ loại nào
triển khai trait `Display`, giống như string literals. Listing 8-12 hiển thị
hai ví dụ.

<Listing number="8-12" caption="Sử dụng method `to_string` để tạo `String` từ string literal">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-12/src/main.rs:here}}
```

</Listing>

Code này tạo một string chứa `initial contents`.

Chúng ta cũng có thể sử dụng hàm `String::from` để tạo `String` từ string
literal. Code trong Listing 8-13 tương đương với code trong Listing 8-12
sử dụng `to_string`.

<Listing number="8-13" caption="Sử dụng hàm `String::from` để tạo `String` từ string literal">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-13/src/main.rs:here}}
```

</Listing>

Vì strings được sử dụng cho rất nhiều thứ, chúng ta có thể sử dụng nhiều generic APIs
khác nhau cho strings, cung cấp cho chúng ta rất nhiều tùy chọn. Một số trong số chúng có thể
dường như là thừa, nhưng tất cả đều có chỗ của chúng! Trong trường hợp này, `String::from` và
`to_string` làm điều tương tự, vì vậy cái nào bạn chọn là vấn đề của style và
readability.

Nhớ rằng strings được mã hóa UTF-8, vì vậy chúng ta có thể bao gồm bất kỳ dữ liệu
được mã hóa đúng trong chúng, như được hiển thị trong Listing 8-14.

<Listing number="8-14" caption="Lưu trữ lời chào trong các ngôn ngữ khác nhau trong strings">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:here}}
```

</Listing>

Tất cả những cái này đều là các giá trị `String` hợp lệ.

### Cập Nhật một String

Một `String` có thể tăng kích thước và nội dung của nó có thể thay đổi, giống như nội dung
của `Vec<T>`, nếu bạn push thêm dữ liệu vào nó. Ngoài ra, bạn có thể tiện lợi
sử dụng toán tử `+` hoặc macro `format!` để nối các giá trị `String`.

<!-- Old headings. Do not remove or links may break. -->

<a id="appending-to-a-string-with-push_str-and-push"></a>

#### Thêm vào với `push_str` hoặc `push`

Chúng ta có thể phát triển `String` bằng cách sử dụng method `push_str` để thêm string slice,
như được hiển thị trong Listing 8-15.

<Listing number="8-15" caption="Thêm string slice vào `String` bằng method `push_str`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-15/src/main.rs:here}}
```

</Listing>

Sau hai dòng này, `s` sẽ chứa `foobar`. Method `push_str` nhận
string slice vì chúng ta không nhất thiết muốn lấy ownership của
parameter. Ví dụ, trong code trong Listing 8-16, chúng ta muốn có thể sử dụng
`s2` sau khi thêm nội dung của nó vào `s1`.

<Listing number="8-16" caption="Sử dụng string slice sau khi thêm nội dung của nó vào `String`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-16/src/main.rs:here}}
```

</Listing>

Nếu method `push_str` lấy ownership của `s2`, chúng ta sẽ không thể in
giá trị của nó trên dòng cuối cùng. Tuy nhiên, code này hoạt động như
chúng ta mong đợi!

Method `push` nhận một ký tự duy nhất làm parameter và thêm nó vào
`String`. Listing 8-17 thêm chữ cái _l_ vào `String` bằng method
`push`.

<Listing number="8-17" caption="Thêm một ký tự vào giá trị `String` bằng `push`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-17/src/main.rs:here}}
```

</Listing>

Kết quả là, `s` sẽ chứa `lol`.

<!-- Old headings. Do not remove or links may break. -->

<a id="concatenation-with-the--operator-or-the-format-macro"></a>

#### Nối với `+` hoặc `format!`

Thường xuyên, bạn sẽ muốn kết hợp hai strings hiện có. Một cách để làm điều đó là sử dụng
toán tử `+`, như được hiển thị trong Listing 8-18.

<Listing number="8-18" caption="Sử dụng toán tử `+` để kết hợp hai giá trị `String` thành một giá trị `String` mới">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-18/src/main.rs:here}}
```

</Listing>

String `s3` sẽ chứa `Hello, world!`. Lý do `s1` không còn
hợp lệ sau phép cộng, và lý do chúng ta sử dụng tham chiếu đến `s2`, liên quan đến
signature của method được gọi khi chúng ta sử dụng toán tử `+`.
Toán tử `+` sử dụng method `add`, có signature trông như thế này:

```rust,ignore
fn add(self, s: &str) -> String {
```

Trong thư viện chuẩn, bạn sẽ thấy `add` được định nghĩa bằng cách sử dụng generics và associated
types. Ở đây, chúng ta đã thay thế các loại cụ thể, đó là những gì xảy ra khi chúng ta
gọi method này với các giá trị `String`. Chúng ta sẽ thảo luận về generics trong Chapter 10.
Signature này cung cấp cho chúng ta các manh mối chúng ta cần để hiểu những phần phức tạp
của toán tử `+`.

Thứ nhất, `s2` có `&`, có nghĩa là chúng ta đang thêm tham chiếu của string
thứ hai vào string thứ nhất. Điều này là vì parameter `s` trong hàm `add`:
Chúng ta chỉ có thể thêm string slice vào `String`; chúng ta không thể thêm hai
giá trị `String` với nhau. Nhưng chờ đã—loại của `&s2` là `&String`, không phải `&str`,
như được chỉ định trong parameter thứ hai của `add`. Vậy tại sao Listing 8-18
lại biên dịch được?

Lý do chúng ta có thể sử dụng `&s2` trong cuộc gọi đến `add` là vì compiler
có thể coerce argument `&String` thành `&str`. Khi chúng ta gọi method `add`,
Rust sử dụng deref coercion, mà ở đây biến `&s2` thành `&s2[..]`. Chúng ta sẽ
thảo luận về deref coercion sâu hơn trong Chapter 15. Vì `add` không lấy
ownership của parameter `s`, `s2` sẽ vẫn là `String` hợp lệ sau
hoạt động này.

Thứ hai, chúng ta có thể thấy trong signature rằng `add` lấy ownership của `self`
vì `self` không _có_ `&`. Điều này có nghĩa là `s1` trong Listing 8-18 sẽ được
moved vào cuộc gọi `add` và sẽ không còn hợp lệ sau đó. Vì vậy, mặc dù
`let s3 = s1 + &s2;` có vẻ như nó sẽ sao chép cả hai strings và tạo một cái mới,
câu lệnh này thực sự lấy ownership của `s1`, thêm bản sao nội dung
của `s2`, và sau đó trả về ownership của kết quả. Nói cách khác, nó có vẻ
như nó đang tạo rất nhiều bản sao, nhưng nó không; triển khai là hiệu quả hơn
so với sao chép.

Nếu chúng ta cần nối nhiều strings, hành vi của toán tử `+`
trở nên khó quản lý:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-01-concat-multiple-strings/src/main.rs:here}}
```

Tại thời điểm này, `s` sẽ là `tic-tac-toe`. Với tất cả các ký tự `+` và `"`
, thật khó để nhìn thấy điều gì đang diễn ra. Để kết hợp strings theo
các cách phức tạp hơn, chúng ta có thể sử dụng macro `format!`:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-02-format/src/main.rs:here}}
```

Code này cũng đặt `s` thành `tic-tac-toe`. Macro `format!` hoạt động giống như
`println!`, nhưng thay vì in output ra màn hình, nó trả về một
`String` với nội dung. Phiên bản code sử dụng `format!` dễ dàng
đọc hơn nhiều, và code được tạo bởi macro `format!` sử dụng các tham chiếu
để cuộc gọi này không lấy ownership của bất kỳ parameter nào của nó.

### Indexing vào Strings

Trong nhiều ngôn ngữ lập trình khác, truy cập các ký tự riêng lẻ trong một
string bằng cách tham chiếu chúng theo index là một hoạt động hợp lệ và phổ biến. Tuy nhiên,
nếu bạn cố gắng truy cập các phần của `String` bằng cách sử dụng cú pháp indexing trong Rust, bạn sẽ
nhận được lỗi. Xem xét code không hợp lệ trong Listing 8-19.

<Listing number="8-19" caption="Cố gắng sử dụng cú pháp indexing với `String`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-19/src/main.rs:here}}
```

</Listing>

Code này sẽ dẫn đến lỗi sau:

```console
{{#include ../listings/ch08-common-collections/listing-08-19/output.txt}}
```

Lỗi kể câu chuyện: Rust strings không hỗ trợ indexing. Nhưng tại sao không? Để
trả lời câu hỏi đó, chúng ta cần thảo luận về cách Rust lưu trữ strings trong bộ nhớ.

#### Biểu Diễn Nội Bộ

Một `String` là một wrapper xung quanh `Vec<u8>`. Hãy xem xét một số string
ví dụ được mã hóa UTF-8 đúng của chúng ta từ Listing 8-14. Trước tiên, cái này:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:spanish}}
```

Trong trường hợp này, `len` sẽ là `4`, có nghĩa là vector lưu trữ string
`”Hola”` dài 4 bytes. Mỗi chữ cái này chiếm 1 byte khi được mã hóa trong
UTF-8. Dòng sau đây, tuy nhiên, có thể làm bạn ngạc nhiên (lưu ý rằng string này
bắt đầu bằng chữ Cyrillic viết hoa _Ze_, không phải số 3):

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:russian}}
```

Nếu bạn được hỏi string dài bao nhiêu, bạn có thể nói 12. Trên thực tế, câu trả lời
của Rust là 24: Đó là số bytes cần thiết để mã hóa “Здравствуйте” trong
UTF-8, vì mỗi Unicode scalar value trong string đó chiếm 2 bytes của
bộ nhớ. Do đó, một index vào các bytes của string sẽ không luôn tương ứng
với một Unicode scalar value hợp lệ. Để minh họa, hãy xem xét code Rust không hợp lệ này:

```rust,ignore,does_not_compile
let hello = “Здравствуйте”;
let answer = &hello[0];
```

Bạn đã biết rằng `answer` sẽ không phải là `З`, chữ cái đầu tiên. Khi được mã hóa
trong UTF-8, byte đầu tiên của `З` là `208` và byte thứ hai là `151`, vì vậy có vẻ như
`answer` thực sự nên là `208`, nhưng `208` không phải là một ký tự hợp lệ
bởi chính nó. Trả về `208` có khả năng không phải là những gì người dùng muốn nếu họ hỏi
về chữ cái đầu tiên của string này; tuy nhiên, đó là dữ liệu duy nhất mà Rust
có tại byte index 0. Người dùng thường không muốn giá trị byte được trả về, ngay cả
nếu string chỉ chứa các chữ cái Latin: Nếu `&”hi”[0]` là code hợp lệ trả về
giá trị byte, nó sẽ trả về `104`, không phải `h`.

Câu trả lời, vì vậy, là để tránh trả về một giá trị không mong đợi và gây ra
các bugs có thể không được phát hiện ngay lập tức, Rust không biên dịch code này
dù sao và ngăn chặn những hiểu lầm sớm trong quá trình phát triển.

<!-- Old headings. Do not remove or links may break. -->

<a id=”bytes-and-scalar-values-and-grapheme-clusters-oh-my”></a>

#### Bytes, Scalar Values, và Grapheme Clusters

Một điểm khác về UTF-8 là thực ra có ba cách liên quan để
nhìn vào strings từ quan điểm của Rust: là bytes, scalar values, và grapheme
clusters (điều gần nhất với những gì chúng ta sẽ gọi _letters_).

Nếu chúng ta nhìn vào từ tiếng Hindi “नमस्ते” viết bằng script Devanagari, nó là
được lưu trữ dưới dạng vector của các giá trị `u8` trông như thế này:

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

Đó là 18 bytes và là cách máy tính cuối cùng lưu trữ dữ liệu này. Nếu chúng ta nhìn vào
chúng dưới dạng Unicode scalar values, đó là những gì loại `char` của Rust là, những
bytes đó trông như thế này:

```text
[‘न’, ‘म’, ‘स’, ‘्’, ‘त’, ‘े’]
```

Có sáu giá trị `char` ở đây, nhưng giá trị thứ tư và thứ sáu không phải là chữ cái:
Chúng là diacritics không có ý nghĩa bởi chính chúng. Cuối cùng, nếu chúng ta nhìn vào
chúng dưới dạng grapheme clusters, chúng ta sẽ nhận được những gì mà một người sẽ gọi là bốn chữ cái
tạo nên từ Hindi:

```text
[“न”, “म”, “स्”, “ते”]
```

Rust cung cấp các cách khác nhau để diễn giải dữ liệu string thô mà máy tính
lưu trữ để mỗi chương trình có thể chọn sự diễn giải nó cần, bất kể
ngôn ngữ con người nào dữ liệu đó sử dụng.

Lý do cuối cùng Rust không cho phép chúng ta indexing vào `String` để lấy một
ký tự là vì các hoạt động indexing được dự kiến sẽ luôn mất thời gian hằng số
(O(1)). Nhưng không thể đảm bảo hiệu năng đó với `String`,
vì Rust sẽ phải đi qua nội dung từ đầu đến
index để xác định có bao nhiêu ký tự hợp lệ.

### Slicing Strings

Indexing vào string thường là một ý tưởng tồi vì không rõ ràng loại
trả về của hoạt động string-indexing nên là gì: giá trị byte, một
ký tự, một grapheme cluster, hay một string slice. Nếu bạn thực sự cần sử dụng
indices để tạo string slices, do đó, Rust yêu cầu bạn cụ thể hơn.

Thay vì indexing bằng `[]` với một số duy nhất, bạn có thể sử dụng `[]` với một
phạm vi để tạo string slice chứa bytes cụ thể:

```rust
let hello = “Здравствуйте”;

let s = &hello[0..4];
```

Ở đây, `s` sẽ là `&str` chứa 4 bytes đầu tiên của string.
Trước đây, chúng ta đã đề cập rằng mỗi ký tự này là 2 bytes, có nghĩa là
`s` sẽ là `Зд`.

Nếu chúng ta cố gắng slice chỉ một phần của bytes của ký tự với cái gì đó như
`&hello[0..1]`, Rust sẽ panic tại runtime theo cách tương tự như nếu index không hợp lệ
được truy cập trong vector:

```console
{{#include ../listings/ch08-common-collections/output-only-01-not-char-boundary/output.txt}}
```

Bạn nên cẩn thận khi tạo string slices với ranges, vì làm như vậy
có thể gây crash chương trình của bạn.

<!-- Old headings. Do not remove or links may break. -->

<a id=”methods-for-iterating-over-strings”></a>

### Lặp Lại Các Strings

Cách tốt nhất để hoạt động trên các phần của strings là rõ ràng về việc
bạn muốn ký tự hay bytes. Đối với các Unicode scalar values riêng lẻ, sử dụng
method `chars`. Gọi `chars` trên “Зд” tách ra và trả về hai giá trị của
loại `char`, và bạn có thể lặp qua kết quả để truy cập từng phần tử:

```rust
for c in “Зд”.chars() {
    println!(“{c}”);
}
```

Code này sẽ in những điều sau:

```text
З
д
```

Ngoài ra, method `bytes` trả về mỗi raw byte, có thể
thích hợp cho lĩnh vực của bạn:

```rust
for b in “Зд”.bytes() {
    println!(“{b}”);
}
```

Code này sẽ in 4 bytes tạo nên string này:

```text
208
151
208
180
```

Nhưng hãy chắc chắn nhớ rằng các Unicode scalar values hợp lệ có thể được tạo thành từ nhiều hơn
1 byte.

Lấy grapheme clusters từ strings, như với script Devanagari, là
phức tạp, vì vậy functionality này không được cung cấp bởi thư viện chuẩn. Crates
có sẵn trên [crates.io](https://crates.io/)<!-- ignore --> nếu đây là
functionality bạn cần.

<!-- Old headings. Do not remove or links may break. -->

<a id="strings-are-not-so-simple"></a>

### Xử Lý Sự Phức Tạp của Strings

Để tóm tắt, strings rất phức tạp. Các ngôn ngữ lập trình khác nhau đưa ra
những lựa chọn khác nhau về cách trình bày sự phức tạp này cho lập trình viên. Rust
đã chọn để làm việc xử lý chính xác dữ liệu `String` là hành vi mặc định
cho tất cả các chương trình Rust, có nghĩa là lập trình viên phải suy nghĩ nhiều hơn về
xử lý dữ liệu UTF-8 từ đầu. Trade-off này tiếp lộ sự phức tạp của
strings hơn là những gì rõ ràng trong các ngôn ngữ lập trình khác, nhưng nó ngăn bạn
khỏi phải xử lý các lỗi liên quan đến các ký tự non-ASCII sau này trong
vòng đời phát triển của bạn.

Tin tốt là thư viện chuẩn cung cấp rất nhiều functionality được xây dựng
từ các loại `String` và `&str` để giúp xử lý những tình huống phức tạp này
một cách chính xác. Hãy chắc chắn kiểm tra tài liệu để tìm các methods hữu ích như
`contains` để tìm kiếm trong string và `replace` để thay thế các phần của
string bằng string khác.

Hãy chuyển sang một cái gì đó ít phức tạp hơn một chút: hash maps!
