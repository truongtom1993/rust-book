## Comments

Tất cả các lập trình viên đều cố gắng làm cho code của họ dễ hiểu, nhưng đôi khi cần có giải thích thêm. Trong những trường hợp đó, các lập trình viên để lại _comments_ trong source code mà compiler sẽ bỏ qua nhưng những người đọc source code có thể thấy hữu ích.

Đây là một comment đơn giản:

```rust
// hello, world
```

Trong Rust, kiểu comment thông dụng bắt đầu comment bằng hai dấu gạch chéo, và comment tiếp tục đến cuối dòng. Đối với các comments kéo dài hơn một dòng, bạn cần bao gồm `//` trên mỗi dòng, như sau:

```rust
// So we're doing something complicated here, long enough that we need
// multiple lines of comments to do it! Whew! Hopefully, this comment will
// explain what's going on.
```

Comments cũng có thể được đặt ở cuối các dòng chứa code:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-24-comments-end-of-line/src/main.rs}}
```

Nhưng bạn sẽ thường thấy chúng được dùng theo định dạng này hơn, với comment trên một dòng riêng phía trên code mà nó annotate:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-25-comments-above-line/src/main.rs}}
```

Rust cũng có một loại comment khác, documentation comments, mà chúng ta sẽ thảo luận trong phần ["Publishing a Crate to Crates.io"][publishing]<!-- ignore --> của Chương 14.

[publishing]: ch14-02-publishing-to-crates-io.html
