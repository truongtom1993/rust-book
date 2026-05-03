## Triển Khai một Mẫu Thiết Kế Hướng Đối Tượng

_State pattern_ là một mẫu thiết kế hướng đối tượng. Điểm chính của mẫu là chúng ta định nghĩa một tập hợp các state mà một giá trị có thể có ở bên trong. Các state được biểu diễn bằng một tập hợp _state objects_, và hành vi của giá trị thay đổi dựa trên state của nó. Chúng ta sẽ làm việc thông qua một ví dụ về một struct blog post có một field để giữ state của nó, sẽ là một state object từ tập hợp "draft," "review," hoặc "published."

Các state objects chia sẻ chức năng: Trong Rust, tất nhiên, chúng ta sử dụng structs và traits chứ không phải objects và kế thừa. Mỗi state object chịu trách nhiệm về hành vi của riêng nó và kiểm soát khi nó nên thay đổi thành một state khác. Giá trị giữ một state object không biết gì về hành vi khác nhau của các state hoặc khi nào để chuyển đổi giữa các state.

Ưu điểm của việc sử dụng state pattern là, khi yêu cầu kinh doanh của chương trình thay đổi, chúng ta sẽ không cần phải thay đổi code của giá trị giữ state hoặc code sử dụng giá trị. Chúng ta sẽ chỉ cần cập nhật code bên trong một trong các state objects để thay đổi quy tắc của nó hoặc có thể thêm các state objects khác.

Đầu tiên, chúng ta sẽ triển khai state pattern theo một cách hướng đối tượng truyền thống hơn. Sau đó, chúng ta sẽ sử dụng một cách tiếp cận hơi tự nhiên hơn trong Rust. Chúng ta hãy đi sâu vào việc từng bước triển khai một quy trình làm việc blog post bằng cách sử dụng state pattern.

Chức năng cuối cùng sẽ trông như thế này:

1. Một blog post bắt đầu như một bản nháp trống.
1. Khi bản nháp hoàn tất, yêu cầu xem xét bài post.
1. Khi bài post được phê duyệt, nó sẽ được xuất bản.
1. Chỉ các bài post đã xuất bản trả về nội dung để in để các bài post chưa được phê duyệt không thể được xuất bản một cách tình cờ.

Bất kỳ thay đổi nào khác được thử trên một bài post nên không có tác dụng. Ví dụ, nếu chúng ta cố gắng phê duyệt một bài post draft trước khi chúng ta yêu cầu xem xét, bài post sẽ vẫn còn là một draft chưa được xuất bản.

<!-- Old headings. Do not remove or links may break. -->

<a id="a-traditional-object-oriented-attempt"></a>

### Cố Gắng Theo Kiểu Hướng Đối Tượng Truyền Thống

Có vô số cách để cấu trúc code để giải quyết cùng một vấn đề, mỗi cách có các trade-off khác nhau. Cách triển khai của phần này mang tính hướng đối tượng truyền thống hơn, có thể viết được trong Rust, nhưng không tận dụng một số sức mạnh của Rust. Sau này, chúng ta sẽ chứng minh một giải pháp khác nhau vẫn sử dụng mẫu thiết kế hướng đối tượng nhưng được cấu trúc theo cách có thể trông kém quen thuộc hơn với các lập trình viên có kinh nghiệm hướng đối tượng. Chúng ta sẽ so sánh hai giải pháp để trải nghiệm các trade-off của việc thiết kế code Rust khác với code trong các ngôn ngữ khác.

Listing 18-11 hiển thị quy trình làm việc này dưới dạng code: Đây là một ví dụ sử dụng API mà chúng ta sẽ triển khai trong một library crate có tên `blog`. Điều này sẽ không biên dịch được nhưng vì chúng ta chưa triển khai crate `blog`.

