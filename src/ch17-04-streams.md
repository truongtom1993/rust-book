<!-- Old headings. Do not remove or links may break. -->

<a id="streams"></a>

## Streams: Futures Trong Sequence

Nhớ lại cách chúng ta đã sử dụng receiver cho async channel của chúng ta trước đó trong chương này trong phần ["Message Passing"][17-02-messages]<!-- ignore -->. Phương thức async `recv` tạo ra một sequence của items qua thời gian. Đây là một instance của một pattern tổng quát hơn nhiều được biết dưới dạng một _stream_. Nhiều concepts một cách tự nhiên được đại diện như streams: items trở thành available trong một queue, chunks của dữ liệu được kéo incrementally từ filesystem khi toàn bộ dữ liệu set quá lớn cho bộ nhớ của máy tính, hoặc dữ liệu tới qua network qua thời gian. Bởi vì streams là futures, chúng ta có thể sử dụng chúng với bất kỳ loại future khác và kết hợp chúng trong cách thú vị. Ví dụ, chúng ta có thể batch up events để tránh triggering quá nhiều network calls, set timeouts trên sequences của long-running operations, hoặc throttle user interface events để tránh làm needless work.

Chúng ta thấy một sequence của items ở lại ở Chương 13, khi chúng ta đã nhìn vào Iterator trait trong phần ["The Iterator Trait and the `next` Method"][iterator-trait]<!-- ignore -->, nhưng có hai differences giữa iterators và async channel receiver. Difference đầu tiên là time: iterators là đồng bộ, trong khi channel receiver là asynchronous. Difference thứ hai là API. Khi làm việc trực tiếp với `Iterator`, chúng ta gọi phương thức `next` đồng bộ của nó. Với `trpl::Receiver` stream đặc biệt, chúng ta gọi một phương thức `recv` không đồng bộ thay vào đó. Ngoài ra, APIs này cảm thấy rất giống nhau, và similarity đó không phải là một coincidence. Một stream giống như một asynchronous form của iteration. Whereas `trpl::Receiver` specifically chờ đợi để nhận messages, tuy nhiên, general-purpose stream API rộng hơn nhiều: nó cung cấp next item như `Iterator` làm, nhưng asynchronously.

Similarity giữa iterators và streams trong Rust có nghĩa là chúng ta thực sự có thể tạo một stream từ bất kỳ iterator nào. Như với một iterator, chúng ta có thể làm việc với một stream bằng cách gọi phương thức `next` của nó và sau đó awaiting output, như trong Listing 17-21, sẽ không biên dịch chưa.

<Listing number="17-21" caption="Tạo một stream từ một iterator và printing các values của nó" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-21/src/main.rs:stream}}
```

</Listing>

Chúng ta bắt đầu với một array của numbers, mà chúng ta chuyển đổi thành một iterator và sau đó gọi `map` để double tất cả các values. Sau đó chúng ta chuyển đổi iterator vào một stream sử dụng function `trpl::stream_from_iter`. Tiếp theo, chúng ta loop qua các items trong stream khi chúng tới với `while let` loop.

Thật không may, khi chúng ta cố gắng chạy mã, nó không biên dịch nhưng thay vào đó reports rằng không có `next` method có sẵn:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-21
cargo build
copy only the error output
-->

```text
error[E0599]: no method named `next` found for struct `tokio_stream::iter::Iter` in the current scope
  --> src/main.rs:10:40
   |
10 |         while let Some(value) = stream.next().await {
   |                                        ^^^^
   |
   = help: items from traits can only be used if the trait is in scope
help: the following traits which provide `next` are implemented but not in scope; perhaps you want to import one of them
   |
1  + use crate::trpl::StreamExt;
   |
1  + use futures_util::stream::stream::StreamExt;
   |
1  + use std::iter::Iterator;
   |
1  + use std::str::pattern::Searcher;
   |
help: there is a method `try_next` with a similar name
   |
10 |         while let Some(value) = stream.try_next().await {
   |                                        ~~~~~~~~
```

Như output này giải thích, lý do cho compiler error là chúng ta cần right trait trong scope để có thể sử dụng phương thức `next`. Được đưa ra thảo luận của chúng ta cho đến nay, bạn có thể reasonably expect rằng trait đó sẽ là `Stream`, nhưng nó thực sự là `StreamExt`. Viết tắt cho _extension_, `Ext` là một common pattern trong Rust community cho extending một trait với một trait khác.

Trait `Stream` định nghĩa một low-level interface mà effectively kết hợp `Iterator` và `Future` traits. `StreamExt` cung cấp một higher-level tập hợp của APIs trên đầu `Stream`, bao gồm phương thức `next` cũng như utility methods khác tương tự như những được cung cấp bởi trait `Iterator`. `Stream` và `StreamExt` chưa phần của Rust's standard library, nhưng hầu hết ecosystem crates sử dụng similar definitions.

Fix cho compiler error là để thêm một `use` statement cho `trpl::StreamExt`, như trong Listing 17-22.

<Listing number="17-22" caption="Successfully sử dụng một iterator như là basis cho một stream" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-22/src/main.rs:all}}
```

</Listing>

Với tất cả những mảnh ghép đó đặt với nhau, mã này hoạt động theo cách chúng ta muốn! Hơn nữa, bây giờ chúng ta có `StreamExt` trong scope, chúng ta có thể sử dụng tất cả utility methods của nó, giống như với iterators.

[17-02-messages]: ch17-02-concurrency-with-async.html#message-passing
[iterator-trait]: ch13-02-iterators.html#the-iterator-trait-and-the-next-method
