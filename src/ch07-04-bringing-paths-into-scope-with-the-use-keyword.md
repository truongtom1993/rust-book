## Đưa Paths Vào Phạm Vi Bằng Từ Khóa `use`

Phải viết ra các paths để gọi functions có thể cảm thấy không tiện lợi và
lặp lại. Trong Listing 7-7, cho dù chúng ta chọn absolute hay relative path
đến function `add_to_waitlist`, mỗi lần chúng ta muốn gọi `add_to_waitlist`
chúng ta phải chỉ định `front_of_house` và `hosting` cũng vậy. May mắn thay,
có một cách để đơn giản hóa quá trình này: Chúng ta có thể tạo một phím tắt
đến một path bằng từ khóa `use` một lần và sau đó sử dụng tên ngắn hơn ở mọi
nơi khác trong phạm vi.

Trong Listing 7-11, chúng ta đưa module `crate::front_of_house::hosting` vào
phạm vi của function `eat_at_restaurant` để chúng ta chỉ phải chỉ định
`hosting::add_to_waitlist` để gọi function `add_to_waitlist` trong
`eat_at_restaurant`.

<Listing number="7-11" file-name="src/lib.rs" caption="Đưa một module vào phạm vi bằng `use`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-11/src/lib.rs}}
```

</Listing>

Thêm `use` và một path trong một phạm vi giống như tạo một symbolic link trong
filesystem. Bằng cách thêm `use crate::front_of_house::hosting` trong crate
root, `hosting` bây giờ là một tên hợp lệ trong phạm vi đó, giống như thể module
`hosting` đã được định nghĩa trong crate root. Paths đưa vào phạm vi bằng `use`
cũng kiểm tra quyền riêng tư, giống như bất kỳ paths nào khác.

Lưu ý rằng `use` chỉ tạo phím tắt cho phạm vi cụ thể mà `use` xảy ra.
Listing 7-12 di chuyển function `eat_at_restaurant` vào một module con mới tên
`customer`, sau đó là một phạm vi khác so với statement `use`, vì vậy phần
thân function sẽ không biên dịch.

<Listing number="7-12" file-name="src/lib.rs" caption="Một statement `use` chỉ áp dụng trong phạm vi nó ở.">

```rust,noplayground,test_harness,does_not_compile,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-12/src/lib.rs}}
```

</Listing>

Compiler error cho thấy rằng phím tắt không còn áp dụng trong module `customer`:

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-12/output.txt}}
```

Lưu ý cũng có một cảnh báo rằng `use` không còn được sử dụng trong phạm vi của
nó! Để khắc phục vấn đề này, hãy di chuyển `use` vào module `customer` cũng,
hoặc tham chiếu phím tắt trong module cha bằng `super::hosting` trong module
con `customer`.

### Tạo Idiomatic `use` Paths

Trong Listing 7-11, bạn có thể tự hỏi tại sao chúng ta chỉ định `use
crate::front_of_house::hosting` và sau đó gọi `hosting::add_to_waitlist` trong
`eat_at_restaurant`, thay vì chỉ định path `use` tất cả cách ra đến function
`add_to_waitlist` để đạt được kết quả tương tự, như trong Listing 7-13.