<Listing number="18-11" file-name="src/main.rs" caption="Code chứng minh hành vi mong muốn mà chúng ta muốn crate `blog` của chúng ta có">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:all}}
```

</Listing>

Chúng ta muốn cho phép người dùng tạo một bài post draft mới với `Post::new`. Chúng ta muốn cho phép thêm văn bản vào bài post. Nếu chúng ta cố gắng lấy nội dung của bài post ngay lập tức, trước khi phê duyệt, chúng ta không nên nhận được bất kỳ văn bản nào vì bài post vẫn còn là một draft. Chúng ta đã thêm `assert_eq!` trong code cho mục đích chứng minh. Một bài kiểm tra đơn vị tuyệt vời cho điều này sẽ là khẳng định rằng một bài post draft trả về một chuỗi trống từ phương thức `content`, nhưng chúng ta sẽ không viết các bài kiểm tra cho ví dụ này.

Tiếp theo, chúng ta muốn cho phép yêu cầu xem xét bài post, và chúng ta muốn `content` trả về một chuỗi trống trong khi chờ xem xét. Khi bài post nhận được phê duyệt, nó sẽ được xuất bản, có nghĩa là văn bản của bài post sẽ được trả về khi `content` được gọi.

Lưu ý rằng type duy nhất mà chúng ta tương tác với từ crate là type `Post`. Type này sẽ sử dụng state pattern và sẽ giữ một giá trị sẽ là một trong ba state objects biểu diễn các state khác nhau mà một bài post có thể ở trong—draft, review, hoặc published. Thay đổi từ một state sang state khác sẽ được quản lý ở bên trong type `Post`. Các state thay đổi đáp ứng với các phương thức được gọi bởi người dùng thư viện của chúng ta trên instance `Post`, nhưng họ không phải quản lý trực tiếp các thay đổi state. Ngoài ra, người dùng không thể mắc lỗi với các state, chẳng hạn như xuất bản một bài post trước khi nó được xem xét.

<!-- Old headings. Do not remove or links may break. -->

<a id="defining-post-and-creating-a-new-instance-in-the-draft-state"></a>

#### Định Nghĩa `Post` và Tạo một Instance Mới

Hãy bắt đầu với việc triển khai thư viện! Chúng ta biết rằng chúng ta cần một struct `Post` công khai giữ một số nội dung, vì vậy chúng ta sẽ bắt đầu với định nghĩa của struct và một hàm `new` liên kết công khai để tạo một instance của `Post`, như được hiển thị trong Listing 18-12. Chúng ta cũng sẽ tạo một trait `State` riêng tư sẽ định nghĩa hành vi mà tất cả các state objects cho một `Post` phải có.

Sau đó, `Post` sẽ giữ một trait object của `Box<dyn State>` bên trong một `Option<T>` trong một field riêng tư có tên `state` để giữ state object. Bạn sẽ thấy tại sao `Option<T>` cần thiết trong một chút.

<Listing number="18-12" file-name="src/lib.rs" caption="Định nghĩa của struct `Post` và hàm `new` tạo một instance `Post` mới, trait `State`, và struct `Draft`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-12/src/lib.rs}}
```

</Listing>

Trait `State` định nghĩa hành vi được chia sẻ bởi các state khác nhau của post. Các state objects là `Draft`, `PendingReview`, và `Published`, và chúng sẽ tất cả triển khai trait `State`. Hiện tại, trait không có bất kỳ phương thức nào, và chúng ta sẽ bắt đầu chỉ bằng cách định nghĩa state `Draft` vì đó là state mà chúng ta muốn một bài post bắt đầu với.

Khi chúng ta tạo một `Post` mới, chúng ta đặt field `state` của nó thành một giá trị `Some` giữ một `Box`. `Box` này trỏ đến một instance mới của struct `Draft`. Điều này đảm bảo rằng bất cứ khi nào chúng ta tạo một instance mới của `Post`, nó sẽ bắt đầu như một draft. Vì field `state` của `Post` là riêng tư, không có cách nào để tạo một `Post` ở bất kỳ state nào khác! Trong hàm `Post::new`, chúng ta đặt field `content` thành một `String` mới, trống.

#### Lưu Trữ Văn Bản của Nội Dung Post

Chúng ta đã thấy trong Listing 18-11 rằng chúng ta muốn có thể gọi một phương thức có tên `add_text` và chuyển nó một `&str` sau đó được thêm làm nội dung văn bản của bài post. Chúng ta triển khai điều này như một phương thức, chứ không là để lộ field `content` dưới dạng `pub`, để sau này chúng ta có thể triển khai một phương thức sẽ kiểm soát cách đọc dữ liệu field `content`. Phương thức `add_text` khá đơn giản, vì vậy hãy thêm cách triển khai trong Listing 18-13 vào khối `impl Post`.

