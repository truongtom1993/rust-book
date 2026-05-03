# Pattern và Matching

Pattern là một cú pháp đặc biệt trong Rust để so khớp với cấu trúc của các kiểu dữ liệu, cả phức tạp lẫn đơn giản. Sử dụng pattern cùng với `match` expressions và các cấu trúc khác giúp bạn có kiểm soát tốt hơn trên flow điều khiển của chương trình. Một pattern bao gồm một số tổ hợp của các phần tử sau:

- Literals
- Destructured arrays, enums, structs, hoặc tuples
- Variables
- Wildcards
- Placeholders

Một số ví dụ về pattern bao gồm `x`, `(a, 3)`, và `Some(Color::Red)`. Trong những ngữ cảnh mà pattern là hợp lệ, những thành phần này mô tả hình dạng của dữ liệu. Chương trình của chúng ta sau đó so khớp các giá trị với các pattern để xác định xem nó có hình dạng dữ liệu chính xác không để tiếp tục chạy một đoạn mã cụ thể.

Để sử dụng pattern, chúng ta so sánh nó với một giá trị nào đó. Nếu pattern khớp với giá trị, chúng ta sử dụng các phần giá trị trong mã của chúng ta. Hãy nhớ lại `match` expressions trong Chương 6 đã sử dụng pattern, chẳng hạn như ví dụ máy phân loại tiền xu. Nếu giá trị phù hợp với hình dạng của pattern, chúng ta có thể sử dụng các phần được đặt tên. Nếu không, mã liên kết với pattern sẽ không chạy.

Chương này là một tài liệu tham khảo về tất cả những điều liên quan đến pattern. Chúng ta sẽ bao gồm những nơi hợp lệ để sử dụng pattern, sự khác biệt giữa pattern refutable và irrefutable, và các loại cú pháp pattern khác nhau mà bạn có thể gặp phải. Khi kết thúc chương, bạn sẽ biết cách sử dụng pattern để diễn đạt nhiều khái niệm một cách rõ ràng.
