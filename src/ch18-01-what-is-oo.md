## Các Đặc Điểm của Ngôn Ngữ Hướng Đối Tượng

Không có sự đồng thuận trong cộng đồng lập trình về những tính năng mà một ngôn ngữ phải có để được coi là hướng đối tượng. Rust bị ảnh hưởng bởi nhiều mô hình lập trình, bao gồm OOP; ví dụ, chúng ta đã khám phá những tính năng đến từ lập trình hàm trong Chương 13. Có thể nói, các ngôn ngữ OOP chia sẻ những đặc điểm chung nhất định—cụ thể là, các đối tượng, đóng gói, và kế thừa. Chúng ta hãy xem xét những đặc điểm này có nghĩa là gì và liệu Rust có hỗ trợ nó hay không.

### Các Đối Tượng Chứa Dữ Liệu và Hành Động

Cuốn sách _Design Patterns: Elements of Reusable Object-Oriented Software_ của Erich Gamma, Richard Helm, Ralph Johnson, và John Vlissides (Addison-Wesley, 1994), thường được gọi là cuốn sách _Gang of Four_, là một danh sách các mẫu thiết kế hướng đối tượng. Nó định nghĩa OOP theo cách này:

> Các chương trình hướng đối tượng được tạo thành từ các đối tượng. Một **đối tượng** gói gọn cả dữ liệu và các thủ tục hoạt động trên dữ liệu đó. Các thủ tục này thường được gọi là **phương thức** hoặc **thao tác**.

Sử dụng định nghĩa này, Rust là hướng đối tượng: Các struct và enum có dữ liệu, và các khối `impl` cung cấp các phương thức cho struct và enum. Mặc dù các struct và enum với các phương thức không được _gọi_ là đối tượng, chúng cung cấp cùng một chức năng, theo định nghĩa của Gang of Four về các đối tượng.

### Đóng Gói Ẩn Chi Tiết Triển Khai

Một khía cạnh khác thường được liên kết với OOP là ý tưởng của _đóng gói_, có nghĩa là chi tiết triển khai của một đối tượng không có thể truy cập được từ code sử dụng đối tượng đó. Do đó, cách duy nhất để tương tác với một đối tượng là thông qua API công khai của nó; code sử dụng đối tượng không nên có thể tiếp cận phần bên trong của đối tượng và thay đổi dữ liệu hoặc hành động trực tiếp. Điều này cho phép người lập trình thay đổi và tái cấu trúc phần bên trong của một đối tượng mà không cần phải thay đổi code sử dụng đối tượng.

Chúng ta đã thảo luận về cách kiểm soát đóng gói trong Chương 7: Chúng ta có thể sử dụng từ khóa `pub` để quyết định những module, type, function, và phương thức nào trong code của chúng ta nên công khai, và theo mặc định mọi thứ khác là riêng tư. Ví dụ, chúng ta có thể định nghĩa một struct `AveragedCollection` có một field chứa một vector các giá trị `i32`. Struct cũng có thể có một field chứa giá trị trung bình của các giá trị trong vector, có nghĩa là giá trị trung bình không phải được tính toán trên nhu cầu bất cứ khi nào ai đó cần nó. Nói cách khác, `AveragedCollection` sẽ lưu giữ giá trị trung bình được tính toán cho chúng ta. Listing 18-1 có định nghĩa của struct `AveragedCollection`.

<Listing number="18-1" file-name="src/lib.rs" caption="Một struct `AveragedCollection` duy trì một danh sách các số nguyên và giá trị trung bình của các mục trong bộ sưu tập">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-01/src/lib.rs}}
```

</Listing>

Struct được đánh dấu `pub` để những code khác có thể sử dụng nó, nhưng các field trong struct vẫn còn riêng tư. Điều này rất quan trọng trong trường hợp này vì chúng ta muốn đảm bảo rằng bất cứ khi nào một giá trị được thêm vào hoặc loại bỏ khỏi danh sách, giá trị trung bình cũng được cập nhật. Chúng ta làm điều này bằng cách triển khai các phương thức `add`, `remove`, và `average` trên struct, như được hiển thị trong Listing 18-2.

<Listing number="18-2" file-name="src/lib.rs" caption="Cách triển khai các phương thức công khai `add`, `remove`, và `average` trên `AveragedCollection`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-02/src/lib.rs:here}}
```

</Listing>

Các phương thức công khai `add`, `remove`, và `average` là những cách duy nhất để truy cập hoặc sửa đổi dữ liệu trong một instance của `AveragedCollection`. Khi một mục được thêm vào `list` bằng phương thức `add` hoặc loại bỏ bằng phương thức `remove`, các cách triển khai của mỗi cái gọi phương thức riêng `update_average` xử lý cập nhật field `average` cũng như vậy.

Chúng ta để các field `list` và `average` riêng tư để không có cách nào cho code bên ngoài thêm hoặc xóa các mục vào hoặc từ field `list` trực tiếp; nếu không, field `average` có thể không đồng bộ khi `list` thay đổi. Phương thức `average` trả về giá trị trong field `average`, cho phép code bên ngoài đọc `average` nhưng không sửa đổi nó.

