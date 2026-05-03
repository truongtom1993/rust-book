<!-- Các tiêu đề cũ. Không được xóa nếu không các liên kết có thể bị hỏng. -->

<a id="traits-defining-shared-behavior"></a>

## Định nghĩa Hành vi Dùng Chung với Trait

Một _trait_ xác định tập chức năng mà một kiểu cụ thể có và có thể chia sẻ với
các kiểu khác. Chúng ta có thể sử dụng trait để định nghĩa hành vi dùng chung
theo cách trừu tượng. Chúng ta có thể sử dụng _ràng buộc trait_ (trait bound)
để chỉ ra rằng một kiểu tổng quát (generic type) có thể là bất kỳ kiểu nào có
một số hành vi nhất định.

> Lưu ý: Trait tương tự với một tính năng thường được gọi là _interface_ trong
> các ngôn ngữ khác, mặc dù vẫn có một số khác biệt.

### Định nghĩa một Trait

Hành vi của một kiểu bao gồm các phương thức mà chúng ta có thể gọi trên kiểu
đó. Các kiểu khác nhau chia sẻ cùng một hành vi nếu chúng ta có thể gọi cùng
một tập phương thức trên tất cả các kiểu đó. Việc định nghĩa trait là một cách
nhóm các chữ ký phương thức (method signature) lại với nhau để xác định một
bộ hành vi cần thiết nhằm đạt được một mục tiêu nào đó.

Ví dụ, giả sử chúng ta có nhiều struct lưu trữ các loại và lượng văn bản khác
nhau: một struct `NewsArticle` lưu trữ một bản tin được gửi từ một địa điểm cụ
thể và một `SocialPost` có thể chứa tối đa 280 ký tự cùng với metadata cho biết
đây là bài đăng mới, bài đăng lại (repost), hay là trả lời (reply) cho một bài
đăng khác.

Chúng ta muốn xây dựng một thư viện tổng hợp nội dung truyền thông (media
aggregator) có tên crate là `aggregator` có khả năng hiển thị các bản tóm tắt
dữ liệu có thể được lưu trong một thể hiện `NewsArticle` hoặc `SocialPost`.
Để làm được điều này, chúng ta cần một bản tóm tắt từ mỗi kiểu, và sẽ yêu cầu
bản tóm tắt đó bằng cách gọi phương thức `summarize` trên một thể hiện. Liệt
kê 10-12 cho thấy định nghĩa trait công khai (public) `Summary` biểu diễn hành
vi này.

<Listing number="10-12" file-name="src/lib.rs" caption="Trait `Summary` bao gồm hành vi được cung cấp bởi phương thức `summarize`">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-12/src/lib.rs}}
```

</Listing>

Ở đây, chúng ta khai báo một trait bằng từ khóa `trait` và sau đó là tên của
trait, trong trường hợp này là `Summary`. Chúng ta cũng khai báo trait này là
`pub` để các crate phụ thuộc vào crate này cũng có thể sử dụng trait đó, như
chúng ta sẽ thấy trong một vài ví dụ. Bên trong cặp ngoặc nhọn, chúng ta khai
báo các chữ ký phương thức mô tả các hành vi của những kiểu triển khai trait
này; trong trường hợp này là `fn summarize(&self) -> String`.

Sau chữ ký phương thức, thay vì cung cấp phần cài đặt trong ngoặc nhọn, chúng
ta dùng dấu chấm phẩy. Mỗi kiểu triển khai trait này bắt buộc phải tự cung
cấp phần cài đặt riêng cho thân (body) của phương thức. Trình biên dịch sẽ ép
buộc rằng mọi kiểu có trait `Summary` đều phải có phương thức `summarize` được
định nghĩa chính xác với chữ ký đó.

Một trait có thể có nhiều phương thức trong thân của nó: các chữ ký phương
thức được liệt kê mỗi dòng một chữ ký, và mỗi dòng kết thúc bằng một dấu chấm
phẩy.

### Triển khai Trait trên một Kiểu

Sau khi đã định nghĩa các chữ ký mong muốn của các phương thức trong trait
`Summary`, chúng ta có thể triển khai trait này trên các kiểu trong bộ tổng
hợp nội dung. Liệt kê 10-13 cho thấy phần triển khai trait `Summary` trên
struct `NewsArticle` sử dụng tiêu đề (headline), tác giả (author) và địa điểm
(location) để tạo giá trị trả về của `summarize`. Đối với struct `SocialPost`,
chúng ta định nghĩa `summarize` là tên người dùng (username) theo sau bởi toàn
bộ nội dung bài đăng, giả định rằng nội dung bài đăng đã bị giới hạn sẵn ở 280
ký tự.

<Listing number="10-13" file-name="src/lib.rs" caption="Triển khai trait `Summary` trên các kiểu `NewsArticle` và `SocialPost`">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-13/src/lib.rs:here}}
```

</Listing>

Triển khai một trait trên một kiểu tương tự như triển khai các phương thức
thông thường. Điểm khác biệt là sau `impl`, chúng ta đặt tên trait muốn triển
khai, sau đó dùng từ khóa `for`, rồi chỉ định tên kiểu mà chúng ta muốn triển
khai trait cho nó. Bên trong khối `impl`, chúng ta đặt các chữ ký phương thức
đã được định nghĩa trong trait. Thay vì thêm dấu chấm phẩy sau mỗi chữ ký,
chúng ta sử dụng ngoặc nhọn và điền phần thân phương thức với hành vi cụ thể
mà chúng ta muốn các phương thức của trait có đối với kiểu đó.

Bây giờ thư viện đã triển khai trait `Summary` trên `NewsArticle` và
`SocialPost`, người dùng của crate có thể gọi các phương thức của trait trên
các thể hiện `NewsArticle` và `SocialPost` theo cùng một cách như gọi các
phương thức thông thường. Điểm khác biệt duy nhất là người dùng phải đưa trait
vào phạm vi (scope) cùng với các kiểu. Dưới đây là một ví dụ về cách một
binary crate có thể sử dụng crate thư viện `aggregator` của chúng ta:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-01-calling-trait-method/src/main.rs}}
```

Đoạn mã này in ra `1 new post: horse_ebooks: of course, as you probably already
know, people`.

Các crate khác phụ thuộc vào crate `aggregator` cũng có thể đưa trait `Summary`
vào phạm vi để triển khai `Summary` trên các kiểu của riêng chúng. Một ràng
buộc cần lưu ý là chúng ta chỉ có thể triển khai một trait trên một kiểu nếu
hoặc trait, hoặc kiểu, hoặc cả hai,