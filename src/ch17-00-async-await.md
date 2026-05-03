# Cơ bản của Lập trình Không Đồng bộ: Async, Await, Futures, và Streams

Nhiều hoạt động mà chúng ta yêu cầu máy tính thực hiện có thể mất một thời gian
để hoàn thành. Sẽ tốt nếu chúng ta có thể làm điều gì đó khác trong khi chờ
những quy trình chạy lâu dài hoàn thành. Các máy tính hiện đại cung cấp hai kỹ
thuật để làm việc trên nhiều hơn một hoạt động cùng một lúc: parallelism và
concurrency. Logic của các chương trình của chúng ta, tuy nhiên, được viết theo
phần lớn là một cách tuyến tính. Chúng ta muốn có thể chỉ định các hoạt động
mà một chương trình nên thực hiện và các điểm tại đó một hàm có thể tạm dừng và
một phần khác của chương trình có thể chạy thay vào đó, mà không cần phải chỉ
định trước chính xác thứ tự và cách mà mỗi phần mã nên chạy. _Lập trình không
đồng bộ_ là một trừu tượng cho phép chúng ta thể hiện mã của chúng ta dưới dạng
các điểm tạm dừng tiềm năng và kết quả cuối cùng giúp chúng ta xử lý các chi
tiết điều phối.

Chương này xây dựng dựa trên việc sử dụng threads của Chương 16 cho parallelism
và concurrency bằng cách giới thiệu một cách tiếp cận thay thế để viết mã: futures,
streams của Rust, và cú pháp `async` và `await` cho phép chúng ta thể hiện cách
các hoạt động có thể không đồng bộ, cùng với các crates bên thứ ba triển khai
asynchronous runtimes: mã quản lý và phối hợp việc thực hiện các hoạt động không
đồng bộ.

Hãy xem xét một ví dụ. Giả sử bạn đang xuất một video mà bạn đã tạo về một buổi
lễ kỷ niệm gia đình, một hoạt động có thể mất bất cứ đâu từ vài phút đến nhiều
giờ. Việc xuất video sẽ sử dụng bao nhiêu CPU và GPU power mà nó có thể. Nếu
bạn chỉ có một CPU core và hệ điều hành của bạn không tạm dừng việc xuất đó cho
đến khi nó hoàn thành—tức là, nếu nó thực hiện việc xuất _một cách đồng bộ_—bạn
không thể làm gì khác trên máy tính của bạn trong khi tác vụ đó đang chạy. Đó sẽ
là một trải nghiệm khá bực bội. May mắn thay, hệ điều hành của máy tính của bạn
có thể, và thực sự, ngắt quãng việc xuất một cách vô hình một cách thường xuyên
đủ để bạn có thể thực hiện công việc khác đồng thời.

Bây giờ giả sử bạn đang tải xuống một video được chia sẻ bởi người khác, điều
này cũng có thể mất một thời gian nhưng không chiếm CPU time nhiều. Trong trường
hợp này, CPU phải chờ dữ liệu đến từ mạng. Mặc dù bạn có thể bắt đầu đọc dữ
liệu khi nó bắt đầu đến, nhưng có thể mất một thời gian để tất cả nó xuất hiện.
Ngay cả khi tất cả dữ liệu đã có, nếu video khá lớn, nó có thể mất ít nhất một
hoặc hai giây để tải tất cả. Điều đó có thể không nghe có vẻ quá tệ, nhưng đó
là một thời gian rất dài cho một bộ xử lý hiện đại, có thể thực hiện hàng tỷ
hoạt động mỗi giây. Một lần nữa, hệ điều hành của bạn sẽ ngắt quãng chương trình
của bạn một cách vô hình để cho phép CPU thực hiện công việc khác trong khi chờ
lệnh gọi mạng hoàn thành.

Việc xuất video là một ví dụ về hoạt động _CPU-bound_ hoặc _compute-bound_.
Nó bị giới hạn bởi tốc độ xử lý dữ liệu tiềm năng của máy tính trong CPU hoặc
GPU, và bao nhiêu tốc độ đó nó có thể dành cho hoạt động. Việc tải xuống video
là một ví dụ về hoạt động _I/O-bound_, vì nó bị giới hạn bởi tốc độ _input và
output_ của máy tính; nó chỉ có thể nhanh bằng tốc độ dữ liệu có thể được gửi
qua mạng.

