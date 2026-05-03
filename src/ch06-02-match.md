## The `match` Control Flow Construct

Rust có một control flow construct cực kỳ mạnh mẽ tên là `match`, cho phép bạn so sánh một value với một loạt patterns và sau đó execute code dựa trên pattern nào khớp. Patterns có thể được tạo từ literal values, variable names, wildcards, và nhiều thứ khác; [Chapter 19][ch19-00-patterns] nói chi tiết tất cả các loại patterns và tác dụng của chúng. Sức mạnh của `match` đến từ khả năng biểu đạt của patterns và việc compiler đảm bảo rằng tất cả các trường hợp có thể có đều được xử lý.

Hãy coi `match` expression giống như một chiếc máy phân loại coin: Coins trượt xuống một đường ray với các lỗ có kích cỡ khác nhau, và mỗi coin sẽ rơi vào cái lỗ đầu tiên mà nó lọt qua. Tương tự, values sẽ đi qua từng pattern trong một `match`, và tại pattern đầu tiên mà value “vừa khít”, value sẽ “rơi” vào code block tương ứng để được dùng khi thực thi.

Nhắc tới coin, hãy dùng chúng làm ví dụ cho `match`! Chúng ta có thể viết một function nhận một đồng coin Mỹ không rõ loại, và giống như máy đếm coin, xác định đó là coin gì và trả về giá trị của nó tính bằng cents, như trong Listing 6-3.

<Listing number="6-3" caption="Một enum và `match` expression dùng các variants của enum làm patterns">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-03/src/main.rs:here}}
```

</Listing>

Hãy phân tích `match` trong function `value_in_cents`. Đầu tiên, chúng ta viết keyword `match` theo sau là một expression, ở đây là value `coin`. Nhìn khá giống conditional expression dùng với `if`, nhưng có một khác biệt lớn: Với `if`, condition phải evaluate thành một Boolean value, còn ở đây nó có thể là bất kỳ type nào. Type của `coin` trong ví dụ này là enum `Coin` mà chúng ta định nghĩa ở dòng đầu.

Tiếp theo là các `match` arms. Một arm có hai phần: một pattern và một đoạn code. Arm đầu tiên có pattern là giá trị `Coin::Penny` rồi đến toán tử `=>` ngăn cách pattern và code cần chạy. Code trong trường hợp này chỉ là giá trị `1`. Mỗi arm được phân tách với arm tiếp theo bằng dấu phẩy.

Khi `match` expression được thực thi, nó sẽ so sánh value với pattern của từng arm theo thứ tự. Nếu một pattern khớp với value, đoạn code gắn với pattern đó sẽ được execute. Nếu pattern đó không khớp, execution tiếp tục với arm kế tiếp, giống như trong máy phân loại coin. Chúng ta có thể có bao nhiêu arms tùy ý: Trong Listing 6-3, `match` của chúng ta có bốn arms.

Code gắn với mỗi arm là một expression, và resultant value của expression trong arm khớp là giá trị được return cho toàn bộ `match` expression.

Thông thường, chúng ta không dùng ngoặc nhọn nếu code trong match arm ngắn, như trong Listing 6-3, nơi mỗi arm chỉ return một value. Nếu bạn muốn chạy nhiều dòng code trong một match arm, bạn phải dùng ngoặc nhọn, và dấu phẩy sau arm đó khi đó là tùy chọn. Ví dụ, đoạn code sau sẽ in ra “Lucky penny!” mỗi khi method được gọi với `Coin::Penny`, nhưng vẫn return giá trị cuối cùng của block, là `1`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-08-match-arm-multiple-lines/src/main.rs:here}}
```

### Patterns That Bind to Values

Một tính năng hữu ích khác của match arms là chúng có thể bind tới các phần của value khớp với pattern. Đây là cách chúng ta extract values từ bên trong enum variants.

Làm ví dụ, hãy sửa một enum variant để chứa data bên trong. Từ 1999 tới 2008, nước Mỹ đúc các đồng quarter với thiết kế khác nhau cho từng bang (50 bang) ở một mặt. Không coin nào khác có thiết kế theo bang, nên chỉ quarters mới có thêm thông tin này. Chúng ta có thể thêm thông tin này vào `enum` bằng cách thay đổi variant `Quarter` để include một giá trị `UsState` lưu bên trong, như trong Listing 6-4.