<Listing number="7-13" file-name="src/lib.rs" caption="Đưa function `add_to_waitlist` vào phạm vi bằng `use`, đó là unidiomatic">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-13/src/lib.rs}}
```

</Listing>

Mặc dù cả Listing 7-11 và Listing 7-13 đều thực hiện cùng một tác vụ, Listing
7-11 là cách idiomatic để đưa một function vào phạm vi bằng `use`. Đưa module
cha của function vào phạm vi bằng `use` có nghĩa là chúng ta phải chỉ định
module cha khi gọi function. Chỉ định module cha khi gọi function làm cho rõ
ràng rằng function không được định nghĩa cục bộ mà vẫn giảm thiểu lặp lại của
đường dẫn đầy đủ. Mã trong Listing 7-13 không rõ ràng về nơi `add_to_waitlist`
được định nghĩa.

Mặt khác, khi đưa vào structs, enums, và các items khác bằng `use`, nó là
idiomatic để chỉ định đường dẫn đầy đủ. Listing 7-14 cho thấy cách idiomatic
để đưa struct `HashMap` của standard library vào phạm vi của một binary crate.

<Listing number="7-14" file-name="src/main.rs" caption="Đưa `HashMap` vào phạm vi theo cách idiomatic">

```rust
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-14/src/main.rs}}
```

</Listing>

Không có lý do mạnh mẽ đằng sau idiom này: Nó chỉ là quy ước đã nổi lên, và
những người đã quen với việc đọc và viết mã Rust theo cách này.

Ngoại lệ với idiom này là nếu chúng ta đang đưa vào hai items có cùng tên vào
phạm vi bằng các statements `use`, vì Rust không cho phép điều đó. Listing 7-15
cho thấy cách đưa vào hai types `Result` vào phạm vi có cùng tên nhưng các
modules cha khác nhau, và cách để tham chiếu đến chúng.

<Listing number="7-15" file-name="src/lib.rs" caption="Đưa vào hai types có cùng tên vào cùng một phạm vi yêu cầu sử dụng các modules cha của chúng.">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-15/src/lib.rs:here}}
```

</Listing>

Như bạn có thể thấy, sử dụng các modules cha phân biệt hai types `Result`.
Nếu thay vào đó chúng ta chỉ định `use std::fmt::Result` và `use std::io::Result`,
chúng ta sẽ có hai types `Result` trong cùng một phạm vi, và Rust sẽ không biết
cái nào chúng ta dự định khi chúng ta sử dụng `Result`.

### Cung Cấp Tên Mới Bằng Từ Khóa `as`

Có một giải pháp khác cho vấn đề của việc đưa vào hai types có cùng tên vào
cùng một phạm vi bằng `use`: Sau path, chúng ta có thể chỉ định `as` và một
tên cục bộ mới, hoặc _alias_, cho type. Listing 7-16 cho thấy một cách khác để
viết mã trong Listing 7-15 bằng cách đổi tên một trong hai types `Result`
sử dụng `as`.

<Listing number="7-16" file-name="src/lib.rs" caption="Đổi tên một type khi nó được đưa vào phạm vi bằng từ khóa `as`">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-16/src/lib.rs:here}}
```

</Listing>

Trong statement `use` thứ hai, chúng ta chọn tên mới `IoResult` cho type
`std::io::Result`, điều này sẽ không xung đột với `Result` từ `std::fmt` mà
chúng ta cũng đã đưa vào phạm vi. Listing 7-15 và Listing 7-16 được coi là
idiomatic, vì vậy lựa chọn tùy thuộc vào bạn!

### Re-exporting Tên Bằng `pub use`

Khi chúng ta đưa một tên vào phạm vi bằng từ khóa `use`, tên là riêng tư đối
với phạm vi mà chúng ta đã nhập nó vào. Để cho phép mã bên ngoài phạm vi đó
tham chiếu đến tên đó như thể nó đã được định nghĩa trong phạm vi đó, chúng ta
có thể kết hợp `pub` và `use`. Kỹ thuật này được gọi là _re-exporting_ vì chúng
ta đang đưa một item vào phạm vi nhưng cũng làm cho item đó khả dụng để những
người khác đưa vào phạm vi của họ.

Listing 7-17 cho thấy mã trong Listing 7-11 với `use` trong module gốc được
thay đổi thành `pub use`.

<Listing number="7-17" file-name="src/lib.rs" caption="Làm cho một tên khả dụng cho bất kỳ mã nào để sử dụng từ một phạm vi mới bằng `pub use`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-17/src/lib.rs}}
```

</Listing>

Trước khi sự thay đổi này, mã bên ngoài sẽ phải gọi function `add_to_waitlist`
bằng cách sử dụng path
`restaurant::front_of_house::hosting::add_to_waitlist()`, điều này cũng sẽ
yêu cầu module `front_of_house` được đánh dấu là `pub`. Bây giờ `pub use` này
đã re-exported module `hosting` từ module gốc, mã bên ngoài có thể sử dụng path
`restaurant::hosting::add_to_waitlist()` thay thế.

