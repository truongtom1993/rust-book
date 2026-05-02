## Paths Để Tham Chiếu Đến Một Item Trong Module Tree

Để chỉ cho Rust biết nơi tìm một item trong module tree, chúng ta sử dụng một
path giống cách chúng ta sử dụng path khi điều hướng một filesystem. Để gọi
một function, chúng ta cần biết path của nó.

Một path có thể có hai dạng:

- Một _absolute path_ là đường dẫn đầy đủ bắt đầu từ crate root; đối với mã
  từ một external crate, absolute path bắt đầu bằng tên crate, và đối với
  mã từ crate hiện tại, nó bắt đầu bằng `crate` literal.
- Một _relative path_ bắt đầu từ module hiện tại và sử dụng `self`, `super`, hoặc
  một identifier trong module hiện tại.

Cả absolute và relative paths đều được theo sau bởi một hoặc nhiều identifiers
được phân tách bằng double colons (`::`).

Quay lại Listing 7-1, giả sử chúng ta muốn gọi function `add_to_waitlist`.
Đây giống như hỏi: Path của function `add_to_waitlist` là gì?
Listing 7-3 chứa Listing 7-1 với một số modules và functions bị xóa.

Chúng tôi sẽ chỉ hai cách để gọi function `add_to_waitlist` từ một function
mới, `eat_at_restaurant`, được định nghĩa trong crate root. Các paths này là
chính xác, nhưng có một vấn đề khác còn lại sẽ ngăn ví dụ này biên dịch như
vậy. Chúng ta sẽ giải thích tại sao ngay bây giờ.

Function `eat_at_restaurant` là một phần của public API của library crate của
chúng ta, vì vậy chúng ta đánh dấu nó bằng từ khóa `pub`. Trong phần
["Exposing Paths with the `pub` Keyword"][pub]<!-- ignore -->, chúng ta sẽ đi
vào chi tiết hơn về `pub`.

<Listing number="7-3" file-name="src/lib.rs" caption="Gọi function `add_to_waitlist` sử dụng absolute và relative paths">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-03/src/lib.rs}}
```

</Listing>

Lần đầu tiên chúng ta gọi function `add_to_waitlist` trong `eat_at_restaurant`,
chúng ta sử dụng một absolute path. Function `add_to_waitlist` được định nghĩa
trong cùng một crate như `eat_at_restaurant`, điều này có nghĩa là chúng ta có
thể sử dụng từ khóa `crate` để bắt đầu một absolute path. Sau đó chúng ta bao
gồm mỗi module liên tiếp cho đến khi chúng ta đến `add_to_waitlist`. Bạn có
thể hình dung một filesystem với cùng cấu trúc: Chúng ta sẽ chỉ định đường dẫn
`/front_of_house/hosting/add_to_waitlist` để chạy chương trình `add_to_waitlist`;
sử dụng tên `crate` để bắt đầu từ crate root giống như sử dụng `/` để bắt đầu
từ filesystem root trong shell của bạn.

Lần thứ hai chúng ta gọi `add_to_waitlist` trong `eat_at_restaurant`, chúng ta
sử dụng một relative path. Path bắt đầu bằng `front_of_house`, tên của module
được định nghĩa ở cùng mức của module tree như `eat_at_restaurant`. Ở đây
filesystem equivalent sẽ là sử dụng path
`front_of_house/hosting/add_to_waitlist`. Bắt đầu bằng tên module có nghĩa
là path là tương đối.

Việc chọn sử dụng relative hay absolute path là một quyết định bạn sẽ đưa ra
dựa trên dự án của bạn, và điều đó phụ thuộc vào việc bạn có nhiều khả năng
di chuyển mã định nghĩa item riêng biệt hoặc cùng với mã sử dụng item hay không.
Ví dụ, nếu chúng ta di chuyển module `front_of_house` và function
`eat_at_restaurant` vào một module tên `customer_experience`, chúng ta sẽ cần
cập nhật absolute path để `add_to_waitlist`, nhưng relative path sẽ vẫn hợp
lệ. Tuy nhiên, nếu chúng ta di chuyển function `eat_at_restaurant` riêng biệt
vào một module tên `dining`, absolute path để gọi `add_to_waitlist` sẽ giữ
nguyên, nhưng relative path sẽ cần được cập nhật. Sở thích chung của chúng ta
là chỉ định absolute paths vì nó có khả năng cao hơn rằng chúng ta muốn di
chuyển các định nghĩa mã và các cuộc gọi item một cách độc lập với nhau.

Hãy thử biên dịch Listing 7-3 và tìm hiểu tại sao nó chưa biên dịch được!
Các lỗi chúng ta nhận được được hiển thị trong Listing 7-4.

<Listing number="7-4" caption="Compiler errors từ việc xây dựng mã trong Listing 7-3">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-03/output.txt}}
```