<Listing number="18-13" file-name="src/lib.rs" caption="Triển khai phương thức `add_text` để thêm văn bản vào `content` của một post">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-13/src/lib.rs:here}}
```

</Listing>

Phương thức `add_text` lấy một mutable reference của `self` vì chúng ta đang thay đổi instance `Post` mà chúng ta đang gọi `add_text` trên. Sau đó chúng ta gọi `push_str` trên `String` trong `content` và chuyển đối số `text` để thêm vào `content` đã lưu. Hành vi này không phụ thuộc vào state mà bài post ở, vì vậy nó không phải là một phần của state pattern. Phương thức `add_text` không tương tác với field `state` cá nhân, nhưng nó là một phần của hành vi mà chúng ta muốn hỗ trợ.

<!-- Old headings. Do not remove or links may break. -->

<a id="ensuring-the-content-of-a-draft-post-is-empty"></a>

#### Đảm Bảo Rằng Nội Dung của một Post Draft Trống

Ngay cả sau khi chúng ta đã gọi `add_text` và thêm một số nội dung vào bài post của chúng ta, chúng ta vẫn muốn phương thức `content` trả về một slick chuỗi trống vì bài post vẫn ở trạng thái draft, như được hiển thị bởi `assert_eq!` đầu tiên trong Listing 18-11. Hiện tại, hãy triển khai phương thức `content` với điều đơn giản nhất sẽ đáp ứng yêu cầu này: luôn trả về một slick chuỗi trống. Chúng ta sẽ thay đổi điều này sau khi chúng ta triển khai khả năng thay đổi state của một bài post để nó có thể được xuất bản. Cho đến nay, các bài post chỉ có thể ở trạng thái draft, vì vậy nội dung bài post luôn phải trống. Listing 18-14 hiển thị cách triển khai giữ chỗ này.

<Listing number="18-14" file-name="src/lib.rs" caption="Thêm một cách triển khai giữ chỗ cho phương thức `content` trên `Post` luôn trả về một slick chuỗi trống">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-14/src/lib.rs:here}}
```

</Listing>

Với phương thức `content` được thêm này, mọi thứ trong Listing 18-11 thông qua `assert_eq!` đầu tiên hoạt động như mong muốn.

<!-- Old headings. Do not remove or links may break. -->

<a id="requesting-a-review-of-the-post-changes-its-state"></a>
<a id="requesting-a-review-changes-the-posts-state"></a>

#### Yêu Cầu Xem Xét, Điều Này Thay Đổi State của Post

Tiếp theo, chúng ta cần thêm chức năng để yêu cầu xem xét một bài post, điều này nên thay đổi state của nó từ `Draft` sang `PendingReview`. Listing 18-15 hiển thị code này.

