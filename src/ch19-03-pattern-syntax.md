## Pattern Syntax

Trong phần này, chúng ta thu thập tất cả cú pháp hợp lệ trong các pattern và thảo luận lý do và khi nào bạn có thể muốn sử dụng mỗi loại.

### Matching Literals

Như bạn đã thấy trong Chương 6, bạn có thể so khớp các pattern với literals trực tiếp. Mã sau cung cấp một số ví dụ:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-01-literals/src/main.rs:here}}
```

Mã này in `one` vì giá trị trong `x` là `1`. Cú pháp này hữu ích khi bạn muốn mã của bạn thực hiện một hành động nếu nó nhận được một giá trị cụ thể.

### Matching Named Variables

Named variables là các pattern irrefutable khớp với bất kỳ giá trị nào, và chúng ta đã sử dụng chúng nhiều lần trong cuốn sách này. Tuy nhiên, có một phức tạp khi bạn sử dụng các biến được đặt tên trong `match`, `if let`, hoặc `while let` expressions. Bởi vì mỗi loại expression này bắt đầu một scope mới, các biến được khai báo như một phần của pattern bên trong các expression này sẽ shade những biến có cùng tên bên ngoài các cấu trúc, như trường hợp với tất cả các biến. Trong Listing 19-11, chúng ta khai báo một biến có tên là `x` với giá trị `Some(5)` và một biến `y` với giá trị `10`. Sau đó chúng ta tạo một `match` expression trên giá trị `x`. Nhìn vào các pattern trong match arms và `println!` ở cuối, và cố gắng tìm ra những gì mã sẽ in trước khi chạy mã này hoặc đọc tiếp.

<Listing number="19-11" file-name="src/main.rs" caption="Một `match` expression với một arm giới thiệu một biến mới shade một biến hiện có `y`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-11/src/main.rs:here}}
```

</Listing>

Hãy xem xét những gì xảy ra khi `match` expression chạy. Pattern trong arm match đầu tiên không khớp với giá trị xác định của `x`, vì vậy mã tiếp tục.

Pattern trong arm match thứ hai giới thiệu một biến mới có tên là `y` sẽ khớp với bất kỳ giá trị nào bên trong một giá trị `Some`. Bởi vì chúng ta ở trong một scope mới bên trong `match` expression, đây là một biến `y` mới, không phải `y` mà chúng ta khai báo ở đầu với giá trị `10`. Binding `y` mới này sẽ khớp với bất kỳ giá trị nào bên trong một `Some`, đó là những gì chúng ta có trong `x`. Do đó, `y` mới này bind với giá trị bên trong của `Some` trong `x`. Giá trị đó là `5`, vì vậy expression cho arm đó thực thi và in `Matched, y = 5`.

Nếu `x` là một giá trị `None` thay vì `Some(5)`, các pattern trong hai arm đầu tiên sẽ không khớp, vì vậy giá trị sẽ khớp với underscore. Chúng ta đã không giới thiệu biến `x` trong pattern của underscore arm, vì vậy `x` trong expression vẫn là `x` bên ngoài không bị shade. Trong trường hợp giả thuyết này, `match` sẽ in `Default case, x = None`.

Khi `match` expression kết thúc, scope của nó kết thúc, và scope của `y` bên trong cũng vậy. `println!` cuối cùng tạo ra `at the end: x = Some(5), y = 10`.