Trong cả hai ví dụ này, các ngắt quãng vô hình của hệ điều hành cung cấp một
hình thức concurrency. Tuy nhiên, concurrency đó chỉ xảy ra ở cấp độ của toàn
bộ chương trình: hệ điều hành ngắt quãng một chương trình để cho phép các chương
trình khác thực hiện công việc. Trong nhiều trường hợp, vì chúng ta hiểu các
chương trình của chúng ta ở một cấp độ chi tiết hơn nhiều so với hệ điều hành,
chúng ta có thể phát hiện các cơ hội cho concurrency mà hệ điều hành không thể
thấy.

Ví dụ, nếu chúng ta đang xây dựng một công cụ để quản lý tải xuống tệp, chúng
ta nên có thể viết chương trình của chúng ta sao cho bắt đầu một tải xuống sẽ
không khóa UI, và người dùng nên có thể bắt đầu nhiều tải xuống cùng một lúc.
Tuy nhiên, nhiều API hệ điều hành để tương tác với mạng là _blocking_; tức là,
chúng chặn tiến trình của chương trình cho đến khi dữ liệu họ đang xử lý hoàn
toàn sẵn sàng.

> Lưu ý: Đây là cách _hầu hết_ các lệnh gọi hàm hoạt động, nếu bạn suy nghĩ về
> nó. Tuy nhiên, thuật ngữ _blocking_ thường được dành riêng cho các lệnh gọi
> hàm tương tác với tệp, mạng hoặc các tài nguyên khác trên máy tính, vì đó
> là những trường hợp mà một chương trình riêng sẽ được hưởng lợi từ hoạt động
> _non_-blocking.

Chúng ta có thể tránh chặn thread chính của chúng ta bằng cách sinh ra một thread
chuyên dụng để tải xuống từng tệp. Tuy nhiên, chi phí tài nguyên hệ thống được
sử dụng bởi những thread đó cuối cùng sẽ trở thành một vấn đề. Sẽ tốt hơn nếu
lệnh gọi không bị chặn ngay từ đầu, và thay vào đó chúng ta có thể định nghĩa
một số tác vụ mà chúng ta muốn chương trình của chúng ta hoàn thành và cho phép
runtime chọn thứ tự và cách tốt nhất để chạy chúng.

Đó chính xác là những gì trừu tượng _async_ (viết tắt của _asynchronous_) của
Rust cung cấp cho chúng ta. Trong chương này, bạn sẽ học tất cả về async khi
chúng ta t覆 các chủ đề sau:

- Cách sử dụng cú pháp `async` và `await` của Rust và thực hiện các hàm không
  đồng bộ với một runtime
- Cách sử dụng mô hình async để giải quyết một số thách thức tương tự mà chúng
  ta xem xét trong Chương 16
- Cách multithreading và async cung cấp các giải pháp bổ sung mà bạn có thể
  kết hợp trong nhiều trường hợp

Tuy nhiên, trước khi chúng ta thấy async hoạt động trong thực tế, chúng ta cần
phải thực hiện một chuyến tham quan ngắn để thảo luận về sự khác biệt giữa
parallelism và concurrency.

## Parallelism và Concurrency

Cho đến nay, chúng ta đã coi parallelism và concurrency như chủ yếu có thể hoán
đổi cho nhau. Bây giờ chúng ta cần phân biệt giữa chúng một cách chính xác hơn,
vì những khác biệt sẽ xuất hiện khi chúng ta bắt đầu làm việc.

Hãy xem xét các cách khác nhau mà một đội có thể chia công việc trên một dự án
phần mềm. Bạn có thể gán cho một thành viên duy nhất nhiều tác vụ, gán cho mỗi
thành viên một tác vụ, hoặc sử dụng sự kết hợp của hai cách tiếp cận.

Khi một cá nhân làm việc trên nhiều tác vụ khác nhau trước khi bất kỳ tác vụ
nào hoàn thành, đây là _concurrency_. Một cách để triển khai concurrency tương
tự như có hai dự án khác nhau được kiểm tra trên máy tính của bạn, và khi bạn
cảm thấy chán hoặc bị mắc kẹt ở một dự án, bạn chuyển sang dự án khác. Bạn chỉ
là một người, vì vậy bạn không thể đạt được tiến bộ trên cả hai tác vụ cùng một
lúc, nhưng bạn có thể làm nhiều việc, đạt được tiến bộ từng cái một bằng cách
chuyển đổi giữa chúng (xem Hình 17-1).

