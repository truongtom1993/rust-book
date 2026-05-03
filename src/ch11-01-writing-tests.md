## Cách viết test

_Test_ là các function Rust dùng để xác minh rằng phần code không phải test đang hoạt động đúng như mong đợi. Thân của test function thường thực hiện ba việc sau:

- Thiết lập dữ liệu hoặc state cần thiết.
- Chạy phần code bạn muốn test.
- Assert rằng kết quả đúng như bạn kỳ vọng.

Hãy cùng xem các tính năng Rust cung cấp riêng cho việc viết test để thực hiện các bước này, bao gồm attribute `test`, một vài macro, và attribute `should_panic`. 

<!-- Old headings. Do not remove or links may break. -->

<a id="the-anatomy-of-a-test-function"></a>

### Cấu trúc của test function

Ở mức đơn giản nhất, một test trong Rust là một function được annotate với attribute `test`. Attribute là metadata gắn với các phần của Rust code; một ví dụ là attribute `derive` mà chúng ta đã dùng với struct trong Chương 5. Để biến một function thành test function, hãy thêm `#[test]` ở dòng ngay trước `fn`. Khi bạn chạy test bằng lệnh `cargo test`, Rust sẽ build một binary test runner để chạy các function đã được annotate và báo cáo mỗi test function pass hay fail. 

Mỗi khi tạo một library project mới với Cargo, một test module cùng một test function bên trong sẽ tự động được generate sẵn. Module này cho bạn một template để viết test, nhờ đó không cần phải tra lại cấu trúc và syntax chính xác mỗi lần bắt đầu project mới. Bạn có thể thêm bao nhiêu test function và bao nhiêu test module tùy ý. 

Chúng ta sẽ khám phá một số khía cạnh về cách test hoạt động bằng cách thử nghiệm với test template trước khi thật sự test bất kỳ code nào. Sau đó, chúng ta sẽ viết các test thực tế gọi vào code đã viết và assert rằng hành vi của nó là đúng. 

Hãy tạo một library project mới tên là `adder` để cộng hai số:

```console
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

Nội dung file _src/lib.rs_ trong thư viện `adder` của bạn sẽ trông như Listing 11-1.

<Listing number="11-1" file-name="src/lib.rs" caption="Phần code được `cargo new` tự động tạo ra">

<!-- manual-regeneration
cd listings/ch11-writing-automated-tests
rm -rf listing-11-01
cargo new listing-11-01 --lib --name adder
cd listing-11-01
echo "$ cargo test" > output.txt
RUSTFLAGS="-A unused_variables -A dead_code" RUST_TEST_THREADS=1 cargo test >> output.txt 2>&1
git diff output.txt # commit mọi thay đổi liên quan; bỏ các thay đổi không liên quan
cd ../../..
-->

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

</Listing>

File bắt đầu với một function `add` ví dụ để chúng ta có thứ gì đó đem đi test. 

Hiện tại, hãy chỉ tập trung vào function `it_works`. Hãy chú ý annotation `#[test]`: attribute này cho biết đây là một test function, để test runner biết cần xử lý function này như một test. Chúng ta cũng có thể có các function không phải test trong module `tests` để hỗ trợ thiết lập các scenario dùng chung hoặc thực hiện các thao tác chung, nên luôn cần chỉ rõ function nào là test. 

Phần thân function ví dụ dùng macro `assert_eq!` để assert rằng `result`, giá trị chứa kết quả của việc gọi `add` với 2 và 2, bằng 4. Assertion này đóng vai trò như một ví dụ về format của một test điển hình. Hãy chạy nó để xem test này pass. 

Lệnh `cargo test` sẽ chạy toàn bộ test trong project, như minh họa ở Listing 11-2. 

