## Defining an Enum

Nếu structs cho bạn cách nhóm các fields và data liên quan với nhau, như một `Rectangle` với `width` và `height`, thì enums cho bạn cách nói rằng một giá trị là một trong một tập các giá trị có thể có. Ví dụ, chúng ta có thể muốn nói rằng `Rectangle` là một trong tập các shapes có thể có, cùng với `Circle` và `Triangle`. Để làm điều này, Rust cho phép encode những khả năng đó dưới dạng một enum.

Hãy xem một tình huống chúng ta có thể muốn biểu diễn bằng code và xem tại sao enums lại hữu ích và phù hợp hơn structs trong trường hợp này. Giả sử chúng ta cần làm việc với IP address. Hiện tại có hai chuẩn chính cho IP address: version bốn và version sáu. Vì đây là những khả năng duy nhất cho một IP address mà chương trình của chúng ta sẽ gặp, chúng ta có thể _enumerate_ (liệt kê) tất cả các variants có thể có, đó cũng là lý do enum có tên gọi như vậy.

Bất kỳ IP address nào cũng có thể là version bốn hoặc version sáu, nhưng không thể là cả hai cùng lúc. Thuộc tính này của IP address làm cho enum trở thành cấu trúc dữ liệu phù hợp vì một giá trị enum chỉ có thể là một trong các variants của nó. Cả version bốn và version sáu đều cơ bản là IP address, nên chúng nên được xử lý như cùng một type khi code xử lý các tình huống áp dụng cho mọi loại IP address.

Chúng ta có thể biểu diễn khái niệm này trong code bằng cách định nghĩa một enum `IpAddrKind` và liệt kê các loại mà một IP address có thể là, `V4` và `V6`. Đây là các variants của enum:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:def}}
```

`IpAddrKind` giờ là một custom data type mà chúng ta có thể dùng ở các nơi khác trong code.

### Enum Values

Chúng ta có thể tạo instance của từng variant của `IpAddrKind` như sau:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:instance}}
```

Lưu ý rằng các variants của enum được đặt trong namespace dưới identifier của nó, và chúng ta dùng dấu hai chấm kép để phân tách. Điều này hữu ích vì giờ cả hai giá trị `IpAddrKind::V4` và `IpAddrKind::V6` đều có cùng type: `IpAddrKind`. Chúng ta có thể, chẳng hạn, định nghĩa một function nhận bất kỳ `IpAddrKind` nào:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn}}
```

Và chúng ta có thể gọi function này với bất kỳ variant nào:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn_call}}
```

Việc dùng enums còn có thêm nhiều lợi thế. Nghĩ sâu hơn về kiểu IP address của chúng ta, hiện tại chúng ta chưa có cách lưu _data_ IP address thực sự; chúng ta chỉ biết _kind_ của nó là gì. Vì bạn vừa học về structs ở Chapter 5, bạn có thể sẽ muốn xử lý vấn đề này bằng structs như trong Listing 6-1.

<Listing number="6-1" caption="Lưu data và variant `IpAddrKind` của một IP address bằng `struct`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-01/src/main.rs:here}}
```

</Listing>

Ở đây, chúng ta định nghĩa một struct `IpAddr` có hai fields: field `kind` có type `IpAddrKind` (enum đã định nghĩa trước đó) và field `address` có type `String`. Chúng ta có hai instance của struct này. Instance đầu tiên là `home`, có giá trị `IpAddrKind::V4` cho field `kind` với address đi kèm là `127.0.0.1`. Instance thứ hai là `loopback`. Nó có variant còn lại của `IpAddrKind` làm giá trị `kind`, là `V6`, và có address `::1` đi kèm. Chúng ta dùng struct để bundle các giá trị `kind` và `address` lại với nhau, nên giờ variant được gắn với value.

Tuy nhiên, biểu diễn cùng khái niệm này chỉ bằng enum thì gọn hơn: Thay vì enum nằm trong struct, chúng ta có thể đặt data trực tiếp vào từng enum variant. Định nghĩa mới của enum `IpAddr` nói rằng cả hai variants `V4` và `V6` sẽ có `String` đi kèm:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-02-enum-with-data/src/main.rs:here}}
```

