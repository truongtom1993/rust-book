## Mở rộng Cargo với các lệnh tùy chỉnh

Cargo được thiết kế để bạn có thể mở rộng nó bằng các lệnh con (subcommand) mới mà không cần sửa đổi bản thân Cargo. Nếu trong `$PATH` của bạn có một binary tên là `cargo-something`, bạn có thể chạy nó như một subcommand của Cargo bằng cách chạy `cargo something`. Các lệnh tùy chỉnh như vậy cũng sẽ được liệt kê khi bạn chạy `cargo --list`. Khả năng dùng `cargo install` để cài đặt các phần mở rộng và sau đó chạy chúng y hệt như các công cụ tích hợp sẵn của Cargo là một lợi ích cực kỳ tiện lợi trong thiết kế của Cargo!

## Tóm tắt

Chia sẻ mã nguồn thông qua Cargo và [crates.io](https://crates.io/) là một phần quan trọng giúp hệ sinh thái Rust trở nên hữu ích cho rất nhiều tác vụ khác nhau. Thư viện chuẩn (standard library) của Rust nhỏ gọn và ổn định, nhưng các crate thì lại dễ dàng chia sẻ, sử dụng, và cải tiến theo một lộ trình khác với ngôn ngữ. Đừng ngần ngại chia sẻ những đoạn mã hữu ích với bạn lên [crates.io](https://crates.io/); rất có khả năng chúng cũng sẽ hữu ích với những người khác!