<Listing number="11-2" caption="Output khi chạy test được tạo tự động">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-01/output.txt}}
```

</Listing>

Cargo đã compile và chạy test. Ta thấy dòng `running 1 test`. Dòng tiếp theo hiển thị tên của test function được generate, là `tests::it_works`, và kết quả chạy test đó là `ok`. Phần tổng kết `test result: ok.` có nghĩa là tất cả test đều pass, và đoạn `1 passed; 0 failed` là tổng số test pass hoặc fail. 

Có thể đánh dấu một test là ignored để nó không chạy trong một lần chạy cụ thể; phần này sẽ được nói ở mục [“Ignoring Tests Unless Specifically Requested”][ignoring]<!-- ignore --> ở phần sau của chương. Vì ở đây chưa làm điều đó, phần summary hiển thị `0 ignored`. Ta cũng có thể truyền argument cho lệnh `cargo test` để chỉ chạy những test có tên khớp với một chuỗi; việc này gọi là _filtering_, và sẽ được nói trong mục [“Running a Subset of Tests by Name”][subset]<!-- ignore -->. Ở đây ta chưa filter test nào nên cuối summary hiển thị `0 filtered out`. 

Thống kê `0 measured` dành cho benchmark test dùng để đo hiệu năng. Tại thời điểm tài liệu này được viết, benchmark test chỉ có trên nightly Rust. Xem [tài liệu về benchmark test][bench] để biết thêm. 

Phần tiếp theo của output test bắt đầu từ `Doc-tests adder` là kết quả của các documentation test, nếu có. Hiện tại chưa có documentation test nào, nhưng Rust có thể compile mọi code example xuất hiện trong API documentation của bạn. Tính năng này giúp giữ cho docs và code luôn đồng bộ. Chúng ta sẽ bàn về cách viết documentation test trong mục [“Documentation Comments as Tests”][doc-comments]<!-- ignore --> ở Chương 14. Còn bây giờ, hãy tạm bỏ qua phần output `Doc-tests`. 

Bây giờ hãy bắt đầu tùy chỉnh test cho phù hợp với nhu cầu của mình. Trước tiên, đổi tên function `it_works` thành một tên khác, chẳng hạn `exploration`, như sau:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/src/lib.rs}}
```

Sau đó chạy lại `cargo test`. Output lúc này sẽ hiển thị `exploration` thay cho `it_works`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/output.txt}}
```

Giờ chúng ta sẽ thêm một test nữa, nhưng lần này sẽ cố tình làm cho test fail. Test sẽ fail khi có thứ gì đó trong test function panic. Mỗi test được chạy trong một thread mới, và khi main thread thấy test thread bị chết, test đó sẽ bị đánh dấu là failed. Ở Chương 9, chúng ta đã nói rằng cách đơn giản nhất để panic là gọi macro `panic!`. Hãy thêm test mới với tên function là `another`, để file _src/lib.rs_ trông như Listing 11-3. 

<Listing number="11-3" file-name="src/lib.rs" caption="Thêm test thứ hai sẽ fail vì gọi macro `panic!`">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-03/src/lib.rs}}
```

</Listing>

Hãy chạy lại test bằng `cargo test`. Output sẽ trông như Listing 11-4, cho thấy test `exploration` pass còn `another` fail. 

<Listing number="11-4" caption="Kết quả test khi một test pass và một test fail">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-03/output.txt}}
```

</Listing>

<!-- manual-regeneration
rg panicked listings/ch11-writing-automated-tests/listing-11-03/output.txt
kiểm tra số dòng panic khớp với số dòng trong đoạn văn bên dưới
 -->

Thay vì `ok`, dòng `test tests::another` sẽ hiển thị `FAILED`. Có hai section mới xuất hiện giữa phần kết quả từng test và phần summary: phần đầu tiên hiển thị lý do chi tiết vì sao mỗi test fail. Trong trường hợp này, ta thấy `tests::another` fail vì panic với message `Make this test fail` ở dòng 17 trong file _src/lib.rs_. Phần kế tiếp chỉ liệt kê tên của toàn bộ test bị fail, rất hữu ích khi có nhiều test và nhiều output lỗi chi tiết. Chúng ta có thể dùng tên của test bị fail để chỉ chạy riêng test đó nhằm debug dễ hơn; phần này sẽ được bàn thêm ở mục [“Controlling How Tests Are Run”][controlling-how-tests-are-run]<!-- ignore -->. 

Dòng summary ở cuối hiển thị: tổng thể, kết quả test là `FAILED`. Chúng ta có một test pass và một test fail. 

Giờ bạn đã thấy kết quả test trông như thế nào trong các tình huống khác nhau, hãy xem thêm một số macro hữu ích khác ngoài `panic!` dành cho test. 

<!-- Old headings. Do not remove or links may break. -->

<a id="checking-results-with-the-assert-macro"></a>

### Kiểm tra kết quả với `assert!`

Macro `assert!`, được standard library cung cấp, rất hữu ích khi bạn muốn đảm bảo một điều kiện nào đó trong test evaluate thành `true`. Ta truyền cho macro `assert!` một argument có giá trị là Boolean. Nếu giá trị là `true`, sẽ không có gì xảy ra và test pass. Nếu giá trị là `false`, macro `assert!` sẽ gọi `panic!` để làm test fail. Dùng macro `assert!` giúp chúng ta kiểm tra rằng code đang hoạt động đúng như ý định. 

Ở Chương 5, Listing 5-15, chúng ta đã dùng một struct `Rectangle` và method `can_hold`, được lặp lại ở đây trong Listing 11-5. Hãy đặt đoạn code này vào file _src/lib.rs_, sau đó viết một vài test cho nó bằng macro `assert!`. 

<Listing number="11-5" file-name="src/lib.rs" caption="Struct `Rectangle` và method `can_hold` của nó từ Chương 5">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-05/src/lib.rs}}
```

