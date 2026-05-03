<!-- Old headings. Do not remove or links may break. -->

<a id="digging-into-the-traits-for-async"></a>

## Một Cái Nhìn Gần Hơn Vào Các Traits Cho Async

Trong suốt chương, chúng ta đã sử dụng các traits `Future`, `Stream`, và `StreamExt` theo các cách khác nhau. Cho đến nay, tuy nhiên, chúng ta đã tránh đi quá sâu vào các chi tiết của cách chúng hoạt động hoặc cách chúng vừa vào với nhau, điều đó tốt hầu hết thời gian cho công việc hàng ngày Rust của bạn. Đôi khi, tuy nhiên, bạn sẽ gặp phải tình huống mà bạn sẽ cần hiểu một vài chi tiết hơn của các traits này, cùng với kiểu `Pin` và trait `Unpin`. Trong phần này, chúng ta sẽ đào sâu vừa đủ để giúp trong những scenarios đó, vẫn để lại _really_ deep dive cho documentation khác.

<!-- Old headings. Do not remove or links may break. -->

<a id="future"></a>

### Trait `Future`

Hãy bắt đầu bằng cách nhìn gần hơn vào cách trait `Future` hoạt động. Đây là cách Rust định nghĩa nó:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

Định nghĩa trait đó bao gồm một bunch của các loại mới và cũng một vài cú pháp chúng ta chưa thấy trước, vì vậy hãy bước qua định nghĩa piece by piece.

Đầu tiên, kiểu liên kết `Output` của `Future` nói những gì future resolve thành. Điều này tương tự với kiểu liên kết `Item` cho trait `Iterator`. Thứ hai, `Future` có phương thức `poll`, nhận một `Pin` reference đặc biệt cho tham số `self` của nó và một mutable reference tới một `Context` type, và trả về một `Poll<Self::Output>`. Chúng ta sẽ nói thêm về `Pin` và `Context` trong một lát nữa. Bây giờ, hãy tập trung vào những gì phương thức trả về, kiểu `Poll`:

```rust
pub enum Poll<T> {
    Ready(T),
    Pending,
}
```

Kiểu `Poll` này giống với một `Option`. Nó có một variant mà có một giá trị, `Ready(T)`, và một không, `Pending`. `Poll` có nghĩa là cái gì đó khá khác biệt từ `Option`, tuy nhiên! Variant `Pending` chỉ ra rằng future vẫn có công việc phải làm, vì vậy người gọi sẽ cần kiểm tra lại sau. Variant `Ready` chỉ ra rằng `Future` đã kết thúc công việc của nó và giá trị `T` có sẵn.

> Ghi chú: Nó hiếm để cần phải gọi `poll` trực tiếp, nhưng nếu bạn cần, hãy ghi nhớ rằng với hầu hết futures, người gọi không nên gọi `poll` lại sau khi future đã trả về `Ready`. Nhiều futures sẽ panic nếu polled lại sau khi trở thành ready. Futures mà an toàn để poll lại sẽ nói vậy một cách rõ ràng trong documentation của chúng. Điều này tương tự như cách `Iterator::next` hoạt động.

Khi bạn thấy mã sử dụng `await`, Rust biên dịch nó dưới cùng thành mã mà gọi `poll`. Nếu bạn nhìn lại Listing 17-4, nơi chúng ta in out tiêu đề trang cho một URL duy nhất một khi nó resolved, Rust biên dịch nó vào một cái gì đó như (mặc dù không chính xác) như thế này:

```rust,ignore
match page_title(url).poll() {
    Ready(page_title) => match page_title {
        Some(title) => println!("The title for {url} was {title}"),
        None => println!("{url} had no title"),
    }
    Pending => {
        // But what goes here?
    }
}
```

Những gì chúng ta nên làm khi future vẫn `Pending`? Chúng ta cần một vài cách để thử lại, và lại, và lại, cho đến khi future cuối cùng sẵn sàng. Nói cách khác, chúng ta cần một loop:

