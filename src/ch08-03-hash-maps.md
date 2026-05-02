## Lưu Trữ Khóa với Các Giá Trị Liên Kết trong Hash Maps

Cái cuối cùng của các common collections của chúng ta là hash map. Loại `HashMap<K, V>`
lưu trữ một mapping của các khóa loại `K` đến các giá trị loại `V` bằng cách sử dụng một _hashing
function_, xác định cách nó đặt các khóa và giá trị này vào bộ nhớ.
Nhiều ngôn ngữ lập trình hỗ trợ loại cấu trúc dữ liệu này, nhưng chúng thường
sử dụng một tên khác, chẳng hạn như _hash_, _map_, _object_, _hash table_,
_dictionary_, hoặc _associative array_, chỉ để kể tên một vài.

Hash maps rất hữu ích khi bạn muốn tìm kiếm dữ liệu không bằng cách sử dụng index, như
bạn có thể với vectors, mà bằng cách sử dụng khóa có thể là loại bất kỳ. Ví dụ,
trong một trò chơi, bạn có thể theo dõi điểm số của mỗi đội trong hash map trong đó
mỗi khóa là tên của đội và các giá trị là điểm số của mỗi đội. Cho một tên đội,
bạn có thể truy cập điểm số của nó.

Chúng ta sẽ xem qua API cơ bản của hash maps trong phần này, nhưng nhiều hơn nữa
những điều tốt đẹp đang ẩn trong các hàm được định nghĩa trên `HashMap<K, V>` bởi thư viện chuẩn.
Như mọi khi, hãy kiểm tra tài liệu thư viện chuẩn để tìm hiểu thêm.

### Tạo một Hash Map Mới

Một cách để tạo hash map rỗng là sử dụng `new` và thêm các phần tử với
`insert`. Trong Listing 8-20, chúng ta đang theo dõi điểm số của hai đội có
tên là _Blue_ và _Yellow_. Đội Blue bắt đầu với 10 điểm, và
đội Yellow bắt đầu với 50.

<Listing number="8-20" caption="Tạo hash map mới và chèn một số khóa và giá trị">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-20/src/main.rs:here}}
```

</Listing>

Lưu ý rằng chúng ta cần trước tiên `use` `HashMap` từ phần collections của
thư viện chuẩn. Trong ba common collections của chúng ta, cái này được
sử dụng ít nhất, vì vậy nó không được đưa vào các features được đưa vào scope
một cách tự động trong prelude. Hash maps cũng có ít support hơn từ
thư viện chuẩn; không có built-in macro để xây dựng chúng, ví dụ.

Giống như vectors, hash maps lưu trữ dữ liệu của chúng trên heap. `HashMap` này có
khóa của loại `String` và giá trị của loại `i32`. Giống như vectors, hash maps là
homogeneous: Tất cả các khóa phải có cùng loại, và tất cả các
giá trị phải có cùng loại.

### Truy Cập Các Giá Trị trong Hash Map

Chúng ta có thể lấy giá trị từ hash map bằng cách cung cấp khóa của nó cho method
`get`, như được hiển thị trong Listing 8-21.

<Listing number="8-21" caption="Truy cập điểm số cho đội Blue được lưu trữ trong hash map">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-21/src/main.rs:here}}
```

</Listing>

Ở đây, `score` sẽ có giá trị liên kết với đội Blue, và
kết quả sẽ là `10`. Method `get` trả về `Option<&V>`; nếu không có
giá trị cho khóa đó trong hash map, `get` sẽ trả về `None`. Chương trình này
xử lý `Option` bằng cách gọi `copied` để nhận `Option<i32>` thay vì
`Option<&i32>`, sau đó `unwrap_or` để đặt `score` thành 0 nếu `scores` không
có entry cho khóa.

Chúng ta có thể lặp qua từng cặp key-value trong hash map theo cách tương tự như chúng ta
làm với vectors, sử dụng vòng lặp `for`:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-03-iterate-over-hashmap/src/main.rs:here}}
```

Code này sẽ in mỗi cặp theo một thứ tự tùy ý:

```text
Yellow: 50
Blue: 10
```

<!-- Old headings. Do not remove or links may break. -->

<a id=”hash-maps-and-ownership”></a>

### Quản Lý Ownership trong Hash Maps

Đối với các loại triển khai trait `Copy`, như `i32`, các giá trị được sao chép
vào hash map. Đối với các owned values như `String`, các giá trị sẽ được moved và
hash map sẽ là chủ sở hữu của những giá trị đó, như được minh họa trong Listing 8-22.

<Listing number=”8-22” caption=”Hiển thị rằng các khóa và giá trị được sở hữu bởi hash map một khi chúng được chèn”>

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-22/src/main.rs:here}}
```

