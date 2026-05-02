Dưới đây là bản dịch tiếng Việt bám sát nội dung, giữ nguyên các thuật ngữ kỹ thuật như bạn yêu cầu.

***

# Introduction – Giới thiệu

> Lưu ý: Ấn bản của cuốn sách này giống với [The Rust Programming
> Language][nsprust] được phát hành dưới dạng sách in và ebook bởi [No Starch
> Press][nsp].

[nsprust]: https://nostarch.com/rust-programming-language-3rd-edition  
[nsp]: https://nostarch.com/

Chào mừng bạn đến với _The Rust Programming Language_, một cuốn sách nhập môn về Rust.  
Ngôn ngữ lập trình Rust giúp bạn viết phần mềm chạy nhanh hơn, đáng tin cậy hơn. 

Trong thiết kế ngôn ngữ lập trình, **tính tiện dụng cấp cao** và **khả năng kiểm soát cấp thấp** thường mâu thuẫn với nhau; Rust thách thức sự xung đột đó. 
Bằng cách cân bằng giữa năng lực kỹ thuật mạnh mẽ và trải nghiệm developer tốt, Rust cho bạn khả năng kiểm soát các chi tiết cấp thấp (như cách sử dụng memory) mà không phải chịu tất cả phiền toái vốn gắn với kiểu kiểm soát này trong các ngôn ngữ truyền thống. 

## Rust dành cho ai

Rust phù hợp với rất nhiều nhóm người vì nhiều lý do khác nhau.  
Hãy xem qua một vài nhóm quan trọng nhất. 

### Các team developer

Rust đang chứng tỏ mình là một công cụ hiệu quả cho việc cộng tác trong những team lớn gồm nhiều developer với mức độ hiểu biết về systems programming khác nhau. 
Code cấp thấp thường dễ mắc nhiều bug tinh vi, mà trong hầu hết các ngôn ngữ khác chỉ có thể phát hiện bằng test rất kỹ và code review cẩn thận bởi những developer giàu kinh nghiệm. 

Trong Rust, compiler đóng vai trò như “gatekeeper” khi từ chối compile những đoạn code chứa các bug khó chịu đó, bao gồm cả bug về concurrency. 
Khi làm việc cùng compiler, team có thể dành thời gian tập trung vào logic của chương trình thay vì đuổi theo bug. 

Rust cũng mang các công cụ hiện đại của developer vào thế giới systems programming: 

- Cargo – dependency manager và build tool đi kèm – giúp việc thêm, compile và quản lý dependency trở nên đơn giản và nhất quán trên toàn ecosystem Rust. 
- Công cụ format `rustfmt` đảm bảo style code nhất quán giữa các developer. 
- Rust Language Server cung cấp tích hợp với IDE để có tính năng code completion và hiển thị error ngay trong editor. 

Bằng cách sử dụng những công cụ này và các công cụ khác trong Rust ecosystem, developer có thể vừa làm việc hiệu quả vừa viết được systems-level code. 

### Sinh viên

Rust dành cho sinh viên và những người muốn tìm hiểu về các khái niệm hệ thống (systems concepts). 
Nhờ Rust, nhiều người đã học được những chủ đề như phát triển hệ điều hành. 

Cộng đồng Rust rất thân thiện và sẵn sàng trả lời câu hỏi của sinh viên. 
Thông qua những nỗ lực như cuốn sách này, các team Rust muốn làm cho các khái niệm hệ thống trở nên dễ tiếp cận hơn với nhiều người, đặc biệt là những người mới lập trình. 

### Công ty

Hàng trăm công ty, từ lớn đến nhỏ, sử dụng Rust trong production cho nhiều loại tác vụ: command line tool, web service, công cụ DevOps, thiết bị embedded, phân tích và chuyển mã (transcoding) audio/video, cryptocurrency, bioinformatics, search engine, ứng dụng Internet of Things, machine learning, và thậm chí cả những phần quan trọng của trình duyệt Firefox. 

### Developer mã nguồn mở

Rust dành cho những người muốn xây dựng chính ngôn ngữ lập trình Rust, cộng đồng, công cụ developer, và các thư viện xung quanh nó. 
Chúng tôi rất mong bạn đóng góp cho ngôn ngữ Rust. 

### Những người coi trọng tốc độ và ổn định

Rust dành cho những người khao khát tốc độ và sự ổn định trong một ngôn ngữ. 
“Tốc độ” ở đây bao gồm cả tốc độ chạy của code Rust và tốc độ mà Rust giúp bạn viết chương trình. 

