## Xử lý một loạt các mục với Iterator

Mẫu iterator cho phép bạn thực hiện một số nhiệm vụ trên một loạt các mục lần lượt. Iterator chịu trách nhiệm thực hiện logic lặp qua từng mục và xác định khi nào loạt mục đã kết thúc. Khi bạn sử dụng iterator, bạn không phải tự triển khai lại logic đó.

Trong Rust, iterator là _lazy_ (lười biếng), có nghĩa là chúng không có tác dụng cho đến khi bạn gọi các phương thức mà consume iterator để sử dụng nó. Ví dụ, code trong Listing 13-10 tạo một iterator trên các mục trong vector `v1` bằng cách gọi phương thức `iter` được định nghĩa trên `Vec<T>`. Code này tự nó không làm bất cứ điều gì hữu ích.

<Listing number="13-10" file-name="src/main.rs" caption="Tạo một iterator">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-10/src/main.rs:here}}
```

</Listing>

Iterator được lưu trữ trong biến `v1_iter`. Sau khi tạo một iterator, chúng ta có thể sử dụng nó theo nhiều cách khác nhau. Trong Listing 3-5, chúng ta đã lặp qua một mảng bằng cách sử dụng vòng lặp `for` để thực thi một số code trên từng mục của nó. Dưới hầu phủ, điều này ngầm tạo và sau đó consume một iterator, nhưng chúng ta đã bỏ qua cách chính xác mà nó hoạt động cho đến bây giờ.

Trong ví dụ trong Listing 13-11, chúng ta tách biệt việc tạo iterator từ việc sử dụng iterator trong vòng lặp `for`. Khi vòng lặp `for` được gọi sử dụng iterator trong `v1_iter`, mỗi phần tử trong iterator được sử dụng trong một lần lặp của vòng lặp, vốn in ra mỗi giá trị.

<Listing number="13-11" file-name="src/main.rs" caption="Sử dụng một iterator trong vòng lặp `for`">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-11/src/main.rs:here}}
```

</Listing>

Trong các ngôn ngữ không có iterator được cung cấp bởi thư viện chuẩn của chúng, bạn có thể sẽ viết chức năng tương tự này bằng cách bắt đầu một biến ở chỉ mục 0, sử dụng biến đó để lập chỉ mục vào vector để lấy một giá trị, và tăng giá trị biến trong một vòng lặp cho đến khi nó đạt tổng số mục trong vector.

Iterator xử lý tất cả logic đó cho bạn, giảm code lặp lại mà bạn có thể có khả năng mắc lỗi. Iterator cung cấp cho bạn tính linh hoạt hơn để sử dụng logic tương tự với nhiều loại chuỗi khác nhau, không chỉ các cấu trúc dữ liệu mà bạn có thể lập chỉ mục vào, như vector. Hãy xem cách iterator thực hiện điều đó.

### Trait `Iterator` và Phương thức `next`

Tất cả các iterator triển khai một trait có tên `Iterator` được định nghĩa trong thư viện chuẩn. Định nghĩa của trait trông như sau:

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations elided
}
```

Chú ý rằng định nghĩa này sử dụng một số cú pháp mới: `type Item` và `Self::Item`, đây là các định nghĩa một liên kết type với trait này. Chúng ta sẽ nói về các associated type sâu hơn trong Chương 20. Hiện tại, tất cả bạn cần biết là code này nói rằng triển khai trait `Iterator` yêu cầu bạn cũng phải định nghĩa một type `Item`, và type `Item` này được sử dụng trong kiểu trả về của phương thức `next`. Nói cách khác, type `Item` sẽ là type được trả về từ iterator.

Trait `Iterator` chỉ yêu cầu người triển khai định nghĩa một phương thức: phương thức `next`, phương thức này trả về một mục của iterator một lần, được bọc trong `Some`, và khi quá trình lặp kết thúc, trả về `None`.

Chúng ta có thể gọi phương thức `next` trực tiếp trên iterator; Listing 13-12 thể hiện các giá trị được trả về từ các lệnh gọi lặp lại tới `next` trên iterator được tạo từ vector.

<Listing number="13-12" file-name="src/lib.rs" caption="Gọi phương thức `next` trên một iterator">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-12/src/lib.rs:here}}
```

