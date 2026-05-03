# Xử Lý Lỗi

Lỗi là một phần không thể tránh trong phần mềm, vì vậy Rust cung cấp một số tính năng
để xử lý các tình huống khi có gì đó sai sót. Trong nhiều trường hợp, Rust yêu cầu
bạn phải thừa nhận khả năng xảy ra lỗi và thực hiện một số hành động trước khi
code của bạn có thể biên dịch. Yêu cầu này giúp chương trình của bạn trở nên mạnh mẽ hơn
bằng cách đảm bảo rằng bạn sẽ phát hiện lỗi và xử lý chúng một cách thích hợp
trước khi triển khai code của bạn vào production!

Rust phân chia lỗi thành hai loại chính: lỗi có thể phục hồi và lỗi không thể phục hồi.
Đối với một _lỗi có thể phục hồi_, chẳng hạn như _lỗi tệp không tìm thấy_,
chúng ta thường chỉ muốn báo cáo vấn đề cho người dùng và thử lại thao tác.
_Lỗi không thể phục hồi_ luôn là triệu chứng của các bugs, chẳng hạn như cố gắng truy cập
một vị trí ngoài phạm vi của một mảng, do đó chúng ta muốn dừng chương trình ngay lập tức.

Hầu hết các ngôn ngữ không phân biệt giữa hai loại lỗi này và xử lý
cả hai theo cách tương tự, sử dụng các cơ chế như exceptions. Rust không có
exceptions. Thay vào đó, nó có kiểu `Result<T, E>` cho những lỗi có thể phục hồi và
macro `panic!` dừng thực thi khi chương trình gặp một lỗi không thể phục hồi. Chương này
bao gồm gọi `panic!` trước tiên và sau đó nói về việc trả về các giá trị `Result<T, E>`.
Ngoài ra, chúng ta sẽ khám phá các cân nhắc khi quyết định liệu có nên cố gắng
phục hồi từ một lỗi hay dừng thực thi.
