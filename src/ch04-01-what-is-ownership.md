## Ownership là gì?

_Ownership_ là một tập hợp các quy tắc chi phối cách chương trình Rust quản lý bộ nhớ. Mọi chương trình đều phải quản lý cách sử dụng bộ nhớ của máy tính trong khi chạy. Một số ngôn ngữ có garbage collection tự động tìm kiếm và giải phóng bộ nhớ không còn được dùng trong khi chương trình chạy; ở các ngôn ngữ khác, lập trình viên phải tự tay cấp phát và giải phóng bộ nhớ. Rust sử dụng cách tiếp cận thứ ba: bộ nhớ được quản lý thông qua hệ thống ownership với một tập hợp các quy tắc mà compiler kiểm tra. Nếu bất kỳ quy tắc nào bị vi phạm, chương trình sẽ không compile được. Không có tính năng nào của ownership làm chậm chương trình trong khi nó đang chạy.

Vì ownership là một khái niệm mới với nhiều lập trình viên, nên cần một chút thời gian để làm quen. Tin tốt là càng có nhiều kinh nghiệm với Rust và các quy tắc của hệ thống ownership, bạn sẽ càng dễ dàng viết code an toàn và hiệu quả một cách tự nhiên. Hãy cố gắng lên!

Khi bạn hiểu ownership, bạn sẽ có nền tảng vững chắc để hiểu các tính năng làm cho Rust trở nên độc đáo. Trong chương này, bạn sẽ học ownership bằng cách làm việc qua một số ví dụ tập trung vào một cấu trúc dữ liệu rất phổ biến: strings.

