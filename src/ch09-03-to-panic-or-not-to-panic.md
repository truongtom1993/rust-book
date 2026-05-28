## Có Nên `panic!` hay Không `panic!`

Vậy, bạn làm cách nào để quyết định khi nào bạn nên gọi `panic!` và khi nào bạn nên trả về `Result`? Khi mã panic, không có cách nào để phục hồi. Bạn có thể gọi `panic!` cho bất kỳ tình huống lỗi nào, cho dù có cách phục hồi hay không, nhưng khi đó bạn đang đưa ra quyết định rằng một tình huống là không thể phục hồi thay mặt cho mã gọi. Khi bạn chọn trả về giá trị `Result`, bạn cung cấp các lựa chọn cho mã gọi. Mã gọi có thể chọn cố gắng phục hồi theo cách thích hợp cho tình huống của nó, hoặc nó có thể quyết định rằng giá trị `Err` trong trường hợp này là không thể phục hồi, vì vậy nó có thể gọi `panic!` và biến lỗi có thể phục hồi của bạn thành lỗi không thể phục hồi. Do đó, trả về `Result` là lựa chọn mặc định tốt khi bạn định nghĩa một function có thể thất bại.

Trong các tình huống như ví dụ, mã prototype, và tests, sẽ thích hợp hơn để viết mã panic thay vì trả về `Result`. Hãy tìm hiểu lý do tại sao, sau đó thảo luận về những tình huống mà trình biên dịch không thể biết được rằng failure là không thể xảy ra, nhưng bạn là một người có thể. Chương sẽ kết thúc với một số hướng dẫn chung về cách quyết định có nên panic trong library code.

### Ví Dụ, Mã Prototype, và Tests

Khi bạn viết một ví dụ để minh họa một khái niệm nào đó, việc bao gồm cả mã xử lý lỗi mạnh mẽ cũng có thể làm cho ví dụ kém rõ ràng hơn. Trong các ví dụ, người ta hiểu rằng lệnh gọi đến một method như `unwrap` có thể panic là được dùng làm placeholder cho cách bạn muốn ứng dụng của bạn xử lý các lỗi, điều có thể khác nhau tùy thuộc vào những gì phần còn lại của mã của bạn đang làm.

Tương tự, các methods `unwrap` và `expect` rất tiện lợi khi bạn đang prototype và bạn chưa sẵn sàng quyết định cách xử lý các lỗi. Chúng để lại các dấu hiệu rõ ràng trong mã của bạn cho khi bạn sẵn sàng làm cho chương trình của bạn mạnh mẽ hơn.

Nếu lệnh gọi method thất bại trong một test, bạn muốn toàn bộ test thất bại, ngay cả khi method đó không phải là chức năng được kiểm tra. Vì `panic!` là cách một test được đánh dấu là thất bại, gọi `unwrap` hoặc `expect` là chính xác những gì sẽ xảy ra.

<!-- Old headings. Do not remove or links may break. -->

<a id="cases-in-which-you-have-more-information-than-the-compiler"></a>

### Khi Bạn Có Nhiều Thông Tin Hơn Trình Biên Dịch

Cũng sẽ thích hợp để gọi `expect` khi bạn có một số logic khác đảm bảo rằng `Result` sẽ có giá trị `Ok`, nhưng logic đó không phải là điều mà trình biên dịch hiểu. Bạn vẫn sẽ có giá trị `Result` mà bạn cần xử lý: Bất kỳ hoạt động nào bạn gọi vẫn có khả năng thất bại nói chung, ngay cả khi logic sẽ không thể xảy ra trong tình huống cụ thể của bạn. Nếu bạn có thể đảm bảo bằng cách kiểm tra thủ công mã rằng bạn sẽ không bao giờ có biến thể `Err`, hoàn toàn có thể chấp nhận được để gọi `expect` và ghi lại lý do bạn nghĩ rằng bạn sẽ không bao giờ có biến thể `Err` trong văn bản argument. Dưới đây là một ví dụ:

```rust
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-08-unwrap-that-cant-fail/src/main.rs:here}}
```

