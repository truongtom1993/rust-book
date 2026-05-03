## Refactoring để cải thiện tính module và xử lý lỗi

Để cải thiện chương trình, chúng ta sẽ sửa bốn vấn đề liên quan đến cấu trúc của chương trình và cách nó xử lý các potential error.

Đầu tiên, function `main` của chúng ta hiện đang thực hiện hai tác vụ: parse argument và đọc file. Khi chương trình phát triển, số lượng tác vụ riêng biệt mà function `main` xử lý sẽ tăng lên. Khi một function có thêm nhiều responsibility, nó sẽ trở nên khó để reason hơn, khó test hơn, và khó thay đổi hơn mà không làm hỏng một phần nào đó. Tốt nhất là nên tách functionality để mỗi function chỉ chịu trách nhiệm cho một tác vụ.

Vấn đề này cũng liên quan đến vấn đề thứ hai: mặc dù `query` và `file_path` là các configuration variable của chương trình, những variable như `contents` lại được dùng để thực hiện logic của chương trình. `main` càng dài thì chúng ta càng cần đưa nhiều variable vào scope; càng có nhiều variable trong scope thì càng khó theo dõi purpose của từng variable. Tốt nhất là nhóm các configuration variable vào một structure để mục đích của chúng trở nên rõ ràng.

Vấn đề thứ ba là chúng ta đã dùng `expect` để in ra error message khi việc đọc file thất bại, nhưng error message đó chỉ in `Should have been able to read the file`. Việc đọc file có thể fail theo nhiều cách: ví dụ, file có thể không tồn tại, hoặc chúng ta không có permission để mở nó. Hiện tại, bất kể tình huống nào, chúng ta cũng in cùng một error message cho mọi trường hợp, điều này không cung cấp thông tin hữu ích nào cho user.

Thứ tư, chúng ta dùng `expect` để handle error, và nếu user chạy chương trình mà không truyền đủ argument, họ sẽ nhận được lỗi `index out of bounds` từ Rust, lỗi này không giải thích rõ vấn đề là gì. Sẽ tốt hơn nếu toàn bộ error-handling code nằm ở một nơi để những maintainer sau này chỉ cần xem một chỗ nếu logic xử lý lỗi cần thay đổi. Việc gom toàn bộ error-handling code vào một nơi cũng giúp đảm bảo rằng chúng ta đang in ra những message có ý nghĩa với end user.

Hãy giải quyết bốn vấn đề này bằng cách refactor project của chúng ta.

<!-- Old headings. Do not remove or links may break. -->

<a id="separation-of-concerns-for-binary-projects"></a>

### Tách biệt trách nhiệm trong Binary Project

Vấn đề tổ chức khi giao responsibility của nhiều tác vụ cho function `main` là một vấn đề phổ biến trong nhiều binary project. Vì vậy, nhiều Rust programmer thấy hữu ích khi tách các concern riêng biệt của một binary program ra khi function `main` bắt đầu trở nên lớn. Quá trình này có các bước sau:

- Tách chương trình của bạn thành file _main.rs_ và file _lib.rs_, rồi chuyển logic của chương trình sang _lib.rs_.
- Miễn là logic parse command line vẫn còn nhỏ, nó có thể tiếp tục nằm trong function `main`.
- Khi logic parse command line bắt đầu phức tạp, hãy tách nó ra khỏi `main` thành các function hoặc type khác.

Sau quá trình này, responsibility còn lại trong function `main` nên được giới hạn ở những việc sau:

- Gọi logic parse command line với các giá trị argument.
- Thiết lập các cấu hình khác nếu cần.
- Gọi một function `run` trong _lib.rs_.
- Xử lý lỗi nếu `run` trả về error.

Pattern này nói về việc tách biệt concern: _main.rs_ xử lý việc chạy chương trình và _lib.rs_ xử lý toàn bộ logic của tác vụ chính. Bởi vì bạn không thể test trực tiếp function `main`, cấu trúc này cho phép bạn test toàn bộ logic của chương trình bằng cách di chuyển nó ra khỏi `main`. Phần code còn lại trong `main` sẽ đủ nhỏ để có thể verify tính đúng đắn chỉ bằng cách đọc. Hãy rework chương trình của chúng ta theo quy trình này.