Re-exporting hữu ích khi cấu trúc nội bộ của mã của bạn khác với cách mà các
programmers gọi mã của bạn sẽ nghĩ về miền. Ví dụ, trong phép ẩn dụ nhà hàng
này, những người chạy nhà hàng suy nghĩ về "front of house" và "back of house."
Nhưng những khách hàng ghé thăm một nhà hàng có lẽ sẽ không nghĩ về các phần
của nhà hàng theo những cách đó. Với `pub use`, chúng ta có thể viết mã của
chúng ta với một cấu trúc nhưng expose một cấu trúc khác. Làm như vậy làm cho
library của chúng ta được tổ chức tốt cho các programmers làm việc trên library
và các programmers gọi library. Chúng ta sẽ xem xét một ví dụ khác về `pub use`
và cách nó ảnh hưởng đến tài liệu crate của bạn trong ["Exporting a Convenient
Public API"][ch14-pub-use]<!-- ignore --> trong Chapter 14.

### Sử Dụng External Packages

Trong Chapter 2, chúng ta lập trình một dự án trò chơi đoán rằng sử dụng một
external package tên `rand` để có được các số ngẫu nhiên. Để sử dụng `rand`
trong dự án của chúng ta, chúng ta thêm dòng này vào _Cargo.toml_:

<!-- Khi cập nhật phiên bản của `rand` được sử dụng, cũng cập nhật phiên bản của
`rand` được sử dụng trong các tệp này để tất cả chúng khớp:
* ch02-00-guessing-game-tutorial.md
* ch14-03-cargo-workspaces.md
-->

<Listing file-name="Cargo.toml">

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:9:}}
```

</Listing>

Thêm `rand` như một dependency trong _Cargo.toml_ cho Cargo biết rằng cần tải
xuống package `rand` và bất kỳ dependencies nào từ [crates.io](https://crates.io/)
và làm cho `rand` khả dụng cho dự án của chúng ta.

Sau đó, để đưa định nghĩa `rand` vào phạm vi của package của chúng ta, chúng
ta thêm một dòng `use` bắt đầu bằng tên của crate, `rand`, và liệt kê các items
chúng ta muốn đưa vào phạm vi. Nhớ lại rằng trong ["Generating a Random
Number"][rand]<!-- ignore --> trong Chapter 2, chúng ta đưa trait `Rng` vào
phạm vi và gọi function `rand::thread_rng`:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:ch07-04}}
```

Các thành viên của cộng đồng Rust đã làm cho nhiều packages khả dụng tại
[crates.io](https://crates.io/), và kéo bất kỳ package nào vào package của
bạn liên quan đến các bước giống nhau: liệt kê chúng trong tệp _Cargo.toml_
của package của bạn và sử dụng `use` để đưa items từ crates của chúng vào
phạm vi.

Lưu ý rằng standard library `std` cũng là một crate bên ngoài package của
chúng ta. Vì standard library được vận chuyển với Rust language, chúng ta
không cần phải thay đổi _Cargo.toml_ để bao gồm `std`. Nhưng chúng ta cần
phải tham chiếu đến nó bằng `use` để đưa items từ đó vào phạm vi package
của chúng ta. Ví dụ, với `HashMap` chúng ta sẽ sử dụng dòng này:

```rust
use std::collections::HashMap;
```

Đây là một absolute path bắt đầu bằng `std`, tên của standard library crate.

<!-- Tiêu đề cũ. Vui lòng không xóa hoặc các liên kết có thể bị hỏng. -->

<a id="using-nested-paths-to-clean-up-large-use-lists"></a>

### Sử Dụng Nested Paths Để Làm Sạch Danh Sách `use` Lớn

Nếu chúng ta đang sử dụng nhiều items được định nghĩa trong cùng một crate hoặc
cùng một module, liệt kê mỗi item trên dòng của nó có thể chiếm nhiều không gian
dọc trong các tệp của chúng ta. Ví dụ, hai statements `use` này chúng ta có
trong trò chơi đoán trong Listing 2-4 đưa items từ `std` vào phạm vi:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-01-use-std-unnested/src/main.rs:here}}
```

</Listing>

Thay vào đó, chúng ta có thể sử dụng nested paths để đưa các items tương tự vào
phạm vi trong một dòng. Chúng ta làm điều này bằng cách chỉ định phần chung của
path, theo sau bằng hai dấu hai chấm, và sau đó dấu ngoặc nhọn xung quanh một
danh sách các phần của paths khác nhau, như được hiển thị trong Listing 7-18.

<Listing number="7-18" file-name="src/main.rs" caption="Chỉ định một nested path để đưa nhiều items có cùng prefix vào phạm vi">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-18/src/main.rs:here}}
```