Chúng ta đang tạo một instance `IpAddr` bằng cách phân tích chuỗi được mã hóa cứng. Chúng ta có thể thấy rằng `127.0.0.1` là một địa chỉ IP hợp lệ, vì vậy chấp nhận được để sử dụng `expect` tại đây. Tuy nhiên, có một chuỗi được mã hóa cứng và hợp lệ không thay đổi kiểu trả về của method `parse`: Chúng ta vẫn nhận được giá trị `Result`, và trình biên dịch vẫn sẽ làm cho chúng ta xử lý `Result` như thể biến thể `Err` là một khả năng vì trình biên dịch không đủ thông minh để thấy rằng chuỗi này luôn là một địa chỉ IP hợp lệ. Nếu chuỗi địa chỉ IP đến từ một người dùng thay vì được mã hóa cứng vào chương trình và do đó _đã_ có khả năng thất bại, chúng ta chắc chắn muốn xử lý `Result` theo cách mạnh mẽ hơn thay vào đó. Đề cập đến giả định rằng địa chỉ IP này được mã hóa cứng sẽ nhắc chúng ta thay đổi `expect` thành mã xử lý lỗi tốt hơn nếu, trong tương lai, chúng ta cần lấy địa chỉ IP từ một nguồn khác thay vào đó.

### Guidelines for Error Handling

It’s advisable to have your code panic when it’s possible that your code could
end up in a bad state. In this context, a _bad state_ is when some assumption,
guarantee, contract, or invariant has been broken, such as when invalid values,
contradictory values, or missing values are passed to your code—plus one or
more of the following:

- The bad state is something that is unexpected, as opposed to something that
  will likely happen occasionally, like a user entering data in the wrong
  format.
- Your code after this point needs to rely on not being in this bad state,
  rather than checking for the problem at every step.
- There’s not a good way to encode this information in the types you use. We’ll
  work through an example of what we mean in [“Encoding States and Behavior as
  Types”][encoding]<!-- ignore --> in Chapter 18.

If someone calls your code and passes in values that don’t make sense, it’s
best to return an error if you can so that the user of the library can decide
what they want to do in that case. However, in cases where continuing could be
insecure or harmful, the best choice might be to call `panic!` and alert the
person using your library to the bug in their code so that they can fix it
during development. Similarly, `panic!` is often appropriate if you’re calling
external code that is out of your control and returns an invalid state that you
have no way of fixing.

However, when failure is expected, it’s more appropriate to return a `Result`
than to make a `panic!` call. Examples include a parser being given malformed
data or an HTTP request returning a status that indicates you have hit a rate
limit. In these cases, returning a `Result` indicates that failure is an
expected possibility that the calling code must decide how to handle.

When your code performs an operation that could put a user at risk if it’s
called using invalid values, your code should verify the values are valid first
and panic if the values aren’t valid. This is mostly for safety reasons:
Attempting to operate on invalid data can expose your code to vulnerabilities.
This is the main reason the standard library will call `panic!` if you attempt
an out-of-bounds memory access: Trying to access memory that doesn’t belong to
the current data structure is a common security problem. Functions often have
_contracts_: Their behavior is only guaranteed if the inputs meet particular
requirements. Panicking when the contract is violated makes sense because a
contract violation always indicates a caller-side bug, and it’s not a kind of
error you want the calling code to have to explicitly handle. In fact, there’s
no reasonable way for calling code to recover; the calling _programmers_ need
to fix the code. Contracts for a function, especially when a violation will
cause a panic, should be explained in the API documentation for the function.

However, having lots of error checks in all of your functions would be verbose
and annoying. Fortunately, you can use Rust’s type system (and thus the type
checking done by the compiler) to do many of the checks for you. If your
function has a particular type as a parameter, you can proceed with your code’s
logic knowing that the compiler has already ensured that you have a valid
value. For example, if you have a type rather than an `Option`, your program
expects to have _something_ rather than _nothing_. Your code then doesn’t have
to handle two cases for the `Some` and `None` variants: It will only have one
case for definitely having a value. Code trying to pass nothing to your
function won’t even compile, so your function doesn’t have to check for that
case at runtime. Another example is using an unsigned integer type such as
`u32`, which ensures that the parameter is never negative.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-custom-types-for-validation"></a>

### Custom Types for Validation