<Listing number="18-15" file-name="src/lib.rs" caption="Triển khai phương thức `request_review` trên `Post` và trait `State`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-15/src/lib.rs:here}}
```

</Listing>

Chúng ta cung cấp `Post` một phương thức công khai có tên `request_review` sẽ lấy một mutable reference của `self`. Sau đó, chúng ta gọi một phương thức `request_review` nội bộ trên state hiện tại của `Post`, và phương thức `request_review` thứ hai này tiêu thụ state hiện tại và trả về một state mới.

Chúng ta thêm phương thức `request_review` vào trait `State`; tất cả các type triển khai trait sẽ bây giờ cần phải triển khai phương thức `request_review`. Lưu ý rằng thay vì có `self`, `&self`, hoặc `&mut self` làm tham số đầu tiên của phương thức, chúng ta có `self: Box<Self>`. Cú pháp này có nghĩa là phương thức chỉ hợp lệ khi được gọi trên một `Box` giữ loại. Cú pháp này lấy quyền sở hữu `Box<Self>`, làm cho state cũ không hợp lệ để giá trị state của `Post` có thể chuyển đổi thành một state mới.

Để tiêu thụ state cũ, phương thức `request_review` cần phải lấy quyền sở hữu của giá trị state. Đây là nơi `Option` trong field `state` của `Post` xuất hiện: Chúng ta gọi phương thức `take` để lấy giá trị `Some` từ field `state` và để lại một `None` tại chỗ của nó vì Rust không cho phép chúng ta có các field không được điền trong structs. Điều này cho phép chúng ta di chuyển giá trị `state` từ `Post` chứ không phải mượn nó. Sau đó, chúng ta sẽ đặt giá trị `state` của bài post thành kết quả của thao tác này.

Chúng ta cần đặt `state` thành `None` tạm thời chứ không phải đặt nó trực tiếp với code như `self.state = self.state.request_review();` để lấy quyền sở hữu của giá trị `state`. Điều này đảm bảo rằng `Post` không thể sử dụng giá trị `state` cũ sau khi chúng ta đã chuyển đổi nó thành một state mới.

Phương thức `request_review` trên `Draft` trả về một instance mới, boxed của một struct `PendingReview` mới, đại diện cho state khi một bài post đang chờ xem xét. Struct `PendingReview` cũng triển khai phương thức `request_review` nhưng không thực hiện bất kỳ chuyển đổi nào. Thay vào đó, nó trả về chính nó vì khi chúng ta yêu cầu xem xét trên một bài post đã ở trạng thái `PendingReview`, nó nên ở lại trong state `PendingReview`.

Bây giờ chúng ta có thể bắt đầu thấy những ưu điểm của state pattern: Phương thức `request_review` trên `Post` là như nhau bất kể giá trị `state` của nó. Mỗi state chịu trách nhiệm về các quy tắc của riêng nó.

Chúng ta sẽ để phương thức `content` trên `Post` như là, trả về một slick chuỗi trống. Chúng ta bây giờ có thể có một `Post` ở trạng thái `PendingReview` cũng như ở trạng thái `Draft`, nhưng chúng ta muốn hành vi tương tự ở trạng thái `PendingReview`. Listing 18-11 bây giờ hoạt động đến lệnh gọi `assert_eq!` thứ hai!

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-the-approve-method-that-changes-the-behavior-of-content"></a>
<a id="adding-approve-to-change-the-behavior-of-content"></a>

#### Thêm `approve` để Thay Đổi Hành Động của `content`

Phương thức `approve` sẽ tương tự như phương thức `request_review`: Nó sẽ đặt `state` thành giá trị mà state hiện tại cho biết nó nên có khi state đó được phê duyệt, như được hiển thị trong Listing 18-16.

<Listing number="18-16" file-name="src/lib.rs" caption="Triển khai phương thức `approve` trên `Post` và trait `State`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-16/src/lib.rs:here}}
```

</Listing>

Chúng ta thêm phương thức `approve` vào trait `State` và thêm một struct mới triển khai `State`, state `Published`.

Tương tự như cách `request_review` trên `PendingReview` hoạt động, nếu chúng ta gọi phương thức `approve` trên `Draft`, nó sẽ không có tác dụng vì `approve` sẽ trả về `self`. Khi chúng ta gọi `approve` trên `PendingReview`, nó trả về một instance mới, boxed của struct `Published`. Struct `Published` triển khai trait `State`, và đối với cả phương thức `request_review` lẫn phương thức `approve`, nó trả về chính nó vì bài post nên ở lại trong state `Published` trong những trường hợp đó.

Bây giờ chúng ta cần cập nhật phương thức `content` trên `Post`. Chúng ta muốn giá trị được trả về từ `content` phụ thuộc vào state hiện tại của `Post`, vì vậy chúng ta sẽ có `Post` phân công cho một phương thức `content` được định nghĩa trên `state` của nó, như được hiển thị trong Listing 18-17.

