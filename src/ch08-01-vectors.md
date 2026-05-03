## Lưu Trữ Danh Sách Các Giá Trị với Vectors

Loại collection đầu tiên chúng ta sẽ xem xét là `Vec<T>`, còn được gọi là vector.
Vectors cho phép bạn lưu trữ nhiều hơn một giá trị trong một cấu trúc dữ liệu duy nhất
mà đặt tất cả các giá trị cạnh nhau trong bộ nhớ. Vectors chỉ có thể lưu trữ các giá trị
có cùng loại. Chúng rất hữu ích khi bạn có một danh sách các mục, chẳng hạn như
các dòng text trong một tệp hoặc giá của các mục trong giỏ hàng.

### Tạo một Vector Mới

Để tạo một vector mới rỗng, chúng ta gọi hàm `Vec::new`, như được hiển thị trong
Listing 8-1.

<Listing number="8-1" caption="Tạo một vector mới rỗng để chứa các giá trị loại `i32`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-01/src/main.rs:here}}
```

</Listing>

Lưu ý rằng chúng ta đã thêm type annotation ở đây. Vì chúng ta không chèn bất kỳ
giá trị nào vào vector này, Rust không biết loại phần tử nào chúng ta dự định
lưu trữ. Đây là một điểm quan trọng. Vectors được triển khai bằng cách sử dụng generics;
chúng ta sẽ tìm hiểu cách sử dụng generics với các loại của riêng bạn trong Chapter 10. Hiện tại,
hãy biết rằng kiểu `Vec<T>` được cung cấp bởi thư viện chuẩn có thể chứa bất kỳ loại nào.
Khi chúng ta tạo một vector để chứa một loại cụ thể, chúng ta có thể chỉ định loại trong
các dấu ngoặc nhọn. Trong Listing 8-1, chúng ta đã nói với Rust rằng `Vec<T>` trong `v` sẽ
chứa các phần tử của loại `i32`.

Thường xuyên hơn, bạn sẽ tạo một `Vec<T>` với các giá trị ban đầu, và Rust sẽ suy ra
loại giá trị mà bạn muốn lưu trữ, vì vậy bạn hiếm khi cần phải thực hiện type
annotation này. Rust tiện lợi cung cấp macro `vec!`, sẽ tạo một
vector mới chứa các giá trị mà bạn cung cấp cho nó. Listing 8-2 tạo một
`Vec<i32>` mới chứa các giá trị `1`, `2`, và `3`. Loại số nguyên là `i32`
vì đó là loại số nguyên mặc định, như chúng ta đã thảo luận trong phần [“Data
Types”][data-types]<!-- ignore --> của Chapter 3.

<Listing number="8-2" caption="Tạo một vector mới chứa các giá trị">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-02/src/main.rs:here}}
```

</Listing>

Vì chúng ta đã cung cấp các giá trị `i32` ban đầu, Rust có thể suy ra rằng loại của `v`
là `Vec<i32>`, và type annotation không cần thiết. Tiếp theo, chúng ta sẽ xem cách
sửa đổi một vector.

### Cập Nhật một Vector

Để tạo một vector và sau đó thêm các phần tử vào nó, chúng ta có thể sử dụng method `push`,
như được hiển thị trong Listing 8-3.

<Listing number="8-3" caption="Sử dụng method `push` để thêm giá trị vào vector">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-03/src/main.rs:here}}
```

</Listing>

Giống như bất kỳ biến nào, nếu chúng ta muốn có thể thay đổi giá trị của nó, chúng ta cần
làm cho nó có thể thay đổi bằng cách sử dụng từ khóa `mut`, như được thảo luận trong Chapter 3. Các số
chúng ta đặt bên trong đều có loại `i32`, và Rust suy ra điều này từ dữ liệu, vì vậy
chúng ta không cần annotation `Vec<i32>`.

### Đọc Các Phần Tử của Vectors

Có hai cách để tham chiếu một giá trị được lưu trữ trong vector: thông qua indexing hoặc bằng
cách sử dụng method `get`. Trong các ví dụ sau, chúng ta đã chú thích các loại của
các giá trị được trả về từ các hàm này để rõ ràng hơn.

Listing 8-4 hiển thị cả hai phương pháp truy cập giá trị trong vector, với cú pháp indexing
và method `get`.

<Listing number="8-4" caption="Sử dụng cú pháp indexing và sử dụng method `get` để truy cập một mục trong vector">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-04/src/main.rs:here}}
```

</Listing>

Lưu ý một vài chi tiết ở đây. Chúng ta sử dụng giá trị index là `2` để lấy phần tử thứ ba
vì vectors được lập chỉ mục theo số, bắt đầu từ 0. Sử dụng `&` và `[]`
cho chúng ta một tham chiếu đến phần tử tại giá trị index. Khi chúng ta sử dụng method
`get` với index được truyền dưới dạng tham số, chúng ta nhận được một `Option<&T>` mà chúng ta có thể
sử dụng với `match`.