Để tạo một `match` expression so sánh các giá trị của `x` bên ngoài và `y`, thay vì giới thiệu một biến mới shade biến `y` hiện có, chúng ta sẽ cần sử dụng một match guard conditional thay thế. Chúng ta sẽ nói về match guards sau ở phần ["Adding Conditionals with Match Guards"](#adding-conditionals-with-match-guards)<!-- ignore -->.

<!-- Old headings. Do not remove or links may break. -->
<a id="multiple-patterns"></a>

### Matching Multiple Patterns

Trong `match` expressions, bạn có thể khớp nhiều pattern bằng cách sử dụng cú pháp `|`, là pattern _or_ operator. Ví dụ, trong mã sau, chúng ta khớp giá trị của `x` với các match arms, cái đầu tiên có một tùy chọn _or_, có nghĩa là nếu giá trị của `x` khớp với bất kỳ giá trị nào trong arm đó, mã của arm đó sẽ chạy:


```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-02-multiple-patterns/src/main.rs:here}}
```

Mã này in `one or two`.

### Matching Ranges of Values with `..=`

Cú pháp `..=` cho phép chúng ta khớp với một phạm vi giá trị bao gồm. Trong mã sau, khi một pattern khớp với bất kỳ giá trị nào trong phạm vi được cung cấp, arm đó sẽ thực thi:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-03-ranges/src/main.rs:here}}
```

Nếu `x` là `1`, `2`, `3`, `4`, hoặc `5`, arm đầu tiên sẽ khớp. Cú pháp này thuận tiện hơn để khớp nhiều giá trị so với sử dụng `|` operator để diễn đạt cùng một ý tưởng; nếu chúng ta sẽ sử dụng `|`, chúng ta phải chỉ định `1 | 2 | 3 | 4 | 5`. Chỉ định một phạm vi ngắn gọn hơn nhiều, đặc biệt là nếu chúng ta muốn khớp, chẳng hạn, bất kỳ số nào từ 1 đến 1.000!

Compiler kiểm tra rằng phạm vi không rỗng tại compile time, và vì những loại duy nhất mà Rust có thể biết một phạm vi có rỗng hay không là `char` và các giá trị số, các phạm vi chỉ được phép với các giá trị số hoặc `char`.

Đây là một ví dụ sử dụng các phạm vi `char`:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-04-ranges-of-char/src/main.rs:here}}
```

Rust có thể biết rằng `'c'` nằm trong phạm vi pattern đầu tiên và in `early ASCII letter`.

### Destructuring to Break Apart Values

Chúng ta cũng có thể sử dụng các pattern để destructure structs, enums, và tuples để sử dụng các phần khác nhau của các giá trị này. Hãy xem xét từng giá trị.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-structs"></a>

#### Structs

Listing 19-12 cho thấy một `Point` struct có hai fields, `x` và `y`, mà chúng ta có thể phá vỡ bằng cách sử dụng một pattern với một `let` statement.

<Listing number="19-12" file-name="src/main.rs" caption="Destructuring các fields của một struct thành các biến riêng biệt">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-12/src/main.rs}}
```

</Listing>

Mã này tạo các biến `a` và `b` khớp với các giá trị của các fields `x` và `y` của struct `p`. Ví dụ này cho thấy rằng các tên của các biến trong pattern không phải khớp với các tên field của struct. Tuy nhiên, thông thường là khớp các tên biến với các tên field để giúp bạn dễ nhớ hơn biến nào đến từ fields nào. Vì cách sử dụng phổ biến này, và vì viết `let Point { x: x, y: y } = p;` chứa rất nhiều sự lặp lại, Rust có một shorthand cho các pattern khớp các struct fields: Bạn chỉ cần liệt kê tên của struct field, và các biến được tạo từ pattern sẽ có các tên giống nhau. Listing 19-13 hoạt động theo cách tương tự như mã trong Listing 19-12, nhưng các biến được tạo trong `let` pattern là `x` và `y` thay vì `a` và `b`.

<Listing number="19-13" file-name="src/main.rs" caption="Destructuring các struct fields sử dụng struct field shorthand">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-13/src/main.rs}}
```

</Listing>

Mã này tạo các biến `x` và `y` khớp với các fields `x` và `y` của biến `p`. Kết quả là các biến `x` và `y` chứa các giá trị từ struct `p`.

Chúng ta cũng có thể destructure với các literal values như một phần của struct pattern thay vì tạo các biến cho tất cả các fields. Làm như vậy cho phép chúng ta kiểm tra một số fields cho các giá trị cụ thể trong khi tạo các biến để destructure các fields khác.