```rust,ignore
let mut page_title_fut = page_title(url);
loop {
    match page_title_fut.poll() {
        Ready(value) => match page_title {
            Some(title) => println!("The title for {url} was {title}"),
            None => println!("{url} had no title"),
        }
        Pending => {
            // continue
        }
    }
}
```

Nếu Rust biên dịch nó tới chính xác cái mã đó, tuy nhiên, mỗi `await` sẽ là blocking—chính xác những gì ngược lại của những gì chúng ta đang đi cho! Thay vào đó, Rust đảm bảo rằng loop có thể tay kiểm soát thứ gì đó mà có thể tạm dừng công việc trên future này để làm việc trên futures khác và sau đó kiểm tra cái này lại sau. Như chúng ta đã thấy, cái gì đó là một async runtime, và công việc scheduling và coordination này là một trong những công việc chính của nó.

Trong phần ["Sending Data Between Two Tasks Using Message Passing"][message-passing]<!-- ignore -->, chúng ta đã mô tả chờ đợi trên `rx.recv`. Lệnh gọi `recv` trả về một future, và awaiting future polls nó. Chúng ta đã ghi chú rằng một runtime sẽ tạm dừng future cho đến khi nó sẵn sàng với `Some(message)` hoặc `None` khi channel đóng. Với hiểu biết sâu hơn của chúng ta về trait `Future`, và đặc biệt là `Future::poll`, chúng ta có thể thấy cách hoạt động. Runtime biết future không sẵn sàng khi nó trả về `Poll::Pending`. Ngược lại, runtime biết future _is_ sẵn sàng và tiến bộ nó khi `poll` trả về `Poll::Ready(Some(message))` hoặc `Poll::Ready(None)`.

Chi tiết chính xác của cách một runtime làm điều đó là ngoài phạm vi của cuốn sách này, nhưng điều chính là để thấy mechanics cơ bản của futures: một runtime _polls_ mỗi future nó chịu trách nhiệm cho, đặt future trở lại ngủ khi nó chưa sẵn sàng.

<!-- Old headings. Do not remove or links may break. -->

<a id="pinning-and-the-pin-and-unpin-traits"></a>
<a id="the-pin-and-unpin-traits"></a>

### Kiểu `Pin` Và Trait `Unpin`

Trở lại Listing 17-13, chúng ta đã sử dụng `trpl::join!` macro để await ba futures. Tuy nhiên, nó phổ biến để có một collection như một vector chứa một số futures mà sẽ không được biết cho đến runtime. Hãy thay đổi Listing 17-13 thành mã trong Listing 17-23 mà đặt ba futures vào một vector và gọi function `trpl::join_all` thay vào đó, sẽ không biên dịch chưa.

<Listing number="17-23" caption="Awaiting futures trong một collection"  file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-23/src/main.rs:here}}
```

</Listing>

Chúng ta đặt mỗi future trong một `Box` để làm chúng vào _trait objects_, giống như chúng ta đã làm trong phần "Returning Errors from `run`" ở Chương 12. (Chúng ta sẽ bao quanh trait objects chi tiết ở Chương 18.) Sử dụng trait objects cho phép chúng ta xử lý mỗi futures ẩn danh được sản xuất bởi các loại này như cùng kiểu dữ liệu, bởi vì tất cả chúng triển khai trait `Future`.

Điều này có thể là surprising. Sau tất cả, không có async blocks nào trả về bất cứ thứ gì, vì vậy mỗi cái tạo ra một `Future<Output = ()>`. Hãy nhớ rằng `Future` là một trait, tuy nhiên, và rằng trình biên dịch tạo một unique enum cho mỗi async block, thậm chí khi chúng có identical output types. Giống như bạn không thể đặt hai handwritten structs khác nhau vào một `Vec`, bạn không thể hỗn hợp compiler-generated enums.

Sau đó chúng ta truyền collection của futures đến function `trpl::join_all` và await kết quả. Tuy nhiên, điều này không biên dịch; đây là phần liên quan của các thông báo lỗi.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-23
cargo build
copy *only* the final `error` block from the errors
-->

```text
error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
  --> src/main.rs:48:33
   |