<Listing number="18-17" file-name="src/lib.rs" caption="Cập nhật phương thức `content` trên `Post` để phân công cho một phương thức `content` trên `State`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-17/src/lib.rs:here}}
```

</Listing>

Vì mục tiêu là giữ tất cả các quy tắc này bên trong các structs triển khai `State`, chúng ta gọi một phương thức `content` trên giá trị trong `state` và chuyển instance bài post (đó là `self`) như một đối số. Sau đó, chúng ta trả về giá trị được trả về từ việc sử dụng phương thức `content` trên giá trị `state`.

Chúng ta gọi phương thức `as_ref` trên `Option` vì chúng ta muốn một reference đến giá trị bên trong `Option` chứ không phải quyền sở hữu của giá trị. Vì `state` là `Option<Box<dyn State>>`, khi chúng ta gọi `as_ref`, `Option<&Box<dyn State>>` được trả về. Nếu chúng ta không gọi `as_ref`, chúng ta sẽ nhận được một lỗi vì chúng ta không thể di chuyển `state` ra khỏi `&self` được mượn của tham số hàm.

Sau đó chúng ta gọi phương thức `unwrap`, mà chúng ta biết sẽ không bao giờ panic vì chúng ta biết rằng các phương thức trên `Post` đảm bảo rằng `state` sẽ luôn chứa một giá trị `Some` khi những phương thức đó xong. Đây là một trong những trường hợp chúng ta đã nói đến trong phần ["When You Have More Information Than the Compiler"][more-info-than-rustc]<!-- ignore --> của Chương 9 khi chúng ta biết rằng một giá trị `None` không bao giờ có thể xảy ra, ngay cả khi compiler không thể hiểu điều đó.

Tại thời điểm này, khi chúng ta gọi `content` trên `&Box<dyn State>`, deref coercion sẽ có hiệu lực trên `&` và `Box` để phương thức `content` sẽ cuối cùng được gọi trên type triển khai trait `State`. Điều này có nghĩa là chúng ta cần phải thêm `content` vào định nghĩa trait `State`, và đó là nơi chúng ta sẽ đặt logic để xác định nội dung nào để trả về tùy thuộc vào state nào chúng ta có, như được hiển thị trong Listing 18-18.

<Listing number="18-18" file-name="src/lib.rs" caption="Thêm phương thức `content` vào trait `State`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-18/src/lib.rs:here}}
```

</Listing>

Chúng ta thêm một cách triển khai mặc định cho phương thức `content` trả về một slick chuỗi trống. Điều này có nghĩa là chúng ta không cần phải triển khai `content` trên các struct `Draft` và `PendingReview`. Struct `Published` sẽ ghi đè phương thức `content` và trả về giá trị trong `post.content`. Mặc dù thuận tiện, việc có phương thức `content` trên `State` xác định nội dung của `Post` đang làm mờ đường ranh giữa trách nhiệm của `State` và trách nhiệm của `Post`.

Lưu ý rằng chúng ta cần chú thích vòng đời trên phương thức này, như chúng ta đã thảo luận trong Chương 10. Chúng ta đang lấy một reference đến một `post` như một đối số và trả về một reference đến một phần của `post` đó, vì vậy vòng đời của reference được trả về có liên quan đến vòng đời của đối số `post`.

Và chúng ta hoàn tất—tất cả Listing 18-11 bây giờ hoạt động! Chúng ta đã triển khai state pattern với các quy tắc của quy trình làm việc blog post. Logic liên quan đến các quy tắc sống trong các state objects chứ không bị phân tán xuyên suốt `Post`.

> ### Tại Sao Không Sử Dụng Enum?
>
> Bạn có thể đã thắc mắc tại sao chúng ta không sử dụng một enum với các state có thể khác nhau của bài post làm các biến thể. Đó chắc chắn là một giải pháp có thể; hãy thử và so sánh kết quả cuối cùng để xem cái nào bạn thích! Một nhược điểm của việc sử dụng enum là mọi nơi kiểm tra giá trị của enum sẽ cần một biểu thức `match` hoặc tương tự để xử lý mọi biến thể có thể. Điều này có thể lặp lại hơn giải pháp trait object này.

<!-- Old headings. Do not remove or links may break. -->

<a id="trade-offs-of-the-state-pattern"></a>

#### Đánh Giá State Pattern

Chúng ta đã chỉ ra rằng Rust có khả năng triển khai state pattern hướng đối tượng để đóng gói các loại hành vi khác nhau mà một bài post nên có ở mỗi state. Các phương thức trên `Post` không biết gì về các hành vi khác nhau. Vì cách chúng ta tổ chức code, chúng ta phải xem ở một nơi duy nhất để biết các cách khác nhau mà một bài post đã xuất bản có thể hành động: cách triển khai trait `State` trên struct `Published`.