> ### Stack và Heap
>
> Nhiều ngôn ngữ lập trình không yêu cầu bạn phải nghĩ nhiều về stack và
> heap. Nhưng trong ngôn ngữ lập trình hệ thống như Rust, việc một giá trị
> nằm trên stack hay heap ảnh hưởng đến cách ngôn ngữ hoạt động và lý do tại
> sao bạn phải đưa ra các quyết định nhất định. Các phần của ownership sẽ
> được mô tả liên quan đến stack và heap ở phần sau của chương này, vì vậy
> đây là một giải thích ngắn gọn để chuẩn bị.
>
> Cả stack và heap đều là các phần bộ nhớ mà code của bạn có thể sử dụng
> trong runtime, nhưng chúng được cấu trúc theo các cách khác nhau. Stack
> lưu trữ các giá trị theo thứ tự nhận được và xóa chúng theo thứ tự ngược
> lại. Điều này được gọi là _last in, first out (LIFO)_. Hãy nghĩ đến một
> chồng đĩa: Khi bạn thêm đĩa, bạn đặt chúng lên trên cùng, và khi bạn cần
> một chiếc đĩa, bạn lấy từ trên xuống. Thêm hoặc lấy đĩa từ giữa hoặc dưới
> cùng sẽ không hiệu quả! Thêm dữ liệu vào được gọi là _pushing onto the
> stack_, và xóa dữ liệu được gọi là _popping off the stack_. Tất cả dữ liệu
> lưu trữ trên stack phải có kích thước đã biết và cố định. Dữ liệu có kích
> thước không xác định tại compile time hoặc kích thước có thể thay đổi phải
> được lưu trữ trên heap thay thế.
>
> Heap ít tổ chức hơn: Khi bạn đặt dữ liệu lên heap, bạn yêu cầu một lượng
> không gian nhất định. Memory allocator tìm một chỗ trống trong heap đủ lớn,
> đánh dấu nó là đang được sử dụng, và trả về một _pointer_, là địa chỉ của
> vị trí đó. Quá trình này được gọi là _allocating on the heap_ và đôi khi
> được viết tắt là _allocating_ (pushing giá trị vào stack không được coi là
> allocating). Vì pointer đến heap có kích thước đã biết và cố định, bạn có
> thể lưu trữ pointer trên stack, nhưng khi bạn muốn dữ liệu thực sự, bạn
> phải đi theo pointer. Hãy nghĩ đến việc được ngồi tại một nhà hàng. Khi
> bạn vào, bạn nói số người trong nhóm, và người phục vụ tìm một bàn trống
> vừa đủ cho mọi người và dẫn bạn đến đó. Nếu ai đó trong nhóm đến muộn,
> họ có thể hỏi bạn đang ngồi ở đâu để tìm bạn.
>
> Pushing vào stack nhanh hơn allocating trên heap vì allocator không bao giờ
> phải tìm kiếm chỗ để lưu dữ liệu mới; vị trí đó luôn ở đầu stack. So với
> vậy, việc cấp phát không gian trên heap đòi hỏi nhiều công việc hơn vì
> allocator trước tiên phải tìm đủ không gian để chứa dữ liệu và sau đó thực
> hiện bookkeeping để chuẩn bị cho lần cấp phát tiếp theo.
>
> Truy cập dữ liệu trong heap thường chậm hơn truy cập dữ liệu trong stack vì
> bạn phải đi theo pointer để đến đó. Các bộ xử lý hiện đại hoạt động nhanh
> hơn nếu chúng di chuyển ít hơn trong bộ nhớ. Tiếp tục ví dụ, hãy nghĩ về
> một server tại nhà hàng nhận đơn từ nhiều bàn. Hiệu quả nhất là lấy hết
> đơn từ một bàn trước khi chuyển sang bàn tiếp theo. Lấy đơn từ bàn A, sau
> đó từ bàn B, rồi lại từ A, rồi lại từ B sẽ chậm hơn nhiều. Tương tự, một
> bộ xử lý thường làm tốt hơn nếu nó làm việc với dữ liệu gần với dữ liệu
> khác (như trong stack) thay vì xa hơn (như trong heap).
>
> Khi code của bạn gọi một function, các giá trị được truyền vào function
> (bao gồm, có thể là, các pointer đến dữ liệu trên heap) và các biến local
> của function được pushed vào stack. Khi function kết thúc, những giá trị
> đó được popped off stack.
>
> Theo dõi phần nào của code đang sử dụng dữ liệu nào trên heap, giảm thiểu
> lượng dữ liệu trùng lặp trên heap, và dọn dẹp dữ liệu không sử dụng trên
> heap để bạn không hết không gian đều là các vấn đề mà ownership giải quyết.
> Khi bạn hiểu ownership, bạn sẽ không cần phải nghĩ nhiều về stack và heap.
> Nhưng biết rằng mục đích chính của ownership là quản lý dữ liệu heap có thể
> giúp giải thích tại sao nó hoạt động theo cách đó.

### Các quy tắc Ownership

Trước tiên, hãy xem xét các quy tắc ownership. Hãy ghi nhớ các quy tắc này khi chúng ta làm việc qua các ví dụ minh họa chúng:

- Mỗi giá trị trong Rust có một _owner_.
- Chỉ có thể có một owner tại một thời điểm.
- Khi owner ra khỏi scope, giá trị sẽ bị dropped.

### Variable Scope

Bây giờ chúng ta đã qua cú pháp Rust cơ bản, chúng ta sẽ không bao gồm toàn bộ code `fn main() {` trong các ví dụ, vì vậy nếu bạn đang theo dõi, hãy nhớ đặt các ví dụ sau vào bên trong một function `main` thủ công. Kết quả là, các ví dụ của chúng ta sẽ ngắn gọn hơn, cho phép chúng ta tập trung vào các chi tiết thực tế thay vì boilerplate code.

Là ví dụ đầu tiên về ownership, chúng ta sẽ xem xét scope của một số biến. Một _scope_ là phạm vi trong một chương trình mà một item có hiệu lực. Lấy biến sau:

```rust
let s = "hello";
```