</Listing>

Các thông báo lỗi nói rằng module `hosting` là riêng tư. Nói cách khác, chúng ta
có các paths chính xác cho module `hosting` và function `add_to_waitlist`, nhưng
Rust không cho phép chúng ta sử dụng chúng vì nó không có quyền truy cập vào
các phần riêng tư. Trong Rust, tất cả các items (functions, methods, structs,
enums, modules, và constants) mặc định là riêng tư từ các modules cha. Nếu bạn
muốn làm cho một item như một function hay struct riêng tư, bạn đặt nó trong
một module.

Các items trong một module cha không thể sử dụng các private items bên trong
các child modules, nhưng các items trong child modules có thể sử dụng các items
trong các ancestor modules của chúng. Điều này là vì các child modules bao bọc
và ẩn các chi tiết thực hiện của chúng, nhưng các child modules có thể thấy
bối cảnh mà chúng được định nghĩa. Để tiếp tục với phép ẩn dụ của chúng ta,
hãy nghĩ về các quy tắc quyền riêng tư như là back office của một nhà hàng: Những
gì xảy ra ở đó là riêng tư cho khách hàng nhà hàng, nhưng những người quản lý
văn phòng có thể thấy và làm mọi thứ trong nhà hàng họ điều hành.

Rust chọn để hệ thống module hoạt động theo cách này sao cho ẩn các chi tiết
thực hiện nội bộ là mặc định. Bằng cách đó, bạn biết phần nào của mã nội bộ
bạn có thể thay đổi mà không làm hỏng mã bên ngoài. Tuy nhiên, Rust cho bạn
tùy chọn để expose các phần nội bộ của mã child modules đến các outer ancestor
modules bằng cách sử dụng từ khóa `pub` để làm cho một item công khai.

### Expose Paths Bằng Từ Khóa `pub`

Hãy quay lại lỗi trong Listing 7-4 nói cho chúng ta biết module `hosting` là
riêng tư. Chúng ta muốn function `eat_at_restaurant` trong module cha có quyền
truy cập vào function `add_to_waitlist` trong module con, vì vậy chúng ta đánh
dấu module `hosting` bằng từ khóa `pub`, như được hiển thị trong Listing 7-5.

<Listing number="7-5" file-name="src/lib.rs" caption="Khai báo module `hosting` là `pub` để sử dụng nó từ `eat_at_restaurant`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-05/src/lib.rs:here}}
```

</Listing>

Thật không may, mã trong Listing 7-5 vẫn dẫn đến các lỗi compiler, như được
hiển thị trong Listing 7-6.

<Listing number="7-6" caption="Compiler errors từ việc xây dựng mã trong Listing 7-5">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-05/output.txt}}
```

</Listing>

Điều gì đã xảy ra? Thêm từ khóa `pub` trước `mod hosting` làm cho module công
khai. Với sự thay đổi này, nếu chúng ta có thể truy cập `front_of_house`, chúng
ta có thể truy cập `hosting`. Nhưng _contents_ của `hosting` vẫn là riêng tư;
làm cho module công khai không làm cho nội dung của nó công khai. Từ khóa `pub`
trên một module chỉ cho phép mã trong các ancestor modules của nó tham chiếu
đến nó, không phải truy cập mã nội bộ của nó. Vì modules là containers, không
có nhiều thứ chúng ta có thể làm bằng cách chỉ làm cho module công khai; chúng
ta cần đi xa hơn và chọn để làm cho một hoặc nhiều items bên trong module công
khai cũng vậy.

Các lỗi trong Listing 7-6 nói rằng function `add_to_waitlist` là riêng tư. Các
quy tắc quyền riêng tư áp dụng cho structs, enums, functions, và methods cũng
như modules.

Hãy cũng làm cho function `add_to_waitlist` công khai bằng cách thêm từ khóa
`pub` trước định nghĩa của nó, như trong Listing 7-7.