Các bước kiểm tra của Rust compiler giúp đảm bảo tính ổn định xuyên suốt quá trình bổ sung feature mới và refactor. 
Điều này trái ngược với legacy code “dễ vỡ” ở những ngôn ngữ không có các kiểm tra như vậy, nơi developer thường sợ chỉnh sửa code. 

Bằng việc hướng tới các zero-cost abstraction – những feature cấp cao compile xuống code cấp thấp nhanh như code viết tay – Rust cố gắng để code an toàn vẫn là code nhanh. 

Ngôn ngữ Rust hy vọng hỗ trợ nhiều kiểu người dùng khác nữa; các nhóm được nhắc đến ở đây chỉ là một vài stakeholder quan trọng nhất. 
Nhìn chung, tham vọng lớn nhất của Rust là loại bỏ những trade-off mà programmer đã chấp nhận suốt hàng thập kỷ bằng cách cung cấp cả safety _và_ productivity, speed _và_ ergonomics. 
Hãy thử Rust và xem liệu những lựa chọn của nó có phù hợp với bạn không. 

## Cuốn sách này dành cho ai

Cuốn sách này giả định rằng bạn đã từng viết code bằng ít nhất một ngôn ngữ lập trình khác, nhưng không giả định cụ thể là ngôn ngữ nào. 
Chúng tôi cố gắng làm cho nội dung phù hợp với những người đến từ nhiều background lập trình khác nhau. 

Chúng tôi không dành nhiều thời gian để nói về “lập trình là gì” hay cách tư duy về lập trình. 
Nếu bạn hoàn toàn mới với lập trình, có lẽ một cuốn sách giới thiệu lập trình từ đầu sẽ phù hợp với bạn hơn. 

## Cách sử dụng cuốn sách này

Về tổng thể, cuốn sách này giả định rằng bạn sẽ đọc tuần tự từ đầu đến cuối. 
Các chương sau xây dựng trên các khái niệm của chương trước, và các chương đầu có thể sẽ không đi sâu vào chi tiết một chủ đề nào đó mà sẽ quay lại nói kỹ hơn ở chương sau. 

Bạn sẽ thấy hai loại chương trong cuốn sách: chương khái niệm (concept chapter) và chương dự án (project chapter). 
Trong các chương khái niệm, bạn sẽ học về một khía cạnh nào đó của Rust.  
Trong các chương dự án, chúng ta sẽ xây dựng những chương trình nhỏ cùng nhau, áp dụng những gì bạn đã học. 
Chapter 2, Chapter 12 và Chapter 21 là các project chapter; các chương còn lại là concept chapter. 

**Chapter 1** giải thích cách cài đặt Rust, cách viết chương trình “Hello, world!” và cách sử dụng Cargo – package manager kiêm build tool của Rust. 
**Chapter 2** là phần giới thiệu thực hành về cách viết một chương trình bằng Rust, trong đó bạn sẽ xây dựng dần một trò chơi đoán số (number-guessing game). 

Ở đây, chúng ta sẽ trình bày các khái niệm ở mức high-level, và những chương sau sẽ cung cấp thêm chi tiết. 
Nếu bạn muốn bắt tay vào code ngay, Chapter 2 là nơi phù hợp. 
Nếu bạn là người học kỹ lưỡng, thích hiểu mọi chi tiết trước khi chuyển sang phần tiếp theo, bạn có thể bỏ qua Chapter 2 và đi thẳng đến **Chapter 3**, nơi nói về những feature của Rust giống với các ngôn ngữ lập trình khác; sau đó bạn có thể quay lại Chapter 2 khi muốn làm một project để áp dụng những chi tiết đã học. 

Trong **Chapter 4**, bạn sẽ học về ownership system của Rust. 
**Chapter 5** thảo luận về struct và method. 
**Chapter 6** nói về enum, expression `match`, và các cấu trúc điều khiển `if let` và `let...else`. 
Bạn sẽ dùng struct và enum để tạo ra các custom type. 

Trong **Chapter 7**, bạn sẽ học về module system của Rust và các quy tắc visibility/privacy để tổ chức code và public API (application programming interface) của nó. 
**Chapter 8** thảo luận một số collection data structure phổ biến mà standard library cung cấp: vector, string và hash map. 
**Chapter 9** khám phá triết lý và kỹ thuật xử lý lỗi (error handling) của Rust. 