Trong Listing 19-14, chúng ta có một `match` expression chia các giá trị `Point` thành ba trường hợp: các điểm nằm trực tiếp trên trục `x` (điều này đúng khi `y = 0`), trên trục `y` (`x = 0`), hoặc không nằm trên trục nào.

<Listing number="19-14" file-name="src/main.rs" caption="Destructuring và matching các literal values trong một pattern">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-14/src/main.rs:here}}
```

</Listing>

Arm đầu tiên sẽ khớp với bất kỳ điểm nào nằm trên trục `x` bằng cách chỉ định rằng field `y` khớp nếu giá trị của nó khớp với literal `0`. Pattern vẫn tạo một biến `x` mà chúng ta có thể sử dụng trong mã cho arm này.

Tương tự, arm thứ hai khớp với bất kỳ điểm nào trên trục `y` bằng cách chỉ định rằng field `x` khớp nếu giá trị của nó là `0` và tạo một biến `y` cho giá trị của field `y`. Arm thứ ba không chỉ định bất kỳ literals nào, vì vậy nó khớp với bất kỳ `Point` khác nào và tạo các biến cho cả hai fields `x` và `y`.

Trong ví dụ này, giá trị `p` khớp với arm thứ hai nhờ `x` chứa một `0`, vì vậy mã này sẽ in `On the y axis at 7`.

Hãy nhớ rằng một `match` expression dừng kiểm tra các arms sau khi nó đã tìm thấy pattern khớp đầu tiên, vì vậy mặc dù `Point { x: 0, y: 0 }` nằm trên cả trục `x` và trục `y`, mã này sẽ chỉ in `On the x axis at 0`.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-enums"></a>

#### Enums

Chúng ta đã destructure các enums trong cuốn sách này (ví dụ, Listing 6-5 trong Chương 6), nhưng chúng ta chưa tường minh thảo luận rằng pattern để destructure một enum tương ứng với cách dữ liệu được lưu trữ trong enum được định nghĩa. Ví dụ, trong Listing 19-15, chúng ta sử dụng `Message` enum từ Listing 6-2 và viết một `match` với các pattern sẽ destructure mỗi inner value.

<Listing number="19-15" file-name="src/main.rs" caption="Destructuring các enum variants giữ các loại giá trị khác nhau">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-15/src/main.rs}}
```

</Listing>

Mã này sẽ in `Change color to red 0, green 160, and blue 255`. Hãy thử thay đổi giá trị của `msg` để xem mã từ các arms khác chạy.

Đối với các enum variants mà không có bất kỳ dữ liệu nào, như `Message::Quit`, chúng ta không thể destructure giá trị nào khác. Chúng ta chỉ có thể khớp trên literal `Message::Quit` value, và không có biến nào trong pattern đó.

Đối với các enum variants giống như struct, chẳng hạn như `Message::Move`, chúng ta có thể sử dụng một pattern tương tự với pattern chúng ta chỉ định để khớp các structs. Sau tên variant, chúng ta đặt dấu ngoặc nhọn và sau đó liệt kê các fields với các biến để chúng ta phá vỡ các phần để sử dụng trong mã cho arm này. Ở đây chúng ta sử dụng dạng shorthand như chúng ta đã làm trong Listing 19-13.

Đối với các tuple-like enum variants, như `Message::Write` giữ một tuple với một phần tử và `Message::ChangeColor` giữ một tuple với ba phần tử, pattern tương tự với pattern chúng ta chỉ định để khớp tuples. Số lượng biến trong pattern phải khớp với số lượng phần tử trong variant mà chúng ta đang khớp.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-nested-structs-and-enums"></a>

#### Nested Structs và Enums

Cho đến nay, tất cả các ví dụ của chúng ta đều khớp các structs hoặc enums một cấp độ sâu, nhưng matching cũng có thể hoạt động trên các mục lồng nhau! Ví dụ, chúng ta có thể refactor mã trong Listing 19-15 để hỗ trợ các màu RGB và HSV trong thông báo `ChangeColor`, như được hiển thị trong Listing 19-16.

