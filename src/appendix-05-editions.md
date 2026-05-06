## Phụ Lục E: Các Edition

Trong Chương 1, bạn đã thấy rằng `cargo new` thêm một ít metadata vào file _Cargo.toml_ của bạn về một edition. Phụ lục này nói về ý nghĩa của điều đó!

Ngôn ngữ Rust và trình biên dịch có chu kỳ phát hành sáu tuần, có nghĩa là người dùng nhận được một luồng liên tục các tính năng mới. Các ngôn ngữ lập trình khác phát hành các thay đổi lớn hơn nhưng ít thường xuyên hơn; Rust phát hành các bản cập nhật nhỏ hơn thường xuyên hơn. Sau một thời gian, tất cả những thay đổi nhỏ này tích lũy lại. Nhưng từ bản phát hành này sang bản phát hành khác, có thể khó nhìn lại và nói rằng, "Ồ, giữa Rust 1.10 và Rust 1.31, Rust đã thay đổi rất nhiều!"

Cứ khoảng ba năm một lần, nhóm Rust tạo ra một Rust _edition_ mới. Mỗi edition tổng hợp lại các tính năng đã được đưa vào thành một gói rõ ràng với tài liệu và công cụ được cập nhật đầy đủ. Các edition mới được phát hành như một phần của quy trình phát hành sáu tuần thông thường.

Các edition phục vụ các mục đích khác nhau cho những người khác nhau:

- Đối với người dùng Rust tích cực, một edition mới tổng hợp các thay đổi dần dần thành một gói dễ hiểu.
- Đối với những người không phải người dùng, một edition mới báo hiệu rằng một số tiến bộ lớn đã đạt được, điều này có thể khiến Rust đáng để xem xét lại.
- Đối với những người phát triển Rust, một edition mới cung cấp một điểm tập hợp cho dự án nói chung.

Tại thời điểm viết bài này, có bốn Rust edition: Rust 2015, Rust 2018, Rust 2021, và Rust 2024. Cuốn sách này được viết theo idiom của Rust edition 2024.

Khóa `edition` trong _Cargo.toml_ cho biết trình biên dịch nên sử dụng edition nào cho code của bạn. Nếu khóa không tồn tại, Rust sử dụng `2015` làm giá trị edition để tương thích ngược.

Mỗi dự án có thể chọn sử dụng một edition khác với edition mặc định 2015. Các edition có thể chứa các thay đổi không tương thích, chẳng hạn như bao gồm một từ khóa mới xung đột với các identifier trong code. Tuy nhiên, trừ khi bạn chọn sử dụng những thay đổi đó, code của bạn sẽ tiếp tục biên dịch ngay cả khi bạn nâng cấp phiên bản trình biên dịch Rust mà bạn sử dụng.

Tất cả các phiên bản trình biên dịch Rust đều hỗ trợ bất kỳ edition nào tồn tại trước khi phát hành trình biên dịch đó, và chúng có thể liên kết các crate của bất kỳ edition được hỗ trợ nào với nhau. Thay đổi edition chỉ ảnh hưởng đến cách trình biên dịch phân tích cú pháp code ban đầu. Do đó, nếu bạn đang sử dụng Rust 2015 và một trong những dependency của bạn sử dụng Rust 2018, dự án của bạn sẽ biên dịch và có thể sử dụng dependency đó. Tình huống ngược lại, nơi dự án của bạn sử dụng Rust 2018 và một dependency sử dụng Rust 2015, cũng hoạt động tốt.

Để rõ ràng: Hầu hết các tính năng sẽ có sẵn trên tất cả các edition. Các developer sử dụng bất kỳ Rust edition nào sẽ tiếp tục thấy các cải tiến khi các bản phát hành stable mới được tạo ra. Tuy nhiên, trong một số trường hợp, chủ yếu khi các từ khóa mới được thêm vào, một số tính năng mới có thể chỉ có sẵn trong các edition sau. Bạn sẽ cần chuyển đổi edition nếu bạn muốn tận dụng các tính năng đó.

Để biết thêm chi tiết, xem [_The Rust Edition Guide_][edition-guide]. Đây là một cuốn sách đầy đủ liệt kê sự khác biệt giữa các edition và giải thích cách tự động nâng cấp code của bạn lên một edition mới thông qua `cargo fix`.

[edition-guide]: https://doc.rust-lang.org/stable/edition-guide