</Listing>

Method `can_hold` trả về một Boolean, nên đây là use case hoàn hảo cho macro `assert!`. Trong Listing 11-6, chúng ta viết một test để kiểm tra `can_hold` bằng cách tạo một instance `Rectangle` có width là 8 và height là 7, rồi assert rằng nó có thể chứa một instance `Rectangle` khác có width là 5 và height là 1. 

<Listing number="11-6" file-name="src/lib.rs" caption="Một test cho `can_hold` để kiểm tra xem rectangle lớn hơn có thật sự chứa được rectangle nhỏ hơn hay không">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-06/src/lib.rs:here}}
```

</Listing>

Hãy chú ý dòng `use super::*;` bên trong module `tests`. Module `tests` là một module bình thường và tuân theo các quy tắc visibility thông thường mà chúng ta đã học trong Chương 7, ở mục [“Paths for Referring to an Item in the Module Tree”][paths-for-referring-to-an-item-in-the-module-tree]<!-- ignore -->. Vì `tests` là một inner module, chúng ta cần đưa phần code đang được test ở outer module vào scope của inner module. Chúng ta dùng glob ở đây, nên mọi thứ được định nghĩa ở outer module đều có thể dùng trong module `tests` này. 

Chúng ta đặt tên test là `larger_can_hold_smaller`, và tạo hai instance `Rectangle` cần thiết. Sau đó gọi macro `assert!` và truyền vào kết quả của việc gọi `larger.can_hold(&smaller)`. Biểu thức này được kỳ vọng sẽ trả về `true`, nên test sẽ pass. Hãy thử xem. 

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-06/output.txt}}
```

Nó đúng là pass. Giờ hãy thêm một test nữa, lần này assert rằng một rectangle nhỏ hơn thì không thể chứa một rectangle lớn hơn:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/src/lib.rs:here}}
```

Vì kết quả đúng của function `can_hold` trong trường hợp này là `false`, chúng ta cần negate kết quả đó trước khi truyền vào macro `assert!`. Nhờ vậy, test sẽ pass nếu `can_hold` trả về `false`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/output.txt}}
```

Hai test đều pass. Giờ hãy xem điều gì xảy ra với kết quả test khi ta cố tình đưa bug vào code. Chúng ta sẽ đổi implementation của method `can_hold` bằng cách thay dấu lớn hơn (`>`) thành dấu nhỏ hơn (`<`) khi so sánh width: 

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/src/lib.rs:here}}
```

Chạy test lúc này sẽ cho ra kết quả sau:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/output.txt}}
```

Các test của chúng ta đã bắt được bug. Vì `larger.width` là `8` còn `smaller.width` là `5`, phép so sánh width trong `can_hold` giờ trả về `false`: 8 không nhỏ hơn 5. 

<!-- Old headings. Do not remove or links may break. -->

<a id="testing-equality-with-the-assert_eq-and-assert_ne-macros"></a>

### Test equality với `assert_eq!` và `assert_ne!`

