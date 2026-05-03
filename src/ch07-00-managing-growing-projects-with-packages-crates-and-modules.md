<!-- Old headings. Do not remove or links may break. -->

<a id="managing-growing-projects-with-packages-crates-and-modules"></a>

# Packages, Crates, và Modules

Khi bạn viết các chương trình lớn, việc tổ chức code sẽ trở nên ngày càng quan trọng. Bằng cách nhóm các tính năng liên quan với nhau và tách riêng code có các tính năng khác biệt, bạn sẽ làm rõ nơi tìm kiếm code mà triển khai một tính năng cụ thể và nơi cần thay đổi cách hoạt động của một tính năng.

Các chương trình chúng tôi viết cho đến nay đều nằm trong một module trong một file. Khi project phát triển, bạn nên tổ chức code bằng cách chia nó thành nhiều modules và sau đó thành nhiều files. Một package có thể chứa nhiều binary crates và tùy chọn một library crate. Khi package phát triển, bạn có thể tách các phần ra thành các crates riêng biệt trở thành các dependencies bên ngoài. Chương này bao quát tất cả những kỹ thuật này. Đối với các projects rất lớn gồm một tập hợp các packages liên quan đến nhau phát triển cùng nhau, Cargo cung cấp workspaces, mà chúng tôi sẽ đề cập trong ["Cargo Workspaces"][workspaces]<!-- ignore --> ở Chương 14.

Chúng tôi cũng sẽ thảo luận về việc đóng gói các chi tiết triển khai, cho phép bạn tái sử dụng code ở một cấp độ cao hơn: Khi bạn đã triển khai một thao tác, code khác có thể gọi code của bạn thông qua public interface của nó mà không cần phải biết cách triển khai hoạt động. Cách bạn viết code xác định những phần nào là public để code khác sử dụng và những phần nào là private implementation details mà bạn bảo lưu quyền thay đổi. Đây là một cách khác để giới hạn lượng chi tiết bạn cần ghi nhớ.

Một khái niệm liên quan là scope: Bối cảnh lồng nhau trong đó code được viết có một tập hợp các tên được định nghĩa là "in scope". Khi đọc, viết và biên dịch code, các programmer và compilers cần biết tên cụ thể tại một vị trí cụ thể có đề cập đến một variable, function, struct, enum, module, constant hay item khác và item đó có nghĩa gì. Bạn có thể tạo scopes và thay đổi những tên nào nằm trong hoặc ngoài scope. Bạn không thể có hai items có cùng tên trong cùng một scope; có các công cụ sẵn có để giải quyết các xung đột tên.

Rust có một số tính năng cho phép bạn quản lý tổ chức code của mình, bao gồm những chi tiết nào được exposed, những chi tiết nào là private và những tên nào nằm trong mỗi scope trong các chương trình của bạn. Những tính năng này, đôi khi được gọi chung là _module system_, bao gồm:

* **Packages**: Một tính năng của Cargo cho phép bạn build, test và chia sẻ crates
* **Crates**: Một cây các modules tạo ra một library hoặc executable
* **Modules và use**: Cho phép bạn kiểm soát tổ chức, scope và privacy của paths
* **Paths**: Một cách để đặt tên cho một item, chẳng hạn như struct, function hoặc module

Trong chương này, chúng tôi sẽ bao quát tất cả những tính năng này, thảo luận về cách chúng tương tác và giải thích cách sử dụng chúng để quản lý scope. Khi kết thúc, bạn sẽ có sự hiểu biết vững chắc về module system và có khả năng làm việc với scopes như một chuyên gia!

[workspaces]: ch14-03-cargo-workspaces.html