48 |         trpl::join_all(futures).await;
   |                                 ^^^^^ the trait `Unpin` is not implemented for `dyn Future<Output = ()>`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<dyn Future<Output = ()>>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`
```

Ghi chú trong thông báo lỗi này cho chúng ta biết rằng chúng ta nên sử dụng `pin!` macro để _pin_ các giá trị, có nghĩa là đặt chúng bên trong kiểu `Pin` mà đảm bảo các giá trị sẽ không được di chuyển trong bộ nhớ. Thông báo lỗi nói pinning là bắt buộc vì `dyn Future<Output = ()>` cần triển khai trait `Unpin` và hiện tại nó không.

Function `trpl::join_all` trả về một struct được gọi là `JoinAll`. Struct đó là generic qua một kiểu `F`, mà được ràng buộc để triển khai trait `Future`. Trực tiếp awaiting một future với `await` pins future một cách ngầm. Đó là lý do tại sao chúng ta không cần sử dụng `pin!` ở mọi nơi chúng ta muốn await futures.

Tuy nhiên, chúng ta không trực tiếp awaiting một future ở đây. Thay vào đó, chúng ta xây dựng một future mới, JoinAll, bằng cách truyền một collection của futures đến function `join_all`. Signature cho `join_all` yêu cầu rằng các loại của các items trong collection tất cả triển khai trait `Future`, và `Box<T>` triển khai `Future` chỉ nếu `T` nó bao bọc là một future mà triển khai trait `Unpin`.

Đó là rất nhiều để hấp thụ! Để thực sự hiểu nó, hãy đào sâu một chút hơn vào cách trait `Future` thực sự hoạt động, đặc biệt xung quanh pinning. Nhìn lại ở định nghĩa của trait `Future`:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    // Required method
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

Tham số `cx` và kiểu `Context` của nó là chìa khóa cho cách một runtime thực sự biết khi nào để kiểm tra bất kỳ future nhất định nào trong khi vẫn còn lười biếng. Một lần nữa, chi tiết chính xác của cách hoạt động là ngoài phạm vi của chương này, và bạn thường chỉ cần suy nghĩ về điều này khi viết một custom `Future` triển khai. Chúng ta sẽ tập trung thay vào loại cho `self`, khi lần đầu tiên chúng ta thấy một phương thức nơi `self` có một type annotation. Một type annotation cho `self` hoạt động giống như type annotations cho các function parameters khác nhưng có hai khác biệt chính:

- Nó cho Rust biết kiểu `self` phải là cho phương thức được gọi.
- Nó không thể chỉ là bất kỳ loại nào. Nó được hạn chế với loại mà phương thức được triển khai, một reference hoặc smart pointer tới loại đó, hoặc một `Pin` bao bọc một reference tới loại đó.

Chúng ta sẽ thấy thêm trong cú pháp này ở [Chương 18][ch-18]<!-- ignore -->. Bây giờ, nó đủ để biết rằng nếu chúng ta muốn poll một future để kiểm tra xem liệu nó có `Pending` hoặc `Ready(Output)`, chúng ta cần một `Pin`-wrapped mutable reference tới loại.

`Pin` là một wrapper cho pointer-like types như `&`, `&mut`, `Box`, và `Rc`. (Kỹ thuật, `Pin` hoạt động với các loại triển khai `Deref` hoặc `DerefMut` traits, nhưng điều này hiệu quả tương đương với làm việc chỉ với references và smart pointers.) `Pin` không phải là một pointer chính nó và không có bất kỳ hành vi nào của riêng nó như `Rc` và `Arc` làm với reference counting; nó hoàn toàn là một tool trình biên dịch có thể sử dụng để thực hiện ràng buộc trên pointer usage.

