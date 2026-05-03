# Con trỏ thông minh (Smart Pointer)

Con trỏ là một khái niệm tổng quát chỉ một biến chứa một địa chỉ trong bộ nhớ. Địa chỉ này tham chiếu tới, hoặc “trỏ đến”, một dữ liệu khác. Loại con trỏ phổ biến nhất trong Rust là tham chiếu (reference), bạn đã được giới thiệu trong Chương 4. Tham chiếu được ký hiệu bằng ký hiệu `&` và mượn (borrow) giá trị mà nó trỏ tới. Chúng không có khả năng đặc biệt nào ngoài việc tham chiếu tới dữ liệu, và gần như không tạo thêm chi phí (overhead).

Ngược lại, _con trỏ thông minh_ (smart pointer) là các cấu trúc dữ liệu hoạt động giống như con trỏ nhưng đồng thời có thêm siêu dữ liệu (metadata) và các khả năng bổ sung. Khái niệm con trỏ thông minh không phải đặc thù của Rust: chúng xuất phát từ C++ và cũng tồn tại trong nhiều ngôn ngữ khác. Rust cung cấp nhiều loại con trỏ thông minh trong thư viện chuẩn để bổ sung các chức năng vượt ra ngoài những gì tham chiếu thông thường cung cấp. Để tìm hiểu khái niệm tổng quát này, chúng ta sẽ xem qua một vài ví dụ khác nhau về con trỏ thông minh, bao gồm một loại con trỏ thông minh _đếm tham chiếu_ (reference counting). Loại con trỏ này cho phép dữ liệu có nhiều “chủ sở hữu” bằng cách theo dõi số lượng chủ sở hữu và khi không còn chủ sở hữu nào, nó sẽ giải phóng dữ liệu.

Trong Rust, với các khái niệm sở hữu (ownership) và mượn (borrowing), tồn tại thêm một điểm khác biệt giữa tham chiếu và con trỏ thông minh: trong khi tham chiếu chỉ mượn dữ liệu, thì trong nhiều trường hợp con trỏ thông minh _sở hữu_ dữ liệu mà chúng trỏ tới.

Con trỏ thông minh thường được hiện thực (implement) bằng các struct. Khác với một struct thông thường, con trỏ thông minh hiện thực (implement) các trait `Deref` và `Drop`. Trait `Deref` cho phép một instance của struct con trỏ thông minh có thể hành xử như một tham chiếu, nhờ đó bạn có thể viết mã hoạt động với cả tham chiếu lẫn con trỏ thông minh. Trait `Drop` cho phép bạn tuỳ biến đoạn mã sẽ được thực thi khi một instance của con trỏ thông minh ra khỏi phạm vi (scope). Trong chương này, chúng ta sẽ thảo luận cả hai trait này và minh họa lý do tại sao chúng lại quan trọng đối với con trỏ thông minh.

Vì mẫu thiết kế (pattern) con trỏ thông minh là một mẫu thiết kế tổng quát được sử dụng thường xuyên trong Rust, chương này sẽ không thể bao quát mọi loại con trỏ thông minh hiện có. Nhiều thư viện có các loại con trỏ thông minh riêng, và bạn cũng có thể tự viết loại của mình. Chúng ta sẽ đề cập đến những con trỏ thông minh phổ biến nhất trong thư viện chuẩn:

- `Box<T>`: cấp phát giá trị trên heap
- `Rc<T>`: một kiểu đếm tham chiếu cho phép nhiều chủ sở hữu
- `Ref<T>` và `RefMut<T>`: truy cập thông qua `RefCell<T>`, một kiểu áp đặt các quy tắc mượn tại thời điểm chạy (runtime) thay vì thời điểm biên dịch (compile time)

Ngoài ra, chúng ta sẽ tìm hiểu mẫu _khả biến nội tại_ (interior mutability), trong đó một kiểu bất biến (immutable type) phơi bày một API cho phép thay đổi (mutate) giá trị bên trong nó. Chúng ta cũng sẽ thảo luận về vòng tham chiếu (reference cycle): cách chúng có thể gây rò rỉ bộ nhớ và cách phòng tránh.

Bắt đầu thôi!