<Listing number="7-7" file-name="src/lib.rs" caption="Thêm từ khóa `pub` vào `mod hosting` và `fn add_to_waitlist` cho phép chúng ta gọi function từ `eat_at_restaurant`.">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-07/src/lib.rs:here}}
```

</Listing>

Bây giờ mã sẽ biên dịch! Để hiểu tại sao thêm từ khóa `pub` cho phép chúng ta
sử dụng các paths này trong `eat_at_restaurant` với respect đến các quy tắc
quyền riêng tư, hãy xem xét absolute và relative paths.

Trong absolute path, chúng ta bắt đầu bằng `crate`, gốc của module tree crate
của chúng ta. Module `front_of_house` được định nghĩa trong crate root. Mặc dù
`front_of_house` không công khai, vì function `eat_at_restaurant` được định
nghĩa trong cùng một module như `front_of_house` (tức là `eat_at_restaurant`
và `front_of_house` là siblings), chúng ta có thể tham chiếu `front_of_house`
từ `eat_at_restaurant`. Tiếp theo là module `hosting` được đánh dấu bằng `pub`.
Chúng ta có thể truy cập module cha của `hosting`, vì vậy chúng ta có thể truy
cập `hosting`. Cuối cùng, function `add_to_waitlist` được đánh dấu bằng `pub`,
và chúng ta có thể truy cập module cha của nó, vì vậy cuộc gọi function này
hoạt động!

Trong relative path, logic giống như absolute path ngoại trừ bước đầu tiên:
Thay vì bắt đầu từ crate root, path bắt đầu từ `front_of_house`. Module
`front_of_house` được định nghĩa trong cùng một module như `eat_at_restaurant`,
vì vậy relative path bắt đầu từ module mà `eat_at_restaurant` được định nghĩa
hoạt động. Sau đó, vì `hosting` và `add_to_waitlist` được đánh dấu bằng `pub`,
phần còn lại của path hoạt động, và cuộc gọi function này là hợp lệ!

Nếu bạn dự định chia sẻ library crate của bạn để các dự án khác có thể sử dụng
mã của bạn, public API của bạn là hợp đồng của bạn với những người dùng crate
của bạn xác định cách họ có thể tương tác với mã của bạn. Có nhiều cân nhắc
xung quanh việc quản lý các thay đổi đối với public API của bạn để làm cho nó
dễ dàng hơn cho mọi người phụ thuộc vào crate của bạn. Những cân nhắc này nằm
ngoài phạm vi của cuốn sách này; nếu bạn quan tâm đến chủ đề này, hãy xem
[the Rust API Guidelines][api-guidelines].

> #### Best Practices cho Packages với Binary và Library
>
> Chúng tôi đề cập rằng một package có thể chứa cả _src/main.rs_ binary crate
> root cũng như _src/lib.rs_ library crate root, và cả hai crates sẽ có tên
> package theo mặc định. Thông thường, các packages với mô hình này của việc
> chứa cả library và binary crate sẽ chỉ có đủ mã trong binary crate để bắt
> đầu một executable gọi mã được định nghĩa trong library crate. Điều này cho
> phép các dự án khác được hưởng lợi từ hầu hết chức năng mà package cung cấp
> vì mã library crate có thể được chia sẻ.
>
> Module tree nên được định nghĩa trong _src/lib.rs_. Sau đó, bất kỳ public
> items nào cũng có thể được sử dụng trong binary crate bằng cách bắt đầu paths
> với tên của package. Binary crate trở thành một người dùng của library crate
> giống như một crate bên ngoài hoàn toàn sẽ sử dụng library crate: Nó chỉ có
> thể sử dụng public API. Điều này giúp bạn thiết kế một API tốt; không chỉ bạn
> là tác giả, bạn cũng là một client!
>
> Trong [Chapter 12][ch12]<!-- ignore -->, chúng ta sẽ chứng minh thực hành tổ
> chức này với một chương trình command line sẽ chứa cả binary crate và library
> crate.

### Bắt Đầu Relative Paths Với `super`

Chúng ta có thể xây dựng relative paths bắt đầu trong module cha, thay vì module
hiện tại hoặc crate root, bằng cách sử dụng `super` ở đầu path. Điều này giống
như bắt đầu một filesystem path với `..` syntax có nghĩa là đi đến thư mục cha.
Sử dụng `super` cho phép chúng ta tham chiếu một item mà chúng ta biết nó ở
trong module cha, điều này có thể làm cho sắp xếp lại module tree dễ dàng hơn
khi module có liên quan chặt chẽ với cha nhưng cha có thể được di chuyển ở nơi
khác trong module tree nào đó.

Xem xét mã trong Listing 7-8 mô hình hóa tình huống mà một đầu bếp sửa một
đơn hàng không chính xác và cá nhân mang nó ra cho khách hàng. Function
`fix_incorrect_order` được định nghĩa trong module `back_of_house` gọi function
`deliver_order` được định nghĩa trong module cha bằng cách chỉ định đường dẫn
đến `deliver_order`, bắt đầu bằng `super`.

<Listing number="7-8" file-name="src/lib.rs" caption="Gọi một function sử dụng một relative path bắt đầu với `super`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-08/src/lib.rs}}
```

</Listing>

