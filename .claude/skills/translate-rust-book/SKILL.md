---
name: translate-rust-book
description: Dịch các bài viết trong Rust Book từ tiếng Anh sang tiếng Việt. Sử dụng skill này khi bạn muốn dịch một hoặc nhiều file markdown (.md) cụ thể trong dự án Rust learning. Skill sẽ giữ nguyên toàn bộ các thẻ HTML, link, code block, và các phần metadata (YAML frontmatter). Chỉ dịch nội dung văn bản chính. Bạn có thể cung cấp danh sách đường dẫn file cần dịch hoặc chỉ định các file cụ thể từ SUMMARY.md.
compatibility: Requires file system access and ability to read/write .md files in the Rust learning project
---

# Dịch Rust Book sang Tiếng Việt

## Mục tiêu
Dịch nội dung các bài viết trong Rust Book từ tiếng Anh sang tiếng Việt, giữ nguyên cấu trúc file và các phần code.

## Quy tắc dịch

### 1. Nội dung KHÔNG được dịch - Giữ nguyên gốc
- Các đoạn HTML metadata (old headings, listing captions, code include):
  ```html
  <!-- Old headings. Do not remove or links may break. -->
  <a id="following-the-pointer-to-the-value-with-the-dereference-operator"></a>
  ```
  
- Các thẻ Listing (code snippets):
  ```
  <Listing number="15-6" file-name="src/main.rs" caption="Using the dereference operator...">
  {{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-06/src/main.rs}}
  </Listing>
  ```

- YAML frontmatter (nếu có)
- Markdown links, file paths, code block references
- Tên hàm, tên biến, syntax Rust trong backticks hoặc code blocks

### 2. Nội dung ĐƯỢC dịch
- Tiêu đề chương, phần tiêu đề (headings)
- Đoạn văn bản giải thích chính
- Bullet points, danh sách
- Captions trong Listing (phần sau "caption=")
- Ghi chú, lưu ý

### 3. Phong cách dịch
- **Văn phong dễ hiểu**: Dịch cho developers, không cần quá chính tác
- **Giữ thuật ngữ chuyên ngành**: function, closure, trait, lifetime, ownership, borrowing, pattern matching, iterator, async/await, v.v.
- **Dịch tự nhiên**: Không dịch máy móc, đảm bảo câu văn dễ hiểu trong tiếng Việt
- **Ví dụ**: 
  - "function" → giữ nguyên "function"
  - "Rust programming language" → "ngôn ngữ lập trình Rust"
  - "the ownership system" → "hệ thống ownership"

## Quy trình dịch

### Bước 1: Xác nhận file cần dịch
- Xin người dùng cung cấp danh sách file cần dịch
- File phải nằm trong thư mục `/src` của dự án
- Ví dụ: `src/ch01-00-getting-started.md`, `src/ch02-00-guessing-game-tutorial.md`

### Bước 2: Đọc file gốc
- Đọc toàn bộ nội dung file markdown
- Xác định các phần không được dịch (HTML, code block, metadata)
- Xác định các phần cần dịch (heading, paragraph, list)

### Bước 3: Dịch nội dung
- Dịch từng phần, giữ nguyên cấu trúc markdown
- Đảm bảo các thẻ HTML và code block không bị thay đổi
- Kiểm tra lại các thuật ngữ chuyên ngành

### Bước 4: Ghi file
- Ghi đè file gốc bằng nội dung dịch
- Đảm bảo encoding UTF-8
- Giữ nguyên line breaks và formatting

### Bước 5: Xác nhận hoàn thành
- Thông báo file nào đã dịch
- Sẵn sàng dịch file tiếp theo hoặc điều chỉnh

## Ví dụ dịch

### Trước dịch:
```markdown
## Understanding Ownership

In this section, we'll explore the ownership system, which is a set of rules that govern how a Rust program manages memory. 

The ownership system has three main rules:
1. Each value in Rust has a variable that is called its owner.
2. There can only be one owner at a time.
3. When the owner goes out of scope, the value will be dropped.
```

### Sau dịch:
```markdown
## Hiểu về Ownership

Trong phần này, chúng ta sẽ khám phá hệ thống ownership, đây là một tập hợp các quy tắc để quản lý bộ nhớ trong chương trình Rust.

Hệ thống ownership có ba quy tắc chính:
1. Mỗi giá trị trong Rust có một biến được gọi là owner của nó.
2. Chỉ có thể có một owner tại một thời điểm.
3. Khi owner vượt ra ngoài scope, giá trị sẽ bị drop.
```

## Xử lý các trường hợp đặc biệt

### Code block
Giữ nguyên toàn bộ code block, không dịch:
```rust
fn main() {
    let mut s = String::from("hello");
    s.push_str(", world!");
    println!("{}", s);
}
```

### Listing captions
Dịch phần caption:
```
<Listing number="15-6" file-name="src/main.rs" caption="Sử dụng toán tử dereference để theo dõi một reference đến giá trị `i32`">
```

### Link
Giữ nguyên link, dịch text:
```markdown
[Tài liệu chính thức](https://doc.rust-lang.org)
```

## Cách sử dụng skill này

1. Nói: "Dịch file src/ch04-00-understanding-ownership.md sang tiếng Việt"
2. Hoặc: "Dịch các file sau: src/ch04-00-understanding-ownership.md, src/ch04-01-what-is-ownership.md"
3. Skill sẽ đọc file, dịch nội dung, ghi đè file gốc
4. Sau dịch, bạn có thể review và yêu cầu điều chỉnh nếu cần

## Lưu ý
- Skill chỉ dịch, không tạo file mới. Nó ghi đè file gốc.
- Trước khi ghi, skill sẽ xin xác nhận danh sách file sẽ bị thay đổi
- Nếu cần điều chỉnh sau dịch, bạn có thể yêu cầu dịch lại hoặc sửa từng phần