Rust cung cấp hai cách này để tham chiếu một phần tử để bạn có thể chọn cách
chương trình hoạt động khi bạn cố gắng sử dụng giá trị index ngoài phạm vi của
các phần tử hiện có. Ví dụ, hãy xem điều gì xảy ra khi chúng ta có một vector
gồm năm phần tử và sau đó chúng ta cố gắng truy cập một phần tử tại index 100 với mỗi
kỹ thuật, như được hiển thị trong Listing 8-5.

<Listing number="8-5" caption="Cố gắng truy cập phần tử tại index 100 trong vector chứa năm phần tử">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-05/src/main.rs:here}}
```

</Listing>

Khi chúng ta chạy code này, method `[]` đầu tiên sẽ làm chương trình panic
vì nó tham chiếu đến một phần tử không tồn tại. Method này được sử dụng tốt nhất khi bạn
muốn chương trình của mình crash nếu có cố gắng truy cập phần tử vượt quá
cuối của vector.

Khi method `get` được truyền một index nằm ngoài vector, nó trả về
`None` mà không panic. Bạn sẽ sử dụng method này nếu truy cập phần tử
vượt quá phạm vi của vector có thể xảy ra thỉnh thoảng dưới các
hoàn cảnh bình thường. Code của bạn sẽ có logic để xử lý việc có
`Some(&element)` hoặc `None`, như được thảo luận trong Chapter 6. Ví dụ, index
có thể đến từ một người nhập một số. Nếu họ vô tình nhập một
số quá lớn và chương trình nhận được giá trị `None`, bạn có thể nói với
người dùng có bao nhiêu mục trong vector hiện tại và cho họ cơ hội khác để
nhập giá trị hợp lệ. Điều đó sẽ thân thiện hơn với người dùng so với crash chương trình
do lỗi đánh máy!

Khi chương trình có một tham chiếu hợp lệ, borrow checker thực thi
ownership và borrowing rules (được đề cập trong Chapter 4) để đảm bảo rằng
tham chiếu này và bất kỳ tham chiếu nào khác đến nội dung của vector vẫn hợp lệ.
Nhớ lại quy tắc nói rằng bạn không thể có mutable và immutable references trong
cùng một scope. Quy tắc đó áp dụng trong Listing 8-6, nơi chúng ta giữ một immutable
reference đến phần tử đầu tiên trong vector và cố gắng thêm một phần tử vào
cuối. Chương trình này sẽ không hoạt động nếu chúng ta cũng cố gắng tham chiếu đến phần tử đó sau đó trong
hàm.

<Listing number="8-6" caption="Cố gắng thêm một phần tử vào vector trong khi giữ tham chiếu đến một mục">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-06/src/main.rs:here}}
```

</Listing>

Biên dịch code này sẽ dẫn đến lỗi này:

```console
{{#include ../listings/ch08-common-collections/listing-08-06/output.txt}}
```

Code trong Listing 8-6 có vẻ như nó nên hoạt động: Tại sao một tham chiếu
đến phần tử đầu tiên lại quan tâm đến những thay đổi ở cuối vector? Lỗi này là
do cách vectors hoạt động: Vì vectors đặt các giá trị cạnh nhau
trong bộ nhớ, thêm một phần tử mới vào cuối vector có thể yêu cầu
cấp phát bộ nhớ mới và sao chép các phần tử cũ vào không gian mới, nếu không có
đủ chỗ để đặt tất cả các phần tử cạnh nhau nơi vector
hiện đang được lưu trữ. Trong trường hợp đó, tham chiếu đến phần tử đầu tiên sẽ
trỏ đến bộ nhớ đã được giải phóng. Borrowing rules ngăn chặn các chương trình
khỏi kết thúc ở tình huống đó.

> Lưu ý: Để tìm hiểu thêm về chi tiết triển khai của kiểu `Vec<T>`, hãy xem [“The
> Rustonomicon”][nomicon].

### Lặp Lại Các Giá Trị trong một Vector

Để truy cập từng phần tử trong vector lần lượt, chúng ta sẽ lặp qua tất cả các
phần tử thay vì sử dụng indices để truy cập một cái tại một thời điểm. Listing 8-7 hiển thị cách
sử dụng vòng lặp `for` để lấy immutable references đến từng phần tử trong vector của
các giá trị `i32` và in chúng.

<Listing number="8-7" caption="In từng phần tử trong vector bằng cách lặp các phần tử bằng vòng lặp `for`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-07/src/main.rs:here}}
```

</Listing>

Chúng ta cũng có thể lặp qua mutable references đến từng phần tử trong một mutable vector
để thực hiện thay đổi đối với tất cả các phần tử. Vòng lặp `for` trong Listing 8-8
sẽ thêm `50` vào từng phần tử.

<Listing number="8-8" caption="Lặp qua mutable references đến các phần tử trong vector">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-08/src/main.rs:here}}
```