<figure>

<img src="img/trpl17-01.svg" class="center" alt="A diagram with stacked boxes labeled Task A and Task B, with diamonds in them representing subtasks. Arrows point from A1 to B1, B1 to A2, A2 to B2, B2 to A3, A3 to A4, and A4 to B3. The arrows between the subtasks cross the boxes between Task A and Task B." />

<figcaption>Hình 17-1: Một quy trình công việc concurrent, chuyển đổi giữa Task A và Task B</figcaption>

</figure>

Khi đội chia một nhóm tác vụ bằng cách để mỗi thành viên lấy một tác vụ và làm
việc với nó một mình, đây là _parallelism_. Mỗi người trong đội có thể đạt được
tiến bộ cùng một lúc (xem Hình 17-2).

<figure>

<img src="img/trpl17-02.svg" class="center" alt="A diagram with stacked boxes labeled Task A and Task B, with diamonds in them representing subtasks. Arrows point from A1 to A2, A2 to A3, A3 to A4, B1 to B2, and B2 to B3. No arrows cross between the boxes for Task A and Task B." />

<figcaption>Hình 17-2: Một quy trình công việc song song, nơi công việc xảy ra trên Task A và Task B một cách độc lập</figcaption>

</figure>

Trong cả hai quy trình công việc này, bạn có thể phải phối hợp giữa các tác vụ
khác nhau. Có thể bạn nghĩ rằng tác vụ được gán cho một người hoàn toàn độc lập
với công việc của mọi người khác, nhưng nó thực sự yêu cầu một người khác trong
đội phải hoàn thành tác vụ của họ trước. Một số công việc có thể được thực hiện
song song, nhưng một số công việc thực tế là _serial_: nó chỉ có thể xảy ra trong
một chuỗi, một tác vụ sau tác vụ khác, như trong Hình 17-3.

<figure>

<img src=”img/trpl17-03.svg” class=”center” alt=”A diagram with stacked boxes labeled Task A and Task B, with diamonds in them representing subtasks. In Task A, arrows point from A1 to A2, from A2 to a pair of thick vertical lines like a “pause” symbol, and from that symbol to A3. In task B, arrows point from B1 to B2, from B2 to B3, from B3 to A3, and from B3 to B4.” />

<figcaption>Hình 17-3: Một quy trình công việc một phần song song, nơi công việc xảy ra trên Task A và Task B một cách độc lập cho đến khi Task A3 bị chặn bởi kết quả của Task B3.</figcaption>

</figure>

Tương tự, bạn có thể nhận ra rằng một trong những tác vụ của bạn phụ thuộc vào
một tác vụ khác của bạn. Bây giờ công việc concurrent của bạn cũng đã trở thành
serial.

Parallelism và concurrency cũng có thể giao nhau với nhau. Nếu bạn biết rằng một
đồng nghiệp bị kẹt cho đến khi bạn hoàn thành một trong các tác vụ của bạn, bạn
có thể sẽ tập trung tất cả nỗ lực của mình vào tác vụ đó để “mở khóa” đồng nghiệp
của bạn. Bạn và người cộng sự không còn có thể làm việc song song, và bạn cũng
không còn có thể làm việc concurrent trên các tác vụ của riêng bạn.

Các động lực cơ bản tương tự cũng đạo diễn với phần mềm và phần cứng. Trên một
máy có một CPU core duy nhất, CPU chỉ có thể thực hiện một hoạt động cùng một
lúc, nhưng nó vẫn có thể làm việc concurrent. Sử dụng các công cụ như threads,
processes, và async, máy tính có thể tạm dừng một hoạt động và chuyển sang các
hoạt động khác trước khi cuối cùng quay lại hoạt động đó. Trên một máy có nhiều
CPU cores, nó cũng có thể thực hiện công việc song song. Một core có thể thực
hiện một tác vụ trong khi một core khác thực hiện một tác vụ hoàn toàn không liên
quan, và những hoạt động đó thực sự xảy ra cùng một lúc.

Chạy mã async trong Rust thường xảy ra một cách concurrent. Tùy thuộc vào phần
cứng, hệ điều hành, và async runtime mà chúng ta đang sử dụng (sẽ nói thêm về
async runtimes ngay sau), concurrency đó cũng có thể sử dụng parallelism dưới
đó.

Bây giờ, hãy tìm hiểu cách lập trình async trong Rust thực sự hoạt động.
