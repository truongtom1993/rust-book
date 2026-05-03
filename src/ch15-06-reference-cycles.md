## Vòng Tham Chiếu Có Thể Làm Rò Rỉ Bộ Nhớ

Những đảm bảo về an toàn bộ nhớ của Rust giúp giảm khó khăn, nhưng không phải
là không thể, để vô tình tạo ra bộ nhớ không bao giờ được dọn sạch (được gọi là
_memory leak_). Ngăn chặn rò rỉ bộ nhớ hoàn toàn không phải là một đảm bảo của
Rust, có nghĩa là rò rỉ bộ nhớ là an toàn về bộ nhớ trong Rust. Chúng ta có thể
thấy rằng Rust cho phép rò rỉ bộ nhớ bằng cách sử dụng `Rc<T>` và `RefCell<T>`:
Có thể tạo ra các tham chiếu trong đó các mục tham chiếu lẫn nhau theo một chu kỳ.
Điều này tạo ra rò rỉ bộ nhớ vì số lượng tham chiếu của mỗi mục trong chu kỳ sẽ
không bao giờ đạt 0, và các giá trị sẽ không bao giờ bị loại bỏ.

### Tạo Một Vòng Tham Chiếu

Hãy xem xét cách một vòng tham chiếu có thể xảy ra và cách ngăn chặn nó,
bắt đầu với định nghĩa của enum `List` và phương thức `tail` trong Listing
15-25.

<Listing number="15-25" file-name="src/main.rs" caption="Định nghĩa cons list chứa một `RefCell<T>` để chúng ta có thể sửa đổi những gì một variant `Cons` đang tham chiếu đến">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-25/src/main.rs:here}}
```

</Listing>

Chúng ta đang sử dụng một biến thể khác của định nghĩa `List` từ Listing 15-5.
Phần tử thứ hai trong variant `Cons` giờ đây là `RefCell<Rc<List>>`, có nghĩa
là thay vì có khả năng sửa đổi giá trị `i32` như chúng ta đã làm trong Listing
15-24, chúng ta muốn sửa đổi giá trị `List` mà một variant `Cons` đang trỏ tới.
Chúng ta cũng đang thêm một phương thức `tail` để thuận tiện cho chúng ta truy cập
mục thứ hai nếu chúng ta có một variant `Cons`.

Trong Listing 15-26, chúng ta đang thêm một hàm `main` sử dụng các định nghĩa
trong Listing 15-25. Đoạn mã này tạo ra một danh sách trong `a` và một danh
sách trong `b` trỏ tới danh sách trong `a`. Sau đó, nó sửa đổi danh sách trong
`a` để trỏ tới `b`, tạo ra một vòng tham chiếu. Có các câu lệnh `println!` dọc
theo để hiển thị số lượng tham chiếu tại các điểm khác nhau trong quy trình này.

<Listing number="15-26" file-name="src/main.rs" caption="Tạo một vòng tham chiếu của hai giá trị `List` trỏ tới nhau">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-26/src/main.rs:here}}
```

</Listing>

Chúng ta tạo một instance `Rc<List>` chứa một giá trị `List` trong biến `a`
với một danh sách ban đầu là `5, Nil`. Sau đó chúng ta tạo một instance
`Rc<List>` chứa một giá trị `List` khác trong biến `b` chứa giá trị `10` và
trỏ tới danh sách trong `a`.

Chúng ta sửa đổi `a` để nó trỏ tới `b` thay vì `Nil`, tạo ra một chu kỳ. Chúng
ta làm điều này bằng cách sử dụng phương thức `tail` để lấy một tham chiếu tới
`RefCell<Rc<List>>` trong `a`, mà chúng ta đặt trong biến `link`. Sau đó, chúng
ta sử dụng phương thức `borrow_mut` trên `RefCell<Rc<List>>` để thay đổi giá trị
bên trong từ một `Rc<List>` chứa giá trị `Nil` thành `Rc<List>` trong `b`.