**Chapter 10** đào sâu vào generic, trait và lifetime – những thứ cho bạn sức mạnh định nghĩa code có thể áp dụng cho nhiều type khác nhau. 
**Chapter 11** nói toàn bộ về testing, thứ vẫn cần thiết ngay cả khi đã có các đảm bảo về safety của Rust để chắc chắn logic chương trình là đúng. 
Trong **Chapter 12**, chúng ta sẽ tự xây dựng một phần chức năng của công cụ dòng lệnh `grep` – công cụ dùng để tìm kiếm text trong file. 
Để làm việc này, chúng ta sẽ dùng nhiều khái niệm đã nói trong các chương trước. 

**Chapter 13** khám phá closure và iterator – các feature của Rust lấy cảm hứng từ các ngôn ngữ lập trình hàm (functional programming language). 
Trong **Chapter 14**, chúng ta sẽ xem xét Cargo kỹ hơn và nói về best practice khi chia sẻ library với người khác. 
**Chapter 15** thảo luận về smart pointer mà standard library cung cấp và các trait giúp chúng hoạt động. 

Trong **Chapter 16**, chúng ta sẽ đi qua những mô hình lập trình concurrent khác nhau và nói về cách Rust giúp bạn lập trình đa luồng (multiple threads) một cách “không sợ hãi”. 
Trong **Chapter 17**, chúng ta xây dựng tiếp bằng cách khám phá async/await syntax của Rust, cùng với task, future và stream, và mô hình concurrency nhẹ (lightweight) mà chúng cho phép. 

**Chapter 18** xem xét cách các idiom trong Rust so sánh với những nguyên lý lập trình hướng đối tượng (OOP) mà bạn có thể đã quen thuộc. 
**Chapter 19** là phần tham khảo về pattern và pattern matching, những cách mạnh mẽ để diễn đạt ý tưởng trong Rust program. 
**Chapter 20** là “buffet” các chủ đề nâng cao thú vị, bao gồm unsafe Rust, macro, và nhiều hơn nữa về lifetime, trait, type, function và closure. 

Trong **Chapter 21**, chúng ta sẽ hoàn thành một project, trong đó chúng ta sẽ implement một web server đa luồng (multithreaded) cấp thấp! 

Cuối cùng, một số appendix chứa thông tin hữu ích về ngôn ngữ dưới dạng tài liệu tham khảo. 
**Appendix A** nói về keyword của Rust, **Appendix B** nói về operator và symbol của Rust, **Appendix C** nói về các derivable trait mà standard library cung cấp, **Appendix D** giới thiệu một số công cụ phát triển hữu ích, và **Appendix E** giải thích về các Rust edition. 
Trong **Appendix F**, bạn có thể tìm các bản dịch của cuốn sách, và trong **Appendix G** chúng ta sẽ bàn về cách Rust được xây dựng và “nightly Rust” là gì. 

Không có “cách đọc sai” cho cuốn sách này: nếu bạn muốn nhảy cóc đến phần sau, cứ làm! 
Có thể bạn sẽ phải quay lại các chương trước nếu cảm thấy bị rối.  
Hãy làm bất cứ điều gì phù hợp với bạn. 

<span id="ferris"></span>

Một phần quan trọng trong việc học Rust là học cách đọc error message mà compiler hiển thị: chúng sẽ dẫn bạn đến code chạy được. 
Vì vậy, chúng tôi sẽ cung cấp nhiều ví dụ **không compile được** kèm theo error message mà compiler đưa ra trong từng tình huống. 

Bạn hãy lưu ý rằng nếu bạn gõ và chạy một ví dụ ngẫu nhiên, có thể nó sẽ **không compile!**  
Hãy chắc chắn rằng bạn đọc phần giải thích xung quanh để xem ví dụ bạn đang chạy có chủ đích gây error hay không. 
Trong hầu hết tình huống, chúng tôi sẽ dẫn bạn đến phiên bản đúng của bất kỳ đoạn code nào không compile được. 
Ferris cũng sẽ giúp bạn phân biệt những đoạn code **không nhằm mục đích chạy đúng**: [scs.stanford](https://www.scs.stanford.edu/~zyedidia/docs/rust/rust_book.pdf)

| Ferris                                                                                                           | Ý nghĩa                                              |
| ---------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| (Hình) Ferris với dấu hỏi                                                                                       | Đoạn code này không compile!                         |
| (Hình) Ferris giơ hai càng lên                                                                                  | Đoạn code này sẽ panic!                              |
| (Hình) Ferris nhún vai, giơ một càng                                                                           | Đoạn code này không tạo ra hành vi như mong muốn.    | 

Trong phần lớn các trường hợp, chúng tôi sẽ dẫn bạn đến phiên bản đúng của bất kỳ đoạn code nào không compile được. 