Nếu chúng ta sẽ tạo một cách triển khai thay thế không sử dụng state pattern, chúng ta có thể thay vào đó sử dụng các biểu thức `match` trong các phương thức trên `Post` hoặc thậm chí trong code `main` kiểm tra state của bài post và thay đổi hành vi ở những nơi đó. Điều này có nghĩa là chúng ta sẽ phải xem ở một số nơi để hiểu tất cả các hàm ý của một bài post ở trạng thái published.

Với state pattern, các phương thức `Post` và những nơi chúng ta sử dụng `Post` không cần biểu thức `match`, và để thêm một state mới, chúng ta sẽ chỉ cần thêm một struct mới và triển khai các phương thức trait trên struct đó trong một vị trí.

Cách triển khai sử dụng state pattern rất dễ mở rộng để thêm chức năng. Để xem sự đơn giản của việc bảo trì code sử dụng state pattern, hãy thử một vài trong số các đề xuất này:

- Thêm một phương thức `reject` thay đổi state của bài post từ `PendingReview` quay trở lại `Draft`.
- Yêu cầu hai lệnh gọi `approve` trước khi state có thể được thay đổi thành `Published`.
- Cho phép người dùng thêm nội dung văn bản chỉ khi một bài post ở trạng thái `Draft`. Gợi ý: có state object chịu trách nhiệm về những gì có thể thay đổi về nội dung nhưng không chịu trách nhiệm về việc sửa đổi `Post`.

Một nhược điểm của state pattern là, vì các state triển khai các chuyển đổi giữa các state, một số state được kết hợp với nhau. Nếu chúng ta thêm một state khác giữa `PendingReview` và `Published`, chẳng hạn như `Scheduled`, chúng ta sẽ phải thay đổi code trong `PendingReview` để chuyển đổi sang `Scheduled` thay thế. Nó sẽ ít công sức hơn nếu `PendingReview` không cần phải thay đổi với sự bổ sung của một state mới, nhưng điều đó sẽ có nghĩa là chuyển sang một mẫu thiết kế khác.

Một nhược điểm khác là chúng ta đã sao chép một số logic. Để loại bỏ một số sao chép, chúng ta có thể cố gắng tạo các cách triển khai mặc định cho các phương thức `request_review` và `approve` trên trait `State` trả về `self`. Tuy nhiên, điều này sẽ không hoạt động: Khi sử dụng `State` làm một trait object, trait không biết chính xác `self` concrete sẽ là, vì vậy kiểu trả về không được biết tại compile time. (Đây là một trong những quy tắc dyn compatibility được đề cập trước đó.)

Sao chép khác bao gồm các cách triển khai tương tự của các phương thức `request_review` và `approve` trên `Post`. Cả hai phương thức sử dụng `Option::take` với field `state` của `Post`, và nếu `state` là `Some`, chúng phân công cho cách triển khai của phương thức tương tự của giá trị được bao bọc và đặt giá trị mới của field `state` thành kết quả. Nếu chúng ta có nhiều phương thức trên `Post` theo sau mẫu này, chúng ta có thể xem xét xác định một macro để loại bỏ sự lặp lại (xem phần ["Macros"][macros]<!-- ignore --> trong Chương 20).

Bằng cách triển khai state pattern chính xác như nó được định nghĩa cho các ngôn ngữ hướng đối tượng, chúng ta không tận dụng đầy đủ sức mạnh của Rust như chúng ta có thể. Hãy xem một số thay đổi chúng ta có thể thực hiện đối với crate `blog` có thể làm cho các state không hợp lệ và chuyển đổi thành các lỗi compile-time.

### Mã Hóa States và Hành Động dưới dạng Types

Chúng ta sẽ chỉ cho bạn cách suy nghĩ lại state pattern để nhận được một tập hợp trade-off khác nhau. Thay vì đóng gói các state và chuyển đổi hoàn toàn để code bên ngoài không có kiến thức về chúng, chúng ta sẽ mã hóa các state vào các type khác nhau. Theo đó, hệ thống type-checking của Rust sẽ ngăn chặn các nỗ lực sử dụng draft posts nơi chỉ các bài post được xuất bản được phép bằng cách phát hành một lỗi compiler.

