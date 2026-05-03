## Sử dụng `Box<T>` để trỏ tới Dữ liệu trên Heap

Con trỏ thông minh đơn giản nhất là box, loại của nó được viết `Box<T>`. _Boxes_ cho phép bạn lưu trữ dữ liệu trên heap thay vì stack. Những gì vẫn nằm trên stack là con trỏ tới dữ liệu trên heap. Tham khảo Chapter 4 để xem lại sự khác biệt giữa stack và heap.

Boxes không có chi phí hiệu năng, ngoài việc lưu trữ dữ liệu của chúng trên heap thay vì trên stack. Nhưng chúng cũng không có nhiều khả năng bổ sung. Bạn sẽ sử dụng chúng phổ biến nhất trong các tình huống này:

- Khi bạn có một kiểu dữ liệu mà kích thước không thể được biết tại thời điểm compile, và bạn muốn sử dụng một giá trị của kiểu đó trong ngữ cảnh yêu cầu một kích thước chính xác
- Khi bạn có một lượng lớn dữ liệu, và bạn muốn chuyển ownership nhưng đảm bảo rằng dữ liệu sẽ không được sao chép khi bạn làm điều đó
- Khi bạn muốn sở hữu một giá trị, và bạn chỉ quan tâm rằng nó là một loại implement một trait cụ thể thay vì là một loại cụ thể

