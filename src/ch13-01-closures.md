<!-- Old headings. Do not remove or links may break. -->

<a id="closures-anonymous-functions-that-can-capture-their-environment"></a>
<a id="closures-anonymous-functions-that-capture-their-environment"></a>

## Closures

Closures trong Rust là các function ẩn danh mà bạn có thể lưu trong một biến hoặc truyền như là argument cho các function khác. Bạn có thể tạo closure ở một nơi và sau đó gọi closure ở nơi khác để đánh giá nó trong một context khác. Không giống như function, closure có thể capture các giá trị từ scope mà chúng được định nghĩa. Chúng ta sẽ trình bày cách các tính năng của closure cho phép tái sử dụng code và tùy chỉnh hành vi.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-an-abstraction-of-behavior-with-closures"></a>
<a id="refactoring-using-functions"></a>
<a id="refactoring-with-closures-to-store-code"></a>
<a id="capturing-the-environment-with-closures"></a>

### Capturing Environment

Đầu tiên chúng ta sẽ xem xét cách sử dụng closure để capture các giá trị từ environment mà chúng được định nghĩa để sử dụng sau này. Đây là kịch bản: Thỉnh thoảng, công ty áo phông của chúng ta tặng một chiếc áo độc quyền, phiên bản giới hạn cho một người nào đó trong danh sách gửi thư của chúng ta như một chương trình khuyến mãi. Những người trong danh sách gửi thư có thể tùy chọn thêm màu yêu thích của họ vào hồ sơ. Nếu người được chọn để nhận áo miễn phí đã đặt màu yêu thích của họ, họ sẽ nhận được chiếc áo màu đó. Nếu người đó chưa chỉ định màu yêu thích, họ sẽ nhận được màu nào mà công ty hiện có nhiều nhất.

Có nhiều cách để triển khai điều này. Đối với ví dụ này, chúng ta sẽ sử dụng một enum có tên `ShirtColor` có các variant `Red` và `Blue` (giới hạn số lượng màu có sẵn để đơn giản hóa). Chúng ta biểu diễn kho hàng của công ty bằng một struct `Inventory` có một field tên là `shirts` chứa một `Vec<ShirtColor>` đại diện cho các màu áo hiện có trong kho. Method `giveaway` được định nghĩa trên `Inventory` nhận tùy chọn màu áo ưa thích của người chiến thắng áo miễn phí, và nó trả về màu áo mà người đó sẽ nhận được. Thiết lập này được hiển thị trong Listing 13-1.

