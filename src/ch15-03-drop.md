## Chạy Code khi Cleanup với Trait `Drop`

Trait thứ hai quan trọng trong smart pointer pattern là `Drop`, cho phép bạn tùy chỉnh hành vi khi một giá trị sắp ra khỏi scope. Bạn có thể cung cấp implementation cho trait `Drop` trên bất kỳ type nào, và code đó có thể được dùng để giải phóng tài nguyên như files hoặc network connections.

Chúng tôi giới thiệu `Drop` trong bối cảnh smart pointers vì functionality của trait `Drop` hầu như luôn được dùng khi implement một smart pointer. Ví dụ, khi một `Box<T>` bị dropped, nó sẽ giải phóng không gian trên heap mà box trỏ tới.

Trong một số ngôn ngữ, đối với một số types, lập trình viên phải gọi code để giải phóng memory hoặc resources mỗi lần họ sử dụng xong một instance của những types đó. Ví dụ như file handles, sockets, và locks. Nếu lập trình viên quên, hệ thống có thể bị quá tải và crash. Trong Rust, bạn có thể chỉ định một đoạn code cụ thể chạy bất cứ khi nào một giá trị ra khỏi scope, và compiler sẽ tự động chèn code này. Kết quả là, bạn không cần phải cẩn thận đặt cleanup code ở mọi nơi trong chương trình mà instance của một type cụ thể được sử dụng xong—bạn vẫn sẽ không bị rò rỉ resources!

Bạn chỉ định code cần chạy khi một giá trị ra khỏi scope bằng cách implement trait `Drop`. Trait `Drop` yêu cầu bạn implement một method được đặt tên `drop` nhận một mutable reference tới `self`. Để xem khi nào Rust gọi `drop`, hãy implement `drop` với các `println!` statements để xem.

Danh sách 15-14 hiển thị một struct `CustomSmartPointer` với custom functionality duy nhất là nó sẽ in ra `Dropping CustomSmartPointer!` khi instance ra khỏi scope, để chỉ ra khi nào Rust chạy method `drop`.

<Listing number="15-14" file-name="src/main.rs" caption="Một struct `CustomSmartPointer` implement trait `Drop` nơi chúng ta sẽ đặt cleanup code">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-14/src/main.rs}}
```

</Listing>

Trait `Drop` được bao gồm trong prelude, vì vậy chúng ta không cần phải đưa nó vào scope. Chúng ta implement trait `Drop` trên `CustomSmartPointer` và cung cấp implementation cho method `drop` gọi `println!`. Phần body của method `drop` là nơi bạn sẽ đặt bất kỳ logic nào mà bạn muốn chạy khi một instance của type của bạn ra khỏi scope. Chúng ta in text ở đây để minh họa trực quan khi nào Rust sẽ gọi `drop`.

Trong `main`, chúng ta tạo hai instances của `CustomSmartPointer` và sau đó in ra `CustomSmartPointers created`. Ở cuối `main`, các instances của `CustomSmartPointer` sẽ ra khỏi scope, và Rust sẽ gọi code chúng ta đặt trong method `drop`, in ra final message của chúng ta. Lưu ý rằng chúng ta không cần phải gọi method `drop` một cách rõ ràng.

Khi chúng ta chạy chương trình này, chúng ta sẽ thấy output sau:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-14/output.txt}}
```

Rust tự động gọi `drop` cho chúng ta khi instances của chúng ta ra khỏi scope, gọi code mà chúng ta đã chỉ định. Variables được dropped theo thứ tự ngược lại của creation, vì vậy `d` đã được dropped trước `c`. Mục đích của ví dụ này là để cung cấp cho bạn một hướng dẫn trực quan về cách method `drop` hoạt động; thông thường bạn sẽ chỉ định cleanup code mà type của bạn cần chạy thay vì một message in ra.

<!-- Old headings. Do not remove or links may break. -->

<a id="dropping-a-value-early-with-std-mem-drop"></a>