</Listing>

Trong các chương trình lớn hơn, đưa nhiều items vào phạm vi từ cùng một crate
hoặc module sử dụng nested paths có thể giảm số lượng statements `use` riêng
biệt cần thiết rất nhiều!

Chúng ta có thể sử dụng một nested path ở bất kỳ mức nào trong một path, điều
này hữu ích khi kết hợp hai statements `use` chia sẻ một subpath. Ví dụ,
Listing 7-19 cho thấy hai statements `use`: một đưa `std::io` vào phạm vi và
một đưa `std::io::Write` vào phạm vi.

<Listing number="7-19" file-name="src/lib.rs" caption="Hai statements `use` nơi cái này là một subpath của cái kia">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-19/src/lib.rs}}
```

</Listing>

Phần chung của hai paths này là `std::io`, và đó là path đầu tiên hoàn chỉnh.
Để gộp hai paths này vào một statement `use`, chúng ta có thể sử dụng `self`
trong nested path, như được hiển thị trong Listing 7-20.

<Listing number="7-20" file-name="src/lib.rs" caption="Kết hợp các paths trong Listing 7-19 vào một statement `use`">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-20/src/lib.rs}}
```

</Listing>

Dòng này đưa `std::io` và `std::io::Write` vào phạm vi.

<!-- Tiêu đề cũ. Vui lòng không xóa hoặc các liên kết có thể bị hỏng. -->

<a id="the-glob-operator"></a>

### Nhập Items Bằng Toán Tử Glob

Nếu chúng ta muốn đưa _tất cả_ public items được định nghĩa trong một path vào
phạm vi, chúng ta có thể chỉ định path đó theo sau bởi toán tử `*` glob:

```rust
use std::collections::*;
```

Statement `use` này đưa tất cả public items được định nghĩa trong `std::collections`
vào phạm vi hiện tại. Hãy cẩn thận khi sử dụng toán tử glob! Glob có thể làm
cho nó khó khăn hơn để nói những tên nào ở trong phạm vi và nơi một tên được
sử dụng trong chương trình của bạn được định nghĩa. Ngoài ra, nếu dependency
thay đổi các định nghĩa của nó, những gì bạn đã nhập thay đổi cũng vậy, điều
này có thể dẫn đến compiler errors khi bạn nâng cấp dependency nếu dependency
thêm một định nghĩa có cùng tên với một định nghĩa của bạn trong cùng một
phạm vi, ví dụ.

Toán tử glob thường được sử dụng khi kiểm tra để đưa mọi thứ dưới sự kiểm tra
vào module `tests`; chúng ta sẽ nói về điều đó trong ["How to Write
Tests"][writing-tests]<!-- ignore --> trong Chapter 11. Toán tử glob cũng
đôi khi được sử dụng như một phần của mô hình prelude: Xem [the standard
library documentation](../std/prelude/index.html#other-preludes)<!-- ignore -->
để biết thêm thông tin về mô hình đó.

[ch14-pub-use]: ch14-02-publishing-to-crates-io.html#exporting-a-convenient-public-api
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
[writing-tests]: ch11-01-writing-tests.html#how-to-write-tests