<Listing number="13-1" file-name="src/main.rs" caption="Tình huống tặng áo của công ty áo phông">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-01/src/main.rs}}
```

</Listing>

`store` được định nghĩa trong `main` còn hai chiếc áo màu xanh lam và một chiếc áo màu đỏ để phân phối cho chương trình khuyến mãi phiên bản giới hạn này. Chúng ta gọi method `giveaway` cho một user có sở thích áo màu đỏ và một user không có bất kỳ sở thích nào.

Một lần nữa, code này có thể được triển khai theo nhiều cách, và ở đây, để tập trung vào closure, chúng ta đã sử dụng các khái niệm bạn đã học, ngoại trừ phần body của method `giveaway` sử dụng một closure. Trong method `giveaway`, chúng ta nhận sở thích của user như một parameter có type `Option<ShirtColor>` và gọi method `unwrap_or_else` trên `user_preference`. [Method `unwrap_or_else` trên `Option<T>`][unwrap-or-else]<!-- ignore --> được định nghĩa bởi thư viện chuẩn. Nó nhận một argument: một closure không có argument nào trả về một giá trị `T` (cùng type được lưu trong variant `Some` của `Option<T>`, trong trường hợp này là `ShirtColor`). Nếu `Option<T>` là variant `Some`, `unwrap_or_else` trả về giá trị từ bên trong `Some`. Nếu `Option<T>` là variant `None`, `unwrap_or_else` gọi closure và trả về giá trị được trả về bởi closure.

Chúng ta chỉ định closure expression `|| self.most_stocked()` làm argument cho `unwrap_or_else`. Đây là một closure không nhận parameter nào (nếu closure có parameter, chúng sẽ xuất hiện giữa hai dấu pipe dọc). Body của closure gọi `self.most_stocked()`. Chúng ta đang định nghĩa closure ở đây, và implementation của `unwrap_or_else` sẽ đánh giá closure sau này nếu kết quả được cần.

Chạy code này in ra như sau:

```console
{{#include ../listings/ch13-functional-features/listing-13-01/output.txt}}
```

Một khía cạnh thú vị ở đây là chúng ta đã truyền một closure gọi `self.most_stocked()` trên instance `Inventory` hiện tại. Thư viện chuẩn không cần biết bất cứ điều gì về các type `Inventory` hoặc `ShirtColor` mà chúng ta định nghĩa, hoặc logic mà chúng ta muốn sử dụng trong kịch bản này. Closure capture một reference bất biến đến instance `Inventory` `self` và truyền nó cùng với code mà chúng ta chỉ định cho method `unwrap_or_else`. Mặt khác, các function không thể capture environment của chúng theo cách này.

<!-- Old headings. Do not remove or links may break. -->

<a id="closure-type-inference-and-annotation"></a>

### Inferring và Annotating Closure Types

Có nhiều khác biệt hơn giữa function và closure. Closure thường không yêu cầu bạn annotate các type của parameter hoặc giá trị trả về như các function `fn` làm. Type annotation được yêu cầu trên function bởi vì các type là một phần của interface rõ ràng được expose cho user của bạn. Định nghĩa interface này một cách nghiêm ngặt là quan trọng để đảm bảo rằng mọi người đồng ý về các type của giá trị mà một function sử dụng và trả về. Mặt khác, closure không được sử dụng trong một interface được expose như thế này: Chúng được lưu trong các biến, và chúng được sử dụng mà không cần đặt tên cho chúng và expose chúng cho user của thư viện của chúng ta.

Closure thường ngắn và chỉ liên quan trong một context hẹp thay vì trong bất kỳ kịch bản tùy ý nào. Trong những context giới hạn này, compiler có thể suy luận các type của parameter và type trả về, tương tự như cách nó có thể suy luận các type của hầu hết các biến (có những trường hợp hiếm hoi mà compiler cần closure type annotation).

Cũng như với các biến, chúng ta có thể thêm type annotation nếu chúng ta muốn tăng tính rõ ràng và minh bạch với chi phí là dài dòng hơn mức cần thiết nghiêm ngặt. Annotating các type cho một closure sẽ trông giống như định nghĩa được hiển thị trong Listing 13-2. Trong ví dụ này, chúng ta đang định nghĩa một closure và lưu nó trong một biến thay vì định nghĩa closure tại chỗ mà chúng ta truyền nó như một argument, như chúng ta đã làm trong Listing 13-1.

<Listing number="13-2" file-name="src/main.rs" caption="Thêm các type annotation tùy chọn của các type parameter và giá trị trả về trong closure">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-02/src/main.rs:here}}
```

</Listing>

Với type annotation được thêm vào, cú pháp của closure trông tương tự hơn với cú pháp của function. Ở đây, chúng ta định nghĩa một function thêm 1 vào parameter của nó và một closure có cùng hành vi, để so sánh. Chúng ta đã thêm một số khoảng trắng để căn chỉnh các phần liên quan. Điều này minh họa cách cú pháp closure tương tự như cú pháp function ngoại trừ việc sử dụng pipe và số lượng cú pháp là tùy chọn:

```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

Dòng đầu tiên hiển thị định nghĩa function và dòng thứ hai hiển thị định nghĩa closure được annotate đầy đủ. Trong dòng thứ ba, chúng ta loại bỏ type annotation khỏi định nghĩa closure. Trong dòng thứ tư, chúng ta loại bỏ dấu ngoặc nhọn, là tùy chọn vì body của closure chỉ có một expression. Đây đều là các định nghĩa hợp lệ sẽ tạo ra cùng một hành vi khi chúng được gọi. Các dòng `add_one_v3` và `add_one_v4` yêu cầu closure được đánh giá để có thể compile vì các type sẽ được suy luận từ cách sử dụng của chúng. Điều này tương tự như `let v = Vec::new();` cần type annotation hoặc các giá trị của một type nào đó được chèn vào `Vec` để Rust có thể suy luận type.

Đối với định nghĩa closure, compiler sẽ suy luận một type cụ thể cho mỗi parameter của chúng và cho giá trị trả về của chúng. Ví dụ, Listing 13-3 hiển thị định nghĩa của một closure ngắn chỉ trả về giá trị mà nó nhận được như một parameter. Closure này không hữu ích lắm ngoại trừ cho mục đích của ví dụ này. Lưu ý rằng chúng ta chưa thêm bất kỳ type annotation nào vào định nghĩa. Bởi vì không có type annotation, chúng ta có thể gọi closure với bất kỳ type nào, điều mà chúng ta đã làm ở đây với `String` lần đầu tiên. Nếu sau đó chúng ta cố gắng gọi `example_closure` với một integer, chúng ta sẽ gặp lỗi.

<Listing number="13-3" file-name="src/main.rs" caption="Cố gắng gọi một closure mà các type được suy luận với hai type khác nhau">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-03/src/main.rs:here}}
```

</Listing>

Compiler cho chúng ta lỗi này:

```console
{{#include ../listings/ch13-functional-features/listing-13-03/output.txt}}
```

Lần đầu tiên chúng ta gọi `example_closure` với giá trị `String`, compiler suy luận type của `x` và type trả về của closure là `String`. Những type đó sau đó được khóa vào closure trong `example_closure`, và chúng ta gặp lỗi type khi chúng ta thử sử dụng một type khác với cùng closure lần tiếp theo.

### Capturing References hoặc Moving Ownership

Closure có thể capture các giá trị từ environment của chúng theo ba cách, ánh xạ trực tiếp đến ba cách mà một function có thể nhận một parameter: borrowing bất biến, borrowing có thể thay đổi, và lấy ownership. Closure sẽ quyết định sử dụng cách nào trong số này dựa trên những gì body của function làm với các giá trị đã được capture.

Trong Listing 13-4, chúng ta định nghĩa một closure capture một reference bất biến đến vector có tên `list` vì nó chỉ cần một reference bất biến để in giá trị.

<Listing number="13-4" file-name="src/main.rs" caption="Định nghĩa và gọi một closure capture một reference bất biến">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-04/src/main.rs}}
```

</Listing>

Ví dụ này cũng minh họa rằng một biến có thể bind với định nghĩa closure, và chúng ta có thể gọi closure sau này bằng cách sử dụng tên biến và dấu ngoặc đơn như thể tên biến là tên function.

Bởi vì chúng ta có thể có nhiều reference bất biến đến `list` cùng một lúc, `list` vẫn có thể truy cập được từ code trước định nghĩa closure, sau định nghĩa closure nhưng trước khi closure được gọi, và sau khi closure được gọi. Code này compile, chạy, và in ra:

```console
{{#include ../listings/ch13-functional-features/listing-13-04/output.txt}}
```

Tiếp theo, trong Listing 13-5, chúng ta thay đổi closure body để nó thêm một phần tử vào vector `list`. Closure bây giờ capture một reference có thể thay đổi.

<Listing number="13-5" file-name="src/main.rs" caption="Định nghĩa và gọi một closure capture một reference có thể thay đổi">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-05/src/main.rs}}
```

</Listing>

Code này compile, chạy, và in ra:

```console
{{#include ../listings/ch13-functional-features/listing-13-05/output.txt}}
```

Lưu ý rằng không còn `println!` giữa định nghĩa và lời gọi của closure `borrows_mutably`: Khi `borrows_mutably` được định nghĩa, nó capture một reference có thể thay đổi đến `list`. Chúng ta không sử dụng closure một lần nữa sau khi closure được gọi, vì vậy mutable borrow kết thúc. Giữa định nghĩa closure và lời gọi closure, một immutable borrow để in không được phép, bởi vì không có borrow nào khác được phép khi có một mutable borrow. Hãy thử thêm một `println!` ở đó để xem thông báo lỗi bạn nhận được!

Nếu bạn muốn buộc closure lấy ownership của các giá trị mà nó sử dụng trong environment ngay cả khi body của closure không thực sự cần ownership, bạn có thể sử dụng từ khóa `move` trước danh sách parameter.

Kỹ thuật này chủ yếu hữu ích khi truyền một closure cho một thread mới để di chuyển dữ liệu sao cho nó được sở hữu bởi thread mới. Chúng ta sẽ thảo luận về thread và tại sao bạn muốn sử dụng chúng chi tiết trong Chương 16 khi chúng ta nói về concurrency, nhưng bây giờ, hãy khám phá ngắn gọn việc spawn một thread mới bằng cách sử dụng một closure cần từ khóa `move`. Listing 13-6 hiển thị Listing 13-4 được sửa đổi để in vector trong một thread mới thay vì trong main thread.

<Listing number="13-6" file-name="src/main.rs" caption="Sử dụng `move` để buộc closure cho thread lấy ownership của `list`">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-06/src/main.rs}}
```

</Listing>

Chúng ta spawn một thread mới, đưa cho thread một closure để chạy như một argument. Closure body in ra list. Trong Listing 13-4, closure chỉ capture `list` bằng cách sử dụng một reference bất biến vì đó là lượng truy cập tối thiểu đến `list` cần thiết để in nó. Trong ví dụ này, mặc dù closure body vẫn chỉ cần một reference bất biến, chúng ta cần chỉ định rằng `list` nên được di chuyển vào closure bằng cách đặt từ khóa `move` ở đầu định nghĩa closure. Nếu main thread thực hiện nhiều thao tác hơn trước khi gọi `join` trên thread mới, thread mới có thể kết thúc trước khi phần còn lại của main thread kết thúc, hoặc main thread có thể kết thúc trước. Nếu main thread duy trì ownership của `list` nhưng kết thúc trước thread mới và drop `list`, reference bất biến trong thread sẽ không hợp lệ. Do đó, compiler yêu cầu `list` được di chuyển vào closure được đưa cho thread mới để reference sẽ hợp lệ. Hãy thử loại bỏ từ khóa `move` hoặc sử dụng `list` trong main thread sau khi closure được định nghĩa để xem compiler error bạn nhận được!

<!-- Old headings. Do not remove or links may break. -->

<a id="storing-closures-using-generic-parameters-and-the-fn-traits"></a>
<a id="limitations-of-the-cacher-implementation"></a>
<a id="moving-captured-values-out-of-the-closure-and-the-fn-traits"></a>
<a id="moving-captured-values-out-of-closures-and-the-fn-traits"></a>

### Moving Captured Values Out of Closures

Một khi closure đã capture một reference hoặc capture ownership của một giá trị từ environment nơi closure được định nghĩa (do đó ảnh hưởng đến những gì, nếu có, được di chuyển _vào_ closure), code trong body của closure định nghĩa điều gì xảy ra với các reference hoặc giá trị khi closure được đánh giá sau này (do đó ảnh hưởng đến những gì, nếu có, được di chuyển _ra khỏi_ closure).

Một closure body có thể làm bất kỳ điều nào sau đây: Di chuyển một giá trị đã capture ra khỏi closure, thay đổi giá trị đã capture, không di chuyển cũng không thay đổi giá trị, hoặc không capture gì từ environment ngay từ đầu.

Cách một closure capture và xử lý các giá trị từ environment ảnh hưởng đến trait nào mà closure implement, và trait là cách mà function và struct có thể chỉ định loại closure nào mà chúng có thể sử dụng. Closure sẽ tự động implement một, hai, hoặc cả ba trait `Fn` này, theo cách bổ sung, tùy thuộc vào cách body của closure xử lý các giá trị:

* `FnOnce` áp dụng cho các closure có thể được gọi một lần. Tất cả closure implement ít nhất trait này bởi vì tất cả closure có thể được gọi. Một closure di chuyển các giá trị đã capture ra khỏi body của nó sẽ chỉ implement `FnOnce` và không có trait `Fn` nào khác bởi vì nó chỉ có thể được gọi một lần.
* `FnMut` áp dụng cho các closure không di chuyển các giá trị đã capture ra khỏi body của chúng nhưng có thể thay đổi các giá trị đã capture. Những closure này có thể được gọi nhiều hơn một lần.
* `Fn` áp dụng cho các closure không di chuyển các giá trị đã capture ra khỏi body của chúng và không thay đổi các giá trị đã capture, cũng như các closure không capture gì từ environment của chúng. Những closure này có thể được gọi nhiều hơn một lần mà không thay đổi environment của chúng, điều này quan trọng trong các trường hợp như gọi một closure nhiều lần đồng thời.

Hãy xem xét định nghĩa của method `unwrap_or_else` trên `Option<T>` mà chúng ta đã sử dụng trong Listing 13-1:

```rust,ignore
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