Recalling rằng `await` được triển khai theo các điều khoản của các lệnh gọi đến `poll` bắt đầu để giải thích thông báo lỗi chúng ta thấy trước, nhưng điều đó là theo các điều khoản của `Unpin`, không phải `Pin`. Vì vậy, làm cách nào chính xác `Pin` liên quan đến `Unpin`, và tại sao `Future` cần `self` để là trong một `Pin` loại để gọi `poll`?

Hãy nhớ từ sớm trong chương này rằng một series của await points trong một future nhận được biên dịch vào một state machine, và trình biên dịch đảm bảo rằng state machine đó theo dõi tất cả các quy tắc bình thường của Rust xung quanh an toàn, bao gồm borrowing và ownership. Để làm điều đó hoạt động, Rust nhìn vào dữ liệu gì được cần giữa một await point và bất kỳ await point tiếp theo hoặc kết thúc của async block. Nó sau đó tạo một corresponding variant trong compiled state machine. Mỗi variant nhận được access mà nó cần tới dữ liệu mà sẽ được sử dụng trong phần đó của source code, liệu bằng cách lấy ownership của dữ liệu đó hoặc bằng cách nhận một mutable hoặc immutable reference tới nó.

Cho đến nay, như vậy: nếu chúng ta nhận được bất cứ điều gì sai về ownership hoặc references trong một async block nhất định, borrow checker sẽ cho chúng ta biết. Khi chúng ta muốn di chuyển quanh future mà tương ứng với block—như di chuyển nó vào một `Vec` để truyền `join_all`—những thứ nhận được trickier.

Khi chúng ta di chuyển một future—dù bằng cách push nó vào một data structure để sử dụng như một iterator với `join_all` hoặc bằng cách trả về nó từ một function—điều đó thực sự có nghĩa là di chuyển state machine Rust tạo cho chúng ta. Và không giống như hầu hết các loại khác trong Rust, futures Rust tạo cho async blocks có thể kết thúc với references tới chính chúng trong các fields của bất kỳ variant nhất định, như thể hiện trong simplified illustration trong Figure 17-4.

<figure>

<img alt="A single-column, three-row table representing a future, fut1, which has data values 0 and 1 in the first two rows and an arrow pointing from the third row back to the second row, representing an internal reference within the future." src="img/trpl17-04.svg" class="center" />

<figcaption>Figure 17-4: A self-referential data type</figcaption>

</figure>

Mặc dù theo mặc định, bất kỳ object nào có một reference tới chính nó là unsafe để di chuyển, bởi vì references luôn luôn chỉ đến địa chỉ memory thực tế của bất kỳ cái gì chúng chỉ tới (xem Figure 17-5). Nếu bạn di chuyển data structure chính nó, các internal references đó sẽ bị bỏ lại chỉ vào vị trí cũ. Tuy nhiên, memory location đó bây giờ là invalid. Đối với một điều, giá trị của nó sẽ không được cập nhật khi bạn tạo các changes tới data structure. Đối với một—more important—thứ, máy tính bây giờ là free để tái sử dụng memory đó cho các mục đích khác! Bạn có thể kết thúc đọc completely unrelated data sau.

<figure>

<img alt="Two tables, depicting two futures, fut1 and fut2, each of which has one column and three rows, representing the result of having moved a future out of fut1 into fut2. The first, fut1, is grayed out, with a question mark in each index, representing unknown memory. The second, fut2, has 0 and 1 in the first and second rows and an arrow pointing from its third row back to the second row of fut1, representing a pointer that is referencing the old location in memory of the future before it was moved." src="img/trpl17-05.svg" class="center" />

<figcaption>Figure 17-5: The unsafe result of moving a self-referential data type</figcaption>

</figure>

