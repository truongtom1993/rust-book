# Dự Án Cuối Cùng: Xây Dựng một Web Server Đa Luồng

Đã là một hành trình dài, nhưng chúng ta đã đến được cuối sách. Trong chương này, chúng ta sẽ xây dựng một dự án nữa cùng nhau để chứng minh một số khái niệm mà chúng ta đã đề cập trong các chương cuối cùng, cũng như ôn tập lại một số bài học trước đó.

Đối với dự án cuối cùng của chúng ta, chúng ta sẽ tạo một web server nói "Hello!" và trông giống như Hình 21-1 trong một trình duyệt web.

Dưới đây là kế hoạch của chúng ta để xây dựng web server:

1. Tìm hiểu một chút về TCP và HTTP.
2. Nghe các kết nối TCP trên một socket.
3. Phân tích một số lượng nhỏ các yêu cầu HTTP.
4. Tạo một phản hồi HTTP thích hợp.
5. Cải thiện thông lượng của server với một thread pool.

<img alt="Ảnh chụp màn hình của một trình duyệt web truy cập địa chỉ 127.0.0.1:8080 hiển thị một trang web với nội dung văn bản "Hello! Hi from Rust"" src="img/trpl21-01.png" class="center" style="width: 50%;" />

<span class="caption">Hình 21-1: Dự án chia sẻ cuối cùng của chúng ta</span>

Trước khi bắt đầu, chúng ta nên đề cập đến hai chi tiết. Trước tiên, phương pháp mà chúng ta sẽ sử dụng sẽ không phải là cách tốt nhất để xây dựng một web server với Rust. Các thành viên cộng đồng đã xuất bản một số crates sẵn sàng sản xuất có sẵn tại [crates.io](https://crates.io/) cung cấp các triển khai web server và thread pool hoàn chỉnh hơn những gì chúng ta sẽ xây dựng. Tuy nhiên, ý định của chúng ta trong chương này là giúp bạn học tập, chứ không phải để lấy con đường dễ dàng. Vì Rust là một ngôn ngữ lập trình hệ thống, chúng ta có thể chọn mức độ trừu tượng mà chúng ta muốn làm việc với và có thể đi đến một mức độ thấp hơn mức độ có thể hoặc thực tế trong các ngôn ngữ khác.

Thứ hai, chúng ta sẽ không sử dụng async và await ở đây. Xây dựng một thread pool đã là một thách thức đủ lớn, mà không cần phải xây dựng một async runtime! Tuy nhiên, chúng ta sẽ lưu ý cách async và await có thể áp dụng cho một số vấn đề tương tự mà chúng ta sẽ thấy trong chương này. Cuối cùng, như chúng ta đã lưu ý lại trong Chương 17, nhiều async runtimes sử dụng thread pools để quản lý công việc của họ.

Do đó, chúng ta sẽ viết HTTP server cơ bản và thread pool theo cách thủ công để bạn có thể học các ý tưởng chung và kỹ thuật đằng sau các crates mà bạn có thể sử dụng trong tương lai.