Nhớ lại rằng `T` là type generic đại diện cho type của giá trị trong variant `Some` của một `Option`. Type `T` đó cũng là type trả về của function `unwrap_or_else`: Code gọi `unwrap_or_else` trên một `Option<String>`, ví dụ, sẽ nhận được một `String`.

Tiếp theo, lưu ý rằng function `unwrap_or_else` có type parameter generic bổ sung `F`. Type `F` là type của parameter có tên `f`, là closure mà chúng ta cung cấp khi gọi `unwrap_or_else`.

Trait bound được chỉ định trên type generic `F` là `FnOnce() -> T`, có nghĩa là `F` phải có thể được gọi một lần, không nhận argument, và trả về một `T`. Sử dụng `FnOnce` trong trait bound thể hiện ràng buộc rằng `unwrap_or_else` sẽ không gọi `f` nhiều hơn một lần. Trong body của `unwrap_or_else`, chúng ta có thể thấy rằng nếu `Option` là `Some`, `f` sẽ không được gọi. Nếu `Option` là `None`, `f` sẽ được gọi một lần. Bởi vì tất cả closure implement `FnOnce`, `unwrap_or_else` chấp nhận cả ba loại closure và linh hoạt nhất có thể.

> Lưu ý: Nếu những gì chúng ta muốn làm không yêu cầu capturing một giá trị từ environment, chúng ta có thể sử dụng tên của một function thay vì một closure nơi chúng ta cần thứ gì đó implement một trong các trait `Fn`. Ví dụ, trên một giá trị `Option<Vec<T>>`, chúng ta có thể gọi `unwrap_or_else(Vec::new)` để nhận một vector mới, rỗng nếu giá trị là `None`. Compiler tự động implement trait `Fn` nào áp dụng cho một định nghĩa function.