Hãy xem xét phần đầu tiên của `main` trong Listing 18-11:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:here}}
```

</Listing>

Chúng ta vẫn cho phép tạo các bài post mới ở trạng thái draft bằng cách sử dụng `Post::new` và khả năng thêm văn bản vào nội dung của bài post. Nhưng thay vì có một phương thức `content` trên một bài post draft trả về một chuỗi trống, chúng ta sẽ làm cho nó sao cho các bài post draft không có phương thức `content` cả. Bằng cách đó, nếu chúng ta cố gắng lấy nội dung của một bài post draft, chúng ta sẽ nhận được một lỗi compiler cho chúng ta biết phương thức không tồn tại. Kết quả là, nó sẽ không thể cho chúng ta vô tình hiển thị nội dung bài post draft trong sản xuất vì code đó thậm chí sẽ không biên dịch. Listing 18-19 hiển thị định nghĩa của struct `Post` và struct `DraftPost`, cũng như các phương thức trên mỗi cái.

<Listing number="18-19" file-name="src/lib.rs" caption="Một `Post` với phương thức `content` và một `DraftPost` không có phương thức `content`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-19/src/lib.rs}}
```

</Listing>

Cả struct `Post` lẫn `DraftPost` đều có một field `content` riêng tư lưu trữ văn bản bài post. Các struct không còn có field `state` nữa vì chúng ta đang di chuyển mã hóa của state sang các type của các struct. Struct `Post` sẽ đại diện cho một bài post được xuất bản, và nó có một phương thức `content` trả về `content`.

Chúng ta vẫn có một hàm `Post::new`, nhưng thay vì trả về một instance của `Post`, nó trả về một instance của `DraftPost`. Vì `content` là riêng tư và không có bất kỳ hàm nào trả về `Post`, nó hiện không thể tạo một instance của `Post`.

Struct `DraftPost` có một phương thức `add_text`, vì vậy chúng ta có thể thêm văn bản vào `content` như trước, nhưng lưu ý rằng `DraftPost` không có phương thức `content` được định nghĩa! Vì vậy bây giờ chương trình đảm bảo rằng tất cả các bài post bắt đầu dưới dạng các bài post draft, và các bài post draft không có nội dung của chúng có sẵn để hiển thị. Bất kỳ nỗ lực nào để vượt quá những ràng buộc này sẽ dẫn đến một lỗi compiler.

<!-- Old headings. Do not remove or links may break. -->

Vậy, làm thế nào chúng ta có thể nhận được một bài post được xuất bản? Chúng ta muốn thực thi quy tắc rằng một bài post draft phải được xem xét và phê duyệt trước khi nó có thể được xuất bản. Một bài post ở trạng thái xem xét pending vẫn không nên hiển thị bất kỳ nội dung nào. Hãy triển khai những ràng buộc này bằng cách thêm một struct khác, `PendingReviewPost`, định nghĩa phương thức `request_review` trên `DraftPost` để trả về `PendingReviewPost` và định nghĩa phương thức `approve` trên `PendingReviewPost` để trả về `Post`, như được hiển thị trong Listing 18-20.