<Listing number="19-16" caption="Matching trên các enums lồng nhau">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-16/src/main.rs}}
```

</Listing>

Pattern của arm đầu tiên trong `match` expression khớp với một enum variant `Message::ChangeColor` chứa một variant `Color::Rgb`; sau đó, pattern bind với ba giá trị `i32` bên trong. Pattern của arm thứ hai cũng khớp với một enum variant `Message::ChangeColor`, nhưng enum bên trong khớp với `Color::Hsv` thay thế. Chúng ta có thể chỉ định các điều kiện phức tạp này trong một `match` expression, mặc dù hai enums được liên quan.

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-structs-and-tuples"></a>

#### Structs và Tuples

Chúng ta có thể trộn, khớp, và lồng các destructuring patterns theo những cách thậm chí còn phức tạp hơn. Ví dụ sau cho thấy một destructure phức tạp nơi chúng ta lồng các structs và tuples bên trong một tuple và destructure tất cả các primitive values ra:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-05-destructuring-structs-and-tuples/src/main.rs:here}}
```

Mã này cho phép chúng ta phá vỡ các loại phức tạp thành các bộ phận thành phần của chúng để chúng ta có thể sử dụng các giá trị mà chúng ta quan tâm riêng biệt.

Destructuring với các pattern là một cách thuận tiện để sử dụng các phần của các giá trị, chẳng hạn như giá trị từ mỗi field trong một struct, riêng biệt với nhau.

### Ignoring Values in a Pattern

Bạn đã thấy rằng đôi khi hữu ích để bỏ qua các giá trị trong một pattern, chẳng hạn như trong arm cuối cùng của một `match`, để nhận được một catch-all không thực sự làm bất cứ điều gì nhưng đáp ứng tất cả các giá trị còn lại có thể. Có một vài cách để bỏ qua toàn bộ giá trị hoặc các phần của giá trị trong một pattern: sử dụng pattern `_` (mà bạn đã thấy), sử dụng pattern `_` trong một pattern khác, sử dụng một tên bắt đầu bằng underscore, hoặc sử dụng `..` để bỏ qua các phần còn lại của một giá trị. Hãy khám phá cách và lý do sử dụng từng loại pattern này.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-an-entire-value-with-_"></a>

#### An Entire Value with `_`

Chúng ta đã sử dụng underscore như một wildcard pattern sẽ khớp với bất kỳ giá trị nào nhưng không bind với giá trị. Điều này đặc biệt hữu ích như arm cuối cùng trong một `match` expression, nhưng chúng ta cũng có thể sử dụng nó trong bất kỳ pattern nào, bao gồm các tham số function, như được hiển thị trong Listing 19-17.

<Listing number="19-17" file-name="src/main.rs" caption="Sử dụng `_` trong một function signature">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-17/src/main.rs}}
```

</Listing>

Mã này sẽ hoàn toàn bỏ qua giá trị `3` được truyền dưới dạng đối số đầu tiên, và sẽ in `This code only uses the y parameter: 4`.

Trong hầu hết các trường hợp khi bạn không còn cần một tham số function cụ thể nữa, bạn sẽ thay đổi signature để nó không bao gồm tham số không sử dụng. Bỏ qua một tham số function có thể đặc biệt hữu ích trong những trường hợp khi, ví dụ, bạn đang triển khai một trait khi bạn cần một signature kiểu nhất định nhưng function body trong triển khai của bạn không cần một trong các tham số. Sau đó bạn tránh nhận được cảnh báo compiler về các tham số function không sử dụng, như bạn sẽ làm nếu bạn sử dụng một tên thay thế.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-parts-of-a-value-with-a-nested-_"></a>

#### Parts of a Value with a Nested `_`

Chúng ta cũng có thể sử dụng `_` bên trong một pattern khác để bỏ qua chỉ một phần của một giá trị, ví dụ, khi chúng ta muốn kiểm tra chỉ một phần của một giá trị nhưng không có sử dụng cho các phần khác trong mã tương ứng mà chúng ta muốn chạy. Listing 19-18 cho thấy mã chịu trách nhiệm quản lý giá trị của một setting. Các yêu cầu kinh doanh là người dùng không nên được phép ghi đè một tùy chỉnh hiện có của một setting nhưng có thể hủy đặt setting và cung cấp cho nó một giá trị nếu nó hiện không được đặt.

<Listing number="19-18" caption="Sử dụng một underscore bên trong các pattern khớp các variant `Some` khi chúng ta không cần sử dụng giá trị bên trong `Some`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-18/src/main.rs:here}}
```

