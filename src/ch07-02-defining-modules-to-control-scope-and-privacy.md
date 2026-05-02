<!-- Tiêu đề cũ. Vui lòng không xóa hoặc các liên kết có thể bị hỏng. -->

<a id="defining-modules-to-control-scope-and-privacy"></a>

## Kiểm Soát Phạm Vi và Quyền Riêng Tư Bằng Modules

Trong phần này, chúng ta sẽ nói về modules và các phần khác của hệ thống modules,
cụ thể là _paths_, cho phép bạn đặt tên các items; từ khóa `use` đưa một
path vào phạm vi; và từ khóa `pub` để công khai các items. Chúng ta cũng sẽ
thảo luận về từ khóa `as`, các packages bên ngoài, và toán tử glob.

### Bảng Tóm Tắt Nhanh Về Modules

Trước khi đi vào chi tiết của modules và paths, chúng tôi cung cấp một tham
chiếu nhanh về cách modules, paths, từ khóa `use`, và từ khóa `pub` hoạt động
trong compiler, và cách hầu hết các developers tổ chức mã của họ. Chúng ta sẽ
đi qua các ví dụ về mỗi quy tắc này trong suốt chương này, nhưng đây là một
nơi tuyệt vời để tham khảo như một lời nhắc về cách modules hoạt động.

- **Bắt đầu từ gốc crate**: Khi biên dịch một crate, compiler trước tiên
  tìm kiếm trong tệp gốc crate (thường là _src/lib.rs_ đối với library crate và
  _src/main.rs_ đối với binary crate) để tìm mã cần biên dịch.
- **Khai báo modules**: Trong tệp gốc crate, bạn có thể khai báo các modules mới;
  giả sử bạn khai báo một module "garden" với `mod garden;`. Compiler sẽ tìm
  mã của module ở các nơi sau:
  - Inline, trong các dấu ngoặc nhọn thay thế dấu chấm phẩy sau `mod
    garden`
  - Trong tệp _src/garden.rs_
  - Trong tệp _src/garden/mod.rs_
- **Khai báo submodules**: Trong bất kỳ tệp nào khác ngoài gốc crate, bạn có thể
  khai báo các submodules. Ví dụ, bạn có thể khai báo `mod vegetables;` trong
  _src/garden.rs_. Compiler sẽ tìm mã của submodule trong thư mục đặt tên cho
  module cha ở những nơi này:
  - Inline, ngay sau `mod vegetables`, trong dấu ngoặc nhọn thay vì dấu chấm phẩy
  - Trong tệp _src/garden/vegetables.rs_
  - Trong tệp _src/garden/vegetables/mod.rs_
- **Paths dẫn đến mã trong modules**: Khi một module đã trở thành một phần của
  crate của bạn, bạn có thể tham chiếu đến mã trong module đó từ bất kỳ đâu
  trong crate đó, miễn là các quy tắc quyền riêng tư cho phép, sử dụng đường
  dẫn đến mã. Ví dụ, một type `Asparagus` trong module garden vegetables sẽ được
  tìm thấy tại `crate::garden::vegetables::Asparagus`.
- **Riêng tư vs công khai**: Mã trong một module mặc định là riêng tư từ các
  modules cha của nó. Để công khai một module, khai báo nó bằng `pub mod`
  thay vì `mod`. Để công khai các items bên trong một module công khai, hãy
  sử dụng `pub` trước các khai báo của chúng.
- **Từ khóa `use`**: Trong một phạm vi, từ khóa `use` tạo các phím tắt đến
  các items để giảm lặp lại các paths dài. Trong bất kỳ phạm vi nào có thể
  tham chiếu đến `crate::garden::vegetables::Asparagus`, bạn có thể tạo một
  phím tắt với `use crate::garden::vegetables::Asparagus;`, và từ đó trở đi
  bạn chỉ cần viết `Asparagus` để sử dụng loại đó trong phạm vi.

Ở đây, chúng ta tạo một binary crate tên `backyard` minh họa các quy tắc này.
Thư mục của crate, cũng tên là _backyard_, chứa các tệp và thư mục này:

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

Tệp gốc crate trong trường hợp này là _src/main.rs_, và nó chứa:

<Listing file-name="src/main.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/main.rs}}
```

</Listing>

Dòng `pub mod garden;` cho compiler biết rằng cần đưa mã nó tìm thấy trong
_src/garden.rs_, đó là:

<Listing file-name="src/garden.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden.rs}}
```

</Listing>