Chúng ta sẽ trình bày tình huống đầu tiên trong ["Enabling Recursive Types with Boxes"](#enabling-recursive-types-with-boxes). Trong trường hợp thứ hai, chuyển ownership của một lượng lớn dữ liệu có thể mất nhiều thời gian vì dữ liệu được sao chép xung quanh trên stack. Để cải thiện hiệu năng trong tình huống này, chúng ta có thể lưu trữ lượng dữ liệu lớn trên heap trong một box. Sau đó, chỉ lượng dữ liệu con trỏ nhỏ được sao chép xung quanh trên stack, trong khi dữ liệu mà nó tham chiếu vẫn ở một nơi trên heap. Trường hợp thứ ba được gọi là _trait object_, và ["Using Trait Objects to Abstract over Shared Behavior"][trait-objects] trong Chapter 18 được dành cho chủ đề đó. Vì vậy, những gì bạn học ở đây bạn sẽ áp dụng lại trong phần đó!

<!-- Old headings. Do not remove or links may break. -->

<a id="using-boxt-to-store-data-on-the-heap"></a>

### Lưu trữ Dữ liệu trên Heap

Trước khi chúng ta thảo luận về trường hợp sử dụng lưu trữ trên heap cho `Box<T>`, chúng ta sẽ bao quát cú pháp và cách tương tác với các giá trị được lưu trữ trong `Box<T>`.

Listing 15-1 cho thấy cách sử dụng box để lưu trữ một giá trị `i32` trên heap.

<Listing number="15-1" file-name="src/main.rs" caption="Lưu trữ một giá trị `i32` trên heap bằng cách sử dụng box">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-01/src/main.rs}}
```

</Listing>

Chúng ta định nghĩa biến `b` có giá trị của một `Box` trỏ tới giá trị `5`, được cấp phát trên heap. Chương trình này sẽ in ra `b = 5`; trong trường hợp này, chúng ta có thể truy cập dữ liệu trong box tương tự như cách chúng ta làm nếu dữ liệu này nằm trên stack. Giống như bất kỳ giá trị được sở hữu nào, khi một box đi ra khỏi scope, như `b` ở cuối `main`, nó sẽ được deallocate. Deallocation xảy ra cả cho box (được lưu trữ trên stack) và dữ liệu mà nó trỏ tới (được lưu trữ trên heap).

Đặt một giá trị duy nhất trên heap không phải là rất hữu ích, vì vậy bạn sẽ không sử dụng boxes riêng lẻ theo cách này thường xuyên. Có các giá trị như một `i32` duy nhất trên stack, nơi chúng được lưu trữ theo mặc định, là phù hợp hơn trong hầu hết các tình huống. Chúng ta hãy xem một trường hợp mà boxes cho phép chúng ta định nghĩa các kiểu dữ liệu mà chúng ta sẽ không được phép định nghĩa nếu chúng ta không có boxes.

### Cho phép các Kiểu Đệ quy với Boxes

Một giá trị của một _kiểu đệ quy_ có thể có một giá trị khác của cùng kiểu như một phần của chính nó. Các kiểu đệ quy gây ra vấn đề vì Rust cần biết tại thời điểm compile có bao nhiêu không gian mà một kiểu chiếm. Tuy nhiên, việc lồng nhau của các giá trị của các kiểu đệ quy có thể lý thuyết tiếp tục vô hạn, vì vậy Rust không thể biết cần bao nhiêu không gian cho giá trị. Vì boxes có một kích thước đã biết, chúng ta có thể cho phép các kiểu đệ quy bằng cách chèn một box vào định nghĩa kiểu đệ quy.

Như một ví dụ về một kiểu đệ quy, chúng ta hãy khám phá cons list. Đây là một loại dữ liệu thường được tìm thấy trong các ngôn ngữ lập trình hàm. Loại cons list mà chúng ta sẽ định nghĩa là đơn giản ngoại trừ đệ quy; do đó, các khái niệm trong ví dụ mà chúng ta sẽ làm việc sẽ hữu ích bất kỳ lúc nào bạn bước vào các tình huống phức tạp hơn liên quan đến các kiểu đệ quy.

<!-- Old headings. Do not remove or links may break. -->

<a id="more-information-about-the-cons-list"></a>

#### Hiểu Cons List

Một _cons list_ là một cấu trúc dữ liệu đến từ ngôn ngữ lập trình Lisp và các biến thể của nó, được tạo thành từ các cặp lồng nhau, và là phiên bản của Lisp của một linked list. Tên của nó đến từ hàm `cons` (viết tắt của _construct function_) trong Lisp có tác dụng xây dựng một cặp mới từ hai argument của nó. Bằng cách gọi `cons` trên một cặp bao gồm một giá trị và một cặp khác, chúng ta có thể xây dựng cons lists được tạo thành từ các cặp đệ quy.

Ví dụ, đây là một biểu diễn mã giả của một cons list chứa danh sách `1, 2, 3` với mỗi cặp trong dấu ngoặc:

```text
(1, (2, (3, Nil)))
```

Mỗi mục trong một cons list chứa hai phần tử: giá trị của mục hiện tại và của mục tiếp theo. Mục cuối cùng trong danh sách chứa chỉ một giá trị được gọi là `Nil` mà không có mục tiếp theo. Một cons list được tạo ra bằng cách gọi hàm `cons` một cách đệ quy. Tên chính tắc để biểu thị trường hợp cơ bản của đệ quy là `Nil`. Lưu ý rằng điều này không giống với khái niệm "null" hoặc "nil" được thảo luận trong Chapter 6, đó là một giá trị không hợp lệ hoặc vắng mặt.

Cons list không phải là một cấu trúc dữ liệu được sử dụng phổ biến trong Rust. Hầu hết thời gian khi bạn có một danh sách các mục trong Rust, `Vec<T>` là một lựa chọn tốt hơn để sử dụng. Các kiểu dữ liệu đệ quy phức tạp hơn khác _là_ hữu ích trong nhiều tình huống, nhưng bằng cách bắt đầu với cons list trong chapter này, chúng ta có thể khám phá cách boxes cho phép chúng ta định nghĩa một kiểu dữ liệu đệ quy mà không cần quá nhiều xao lãng.

Listing 15-2 chứa một định nghĩa enum cho một cons list. Lưu ý rằng code này sẽ không compile được, vì kiểu `List` không có kích thước đã biết, mà chúng ta sẽ trình bày.

<Listing number="15-2" file-name="src/main.rs" caption="Nỗ lực đầu tiên trong việc định nghĩa một enum để đại diện cho một cấu trúc dữ liệu cons list của các giá trị `i32`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-02/src/main.rs:here}}
```

</Listing>

> Lưu ý: Chúng ta đang implement một cons list chỉ giữ các giá trị `i32` cho mục đích của ví dụ này. Chúng ta có thể đã implement nó bằng cách sử dụng generics, như chúng ta đã thảo luận trong Chapter 10, để định nghĩa một loại cons list có thể lưu trữ các giá trị của bất kỳ loại nào.