Let’s take the idea of using Rust’s type system to ensure that we have a valid
value one step further and look at creating a custom type for validation.
Recall the guessing game in Chapter 2 in which our code asked the user to guess
a number between 1 and 100. We never validated that the user’s guess was
between those numbers before checking it against our secret number; we only
validated that the guess was positive. In this case, the consequences were not
very dire: Our output of “Too high” or “Too low” would still be correct. But it
would be a useful enhancement to guide the user toward valid guesses and have
different behavior when the user guesses a number that’s out of range versus
when the user types, for example, letters instead.

One way to do this would be to parse the guess as an `i32` instead of only a
`u32` to allow potentially negative numbers, and then add a check for the
number being in range, like so:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-09-guess-out-of-range/src/main.rs:here}}
```

</Listing>

The `if` expression checks whether our value is out of range, tells the user
about the problem, and calls `continue` to start the next iteration of the loop
and ask for another guess. After the `if` expression, we can proceed with the
comparisons between `guess` and the secret number knowing that `guess` is
between 1 and 100.

However, this is not an ideal solution: If it were absolutely critical that the
program only operated on values between 1 and 100, and it had many functions
with this requirement, having a check like this in every function would be
tedious (and might impact performance).

Instead, we can make a new type in a dedicated module and put the validations
in a function to create an instance of the type rather than repeating the
validations everywhere. That way, it’s safe for functions to use the new type
in their signatures and confidently use the values they receive. Listing 9-13
shows one way to define a `Guess` type that will only create an instance of
`Guess` if the `new` function receives a value between 1 and 100.

<Listing number="9-13" caption="A `Guess` type that will only continue with values between 1 and 100" file-name="src/guessing_game.rs">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-13/src/guessing_game.rs}}
```

</Listing>

Note that this code in *src/guessing_game.rs* depends on adding a module
declaration `mod guessing_game;` in *src/lib.rs* that we haven’t shown here.
Within this new module’s file, we define a struct named `Guess` that has a
field named `value` that holds an `i32`. This is where the number will be
stored.

Then, we implement an associated function named `new` on `Guess` that creates
instances of `Guess` values. The `new` function is defined to have one
parameter named `value` of type `i32` and to return a `Guess`. The code in the
body of the `new` function tests `value` to make sure it’s between 1 and 100.
If `value` doesn’t pass this test, we make a `panic!` call, which will alert
the programmer who is writing the calling code that they have a bug they need
to fix, because creating a `Guess` with a `value` outside this range would
violate the contract that `Guess::new` is relying on. The conditions in which
`Guess::new` might panic should be discussed in its public-facing API
documentation; we’ll cover documentation conventions indicating the possibility
of a `panic!` in the API documentation that you create in Chapter 14. If
`value` does pass the test, we create a new `Guess` with its `value` field set
to the `value` parameter and return the `Guess`.

Next, we implement a method named `value` that borrows `self`, doesn’t have any
other parameters, and returns an `i32`. This kind of method is sometimes called
a _getter_ because its purpose is to get some data from its fields and return
it. This public method is necessary because the `value` field of the `Guess`
struct is private. It’s important that the `value` field be private so that
code using the `Guess` struct is not allowed to set `value` directly: Code
outside the `guessing_game` module _must_ use the `Guess::new` function to
create an instance of `Guess`, thereby ensuring that there’s no way for a
`Guess` to have a `value` that hasn’t been checked by the conditions in the
`Guess::new` function.

A function that has a parameter or returns only numbers between 1 and 100 could
then declare in its signature that it takes or returns a `Guess` rather than an
`i32` and wouldn’t need to do any additional checks in its body.

## Summary

Rust’s error-handling features are designed to help you write more robust code.
The `panic!` macro signals that your program is in a state it can’t handle and
lets you tell the process to stop instead of trying to proceed with invalid or
incorrect values. The `Result` enum uses Rust’s type system to indicate that
operations might fail in a way that your code could recover from. You can use
`Result` to tell code that calls your code that it needs to handle potential
success or failure as well. Using `panic!` and `Result` in the appropriate
situations will make your code more reliable in the face of inevitable problems.

Now that you’ve seen useful ways that the standard library uses generics with
the `Option` and `Result` enums, we’ll talk about how generics work and how you
can use them in your code.

[encoding]: ch18-03-oo-design-patterns.html#encoding-states-and-behavior-as-types
