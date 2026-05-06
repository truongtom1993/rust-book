## Phụ Lục C: Các Trait Có Thể Derive

Ở nhiều nơi trong cuốn sách, chúng ta đã thảo luận về thuộc tính `derive`, mà bạn có thể áp dụng cho định nghĩa struct hoặc enum. Thuộc tính `derive` tạo ra code sẽ implement một trait với implementation mặc định của chính nó trên kiểu mà bạn đã chú thích bằng cú pháp `derive`.

Trong phụ lục này, chúng ta cung cấp tài liệu tham khảo về tất cả các trait trong thư viện chuẩn mà bạn có thể sử dụng với `derive`. Mỗi phần bao gồm:

- Các toán tử và method nào mà việc derive trait này sẽ kích hoạt
- Implementation của trait được cung cấp bởi `derive` làm gì
- Việc implement trait đó nói lên điều gì về kiểu dữ liệu
- Các điều kiện mà bạn được phép hoặc không được phép implement trait
- Các ví dụ về các thao tác yêu cầu trait

Nếu bạn muốn hành vi khác với hành vi được cung cấp bởi thuộc tính `derive`, hãy tham khảo [tài liệu thư viện chuẩn](../std/index.html)<!-- ignore --> cho từng trait để biết chi tiết về cách implement thủ công.

Các trait được liệt kê ở đây là những trait duy nhất được định nghĩa bởi thư viện chuẩn có thể được implement trên các kiểu của bạn bằng `derive`. Các trait khác được định nghĩa trong thư viện chuẩn không có hành vi mặc định hợp lý, vì vậy việc implement chúng theo cách có nghĩa với những gì bạn đang cố gắng thực hiện là tùy thuộc vào bạn.

Một ví dụ về trait không thể derive là `Display`, xử lý định dạng cho người dùng cuối. Bạn nên luôn xem xét cách hiển thị phù hợp một kiểu cho người dùng cuối. Phần nào của kiểu mà người dùng cuối được phép xem? Phần nào họ sẽ thấy liên quan? Định dạng dữ liệu nào sẽ phù hợp nhất với họ? Trình biên dịch Rust không có cái nhìn sâu sắc này, vì vậy nó không thể cung cấp hành vi mặc định phù hợp cho bạn.

Danh sách các trait có thể derive được cung cấp trong phụ lục này không phải là toàn diện: Các thư viện có thể implement `derive` cho các trait của riêng chúng, làm cho danh sách các trait bạn có thể sử dụng `derive` thực sự là mở. Việc implement `derive` liên quan đến việc sử dụng procedural macro, được đề cập trong phần ["Custom `derive` Macros"][custom-derive-macros]<!-- ignore --> trong Chương 20.

### `Debug` cho Đầu Ra Lập Trình Viên

Trait `Debug` kích hoạt định dạng debug trong các chuỗi format, mà bạn chỉ định bằng cách thêm `:?` trong các placeholder `{}`.

Trait `Debug` cho phép bạn in các instance của một kiểu cho mục đích debug, để bạn và các lập trình viên khác sử dụng kiểu của bạn có thể kiểm tra một instance tại một điểm cụ thể trong quá trình thực thi chương trình.

Trait `Debug` là bắt buộc, ví dụ, khi sử dụng macro `assert_eq!`. Macro này in các giá trị của các instance được đưa ra làm đối số nếu khẳng định bằng nhau thất bại để các lập trình viên có thể thấy tại sao hai instance không bằng nhau.

### `PartialEq` và `Eq` cho So Sánh Bằng Nhau

Trait `PartialEq` cho phép bạn so sánh các instance của một kiểu để kiểm tra tính bằng nhau và kích hoạt sử dụng các toán tử `==` và `!=`.

Việc derive `PartialEq` implement method `eq`. Khi `PartialEq` được derive trên struct, hai instance bằng nhau chỉ khi _tất cả_ các trường đều bằng nhau, và các instance không bằng nhau nếu _bất kỳ_ trường nào không bằng nhau. Khi được derive trên enum, mỗi variant bằng với chính nó và không bằng với các variant khác.