Một cách phổ biến để xác minh functionality là test sự bằng nhau giữa kết quả của code đang được test và giá trị bạn kỳ vọng code trả về. Bạn có thể làm điều này bằng macro `assert!` và truyền vào một biểu thức dùng toán tử `==`. Tuy nhiên, vì đây là nhu cầu rất phổ biến, standard library cung cấp sẵn một cặp macro là `assert_eq!` và `assert_ne!` để thực hiện việc này thuận tiện hơn. Hai macro này lần lượt so sánh hai argument để kiểm tra bằng nhau hoặc khác nhau. Chúng cũng sẽ in ra cả hai giá trị nếu assertion fail, giúp dễ hiểu _vì sao_ test fail hơn; ngược lại, macro `assert!` chỉ cho biết nó nhận được giá trị `false` từ biểu thức `==`, mà không in ra các giá trị đã dẫn đến kết quả `false` đó. 

Trong Listing 11-7, chúng ta viết một function tên `add_two` để cộng thêm `2` vào parameter của nó, rồi test function này bằng macro `assert_eq!`. 

<Listing number="11-7" file-name="src/lib.rs" caption="Test function `add_two` bằng macro `assert_eq!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-07/src/lib.rs}}
```

</Listing>

Hãy kiểm tra xem nó có pass không:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-07/output.txt}}
```

Chúng ta tạo một biến tên `result` để giữ kết quả của việc gọi `add_two(2)`. Sau đó truyền `result` và `4` làm argument cho macro `assert_eq!`. Dòng output cho test này là `test tests::it_adds_two ... ok`, và chữ `ok` cho biết test đã pass. 

Hãy cố tình đưa bug vào code để xem `assert_eq!` trông như thế nào khi fail. Hãy đổi implementation của function `add_two` thành cộng thêm `3` thay vì `2`:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/src/lib.rs:here}}
```

Chạy lại test:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/output.txt}}
```

Test của chúng ta đã bắt được bug. Test `tests::it_adds_two` bị fail, và message cho biết assertion bị fail là `left == right`, đồng thời cho biết giá trị `left` và `right` là gì. Message này giúp chúng ta bắt đầu debug: argument `left`, tức nơi chứa kết quả của `add_two(2)`, là `5`, trong khi argument `right` là `4`. Bạn có thể hình dung điều này đặc biệt hữu ích khi có rất nhiều test đang chạy. 

Lưu ý rằng trong một số ngôn ngữ và framework test khác, các parameter của function assertion equality được gọi là `expected` và `actual`, và thứ tự truyền argument có ý nghĩa. Tuy nhiên trong Rust, chúng được gọi là `left` và `right`, và thứ tự bạn truyền giá trị mong đợi hay giá trị do code tạo ra là không quan trọng. Ta hoàn toàn có thể viết assertion trong test này là `assert_eq!(4, result)`, và vẫn nhận được đúng message fail `` assertion `left == right` failed `` như cũ. 

Macro `assert_ne!` sẽ pass nếu hai giá trị được truyền vào không bằng nhau, và fail nếu chúng bằng nhau. Macro này hữu ích nhất trong các trường hợp ta không chắc giá trị _sẽ là gì_, nhưng biết rõ giá trị đó chắc chắn _không được là gì_. Ví dụ, nếu đang test một function được đảm bảo sẽ thay đổi input theo một cách nào đó, nhưng cách thay đổi cụ thể lại phụ thuộc vào ngày trong tuần bạn chạy test, thì điều tốt nhất để assert có thể là output của function không được bằng input. 

Bên dưới, các macro `assert_eq!` và `assert_ne!` lần lượt dùng các toán tử `==` và `!=`. Khi assertion fail, các macro này in argument của chúng bằng debug formatting, nghĩa là các giá trị được so sánh phải implement trait `PartialEq` và `Debug`. Tất cả primitive type và hầu hết type trong standard library đều implement các trait này. Với struct và enum do bạn tự định nghĩa, bạn cần implement `PartialEq` để có thể assert equality trên chúng. Bạn cũng cần implement `Debug` để in giá trị ra khi assertion fail. Vì cả hai đều là derivable trait, như đã nói ở Listing 5-12 trong Chương 5, việc này thường đơn giản như thêm annotation `#[derive(PartialEq, Debug)]` vào định nghĩa struct hoặc enum. Xem Appendix C, [“Derivable Traits,”][derivable-traits]<!-- ignore --> để biết thêm chi tiết về các derivable trait này và các trait khác. 