#### Tách bộ parse argument

Chúng ta sẽ tách functionality parse argument thành một function mà `main` sẽ gọi. Listing 12-5 cho thấy phần mở đầu mới của function `main`, nơi gọi một function mới tên là `parse_config`, function này sẽ được định nghĩa trong _src/main.rs_.

<Listing number="12-5" file-name="src/main.rs" caption="Tách một function `parse_config` ra khỏi `main`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-05/src/main.rs:here}}
```

</Listing>

Chúng ta vẫn collect các command line argument vào một vector, nhưng thay vì gán giá trị argument ở index 1 cho variable `query` và giá trị ở index 2 cho variable `file_path` ngay trong function `main`, chúng ta truyền toàn bộ vector vào function `parse_config`. Sau đó, `parse_config` sẽ giữ logic xác định argument nào tương ứng với variable nào và trả các giá trị đó về cho `main`. Chúng ta vẫn tạo các variable `query` và `file_path` trong `main`, nhưng `main` không còn chịu trách nhiệm xác định mối tương ứng giữa command line argument và variable nữa.

Việc rework này có thể trông hơi quá mức đối với chương trình nhỏ của chúng ta, nhưng chúng ta đang refactor theo các bước nhỏ, tăng dần. Sau khi thực hiện thay đổi này, hãy chạy lại chương trình để verify rằng việc parse argument vẫn hoạt động. Việc kiểm tra tiến độ thường xuyên là tốt, vì nó giúp xác định nguyên nhân của vấn đề khi chúng xảy ra.

#### Gom các giá trị cấu hình lại

Chúng ta có thể thực hiện thêm một bước nhỏ để cải thiện function `parse_config`. Hiện tại, chúng ta đang trả về một tuple, nhưng ngay sau đó lại tách tuple đó thành từng phần riêng lẻ. Đây là dấu hiệu cho thấy có lẽ chúng ta vẫn chưa có abstraction phù hợp.

Một dấu hiệu khác cho thấy vẫn còn chỗ để cải thiện là phần `config` trong tên `parse_config`, điều này ngụ ý rằng hai giá trị mà chúng ta trả về có liên quan với nhau và đều là một phần của cùng một configuration value. Hiện tại, chúng ta chưa thể hiện rõ ý nghĩa này trong structure của dữ liệu, ngoài việc nhóm hai giá trị vào tuple; thay vào đó, chúng ta sẽ đặt hai giá trị này vào một struct và đặt tên có ý nghĩa cho từng field. Làm vậy sẽ giúp các maintainer sau này hiểu dễ hơn các giá trị khác nhau liên hệ với nhau thế nào và purpose của chúng là gì.

Listing 12-6 cho thấy các cải tiến đối với function `parse_config`.

<Listing number="12-6" file-name="src/main.rs" caption="Refactor `parse_config` để trả về một instance của struct `Config`">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-06/src/main.rs:here}}
```

</Listing>

Chúng ta đã thêm một struct tên là `Config`, được định nghĩa với các field `query` và `file_path`. Signature của `parse_config` giờ đây cho biết rằng nó trả về một giá trị `Config`. Trong body của `parse_config`, trước đây chúng ta trả về các string slice tham chiếu đến các giá trị `String` trong `args`, còn bây giờ chúng ta định nghĩa `Config` để nó chứa các giá trị `String` được sở hữu trực tiếp. Variable `args` trong `main` là owner của các giá trị argument và chỉ cho phép function `parse_config` borrow chúng, điều này có nghĩa là chúng ta sẽ vi phạm borrowing rule của Rust nếu `Config` cố lấy ownership của các giá trị trong `args`.