Biến `s` tham chiếu đến một string literal, trong đó giá trị của string được hardcode vào text của chương trình. Biến có hiệu lực từ điểm mà nó được khai báo cho đến cuối scope hiện tại. Listing 4-1 cho thấy một chương trình với các comment ghi chú nơi biến `s` sẽ có hiệu lực.

<Listing number="4-1" caption="Một biến và scope mà nó có hiệu lực">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-01/src/main.rs:here}}
```

</Listing>

Nói cách khác, có hai điểm quan trọng trong thời gian ở đây:

- Khi `s` _đi vào_ scope, nó có hiệu lực.
- Nó vẫn có hiệu lực cho đến khi nó _ra khỏi_ scope.

Tại thời điểm này, mối quan hệ giữa các scope và thời điểm biến có hiệu lực tương tự như trong các ngôn ngữ lập trình khác. Bây giờ chúng ta sẽ xây dựng trên sự hiểu biết này bằng cách giới thiệu kiểu `String`.

### Kiểu `String`

Để minh họa các quy tắc ownership, chúng ta cần một kiểu dữ liệu phức tạp hơn những gì chúng ta đã đề cập trong phần ["Data Types"][data-types]<!-- ignore --> của Chương 3. Các kiểu được đề cập trước đây có kích thước đã biết, có thể được lưu trữ trên stack và popped off stack khi scope của chúng kết thúc, và có thể được sao chép nhanh chóng để tạo một instance mới, độc lập nếu một phần khác của code cần sử dụng cùng giá trị trong một scope khác. Nhưng chúng ta muốn xem xét dữ liệu được lưu trữ trên heap và khám phá cách Rust biết khi nào để dọn dẹp dữ liệu đó, và kiểu `String` là một ví dụ tuyệt vời.

Chúng ta sẽ tập trung vào các phần của `String` liên quan đến ownership. Những khía cạnh này cũng áp dụng cho các kiểu dữ liệu phức tạp khác, dù chúng được cung cấp bởi thư viện chuẩn hay do bạn tạo ra. Chúng ta sẽ thảo luận về các khía cạnh không liên quan đến ownership của `String` trong [Chương 8][ch8]<!-- ignore -->.

Chúng ta đã thấy string literals, nơi một giá trị string được hardcode vào chương trình. String literals tiện lợi, nhưng chúng không phù hợp với mọi tình huống mà chúng ta có thể muốn sử dụng text. Một lý do là chúng immutable. Lý do khác là không phải mọi giá trị string đều có thể được biết khi chúng ta viết code: Ví dụ, nếu chúng ta muốn lấy input từ người dùng và lưu nó thì sao? Đó là lý do tại sao Rust có kiểu `String`. Kiểu này quản lý dữ liệu được cấp phát trên heap và vì vậy có thể lưu trữ một lượng text không xác định với chúng ta tại compile time. Bạn có thể tạo một `String` từ một string literal bằng cách sử dụng function `from`, như sau:

```rust
let s = String::from("hello");
```

Toán tử `::` cho phép chúng ta đặt function `from` cụ thể này dưới namespace của kiểu `String` thay vì sử dụng một tên nào đó như `string_from`. Chúng ta sẽ thảo luận thêm về cú pháp này trong phần ["Methods"][methods]<!-- ignore --> của Chương 5, và khi chúng ta nói về namespacing với modules trong ["Paths for Referring to an Item in the Module Tree"][paths-module-tree]<!-- ignore --> trong Chương 7.

Loại string này _có thể_ được mutate:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-01-can-mutate-string/src/main.rs:here}}
```

Vậy, sự khác biệt ở đây là gì? Tại sao `String` có thể được mutate nhưng literals thì không? Sự khác biệt nằm ở cách hai kiểu này xử lý bộ nhớ.

### Bộ nhớ và Allocation

