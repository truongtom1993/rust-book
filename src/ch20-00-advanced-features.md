# Các Tính Năng Nâng Cao

Đến bây giờ, bạn đã học được các phần thường được sử dụng nhất của ngôn ngữ lập trình Rust. Trước khi chúng ta làm một dự án khác trong Chương 21, chúng ta sẽ xem xét một vài khía cạnh của ngôn ngữ mà bạn có thể gặp phải thỉnh thoảng nhưng có thể không sử dụng hàng ngày. Bạn có thể sử dụng chương này như một tài liệu tham khảo khi bạn gặp phải bất kỳ điều gì chưa biết. Các tính năng được đề cập ở đây hữu ích trong những tình huống rất cụ thể. Mặc dù bạn có thể không thường xuyên sử dụng chúng, chúng tôi muốn chắc chắn rằng bạn hiểu toàn bộ các tính năng mà Rust cung cấp.

Trong chương này, chúng ta sẽ đề cập đến:

- Unsafe Rust: Cách thoát khỏi một số đảm bảo của Rust và chịu trách nhiệm thủ công duy trì các đảm bảo đó
- Advanced traits: Các loại liên kết, các tham số kiểu mặc định, cú pháp đủ điều kiện, siêu traits, và mô hình newtype liên quan đến traits
- Advanced types: Thêm về mô hình newtype, các alias kiểu, kiểu never, và các kiểu được định kích thước động
- Advanced functions and closures: Con trỏ hàm và trả về closures
- Macros: Cách để định nghĩa code mà định nghĩa thêm code tại thời gian compile

Đây là một bộ sưu tập phong phú các tính năng của Rust với cái gì dành cho mọi người! Hãy bắt đầu!