Khi chúng ta chạy đoạn mã này, giữ lại `println!` cuối cùng được chú thích lại
tạm thời, chúng ta sẽ nhận được output này:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-26/output.txt}}
```

Số lượng tham chiếu của các instance `Rc<List>` trong cả `a` và `b` là 2 sau khi
chúng ta thay đổi danh sách trong `a` để trỏ tới `b`. Ở cuối `main`, Rust drop
biến `b`, điều này làm giảm số lượng tham chiếu của instance `Rc<List>` trong `b`
từ 2 xuống 1. Bộ nhớ mà `Rc<List>` có trên heap sẽ không bị drop tại thời điểm
này vì số lượng tham chiếu của nó là 1, không phải 0. Sau đó, Rust drop `a`,
điều này cũng giảm số lượng tham chiếu của instance `Rc<List>` trong `a` từ 2
xuống 1. Bộ nhớ của instance này cũng không thể bị drop, vì instance `Rc<List>`
khác vẫn tham chiếu nó. Bộ nhớ được cấp phát cho danh sách sẽ vẫn chưa được dọn
sạch mãi mãi. Để hình dung chu kỳ tham chiếu này, chúng ta đã tạo sơ đồ trong
Hình 15-4.

<img alt="A rectangle labeled 'a' that points to a rectangle containing the integer 5. A rectangle labeled 'b' that points to a rectangle containing the integer 10. The rectangle containing 5 points to the rectangle containing 10, and the rectangle containing 10 points back to the rectangle containing 5, creating a cycle." src="img/trpl15-04.svg" class="center" />

<span class="caption">Hình 15-4: Một vòng tham chiếu của danh sách `a` và `b`
trỏ tới nhau</span>

Nếu bạn bỏ chú thích dòng `println!` cuối cùng và chạy chương trình, Rust sẽ cố
gắng in chu kỳ này với `a` trỏ tới `b` trỏ tới `a` và cứ tiếp tục như vậy cho
đến khi tràn stack.

So với một chương trình thực tế, hậu quả của việc tạo một vòng tham chiếu trong
ví dụ này không phải là rất tồi tệ: Ngay sau khi chúng ta tạo vòng tham chiếu,
chương trình kết thúc. Tuy nhiên, nếu một chương trình phức tạp hơn cấp phát
rất nhiều bộ nhớ trong một chu kỳ và giữ nó trong thời gian dài, chương trình
sẽ sử dụng nhiều bộ nhớ hơn cần thiết và có thể làm quá tải hệ thống, khiến nó
không có bộ nhớ khả dụng.

Tạo các vòng tham chiếu không phải là dễ dàng, nhưng cũng không phải là không
thể. Nếu bạn có các giá trị `RefCell<T>` chứa các giá trị `Rc<T>` hoặc các
kết hợp lồng nhau tương tự của các loại có interior mutability và reference
counting, bạn phải đảm bảo rằng bạn không tạo chu kỳ; bạn không thể dựa vào Rust
để bắt chúng. Tạo một vòng tham chiếu sẽ là một lỗi logic trong chương trình
của bạn mà bạn nên sử dụng các bài kiểm tra tự động, đánh giá mã và các thực
hành phát triển phần mềm khác để giảm thiểu.

Một giải pháp khác để tránh vòng tham chiếu là tổ chức lại các cấu trúc dữ liệu
của bạn sao cho một số tham chiếu thể hiện ownership và một số tham chiếu không.
Kết quả là, bạn có thể có chu kỳ được tạo thành từ một số mối quan hệ ownership
và một số mối quan hệ non-ownership, và chỉ các mối quan hệ ownership ảnh hưởng
đến việc liệu một giá trị có thể bị drop hay không. Trong Listing 15-25, chúng
ta luôn muốn các variant `Cons` sở hữu danh sách của chúng, vì vậy việc tổ chức
lại cấu trúc dữ liệu là không thể. Hãy xem xét một ví dụ sử dụng đồ thị gồm
các nút cha và nút con để xem khi nào các mối quan hệ non-ownership là một cách
thích hợp để ngăn chặn vòng tham chiếu.

<!-- Old headings. Do not remove or links may break. -->

<a id="preventing-reference-cycles-turning-an-rct-into-a-weakt"></a>

### Ngăn Chặn Vòng Tham Chiếu Sử Dụng `Weak<T>`

Cho đến nay, chúng ta đã chứng minh rằng gọi `Rc::clone` tăng `strong_count`
của một instance `Rc<T>`, và một instance `Rc<T>` chỉ được dọn sạch nếu
`strong_count` của nó là 0. Bạn cũng có thể tạo một tham chiếu yếu tới giá trị
bên trong một instance `Rc<T>` bằng cách gọi `Rc::downgrade` và truyền một tham
chiếu tới `Rc<T>`. *Strong references* là cách bạn có thể chia sẻ ownership của
một instance `Rc<T>`. *Weak references* không thể hiện một mối quan hệ ownership,
và số lượng của chúng không ảnh hưởng đến khi nào một instance `Rc<T>` được dọn
sạch. Chúng sẽ không gây ra vòng tham chiếu, vì bất kỳ chu kỳ nào liên quan tới
một số tham chiếu yếu sẽ bị phá vỡ một khi strong reference count của các giá trị
liên quan là 0.

Khi bạn gọi `Rc::downgrade`, bạn nhận được một con trỏ thông minh loại `Weak<T>`.
Thay vì tăng `strong_count` trong instance `Rc<T>` lên 1, gọi `Rc::downgrade`
tăng `weak_count` lên 1. Loại `Rc<T>` sử dụng `weak_count` để theo dõi có bao
nhiêu tham chiếu `Weak<T>` tồn tại, tương tự như `strong_count`. Sự khác biệt
là `weak_count` không cần phải là 0 để instance `Rc<T>` được dọn sạch.

Vì giá trị mà `Weak<T>` tham chiếu có thể đã bị drop, để làm bất cứ điều gì với
giá trị mà `Weak<T>` đang trỏ tới, bạn phải chắc chắn rằng giá trị vẫn tồn tại.
Làm điều này bằng cách gọi phương thức `upgrade` trên một instance `Weak<T>`,
sẽ trả về `Option<Rc<T>>`. Bạn sẽ nhận được một kết quả `Some` nếu giá trị
`Rc<T>` chưa bị drop và kết quả `None` nếu giá trị `Rc<T>` đã bị drop. Vì
`upgrade` trả về `Option<Rc<T>>`, Rust sẽ đảm bảo rằng cả trường hợp `Some` và
trường hợp `None` đều được xử lý, và sẽ không có con trỏ không hợp lệ.

Ví dụ, thay vì sử dụng một danh sách có các mục chỉ biết về mục tiếp theo, chúng
ta sẽ tạo một cây có các mục biết về các mục con _và_ các mục cha của chúng.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-tree-data-structure-a-node-with-child-nodes"></a>

#### Tạo Cấu Trúc Dữ Liệu Cây

Để bắt đầu, chúng ta sẽ xây dựng một cây có các nút biết về các nút con của
chúng. Chúng ta sẽ tạo một struct có tên `Node` chứa giá trị `i32` của riêng nó
cũng như các tham chiếu tới các giá trị `Node` con của nó:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:here}}
```