Trong trường hợp của một string literal, chúng ta biết nội dung tại compile time, vì vậy text được hardcode trực tiếp vào file thực thi cuối cùng. Đây là lý do tại sao string literals nhanh và hiệu quả. Nhưng những thuộc tính này chỉ đến từ tính immutable của string literal. Thật không may, chúng ta không thể đặt một blob bộ nhớ vào binary cho mỗi đoạn text có kích thước không biết tại compile time và kích thước có thể thay đổi trong khi chạy chương trình.

Với kiểu `String`, để hỗ trợ một đoạn text có thể mutate và mở rộng, chúng ta cần cấp phát một lượng bộ nhớ trên heap, không biết tại compile time, để chứa nội dung. Điều này có nghĩa là:

- Bộ nhớ phải được yêu cầu từ memory allocator tại runtime.
- Chúng ta cần một cách trả lại bộ nhớ này cho allocator khi chúng ta xong với `String`.

Phần đầu tiên được thực hiện bởi chúng ta: Khi chúng ta gọi `String::from`, implementation của nó yêu cầu bộ nhớ cần thiết. Điều này khá phổ biến trong các ngôn ngữ lập trình.

Tuy nhiên, phần thứ hai khác nhau. Trong các ngôn ngữ có _garbage collector (GC)_, GC theo dõi và dọn dẹp bộ nhớ không còn được sử dụng, và chúng ta không cần phải nghĩ về điều đó. Trong hầu hết các ngôn ngữ không có GC, trách nhiệm của chúng ta là xác định khi nào bộ nhớ không còn được sử dụng và gọi code để giải phóng nó một cách tường minh, giống như chúng ta đã làm để yêu cầu nó. Làm điều này đúng cách đã là một vấn đề lập trình khó khăn trong lịch sử. Nếu chúng ta quên, chúng ta sẽ lãng phí bộ nhớ. Nếu chúng ta làm quá sớm, chúng ta sẽ có một biến không hợp lệ. Nếu chúng ta làm hai lần, đó cũng là một bug. Chúng ta cần ghép chính xác một `allocate` với chính xác một `free`.

Rust đi theo một con đường khác: Bộ nhớ được tự động trả lại sau khi biến sở hữu nó ra khỏi scope. Đây là một phiên bản của ví dụ scope từ Listing 4-1 sử dụng `String` thay vì string literal:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-02-string-scope/src/main.rs:here}}
```

Có một điểm tự nhiên mà chúng ta có thể trả lại bộ nhớ mà `String` cần cho allocator: khi `s` ra khỏi scope. Khi một biến ra khỏi scope, Rust gọi một function đặc biệt cho chúng ta. Function này được gọi là `drop`, và đây là nơi tác giả của `String` có thể đặt code để trả lại bộ nhớ. Rust gọi `drop` tự động ở dấu ngoặc nhọn đóng.

> Lưu ý: Trong C++, pattern giải phóng tài nguyên ở cuối vòng đời của một
> item đôi khi được gọi là _Resource Acquisition Is Initialization (RAII)_.
> Function `drop` trong Rust sẽ quen thuộc với bạn nếu bạn đã sử dụng RAII
> patterns.

Pattern này có ảnh hưởng sâu sắc đến cách Rust code được viết. Nó có vẻ đơn giản lúc này, nhưng hành vi của code có thể bất ngờ trong các tình huống phức tạp hơn khi chúng ta muốn có nhiều biến sử dụng dữ liệu chúng ta đã cấp phát trên heap. Hãy khám phá một số tình huống đó ngay bây giờ.

<!-- Old headings. Do not remove or links may break. -->

<a id="ways-variables-and-data-interact-move"></a>

#### Các biến và dữ liệu tương tác với Move

Nhiều biến có thể tương tác với cùng một dữ liệu theo các cách khác nhau trong Rust. Listing 4-2 cho thấy một ví dụ sử dụng một integer.

<Listing number="4-2" caption="Gán giá trị integer của biến `x` cho `y`">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-02/src/main.rs:here}}
```