</Listing>

Mã này sẽ in `Can't overwrite an existing customized value` và sau đó `setting is Some(5)`. Trong arm match đầu tiên, chúng ta không cần phải khớp hoặc sử dụng các giá trị bên trong bất kỳ variant `Some` nào, nhưng chúng ta cần kiểm tra trường hợp khi `setting_value` và `new_setting_value` là variant `Some`. Trong trường hợp đó, chúng ta in lý do không thay đổi `setting_value`, và nó không bị thay đổi.

Trong tất cả các trường hợp khác (nếu hoặc `setting_value` hoặc `new_setting_value` là `None`) được biểu thị bằng pattern `_` trong arm thứ hai, chúng ta muốn cho phép `new_setting_value` trở thành `setting_value`.

Chúng ta cũng có thể sử dụng underscores ở nhiều vị trí trong một pattern để bỏ qua các giá trị cụ thể. Listing 19-19 cho thấy một ví dụ về việc bỏ qua các giá trị thứ hai và thứ tư trong một tuple gồm năm mục.

<Listing number="19-19" caption="Bỏ qua nhiều phần của một tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-19/src/main.rs:here}}
```

</Listing>

Mã này sẽ in `Some numbers: 2, 8, 32`, và các giá trị `4` và `16` sẽ bị bỏ qua.

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-an-unused-variable-by-starting-its-name-with-_"></a>

#### An Unused Variable by Starting Its Name with `_`

Nếu bạn tạo một biến nhưng không sử dụng nó ở bất kỳ đâu, Rust sẽ thường phát hành một cảnh báo vì một biến không sử dụng có thể là một lỗi. Tuy nhiên, đôi khi hữu ích để có thể tạo một biến mà bạn sẽ không sử dụng chưa, chẳng hạn như khi bạn đang tạo prototype hoặc chỉ mới bắt đầu một dự án. Trong tình huống này, bạn có thể nói với Rust không cảnh báo bạn về biến không sử dụng bằng cách bắt đầu tên của biến bằng một underscore. Trong Listing 19-20, chúng ta tạo hai biến không sử dụng, nhưng khi chúng ta biên dịch mã này, chúng ta chỉ nên nhận được cảnh báo về một trong chúng.

<Listing number="19-20" file-name="src/main.rs" caption="Bắt đầu tên biến bằng underscore để tránh nhận cảnh báo biến không sử dụng">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-20/src/main.rs}}
```

</Listing>

Ở đây, chúng ta nhận được cảnh báo về việc không sử dụng biến `y`, nhưng chúng ta không nhận được cảnh báo về việc không sử dụng `_x`.

Lưu ý rằng có một sự khác biệt tinh tế giữa sử dụng chỉ `_` và sử dụng một tên bắt đầu bằng underscore. Cú pháp `_x` vẫn bind giá trị vào biến, trong khi `_` không bind ở tất cả. Để cho thấy một trường hợp mà sự khác biệt này quan trọng, Listing 19-21 sẽ cung cấp cho chúng ta một lỗi.

<Listing number="19-21" caption="Một biến không sử dụng bắt đầu bằng underscore vẫn bind giá trị, có thể chiếm quyền sở hữu của giá trị.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-21/src/main.rs:here}}
```

</Listing>

Chúng ta sẽ nhận được một lỗi vì giá trị `s` vẫn sẽ được chuyển vào `_s`, điều này ngăn chúng ta sử dụng `s` lại. Tuy nhiên, sử dụng underscore bằng chính nó không bao giờ bind với giá trị. Listing 19-22 sẽ biên dịch mà không có bất kỳ lỗi nào vì `s` không bị chuyển vào `_`.

<Listing number="19-22" caption="Sử dụng một underscore không bind giá trị.">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-22/src/main.rs:here}}
```