<Listing number="6-4" caption="Enum `Coin` trong đó variant `Quarter` cũng chứa giá trị `UsState`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-04/src/main.rs:here}}
```

</Listing>

Hãy tưởng tượng một người bạn đang cố sưu tầm đủ 50 state quarters. Khi chúng ta phân loại chỗ tiền lẻ của mình theo loại coin, chúng ta cũng sẽ đọc tên bang gắn với mỗi quarter để nếu đó là một bang người bạn chưa có, họ có thể thêm vào bộ sưu tập.

Trong match expression của đoạn code này, chúng ta thêm một biến tên `state` vào pattern khớp với các giá trị variant `Coin::Quarter`. Khi một `Coin::Quarter` khớp, biến `state` sẽ bind với giá trị bang của quarter đó. Sau đó chúng ta có thể dùng `state` trong code của arm đó, như sau:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-09-variable-in-pattern/src/main.rs:here}}
```

Nếu chúng ta gọi `value_in_cents(Coin::Quarter(UsState::Alaska))`, `coin` sẽ là `Coin::Quarter(UsState::Alaska)`. Khi so sánh value này với từng match arm, không arm nào khớp cho tới khi gặp `Coin::Quarter(state)`. Tại thời điểm đó, binding cho `state` sẽ là giá trị `UsState::Alaska`. Chúng ta có thể dùng binding này trong `println!`, từ đó lấy được inner state value ra khỏi enum variant `Coin` cho `Quarter`.

### The `Option<T>` `match` Pattern

Trong phần trước, chúng ta muốn lấy inner value `T` ra khỏi case `Some` khi dùng `Option<T>`; chúng ta cũng có thể xử lý `Option<T>` bằng `match`, giống như với enum `Coin`! Thay vì so sánh coins, chúng ta sẽ so sánh các variants của `Option<T>`, nhưng cách `match` expression hoạt động vẫn như cũ.

Giả sử chúng ta muốn viết một function nhận `Option<i32>` và nếu bên trong có value thì cộng thêm 1. Nếu bên trong không có value, function nên trả về `None` và không cố thực hiện bất kỳ phép toán nào.

Function này rất dễ viết nhờ `match`, và sẽ trông như Listing 6-5.

<Listing number="6-5" caption="Function dùng `match` expression trên một `Option&lt;i32&gt;`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:here}}
```

</Listing>

Hãy phân tích lần thực thi đầu tiên của `plus_one` chi tiết hơn. Khi gọi `plus_one(five)`, biến `x` trong body của `plus_one` sẽ có giá trị `Some(5)`. Sau đó, chúng ta so sánh giá trị này với từng match arm:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

Giá trị `Some(5)` không khớp với pattern `None`, nên chúng ta tiếp tục arm tiếp theo:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:second_arm}}
```

`Some(5)` có khớp `Some(i)` không? Có! Chúng ta có cùng variant. `i` bind với value chứa trong `Some`, nên `i` nhận giá trị `5`. Đoạn code trong match arm đó được thực thi, chúng ta cộng 1 vào `i` và tạo một `Some` mới với tổng `6` bên trong.

Giờ hãy xem lần gọi thứ hai của `plus_one` trong Listing 6-5, khi `x` là `None`. Chúng ta vào `match` và so sánh với arm đầu tiên:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

Khớp luôn! Không có value nào để cộng thêm nên chương trình dừng tại đây và trả về giá trị `None` ở bên phải `=>`. Vì arm đầu tiên đã khớp nên các arm khác không được so sánh nữa.

Kết hợp `match` và enums hữu ích trong rất nhiều tình huống. Bạn sẽ thấy pattern này rất nhiều trong Rust code: `match` trên một enum, bind một biến với data bên trong, rồi execute code dựa trên nó. Ban đầu hơi khó quen, nhưng một khi đã quen bạn sẽ ước gì ngôn ngữ nào cũng có. Đây là một feature được người dùng yêu thích.

### Matches Are Exhaustive

Có một khía cạnh khác của `match` cần bàn tới: Patterns trong các arms phải bao phủ tất cả khả năng. Hãy xem phiên bản sau của function `plus_one`, trong đó có bug và sẽ không compile:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/src/main.rs:here}}
```

Chúng ta đã không xử lý case `None`, nên đoạn code này sẽ gây bug. May mắn là đây là kiểu bug Rust biết cách bắt. Nếu cố compile đoạn code này, chúng ta sẽ nhận được error:

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/output.txt}}
```