Trait `PartialEq` là bắt buộc, ví dụ, khi sử dụng macro `assert_eq!`, cần có khả năng so sánh hai instance của một kiểu để kiểm tra tính bằng nhau.

Trait `Eq` không có method. Mục đích của nó là để báo hiệu rằng với mọi giá trị của kiểu được chú thích, giá trị bằng với chính nó. Trait `Eq` chỉ có thể được áp dụng cho các kiểu cũng implement `PartialEq`, mặc dù không phải tất cả các kiểu implement `PartialEq` đều có thể implement `Eq`. Một ví dụ về điều này là các kiểu số dấu phẩy động: Implementation của số dấu phẩy động quy định rằng hai instance của giá trị not-a-number (`NaN`) không bằng nhau.

Một ví dụ khi `Eq` là bắt buộc là cho các khóa trong `HashMap<K, V>` để `HashMap<K, V>` có thể biết liệu hai khóa có giống nhau không.

### `PartialOrd` và `Ord` cho So Sánh Thứ Tự

Trait `PartialOrd` cho phép bạn so sánh các instance của một kiểu cho mục đích sắp xếp. Một kiểu implement `PartialOrd` có thể được sử dụng với các toán tử `<`, `>`, `<=`, và `>=`. Bạn chỉ có thể áp dụng trait `PartialOrd` cho các kiểu cũng implement `PartialEq`.

Việc derive `PartialOrd` implement method `partial_cmp`, trả về `Option<Ordering>` sẽ là `None` khi các giá trị được đưa ra không tạo ra thứ tự. Một ví dụ về giá trị không tạo ra thứ tự, mặc dù hầu hết các giá trị của kiểu đó có thể được so sánh, là giá trị dấu phẩy động `NaN`. Gọi `partial_cmp` với bất kỳ số dấu phẩy động nào và giá trị dấu phẩy động `NaN` sẽ trả về `None`.

Khi được derive trên struct, `PartialOrd` so sánh hai instance bằng cách so sánh giá trị trong mỗi trường theo thứ tự mà các trường xuất hiện trong định nghĩa struct. Khi được derive trên enum, các variant của enum được khai báo trước trong định nghĩa enum được coi là nhỏ hơn các variant được liệt kê sau.

Trait `PartialOrd` là bắt buộc, ví dụ, cho method `gen_range` từ crate `rand` tạo ra một giá trị ngẫu nhiên trong phạm vi được chỉ định bởi một biểu thức phạm vi.

Trait `Ord` cho phép bạn biết rằng với bất kỳ hai giá trị nào của kiểu được chú thích, một thứ tự hợp lệ sẽ tồn tại. Trait `Ord` implement method `cmp`, trả về `Ordering` thay vì `Option<Ordering>` vì một thứ tự hợp lệ sẽ luôn có thể. Bạn chỉ có thể áp dụng trait `Ord` cho các kiểu cũng implement `PartialOrd` và `Eq` (và `Eq` yêu cầu `PartialEq`). Khi được derive trên struct và enum, `cmp` hoạt động theo cách tương tự như implementation được derive cho `partial_cmp` với `PartialOrd`.

Một ví dụ khi `Ord` là bắt buộc là khi lưu trữ các giá trị trong `BTreeSet<T>`, một cấu trúc dữ liệu lưu trữ dữ liệu dựa trên thứ tự sắp xếp của các giá trị.

### `Clone` và `Copy` cho Nhân Đôi Giá Trị

Trait `Clone` cho phép bạn tạo rõ ràng một bản sao sâu của một giá trị, và quá trình nhân đôi có thể liên quan đến việc chạy code tùy ý và sao chép dữ liệu heap. Xem phần ["Variables and Data Interacting with Clone"][variables-and-data-interacting-with-clone]<!-- ignore --> trong Chương 4 để biết thêm thông tin về `Clone`.

Việc derive `Clone` implement method `clone`, khi được implement cho toàn bộ kiểu, gọi `clone` trên từng phần của kiểu. Điều này có nghĩa là tất cả các trường hoặc giá trị trong kiểu cũng phải implement `Clone` để derive `Clone`.