<Listing number="18-20" file-name="src/lib.rs" caption="Một `PendingReviewPost` được tạo bằng cách gọi `request_review` trên `DraftPost` và phương thức `approve` biến `PendingReviewPost` thành một `Post` được xuất bản">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-20/src/lib.rs:here}}
```

</Listing>

Các phương thức `request_review` và `approve` lấy quyền sở hữu của `self`, do đó tiêu thụ các instance `DraftPost` và `PendingReviewPost` và chuyển đổi chúng thành `PendingReviewPost` và `Post` được xuất bản tương ứng. Bằng cách này, chúng ta sẽ không có bất kỳ instance `DraftPost` nào vẫn còn sau khi chúng ta gọi `request_review` trên chúng, và như vậy. Struct `PendingReviewPost` không có phương thức `content` được định nghĩa trên nó, vì vậy cố gắng đọc nội dung của nó dẫn đến một lỗi compiler, như với `DraftPost`. Vì cách duy nhất để lấy một instance `Post` được xuất bản có phương thức `content` được định nghĩa là gọi phương thức `approve` trên `PendingReviewPost`, và cách duy nhất để lấy `PendingReviewPost` là gọi phương thức `request_review` trên `DraftPost`, chúng ta bây giờ đã mã hóa quy trình làm việc bài post vào hệ thống type.

Nhưng chúng ta cũng phải thực hiện một số thay đổi nhỏ đối với `main`. Các phương thức `request_review` và `approve` trả về các instance mới thay vì sửa đổi struct chúng được gọi trên, vì vậy chúng ta cần thêm nhiều hơn các gán shadowing `let post =` để lưu các instance được trả về. Chúng ta cũng không thể có các khẳng định về nội dung của các bài post draft và review pending là các chuỗi trống, cũng không cần chúng: Chúng ta không thể biên dịch code cố gắng sử dụng nội dung của các bài post ở những state đó nữa. Code cập nhật trong `main` được hiển thị trong Listing 18-21.

<Listing number="18-21" file-name="src/main.rs" caption="Các sửa đổi `main` để sử dụng cách triển khai mới của quy trình làm việc bài post">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-21/src/main.rs}}
```

</Listing>

Các thay đổi chúng ta cần phải thực hiện đối với `main` để gán lại `post` có nghĩa là cách triển khai này không hoàn toàn tuân theo state pattern hướng đối tượng nữa: Các chuyển đổi giữa các state không còn được đóng gói hoàn toàn trong cách triển khai `Post`. Tuy nhiên, lợi ích của chúng ta là các state không hợp lệ bây giờ không thể xảy ra vì của hệ thống type và type checking xảy ra tại compile time! Điều này đảm bảo rằng các lỗi nhất định, chẳng hạn như hiển thị nội dung của một bài post chưa được xuất bản, sẽ được phát hiện trước khi chúng đi vào sản xuất.

Hãy thử các công việc được đề xuất ở đầu phần này trên crate `blog` khi nó ở sau Listing 18-21 để xem bạn nghĩ gì về thiết kế của phiên bản code này. Lưu ý rằng một số công việc có thể đã được hoàn thành trong thiết kế này.

Chúng ta đã thấy rằng ngay cả khi Rust có khả năng triển khai các mẫu thiết kế hướng đối tượng, các mẫu khác, chẳng hạn như mã hóa state vào hệ thống type, cũng có sẵn trong Rust. Những mẫu này có các trade-off khác nhau. Mặc dù bạn có thể rất quen thuộc với các mẫu hướng đối tượng, suy nghĩ lại về vấn đề để tận dụng các tính năng của Rust có thể cung cấp các lợi ích, chẳng hạn như ngăn chặn một số lỗi tại compile time. Các mẫu hướng đối tượng sẽ không luôn là giải pháp tốt nhất trong Rust do các tính năng nhất định, như ownership, mà các ngôn ngữ hướng đối tượng không có.

## Tóm Tắt

Bất kể bạn có nghĩ rằng Rust là một ngôn ngữ hướng đối tượng sau khi đọc chương này, bây giờ bạn biết rằng bạn có thể sử dụng trait objects để có được một số tính năng hướng đối tượng trong Rust. Dynamic dispatch có thể cung cấp cho code của bạn một số tính linh hoạt để đổi lấy một chút hiệu suất runtime. Bạn có thể sử dụng tính linh hoạt này để triển khai các mẫu hướng đối tượng có thể giúp code của bạn về khả năng bảo trì. Rust cũng có các tính năng khác, như ownership, mà các ngôn ngữ hướng đối tượng không có. Một mẫu hướng đối tượng sẽ không luôn là cách tốt nhất để tận dụng sức mạnh của Rust, nhưng nó là một tùy chọn có sẵn.

Tiếp theo, chúng ta sẽ xem xét các mẫu, đây là một trong những tính năng khác của Rust cho phép rất nhiều tính linh hoạt. Chúng ta đã xem xét chúng ngắn gọn xuyên suốt cuốn sách nhưng chưa thấy đầy đủ khả năng của chúng. Chúng ta hãy cùng nhau!

[more-info-than-rustc]: ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler
[macros]: ch20-05-macros.html#macros
