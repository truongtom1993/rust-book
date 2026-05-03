## Tùy biến bản dựng với hồ sơ phát hành (release profiles)

Trong Rust, _hồ sơ phát hành_ (release profile) là các hồ sơ được định nghĩa sẵn và có thể tùy biến với những cấu hình khác nhau, cho phép lập trình viên kiểm soát chi tiết hơn các tùy chọn khi biên dịch mã. Mỗi hồ sơ được cấu hình một cách độc lập với các hồ sơ khác.

Cargo có hai hồ sơ chính: hồ sơ `dev` mà Cargo sử dụng khi bạn chạy `cargo build`, và hồ sơ `release` mà Cargo sử dụng khi bạn chạy `cargo build --release`. Hồ sơ `dev` được định nghĩa với các giá trị mặc định phù hợp cho quá trình phát triển, còn hồ sơ `release` thì có các giá trị mặc định tối ưu cho các bản dựng phát hành.

Các tên hồ sơ này có thể đã quen thuộc với bạn từ phần output của quá trình build:

```console
$ cargo build
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
$ cargo build --release
    Finished `release` profile [optimized] target(s) in 0.32s
```

`dev` và `release` chính là các hồ sơ khác nhau mà trình biên dịch sử dụng.

Cargo có các thiết lập mặc định cho từng hồ sơ, được áp dụng trong trường hợp bạn chưa bổ sung bất kỳ phần `[profile.*]` nào trong tệp _Cargo.toml_ của dự án. Bằng cách thêm các phần `[profile.*]` cho bất kỳ hồ sơ nào bạn muốn tùy biến, bạn sẽ ghi đè một phần (hoặc toàn bộ) các thiết lập mặc định. Ví dụ, dưới đây là các giá trị mặc định cho thiết lập `opt-level` của hồ sơ `dev` và `release`:

<span class="filename">Tên tệp: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

Thiết lập `opt-level` điều khiển số lượng tối ưu hóa mà Rust sẽ áp dụng lên mã của bạn, với phạm vi từ 0 đến 3. Áp dụng nhiều tối ưu hóa hơn sẽ làm tăng thời gian biên dịch, nên nếu bạn đang trong giai đoạn phát triển và biên dịch mã thường xuyên, bạn sẽ muốn ít tối ưu hóa hơn để quá trình biên dịch nhanh hơn, dù mã chạy chậm hơn. Do đó, `opt-level` mặc định cho `dev` là `0`. Khi bạn sẵn sàng phát hành mã của mình, việc chấp nhận thời gian biên dịch lâu hơn là phù hợp. Bạn chỉ biên dịch ở chế độ phát hành (release mode) một lần, nhưng sẽ chạy chương trình đã biên dịch rất nhiều lần, nên chế độ phát hành đánh đổi thời gian biên dịch dài hơn để có mã chạy nhanh hơn. Đó là lý do `opt-level` mặc định cho hồ sơ `release` là `3`.

Bạn có thể ghi đè một thiết lập mặc định bằng cách đặt một giá trị khác cho nó trong _Cargo.toml_. Ví dụ, nếu chúng ta muốn dùng mức tối ưu hóa 1 trong hồ sơ phát triển, ta có thể thêm hai dòng sau vào tệp _Cargo.toml_ của dự án:

<span class="filename">Tên tệp: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 1
```

Đoạn cấu hình này ghi đè giá trị mặc định là `0`. Bây giờ, khi chúng ta chạy `cargo build`, Cargo sẽ sử dụng các giá trị mặc định của hồ sơ `dev` cộng với phần tùy biến `opt-level` mà chúng ta đã cấu hình. Vì chúng ta đặt `opt-level` là `1`, Cargo sẽ áp dụng nhiều tối ưu hóa hơn so với mặc định, nhưng vẫn ít hơn so với bản dựng ở chế độ phát hành.

Để xem đầy đủ danh sách các tùy chọn cấu hình và giá trị mặc định cho từng hồ sơ, tham khảo thêm tại [tài liệu của Cargo](https://doc.rust-lang.org/cargo/reference/profiles.html).