Một ví dụ khi `Clone` là bắt buộc là khi gọi method `to_vec` trên một slice. Slice không sở hữu các instance kiểu mà nó chứa, nhưng vector được trả về từ `to_vec` sẽ cần sở hữu các instance của nó, vì vậy `to_vec` gọi `clone` trên mỗi item. Do đó, kiểu được lưu trữ trong slice phải implement `Clone`.

Trait `Copy` cho phép bạn nhân đôi một giá trị bằng cách chỉ sao chép các bit được lưu trữ trên stack; không cần code tùy ý nào. Xem phần ["Stack-Only Data: Copy"][stack-only-data-copy]<!-- ignore --> trong Chương 4 để biết thêm thông tin về `Copy`.

Trait `Copy` không định nghĩa bất kỳ method nào để ngăn các lập trình viên overload những method đó và vi phạm giả định rằng không có code tùy ý nào đang chạy. Theo đó, tất cả các lập trình viên có thể giả định rằng việc sao chép một giá trị sẽ rất nhanh.

Bạn có thể derive `Copy` trên bất kỳ kiểu nào mà tất cả các phần của nó đều implement `Copy`. Một kiểu implement `Copy` cũng phải implement `Clone` vì một kiểu implement `Copy` có implementation tầm thường của `Clone` thực hiện cùng tác vụ như `Copy`.

Trait `Copy` hiếm khi được yêu cầu; các kiểu implement `Copy` có sẵn các tối ưu hóa, có nghĩa là bạn không phải gọi `clone`, điều này làm cho code ngắn gọn hơn.

Mọi thứ có thể thực hiện với `Copy` bạn cũng có thể thực hiện với `Clone`, nhưng code có thể chậm hơn hoặc phải sử dụng `clone` ở một số nơi.

### `Hash` cho Ánh Xạ Giá Trị sang Giá Trị Có Kích Thước Cố Định

Trait `Hash` cho phép bạn lấy một instance của một kiểu có kích thước tùy ý và ánh xạ instance đó sang một giá trị có kích thước cố định bằng cách sử dụng hàm hash. Việc derive `Hash` implement method `hash`. Implementation được derive của method `hash` kết hợp kết quả của việc gọi `hash` trên từng phần của kiểu, có nghĩa là tất cả các trường hoặc giá trị cũng phải implement `Hash` để derive `Hash`.

Một ví dụ khi `Hash` là bắt buộc là khi lưu trữ các khóa trong `HashMap<K, V>` để lưu trữ dữ liệu hiệu quả.

### `Default` cho Các Giá Trị Mặc Định

Trait `Default` cho phép bạn tạo giá trị mặc định cho một kiểu. Việc derive `Default` implement function `default`. Implementation được derive của function `default` gọi function `default` trên từng phần của kiểu, có nghĩa là tất cả các trường hoặc giá trị trong kiểu cũng phải implement `Default` để derive `Default`.

Function `Default::default` thường được sử dụng kết hợp với cú pháp cập nhật struct được thảo luận trong phần ["Creating Instances from Other Instances with Struct Update Syntax"][creating-instances-from-other-instances-with-struct-update-syntax]<!-- ignore --> trong Chương 5. Bạn có thể tùy chỉnh một vài trường của struct và sau đó đặt và sử dụng giá trị mặc định cho phần còn lại của các trường bằng cách sử dụng `..Default::default()`.

Trait `Default` là bắt buộc khi bạn sử dụng method `unwrap_or_default` trên các instance `Option<T>`, ví dụ. Nếu `Option<T>` là `None`, method `unwrap_or_default` sẽ trả về kết quả của `Default::default` cho kiểu `T` được lưu trữ trong `Option<T>`.

[creating-instances-from-other-instances-with-struct-update-syntax]: ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax
[stack-only-data-copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
[variables-and-data-interacting-with-clone]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-clone
[custom-derive-macros]: ch20-05-macros.html#custom-derive-macros