Vì chúng ta đã đóng gói chi tiết triển khai của struct `AveragedCollection`, chúng ta có thể dễ dàng thay đổi các khía cạnh, chẳng hạn như cấu trúc dữ liệu, trong tương lai. Ví dụ, chúng ta có thể sử dụng `HashSet<i32>` thay vì `Vec<i32>` cho field `list`. Miễn là các chữ ký của các phương thức công khai `add`, `remove`, và `average` vẫn giữ nguyên, code sử dụng `AveragedCollection` không cần phải thay đổi. Nếu chúng ta đã làm cho `list` công khai thay thế, điều này có thể không nhất thiết phải là trường hợp: `HashSet<i32>` và `Vec<i32>` có các phương thức khác nhau để thêm và xóa các mục, vì vậy code bên ngoài có thể sẽ phải thay đổi nếu nó sửa đổi `list` trực tiếp.

Nếu đóng gói là một khía cạnh bắt buộc để một ngôn ngữ được coi là hướng đối tượng, thì Rust đáp ứng yêu cầu đó. Tùy chọn sử dụng `pub` hay không cho các phần khác nhau của code cho phép đóng gói chi tiết triển khai.

### Kế Thừa như một Hệ Thống Type và chia sẻ Code

_Kế thừa_ là một cơ chế mà theo đó một đối tượng có thể kế thừa các phần tử từ định nghĩa của một đối tượng cha, do đó thu được dữ liệu và hành động của đối tượng cha mà không cần phải định nghĩa chúng lại.

Nếu một ngôn ngữ phải có kế thừa để được coi là hướng đối tượng, thì Rust không phải là một ngôn ngữ như vậy. Không có cách nào để định nghĩa một struct kế thừa các field và cách triển khai phương thức của struct cha mà không sử dụng macro.

Tuy nhiên, nếu bạn đã quen với việc có kế thừa trong bộ công cụ lập trình của mình, bạn có thể sử dụng các giải pháp khác trong Rust, tùy thuộc vào lý do ban đầu bạn sử dụng kế thừa.

Bạn sẽ chọn kế thừa vì hai lý do chính. Một là để tái sử dụng code: Bạn có thể triển khai một hành vi cụ thể cho một type, và kế thừa cho phép bạn tái sử dụng việc triển khai đó cho một type khác. Bạn có thể làm điều này theo cách giới hạn trong code Rust bằng cách sử dụng các cách triển khai phương thức trait mặc định, mà bạn đã thấy trong Listing 10-14 khi chúng ta thêm một cách triển khai mặc định của phương thức `summarize` trên trait `Summary`. Bất kỳ type nào triển khai trait `Summary` sẽ có phương thức `summarize` có sẵn mà không cần bất kỳ code nào nữa. Điều này tương tự như một lớp cha có một cách triển khai của một phương thức và một lớp con kế thừa cũng có cách triển khai của phương thức. Chúng ta cũng có thể ghi đè cách triển khai mặc định của phương thức `summarize` khi chúng ta triển khai trait `Summary`, giống như một lớp con ghi đè cách triển khai của một phương thức kế thừa từ lớp cha.

Lý do khác để sử dụng kế thừa liên quan đến hệ thống type: để cho phép một type con được sử dụng ở những nơi giống như type cha. Điều này cũng được gọi là _đa hình_, có nghĩa là bạn có thể thay thế nhiều đối tượng cho nhau khi chạy nếu chúng chia sẻ những đặc điểm nhất định.

> ### Đa Hình
>
> Đối với nhiều người, đa hình là đồng nghĩa với kế thừa. Nhưng nó thực sự là một khái niệm tổng quát hơn đề cập đến code có thể hoạt động với dữ liệu của nhiều type. Đối với kế thừa, những type đó thường là các subclass.
>
> Rust thay vào đó sử dụng generics để trừu tượng hóa các type có thể khác nhau và trait bounds để áp đặt ràng buộc về những gì những type đó phải cung cấp. Điều này đôi khi được gọi là _đa hình tham số bị giới hạn_.

Rust đã chọn một tập hợp trade-off khác nhau bằng cách không cung cấp kế thừa. Kế thừa thường có nguy cơ chia sẻ nhiều code hơn cần thiết. Các subclass không nên luôn chia sẻ tất cả các đặc điểm của lớp cha của chúng nhưng sẽ làm như vậy với kế thừa. Điều này có thể làm cho thiết kế của một chương trình kém linh hoạt hơn. Nó cũng giới thiệu khả năng gọi các phương thức trên subclass mà không hợp lý hoặc gây ra lỗi vì các phương thức không áp dụng cho subclass. Ngoài ra, một số ngôn ngữ sẽ chỉ cho phép _kế thừa đơn_ (có nghĩa là một subclass chỉ có thể kế thừa từ một lớp), hạn chế thêm tính linh hoạt của thiết kế chương trình.

Vì những lý do này, Rust áp dụng cách tiếp cận khác bằng cách sử dụng trait objects thay vì kế thừa để đạt được đa hình tại runtime. Hãy xem xét cách trait objects hoạt động.