</Listing>

Mã này hoạt động tốt vì chúng ta không bao giờ bind `s` vào bất cứ thứ gì; nó không bị chuyển.

<a id="ignoring-remaining-parts-of-a-value-with-"></a>

#### Remaining Parts of a Value with `..`

Với các giá trị có nhiều phần, chúng ta có thể sử dụng cú pháp `..` để sử dụng các phần cụ thể và bỏ qua phần còn lại, tránh cần phải liệt kê các underscores cho mỗi giá trị bị bỏ qua. Pattern `..` bỏ qua bất kỳ phần nào của một giá trị mà chúng ta không khớp tường minh trong phần còn lại của pattern. Trong Listing 19-23, chúng ta có một `Point` struct giữ một tọa độ trong không gian ba chiều. Trong `match` expression, chúng ta muốn hoạt động chỉ trên tọa độ `x` và bỏ qua các giá trị trong các fields `y` và `z`.

<Listing number="19-23" caption="Bỏ qua tất cả các fields của một `Point` ngoại trừ `x` bằng cách sử dụng `..`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-23/src/main.rs:here}}
```

</Listing>

Chúng ta liệt kê giá trị `x` và sau đó chỉ bao gồm pattern `..`. Điều này nhanh hơn so với việc phải liệt kê `y: _` và `z: _`, đặc biệt khi chúng ta đang làm việc với các structs có rất nhiều fields trong các tình huống mà chỉ một hoặc hai fields là liên quan.

Cú pháp `..` sẽ mở rộng thành bao nhiêu giá trị cần thiết. Listing 19-24 cho thấy cách sử dụng `..` với một tuple.

<Listing number="19-24" file-name="src/main.rs" caption="Khớp chỉ giá trị đầu tiên và cuối cùng trong một tuple và bỏ qua tất cả các giá trị khác">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-24/src/main.rs}}
```

</Listing>

Trong mã này, các giá trị đầu tiên và cuối cùng được khớp với `first` và `last`. `..` sẽ khớp và bỏ qua mọi thứ ở giữa.

Tuy nhiên, sử dụng `..` phải rõ ràng. Nếu không rõ giá trị nào được dự định để khớp và cái nào nên bị bỏ qua, Rust sẽ gây cho chúng ta một lỗi. Listing 19-25 cho thấy một ví dụ về việc sử dụng `..` một cách mơ hồ, vì vậy nó sẽ không biên dịch.

<Listing number="19-25" file-name="src/main.rs" caption="Một nỗ lực sử dụng `..` một cách mơ hồ">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-25/src/main.rs}}
```

</Listing>

Khi chúng ta biên dịch ví dụ này, chúng ta nhận được lỗi này:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-25/output.txt}}
```

Không thể cho Rust xác định bao nhiêu giá trị trong tuple để bỏ qua trước khi khớp một giá trị với `second` và sau đó bao nhiêu giá trị tiếp theo để bỏ qua sau đó. Mã này có thể có nghĩa là chúng ta muốn bỏ qua `2`, bind `second` với `4`, và sau đó bỏ qua `8`, `16`, và `32`; hoặc rằng chúng ta muốn bỏ qua `2` và `4`, bind `second` với `8`, và sau đó bỏ qua `16` và `32`; và cứ thế. Tên biến `second` không có ý nghĩa đặc biệt nào đối với Rust, vì vậy chúng ta nhận được lỗi compiler vì sử dụng `..` ở hai vị trí như thế này là mơ hồ.

<!-- Old headings. Do not remove or links may break. -->

<a id="extra-conditionals-with-match-guards"></a>

### Adding Conditionals with Match Guards