Theo lý thuyết, trình biên dịch Rust có thể cố gắng cập nhật mỗi reference tới một object bất cứ khi nào nó nhận được di chuyển, nhưng điều đó có thể thêm rất nhiều performance overhead, đặc biệt nếu một toàn bộ web của references cần cập nhật. Nếu chúng ta có thể thay vào đó đảm bảo data structure trong câu hỏi _doesn't di chuyển trong memory_, chúng ta không phải cập nhật bất kỳ references. Đây là chính xác những gì Rust's borrow checker cho: trong safe code, nó ngăn chặn bạn khỏi di chuyển bất kỳ item nào có một active reference tới nó.

`Pin` xây dựng trên đó để cung cấp chúng ta những bảo đảm chính xác mà chúng ta cần. Khi chúng ta _pin_ một giá trị bằng cách bao bọc một pointer tới giá trị đó trong `Pin`, nó không thể di chuyển lên. Do đó, nếu bạn có `Pin<Box<SomeType>>`, bạn thực sự pin giá trị `SomeType`, _not_ `Box` pointer. Figure 17-6 minh họa quá trình này.

<figure>

<img alt="Three boxes laid out side by side. The first is labeled "Pin", the second "b1", and the third "pinned". Within "pinned" is a table labeled "fut", with a single column; it represents a future with cells for each part of the data structure. Its first cell has the value "0", its second cell has an arrow coming out of it and pointing to the fourth and final cell, which has the value "1" in it, and the third cell has dashed lines and an ellipsis to indicate there may be other parts to the data structure. All together, the "fut" table represents a future which is self-referential. An arrow leaves the box labeled "Pin", goes through the box labeled "b1" and terminates inside the "pinned" box at the "fut" table." src="img/trpl17-06.svg" class="center" />

<figcaption>Figure 17-6: Pinning a `Box` that points to a self-referential future type</figcaption>

</figure>

Trong thực tế, `Box` pointer vẫn có thể di chuyển quanh một cách tự do. Hãy nhớ: chúng ta quan tâm về việc đảm bảo data ultimately được referenced stays in place. Nếu một pointer di chuyển quanh, _nhưng data mà nó chỉ đến_ là trong cùng một place, như trong Figure 17-7, không có potential problem. (Như một independent exercise, nhìn vào docs cho các loại cũng như `std::pin` module và cố gắng làm việc ra cách bạn sẽ làm điều này với một `Pin` bao bọc một `Box`.) Chìa khóa là self-referential type chính nó không thể di chuyển, bởi vì nó vẫn còn pinned.

<figure>

<img alt="Four boxes laid out in three rough columns, identical to the previous diagram with a change to the second column. Now there are two boxes in the second column, labeled "b1" and "b2", "b1" is grayed out, and the arrow from "Pin" goes through "b2" instead of "b1", indicating that the pointer has moved from "b1" to "b2", but the data in "pinned" has not moved." src="img/trpl17-07.svg" class="center" />

<figcaption>Figure 17-7: Moving a `Box` which points to a self-referential future type</figcaption>

</figure>

Tuy nhiên, hầu hết các loại được hoàn toàn an toàn để di chuyển quanh, thậm chí nếu chúng xảy ra để được đằng sau một `Pin` pointer. Chúng ta chỉ cần suy nghĩ về pinning khi items có internal references. Primitive values như numbers và Booleans được an toàn bởi vì chúng rõ ràng không có bất kỳ internal references nào. Cũng không làm hầu hết các loại bạn bình thường làm việc với trong Rust. Bạn có thể di chuyển quanh một `Vec`, ví dụ, mà không lo lắng. Được đưa ra những gì chúng ta đã thấy cho đến nay, nếu bạn có một `Pin<Vec<String>>`, bạn phải làm mọi thứ qua safe nhưng restrictive APIs được cung cấp bởi `Pin`, thậm chí mặc dù `Vec<String>` luôn luôn an toàn để di chuyển nếu không có các references khác tới nó. Chúng ta cần một cách để cho trình biên dịch biết rằng nó được ok để di chuyển các items quanh trong trường hợp như thế này—và đó là nơi `Unpin` đi vào.