Chúng ta muốn một `Node` sở hữu các con của nó, và chúng ta muốn chia sẻ ownership
đó với các biến sao cho chúng ta có thể truy cập trực tiếp từng `Node` trong cây.
Để làm điều này, chúng ta định nghĩa các mục `Vec<T>` là các giá trị của loại
`Rc<Node>`. Chúng ta cũng muốn sửa đổi nút nào là con của nút khác, vì vậy chúng
ta có một `RefCell<T>` trong `children` xung quanh `Vec<Rc<Node>>`.

Tiếp theo, chúng ta sẽ sử dụng định nghĩa struct của chúng ta và tạo một instance
`Node` có tên `leaf` với giá trị `3` và không có con, và một instance khác có tên
`branch` với giá trị `5` và `leaf` là một trong những con của nó, như được hiển thị
trong Listing 15-27.

<Listing number="15-27" file-name="src/main.rs" caption="Tạo một nút `leaf` không có con và một nút `branch` với `leaf` là một trong những con của nó">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:there}}
```

</Listing>

Chúng ta sao chép `Rc<Node>` trong `leaf` và lưu trữ nó trong `branch`, có nghĩa
là `Node` trong `leaf` giờ có hai chủ sở hữu: `leaf` và `branch`. Chúng ta có
thể đi từ `branch` tới `leaf` thông qua `branch.children`, nhưng không có cách
nào để đi từ `leaf` tới `branch`. Lý do là `leaf` không có tham chiếu tới
`branch` và không biết chúng có liên quan. Chúng ta muốn `leaf` biết rằng
`branch` là cha của nó. Chúng ta sẽ làm điều đó tiếp theo.

#### Thêm Một Tham Chiếu Từ Con Đến Cha Của Nó

Để làm cho nút con biết được cha của nó, chúng ta cần thêm một trường `parent`
vào định nghĩa struct `Node` của chúng ta. Sự rắc rối là ở việc quyết định loại
`parent` nên là gì. Chúng ta biết rằng nó không thể chứa `Rc<T>`, vì điều đó
sẽ tạo ra một vòng tham chiếu với `leaf.parent` trỏ tới `branch` và
`branch.children` trỏ tới `leaf`, điều này sẽ khiến các giá trị `strong_count`
của chúng không bao giờ là 0.

Suy nghĩ về các mối quan hệ theo một cách khác, một nút cha nên sở hữu các con
của nó: Nếu một nút cha bị drop, các nút con của nó cũng nên bị drop. Tuy nhiên,
một con không nên sở hữu cha của nó: Nếu chúng ta drop một nút con, cha vẫn
nên tồn tại. Đây là một trường hợp để sử dụng weak references!

Vì vậy, thay vì `Rc<T>`, chúng ta sẽ làm cho loại `parent` sử dụng `Weak<T>`,
cụ thể là `RefCell<Weak<Node>>`. Bây giờ định nghĩa struct `Node` của chúng ta
trông như thế này:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:here}}
```