Một _match guard_ là một điều kiện `if` bổ sung, được chỉ định sau pattern trong một match arm, cũng phải khớp để arm đó được chọn. Match guards hữu ích để diễn đạt những ý tưởng phức tạp hơn so với pattern một mình cho phép. Lưu ý rằng chúng chỉ khả dụng trong `match` expressions, không phải là `if let` hoặc `while let` expressions.

Điều kiện có thể sử dụng các biến được tạo trong pattern. Listing 19-26 cho thấy một `match` nơi arm đầu tiên có pattern `Some(x)` và cũng có một match guard của `if x % 2 == 0` (sẽ là `true` nếu số là chẵn).

<Listing number="19-26" caption="Thêm một match guard vào một pattern">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-26/src/main.rs:here}}
```

</Listing>

Ví dụ này sẽ in `The number 4 is even`. Khi `num` được so sánh với pattern trong arm đầu tiên, nó khớp vì `Some(4)` khớp với `Some(x)`. Sau đó, match guard kiểm tra xem phần dư của chia `x` cho 2 có bằng 0 không, và vì nó là, arm đầu tiên được chọn.

Nếu `num` là `Some(5)` thay thế, match guard trong arm đầu tiên sẽ là `false` vì phần dư của 5 chia cho 2 là 1, không bằng 0. Rust sau đó sẽ đi đến arm thứ hai, sẽ khớp vì arm thứ hai không có match guard và do đó khớp với bất kỳ variant `Some` nào.

Không có cách nào để diễn đạt điều kiện `if x % 2 == 0` trong một pattern, vì vậy match guard mang lại cho chúng ta khả năng diễn đạt logic này. Nhược điểm của cách biểu đạt bổ sung này là compiler không cố gắng kiểm tra exhaustiveness khi các match guard expressions được liên quan.

Khi thảo luận Listing 19-11, chúng ta đã đề cập rằng chúng ta có thể sử dụng match guards để giải quyết vấn đề pattern-shadowing của chúng ta. Nhớ lại rằng chúng ta đã tạo một biến mới bên trong pattern trong `match` expression thay vì sử dụng biến bên ngoài `match`. Biến mới đó có nghĩa là chúng ta không thể kiểm tra so với giá trị của biến bên ngoài. Listing 19-27 cho thấy cách chúng ta có thể sử dụng một match guard để giải quyết vấn đề này.

<Listing number="19-27" file-name="src/main.rs" caption="Sử dụng một match guard để kiểm tra bình đẳng với một biến bên ngoài">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-27/src/main.rs}}
```

</Listing>

Mã này sẽ in `Default case, x = Some(5)`. Pattern trong arm match thứ hai không giới thiệu một biến `y` mới sẽ shade `y` bên ngoài, có nghĩa là chúng ta có thể sử dụng `y` bên ngoài trong match guard. Thay vì chỉ định pattern là `Some(y)`, sẽ đã shade `y` bên ngoài, chúng ta chỉ định `Some(n)`. Điều này tạo một biến mới `n` không shade bất cứ thứ gì vì không có biến `n` nào bên ngoài `match`.

Match guard `if n == y` không phải là một pattern và do đó không giới thiệu các biến mới. `y` này _là_ `y` bên ngoài thay vì một `y` mới shade nó, và chúng ta có thể tìm kiếm một giá trị có cùng giá trị như `y` bên ngoài bằng cách so sánh `n` với `y`.

Bạn cũng có thể sử dụng _or_ operator `|` trong một match guard để chỉ định nhiều pattern; điều kiện match guard sẽ áp dụng cho tất cả các pattern. Listing 19-28 cho thấy sự ưu tiên khi kết hợp một pattern sử dụng `|` với một match guard. Phần quan trọng của ví dụ này là `if y` match guard áp dụng cho `4`, `5`, _và_ `6`, mặc dù nó có thể trông như `if y` chỉ áp dụng cho `6`.

