## Đọc một tệp

Giờ chúng ta sẽ bổ sung chức năng để đọc tệp được chỉ định trong đối số `file_path`. Trước hết, chúng ta cần một tệp mẫu để kiểm thử: ta sẽ dùng một tệp chứa một lượng nhỏ văn bản trên nhiều dòng, với một số từ được lặp lại. Liệt kê 12-3 có một bài thơ của Emily Dickinson, rất phù hợp cho mục đích này! Hãy tạo một tệp tên _poem.txt_ ở thư mục gốc của dự án và nhập bài thơ “I’m Nobody! Who are you?” vào đó.

<Listing number="12-3" file-name="poem.txt" caption="Một bài thơ của Emily Dickinson là một trường hợp kiểm thử tốt.">

```text
{{#include ../listings/ch12-an-io-project/listing-12-03/poem.txt}}
```

</Listing>

Sau khi đã có văn bản, hãy chỉnh sửa _src/main.rs_ và bổ sung mã để đọc tệp, như minh họa trong Liệt kê 12-4.

<Listing number="12-4" file-name="src/main.rs" caption="Đọc nội dung của tệp được chỉ định bởi đối số thứ hai">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/src/main.rs:here}}
```

</Listing>

Trước tiên, chúng ta đưa một phần liên quan của thư viện chuẩn vào bằng câu lệnh `use`: chúng ta cần `std::fs` để thao tác với tệp.

Trong `main`, câu lệnh mới `fs::read_to_string` nhận `file_path`, mở tệp đó và trả về một giá trị kiểu `std::io::Result<String>` chứa nội dung của tệp.

Sau đó, chúng ta lại thêm một câu lệnh `println!` tạm thời để in giá trị của `contents` sau khi tệp được đọc, nhằm kiểm tra xem chương trình hiện tại hoạt động đúng hay không.

Hãy chạy đoạn mã này với bất kỳ chuỗi nào làm đối số dòng lệnh thứ nhất (vì chúng ta vẫn chưa triển khai phần tìm kiếm) và tệp _poem.txt_ làm đối số thứ hai:

```console
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-04/output.txt}}
```

Tốt! Đoạn mã đã đọc và in ra nội dung của tệp. Tuy nhiên, mã hiện tại có một số điểm chưa ổn. Thời điểm này, hàm `main` đang đảm nhiệm nhiều trách nhiệm: nói chung, các hàm sẽ rõ ràng và dễ bảo trì hơn nếu mỗi hàm chỉ chịu trách nhiệm về một ý tưởng duy nhất. Vấn đề khác là chúng ta chưa xử lý lỗi tốt như có thể. Chương trình vẫn còn nhỏ nên những điểm yếu này chưa phải vấn đề lớn, nhưng khi chương trình phát triển, sẽ khó khắc phục chúng một cách gọn gàng hơn. Thực hành tốt là nên bắt đầu tái cấu trúc (refactor) sớm trong quá trình phát triển, vì việc tái cấu trúc một lượng mã nhỏ sẽ dễ dàng hơn nhiều. Chúng ta sẽ thực hiện điều đó ở bước tiếp theo.