`Unpin` là một marker trait, tương tự đến `Send` và `Sync` traits chúng ta thấy ở Chương 16, và do đó có không có chức năng của riêng nó. Marker traits tồn tại chỉ để cho trình biên dịch biết nó an toàn để sử dụng loại triển khai một trait nhất định trong một particular context. `Unpin` thông báo rằng trình biên dịch rằng một loại nhất định không _need_ để uphold bất kỳ bảo đảm nào về liệu giá trị trong câu hỏi có thể được di chuyển một cách an toàn hay không.

<!--
  The inline `<code>` in the next block is to allow the inline `<em>` inside it,
  matching what NoStarch does style-wise, and emphasizing within the text here
  that it is something distinct from a normal type.
-->

Giống như với `Send` và `Sync`, trình biên dịch triển khai `Unpin` tự động cho tất cả các loại nơi nó có thể chứng minh nó an toàn. Một special case, lại tương tự đến `Send` và `Sync`, là nơi `Unpin` _not_ được triển khai cho một loại. Ký hiệu cho điều này là <code>impl !Unpin for <em>SomeType</em></code>, nơi <code><em>SomeType</em></code> là tên của một loại mà _does_ cần uphold những bảo đảm đó để an toàn bất cứ khi nào một pointer tới loại đó được sử dụng trong một `Pin`.

Nói cách khác, có hai những điều để giữ trong tâm trí về mối quan hệ giữa `Pin` và `Unpin`. Đầu tiên, `Unpin` là "normal" case, và `!Unpin` là special case. Thứ hai, liệu một loại triển khai `Unpin` hoặc `!Unpin` _only_ vấn đề khi bạn đang sử dụng một pinned pointer tới loại đó như <code>Pin<&mut <em>SomeType</em>></code>.

Để làm điều đó cụ thể, suy nghĩ về một `String`: nó có một length và các Unicode characters mà tạo nó lên. Chúng ta có thể bao bọc một `String` trong `Pin`, như thấy trong Figure 17-8. Tuy nhiên, `String` tự động triển khai `Unpin`, như làm hầu hết các loại khác trong Rust.

<figure>

<img alt="A box labeled "Pin" on the left with an arrow going from it to a box labeled "String" on the right. The "String" box contains the data 5usize, representing the length of the string, and the letters "h", "e", "l", "l", and "o" representing the characters of the string "hello" stored in this String instance. A dotted rectangle surrounds the "String" box and its label, but not the "Pin" box." src="img/trpl17-08.svg" class="center" />

<figcaption>Figure 17-8: Pinning a `String`; the dotted line indicates that the `String` implements the `Unpin` trait and thus is not pinned</figcaption>

</figure>

Như một kết quả, chúng ta có thể làm những điều mà sẽ là bất hợp pháp nếu `String` triển khai `!Unpin` thay vào đó, như thay thế một string với một string khác tại chính xác cùng một vị trí trong memory như trong Figure 17-9. Điều này không vi phạm `Pin` contract, bởi vì `String` không có internal references mà làm cho nó unsafe để di chuyển quanh. Đó chính xác là tại sao nó triển khai `Unpin` thay vì `!Unpin`.

<figure>

<img alt="The same "hello" string data from the previous example, now labeled "s1" and grayed out. The "Pin" box from the previous example now points to a different String instance, one that is labeled "s2", is valid, has a length of 7usize, and contains the characters of the string "goodbye". s2 is surrounded by a dotted rectangle because it, too, implements the Unpin trait." src="img/trpl17-09.svg" class="center" />

<figcaption>Figure 17-9: Replacing the `String` with an entirely different `String` in memory</figcaption>

</figure>

