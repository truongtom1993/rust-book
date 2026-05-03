# Kiểu Tổng Quát (Generic), Trait và Lifetime

Mọi ngôn ngữ lập trình đều có các công cụ để xử lý hiệu quả việc lặp lại các khái niệm. Trong Rust, một trong những công cụ đó là _generics_: các phần tử trừu tượng đại diện cho các kiểu dữ liệu cụ thể hoặc các thuộc tính khác. Chúng ta có thể mô tả hành vi của generics hoặc cách chúng liên hệ với các generics khác mà không cần biết chính xác chúng sẽ được thay thế bằng kiểu gì khi biên dịch và chạy chương trình.

Các hàm có thể nhận tham số thuộc một kiểu tổng quát nào đó thay vì một kiểu cụ thể như `i32` hoặc `String`, tương tự như cách chúng nhận tham số có giá trị chưa biết để chạy cùng một đoạn mã trên nhiều giá trị cụ thể khác nhau. Thực tế, chúng ta đã sử dụng generics ở Chương 6 với `Option<T>`, ở Chương 8 với `Vec<T>` và `HashMap<K, V>`, và ở Chương 9 với `Result<T, E>`. Trong chương này, bạn sẽ tìm hiểu cách định nghĩa các kiểu dữ liệu, hàm và phương thức của riêng mình với generics.

Trước tiên, chúng ta sẽ xem lại cách trích xuất một hàm nhằm giảm sự trùng lặp mã nguồn. Sau đó, chúng ta sẽ sử dụng cùng kỹ thuật đó để tạo một hàm generic từ hai hàm chỉ khác nhau về kiểu của tham số. Chúng ta cũng sẽ giải thích cách sử dụng kiểu generic trong định nghĩa struct và enum.

Tiếp theo, bạn sẽ học cách sử dụng trait để định nghĩa hành vi theo cách tổng quát. Bạn có thể kết hợp trait với kiểu generic để ràng buộc một kiểu generic chỉ chấp nhận những kiểu có hành vi cụ thể, thay vì chấp nhận bất kỳ kiểu nào.

Cuối cùng, chúng ta sẽ thảo luận về _lifetimes_: một dạng generics cung cấp cho trình biên dịch thông tin về cách các tham chiếu (reference) liên hệ với nhau. Lifetime cho phép chúng ta cung cấp đủ thông tin về các giá trị được mượn (borrowed values) để trình biên dịch có thể đảm bảo rằng các tham chiếu sẽ hợp lệ trong nhiều tình huống hơn so với khi không có thông tin này.

## Loại Bỏ Sự Trùng Lặp Bằng Cách Trích Xuất Hàm

Generics cho phép chúng ta thay thế các kiểu cụ thể bằng một placeholder đại diện cho nhiều kiểu khác nhau nhằm loại bỏ sự trùng lặp mã nguồn. Trước khi đi sâu vào cú pháp generics, trước tiên hãy xem cách loại bỏ sự trùng lặp theo một phương pháp không liên quan đến kiểu generic: trích xuất một hàm thay thế các giá trị cụ thể bằng một placeholder đại diện cho nhiều giá trị. Sau đó, chúng ta sẽ áp dụng cùng kỹ thuật này để trích xuất một hàm generic. Khi quan sát cách nhận diện mã nguồn bị lặp để trích xuất thành hàm, bạn cũng sẽ bắt đầu nhận ra những đoạn mã lặp có thể sử dụng generics.

Chúng ta bắt đầu với chương trình ngắn trong Listing 10-1 để tìm số lớn nhất trong một danh sách.