<Listing number="19-28" caption="Kết hợp nhiều pattern với một match guard">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-28/src/main.rs:here}}
```

</Listing>

Điều kiện match nói rằng arm chỉ khớp nếu giá trị của `x` bằng `4`, `5`, hoặc `6` _và_ nếu `y` là `true`. Khi mã này chạy, pattern của arm đầu tiên khớp vì `x` là `4`, nhưng match guard `if y` là `false`, vì vậy arm đầu tiên không được chọn. Mã chuyển sang arm thứ hai, khớp, và chương trình này in `no`. Lý do là điều kiện `if` áp dụng cho toàn bộ pattern `4 | 5 | 6`, không chỉ giá trị cuối cùng `6`. Nói cách khác, sự ưu tiên của một match guard liên quan đến một pattern hoạt động như thế này:

```text
(4 | 5 | 6) if y => ...
```

thay vì cái này:

```text
4 | 5 | (6 if y) => ...
```

Sau khi chạy mã, hành vi ưu tiên là rõ ràng: Nếu match guard chỉ được áp dụng cho giá trị cuối cùng trong danh sách các giá trị được chỉ định bằng `|` operator, arm sẽ khớp, và chương trình sẽ in `yes`.

<!-- Old headings. Do not remove or links may break. -->

<a id="-bindings"></a>

### Using `@` Bindings

Operator _at_ `@` cho phép chúng ta tạo một biến giữ một giá trị tại cùng thời điểm chúng ta đang kiểm tra giá trị đó cho một pattern match. Trong Listing 19-29, chúng ta muốn kiểm tra rằng field `id` `Message::Hello` nằm trong phạm vi `3..=7`. Chúng ta cũng muốn bind giá trị với biến `id` để chúng ta có thể sử dụng nó trong mã liên kết với arm.

<Listing number="19-29" caption="Sử dụng `@` để bind với một giá trị trong một pattern trong khi cũng kiểm tra nó">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-29/src/main.rs:here}}
```

</Listing>

Ví dụ này sẽ in `Found an id in range: 5`. Bằng cách chỉ định `id @` trước phạm vi `3..=7`, chúng ta đang capturing bất kỳ giá trị nào khớp với phạm vi trong một biến có tên là `id` trong khi cũng kiểm tra rằng giá trị khớp với pattern phạm vi.

Trong arm thứ hai, nơi chúng ta chỉ có một phạm vi được chỉ định trong pattern, mã liên kết với arm không có một biến chứa giá trị thực của field `id`. Giá trị field `id` có thể là 10, 11, hoặc 12, nhưng mã đi kèm với pattern không biết nó là cái nào. Mã pattern không thể sử dụng giá trị từ field `id` vì chúng ta chưa lưu giá trị `id` trong một biến.

Trong arm cuối cùng, nơi chúng ta đã chỉ định một biến mà không có phạm vi, chúng ta có giá trị khả dụng để sử dụng trong mã arm trong một biến có tên là `id`. Lý do là chúng ta đã sử dụng cú pháp shorthand struct field. Nhưng chúng ta chưa áp dụng bất kỳ kiểm tra nào cho giá trị trong field `id` trong arm này, như chúng ta đã làm với hai arm đầu tiên: Bất kỳ giá trị nào sẽ khớp với pattern này.

Sử dụng `@` cho phép chúng ta kiểm tra một giá trị và lưu nó trong một biến trong một pattern.

## Summary

Các pattern của Rust rất hữu ích trong việc phân biệt giữa các loại dữ liệu khác nhau. Khi được sử dụng trong `match` expressions, Rust đảm bảo rằng các pattern của bạn bao gồm mọi giá trị có thể, hoặc chương trình của bạn sẽ không biên dịch. Các pattern trong `let` statements và các tham số function làm cho các cấu trúc đó hữu ích hơn, cho phép destructuring các giá trị thành các phần nhỏ hơn và gán các phần đó cho các biến. Chúng ta có thể tạo các pattern đơn giản hoặc phức tạp để phù hợp với nhu cầu của chúng ta.

Tiếp theo, đối với chương gần cuối cùng của cuốn sách, chúng ta sẽ xem xét một số khía cạnh nâng cao của các tính năng Rust khác nhau.