Bây giờ chúng ta biết đủ để hiểu các lỗi được báo cáo cho lệnh gọi `join_all` đó từ ở lại ở Listing 17-23. Chúng ta đã cố gắng di chuyển futures được tạo ra bởi async blocks vào một `Vec<Box<dyn Future<Output = ()>>>`, nhưng như chúng ta đã thấy, những futures đó có thể có internal references, vì vậy chúng không tự động triển khai `Unpin`. Khi chúng ta pin chúng, chúng ta có thể truyền resulting `Pin` type vào `Vec`, confident rằng underlying data trong futures sẽ _not_ được di chuyển. Listing 17-24 hiển thị cách fix mã bằng cách gọi `pin!` macro nơi mỗi ba futures được định nghĩa và điều chỉnh trait object type.

<Listing number="17-24" caption="Pinning futures để enable di chuyển chúng vào vector">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-24/src/main.rs:here}}
```

</Listing>

Ví dụ này bây giờ biên dịch và chạy, và chúng ta có thể thêm hoặc loại bỏ futures từ vector tại runtime và join chúng tất cả.

`Pin` và `Unpin` đều quan trọng chủ yếu cho xây dựng lower-level libraries, hoặc khi bạn đang xây dựng một runtime chính nó, thay vì cho day-to-day Rust code. Khi bạn thấy các traits này trong thông báo lỗi, tuy nhiên, bây giờ bạn sẽ có một ý tưởng tốt hơn về cách fix mã của bạn!

> Ghi chú: Combination này của `Pin` và `Unpin` làm nó có thể an toàn để triển khai một toàn bộ class của complex types trong Rust mà sẽ nếu không chứng minh challenging vì chúng là self-referential. Types mà yêu cầu `Pin` hiển thị up hầu hết thường xuyên trong async Rust ngày nay, nhưng mỗi mỗi lúc, bạn có thể thấy chúng trong các contexts khác, quá.
>
> Specifics của cách `Pin` và `Unpin` hoạt động, và các quy tắc chúng được yêu cầu để uphold, được bao quanh mở rộng trong API documentation cho `std::pin`, vì vậy nếu bạn quan tâm đến việc tìm hiểu thêm, đó là một great place để bắt đầu.
>
> Nếu bạn muốn hiểu cách những thứ hoạt động dưới cùng thậm chí chi tiết hơn, xem Chapters [2][under-the-hood]<!-- ignore --> và [4][pinning]<!-- ignore --> của [_Asynchronous Programming in Rust_][async-book].

### Trait `Stream`

Bây giờ rằng bạn có một deeper grasp trên `Future`, `Pin`, và `Unpin` traits, chúng ta có thể quay sự chú ý của chúng ta tới trait `Stream`. Như bạn đã học sớm trong chương, streams giống như asynchronous iterators. Không giống như `Iterator` và `Future`, tuy nhiên, `Stream` có không có định nghĩa trong standard library kể từ lúc viết này, nhưng có là một rất common định nghĩa từ crate `futures` được sử dụng trong toàn bộ hệ sinh thái.

Hãy xem lại các định nghĩa của `Iterator` và `Future` traits trước khi nhìn vào cách trait `Stream` có thể hợp nhất chúng lại với nhau. Từ `Iterator`, chúng ta có ý tưởng của một sequence: phương thức `next` của nó cung cấp một `Option<Self::Item>`. Từ `Future`, chúng ta có ý tưởng của readiness qua time: phương thức `poll` của nó cung cấp một `Poll<Self::Output>`. Để đại diện cho một sequence của items mà trở thành ready qua time, chúng ta xác định một trait `Stream` mà đặt những features đó với nhau:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

trait Stream {
    type Item;

    fn poll_next(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>
    ) -> Poll<Option<Self::Item>>;
}
```

Trait `Stream` xác định một kiểu liên kết được gọi là `Item` cho loại của các items được tạo ra bởi stream. Điều này tương tự đến `Iterator`, nơi có thể zero tới nhiều items, và không giống như `Future`, nơi là luôn có một single `Output`, thậm chí nếu nó là unit type `()`.