Có nhiều cách để quản lý dữ liệu `String`; cách dễ nhất, dù hơi kém hiệu quả, là gọi method `clone` trên các giá trị. Điều này sẽ tạo một bản copy đầy đủ của dữ liệu để `Config` instance sở hữu, việc này tốn thời gian và memory hơn so với việc lưu reference đến dữ liệu chuỗi. Tuy nhiên, clone dữ liệu cũng làm cho code của chúng ta rất đơn giản vì không phải quản lý lifetime của các reference; trong trường hợp này, đánh đổi một chút performance để lấy sự đơn giản là điều hợp lý.

> ### Trade-off khi dùng `clone`
>
> Nhiều Rustacean có xu hướng tránh dùng `clone` để sửa các ownership problem vì runtime cost của nó. Ở [Chapter 13][ch13]<!-- ignore -->, bạn sẽ học cách dùng các method hiệu quả hơn trong kiểu tình huống này. Nhưng hiện tại, copy một vài string để tiếp tục tiến lên là hoàn toàn ổn vì bạn chỉ thực hiện các bản copy này đúng một lần và file path cùng query string của bạn đều rất nhỏ. Tốt hơn là có một chương trình chạy được dù hơi kém hiệu quả, còn hơn là cố hyperoptimize code ngay ở lần đầu. Khi bạn có nhiều kinh nghiệm hơn với Rust, bạn sẽ dễ bắt đầu với solution hiệu quả nhất hơn, nhưng hiện tại, gọi `clone` là hoàn toàn chấp nhận được.

Chúng ta đã cập nhật `main` để đặt instance `Config` được trả về từ `parse_config` vào một variable tên là `config`, đồng thời cập nhật phần code trước đây dùng các variable riêng lẻ `query` và `file_path` để bây giờ dùng các field trên struct `Config`.

Giờ đây code của chúng ta truyền đạt rõ ràng hơn rằng `query` và `file_path` có liên quan với nhau và purpose của chúng là để configure cách chương trình hoạt động. Bất kỳ code nào sử dụng các giá trị này đều biết rằng chúng nằm trong instance `config`, trong các field được đặt tên đúng theo purpose của chúng.

#### Tạo constructor cho `Config`

Cho đến lúc này, chúng ta đã tách logic chịu trách nhiệm parse command line argument ra khỏi `main` và đặt nó vào function `parse_config`. Việc này giúp chúng ta nhận ra rằng `query` và `file_path` có liên quan với nhau, và mối quan hệ đó nên được thể hiện trong code. Sau đó, chúng ta thêm struct `Config` để đặt tên cho purpose chung của `query` và `file_path`, đồng thời có thể trả về tên của các giá trị dưới dạng field name của struct từ function `parse_config`.

Vì vậy, bây giờ khi purpose của function `parse_config` là tạo một instance `Config`, chúng ta có thể thay đổi `parse_config` từ một plain function thành một function tên `new` được associate với struct `Config`. Việc thay đổi này sẽ khiến code idiomatic hơn. Chúng ta có thể tạo instance của các type trong standard library, chẳng hạn như `String`, bằng cách gọi `String::new`. Tương tự, bằng cách đổi `parse_config` thành một function `new` associate với `Config`, chúng ta sẽ có thể tạo instance `Config` bằng cách gọi `Config::new`. Listing 12-7 cho thấy những thay đổi cần thực hiện.