Một nút sẽ có thể tham chiếu tới nút cha của nó nhưng không sở hữu cha. Trong
Listing 15-28, chúng ta cập nhật `main` để sử dụng định nghĩa mới này sao cho
nút `leaf` sẽ có một cách để tham chiếu tới cha của nó, `branch`.

<Listing number="15-28" file-name="src/main.rs" caption="Một nút `leaf` với một tham chiếu yếu tới nút cha của nó, `branch`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:there}}
```

</Listing>

Tạo nút `leaf` trông tương tự như Listing 15-27 ngoại trừ trường `parent`:
`leaf` bắt đầu mà không có cha, vì vậy chúng ta tạo một instance tham chiếu
`Weak<Node>` mới, rỗng.

Tại điểm này, khi chúng ta cố gắng lấy một tham chiếu tới cha của `leaf` bằng
cách sử dụng phương thức `upgrade`, chúng ta nhận được giá trị `None`. Chúng ta
thấy điều này trong output từ câu lệnh `println!` đầu tiên:

```text
leaf parent = None
```

Khi chúng ta tạo nút `branch`, nó cũng sẽ có một tham chiếu `Weak<Node>` mới
trong trường `parent` vì `branch` không có nút cha. Chúng ta vẫn có `leaf` là
một trong những con của `branch`. Khi chúng ta có instance `Node` trong `branch`,
chúng ta có thể sửa đổi `leaf` để cung cấp cho nó một tham chiếu `Weak<Node>`
tới cha của nó. Chúng ta sử dụng phương thức `borrow_mut` trên `RefCell<Weak<Node>>`
trong trường `parent` của `leaf`, và sau đó chúng ta sử dụng hàm `Rc::downgrade`
để tạo một tham chiếu `Weak<Node>` tới `branch` từ `Rc<Node>` trong `branch`.

Khi chúng ta in cha của `leaf` một lần nữa, lần này chúng ta sẽ nhận được một
variant `Some` chứa `branch`: Bây giờ `leaf` có thể truy cập cha của nó! Khi
chúng ta in `leaf`, chúng ta cũng tránh được chu kỳ cuối cùng dẫn tới tràn stack
như chúng ta có trong Listing 15-26; các tham chiếu `Weak<Node>` được in dưới
dạng `(Weak)`:

```text
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
```

Sự thiếu vắng output vô hạn cho thấy đoạn mã này không tạo vòng tham chiếu.
Chúng ta cũng có thể biết điều này bằng cách nhìn vào các giá trị chúng ta nhận
được từ việc gọi `Rc::strong_count` và `Rc::weak_count`.

#### Hình Dung Những Thay Đổi Trong `strong_count` và `weak_count`

Hãy xem xét cách các giá trị `strong_count` và `weak_count` của các instance
`Rc<Node>` thay đổi bằng cách tạo một scope bên trong mới và di chuyển việc
tạo `branch` vào scope đó. Bằng cách này, chúng ta có thể thấy điều gì xảy ra
khi `branch` được tạo và sau đó bị drop khi nó vượt ra ngoài scope. Các sửa đổi
được hiển thị trong Listing 15-29.

<Listing number="15-29" file-name="src/main.rs" caption="Tạo `branch` trong một scope bên trong và kiểm tra số lượng tham chiếu mạnh và yếu">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-29/src/main.rs:here}}
```