</Listing>

Chúng ta có thể đoán được điều này đang làm gì: "Bind giá trị `5` cho `x`; sau đó, tạo một bản sao của giá trị trong `x` và bind nó cho `y`." Chúng ta bây giờ có hai biến, `x` và `y`, và cả hai đều bằng `5`. Đây thực sự là điều đang xảy ra, vì integers là các giá trị đơn giản với kích thước đã biết và cố định, và hai giá trị `5` này được pushed vào stack.

Bây giờ hãy xem phiên bản `String`:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-03-string-move/src/main.rs:here}}
```

Điều này trông rất giống nhau, vì vậy chúng ta có thể giả định rằng cách nó hoạt động sẽ giống nhau: Nghĩa là, dòng thứ hai sẽ tạo một bản sao của giá trị trong `s1` và bind nó cho `s2`. Nhưng đây không hoàn toàn là điều xảy ra.

Hãy xem Hình 4-1 để thấy điều gì đang xảy ra với `String` bên trong. Một `String` được tạo thành từ ba phần, được hiển thị ở bên trái: một pointer đến bộ nhớ chứa nội dung của string, một độ dài, và một capacity. Nhóm dữ liệu này được lưu trữ trên stack. Bên phải là bộ nhớ trên heap chứa nội dung.

<img alt="Hai bảng: bảng đầu tiên chứa biểu diễn của s1 trên
stack, bao gồm độ dài (5), capacity (5), và một pointer đến giá trị đầu tiên
trong bảng thứ hai. Bảng thứ hai chứa biểu diễn của dữ liệu string trên heap,
byte theo byte." src="img/trpl04-01.svg" class="center"
style="width: 50%;" />

<span class="caption">Hình 4-1: Biểu diễn trong bộ nhớ của một `String`
chứa giá trị `"hello"` được bind cho `s1`</span>

Độ dài là lượng bộ nhớ, tính bằng bytes, mà nội dung của `String` đang sử dụng hiện tại. Capacity là tổng lượng bộ nhớ, tính bằng bytes, mà `String` đã nhận được từ allocator. Sự khác biệt giữa độ dài và capacity quan trọng, nhưng không trong ngữ cảnh này, vì vậy bây giờ, không cần quan tâm đến capacity.

Khi chúng ta gán `s1` cho `s2`, dữ liệu `String` được sao chép, nghĩa là chúng ta sao chép pointer, độ dài, và capacity nằm trên stack. Chúng ta không sao chép dữ liệu trên heap mà pointer tham chiếu đến. Nói cách khác, biểu diễn dữ liệu trong bộ nhớ trông như Hình 4-2.

<img alt="Ba bảng: bảng s1 và s2 biểu diễn các string đó trên
stack, tương ứng, và cả hai đều trỏ đến cùng dữ liệu string trên heap."
src="img/trpl04-02.svg" class="center" style="width: 50%;" />

<span class="caption">Hình 4-2: Biểu diễn trong bộ nhớ của biến
`s2` có bản sao của pointer, độ dài và capacity của `s1`</span>

Biểu diễn _không_ trông như Hình 4-3, đó là hình dạng của bộ nhớ nếu Rust thay vào đó cũng sao chép dữ liệu heap. Nếu Rust làm điều này, hoạt động `s2 = s1` có thể rất tốn kém về hiệu suất runtime nếu dữ liệu trên heap lớn.

<img alt="Bốn bảng: hai bảng biểu diễn dữ liệu stack cho s1 và s2,
và mỗi cái trỏ đến bản sao dữ liệu string riêng của mình trên heap."
src="img/trpl04-03.svg" class="center" style="width: 50%;" />

<span class="caption">Hình 4-3: Khả năng khác cho điều `s2 = s1` có thể
làm nếu Rust cũng sao chép dữ liệu heap</span>

Trước đó, chúng ta đã nói rằng khi một biến ra khỏi scope, Rust tự động gọi function `drop` và dọn dẹp bộ nhớ heap cho biến đó. Nhưng Hình 4-2 cho thấy cả hai data pointer đều trỏ đến cùng một vị trí. Đây là một vấn đề: Khi `s2` và `s1` ra khỏi scope, cả hai đều sẽ cố gắng giải phóng cùng một bộ nhớ. Điều này được gọi là lỗi _double free_ và là một trong những memory safety bug mà chúng ta đã đề cập trước đây. Giải phóng bộ nhớ hai lần có thể dẫn đến hỏng bộ nhớ, có thể dẫn đến lỗ hổng bảo mật.

Để đảm bảo an toàn bộ nhớ, sau dòng `let s2 = s1;`, Rust coi `s1` là không còn hợp lệ. Do đó, Rust không cần giải phóng bất cứ thứ gì khi `s1` ra khỏi scope. Xem điều gì xảy ra khi bạn cố gắng sử dụng `s1` sau khi `s2` được tạo; nó sẽ không hoạt động:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/src/main.rs:here}}
```