Thật không may, không phải là đơn giản để vô hiệu hóa automatic `drop` functionality. Vô hiệu hóa `drop` thường không cần thiết; toàn bộ điểm của trait `Drop` là nó được xử lý tự động. Tuy nhiên, thỉnh thoảng, bạn có thể muốn clean up một giá trị sớm. Một ví dụ là khi dùng smart pointers quản lý locks: Bạn có thể muốn buộc method `drop` giải phóng lock để code khác trong cùng scope có thể acquire lock. Rust không cho phép bạn gọi method `drop` của trait `Drop` một cách thủ công; thay vào đó, bạn phải gọi function `std::mem::drop` được cung cấp bởi standard library nếu bạn muốn buộc một giá trị bị dropped trước khi kết thúc scope của nó.

Thử gọi method `drop` của trait `Drop` một cách thủ công bằng cách sửa đổi function `main` từ Danh sách 15-14 sẽ không hoạt động, như được hiển thị trong Danh sách 15-15.

<Listing number="15-15" file-name="src/main.rs" caption="Thử gọi method `drop` từ trait `Drop` một cách thủ công để clean up sớm">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-15/src/main.rs:here}}
```

</Listing>

Khi chúng ta thử compile code này, chúng ta sẽ nhận được error này:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-15/output.txt}}
```

Error message này nói rằng chúng ta không được phép gọi `drop` một cách rõ ràng. Error message sử dụng term _destructor_, là general programming term cho một function cleanup một instance. Một _destructor_ là tương tự với một _constructor_, cái tạo một instance. Function `drop` trong Rust là một destructor cụ thể.

Rust không cho phép chúng ta gọi `drop` một cách rõ ràng, vì Rust vẫn sẽ tự động gọi `drop` trên giá trị ở cuối `main`. Điều này sẽ gây ra lỗi double free vì Rust sẽ cố gắng clean up giá trị tương tự hai lần.

Chúng ta không thể vô hiệu hóa automatic insertion của `drop` khi một giá trị ra khỏi scope, và chúng ta không thể gọi method `drop` một cách rõ ràng. Vì vậy, nếu chúng ta cần buộc một giá trị được clean up sớm, chúng ta dùng function `std::mem::drop`.

Function `std::mem::drop` khác với method `drop` trong trait `Drop`. Chúng ta gọi nó bằng cách truyền vào làm argument giá trị mà chúng ta muốn force-drop. Function này ở trong prelude, vì vậy chúng ta có thể sửa đổi `main` trong Danh sách 15-15 để gọi function `drop`, như được hiển thị trong Danh sách 15-16.

<Listing number="15-16" file-name="src/main.rs" caption="Gọi `std::mem::drop` để explicitly drop một giá trị trước khi nó ra khỏi scope">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-16/src/main.rs:here}}
```

</Listing>

Chạy code này sẽ in ra:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-16/output.txt}}
```

Text ``Dropping CustomSmartPointer with data `some data`!`` được in giữa text `CustomSmartPointer created` và `CustomSmartPointer dropped before the end of main`, showing rằng code của method `drop` được gọi để drop `c` tại điểm đó.

Bạn có thể dùng code được chỉ định trong implementation của trait `Drop` theo nhiều cách để làm cleanup tiện lợi và an toàn: Ví dụ, bạn có thể dùng nó để tạo memory allocator của riêng bạn! Với trait `Drop` và ownership system của Rust, bạn không cần phải nhớ clean up, vì Rust làm nó tự động.

Bạn cũng không phải lo lắng về các vấn đề kết quả từ việc vô tình clean up các giá trị vẫn đang được dùng: Ownership system mà đảm bảo references luôn hợp lệ cũng đảm bảo rằng `drop` được gọi chỉ một lần khi giá trị không còn được dùng.

Bây giờ chúng ta đã kiểm tra `Box<T>` và một số đặc điểm của smart pointers, hãy nhìn vào một vài smart pointers khác được định nghĩa trong standard library.