</Listing>

Sau khi `leaf` được tạo, `Rc<Node>` của nó có strong count là 1 và weak count
là 0. Trong scope bên trong, chúng ta tạo `branch` và liên kết nó với `leaf`,
tại thời điểm đó khi chúng ta in số lượng, `Rc<Node>` trong `branch` sẽ có
strong count là 1 và weak count là 1 (cho `leaf.parent` trỏ tới `branch` với
`Weak<Node>`). Khi chúng ta in số lượng trong `leaf`, chúng ta sẽ thấy nó sẽ
có strong count là 2 vì `branch` bây giờ có một bản sao của `Rc<Node>` của
`leaf` được lưu trữ trong `branch.children` nhưng vẫn sẽ có weak count là 0.

Khi scope bên trong kết thúc, `branch` vượt ra ngoài scope và strong count của
`Rc<Node>` giảm xuống 0, vì vậy `Node` của nó bị drop. Weak count là 1 từ
`leaf.parent` không có ảnh hưởng đến việc liệu `Node` có bị drop hay không,
vì vậy chúng ta không gặp phải rò rỉ bộ nhớ!

Nếu chúng ta cố gắng truy cập cha của `leaf` sau khi kết thúc scope, chúng ta
sẽ nhận được `None` một lần nữa. Ở cuối chương trình, `Rc<Node>` trong `leaf`
có strong count là 1 và weak count là 0 vì biến `leaf` bây giờ là tham chiếu
duy nhất tới `Rc<Node>` một lần nữa.

Tất cả logic quản lý số lượng và dropping giá trị được xây dựng trong `Rc<T>`
và `Weak<T>` và các cách triển khai trait `Drop` của chúng. Bằng cách chỉ định
rằng mối quan hệ từ con đến cha của nó nên là tham chiếu `Weak<T>` trong định
nghĩa `Node`, bạn có thể có các nút cha trỏ tới các nút con và ngược lại mà không
tạo ra vòng tham chiếu và rò rỉ bộ nhớ.

## Tóm Tắt

Chương này đã trình bày cách sử dụng smart pointers để đưa ra các đảm bảo khác
nhau và các trade-off khác so với những gì Rust làm theo mặc định với các tham
chiếu thông thường. Loại `Box<T>` có kích thước đã biết và trỏ tới dữ liệu được
cấp phát trên heap. Loại `Rc<T>` theo dõi số lượng tham chiếu tới dữ liệu trên
heap sao cho dữ liệu có thể có nhiều chủ sở hữu. Loại `RefCell<T>` với interior
mutability của nó cung cấp cho chúng ta một loại mà chúng ta có thể sử dụng khi
chúng ta cần một loại immutable nhưng cần thay đổi một giá trị bên trong của loại
đó; nó cũng thực thi các quy tắc mượn tại runtime thay vì lúc compile time.

Cũng được thảo luận là các trait `Deref` và `Drop`, cho phép rất nhiều chức
năng của smart pointers. Chúng ta đã khám phá vòng tham chiếu có thể gây rò rỉ
bộ nhớ và cách ngăn chặn chúng bằng cách sử dụng `Weak<T>`.

Nếu chương này đã gây hứng thú cho bạn và bạn muốn triển khai smart pointers
của riêng mình, hãy xem [“The Rustonomicon”][nomicon] để biết thêm thông tin
hữu ích.

Tiếp theo, chúng ta sẽ nói về concurrency trong Rust. Bạn thậm chí sẽ tìm
hiểu về một vài smart pointers mới.

[nomicon]: ../nomicon/index.html