Bạn sẽ gặp lỗi như thế này vì Rust ngăn bạn sử dụng reference không hợp lệ:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/output.txt}}
```

Nếu bạn đã nghe các thuật ngữ _shallow copy_ và _deep copy_ khi làm việc với các ngôn ngữ khác, khái niệm sao chép pointer, độ dài và capacity mà không sao chép dữ liệu có vẻ giống như tạo một shallow copy. Nhưng vì Rust cũng vô hiệu hóa biến đầu tiên, thay vì được gọi là shallow copy, nó được gọi là _move_. Trong ví dụ này, chúng ta sẽ nói rằng `s1` đã được _moved_ vào `s2`. Vì vậy, điều thực sự xảy ra được hiển thị trong Hình 4-4.

<img alt="Ba bảng: bảng s1 và s2 biểu diễn các string đó trên
stack, tương ứng, và cả hai đều trỏ đến cùng dữ liệu string trên heap.
Bảng s1 bị làm mờ vì s1 không còn hợp lệ; chỉ s2 mới có thể được sử dụng để
truy cập dữ liệu heap." src="img/trpl04-04.svg" class="center" style="width:
50%;" />

<span class="caption">Hình 4-4: Biểu diễn trong bộ nhớ sau khi `s1` đã bị
vô hiệu hóa</span>

Điều đó giải quyết vấn đề của chúng ta! Chỉ với `s2` hợp lệ, khi nó ra khỏi scope, nó một mình sẽ giải phóng bộ nhớ, và chúng ta đã xong.

Ngoài ra, còn có một lựa chọn thiết kế được ngụ ý bởi điều này: Rust sẽ không bao giờ tự động tạo các bản sao "deep" của dữ liệu của bạn. Do đó, bất kỳ việc sao chép _tự động_ nào cũng có thể được coi là không tốn kém về hiệu suất runtime.

#### Scope và Assignment

Điều ngược lại cũng đúng đối với mối quan hệ giữa scoping, ownership và bộ nhớ được giải phóng thông qua function `drop`. Khi bạn gán một giá trị hoàn toàn mới cho một biến hiện có, Rust sẽ gọi `drop` và giải phóng bộ nhớ của giá trị gốc ngay lập tức. Hãy xem xét code này, ví dụ:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-04b-replacement-drop/src/main.rs:here}}
```

Đầu tiên chúng ta khai báo biến `s` và bind nó vào một `String` với giá trị `"hello"`. Sau đó, chúng ta ngay lập tức tạo một `String` mới với giá trị `"ahoy"` và gán nó cho `s`. Tại thời điểm này, không có gì tham chiếu đến giá trị gốc trên heap nữa. Hình 4-5 minh họa dữ liệu stack và heap bây giờ:

