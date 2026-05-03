<!-- Các tiêu đề cũ. Đừng xóa nếu không liên kết có thể bị hỏng. -->

<a id="comparing-performance-loops-vs-iterators"></a>

## Hiệu năng trong Vòng lặp so với Bộ lặp (Iterator)

Để quyết định nên dùng vòng lặp hay bộ lặp, bạn cần biết cách cài đặt nào nhanh hơn: phiên bản hàm `search` với vòng lặp `for` tường minh hay phiên bản sử dụng bộ lặp.

Chúng tôi đã chạy benchmark bằng cách nạp toàn bộ nội dung _The Adventures of
Sherlock Holmes_ của Sir Arthur Conan Doyle vào một `String` và tìm kiếm từ
_the_ trong nội dung đó. Sau đây là kết quả benchmark trên phiên bản `search`
đ dùng vòng lặp `for` và phiên bản sử dụng bộ lặp:

```text
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

Hai cách cài đặt có hiệu năng tương đương nhau! Chúng tôi sẽ không giải thích
mã benchmark ở đây vì mục tiêu không phải là chứng minh hai phiên bản là
hoàn toàn tương đương, mà là để có một cái nhìn tổng quan về cách hai cách
cài đặt này so sánh với nhau về mặt hiệu năng.

Đối với một benchmark toàn diện hơn, bạn nên kiểm tra với nhiều văn bản khác
nhau có kích thước khác nhau làm `contents`, các từ tìm kiếm khác nhau với độ
dài khác nhau làm `query`, và nhiều biến thể khác nữa. Ý chính là:
Mặc dù là một tầng trừu tượng cấp cao, bộ lặp (iterator) khi biên dịch sẽ được
chuyển xuống thành mã máy gần như tương đương với việc bạn tự viết mã cấp thấp.
Iterator là một trong những _trừu tượng zero-cost_ của Rust, nghĩa là việc sử
dụng trừu tượng này không gây thêm chi phí thời gian chạy. Điều này tương tự
cách Bjarne Stroustrup, nhà thiết kế và hiện thực ban đầu của C++, định nghĩa
zero-overhead trong bài keynote ETAPS 2012 “Foundations of C++”:

> Nói chung, các bản cài đặt C++ tuân theo nguyên tắc zero-overhead: Những gì
> bạn không dùng thì bạn không phải trả giá. Và hơn nữa: Những gì bạn có dùng
> thì bạn cũng không thể tự viết tay tốt hơn.

Trong nhiều trường hợp, mã Rust sử dụng iterator được biên dịch thành cùng
một dạng mã assembly mà bạn sẽ viết bằng tay. Các tối ưu như unroll vòng lặp
và loại bỏ kiểm tra biên (bounds checking) khi truy cập mảng vẫn được áp dụng
và khiến mã kết quả cực kỳ hiệu quả. Giờ khi bạn đã biết điều này, bạn có thể
sử dụng iterator và closure mà không phải lo ngại! Chúng khiến mã nguồn có vẻ
mang tính trừu tượng cao hơn nhưng không gây ra chi phí hiệu năng thời gian
chạy.

## Tóm tắt

Closure và iterator là các tính năng của Rust được lấy cảm hứng từ các ý tưởng
trong ngôn ngữ lập trình hàm. Chúng đóng góp vào khả năng của Rust trong việc
biểu đạt rõ ràng các ý tưởng cấp cao với hiệu năng cấp thấp. Cách cài đặt
closure và iterator đảm bảo hiệu năng thời gian chạy không bị ảnh hưởng. Đây
là một phần trong mục tiêu của Rust nhằm cung cấp các trừu tượng zero-cost.

Giờ khi chúng ta đã cải thiện khả năng biểu đạt của dự án I/O, hãy xem tiếp
một số tính năng khác của `cargo` sẽ giúp chúng ta chia sẻ dự án với cộng đồng.