<Listing number="12-7" file-name="src/main.rs" caption="Chuyển `parse_config` thành `Config::new`">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-07/src/main.rs:here}}
```

</Listing>

Chúng ta đã cập nhật `main`, nơi trước đây gọi `parse_config`, để thay vào đó gọi `Config::new`. Chúng ta đổi tên `parse_config` thành `new` và chuyển nó vào trong một `impl` block, qua đó associate function `new` với `Config`. Hãy thử compile lại đoạn code này để chắc rằng nó vẫn hoạt động.

### Sửa phần xử lý lỗi

Bây giờ chúng ta sẽ xử lý phần error handling. Hãy nhớ rằng việc truy cập các giá trị trong vector `args` tại index 1 hoặc index 2 sẽ khiến chương trình panic nếu vector chứa ít hơn ba phần tử. Hãy thử chạy chương trình mà không truyền argument nào; nó sẽ trông như sau:

```console
{{#include ../listings/ch12-an-io-project/listing-12-07/output.txt}}
```

Dòng `index out of bounds: the len is 1 but the index is 1` là một error message dành cho programmer. Nó sẽ không giúp end user hiểu họ nên làm gì thay thế. Hãy sửa điều đó ngay bây giờ.

#### Cải thiện error message

Trong Listing 12-8, chúng ta thêm một check trong function `new` để verify rằng slice đủ dài trước khi truy cập index 1 và index 2. Nếu slice không đủ dài, chương trình panic và hiển thị một error message tốt hơn.

<Listing number="12-8" file-name="src/main.rs" caption="Thêm một check cho số lượng argument">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-08/src/main.rs:here}}
```

</Listing>

Đoạn code này tương tự với [function `Guess::new` mà chúng ta đã viết trong Listing 9-13][ch9-custom-types]<!-- ignore -->, nơi chúng ta gọi `panic!` khi argument `value` nằm ngoài phạm vi giá trị hợp lệ. Thay vì check một range giá trị như ở đó, ở đây chúng ta kiểm tra xem độ dài của `args` có ít nhất là `3` hay không, và phần còn lại của function có thể hoạt động với assumption rằng condition này đã được thỏa mãn. Nếu `args` có ít hơn ba phần tử, condition này sẽ là `true`, và chúng ta gọi macro `panic!` để kết thúc chương trình ngay lập tức.

Với thêm vài dòng code này trong `new`, hãy chạy lại chương trình mà không truyền argument nào để xem error bây giờ trông ra sao:

```console
{{#include ../listings/ch12-an-io-project/listing-12-08/output.txt}}
```

Output này tốt hơn: giờ chúng ta có một error message hợp lý. Tuy nhiên, chúng ta vẫn có thêm những thông tin thừa mà không muốn đưa cho user. Có lẽ kỹ thuật mà chúng ta dùng trong Listing 9-13 không phải là lựa chọn tốt nhất trong trường hợp này: gọi `panic!` phù hợp hơn với programming problem hơn là usage problem, [như đã thảo luận trong Chapter 9][ch9-error-guidelines]<!-- ignore -->. Thay vào đó, chúng ta sẽ dùng kỹ thuật còn lại mà bạn đã học ở Chapter 9—[trả về một `Result`][ch9-result]<!-- ignore --> để biểu thị success hoặc error.

<!-- Old headings. Do not remove or links may break. -->

<a id="returning-a-result-from-new-instead-of-calling-panic"></a>

#### Trả về `Result` thay vì gọi `panic!`

Thay vào đó, chúng ta có thể trả về một giá trị `Result`, giá trị này sẽ chứa một instance `Config` trong trường hợp thành công và mô tả vấn đề trong trường hợp lỗi. Đồng thời, chúng ta cũng sẽ đổi tên function từ `new` thành `build` vì nhiều programmer kỳ vọng các function `new` sẽ không bao giờ fail. Khi `Config::build` giao tiếp với `main`, chúng ta có thể dùng type `Result` để signal rằng đã có vấn đề. Sau đó, chúng ta có thể thay đổi `main` để chuyển một `Err` variant thành một lỗi thực tế hơn cho user, không có các phần text đi kèm như `thread 'main'` và `RUST_BACKTRACE` mà lời gọi `panic!` gây ra.

Listing 12-9 cho thấy những thay đổi cần thực hiện đối với return value của function mà bây giờ chúng ta gọi là `Config::build`, cùng với phần body của function để trả về một `Result`. Lưu ý rằng đoạn này sẽ chưa compile cho đến khi chúng ta cập nhật `main` trong listing tiếp theo.

<Listing number="12-9" file-name="src/main.rs" caption="Trả về một `Result` từ `Config::build`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-09/src/main.rs:here}}
```

</Listing>

Function `build` của chúng ta trả về một `Result` với một instance `Config` trong trường hợp thành công và một string literal trong trường hợp lỗi. Các error value của chúng ta sẽ luôn là string literal có lifetime `'static`.

Chúng ta đã thực hiện hai thay đổi trong body của function: thay vì gọi `panic!` khi user không truyền đủ argument, giờ đây chúng ta trả về một giá trị `Err`, và chúng ta bọc giá trị trả về `Config` trong `Ok`. Những thay đổi này làm cho function tuân theo type signature mới của nó.

Việc trả về một giá trị `Err` từ `Config::build` cho phép function `main` xử lý giá trị `Result` được trả về từ function `build` và exit process một cách clean hơn trong trường hợp lỗi.

<!-- Old headings. Do not remove or links may break. -->

<a id="calling-confignew-and-handling-errors"></a>

#### Gọi `Config::build` và xử lý lỗi

Để xử lý trường hợp lỗi và in ra một message thân thiện với user, chúng ta cần cập nhật `main` để handle `Result` được trả về bởi `Config::build`, như trong Listing 12-10. Chúng ta cũng sẽ lấy responsibility thoát command line tool với nonzero error code khỏi `panic!`, và thay vào đó tự implement việc này. Một exit status khác 0 là convention để signal cho process đã gọi chương trình của chúng ta biết rằng chương trình kết thúc với trạng thái lỗi.

<Listing number="12-10" file-name="src/main.rs" caption="Thoát với error code nếu việc build `Config` thất bại">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-10/src/main.rs:here}}
```

</Listing>

Trong listing này, chúng ta dùng một method mà trước đây chưa được nói chi tiết: `unwrap_or_else`, method này được standard library định nghĩa trên `Result<T, E>`. Việc dùng `unwrap_or_else` cho phép chúng ta định nghĩa custom error handling không dùng `panic!`. Nếu `Result` là giá trị `Ok`, hành vi của method này tương tự như `unwrap`: nó trả về inner value mà `Ok` đang bọc. Tuy nhiên, nếu giá trị là `Err`, method này sẽ gọi đoạn code trong closure, là một anonymous function mà chúng ta định nghĩa và truyền làm argument cho `unwrap_or_else`. Chúng ta sẽ tìm hiểu closures chi tiết hơn ở [Chapter 13][ch13]<!-- ignore -->. Hiện tại, bạn chỉ cần biết rằng `unwrap_or_else` sẽ truyền inner value của `Err`, trong trường hợp này là static string `"not enough arguments"` mà chúng ta đã thêm ở Listing 12-9, vào closure thông qua argument `err` nằm giữa các dấu gạch đứng. Khi closure chạy, code bên trong có thể dùng giá trị `err`.

Chúng ta đã thêm một dòng `use` mới để đưa `process` từ standard library vào scope. Đoạn code trong closure được chạy ở trường hợp lỗi chỉ có hai dòng: chúng ta in giá trị `err` và sau đó gọi `process::exit`. Function `process::exit` sẽ dừng chương trình ngay lập tức và trả về số được truyền vào làm exit status code. Điều này tương tự với cách xử lý dựa trên `panic!` mà chúng ta đã dùng ở Listing 12-8, nhưng bây giờ không còn tất cả output dư thừa nữa. Hãy thử xem:

```console
{{#include ../listings/ch12-an-io-project/listing-12-10/output.txt}}
```

Tuyệt! Output này thân thiện hơn nhiều với user.

<!-- Old headings. Do not remove or links may break. -->

<a id="extracting-logic-from-the-main-function"></a>

### Tách logic khỏi `main`

Giờ đây khi đã refactor xong phần parse configuration, hãy chuyển sang logic của chương trình. Như đã nói trong [“Tách biệt trách nhiệm trong Binary Project”](#separation-of-concerns-for-binary-projects)<!-- ignore -->, chúng ta sẽ tách một function tên là `run`, function này sẽ giữ toàn bộ logic hiện đang nằm trong `main` mà không liên quan đến việc thiết lập cấu hình hoặc xử lý lỗi. Khi hoàn tất, function `main` sẽ ngắn gọn, dễ verify bằng cách quan sát, và chúng ta sẽ có thể viết test cho tất cả logic còn lại.

Listing 12-11 cho thấy bước cải tiến nhỏ, tăng dần này: tách một function `run`.

<Listing number="12-11" file-name="src/main.rs" caption="Tách một function `run` chứa phần logic còn lại của chương trình">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-11/src/main.rs:here}}
```

</Listing>

Function `run` bây giờ chứa toàn bộ logic còn lại từ `main`, bắt đầu từ việc đọc file. Function `run` nhận một instance `Config` làm argument.

<!-- Old headings. Do not remove or links may break. -->

<a id="returning-errors-from-the-run-function"></a>

#### Trả về lỗi từ `run`

Khi phần logic còn lại của chương trình đã được tách vào function `run`, chúng ta có thể cải thiện phần error handling giống như đã làm với `Config::build` ở Listing 12-9. Thay vì để chương trình panic bằng cách gọi `expect`, function `run` sẽ trả về một `Result<T, E>` khi có lỗi xảy ra. Điều này cho phép chúng ta tiếp tục gom logic xử lý lỗi về `main` theo cách thân thiện với user. Listing 12-12 cho thấy những thay đổi cần thực hiện đối với signature và body của `run`.

<Listing number="12-12" file-name="src/main.rs" caption="Thay đổi function `run` để trả về `Result`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-12/src/main.rs:here}}
```

</Listing>

Chúng ta đã thực hiện ba thay đổi quan trọng ở đây. Đầu tiên, chúng ta thay đổi return type của function `run` thành `Result<(), Box<dyn Error>>`. Trước đây function này trả về unit type `()`, và chúng ta vẫn giữ giá trị đó trong trường hợp `Ok`.

Đối với error type, chúng ta dùng trait object `Box<dyn Error>` (và đã đưa `std::error::Error` vào scope bằng một `use` statement ở đầu file). Chúng ta sẽ tìm hiểu trait object ở [Chapter 18][ch18]<!-- ignore -->. Hiện tại, bạn chỉ cần biết rằng `Box<dyn Error>` nghĩa là function sẽ trả về một type implement trait `Error`, nhưng chúng ta không cần chỉ rõ đó là type cụ thể nào. Điều này cho chúng ta flexibility để trả về các error value có thể thuộc nhiều type khác nhau trong các trường hợp lỗi khác nhau. Từ khóa `dyn` là viết tắt của _dynamic_.

Thứ hai, chúng ta bỏ lời gọi `expect` để thay bằng operator `?`, như đã nói ở [Chapter 9][ch9-question-mark]<!-- ignore -->. Thay vì `panic!` khi có lỗi, `?` sẽ trả về error value từ function hiện tại để caller xử lý.

Thứ ba, function `run` bây giờ trả về một giá trị `Ok` trong trường hợp thành công. Chúng ta khai báo success type của function `run` trong signature là `()`, vì vậy chúng ta cần bọc unit type value trong `Ok`. Cú pháp `Ok(())` này ban đầu có thể trông hơi lạ. Nhưng dùng `()` theo cách này là cách idiomatic để biểu thị rằng chúng ta gọi `run` chỉ vì side effect của nó; nó không trả về một giá trị mà chúng ta cần dùng.

Khi bạn chạy đoạn code này, nó sẽ compile nhưng hiển thị một warning:

```console
{{#include ../listings/ch12-an-io-project/listing-12-12/output.txt}}
```

Rust đang nói với chúng ta rằng code của chúng ta đã bỏ qua giá trị `Result`, trong khi giá trị `Result` đó có thể cho biết rằng một lỗi đã xảy ra. Nhưng chúng ta không kiểm tra xem có lỗi hay không, và compiler đang nhắc rằng có lẽ chúng ta đã định viết một đoạn error-handling code ở đây. Hãy sửa vấn đề này ngay.

#### Xử lý lỗi được trả về từ `run` trong `main`

Chúng ta sẽ kiểm tra lỗi và xử lý nó bằng một kỹ thuật tương tự như kỹ thuật đã dùng với `Config::build` ở Listing 12-10, nhưng có một khác biệt nhỏ:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-01-handling-errors-in-main/src/main.rs:here}}
```

Chúng ta dùng `if let` thay vì `unwrap_or_else` để kiểm tra xem `run` có trả về giá trị `Err` hay không, và gọi `process::exit(1)` nếu có. Function `run` không trả về một giá trị mà chúng ta muốn `unwrap` theo cách `Config::build` trả về instance `Config`. Bởi vì `run` trả về `()` trong trường hợp thành công, chúng ta chỉ quan tâm đến việc phát hiện lỗi, nên không cần `unwrap_or_else` trả về giá trị đã được unwrapped, vì giá trị đó cũng chỉ là `()`.

Phần body của `if let` và của closure trong `unwrap_or_else` là giống nhau ở cả hai trường hợp: chúng ta in lỗi và exit.

### Tách code sang library crate

Project `minigrep` của chúng ta hiện đã khá ổn! Bây giờ chúng ta sẽ tách file _src/main.rs_ và chuyển một phần code vào file _src/lib.rs_. Bằng cách đó, chúng ta có thể test code và có một file _src/main.rs_ với ít responsibility hơn.

Hãy định nghĩa phần code chịu trách nhiệm search text trong _src/lib.rs_ thay vì trong _src/main.rs_, điều này sẽ cho phép chúng ta (hoặc bất kỳ ai khác sử dụng library `minigrep`) có thể gọi function search từ nhiều context khác ngoài binary `minigrep`.

Trước tiên, hãy định nghĩa signature của function `search` trong _src/lib.rs_ như trong Listing 12-13, với body gọi macro `unimplemented!`. Chúng ta sẽ giải thích signature này chi tiết hơn khi điền implementation.

<Listing number="12-13" file-name="src/lib.rs" caption="Định nghĩa function `search` trong *src/lib.rs*">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-13/src/lib.rs}}
```

</Listing>

Chúng ta đã dùng từ khóa `pub` trên function definition để chỉ định rằng `search` là một phần của public API của library crate. Giờ đây chúng ta có một library crate có thể dùng từ binary crate và có thể test được!

Bây giờ chúng ta cần đưa code được định nghĩa trong _src/lib.rs_ vào scope của binary crate trong _src/main.rs_ và gọi nó, như trong Listing 12-14.

<Listing number="12-14" file-name="src/main.rs" caption="Dùng function `search` của library crate `minigrep` trong *src/main.rs*">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-14/src/main.rs:here}}
```

</Listing>

Chúng ta thêm dòng `use minigrep::search` để đưa function `search` từ library crate vào scope của binary crate. Sau đó, trong function `run`, thay vì in ra nội dung của file, chúng ta gọi function `search` và truyền `config.query` cùng `contents` làm argument. Tiếp theo, `run` sẽ dùng một vòng lặp `for` để in ra từng dòng được `search` trả về mà match với query. Đây cũng là thời điểm tốt để bỏ các lời gọi `println!` trong function `main` từng dùng để hiển thị query và file path, ताकि chương trình của chúng ta chỉ in kết quả search (nếu không có lỗi xảy ra).

Lưu ý rằng function `search` sẽ collect tất cả result vào một vector rồi mới trả về trước khi có bất kỳ việc in nào xảy ra. Implementation này có thể chậm khi hiển thị result lúc search trên file lớn, vì kết quả không được in ra ngay khi được tìm thấy; chúng ta sẽ thảo luận một cách có thể cải thiện việc này bằng iterator trong Chapter 13.

Phù! Đó là khá nhiều việc, nhưng chúng ta đã đặt nền tảng tốt cho tương lai. Bây giờ việc xử lý lỗi trở nên dễ hơn nhiều, và code cũng modular hơn. Từ đây trở đi, gần như toàn bộ công việc của chúng ta sẽ được thực hiện trong _src/lib.rs_.

Hãy tận dụng tính modular mới này bằng cách làm một việc mà với code cũ sẽ khó hơn nhiều nhưng với code mới lại rất dễ: chúng ta sẽ viết test!

[ch13]: ch13-00-functional-features.html
[ch9-custom-types]: ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation
[ch9-error-guidelines]: ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling
[ch9-result]: ch09-02-recoverable-errors-with-result.html
[ch18]: ch18-00-oop.html
[ch9-question-mark]: ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator