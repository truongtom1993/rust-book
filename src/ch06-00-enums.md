# Kiểu dữ liệu Enum và Pattern Matching

Trong chương này, chúng ta sẽ tìm hiểu về các kiểu dữ liệu enumerations, thường
gọi tắt là _enums_. Enums cho phép bạn định nghĩa một kiểu dữ liệu bằng cách
liệt kê tất cả các biến thể có thể của nó. Trước tiên, chúng ta sẽ định nghĩa
và sử dụng một enum để hiểu cách enum có thể mã hoá ý nghĩa cùng với dữ liệu.
Tiếp theo, chúng ta sẽ khám phá một enum đặc biệt hữu ích, gọi là `Option`,
nó biểu thị rằng một giá trị có thể là một cái gì đó hoặc không có gì. Sau đó,
chúng ta sẽ xem cách pattern matching trong expression `match` giúp dễ dàng chạy
các code khác nhau cho các giá trị khác nhau của một enum. Cuối cùng, chúng ta
sẽ tìm hiểu về cách construct `if let` là một idiom tiện lợi và ngắn gọn khác
để xử lý enums trong code của bạn.
