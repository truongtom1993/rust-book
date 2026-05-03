# Một dự án I/O: Xây dựng chương trình dòng lệnh

Chương này tổng hợp lại nhiều kỹ năng bạn đã học cho đến hiện tại và đồng thời khám phá thêm một số tính năng khác của thư viện chuẩn. Chúng ta sẽ xây dựng một công cụ dòng lệnh tương tác với đầu vào/đầu ra từ tệp và dòng lệnh để thực hành một số khái niệm Rust mà bạn đã nắm được.

Tốc độ, tính an toàn, khả năng biên dịch thành một tệp nhị phân đơn và hỗ trợ đa nền tảng khiến Rust trở thành một ngôn ngữ lý tưởng để tạo ra các công cụ dòng lệnh. Vì vậy, trong dự án này, chúng ta sẽ tự xây dựng một phiên bản của công cụ tìm kiếm dòng lệnh kinh điển `grep` (**g**lobally search a **r**egular **e**xpression and **p**rint). Trong trường hợp sử dụng đơn giản nhất, `grep` tìm kiếm một chuỗi cụ thể trong một tệp được chỉ định. Để làm được điều đó, `grep` nhận đường dẫn tệp và chuỗi tìm kiếm làm đối số. Sau đó, nó đọc tệp, tìm các dòng trong tệp có chứa chuỗi đó và in các dòng đó ra.

Trong quá trình thực hiện, chúng ta sẽ chỉ ra cách làm cho công cụ dòng lệnh của mình sử dụng các tính năng của terminal mà nhiều công cụ dòng lệnh khác sử dụng. Chúng ta sẽ đọc giá trị của một biến môi trường để cho phép người dùng cấu hình hành vi của công cụ. Chúng ta cũng sẽ in thông báo lỗi ra luồng lỗi chuẩn (`stderr`) thay vì luồng xuất chuẩn (`stdout`) để, chẳng hạn, người dùng có thể chuyển hướng phần kết quả thành công sang một tệp trong khi vẫn nhìn thấy thông báo lỗi trên màn hình.

Một thành viên trong cộng đồng Rust, Andrew Gallant, đã xây dựng một phiên bản `grep` đầy đủ tính năng và rất nhanh, có tên là `ripgrep`. So với `ripgrep`, phiên bản của chúng ta sẽ tương đối đơn giản, nhưng chương này sẽ cung cấp cho bạn một số kiến thức nền tảng cần thiết để hiểu các dự án thực tế như `ripgrep`.

Dự án `grep` của chúng ta sẽ kết hợp một số khái niệm mà bạn đã học đến hiện tại:

- Tổ chức mã nguồn ([Chương 7][ch7]<!-- ignore -->)
- Sử dụng vector và string ([Chương 8][ch8]<!-- ignore -->)
- Xử lý lỗi ([Chương 9][ch9]<!-- ignore -->)
- Sử dụng trait và lifetime khi phù hợp ([Chương 10][ch10]<!-- ignore -->)
- Viết kiểm thử ([Chương 11][ch11]<!-- ignore -->)

Chúng ta cũng sẽ giới thiệu ngắn gọn về closure, iterator và trait object, những nội dung sẽ được trình bày chi tiết trong [Chương 13][ch13]<!-- ignore --> và [Chương 18][ch18]<!-- ignore -->.

[ch7]: ch07-00-managing-growing-projects-with-packages-crates-and-modules.html
[ch8]: ch08-00-common-collections.html
[ch9]: ch09-00-error-handling.html
[ch10]: ch10-00-generics.html
[ch11]: ch11-00-testing.html
[ch13]: ch13-00-functional-features.html
[ch18]: ch18-00-oop.html