</Listing>

Chúng ta không thể sử dụng các biến `field_name` và `field_value` sau
khi chúng được moved vào hash map bằng cuộc gọi `insert`.

Nếu chúng ta chèn các tham chiếu đến các giá trị vào hash map, các giá trị sẽ không được moved
vào hash map. Các giá trị mà các tham chiếu trỏ đến phải hợp lệ ít nhất
miễn là hash map hợp lệ. Chúng ta sẽ nói nhiều hơn về những vấn đề này trong
[“Validating References with
Lifetimes”][validating-references-with-lifetimes]<!-- ignore --> trong Chapter 10.

### Cập Nhật Hash Map

Mặc dù số lượng cặp key-value có thể phát triển, mỗi khóa duy nhất chỉ có thể
có một giá trị liên kết với nó tại một thời điểm (nhưng không phải ngược lại: Ví dụ,
cả đội Blue và đội Yellow đều có thể có giá trị `10`
được lưu trữ trong hash map `scores`).

Khi bạn muốn thay đổi dữ liệu trong hash map, bạn phải quyết định cách
xử lý trường hợp khi khóa đã có giá trị được gán. Bạn có thể thay thế
giá trị cũ bằng giá trị mới, hoàn toàn bỏ qua giá trị cũ. Bạn có thể
giữ giá trị cũ và bỏ qua giá trị mới, chỉ thêm giá trị mới nếu
khóa _không_ đã có giá trị. Hoặc bạn có thể kết hợp giá trị cũ và
giá trị mới. Hãy xem cách thực hiện từng cách!

#### Ghi Đè Giá Trị

Nếu chúng ta chèn khóa và giá trị vào hash map và sau đó chèn khóa đó
với giá trị khác, giá trị liên kết với khóa đó sẽ được thay thế.
Mặc dù code trong Listing 8-23 gọi `insert` hai lần, hash map sẽ
chỉ chứa một cặp key-value vì chúng ta đang chèn giá trị cho khóa
của đội Blue cả hai lần.

<Listing number="8-23" caption="Thay thế giá trị được lưu trữ với khóa cụ thể">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-23/src/main.rs:here}}
```

</Listing>

Code này sẽ in `{"Blue": 25}`. Giá trị gốc `10` đã
được ghi đè.

<!-- Old headings. Do not remove or links may break. -->

<a id="only-inserting-a-value-if-the-key-has-no-value"></a>

#### Thêm Khóa và Giá Trị Chỉ Nếu Khóa Không Có

Thông thường kiểm tra xem khóa cụ thể đã tồn tại trong hash map
với giá trị chưa và sau đó thực hiện các hành động sau: Nếu khóa tồn tại trong
hash map, giá trị hiện có nên vẫn như cũ; nếu khóa
không tồn tại, hãy chèn nó và một giá trị cho nó.

Hash maps có một API đặc biệt cho điều này được gọi là `entry` lấy khóa bạn
muốn kiểm tra làm parameter. Giá trị trả về của method `entry` là enum
gọi là `Entry` đại diện cho giá trị có thể hoặc không tồn tại. Giả sử
chúng ta muốn kiểm tra xem khóa cho đội Yellow có giá trị được liên kết
với nó không. Nếu không, chúng ta muốn chèn giá trị `50`, và tương tự cho
đội Blue. Sử dụng `entry` API, code trông giống như Listing 8-24.

<Listing number="8-24" caption="Sử dụng method `entry` để chỉ chèn nếu khóa không đã có giá trị">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-24/src/main.rs:here}}
```

</Listing>

Method `or_insert` trên `Entry` được định nghĩa để trả về mutable reference đến
giá trị cho khóa `Entry` tương ứng nếu khóa đó tồn tại, và nếu không, nó
chèn parameter làm giá trị mới cho khóa này và trả về mutable
reference đến giá trị mới. Kỹ thuật này sạch sẽ hơn nhiều so với viết
logic của chúng ta và, ngoài ra, hoạt động tốt hơn với borrow checker.

Chạy code trong Listing 8-24 sẽ in `{"Yellow": 50, "Blue": 10}`. Cuộc gọi
đầu tiên đến `entry` sẽ chèn khóa cho đội Yellow với giá trị
`50` vì đội Yellow không có giá trị sẵn. Cuộc gọi thứ hai đến
`entry` sẽ không thay đổi hash map, vì đội Blue đã có
giá trị `10`.

#### Cập Nhật Giá Trị Dựa trên Giá Trị Cũ