<Listing number="10-1" file-name="src/main.rs" caption="Tìm số lớn nhất trong một danh sách số">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-01/src/main.rs:here}}
```

</Listing>

Chúng ta lưu một danh sách số nguyên trong biến `number_list` và đặt một tham chiếu đến phần tử đầu tiên của danh sách vào biến có tên `largest`. Sau đó, chúng ta lặp qua toàn bộ các số trong danh sách; nếu số hiện tại lớn hơn số được lưu trong `largest`, chúng ta sẽ thay thế tham chiếu trong biến đó. Tuy nhiên, nếu số hiện tại nhỏ hơn hoặc bằng số lớn nhất đã thấy cho đến thời điểm đó, biến sẽ không thay đổi và chương trình tiếp tục kiểm tra số tiếp theo trong danh sách. Sau khi xét tất cả các số trong danh sách, `largest` sẽ tham chiếu đến số lớn nhất, trong ví dụ này là 100.

Bây giờ chúng ta được yêu cầu tìm số lớn nhất trong hai danh sách số khác nhau. Để thực hiện điều này, chúng ta có thể sao chép đoạn mã trong Listing 10-1 và sử dụng cùng logic ở hai vị trí khác nhau trong chương trình, như minh họa trong Listing 10-2.

<Listing number="10-2" file-name="src/main.rs" caption="Mã nguồn để tìm số lớn nhất trong *hai* danh sách số">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-02/src/main.rs}}
```

</Listing>

Mặc dù đoạn mã này hoạt động, việc sao chép mã là tốn công và dễ gây lỗi. Ngoài ra, khi muốn thay đổi logic, chúng ta phải nhớ cập nhật mã ở nhiều vị trí khác nhau.

Để loại bỏ sự trùng lặp này, chúng ta sẽ tạo một mức trừu tượng bằng cách định nghĩa một hàm hoạt động trên bất kỳ danh sách số nguyên nào được truyền vào dưới dạng tham số. Giải pháp này làm cho mã nguồn rõ ràng hơn và cho phép chúng ta biểu diễn khái niệm tìm số lớn nhất trong một danh sách theo cách trừu tượng.

Trong Listing 10-3, chúng ta trích xuất đoạn mã tìm số lớn nhất vào một hàm có tên `largest`. Sau đó, chúng ta gọi hàm này để tìm số lớn nhất trong hai danh sách từ Listing 10-2. Chúng ta cũng có thể sử dụng hàm này cho bất kỳ danh sách giá trị `i32` nào khác trong tương lai.

<Listing number="10-3" file-name="src/main.rs" caption="Mã được trừu tượng hóa để tìm số lớn nhất trong hai danh sách">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-03/src/main.rs:here}}
```

</Listing>

Hàm `largest` có một tham số tên là `list`, đại diện cho bất kỳ lát cắt (slice) cụ thể nào của các giá trị `i32` được truyền vào hàm. Do đó, khi gọi hàm, đoạn mã sẽ được thực thi trên các giá trị cụ thể mà chúng ta cung cấp.

Tóm lại, đây là các bước chúng ta đã thực hiện để thay đổi mã từ Listing 10-2 sang Listing 10-3:

1. Xác định đoạn mã bị trùng lặp.
2. Trích xuất đoạn mã trùng lặp vào phần thân của một hàm, đồng thời xác định các tham số đầu vào và giá trị trả về của đoạn mã đó trong chữ ký hàm (function signature).
3. Cập nhật hai vị trí có mã trùng lặp để gọi hàm này thay thế.

Tiếp theo, chúng ta sẽ sử dụng chính các bước này với generics để giảm sự trùng lặp mã. Tương tự như việc phần thân hàm có thể thao tác trên một `list` trừu tượng thay vì các giá trị cụ thể, generics cho phép mã nguồn thao tác trên các kiểu dữ liệu trừu tượng.

Ví dụ, giả sử chúng ta có hai hàm: một hàm tìm phần tử lớn nhất trong một slice gồm các giá trị `i32`, và một hàm khác tìm phần tử lớn nhất trong một slice gồm các giá trị `char`. Làm thế nào để loại bỏ sự trùng lặp này? Hãy cùng tìm hiểu!