<img alt="Một bảng biểu diễn giá trị string trên stack, trỏ đến
đoạn dữ liệu string thứ hai (ahoy) trên heap, với dữ liệu string gốc (hello)
bị làm mờ vì không thể truy cập nữa."
src="img/trpl04-05.svg" class="center" style="width: 50%;" />

<span class="caption">Hình 4-5: Biểu diễn trong bộ nhớ sau khi giá trị ban đầu
đã được thay thế hoàn toàn</span>

String gốc do đó ngay lập tức ra khỏi scope. Rust sẽ chạy function `drop` trên nó và bộ nhớ của nó sẽ được giải phóng ngay lập tức. Khi chúng ta in giá trị ở cuối, nó sẽ là `"ahoy, world!"`.

<!-- Old headings. Do not remove or links may break. -->

<a id="ways-variables-and-data-interact-clone"></a>

#### Các biến và dữ liệu tương tác với Clone

Nếu chúng ta _muốn_ deep copy dữ liệu heap của `String`, không chỉ dữ liệu stack, chúng ta có thể sử dụng một method phổ biến gọi là `clone`. Chúng ta sẽ thảo luận về method syntax trong Chương 5, nhưng vì methods là tính năng phổ biến trong nhiều ngôn ngữ lập trình, bạn có thể đã thấy chúng trước đây.

Đây là một ví dụ về method `clone` trong action:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-05-clone/src/main.rs:here}}
```

Điều này hoạt động tốt và tường minh tạo ra hành vi được hiển thị trong Hình 4-3, nơi dữ liệu heap _thực sự_ được sao chép.

Khi bạn thấy một lệnh gọi đến `clone`, bạn biết rằng một số code tùy ý đang được thực thi và code đó có thể tốn kém. Đây là dấu hiệu trực quan cho biết có điều gì đó khác biệt đang diễn ra.

#### Dữ liệu chỉ trên Stack: Copy

Còn một điều khác chúng ta chưa nói đến. Code sử dụng integers—một phần trong số đó được hiển thị trong Listing 4-2—hoạt động và hợp lệ:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-06-copy/src/main.rs:here}}
```

Nhưng code này có vẻ mâu thuẫn với những gì chúng ta vừa học: Chúng ta không có lệnh gọi đến `clone`, nhưng `x` vẫn hợp lệ và không bị moved vào `y`.

Lý do là các kiểu như integers có kích thước đã biết tại compile time được lưu trữ hoàn toàn trên stack, vì vậy các bản sao của giá trị thực sự rất nhanh để tạo. Điều đó có nghĩa là không có lý do gì chúng ta muốn ngăn `x` khỏi hợp lệ sau khi chúng ta tạo biến `y`. Nói cách khác, không có sự khác biệt giữa deep và shallow copying ở đây, vì vậy việc gọi `clone` sẽ không làm gì khác so với shallow copying thông thường, và chúng ta có thể bỏ qua nó.

Rust có một annotation đặc biệt gọi là `Copy` trait mà chúng ta có thể đặt trên các kiểu được lưu trữ trên stack, như integers (chúng ta sẽ nói nhiều hơn về traits trong [Chương 10][traits]<!-- ignore -->). Nếu một kiểu implement `Copy` trait, các biến sử dụng nó không bị moved, mà thay vào đó được sao chép một cách bình thường, khiến chúng vẫn hợp lệ sau khi gán cho biến khác.

Rust sẽ không cho phép chúng ta annotate một kiểu với `Copy` nếu kiểu đó, hoặc bất kỳ phần nào của nó, đã implement `Drop` trait. Nếu kiểu cần có điều gì đó đặc biệt xảy ra khi giá trị ra khỏi scope và chúng ta thêm annotation `Copy` vào kiểu đó, chúng ta sẽ gặp lỗi compile-time. Để tìm hiểu về cách thêm annotation `Copy` vào kiểu của bạn để implement trait, xem ["Derivable Traits"][derivable-traits]<!-- ignore --> trong Phụ lục C.