Sử dụng kiểu `List` để lưu trữ danh sách `1, 2, 3` sẽ trông như code trong Listing 15-3.

<Listing number="15-3" file-name="src/main.rs" caption="Sử dụng enum `List` để lưu trữ danh sách `1, 2, 3`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-03/src/main.rs:here}}
```

</Listing>

Giá trị `Cons` đầu tiên giữ `1` và một giá trị `List` khác. Giá trị `List` này là một giá trị `Cons` khác giữ `2` và một giá trị `List` khác. Giá trị `List` này là một giá trị `Cons` khác nữa giữ `3` và một giá trị `List`, cuối cùng là `Nil`, biến thể không đệ quy báo hiệu kết thúc danh sách.

Nếu chúng ta cố gắng biên dịch code trong Listing 15-3, chúng ta sẽ nhận được lỗi được hiển thị trong Listing 15-4.

<Listing number="15-4" caption="Lỗi chúng ta nhận được khi cố gắng định nghĩa một enum đệ quy">

```console
{{#include ../listings/ch15-smart-pointers/listing-15-03/output.txt}}
```

</Listing>

Lỗi cho thấy kiểu này "có kích thước vô hạn." Lý do là chúng ta đã định nghĩa `List` với một biến thể là đệ quy: Nó giữ một giá trị khác của chính nó trực tiếp. Kết quả là, Rust không thể tìm ra cần bao nhiêu không gian để lưu trữ một giá trị `List`. Chúng ta hãy phân tích tại sao chúng ta nhận được lỗi này. Trước tiên, chúng ta sẽ xem Rust quyết định cần bao nhiêu không gian để lưu trữ một giá trị của một kiểu không đệ quy.

#### Tính kích thước của một kiểu không đệ quy

Hãy nhớ lại enum `Message` mà chúng ta đã định nghĩa trong Listing 6-2 khi chúng ta thảo luận về các định nghĩa enum trong Chapter 6:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

Để xác định bao nhiêu không gian cần cấp phát cho một giá trị `Message`, Rust đi qua từng biến thể để xem biến thể nào cần nhiều không gian nhất. Rust thấy rằng `Message::Quit` không cần bất kỳ không gian nào, `Message::Move` cần đủ không gian để lưu trữ hai giá trị `i32`, v.v. Vì chỉ một biến thể sẽ được sử dụng, không gian tối đa mà một giá trị `Message` sẽ cần là không gian mà nó sẽ chiếm để lưu trữ biến thể lớn nhất của nó.

Đối chiếu điều này với những gì xảy ra khi Rust cố gắng xác định bao nhiêu không gian mà một kiểu đệ quy như enum `List` trong Listing 15-2 cần. Trình biên dịch bắt đầu bằng cách xem xét biến thể `Cons`, giữ một giá trị của loại `i32` và một giá trị của loại `List`. Do đó, `Cons` cần một lượng không gian bằng kích thước của một `i32` cộng với kích thước của một `List`. Để tìm ra bao nhiêu bộ nhớ loại `List` cần, trình biên dịch xem xét các biến thể, bắt đầu với biến thể `Cons`. Biến thể `Cons` giữ một giá trị của loại `i32` và một giá trị của loại `List`, và quá trình này tiếp tục vô hạn, như được hiển thị trong Figure 15-1.

<img alt="An infinite Cons list: a rectangle labeled 'Cons' split into two smaller rectangles. The first smaller rectangle holds the label 'i32', and the second smaller rectangle holds the label 'Cons' and a smaller version of the outer 'Cons' rectangle. The 'Cons' rectangles continue to hold smaller and smaller versions of themselves until the smallest comfortably sized rectangle holds an infinity symbol, indicating that this repetition goes on forever." src="img/trpl15-01.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 15-1: Một `List` vô hạn bao gồm các biến thể `Cons` vô hạn</span>

<!-- Old headings. Do not remove or links may break. -->

<a id="using-boxt-to-get-a-recursive-type-with-a-known-size"></a>

#### Lấy một kiểu đệ quy với một kích thước đã biết

Vì Rust không thể tìm ra bao nhiêu không gian để cấp phát cho các kiểu được định nghĩa một cách đệ quy, trình biên dịch sẽ đưa ra một lỗi với đề xuất hữu ích:

<!-- manual-regeneration
after doing automatic regeneration, look at listings/ch15-smart-pointers/listing-15-03/output.txt and copy the relevant line
-->

```text
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to break the cycle
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +
```

Trong đề xuất này, _indirection_ có nghĩa là thay vì lưu trữ một giá trị trực tiếp, chúng ta nên thay đổi cấu trúc dữ liệu để lưu trữ giá trị gián tiếp bằng cách lưu trữ một con trỏ tới giá trị thay thế.

Vì `Box<T>` là một con trỏ, Rust luôn biết bao nhiêu không gian mà `Box<T>` cần: Kích thước của một con trỏ không thay đổi dựa trên lượng dữ liệu mà nó trỏ tới. Điều này có nghĩa là chúng ta có thể đặt một `Box<T>` bên trong biến thể `Cons` thay vì một giá trị `List` khác trực tiếp. `Box<T>` sẽ trỏ tới giá trị `List` tiếp theo sẽ nằm trên heap thay vì bên trong biến thể `Cons`.

Từ khía cạnh khái niệm, chúng ta vẫn có một danh sách, được tạo ra với các danh sách giữ các danh sách khác, nhưng implementation này bây giờ giống như việc đặt các mục bên cạnh nhau thay vì bên trong nhau.

Chúng ta có thể thay đổi định nghĩa của enum `List` trong Listing 15-2 và cách sử dụng `List` trong Listing 15-3 thành code trong Listing 15-5, sẽ biên dịch được.

<Listing number="15-5" file-name="src/main.rs" caption="Định nghĩa của `List` sử dụng `Box<T>` để có một kích thước đã biết">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-05/src/main.rs}}
```

</Listing>

Biến thể `Cons` cần kích thước của một `i32` cộng với không gian để lưu trữ dữ liệu con trỏ của box. Biến thể `Nil` không lưu trữ bất kỳ giá trị nào, vì vậy nó cần ít không gian hơn trên stack so với biến thể `Cons`. Bây giờ chúng ta biết rằng bất kỳ giá trị `List` nào sẽ chiếm kích thước của một `i32` cộng với kích thước của dữ liệu con trỏ của box. Bằng cách sử dụng box, chúng ta đã phá vỡ chuỗi đệ quy vô hạn, vì vậy trình biên dịch có thể tìm ra kích thước nó cần để lưu trữ một giá trị `List`. Figure 15-2 cho thấy biến thể `Cons` trông như thế nào bây giờ.

<img alt="A rectangle labeled 'Cons' split into two smaller rectangles. The first smaller rectangle holds the label 'i32', and the second smaller rectangle holds the label 'Box' with one inner rectangle that contains the label 'usize', representing the finite size of the box's pointer." src="img/trpl15-02.svg" class="center" />

<span class="caption">Figure 15-2: Một `List` không có kích thước vô hạn, vì `Cons` giữ một `Box`</span>

Boxes chỉ cung cấp indirection và heap allocation; chúng không có bất kỳ khả năng đặc biệt nào khác, như những khả năng chúng ta sẽ thấy với các loại con trỏ thông minh khác. Chúng cũng không có chi phí hiệu năng mà những khả năng đặc biệt này gây ra, vì vậy chúng có thể hữu ích trong các trường hợp như cons list nơi mà indirection là tính năng duy nhất chúng ta cần. Chúng ta sẽ xem xét thêm các trường hợp sử dụng cho boxes trong Chapter 18.

Loại `Box<T>` là một con trỏ thông minh vì nó implement trait `Deref`, cho phép các giá trị `Box<T>` được coi như các references. Khi một giá trị `Box<T>` đi ra khỏi scope, dữ liệu trên heap mà box trỏ tới cũng được làm sạch vì implement trait `Drop`. Hai trait này sẽ còn quan trọng hơn nữa đối với chức năng được cung cấp bởi các loại con trỏ thông minh khác mà chúng ta sẽ thảo luận trong phần còn lại của chapter này. Hãy khám phá hai trait này chi tiết hơn.

[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior