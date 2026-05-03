## Advanced Traits

Chúng ta lần đầu tiên đề cập đến traits trong phần ["Defining Shared Behavior with Traits"][traits]<!-- ignore --> trong Chương 10, nhưng chúng ta không thảo luận chi tiết nâng cao. Bây giờ bạn biết nhiều hơn về Rust, chúng ta có thể đi vào chi tiết.

<!-- Old headings. Do not remove or links may break. -->

<a id="specifying-placeholder-types-in-trait-definitions-with-associated-types"></a>
<a id="associated-types"></a>

### Defining Traits with Associated Types

_Associated types_ kết nối một placeholder kiểu với một trait sao cho các định nghĩa method của trait có thể sử dụng những placeholder kiểu này trong signature của chúng. Người triển khai trait sẽ chỉ định loại cụ thể để sử dụng thay cho placeholder kiểu cho việc triển khai cụ thể. Bằng cách đó, chúng ta có thể định nghĩa một trait mà sử dụng một số kiểu mà không cần phải biết chính xác những kiểu đó là gì cho đến khi trait được triển khai.

Chúng ta đã mô tả hầu hết các tính năng nâng cao trong chương này như ít được cần đến. Associated types nằm ở đâu đó ở giữa: Chúng được sử dụng hiếm hơn so với các tính năng được giải thích trong phần còn lại của sách nhưng thường xuyên hơn so với nhiều tính năng khác được thảo luận trong chương này.

Một ví dụ của trait với associated type là `Iterator` trait mà thư viện chuẩn cung cấp. Associated type được đặt tên là `Item` và đại diện cho loại giá trị mà kiểu triển khai trait `Iterator` đang lặp lại. Định nghĩa của trait `Iterator` được hiển thị trong Listing 20-13.

<Listing number="20-13" caption="The definition of the `Iterator` trait that has an associated type `Item`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-13/src/lib.rs}}
```

</Listing>

Kiểu `Item` là một placeholder, và định nghĩa method `next` cho thấy rằng nó sẽ trả về giá trị của kiểu `Option<Self::Item>`. Người triển khai trait `Iterator` sẽ chỉ định loại cụ thể cho `Item`, và method `next` sẽ trả về một `Option` chứa giá trị của loại cụ thể đó.

Associated types có vẻ giống như một khái niệm tương tự như generics, ở chỗ cái sau cho phép chúng ta định nghĩa một function mà không chỉ định những kiểu nào mà nó có thể xử lý. Để kiểm tra sự khác biệt giữa hai khái niệm, chúng ta sẽ xem xét việc triển khai trait `Iterator` trên một loại được đặt tên là `Counter` mà chỉ định loại `Item` là `u32`:

<Listing file-name="src/lib.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-22-iterator-on-counter/src/lib.rs:ch19}}
```

</Listing>

Cú pháp này dường như có thể so sánh được với cách generics. Vậy tại sao không chỉ định trait `Iterator` với generics, như được hiển thị trong Listing 20-14?