Chúng ta gắn data trực tiếp vào mỗi variant của enum, nên không cần thêm struct nữa. Ở đây cũng dễ thấy thêm một chi tiết về cách enums hoạt động: Tên mỗi enum variant mà chúng ta định nghĩa cũng trở thành một function dùng để construct instance của enum. Tức là, `IpAddr::V4()` là một function call nhận một argument kiểu `String` và trả về một instance của type `IpAddr`. Chúng ta tự động có constructor function này do việc định nghĩa enum.

Có một lợi thế khác khi dùng enum thay vì struct: Mỗi variant có thể có kiểu và số lượng data đi kèm khác nhau. IP address version bốn luôn có bốn thành phần số với giá trị từ 0 đến 255. Nếu muốn lưu địa chỉ `V4` dưới dạng bốn giá trị `u8` nhưng vẫn biểu diễn địa chỉ `V6` là một `String`, chúng ta không thể làm vậy với struct. Enums xử lý trường hợp này rất dễ:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-03-variants-with-different-data/src/main.rs:here}}
```

Chúng ta vừa trình bày vài cách khác nhau để định nghĩa data structures lưu IP address version bốn và version sáu. Tuy nhiên, thực tế việc muốn lưu IP address và encode loại của chúng phổ biến đến mức [standard library đã có sẵn một định nghĩa cho chúng ta dùng!][IpAddr] Hãy xem standard library định nghĩa `IpAddr` thế nào. Nó có đúng enum và các variants như chúng ta đã định nghĩa và dùng, nhưng nó embed data của address vào trong các variants dưới dạng hai struct khác nhau, được định nghĩa khác nhau cho từng variant:

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

Đoạn code này cho thấy bạn có thể đặt bất kỳ loại data nào vào bên trong một enum variant: ví dụ strings, numeric types, hoặc structs. Bạn thậm chí có thể include một enum khác! Ngoài ra, các types trong standard library thường không phức tạp hơn nhiều so với những gì bạn tự viết ra.

Lưu ý rằng dù standard library có chứa một định nghĩa cho `IpAddr`, chúng ta vẫn có thể tự tạo và dùng định nghĩa của mình mà không bị conflict vì chúng ta chưa đưa định nghĩa của standard library vào scope. Chúng ta sẽ nói thêm về việc đưa types vào scope ở Chapter 7.

Hãy xem một ví dụ khác về enum trong Listing 6-2: enum này có nhiều loại type được embed trong các variants của nó.

<Listing number="6-2" caption="Enum `Message` với các variants lưu trữ số lượng và loại values khác nhau">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

</Listing>

Enum này có bốn variants với các kiểu khác nhau:

- `Quit`: Không có data đi kèm
- `Move`: Có các named fields, giống như struct
- `Write`: Chứa một `String` duy nhất
- `ChangeColor`: Chứa ba giá trị `i32`

Định nghĩa một enum với các variants như trong Listing 6-2 tương tự như định nghĩa các dạng struct khác nhau, ngoại trừ việc enum không dùng keyword `struct` và tất cả variants được gom chung dưới type `Message`. Các structs sau đây có thể giữ cùng data như các enum variants phía trên:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-04-structs-similar-to-message-enum/src/main.rs:here}}
```

Nhưng nếu dùng các structs khác nhau, mỗi struct có type riêng, chúng ta sẽ khó định nghĩa một function nhận bất kỳ loại message nào trong số đó hơn so với việc dùng enum `Message` trong Listing 6-2, vốn chỉ là một type duy nhất.