Ở đây, `pub mod vegetables;` có nghĩa là mã trong _src/garden/vegetables.rs_
cũng được đưa vào. Mã đó là:

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden/vegetables.rs}}
```

Bây giờ hãy đi vào chi tiết của các quy tắc này và chứng minh chúng trong hành động!

### Nhóm Mã Liên Quan Vào Modules

_Modules_ cho phép chúng ta tổ chức mã trong một crate để dễ đọc và tái sử dụng.
Modules cũng cho phép chúng ta kiểm soát _quyền riêng tư_ của các items vì mã
trong một module mặc định là riêng tư. Các items riêng tư là chi tiết thực hiện
nội bộ không có sẵn để sử dụng bên ngoài. Chúng ta có thể chọn công khai các
modules và các items bên trong chúng, điều này cho phép mã bên ngoài sử dụng
và phụ thuộc vào chúng.

Ví dụ, hãy viết một library crate cung cấp chức năng của một nhà hàng. Chúng ta
sẽ định nghĩa các chữ ký hàm nhưng để các phần thân của chúng trống để tập
trung vào tổ chức mã hơn là thực hiện một nhà hàng.

Trong ngành công nghiệp nhà hàng, một số phần của nhà hàng được gọi là front
of house và những phần khác là back of house. _Front of house_ là nơi khách
hàng ở; điều này bao gồm nơi chủ nhà chỗ ngồi cho khách hàng, những người phục
vụ nhận đơn hàng và thanh toán, và bartenders pha chế đồ uống. _Back of house_
là nơi các đầu bếp và người nấu ăn làm việc trong bếp, những người rửa chén
làm sạch, và những người quản lý làm công việc hành chính.

Để cấu trúc crate của chúng ta theo cách này, chúng ta có thể tổ chức các
functions của nó vào các modules lồng nhau. Tạo một library mới tên `restaurant`
bằng cách chạy `cargo new restaurant --lib`. Sau đó, nhập mã trong Listing 7-1
vào _src/lib.rs_ để định nghĩa các modules và chữ ký hàm; mã này là phần
front of house.

<Listing number="7-1" file-name="src/lib.rs" caption="Một module `front_of_house` chứa các modules khác mà sau đó chứa các functions">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-01/src/lib.rs}}
```

</Listing>

Chúng ta định nghĩa một module bằng từ khóa `mod` theo sau là tên của module
(trong trường hợp này, `front_of_house`). Phần thân của module sau đó đi vào
trong dấu ngoặc nhọn. Bên trong modules, chúng ta có thể đặt các modules khác,
như trong trường hợp này với các modules `hosting` và `serving`. Modules cũng
có thể chứa các định nghĩa cho các items khác, chẳng hạn như structs, enums,
constants, traits, và như trong Listing 7-1, các functions.

Bằng cách sử dụng modules, chúng ta có thể nhóm các định nghĩa liên quan lại
với nhau và giải thích tại sao chúng liên quan. Các programmers sử dụng mã này
có thể điều hướng mã dựa trên các nhóm thay vì phải đọc qua tất cả các định
nghĩa, giúp dễ dàng hơn để tìm các định nghĩa liên quan đến họ. Các programmers
thêm chức năng mới vào mã này sẽ biết nơi đặt mã để giữ chương trình được tổ
chức.

Trước đó, chúng tôi đề cập rằng _src/main.rs_ và _src/lib.rs_ được gọi là
_crate roots_. Lý do cho tên của chúng là nội dung của một trong hai tệp này
tạo thành một module tên `crate` ở gốc của cấu trúc module crate, được biết đến
với tên _module tree_.

Listing 7-2 cho thấy module tree cho cấu trúc trong Listing 7-1.

<Listing number="7-2" caption="Module tree cho mã trong Listing 7-1">

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

</Listing>

Cây này cho thấy cách một số modules lồng vào các modules khác; ví dụ,
`hosting` lồng vào `front_of_house`. Cây cũng cho thấy rằng một số modules
là _siblings_, có nghĩa là chúng được định nghĩa trong cùng một module; `hosting`
và `serving` là các siblings được định nghĩa trong `front_of_house`. Nếu module
A được chứa bên trong module B, chúng ta nói rằng module A là _child_ của module
B và module B là _parent_ của module A. Lưu ý rằng toàn bộ module tree được gốc
dưới module ngầm tên `crate`.

Module tree có thể nhắc bạn về cây thư mục của filesystem trên máy tính của
bạn; đây là một so sánh rất phù hợp! Giống như các thư mục trong filesystem,
bạn sử dụng modules để tổ chức mã của bạn. Và giống như các tệp trong một thư
mục, chúng ta cần một cách để tìm các modules của chúng ta.