`Stream` cũng xác định một phương thức để nhận những items. Chúng ta gọi nó `poll_next`, để làm rõ ràng rằng nó polls theo cùng cách `Future::poll` làm và tạo ra một sequence của items theo cùng cách `Iterator::next` làm. Return type của nó kết hợp `Poll` với `Option`. Outer type là `Poll`, bởi vì nó có khác kiểm tra cho readiness, giống như một future làm. Inner type là `Option`, bởi vì nó cần signal liệu có nhiều messages, giống như một iterator làm.

Một cái gì đó rất tương tự như định nghĩa này sẽ có thể kết thúc như một phần của Rust's standard library. Trong khoảng thời gian, nó là phần của toolkit của hầu hết runtimes, vì vậy bạn có thể dựa trên nó, và mọi thứ chúng ta bao quanh next sẽ nói chung áp dụng!

Trong các ví dụ chúng ta thấy trong phần ["Streams: Futures in Sequence"][streams]<!-- ignore -->, tuy nhiên, chúng ta không đã sử dụng `poll_next` _or_ `Stream`, nhưng thay vào đó đã sử dụng `next` và `StreamExt`. Chúng ta _could_ làm việc trực tiếp theo các điều khoản của `poll_next` API bằng cách hand-writing state machines của chúng ta, tất nhiên, giống như chúng ta _could_ làm việc với futures trực tiếp qua phương thức `poll` của chúng. Sử dụng `await` là rất nhiều nicer, tuy nhiên, và trait `StreamExt` cung cấp phương thức `next` vì vậy chúng ta có thể làm chính xác điều đó:

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-stream-ext/src/lib.rs:here}}
```

<!--
TODO: update this if/when tokio/etc. update their MSRV and switch to using async functions
in traits, since the lack thereof is the reason they do not yet have this.
-->

> Ghi chú: Định nghĩa thực tế chúng ta đã sử dụng sớm trong chương trông hơi khác so với cái này, bởi vì nó hỗ trợ các phiên bản của Rust mà không có yet hỗ trợ sử dụng async functions trong traits. Như một kết quả, nó trông như thế này:
>
> ```rust,ignore
> fn next(&mut self) -> Next<'_, Self> where Self: Unpin;
> ```
>
> Mà `Next` type là một `struct` mà triển khai `Future` và cho phép chúng ta name lifetime của reference tới `self` với `Next<'_, Self>`, vì vậy `await` có thể làm việc với phương thức này.

Trait `StreamExt` cũng là home của tất cả các interesting methods có sẵn để sử dụng với streams. `StreamExt` là tự động triển khai cho mỗi loại mà triển khai `Stream`, nhưng những traits đó được xác định riêng biệt để enable community để iterate trên convenience APIs mà không ảnh hưởng đến foundational trait.

Trong phiên bản của `StreamExt` được sử dụng trong crate `trpl`, trait không chỉ xác định phương thức `next` nhưng cũng cung cấp một default triển khai của `next` mà correctly xử lý các chi tiết của gọi `Stream::poll_next`. Điều này có nghĩa rằng thậm chí khi bạn cần viết streaming data type của riêng bạn, bạn _only_ có cần triển khai `Stream`, và sau đó bất kỳ ai sử dụng data type của bạn có thể sử dụng `StreamExt` và các phương thức của nó với nó tự động.

Đó là tất cả chúng ta sẽ bao quanh cho lower-level details trên những traits. Để bọc lên, hãy suy nghĩ về cách futures (bao gồm streams), tasks, và threads tất cả vừa với nhau!

[message-passing]: ch17-02-concurrency-with-async.md#sending-data-between-two-tasks-using-message-passing
[ch-18]: ch18-00-oop.html
[async-book]: https://rust-lang.github.io/async-book/
[under-the-hood]: https://rust-lang.github.io/async-book/02_execution/01_chapter.html
[pinning]: https://rust-lang.github.io/async-book/04_pinning/01_chapter.html
[first-async]: ch17-01-futures-and-syntax.html#our-first-async-program
[any-number-futures]: ch17-03-more-futures.html#working-with-any-number-of-futures
[streams]: ch17-04-streams.html