<Listing number="20-14" caption="A hypothetical definition of the `Iterator` trait using generics">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-14/src/lib.rs}}
```

</Listing>

Sự khác biệt là khi sử dụng generics, như trong Listing 20-14, chúng ta phải chú thích những kiểu trong mỗi triển khai; vì chúng ta cũng có thể triển khai `Iterator<String> for Counter` hoặc bất kỳ loại nào khác, chúng ta có thể có nhiều triển khai của `Iterator` cho `Counter`. Nói cách khác, khi một trait có một tham số generic, nó có thể được triển khai cho một loại nhiều lần, thay đổi những kiểu cụ thể của các tham số kiểu generic mỗi lần. Khi chúng ta sử dụng method `next` trên `Counter`, chúng ta sẽ phải cung cấp chú thích kiểu để biểu thị triển khai nào của `Iterator` chúng ta muốn sử dụng.

Với associated types, chúng ta không cần chú thích các kiểu, vì chúng ta không thể triển khai một trait trên một loại nhiều lần. Trong Listing 20-13 với định nghĩa mà sử dụng associated types, chúng ta có thể chọn loại `Item` sẽ chỉ được một lần vì chỉ có thể có một `impl Iterator for Counter`. Chúng ta không phải chỉ định rằng chúng ta muốn một iterator của các giá trị `u32` ở mọi nơi chúng ta gọi `next` trên `Counter`.

Associated types cũng trở thành một phần của hợp đồng của trait: Người triển khai trait phải cung cấp một kiểu để đứng vào placeholder associated type. Associated types thường có một tên mô tả cách kiểu sẽ được sử dụng, và việc ghi chép associated type trong tài liệu API là một phương pháp tốt.

<!-- Old headings. Do not remove or links may break. -->

<a id="default-generic-type-parameters-and-operator-overloading"></a>

### Using Default Generic Parameters and Operator Overloading

Khi chúng ta sử dụng generic type parameters, chúng ta có thể chỉ định một loại cụ thể mặc định cho generic type. Điều này loại bỏ nhu cầu người triển khai trait phải chỉ định một loại cụ thể nếu loại mặc định hoạt động. Bạn chỉ định một loại mặc định khi khai báo một generic type với cú pháp `<PlaceholderType=ConcreteType>`.

Một ví dụ tuyệt vời của tình huống mà kỹ thuật này hữu ích là với _operator overloading_, mà trong đó bạn tùy chỉnh hành vi của một toán tử (chẳng hạn như `+`) trong những tình huống cụ thể.

Rust không cho phép bạn tạo toán tử của riêng bạn hoặc overload các toán tử tùy ý. Nhưng bạn có thể overload các hoạt động và những trait tương ứng được liệt kê trong `std::ops` bằng cách triển khai những trait được liên kết với toán tử. Ví dụ, trong Listing 20-15, chúng ta overload toán tử `+` để thêm hai instance `Point` với nhau. Chúng ta làm điều này bằng cách triển khai trait `Add` trên struct `Point`.

<Listing number="20-15" file-name="src/main.rs" caption="Implementing the `Add` trait to overload the `+` operator for `Point` instances">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-15/src/main.rs}}
```

</Listing>

Method `add` thêm giá trị `x` của hai instance `Point` và những giá trị `y` của hai instance `Point` để tạo một `Point` mới. Trait `Add` có một associated type được đặt tên là `Output` mà xác định loại được trả về từ method `add`.

Generic type mặc định trong code này nằm trong trait `Add`. Dưới đây là định nghĩa của nó:

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

Code này nên nhìn chung quen thuộc: một trait với một method và một associated type. Phần mới là `Rhs=Self`: Cú pháp này được gọi là _default type parameters_. Generic type parameter `Rhs` (viết tắt của "right-hand side") xác định loại của tham số `rhs` trong method `add`. Nếu chúng ta không chỉ định một loại cụ thể cho `Rhs` khi chúng ta triển khai trait `Add`, loại của `Rhs` sẽ mặc định là `Self`, sẽ là loại chúng ta đang triển khai `Add` trên.

Khi chúng ta triển khai `Add` cho `Point`, chúng ta đã sử dụng default cho `Rhs` vì chúng ta muốn thêm hai instance `Point`. Hãy xem xét một ví dụ về việc triển khai trait `Add` mà chúng ta muốn tùy chỉnh loại `Rhs` thay vì sử dụng default.

Chúng ta có hai structs, `Millimeters` và `Meters`, giữ những giá trị trong những đơn vị khác nhau. Wrapper mỏng này của một loại hiện có trong một struct khác được gọi là _newtype pattern_, mà chúng ta mô tả chi tiết hơn trong phần ["Implementing External Traits with the Newtype Pattern"][newtype]<!-- ignore -->. Chúng ta muốn thêm những giá trị trong millimet vào những giá trị trong meter và có việc triển khai `Add` thực hiện việc chuyển đổi một cách chính xác. Chúng ta có thể triển khai `Add` cho `Millimeters` với `Meters` như `Rhs`, như được hiển thị trong Listing 20-16.

