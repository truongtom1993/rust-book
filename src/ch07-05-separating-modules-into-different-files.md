## Tách Modules Vào Các Files Khác Nhau

Cho đến nay, tất cả các ví dụ trong chương này đã định nghĩa nhiều modules
trong một file. Khi các modules trở nên lớn, bạn có thể muốn di chuyển các
định nghĩa của chúng vào một file riêng để làm cho mã dễ dàng hơn để điều
hướng.

Ví dụ, hãy bắt đầu từ mã trong Listing 7-17 có nhiều restaurant modules.
Chúng ta sẽ trích xuất modules vào files thay vì có tất cả các modules được
định nghĩa trong tệp gốc crate. Trong trường hợp này, tệp gốc crate là
_src/lib.rs_, nhưng quy trình này cũng hoạt động với binary crates có tệp
gốc crate là _src/main.rs_.

Trước tiên, chúng ta sẽ trích xuất module `front_of_house` thành file của nó.
Xóa mã bên trong dấu ngoặc nhọn cho module `front_of_house`, để lại chỉ là
khai báo `mod front_of_house;`, vì vậy _src/lib.rs_ chứa mã được hiển thị
trong Listing 7-21. Lưu ý rằng điều này sẽ không biên dịch cho đến khi chúng
ta tạo tệp _src/front_of_house.rs_ trong Listing 7-22.

<Listing number="7-21" file-name="src/lib.rs" caption="Khai báo module `front_of_house` có phần thân sẽ ở trong *src/front_of_house.rs*">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/lib.rs}}
```

</Listing>

Tiếp theo, đặt mã nằm trong dấu ngoặc nhọn vào một file mới tên _src/front_of_house.rs_,
như được hiển thị trong Listing 7-22. Compiler biết để tìm trong file này vì
nó đã gặp phải khai báo module trong crate root với tên `front_of_house`.

<Listing number="7-22" file-name="src/front_of_house.rs" caption="Định nghĩa bên trong module `front_of_house` trong *src/front_of_house.rs*">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/front_of_house.rs}}
```

</Listing>

Lưu ý rằng bạn chỉ cần tải một file bằng cách sử dụng khai báo `mod` _một lần_
trong module tree của bạn. Khi compiler biết file là một phần của dự án (và biết
nơi trong module tree mã sinh sống vì cách bạn đặt statement `mod`), các files
khác trong dự án của bạn nên tham chiếu đến mã của file đã tải bằng cách sử
dụng một path đến nơi nó được khai báo, như được bao gồm trong phần ["Paths for
Referring to an Item in the Module Tree"][paths]<!-- ignore -->. Nói cách khác,
`mod` _không_ là một "include" operation mà bạn có thể đã thấy trong các ngôn
ngữ lập trình khác.

Tiếp theo, chúng ta sẽ trích xuất module `hosting` thành file của nó. Quá trình
này có phần khác vì `hosting` là một child module của `front_of_house`, không
phải của module gốc. Chúng ta sẽ đặt file cho `hosting` trong một thư mục mới
sẽ được đặt tên cho các ancestors của nó trong module tree, trong trường hợp
này là _src/front_of_house_.

Để bắt đầu di chuyển `hosting`, chúng ta thay đổi _src/front_of_house.rs_ để
chỉ chứa khai báo của module `hosting`:

<Listing file-name="src/front_of_house.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house.rs}}
```

</Listing>

Sau đó, chúng ta tạo một thư mục _src/front_of_house_ và một file _hosting.rs_
để chứa các định nghĩa được tạo trong module `hosting`:

<Listing file-name="src/front_of_house/hosting.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house/hosting.rs}}
```

</Listing>

Nếu chúng ta thay vào đó đặt _hosting.rs_ trong thư mục _src_, compiler sẽ
dự kiến mã _hosting.rs_ sẽ ở trong module `hosting` được khai báo trong crate
root và không được khai báo như một child của module `front_of_house`. Các quy
tắc của compiler để kiểm tra các files nào cho mã modules nào có nghĩa là các
thư mục và files khớp chặt chẽ hơn với module tree.

> ### Đường Dẫn File Thay Thế
>
> Cho đến nay chúng ta đã bao gồm các đường dẫn file idiomatic nhất mà Rust
> compiler sử dụng, nhưng Rust cũng hỗ trợ một kiểu cũ hơn của đường dẫn file.
> Đối với một module tên `front_of_house` được khai báo trong crate root,
> compiler sẽ tìm mã của module ở:
>
> - _src/front_of_house.rs_ (những gì chúng ta đã bao gồm)
> - _src/front_of_house/mod.rs_ (kiểu cũ hơn, vẫn hỗ trợ đường dẫn)
>
> Đối với một module tên `hosting` là một submodule của `front_of_house`,
> compiler sẽ tìm mã của module ở:
>
> - _src/front_of_house/hosting.rs_ (những gì chúng ta đã bao gồm)
> - _src/front_of_house/hosting/mod.rs_ (kiểu cũ hơn, vẫn hỗ trợ đường dẫn)
>
> Nếu bạn sử dụng cả hai kiểu cho cùng một module, bạn sẽ nhận được một
> compiler error. Sử dụng một hỗn hợp của cả hai kiểu cho các modules khác
> nhau trong cùng một dự án được phép nhưng có thể gây nhầm lẫn cho những người
> điều hướng dự án của bạn.
>
> Nhược điểm chính của kiểu sử dụng các files được đặt tên _mod.rs_ là dự án
> của bạn có thể kết thúc với nhiều files tên _mod.rs_, điều này có thể gây
> nhầm lẫn khi bạn có chúng mở trong editor của bạn cùng một lúc.

Chúng ta đã di chuyển mã của mỗi module sang một file riêng, và module tree
vẫn giữ nguyên. Các cuộc gọi function trong `eat_at_restaurant` sẽ hoạt động
mà không cần bất kỳ sửa đổi nào, mặc dù các định nghĩa sống trong các files
khác nhau. Kỹ thuật này cho phép bạn di chuyển modules sang các files mới khi
chúng phát triển về kích thước.

Lưu ý rằng statement `pub use crate::front_of_house::hosting` trong _src/lib.rs_
cũng không thay đổi, cũng không `use` có bất kỳ tác động nào đối với những files
nào được biên dịch như một phần của crate. Từ khóa `mod` khai báo modules, và
Rust tìm kiếm trong một file có cùng tên với module cho mã đi vào module đó.

## Tóm Lại

Rust cho phép bạn chia một package thành nhiều crates và một crate thành các
modules để bạn có thể tham chiếu đến các items được định nghĩa trong một module
từ một module khác. Bạn có thể làm điều này bằng cách chỉ định absolute hoặc
relative paths. Các paths này có thể được đưa vào phạm vi bằng một statement
`use` để bạn có thể sử dụng một đường dẫn ngắn hơn cho các lần sử dụng item
trong phạm vi đó. Mã module là riêng tư theo mặc định, nhưng bạn có thể làm cho
các định nghĩa công khai bằng cách thêm từ khóa `pub`.

Trong chương tiếp theo, chúng ta sẽ xem xét một số cấu trúc dữ liệu collection
trong standard library mà bạn có thể sử dụng trong mã được tổ chức gọn gàng của
bạn.

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