Rust biết rằng chúng ta chưa cover tất cả trường hợp có thể và thậm chí chỉ ra luôn pattern nào đã bị quên! Matches trong Rust là _exhaustive_: Chúng ta phải cover mọi trường hợp cuối cùng thì code mới hợp lệ. Đặc biệt với `Option<T>`, khi Rust buộc chúng ta không được quên xử lý tường minh case `None`, nó giúp chúng ta tránh việc giả định có value trong khi thực tế là null, từ đó khiến “sai lầm tỉ đô” nói ở trên trở nên bất khả thi.

### Catch-All Patterns and the `_` Placeholder

Khi dùng enums, chúng ta cũng có thể thực hiện các hành động đặc biệt cho một vài giá trị cụ thể, nhưng với mọi giá trị khác thì thực hiện một hành động mặc định. Hãy tưởng tượng chúng ta đang implement một game mà nếu bạn đổ được 3 trên xúc xắc thì player không di chuyển nhưng được tặng một chiếc mũ xịn. Nếu bạn đổ được 7, player sẽ mất một chiếc mũ xịn. Với mọi giá trị khác, player sẽ di chuyển số bước tương ứng trên game board. Dưới đây là một `match` implement logic đó, với kết quả đổ xúc xắc được hardcode thay vì random, và toàn bộ logic còn lại được biểu diễn bằng các function không có body vì việc implement thật sự nằm ngoài phạm vi ví dụ này:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-15-binding-catchall/src/main.rs:here}}
```

Với hai arms đầu tiên, patterns là literal values `3` và `7`. Với arm cuối cùng, cover mọi giá trị khác có thể, pattern là biến mà chúng ta chọn đặt tên `other`. Đoạn code chạy cho arm `other` dùng biến này bằng cách truyền nó vào function `move_player`.

Đoạn code này compile được, dù chúng ta không liệt kê tất cả các giá trị có thể của `u8`, vì pattern cuối cùng sẽ match mọi value không được liệt kê riêng. Catch-all pattern này đáp ứng yêu cầu `match` phải exhaustive. Lưu ý là chúng ta phải đặt catch-all arm ở cuối vì patterns được evaluate theo thứ tự. Nếu đặt catch-all arm sớm hơn, các arms khác sẽ không bao giờ chạy, nên Rust sẽ cảnh báo nếu bạn thêm arms sau một catch-all arm!

Rust cũng có một pattern dùng khi chúng ta muốn một catch-all nhưng không muốn _dùng_ value trong catch-all pattern: `_` là một special pattern match với mọi value và không bind vào value đó. Điều này nói với Rust rằng chúng ta sẽ không dùng value, nên Rust sẽ không cảnh báo về một biến không dùng đến.

Hãy thay đổi luật chơi: Giờ nếu bạn đổ bất cứ gì ngoài 3 hoặc 7, bạn phải đổ lại. Chúng ta không còn cần dùng value trong catch-all, nên có thể đổi code để dùng `_` thay cho biến tên `other`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-16-underscore-catchall/src/main.rs:here}}
```

Ví dụ này cũng đáp ứng yêu cầu exhaustiveness vì chúng ta đang explicit ignore tất cả các values khác trong arm cuối; chúng ta không bỏ sót gì.

Cuối cùng, hãy đổi luật chơi thêm lần nữa sao cho không có gì xảy ra trong lượt của bạn nếu đổ bất kỳ giá trị nào ngoài 3 hoặc 7. Chúng ta có thể biểu diễn điều đó bằng cách dùng unit value (empty tuple type đã nhắc tới trong phần [“The Tuple Type”][tuples]) làm code đi kèm với arm `_`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-17-underscore-unit/src/main.rs:here}}
```

Ở đây, chúng ta nói rõ với Rust rằng chúng ta sẽ không dùng bất kỳ value nào khác không match patterns ở các arm trước đó, và chúng ta cũng không muốn chạy code nào trong trường hợp này.

Sẽ còn nhiều điều về patterns và matching được trình bày trong [Chapter 19][ch19-00-patterns]. Còn bây giờ, chúng ta sẽ chuyển sang cú pháp `if let`, vốn hữu ích trong các tình huống mà `match` expression hơi dài dòng.

[tuples]: ch03-02-data-types.html#the-tuple-type  
[ch19-00-patterns]: ch19-00-patterns.html