</Listing>

Lưu ý rằng chúng ta cần phải làm cho `v1_iter` có thể thay đổi: Gọi phương thức `next` trên một iterator thay đổi trạng thái nội bộ mà iterator sử dụng để theo dõi nó ở đâu trong chuỗi. Nói cách khác, code này _consume_ (tiêu thụ) hoặc dùng hết iterator. Mỗi lệnh gọi tới `next` ăn hết một mục từ iterator. Chúng ta không cần phải làm cho `v1_iter` có thể thay đổi khi chúng ta sử dụng vòng lặp `for`, bởi vì vòng lặp đã lấy ownership của `v1_iter` và làm nó có thể thay đổi phía sau.

Cũng lưu ý rằng các giá trị chúng ta nhận được từ các lệnh gọi tới `next` là các reference không thay đổi tới các giá trị trong vector. Phương thức `iter` tạo ra một iterator trên các reference không thay đổi. Nếu chúng ta muốn tạo một iterator mà lấy ownership của `v1` và trả về các giá trị được sở hữu, chúng ta có thể gọi `into_iter` thay vì `iter`. Tương tự, nếu chúng ta muốn lặp qua các reference có thể thay đổi, chúng ta có thể gọi `iter_mut` thay vì `iter`.

### Các Phương thức mà Consume Iterator

Trait `Iterator` có một số phương thức khác nhau với các triển khai mặc định được cung cấp bởi thư viện chuẩn; bạn có thể tìm hiểu về các phương thức này bằng cách xem tài liệu API của thư viện chuẩn cho trait `Iterator`. Một số phương thức này gọi phương thức `next` trong định nghĩa của chúng, đây là lý do tại sao bạn cần triển khai phương thức `next` khi triển khai trait `Iterator`.

Các phương thức gọi `next` được gọi là _consuming adapter_ (bộ adapter consume) bởi vì gọi chúng dùng hết iterator. Một ví dụ là phương thức `sum`, phương thức này lấy ownership của iterator và lặp qua các mục bằng cách gọi liên tục `next`, do đó consume iterator. Khi nó lặp qua, nó thêm mỗi mục vào một tổng chạy và trả về tổng khi quá trình lặp hoàn tất. Listing 13-13 có một bài kiểm tra minh họa một cách sử dụng phương thức `sum`.