Bây giờ hãy xem xét method thư viện chuẩn `sort_by_key`, được định nghĩa trên slice, để xem nó khác với `unwrap_or_else` như thế nào và tại sao `sort_by_key` sử dụng `FnMut` thay vì `FnOnce` cho trait bound. Closure nhận một argument dưới dạng một reference đến item hiện tại trong slice đang được xem xét, và nó trả về một giá trị có type `K` có thể được sắp xếp. Function này hữu ích khi bạn muốn sắp xếp một slice theo một thuộc tính cụ thể của mỗi item. Trong Listing 13-7, chúng ta có một danh sách các instance `Rectangle`, và chúng ta sử dụng `sort_by_key` để sắp xếp chúng theo thuộc tính `width` từ thấp đến cao.

<Listing number="13-7" file-name="src/main.rs" caption="Sử dụng `sort_by_key` để sắp xếp các rectangle theo chiều rộng">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-07/src/main.rs}}
```

</Listing>

Code này in ra:

```console
{{#include ../listings/ch13-functional-features/listing-13-07/output.txt}}
```

Lý do `sort_by_key` được định nghĩa để nhận một closure `FnMut` là nó gọi closure nhiều lần: một lần cho mỗi item trong slice. Closure `|r| r.width` không capture, thay đổi, hoặc di chuyển bất cứ thứ gì ra khỏi environment của nó, vì vậy nó đáp ứng yêu cầu trait bound.

Ngược lại, Listing 13-8 hiển thị một ví dụ về một closure chỉ implement trait `FnOnce`, bởi vì nó di chuyển một giá trị ra khỏi environment. Compiler sẽ không cho phép chúng ta sử dụng closure này với `sort_by_key`.

<Listing number="13-8" file-name="src/main.rs" caption="Cố gắng sử dụng một closure `FnOnce` với `sort_by_key`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-08/src/main.rs}}
```

</Listing>

Đây là một cách giả tạo, phức tạp (không hoạt động) để thử đếm số lần `sort_by_key` gọi closure khi sắp xếp `list`. Code này cố gắng thực hiện việc đếm này bằng cách đẩy `value`—một `String` từ environment của closure—vào vector `sort_operations`. Closure capture `value` và sau đó di chuyển `value` ra khỏi closure bằng cách chuyển ownership của `value` cho vector `sort_operations`. Closure này có thể được gọi một lần; cố gắng gọi nó lần thứ hai sẽ không hoạt động, bởi vì `value` sẽ không còn trong environment để được đẩy vào `sort_operations` một lần nữa! Do đó, closure này chỉ implement `FnOnce`. Khi chúng ta cố gắng compile code này, chúng ta gặp lỗi này rằng `value` không thể được di chuyển ra khỏi closure vì closure phải implement `FnMut`:

```console
{{#include ../listings/ch13-functional-features/listing-13-08/output.txt}}
```

Lỗi chỉ đến dòng trong closure body di chuyển `value` ra khỏi environment. Để sửa lỗi này, chúng ta cần thay đổi closure body để nó không di chuyển các giá trị ra khỏi environment. Giữ một counter trong environment và tăng giá trị của nó trong closure body là một cách đơn giản hơn để đếm số lần closure được gọi. Closure trong Listing 13-9 hoạt động với `sort_by_key` vì nó chỉ capture một reference có thể thay đổi đến counter `num_sort_operations` và do đó có thể được gọi nhiều hơn một lần.

<Listing number="13-9" file-name="src/main.rs" caption="Sử dụng một closure `FnMut` với `sort_by_key` được phép.">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-09/src/main.rs}}
```

</Listing>

Các trait `Fn` quan trọng khi định nghĩa hoặc sử dụng function hoặc type sử dụng closure. Trong phần tiếp theo, chúng ta sẽ thảo luận về iterator. Nhiều method iterator nhận closure argument, vì vậy hãy ghi nhớ những chi tiết về closure này khi chúng ta tiếp tục!

[unwrap-or-else]: ../std/option/enum.Option.html#method.unwrap_or_else