Function `fix_incorrect_order` là trong module `back_of_house`, vì vậy chúng ta
có thể sử dụng `super` để đi đến module cha của `back_of_house`, trong trường
hợp này là `crate`, gốc. Từ đó, chúng ta tìm kiếm `deliver_order` và tìm thấy
nó. Thành công! Chúng ta nghĩ rằng module `back_of_house` và function
`deliver_order` có khả năng cao sẽ ở lại trong cùng một mối quan hệ với nhau
và được di chuyển cùng nhau nếu chúng ta quyết định sắp xếp lại module tree
crate. Do đó, chúng ta sử dụng `super` để chúng ta sẽ có ít nơi cập nhật mã
hơn trong tương lai nếu mã này được di chuyển đến một module khác.

### Làm Cho Structs và Enums Công Khai

Chúng ta cũng có thể sử dụng `pub` để chỉ định structs và enums là công khai,
nhưng có một vài chi tiết bổ sung cho việc sử dụng `pub` với structs và enums.
Nếu chúng ta sử dụng `pub` trước một định nghĩa struct, chúng ta làm cho struct
công khai, nhưng các trường của struct sẽ vẫn là riêng tư. Chúng ta có thể làm
cho mỗi trường công khai hoặc không trên từng trường hợp. Trong Listing 7-9,
chúng ta đã định nghĩa một public struct `back_of_house::Breakfast` với một
public trường `toast` nhưng một riêng tư trường `seasonal_fruit`. Điều này mô
hình hóa trường hợp trong một nhà hàng nơi khách hàng có thể chọn loại bánh
mì đi kèm với một bữa ăn, nhưng đầu bếp quyết định loại trái cây nào đi kèm
với bữa ăn dựa trên những gì có trong mùa vụ và trong kho. Trái cây khả dụng
thay đổi nhanh chóng, vì vậy khách hàng không thể chọn trái cây hoặc thậm chí
thấy loại trái cây nào họ sẽ nhận được.

<Listing number="7-9" file-name="src/lib.rs" caption="Một struct có một số trường công khai và một số trường riêng tư">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-09/src/lib.rs}}
```

</Listing>

Vì trường `toast` trong struct `back_of_house::Breakfast` là công khai, trong
`eat_at_restaurant` chúng ta có thể viết và đọc đến trường `toast` sử dụng
dot notation. Lưu ý rằng chúng ta không thể sử dụng trường `seasonal_fruit`
trong `eat_at_restaurant`, vì `seasonal_fruit` là riêng tư. Hãy thử uncomment
dòng sửa đổi giá trị trường `seasonal_fruit` để xem lỗi bạn nhận được!

Cũng lưu ý rằng vì `back_of_house::Breakfast` có một private trường, struct
cần cung cấp một public associated function xây dựng một instance của
`Breakfast` (chúng tôi đã đặt tên nó `summer` ở đây). Nếu `Breakfast` không
có function như vậy, chúng ta không thể tạo một instance của `Breakfast` trong
`eat_at_restaurant`, vì chúng ta không thể đặt giá trị của trường riêng tư
`seasonal_fruit` trong `eat_at_restaurant`.

Ngược lại, nếu chúng ta làm cho một enum công khai, tất cả các variants của
nó sau đó là công khai. Chúng ta chỉ cần `pub` trước từ khóa `enum`, như được
hiển thị trong Listing 7-10.

<Listing number="7-10" file-name="src/lib.rs" caption="Chỉ định một enum là công khai làm cho tất cả các variants của nó công khai.">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-10/src/lib.rs}}
```

</Listing>

Vì chúng ta đã làm cho enum `Appetizer` công khai, chúng ta có thể sử dụng các
variants `Soup` và `Salad` trong `eat_at_restaurant`.

Enums không rất hữu ích trừ khi các variants của chúng là công khai; nó sẽ khó
chịu để phải annotate tất cả các enum variants bằng `pub` trong mọi trường hợp,
vì vậy mặc định cho các enum variants là công khai. Structs thường hữu ích mà
không cần các fields của chúng là công khai, vì vậy các struct fields tuân theo
quy tắc chung của mọi thứ là riêng tư theo mặc định trừ khi được annotate bằng
`pub`.

Có một tình huống khác liên quan đến `pub` mà chúng ta chưa bao gồm, và đó là
tính năng module system cuối cùng của chúng ta: từ khóa `use`. Chúng ta sẽ bao
gồm `use` bởi chính nó trước tiên, và sau đó chúng ta sẽ chỉ cách kết hợp `pub`
và `use`.

[pub]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[api-guidelines]: https://rust-lang.github.io/api-guidelines/
[ch12]: ch12-00-an-io-project.html