<Listing number="20-16" file-name="src/lib.rs" caption="Implementing the `Add` trait on `Millimeters` to add `Millimeters` and `Meters`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-16/src/lib.rs}}
```

</Listing>

Để thêm `Millimeters` và `Meters`, chúng ta chỉ định `impl Add<Meters>` để đặt giá trị của tham số `Rhs` type thay vì sử dụng default của `Self`.

Bạn sẽ sử dụng default type parameters theo hai cách chính:

1. Để mở rộng một loại mà không phá vỡ code hiện có
2. Để cho phép tùy chỉnh trong những trường hợp cụ thể mà hầu hết người dùng không cần

Trait `Add` của thư viện chuẩn là một ví dụ của mục đích thứ hai: Thông thường, bạn sẽ thêm hai loại giống nhau, nhưng trait `Add` cung cấp khả năng tùy chỉnh vượt quá điều đó. Sử dụng một tham số kiểu mặc định trong định nghĩa trait `Add` có nghĩa là bạn không phải chỉ định tham số thêm hầu hết thời gian. Nói cách khác, một chút boilerplate thực hiện không được cần thiết, làm cho nó dễ dàng hơn để sử dụng trait.

Mục đích đầu tiên tương tự như mục đích thứ hai nhưng theo hướng ngược lại: Nếu bạn muốn thêm một tham số kiểu cho một trait hiện có, bạn có thể cung cấp một default để cho phép mở rộng chức năng của trait mà không phá vỡ code triển khai hiện có.

<!-- Old headings. Do not remove or links may break. -->

<a id="fully-qualified-syntax-for-disambiguation-calling-methods-with-the-same-name"></a>
<a id="disambiguating-between-methods-with-the-same-name"></a>

### Disambiguating Between Identically Named Methods

Không có gì trong Rust ngăn chặn một trait từ việc có một method có cùng tên với method của một trait khác, Rust cũng không ngăn chặn bạn từ việc triển khai cả hai trait trên một loại. Cũng có thể triển khai một method trực tiếp trên loại với cùng tên như những method từ traits.

Khi gọi những method có cùng tên, bạn sẽ cần nói với Rust cái mà bạn muốn sử dụng. Xem xét code trong Listing 20-17 mà chúng ta đã định nghĩa hai traits, `Pilot` và `Wizard`, mà cả hai đều có một method được gọi là `fly`. Chúng ta sau đó triển khai cả hai traits trên một loại `Human` mà đã có một method được đặt tên là `fly` được triển khai trên nó. Mỗi method `fly` làm cái gì đó khác nhau.

<Listing number="20-17" file-name="src/main.rs" caption="Two traits are defined to have a `fly` method and are implemented on the `Human` type, and a `fly` method is implemented on `Human` directly.">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-17/src/main.rs:here}}
```

</Listing>

Khi chúng ta gọi `fly` trên một instance của `Human`, trình biên dịch mặc định gọi method được triển khai trực tiếp trên loại, như được hiển thị trong Listing 20-18.

<Listing number="20-18" file-name="src/main.rs" caption="Calling `fly` on an instance of `Human`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-18/src/main.rs:here}}
```

</Listing>

Chạy code này sẽ in `*waving arms furiously*`, cho thấy rằng Rust đã gọi method `fly` được triển khai trên `Human` trực tiếp.

Để gọi những method `fly` từ một trong hai traits `Pilot` hoặc `Wizard`, chúng ta cần sử dụng cú pháp rõ ràng hơn để chỉ định phương pháp `fly` nào chúng ta có ý. Listing 20-19 trình bày cú pháp này.

<Listing number="20-19" file-name="src/main.rs" caption="Specifying which trait's `fly` method we want to call">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-19/src/main.rs:here}}
```

</Listing>

Chỉ định tên trait trước tên method làm rõ cho Rust triển khai `fly` nào chúng ta muốn gọi. Chúng ta cũng có thể viết `Human::fly(&person)`, cái mà tương đương với `person.fly()` mà chúng ta đã sử dụng trong Listing 20-19, nhưng cái này dài hơn một chút để viết nếu chúng ta không cần phải phân biệt.

Chạy code này in những cái sau:

