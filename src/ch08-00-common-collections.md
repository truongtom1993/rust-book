# Các Collection Phổ Biến

Thư viện chuẩn của Rust bao gồm một số cấu trúc dữ liệu rất hữu ích được gọi là
_collections_. Hầu hết các kiểu dữ liệu khác chỉ đại diện cho một giá trị cụ thể, nhưng
các collections có thể chứa nhiều giá trị. Không giống với các kiểu array và tuple được
xây dựng sẵn, dữ liệu mà các collections trỏ tới được lưu trữ trên heap,
điều này có nghĩa là lượng dữ liệu không cần phải được biết tại thời điểm biên dịch và có thể
tăng hoặc giảm khi chương trình chạy. Mỗi loại collection có các khả năng và chi phí khác nhau,
và việc chọn một cái phù hợp cho tình huống hiện tại của bạn là một kỹ năng bạn sẽ phát triển
theo thời gian. Trong chương này, chúng ta sẽ thảo luận về ba collections được sử dụng rất
thường xuyên trong các chương trình Rust:

- Một _vector_ cho phép bạn lưu trữ một số lượng giá trị thay đổi cạnh nhau.
- Một _string_ là một collection của các ký tự. Chúng tôi đã đề cập đến kiểu `String`
  trước đây, nhưng trong chương này, chúng ta sẽ thảo luận về nó sâu hơn.
- Một _hash map_ cho phép bạn liên kết một giá trị với một khóa cụ thể. Nó là
  một triển khai cụ thể của cấu trúc dữ liệu tổng quát hơn được gọi là _map_.

Để tìm hiểu về các loại collections khác do thư viện chuẩn cung cấp,
hãy xem [tài liệu][collections].

Chúng ta sẽ thảo luận về cách tạo và cập nhật vectors, strings, và hash maps, cũng như
những gì làm cho mỗi cái trở nên đặc biệt.

[collections]: ../std/collections/index.html