Có thêm một điểm tương đồng giữa enums và structs: Cũng như chúng ta có thể định nghĩa methods trên structs bằng `impl`, chúng ta cũng có thể định nghĩa methods trên enums. Đây là một method tên `call` mà chúng ta có thể định nghĩa trên enum `Message`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-05-methods-on-enums/src/main.rs:here}}
```

Body của method sẽ dùng `self` để lấy giá trị mà chúng ta gọi method trên đó. Trong ví dụ này, chúng ta tạo một biến `m` có giá trị `Message::Write(String::from("hello"))`, và đó sẽ là `self` trong body của method `call` khi `m.call()` chạy.

Giờ hãy nhìn sang một enum khác trong standard library rất phổ biến và hữu ích: `Option`.

### The `Option` Enum

Phần này sẽ xem xét `Option` như một case study; đây là một enum khác được định nghĩa bởi standard library. Kiểu `Option` encode tình huống rất phổ biến trong đó một value có thể tồn tại, hoặc có thể không.

Ví dụ, nếu bạn yêu cầu phần tử đầu tiên của một list không rỗng, bạn sẽ nhận được một value. Nếu yêu cầu phần tử đầu tiên của một list rỗng, bạn sẽ không nhận được gì. Diễn đạt khái niệm này bằng type system nghĩa là compiler có thể kiểm tra xem bạn đã xử lý đầy đủ các trường hợp đáng lẽ phải xử lý hay chưa; tính năng này có thể ngăn chặn những bug cực kỳ phổ biến trong các ngôn ngữ lập trình khác.

Thiết kế ngôn ngữ lập trình thường được nghĩ đến dưới góc độ những features bạn _include_, nhưng những features bạn _exclude_ cũng quan trọng không kém. Rust không có feature null mà nhiều ngôn ngữ khác có. _Null_ là một value có nghĩa là không có value ở đó. Trong các ngôn ngữ có null, variables luôn ở một trong hai trạng thái: null hoặc not-null.

Trong bài nói năm 2009 “Null References: The Billion Dollar Mistake”, Tony Hoare, người phát minh null, đã nói:

> Tôi gọi nó là sai lầm tỉ đô. Lúc đó, tôi đang thiết kế hệ thống type có tham chiếu đầu tiên cho một ngôn ngữ hướng đối tượng. Mục tiêu của tôi là đảm bảo mọi việc sử dụng references đều tuyệt đối an toàn, với việc kiểm tra được thực hiện tự động bởi compiler. Nhưng tôi không cưỡng lại được cám dỗ đưa vào một null reference, đơn giản vì nó quá dễ implement. Điều này dẫn đến vô số lỗi, lỗ hổng, crash hệ thống, có lẽ đã gây ra cả tỉ đô la tổn thất và đau đớn trong bốn mươi năm qua.

Vấn đề với null values là nếu bạn cố dùng một null value như một not-null value, bạn sẽ nhận lỗi dạng nào đó. Vì thuộc tính null hoặc not-null này xuất hiện khắp nơi, rất dễ mắc lỗi kiểu này.

Tuy nhiên, khái niệm mà null cố gắng diễn đạt vẫn là một khái niệm hữu ích: Null là một value hiện tại đang invalid hoặc vắng mặt vì lý do nào đó.

Vấn đề không nằm ở khái niệm mà ở implementation cụ thể. Vì vậy, Rust không có null, nhưng có một enum có thể encode khái niệm một value hiện diện hoặc vắng mặt. Enum này là `Option<T>`, và nó được [định nghĩa trong standard library][option] như sau:

```rust
enum Option<T> {
    None,
    Some(T),
}
```

Enum `Option<T>` hữu ích đến mức nó còn được include sẵn trong prelude; bạn không cần đưa nó vào scope một cách tường minh. Các variants của nó cũng có trong prelude: Bạn có thể dùng `Some` và `None` trực tiếp mà không cần prefix `Option::`. Enum `Option<T>` vẫn chỉ là một enum bình thường, và `Some(T)` cũng như `None` vẫn là các variants của type `Option<T>`.

Cú pháp `<T>` là một feature của Rust mà chúng ta chưa bàn đến. Đây là generic type parameter, và chúng ta sẽ nói chi tiết hơn về generics trong Chapter 10. Hiện tại, bạn chỉ cần biết `<T>` nghĩa là variant `Some` của enum `Option` có thể chứa một mẩu data của bất kỳ type nào, và mỗi concrete type được dùng thay cho `T` sẽ làm cho type tổng thể `Option<T>` trở thành một type khác. Dưới đây là một số ví dụ dùng `Option` để chứa number types và char types:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-06-option-examples/src/main.rs:here}}
```