### Thêm custom failure message

Bạn cũng có thể thêm một custom message để in kèm với failure message bằng cách truyền các optional argument vào các macro `assert!`, `assert_eq!`, và `assert_ne!`. Mọi argument được chỉ định sau các argument bắt buộc sẽ được chuyển tiếp cho macro `format!` (đã thảo luận trong [“Concatenating with `+` or `format!`”][concatenating]<!--
ignore --> ở Chương 8), nên bạn có thể truyền một format string chứa placeholder `{}` cùng các giá trị cần chèn vào. Custom message rất hữu ích để mô tả assertion đang có ý nghĩa gì; khi test fail, bạn sẽ hiểu vấn đề của code rõ hơn. 

Ví dụ, giả sử ta có một function dùng để chào một người theo tên, và muốn test rằng tên được truyền vào function có xuất hiện trong output:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-05-greeter/src/lib.rs}}
```

Các yêu cầu của chương trình này vẫn chưa được chốt, và ta khá chắc rằng phần text `Hello` ở đầu lời chào sẽ còn thay đổi. Ta quyết định không muốn phải cập nhật test mỗi khi yêu cầu thay đổi, nên thay vì kiểm tra exact equality với giá trị trả về từ function `greeting`, ta chỉ assert rằng output có chứa text của input parameter. 

Giờ hãy cố tình đưa bug vào code bằng cách sửa `greeting` để không còn đưa `name` vào nữa, nhằm xem default test failure trông như thế nào:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/src/lib.rs:here}}
```

Chạy test này sẽ cho ra kết quả:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/output.txt}}
```

Kết quả này chỉ cho biết assertion đã fail và fail ở dòng nào. Một failure message hữu ích hơn sẽ in ra giá trị từ function `greeting`. Hãy thêm một custom failure message gồm một format string với placeholder được điền bằng giá trị thực tế mà ta nhận được từ function `greeting`: 

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/src/lib.rs:here}}
```

Giờ khi chạy test, ta sẽ nhận được một error message nhiều thông tin hơn:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/output.txt}}
```

Ta có thể thấy giá trị thực tế nhận được ngay trong output test, điều này giúp debug chuyện gì đã xảy ra thay vì chỉ biết mình đang mong đợi điều gì. 

### Kiểm tra panic với `should_panic`

Ngoài việc kiểm tra return value, việc kiểm tra code có xử lý các điều kiện lỗi đúng như mong đợi hay không cũng rất quan trọng. Ví dụ, hãy xét type `Guess` mà chúng ta đã tạo trong Chương 9, Listing 9-13. Những code khác dùng `Guess` phụ thuộc vào guarantee rằng các instance `Guess` chỉ chứa giá trị từ 1 đến 100. Ta có thể viết một test để đảm bảo rằng việc cố tạo một instance `Guess` với giá trị ngoài khoảng đó sẽ gây panic. 

Ta làm điều này bằng cách thêm attribute `should_panic` vào test function. Test sẽ pass nếu code bên trong function panic; test sẽ fail nếu code bên trong function không panic. 

Listing 11-8 cho thấy một test dùng để kiểm tra rằng các điều kiện lỗi của `Guess::new` thực sự xảy ra khi ta mong đợi. 

<Listing number="11-8" file-name="src/lib.rs" caption="Test rằng một điều kiện nào đó sẽ gây ra `panic!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-08/src/lib.rs}}
```

</Listing>

Chúng ta đặt attribute `#[should_panic]` sau attribute `#[test]` và trước test function mà nó áp dụng. Hãy xem kết quả khi test này pass:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-08/output.txt}}
```

Mọi thứ ổn. Giờ hãy cố tình đưa bug vào code bằng cách bỏ điều kiện khiến function `new` panic khi giá trị lớn hơn 100:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/src/lib.rs:here}}
```

Khi chạy test ở Listing 11-8, nó sẽ fail:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/output.txt}}
```

Trong trường hợp này, chúng ta không nhận được một message quá hữu ích, nhưng khi nhìn vào test function, ta thấy nó được annotate với `#[should_panic]`. Failure vừa nhận được có nghĩa là code trong test function đã không gây panic. 