Một trường hợp sử dụng phổ biến khác của hash maps là tìm kiếm giá trị của khóa và sau đó
cập nhật nó dựa trên giá trị cũ. Ví dụ, Listing 8-25 hiển thị code
đếm bao nhiêu lần mỗi từ xuất hiện trong một số text. Chúng ta sử dụng hash map với
các từ làm khóa và tăng giá trị để theo dõi bao nhiêu lần chúng ta đã
nhìn thấy từ đó. Nếu đây là lần đầu tiên chúng ta nhìn thấy từ, chúng ta sẽ trước tiên chèn
giá trị `0`.

<Listing number=”8-25” caption=”Đếm sự xuất hiện của các từ bằng hash map lưu trữ các từ và số lượng”>

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-25/src/main.rs:here}}
```

</Listing>

Code này sẽ in `{“world”: 2, “hello”: 1, “wonderful”: 1}`. Bạn có thể thấy
các cặp key-value giống nhau được in theo thứ tự khác: Nhớ lại từ [“Accessing
Values in a Hash Map”][access]<!-- ignore --> rằng lặp qua hash map
xảy ra theo thứ tự tùy ý.

Method `split_whitespace` trả về iterator trên các subslices, được phân tách bởi
whitespace, của giá trị trong `text`. Method `or_insert` trả về mutable
reference (`&mut V`) đến giá trị cho khóa được chỉ định. Ở đây, chúng ta lưu trữ
mutable reference đó trong biến `count`, vì vậy để gán cho giá trị đó,
chúng ta phải trước tiên dereference `count` bằng cách sử dụng dấu hoa thị (`*`). Mutable
reference ra khỏi scope ở cuối vòng lặp `for`, vì vậy tất cả những
thay đổi này là an toàn và được phép bởi borrowing rules.

### Hashing Functions

Theo mặc định, `HashMap` sử dụng hashing function gọi là _SipHash_ có thể cung cấp
kháng lại denial-of-service (DoS) attacks liên quan đến hash
tables[^siphash]<!-- ignore -->. Đây không phải là hashing algorithm nhanh nhất
có sẵn, nhưng trade-off để có security tốt hơn đi kèm với sự giảm
hiệu năng là đáng giá. Nếu bạn profile code của mình và thấy rằng
hashing function mặc định quá chậm cho mục đích của bạn, bạn có thể chuyển sang function khác
bằng cách chỉ định hasher khác. _Hasher_ là loại triển khai
trait `BuildHasher`. Chúng ta sẽ nói về traits và cách triển khai chúng trong
[Chapter 10][traits]<!-- ignore -->. Bạn không nhất thiết phải triển khai
hasher của riêng bạn từ đầu; [crates.io](https://crates.io/)<!-- ignore -->
có libraries được chia sẻ bởi các Rust users khác cung cấp các hashers triển khai nhiều
common hashing algorithms.

[^siphash]: [https://en.wikipedia.org/wiki/SipHash](https://en.wikipedia.org/wiki/SipHash)

## Tóm Tắt

Vectors, strings, và hash maps sẽ cung cấp lượng lớn functionality
cần thiết trong các chương trình khi bạn cần lưu trữ, truy cập và sửa đổi dữ liệu. Đây là
một số bài tập bạn nên được trang bị để giải quyết:

1. Cho danh sách các số nguyên, sử dụng vector và trả về median (khi được sắp xếp,
   giá trị ở vị trí giữa) và mode (giá trị xảy ra thường xuyên nhất;
   hash map sẽ hữu ích ở đây) của danh sách.
1. Chuyển đổi strings thành Pig Latin. Phụ âm đầu tiên của mỗi từ được di chuyển
   vào cuối của từ và _ay_ được thêm vào, vì vậy _first_ trở thành _irst-fay_. Các từ
   bắt đầu bằng nguyên âm có _hay_ được thêm vào cuối thay vào đó (_apple_ trở thành
   _apple-hay_). Hãy nhớ về các chi tiết về UTF-8 encoding!
1. Sử dụng hash map và vectors, tạo text interface để cho phép người dùng thêm
   tên nhân viên vào bộ phận trong công ty; ví dụ, “Add Sally to
   Engineering” hoặc “Add Amir to Sales.” Sau đó, hãy cho phép người dùng truy cập danh sách
   tất cả mọi người trong bộ phận hoặc tất cả mọi người trong công ty theo bộ phận, được sắp xếp
   theo bảng chữ cái.

Tài liệu API thư viện chuẩn mô tả các methods mà vectors, strings,
và hash maps có sẽ hữu ích cho những bài tập này!

Chúng ta đang bước vào các chương trình phức tạp hơn trong đó các hoạt động có thể thất bại, vì vậy đó là
lúc hoàn hảo để thảo luận về error handling. Chúng ta sẽ làm điều đó tiếp theo!

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[access]: #accessing-values-in-a-hash-map
[traits]: ch10-02-traits.html