Type của `some_number` là `Option<i32>`. Type của `some_char` là `Option<char>`, là một type khác. Rust có thể suy luận các type này vì chúng ta đã chỉ rõ một value bên trong variant `Some`. Với `absent_number`, Rust yêu cầu chúng ta annotate type tổng thể `Option`: Compiler không thể suy luận type mà variant `Some` tương ứng sẽ chứa chỉ bằng cách nhìn vào một giá trị `None`. Ở đây, chúng ta nói cho Rust rằng chúng ta muốn `absent_number` thuộc type `Option<i32>`.

Khi có một giá trị `Some`, chúng ta biết rằng một value đang hiện diện, và value đó được giữ bên trong `Some`. Khi có một giá trị `None`, theo một nghĩa nào đó, nó giống null: Chúng ta không có value hợp lệ. Vậy tại sao có `Option<T>` lại tốt hơn có null?

Tóm lại, vì `Option<T>` và `T` (với `T` có thể là bất kỳ type nào) là hai types khác nhau, compiler sẽ không cho phép dùng một giá trị `Option<T>` như thể nó chắc chắn là một value hợp lệ. Ví dụ, đoạn code này sẽ không compile vì nó cố cộng một `i8` với một `Option<i8>`:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/src/main.rs:here}}
```

Nếu chạy đoạn code này, chúng ta sẽ được một error message như sau:

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/output.txt}}
```

Căng thật! Về bản chất, error message này nghĩa là Rust không biết cộng một `i8` và một `Option<i8>` thế nào vì chúng là hai types khác nhau. Khi chúng ta có một value của type như `i8` trong Rust, compiler sẽ đảm bảo rằng chúng ta luôn có một value hợp lệ. Chúng ta có thể yên tâm tiếp tục mà không cần check null trước khi dùng value đó. Chỉ khi có một `Option<i8>` (hoặc bất kỳ type nào đang làm việc cùng) chúng ta mới phải lo chuyện có thể không có value, và compiler sẽ đảm bảo chúng ta xử lý case đó trước khi dùng value.

Nói cách khác, bạn phải convert một `Option<T>` thành `T` trước khi có thể thực hiện các operations của `T` với nó. Nhìn chung, điều này giúp bắt được một trong những vấn đề phổ biến nhất với null: giả định rằng thứ gì đó không null trong khi thực tế nó null.

Loại bỏ rủi ro giả định sai một not-null value giúp bạn tự tin hơn với code của mình. Để có một value có thể là null, bạn phải chủ động opt in bằng cách đặt type của value đó là `Option<T>`. Sau đó, khi dùng value đó, bạn buộc phải xử lý tường minh trường hợp value là null. Ở mọi nơi một value có type không phải `Option<T>`, bạn có thể an tâm giả định rằng value đó không null. Đây là một quyết định thiết kế có chủ ý của Rust để hạn chế sự lan rộng của null và tăng độ an toàn cho code Rust.

Vậy làm sao lấy được value `T` ra khỏi variant `Some` khi bạn có một value kiểu `Option<T>` để dùng value đó? Enum `Option<T>` có rất nhiều methods hữu ích trong nhiều tình huống khác nhau; bạn có thể xem chúng trong [documentation của nó][docs]. Việc quen thuộc với các methods trên `Option<T>` sẽ cực kỳ hữu ích trong hành trình học Rust của bạn.

Nhìn chung, để dùng một giá trị `Option<T>`, bạn sẽ muốn có code xử lý từng variant. Bạn muốn một đoạn code chỉ chạy khi bạn có `Some(T)`, và đoạn code này được phép dùng `T` bên trong. Bạn muốn một đoạn code khác chỉ chạy khi bạn có `None`, và đoạn code đó không có `T` nào để dùng. `match` expression là một control flow construct làm đúng điều này khi dùng với enums: Nó chạy các đoạn code khác nhau tùy theo variant của enum mà nó nhận, và đoạn code đó có thể dùng data bên trong giá trị khớp.

[IpAddr]: ../std/net/enum.IpAddr.html  
[option]: ../std/option/enum.Option.html  
[docs]: ../std/option/enum.Option.html