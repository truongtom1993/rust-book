# Sử dụng Structs để Cấu trúc Dữ liệu Liên quan

_Struct_ hay _structure_ là một kiểu dữ liệu tùy chỉnh cho phép bạn đóng gói
và đặt tên cho nhiều giá trị liên quan tạo thành một nhóm có nghĩa. Nếu bạn
quen thuộc với các ngôn ngữ hướng đối tượng, struct giống như các thuộc tính
dữ liệu của một object. Trong chương này, chúng ta sẽ so sánh và đối chiếu
tuple với structs để xây dựng trên những gì bạn đã biết và chứng minh khi
nào structs là cách tốt hơn để nhóm dữ liệu.

Chúng ta sẽ chứng minh cách định nghĩa và khởi tạo structs. Chúng ta sẽ
thảo luận cách định nghĩa các associated functions, đặc biệt là loại
associated functions gọi là _methods_, để chỉ định behavior liên kết với
một kiểu struct. Structs và enums (được thảo luận trong Chương 6) là những
khối xây dựng cơ bản để tạo các kiểu mới trong miền của chương trình của bạn
để tận dụng đầy đủ kiểm tra kiểu lúc compile-time của Rust.