```console
{{#include ../listings/ch20-advanced-features/listing-20-19/output.txt}}
```

Vì method `fly` lấy một tham số `self`, nếu chúng ta có hai _types_ mà cả hai triển khai một _trait_, Rust có thể hình ra triển khai nào của một trait để sử dụng dựa trên loại của `self`.

Tuy nhiên, những associated functions mà không phải là methods không có một tham số `self`. Khi có nhiều loại hoặc traits mà định nghĩa những hàm không phải là method với cùng tên function, Rust không phải lúc nào cũng biết loại nào bạn có ý trừ khi bạn sử dụng fully qualified syntax. Ví dụ, trong Listing 20-20, chúng ta tạo một trait cho một nơi trú ẩn động vật mà muốn đặt tên tất cả những chó con là Spot. Chúng ta tạo một trait `Animal` với một associated non-method function `baby_name`. Trait `Animal` được triển khai cho struct `Dog`, trên đó chúng ta cũng cung cấp một associated non-method function `baby_name` trực tiếp.

<Listing number="20-20" file-name="src/main.rs" caption="A trait with an associated function and a type with an associated function of the same name that also implements the trait">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-20/src/main.rs}}
```

</Listing>

Chúng ta triển khai code để đặt tên tất cả những con chó con là Spot trong associated function `baby_name` mà được định nghĩa trên `Dog`. Loại `Dog` cũng triển khai trait `Animal`, mà mô tả những đặc điểm mà tất cả động vật có. Những chó con được gọi là puppies, và điều đó được thể hiện trong việc triển khai trait `Animal` trên `Dog` trong function `baby_name` liên kết với trait `Animal`.

Trong `main`, chúng ta gọi function `Dog::baby_name`, cái mà gọi associated function được định nghĩa trên `Dog` trực tiếp. Code này in những cái sau:

```console
{{#include ../listings/ch20-advanced-features/listing-20-20/output.txt}}
```

Output này không phải là cái chúng ta muốn. Chúng ta muốn gọi function `baby_name` mà là một phần của trait `Animal` mà chúng ta triển khai trên `Dog` để code in `A baby dog is called a puppy`. Kỹ thuật của việc chỉ định tên trait mà chúng ta đã sử dụng trong Listing 20-19 không giúp ở đây; nếu chúng ta thay đổi `main` cho code trong Listing 20-21, chúng ta sẽ nhận được một lỗi compilation.

<Listing number="20-21" file-name="src/main.rs" caption="Attempting to call the `baby_name` function from the `Animal` trait, but Rust doesn't know which implementation to use">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-21/src/main.rs:here}}
```

</Listing>

Vì `Animal::baby_name` không có một tham số `self`, và có thể có những loại khác mà triển khai trait `Animal`, Rust không thể hình ra triển khai nào của `Animal::baby_name` chúng ta muốn. Chúng ta sẽ nhận được lỗi trình biên dịch này:

```console
{{#include ../listings/ch20-advanced-features/listing-20-21/output.txt}}
```

Để phân biệt và nói với Rust rằng chúng ta muốn sử dụng việc triển khai của `Animal` cho `Dog` đối với việc triển khai của `Animal` cho một số loại khác, chúng ta cần sử dụng fully qualified syntax. Listing 20-22 trình bày cách sử dụng fully qualified syntax.

<Listing number="20-22" file-name="src/main.rs" caption="Using fully qualified syntax to specify that we want to call the `baby_name` function from the `Animal` trait as implemented on `Dog`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-22/src/main.rs:here}}
```

</Listing>

Chúng ta cung cấp Rust với chú thích kiểu trong những dấu ngoặc nhọn, mà biểu thị chúng ta muốn gọi method `baby_name` từ trait `Animal` như được triển khai trên `Dog` bằng cách nói rằng chúng ta muốn xem loại `Dog` như một `Animal` cho việc gọi function này. Code này sẽ bây giờ in những cái chúng ta muốn:

```console
{{#include ../listings/ch20-advanced-features/listing-20-22/output.txt}}
```

Nói chung, fully qualified syntax được định nghĩa như sau:

```rust,ignore
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

Với những associated functions mà không phải là methods, sẽ không có một `receiver`: Sẽ chỉ có danh sách những đối số khác. Bạn có thể sử dụng fully qualified syntax ở mọi nơi mà bạn gọi functions hoặc methods. Tuy nhiên, bạn được phép để bỏ qua bất kỳ phần nào của cú pháp này mà Rust có thể hình ra từ những thông tin khác trong chương trình. Bạn chỉ cần sử dụng cú pháp chi tiết hơn này trong những trường hợp mà có nhiều triển khai mà sử dụng cùng tên và Rust cần giúp để xác định triển khai nào chúng ta muốn gọi.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-supertraits-to-require-one-traits-functionality-within-another-trait"></a>

### Using Supertraits

Tôi nói rằng bạn có thể viết một định nghĩa trait mà phụ thuộc vào một trait khác: Để một loại triển khai trait đầu tiên, bạn muốn yêu cầu loại đó cũng triển khai trait thứ hai. Bạn sẽ làm điều này để định nghĩa trait của bạn có thể sử dụng những associated items của trait thứ hai. Trait mà định nghĩa trait của bạn dựa vào được gọi là _supertrait_ của trait của bạn.

Ví dụ, chúng ta nói chúng ta muốn tạo một trait `OutlinePrint` với một method `outline_print` mà sẽ in một giá trị nhất định được định dạng để nó được khung trong dấu sao. Đó là, với một struct `Point` mà triển khai trait chuẩn `Display` để kết quả là `(x, y)`, khi chúng ta gọi `outline_print` trên một instance `Point` mà có `1` cho `x` và `3` cho `y`, nó sẽ in những cái sau:

```text
**********
*        *
* (1, 3) *
*        *
**********
```

Trong việc triển khai method `outline_print`, chúng ta muốn sử dụng chức năng của trait `Display`. Do đó, chúng ta cần chỉ định rằng trait `OutlinePrint` sẽ chỉ làm việc cho những loại mà cũng triển khai `Display` và cung cấp chức năng mà `OutlinePrint` cần. Chúng ta có thể làm điều đó trong định nghĩa trait bằng cách chỉ định `OutlinePrint: Display`. Kỹ thuật này tương tự như thêm một trait bound cho trait. Listing 20-23 cho thấy một triển khai của trait `OutlinePrint`.

<Listing number="20-23" file-name="src/main.rs" caption="Implementing the `OutlinePrint` trait that requires the functionality from `Display`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-23/src/main.rs:here}}
```

</Listing>

Vì chúng ta đã chỉ định rằng `OutlinePrint` yêu cầu trait `Display`, chúng ta có thể sử dụng function `to_string` mà được tự động triển khai cho bất kỳ loại nào mà triển khai `Display`. Nếu chúng ta cố gắng sử dụng `to_string` mà không thêm một dấu hai chấm và chỉ định trait `Display` sau tên trait, chúng ta sẽ nhận được một error nói rằng không có method được đặt tên là `to_string` được tìm thấy cho loại `&Self` trong phạm vi hiện tại.

Chúng ta hãy xem những gì xảy ra khi chúng ta cố gắng triển khai `OutlinePrint` trên một loại mà không triển khai `Display`, chẳng hạn như struct `Point`:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/src/main.rs:here}}
```

</Listing>

Chúng ta nhận được một error nói rằng `Display` được yêu cầu nhưng không được triển khai:

```console
{{#include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/output.txt}}
```

Để sửa chữa điều này, chúng ta triển khai `Display` trên `Point` và thoả mãn constraint mà `OutlinePrint` yêu cầu, như sau:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-03-impl-display-for-point/src/main.rs:here}}
```

</Listing>

Sau đó, việc triển khai trait `OutlinePrint` trên `Point` sẽ compile một cách thành công, và chúng ta có thể gọi `outline_print` trên một instance `Point` để hiển thị nó trong một outline của dấu sao.

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-newtype-pattern-to-implement-external-traits-on-external-types"></a>
<a id="using-the-newtype-pattern-to-implement-external-traits"></a>

### Implementing External Traits with the Newtype Pattern

Trong phần ["Implementing a Trait on a Type"][implementing-a-trait-on-a-type]<!-- ignore --> trong Chương 10, chúng ta đã đề cập đến orphan rule mà nói rằng chúng ta chỉ được phép triển khai một trait trên một loại nếu trait hoặc loại, hoặc cả hai, là địa phương đối với crate của chúng ta. Có thể để vượt qua hạn chế này bằng cách sử dụng newtype pattern, mà liên quan đến việc tạo một loại mới trong một tuple struct. (Chúng ta đã đề cập đến tuple structs trong phần ["Creating Different Types with Tuple Structs"][tuple-structs]<!-- ignore --> trong Chương 5.) Tuple struct sẽ có một field và là một wrapper mỏng xung quanh loại mà chúng ta muốn triển khai một trait. Sau đó, loại wrapper là địa phương đối với crate của chúng ta, và chúng ta có thể triển khai trait trên wrapper. _Newtype_ là một thuật ngữ mà bắt nguồn từ ngôn ngữ lập trình Haskell. Không có hình phạt hiệu suất thời gian chạy cho việc sử dụng pattern này, và loại wrapper được bỏ qua tại thời gian compile.

Ví dụ, chúng ta nói chúng ta muốn triển khai `Display` trên `Vec<T>`, mà orphan rule ngăn chặn chúng ta từ việc làm trực tiếp vì trait `Display` và loại `Vec<T>` được định nghĩa bên ngoài crate của chúng ta. Chúng ta có thể tạo một struct `Wrapper` mà giữ một instance của `Vec<T>`; sau đó, chúng ta có thể triển khai `Display` trên `Wrapper` và sử dụng giá trị `Vec<T>`, như được hiển thị trong Listing 20-24.

<Listing number="20-24" file-name="src/main.rs" caption="Creating a `Wrapper` type around `Vec<String>` to implement `Display`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-24/src/main.rs}}
```

</Listing>

Việc triển khai của `Display` sử dụng `self.0` để truy cập `Vec<T>` bên trong vì `Wrapper` là một tuple struct và `Vec<T>` là item tại chỉ số 0 trong tuple. Sau đó, chúng ta có thể sử dụng chức năng của trait `Display` trên `Wrapper`.

Nhược điểm của việc sử dụng kỹ thuật này là `Wrapper` là một loại mới, vì vậy nó không có những methods của giá trị mà nó đang giữ. Chúng ta sẽ phải triển khai tất cả những methods của `Vec<T>` trực tiếp trên `Wrapper` sao cho những methods ủy quyền cho `self.0`, cái mà sẽ cho phép chúng ta xem `Wrapper` chính xác giống như một `Vec<T>`. Nếu chúng ta muốn loại mới để có mọi method mà loại bên trong có, việc triển khai trait `Deref` trên `Wrapper` để trả về loại bên trong sẽ là một giải pháp (chúng ta đã thảo luận việc triển khai trait `Deref` trong phần ["Treating Smart Pointers Like Regular References"][smart-pointer-deref]<!-- ignore --> trong Chương 15). Nếu chúng ta không muốn loại `Wrapper` để có tất cả những methods của loại bên trong—ví dụ, để hạn chế hành vi của loại `Wrapper`—chúng ta sẽ phải triển khai những methods chúng ta muốn thủ công.

Newtype pattern này cũng hữu ích thậm chí khi traits không được liên quan. Chúng ta hãy chuyển tập trung và xem xét một số cách nâng cao để tương tác với hệ thống kiểu của Rust.

[newtype]: ch20-02-advanced-traits.html#implementing-external-traits-with-the-newtype-pattern
[implementing-a-trait-on-a-type]: ch10-02-traits.html#implementing-a-trait-on-a-type
[traits]: ch10-02-traits.html
[smart-pointer-deref]: ch15-02-deref.html#treating-smart-pointers-like-regular-references
[tuple-structs]: ch05-01-defining-structs.html#creating-different-types-with-tuple-structs