</Listing>

Để thay đổi giá trị mà mutable reference tham chiếu đến, chúng ta phải sử dụng
toán tử dereference `*` để truy cập giá trị trong `i` trước khi chúng ta có thể sử dụng toán tử `+=`.
Chúng ta sẽ nói nhiều hơn về toán tử dereference trong phần [“Following the
Reference to the Value”][deref]<!-- ignore --> của Chapter 15.

Lặp lại một vector, cho dù immutably hay mutably, là an toàn vì các
rules của borrow checker. Nếu chúng ta cố gắng chèn hoặc xóa các mục trong phần thân `for`
loop trong Listing 8-7 và Listing 8-8, chúng ta sẽ nhận được lỗi compiler
tương tự như lỗi chúng ta nhận được với code trong Listing 8-6. Tham chiếu đến
vector mà vòng lặp `for` giữ ngăn chặn sửa đổi đồng thời của
toàn bộ vector.

### Sử Dụng Enum để Lưu Trữ Nhiều Loại

Vectors chỉ có thể lưu trữ các giá trị có cùng loại. Điều này có thể
bất tiện; chắc chắn có các trường hợp sử dụng cần lưu trữ danh sách
các mục có các loại khác nhau. May mắn thay, các biến thể của một enum được định nghĩa
dưới cùng một loại enum, vì vậy khi chúng ta cần một loại để đại diện cho các phần tử của
các loại khác nhau, chúng ta có thể xác định và sử dụng enum!

Ví dụ, giả sử chúng ta muốn lấy các giá trị từ một hàng trong spreadsheet trong đó
một số cột trong hàng chứa các số nguyên, một số số thực,
và một số strings. Chúng ta có thể định nghĩa một enum có các biến thể sẽ giữ
các loại giá trị khác nhau, và tất cả các biến thể enum sẽ được coi là cùng một loại: loại
của enum. Sau đó, chúng ta có thể tạo một vector để giữ enum đó và do đó, cuối cùng,
giữ các loại khác nhau. Chúng ta đã minh họa điều này trong Listing 8-9.

<Listing number="8-9" caption="Định nghĩa một enum để lưu trữ các giá trị của các loại khác nhau trong một vector">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-09/src/main.rs:here}}
```

</Listing>

Rust cần biết những loại nào sẽ có trong vector tại thời điểm biên dịch để nó
biết chính xác bao nhiêu bộ nhớ trên heap sẽ cần thiết để lưu trữ mỗi phần tử.
Chúng ta cũng phải rõ ràng về những loại nào được phép trong vector này. Nếu Rust
cho phép vector giữ bất kỳ loại nào, sẽ có cơ hội một hoặc nhiều
loại sẽ gây ra lỗi với các hoạt động được thực hiện trên các phần tử của
vector. Sử dụng enum cộng với biểu thức `match` có nghĩa là Rust sẽ đảm bảo
tại thời điểm biên dịch rằng mọi trường hợp có thể xảy ra đều được xử lý, như được thảo luận trong Chapter 6.

Nếu bạn không biết tập hợp loại toàn diện mà chương trình sẽ nhận được tại runtime để
lưu trữ trong vector, kỹ thuật enum sẽ không hoạt động. Thay vào đó, bạn có thể sử dụng
object trait, mà chúng ta sẽ đề cập trong Chapter 18.

Bây giờ chúng ta đã thảo luận về một số cách phổ biến nhất để sử dụng vectors, hãy chắc chắn
xem lại [tài liệu API][vec-api]<!-- ignore --> cho tất cả các
methods hữu ích được định nghĩa trên `Vec<T>` bởi thư viện chuẩn. Ví dụ,
ngoài `push`, method `pop` loại bỏ và trả về phần tử cuối cùng.

### Dropping a Vector Drops Its Elements

Giống như bất kỳ `struct` nào khác, vector được giải phóng khi nó ra khỏi scope,
như được chú thích trong Listing 8-10.

<Listing number="8-10" caption="Hiển thị nơi vector và các phần tử của nó được dropped">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-10/src/main.rs:here}}
```

</Listing>

Khi vector bị dropped, tất cả nội dung của nó cũng bị dropped, có nghĩa là các
số nguyên mà nó giữ sẽ được dọn sạch. Borrow checker đảm bảo rằng bất kỳ
tham chiếu nào đến nội dung của vector chỉ được sử dụng trong khi chính vector đó là
hợp lệ.

Hãy chuyển sang loại collection tiếp theo: `String`!

[data-types]: ch03-02-data-types.html#data-types
[nomicon]: ../nomicon/vec/vec.html
[vec-api]: ../std/vec/struct.Vec.html
[deref]: ch15-02-deref.html#following-the-pointer-to-the-value-with-the-dereference-operator
