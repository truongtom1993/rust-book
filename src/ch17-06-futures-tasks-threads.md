## Kết Hợp Tất Cả: Futures, Tasks, Và Threads

Như chúng ta thấy trong [Chương 16][ch16]<!-- ignore -->, threads cung cấp một approach tới concurrency. Chúng ta đã thấy một approach khác trong chương này: sử dụng async với futures và streams. Nếu bạn đang tự hỏi khi nào để chọn một method so với phương pháp khác, câu trả lời là: nó phụ thuộc! Và trong nhiều trường hợp, sự lựa chọn không phải là threads _or_ async nhưng thay vào đó threads _and_ async.

Nhiều hệ điều hành đã cung cấp threading-based concurrency models cho nhiều thập kỷ bây giờ, và nhiều ngôn ngữ lập trình hỗ trợ chúng như một kết quả. Tuy nhiên, những models này không phải là mà không tradeoffs của chúng. Trên nhiều hệ điều hành, chúng sử dụng một fair bit của bộ nhớ cho mỗi thread. Threads cũng chỉ là một option khi hệ điều hành của bạn và hardware hỗ trợ chúng. Không giống như mainstream desktop và mobile computers, một số embedded systems không có một OS tại tất cả, vì vậy chúng cũng không có threads.

Mô hình async cung cấp một different—và cuối cùng complementary—tập hợp của tradeoffs. Trong mô hình async, concurrent operations không yêu cầu threads của riêng chúng. Thay vào đó, chúng có thể chạy trên tasks, như khi chúng ta đã sử dụng `trpl::spawn_task` để kick off work từ một synchronous function trong streams section. Một task là tương tự với một thread, nhưng thay vì được quản lý bởi operating system, nó được quản lý bởi library-level code: runtime.

Có một lý do các APIs cho spawning threads và spawning tasks đó là vì vậy tương tự. Threads hoạt động như một boundary cho sets của synchronous operations; concurrency có thể xảy ra _between_ threads. Tasks hoạt động như một boundary cho sets của _asynchronous_ operations; concurrency có thể xảy ra cả _between_ và _within_ tasks, bởi vì một task có thể chuyển đổi giữa futures trong phần thân của nó. Cuối cùng, futures là Rust's hạt nhân granular nhất của concurrency, và mỗi future có thể đại diện cho một tree của futures khác. Runtime—đặc biệt, executor của nó—quản lý tasks, và tasks quản lý futures. Trong đó regard, tasks là tương tự với lightweight, runtime-managed threads với thêm capabilities đó đến từ được quản lý bởi một runtime thay vì bởi operating system.

Điều này không có nghĩa rằng async tasks luôn luôn tốt hơn so với threads (hoặc ngược lại). Concurrency với threads là theo một số cách một simpler programming model hơn concurrency với `async`. Điều đó có thể là một strength hoặc weakness. Threads là somewhat "fire và forget"; chúng có không có native equivalent tới một future, vì vậy chúng đơn giản chạy để hoàn thành mà không bị gián đoạn ngoại trừ bởi operating system chính nó.

Và nó quay ra rằng threads và tasks thường làm việc rất tốt với nhau, vì tasks có thể (ít nhất là trong một số runtimes) được di chuyển quanh giữa threads. Trên thực tế, dưới cùng, runtime chúng ta đã được sử dụng—bao gồm `spawn_blocking` và `spawn_task` functions—là multithreaded mặc định! Nhiều runtimes sử dụng một approach được gọi là _work stealing_ để transparently di chuyển tasks quanh giữa threads, dựa trên cách threads hiện tại đang được sử dụng, để cải thiện performance tổng hệ thống của hệ thống. Đó approach thực sự yêu cầu threads _and_ tasks, và do đó futures.

Khi suy nghĩ về loại nào để sử dụng khi, xem xét những quy tắc này của thumb:

- Nếu công việc là _very parallelizable_ (có nghĩa là, CPU-bound), như xử lý một bunch của dữ liệu nơi mỗi phần có thể được xử lý riêng biệt, threads là một better choice.
- Nếu công việc là _very concurrent_ (có nghĩa là, I/O-bound), như xử lý messages từ một bunch của các nguồn khác nhau có thể đến tại các khoảng khác nhau hoặc khác nhau rates, async là một better choice.

Và nếu bạn cần cả parallelism và concurrency, bạn không phải để chọn giữa threads và async. Bạn có thể sử dụng chúng với nhau freely, để mỗi cái chơi phần nó của tốt nhất tại. Ví dụ, Listing 17-25 cho thấy một rather common ví dụ của loại này của hỗn hợp trong real-world Rust code.

<Listing number="17-25" caption="Gửi messages với blocking code trong một thread và awaiting messages trong một async block" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-25/src/main.rs:all}}
```

</Listing>

Chúng ta bắt đầu bằng cách tạo một async channel, sau đó spawning một thread mà lấy ownership của bên gửi của channel sử dụng từ khóa `move`. Trong thread, chúng ta gửi các numbers 1 qua 10, ngủ cho một giây giữa mỗi. Cuối cùng, chúng ta chạy một future tạo với một async block được truyền đến `trpl::block_on` giống như chúng ta có xuyên suốt chương. Trong future đó, chúng ta await những messages, giống như trong các message-passing examples khác chúng ta đã thấy.

Để quay lại scenrio chúng ta mở chương với, hãy tưởng tượng chạy một tập hợp của video encoding tasks sử dụng một dedicated thread (bởi vì video encoding là compute-bound) nhưng notifying UI mà những operations đó xong với một async channel. Có vô số ví dụ của những loại này của combinations trong real-world sử dụng trường hợp.

## Tóm Tắt

Đây không phải lần cuối cùng bạn sẽ thấy concurrency trong cuốn sách này. Dự án trong [Chương 21][ch21]<!-- ignore --> sẽ áp dụng những concepts này trong một more realistic situation hơn so với các simpler examples thảo luận ở đây và so sánh problem-solving với threading so với tasks và futures more directly.

Không vấn đề loại nào của các approaches bạn chọn, Rust cung cấp cho bạn các tools bạn cần để viết safe, fast, concurrent code—liệu cho một high-throughput web server hoặc một embedded operating system.

Tiếp theo, chúng ta sẽ nói về idiomatic cách để model problems và structure solutions như các Rust programs của bạn nhận được bigger. Ngoài ra, chúng ta sẽ thảo luận về cách Rust's idioms liên quan đến những bạn có thể quen thuộc từ object-oriented lập trình.

[ch16]: http://localhost:3000/ch16-00-concurrency.html
[combining-futures]: ch17-03-more-futures.html#building-our-own-async-abstractions
[streams]: ch17-04-streams.html#composing-streams
[ch21]: ch21-00-final-project-a-web-server.html