Các test dùng `should_panic` có thể thiếu chính xác. Một test `should_panic` vẫn có thể pass ngay cả khi nó panic vì một lý do khác với điều ta mong đợi. Để làm cho test `should_panic` chính xác hơn, ta có thể thêm optional parameter `expected` vào attribute `should_panic`. Test harness sẽ kiểm tra để chắc chắn rằng failure message có chứa đoạn text được cung cấp. Ví dụ, hãy xem đoạn code đã chỉnh sửa cho `Guess` trong Listing 11-9, nơi function `new` panic với các message khác nhau tùy vào việc giá trị quá nhỏ hay quá lớn. 

<Listing number="11-9" file-name="src/lib.rs" caption="Test một `panic!` với panic message chứa một substring được chỉ định">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-09/src/lib.rs:here}}
```

</Listing>

Test này sẽ pass vì giá trị ta đặt trong parameter `expected` của attribute `should_panic` là một substring của message mà `Guess::new` panic ra. Ta cũng có thể chỉ định toàn bộ panic message mong đợi, trong trường hợp này là `Guess value must be less than or equal to 100, got 200`. Bạn chọn chỉ định bao nhiêu phụ thuộc vào mức độ unique hoặc dynamic của panic message và độ chính xác mà bạn muốn test đạt được. Trong trường hợp này, chỉ cần một substring là đủ để đảm bảo code trong test function đi vào nhánh `else if value > 100`. 

Để xem chuyện gì xảy ra khi một test `should_panic` có message `expected` bị fail, hãy lại cố tình đưa bug vào code bằng cách hoán đổi thân của khối `if value < 1` và `else if value > 100`:

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/src/lib.rs:here}}
```

Lần này khi chạy test `should_panic`, nó sẽ fail:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/output.txt}}
```

Failure message cho biết rằng test này đúng là đã panic như mong đợi, nhưng panic message lại không chứa chuỗi mong đợi `less than or equal to 100`. Panic message thực tế nhận được trong trường hợp này là `Guess value must be greater than or equal to 1, got 200`. Giờ ta có thể bắt đầu lần ra bug nằm ở đâu. 

### Dùng `Result<T, E>` trong test

Toàn bộ test từ đầu đến giờ đều panic khi fail. Nhưng ta cũng có thể viết test dùng `Result<T, E>`. Dưới đây là test trong Listing 11-1 được viết lại để dùng `Result<T, E>` và trả về `Err` thay vì panic:

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-10-result-in-tests/src/lib.rs:here}}
```

Function `it_works` giờ có return type là `Result<(), String>`. Trong thân function, thay vì gọi macro `assert_eq!`, ta trả về `Ok(())` khi test pass và trả về `Err` chứa một `String` khi test fail. 

Viết test theo kiểu trả về `Result<T, E>` cho phép bạn dùng toán tử question mark trong thân test, đây có thể là một cách rất tiện để viết các test cần fail nếu bất kỳ thao tác nào bên trong trả về `Err` variant. 

Bạn không thể dùng annotation `#[should_panic]` cho các test sử dụng `Result<T, E>`. Để assert rằng một thao tác trả về `Err` variant, _đừng_ dùng toán tử question mark trên giá trị `Result<T, E>`. Thay vào đó, hãy dùng `assert!(value.is_err())`. 

Giờ bạn đã biết vài cách khác nhau để viết test, hãy xem điều gì diễn ra khi chúng ta chạy test và khám phá các tùy chọn khác nhau có thể dùng với `cargo test`. 

[concatenating]: ch08-02-strings.html#concatenating-with--or-format
[bench]: ../unstable-book/library-features/test.html
[ignoring]: ch11-02-running-tests.html#ignoring-tests-unless-specifically-requested
[subset]: ch11-02-running-tests.html#running-a-subset-of-tests-by-name
[controlling-how-tests-are-run]: ch11-02-running-tests.html#controlling-how-tests-are-run
[derivable-traits]: appendix-03-derivable-traits.html
[doc-comments]: ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests
[paths-for-referring-to-an-item-in-the-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html