<Listing number="13-13" file-name="src/lib.rs" caption="Gọi phương thức `sum` để lấy tổng của tất cả các mục trong iterator">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-13/src/lib.rs:here}}
```

</Listing>

Chúng ta không được phép sử dụng `v1_iter` sau lệnh gọi tới `sum`, bởi vì `sum` lấy ownership của iterator mà chúng ta gọi nó lên.

### Các Phương thức mà Tạo ra các Iterator khác

_Iterator adapter_ (bộ adapter iterator) là các phương thức được định nghĩa trên trait `Iterator` mà không consume iterator. Thay vào đó, chúng tạo ra các iterator khác nhau bằng cách thay đổi một số khía cạnh của iterator gốc.

Listing 13-14 cho thấy một ví dụ về gọi phương thức iterator adapter `map`, phương thức này lấy một closure để gọi trên mỗi mục khi các mục được lặp qua. Phương thức `map` trả về một iterator mới tạo ra các mục được sửa đổi. Closure ở đây tạo ra một iterator mới trong đó mỗi mục từ vector sẽ được tăng thêm 1.

<Listing number="13-14" file-name="src/main.rs" caption="Gọi iterator adapter `map` để tạo một iterator mới">

```rust,not_desired_behavior
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-14/src/main.rs:here}}
```

</Listing>

Tuy nhiên, code này tạo ra một cảnh báo:

```console
{{#include ../listings/ch13-functional-features/listing-13-14/output.txt}}
```

Code trong Listing 13-14 không làm bất cứ điều gì; closure mà chúng ta đã chỉ định không bao giờ được gọi. Cảnh báo nhắc nhở chúng ta tại sao: Iterator adapter lười biếng, và chúng ta cần consume iterator ở đây.

Để sửa cảnh báo này và consume iterator, chúng ta sẽ sử dụng phương thức `collect`, phương thức mà chúng ta đã sử dụng với `env::args` trong Listing 12-1. Phương thức này consume iterator và sưu tập các giá trị kết quả vào một kiểu dữ liệu collection.

Trong Listing 13-15, chúng ta sưu tập các kết quả của lặp qua iterator được trả về từ lệnh gọi tới `map` vào một vector. Vector này sẽ kết thúc chứa mỗi mục từ vector gốc, tăng thêm 1.

<Listing number="13-15" file-name="src/main.rs" caption="Gọi phương thức `map` để tạo một iterator mới, và sau đó gọi phương thức `collect` để consume iterator mới và tạo một vector">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-15/src/main.rs:here}}
```

</Listing>

Bởi vì `map` lấy một closure, chúng ta có thể chỉ định bất kỳ phương thức nào mà chúng ta muốn thực hiện trên mỗi mục. Đây là một ví dụ tuyệt vời về cách closure cho phép bạn tùy chỉnh một số hành vi trong khi tái sử dụng hành vi lặp mà trait `Iterator` cung cấp.

Bạn có thể chuỗi các lệnh gọi lặp lại tới iterator adapter để thực hiện các hành động phức tạp theo một cách dễ đọc. Nhưng bởi vì tất cả các iterator đều lười biếng, bạn phải gọi một trong những phương thức adapter consume để có được kết quả từ các lệnh gọi tới iterator adapter.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-closures-that-capture-their-environment"></a>

### Closure mà Capture Environment của Chúng

Nhiều iterator adapter lấy closure làm đối số, và thường những closure mà chúng ta sẽ chỉ định làm đối số cho iterator adapter sẽ là những closure mà capture environment của chúng.

Đối với ví dụ này, chúng ta sẽ sử dụng phương thức `filter` mà lấy một closure. Closure nhận một mục từ iterator và trả về một `bool`. Nếu closure trả về `true`, giá trị sẽ được bao gồm trong sự lặp được tạo ra bởi `filter`. Nếu closure trả về `false`, giá trị sẽ không được bao gồm.

Trong Listing 13-16, chúng ta sử dụng `filter` với một closure mà capture biến `shoe_size` từ environment của nó để lặp qua một collection của các instance của struct `Shoe`. Nó sẽ trả về chỉ những đôi giày có kích cỡ được chỉ định.

<Listing number="13-16" file-name="src/lib.rs" caption="Sử dụng phương thức `filter` với một closure mà capture `shoe_size`">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-16/src/lib.rs}}
```

</Listing>

Hàm `shoes_in_size` lấy ownership của một vector của các giày và một kích cỡ giày làm tham số. Nó trả về một vector chứa chỉ những đôi giày có kích cỡ được chỉ định.

Trong body của `shoes_in_size`, chúng ta gọi `into_iter` để tạo một iterator mà lấy ownership của vector. Sau đó, chúng ta gọi `filter` để điều chỉnh iterator đó thành một iterator mới chỉ chứa các phần tử mà closure trả về `true`.

Closure capture tham số `shoe_size` từ environment và so sánh giá trị với kích cỡ của mỗi giày, giữ lại chỉ những đôi giày có kích cỡ được chỉ định. Cuối cùng, gọi `collect` sưu tập các giá trị được trả về bởi iterator điều chỉnh vào một vector được trả về bởi hàm.

Bài kiểm tra cho thấy rằng khi chúng ta gọi `shoes_in_size`, chúng ta sẽ nhận lại chỉ những đôi giày có cùng kích cỡ với giá trị mà chúng ta đã chỉ định.