Vậy, những kiểu nào implement `Copy` trait? Bạn có thể kiểm tra tài liệu cho kiểu đã cho để chắc chắn, nhưng theo quy tắc chung, bất kỳ nhóm giá trị scalar đơn giản nào đều có thể implement `Copy`, và không có gì yêu cầu allocation hoặc là một dạng tài nguyên có thể implement `Copy`. Đây là một số kiểu implement `Copy`:

- Tất cả các kiểu integer, chẳng hạn như `u32`.
- Kiểu Boolean, `bool`, với các giá trị `true` và `false`.
- Tất cả các kiểu floating-point, chẳng hạn như `f64`.
- Kiểu character, `char`.
- Tuples, nếu chúng chỉ chứa các kiểu cũng implement `Copy`. Ví dụ,
  `(i32, i32)` implement `Copy`, nhưng `(i32, String)` thì không.

### Ownership và Functions

Cơ chế truyền một giá trị vào một function tương tự như khi gán một giá trị cho một biến. Truyền một biến vào một function sẽ move hoặc copy, giống như assignment. Listing 4-3 có một ví dụ với một số annotation cho thấy các biến đi vào và ra khỏi scope ở đâu.

<Listing number="4-3" file-name="src/main.rs" caption="Các functions với ownership và scope được annotated">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-03/src/main.rs}}
```

</Listing>

Nếu chúng ta cố gắng sử dụng `s` sau khi gọi `takes_ownership`, Rust sẽ throw lỗi compile-time. Những kiểm tra tĩnh này bảo vệ chúng ta khỏi những sai lầm. Hãy thử thêm code vào `main` sử dụng `s` và `x` để xem bạn có thể sử dụng chúng ở đâu và các quy tắc ownership ngăn bạn ở đâu.

### Return Values và Scope

Return values cũng có thể chuyển ownership. Listing 4-4 cho thấy một ví dụ về một function trả về một giá trị, với các annotation tương tự như trong Listing 4-3.

<Listing number="4-4" file-name="src/main.rs" caption="Chuyển ownership của return values">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-04/src/main.rs}}
```

</Listing>

Ownership của một biến theo cùng một pattern mỗi lần: Gán một giá trị cho biến khác sẽ move nó. Khi một biến bao gồm dữ liệu trên heap ra khỏi scope, giá trị sẽ được dọn dẹp bởi `drop` trừ khi ownership của dữ liệu đã được moved sang một biến khác.

Mặc dù điều này hoạt động, việc lấy ownership và sau đó trả lại ownership với mỗi function là một chút tẻ nhạt. Nếu chúng ta muốn để một function sử dụng một giá trị nhưng không lấy ownership thì sao? Khá phiền phức là bất cứ thứ gì chúng ta truyền vào cũng cần được trả lại nếu chúng ta muốn sử dụng lại, ngoài ra bất kỳ dữ liệu nào phát sinh từ thân function mà chúng ta có thể muốn trả về cũng vậy.

Rust cho phép chúng ta trả về nhiều giá trị sử dụng một tuple, như được hiển thị trong Listing 4-5.

<Listing number="4-5" file-name="src/main.rs" caption="Trả lại ownership của các tham số">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-05/src/main.rs}}
```

</Listing>

Nhưng đây là quá nhiều nghi lễ và nhiều công việc cho một khái niệm nên phổ biến. May mắn cho chúng ta, Rust có một tính năng để sử dụng một giá trị mà không chuyển ownership: references.

[data-types]: ch03-02-data-types.html#data-types
[ch8]: ch08-02-strings.html
[traits]: ch10-02-traits.html
[derivable-traits]: appendix-03-derivable-traits.html
[methods]: ch05-03-method-syntax.html#methods
[paths-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
