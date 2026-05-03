# Các Tính Năng Ngôn Ngữ Mang Tính Hàm: Iterator và Closure

Thiết kế của Rust chịu ảnh hưởng từ nhiều ngôn ngữ và kỹ thuật hiện có, trong đó một ảnh hưởng đáng kể là _lập trình hàm_ (functional programming).
Lập trình theo phong cách hàm thường bao gồm việc sử dụng hàm như các giá trị: truyền chúng như tham số, trả về chúng từ các hàm khác, gán chúng cho biến để thực thi sau này, v.v.

Trong chương này, chúng ta sẽ không tranh luận về lập trình hàm là gì hoặc không phải là gì, mà sẽ tập trung thảo luận một số tính năng của Rust tương đồng với các tính năng trong nhiều ngôn ngữ thường được xem là ngôn ngữ hàm.

Cụ thể hơn, chúng ta sẽ đề cập đến:

- _Closure_ — một cấu trúc giống hàm có thể được lưu trong biến
- _Iterator_ — một cơ chế xử lý một dãy phần tử
- Cách sử dụng closure và iterator để cải tiến dự án I/O ở Chương 12
- Hiệu năng của closure và iterator (tiết lộ trước: Chúng nhanh hơn bạn nghĩ!)

Chúng ta đã đề cập đến một số tính năng khác của Rust, chẳng hạn như pattern matching và enum, cũng chịu ảnh hưởng từ phong cách lập trình hàm.
Vì việc nắm vững closure và iterator là một phần quan trọng trong việc viết mã Rust nhanh và đúng chuẩn (idiomatic), nên toàn bộ chương này sẽ được dành để trình bày về chúng.