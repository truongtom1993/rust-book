## Phụ Lục G - Cách Rust Được Tạo Ra và "Nightly Rust"

Phụ lục này nói về cách Rust được tạo ra và điều đó ảnh hưởng đến bạn như thế nào với tư cách là một Rust developer.

### Ổn Định Không Trì Trệ

Là một ngôn ngữ lập trình, Rust quan tâm _rất nhiều_ đến tính ổn định của code của bạn. Chúng tôi muốn Rust là một nền tảng vững chắc mà bạn có thể xây dựng trên đó, và nếu mọi thứ liên tục thay đổi, điều đó sẽ là không thể. Đồng thời, nếu chúng tôi không thể thử nghiệm các tính năng mới, chúng tôi có thể không phát hiện ra các lỗ hổng quan trọng cho đến sau khi phát hành, khi chúng tôi không thể thay đổi mọi thứ nữa.

Giải pháp của chúng tôi cho vấn đề này là những gì chúng tôi gọi là "ổn định không trì trệ", và nguyên tắc hướng dẫn của chúng tôi là: bạn không bao giờ phải lo sợ khi nâng cấp lên một phiên bản mới của Rust stable. Mỗi lần nâng cấp nên không đau đớn, nhưng cũng nên mang lại cho bạn các tính năng mới, ít lỗi hơn, và thời gian biên dịch nhanh hơn.

### Choo, Choo! Các Kênh Phát Hành và Đi Theo Chuyến Tàu

Phát triển Rust hoạt động theo _lịch trình tàu_. Nghĩa là, tất cả các phát triển được thực hiện trên nhánh chính của kho lưu trữ Rust. Các bản phát hành tuân theo mô hình train phát hành phần mềm, đã được sử dụng bởi Cisco IOS và các dự án phần mềm khác. Có ba _kênh phát hành_ cho Rust:

- Nightly
- Beta
- Stable

Hầu hết các Rust developer chủ yếu sử dụng kênh stable, nhưng những người muốn thử các tính năng thử nghiệm mới có thể sử dụng nightly hoặc beta.

Đây là ví dụ về cách quy trình phát triển và phát hành hoạt động: hãy giả sử rằng nhóm Rust đang làm việc để phát hành Rust 1.5. Bản phát hành đó xảy ra vào tháng 12 năm 2015, nhưng nó sẽ cung cấp cho chúng ta các số phiên bản thực tế. Một tính năng mới được thêm vào Rust: một commit mới được đưa vào nhánh chính. Mỗi đêm, một phiên bản nightly mới của Rust được tạo ra. Mỗi ngày là một ngày phát hành, và các bản phát hành này được tạo ra bởi cơ sở hạ tầng phát hành của chúng tôi một cách tự động. Vì vậy khi thời gian trôi qua, các bản phát hành của chúng tôi trông như thế này, mỗi đêm một lần:

```text
nightly: * - - * - - *
```

Cứ sáu tuần một lần, đã đến lúc chuẩn bị một bản phát hành mới! Nhánh `beta` của kho lưu trữ Rust phân nhánh từ nhánh chính được sử dụng bởi nightly. Bây giờ, có hai bản phát hành:

```text
nightly: * - - * - - *
                     |
beta:                *
```

Hầu hết người dùng Rust không sử dụng các bản beta một cách tích cực, nhưng kiểm tra với beta trong hệ thống CI của họ để giúp Rust phát hiện các regression có thể xảy ra. Trong khi đó, vẫn có một bản phát hành nightly mỗi đêm:

```text
nightly: * - - * - - * - - * - - *
                     |
beta:                *
```

Giả sử một regression được tìm thấy. Thật may là chúng ta đã có thời gian để kiểm tra bản beta trước khi regression lọt vào bản phát hành stable! Bản sửa lỗi được áp dụng vào nhánh chính, để nightly được sửa, sau đó bản sửa lỗi được backport đến nhánh `beta`, và một bản phát hành beta mới được tạo ra:

```text
nightly: * - - * - - * - - * - - * - - *
                     |
beta:                * - - - - - - - - *
```

Sáu tuần sau khi beta đầu tiên được tạo ra, đã đến lúc phát hành stable! Nhánh `stable` được tạo ra từ nhánh `beta`:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |
beta:                * - - - - - - - - *
                                       |
stable:                                *
```

Tuyệt vời! Rust 1.5 đã xong! Tuy nhiên, chúng ta đã quên một thứ: vì sáu tuần đã trôi qua, chúng ta cũng cần một beta mới của phiên bản _tiếp theo_ của Rust, 1.6. Vì vậy sau khi `stable` phân nhánh từ `beta`, phiên bản tiếp theo của `beta` phân nhánh từ `nightly` một lần nữa:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

Điều này được gọi là "mô hình tàu" vì cứ sáu tuần, một bản phát hành "rời ga", nhưng vẫn phải đi qua kênh beta trước khi đến với tư cách là bản phát hành stable.

Rust phát hành mỗi sáu tuần, như đồng hồ. Nếu bạn biết ngày của một bản phát hành Rust, bạn có thể biết ngày của bản tiếp theo: đó là sáu tuần sau. Một khía cạnh hay của việc lên lịch phát hành mỗi sáu tuần là chuyến tàu tiếp theo sẽ đến sớm. Nếu một tính năng tình cờ bỏ lỡ một bản phát hành cụ thể, không cần phải lo lắng: một bản khác đang xảy ra trong thời gian ngắn! Điều này giúp giảm áp lực phải nhồi nhét các tính năng có thể chưa hoàn thiện vào gần thời hạn phát hành.

Nhờ quy trình này, bạn luôn có thể kiểm tra bản build tiếp theo của Rust và tự xác minh rằng việc nâng cấp là dễ dàng: nếu một bản beta không hoạt động như mong đợi, bạn có thể báo cáo cho nhóm và sửa trước khi bản phát hành stable tiếp theo xảy ra! Sự cố trong bản beta là tương đối hiếm, nhưng `rustc` vẫn là một phần mềm, và các lỗi vẫn tồn tại.

### Thời Gian Bảo Trì

Dự án Rust hỗ trợ phiên bản stable gần nhất. Khi một phiên bản stable mới được phát hành, phiên bản cũ đạt đến cuối vòng đời (EOL). Điều này có nghĩa là mỗi phiên bản được hỗ trợ trong sáu tuần.

### Các Tính Năng Không Ổn Định

Còn một điều nữa với mô hình phát hành này: các tính năng không ổn định. Rust sử dụng một kỹ thuật gọi là "feature flag" để xác định các tính năng nào được kích hoạt trong một bản phát hành nhất định. Nếu một tính năng mới đang trong quá trình phát triển tích cực, nó sẽ được đưa vào nhánh chính, và do đó, vào nightly, nhưng phía sau một _feature flag_. Nếu bạn, với tư cách là người dùng, muốn thử tính năng đang trong quá trình phát triển, bạn có thể, nhưng bạn phải sử dụng bản phát hành nightly của Rust và chú thích code nguồn của bạn với flag thích hợp để chọn tham gia.

Nếu bạn đang sử dụng bản phát hành beta hoặc stable của Rust, bạn không thể sử dụng bất kỳ feature flag nào. Đây là chìa khóa cho phép chúng ta có được việc sử dụng thực tế với các tính năng mới trước khi chúng ta tuyên bố chúng stable mãi mãi. Những người muốn chọn tham gia vào bleeding edge có thể làm vậy, và những người muốn trải nghiệm vững chắc có thể gắn bó với stable và biết rằng code của họ sẽ không bị hỏng. Ổn định không trì trệ.

Cuốn sách này chỉ chứa thông tin về các tính năng stable, vì các tính năng đang trong quá trình phát triển vẫn đang thay đổi, và chắc chắn chúng sẽ khác nhau giữa thời điểm cuốn sách này được viết và khi chúng được kích hoạt trong các bản build stable. Bạn có thể tìm tài liệu cho các tính năng chỉ dành cho nightly trực tuyến.

### Rustup và Vai Trò của Rust Nightly

Rustup giúp dễ dàng chuyển đổi giữa các kênh phát hành khác nhau của Rust, trên cơ sở toàn cục hoặc theo từng dự án. Theo mặc định, bạn sẽ có Rust stable được cài đặt. Để cài đặt nightly, ví dụ:

```console
$ rustup toolchain install nightly
```

Bạn có thể xem tất cả các _toolchain_ (các bản phát hành của Rust và các thành phần liên quan) mà bạn đã cài đặt với `rustup` như sau. Đây là ví dụ trên máy tính Windows của một trong những tác giả:

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

Như bạn có thể thấy, toolchain stable là mặc định. Hầu hết người dùng Rust sử dụng stable hầu hết thời gian. Bạn có thể muốn sử dụng stable hầu hết thời gian, nhưng sử dụng nightly trên một dự án cụ thể, vì bạn quan tâm đến một tính năng tiên tiến. Để làm điều đó, bạn có thể sử dụng `rustup override` trong thư mục của dự án đó để đặt toolchain nightly là toolchain mà `rustup` nên sử dụng khi bạn ở trong thư mục đó:

```console
$ cd ~/projects/needs-nightly
$ rustup override set nightly
```

Bây giờ, mỗi khi bạn gọi `rustc` hoặc `cargo` bên trong _~/projects/needs-nightly_, `rustup` sẽ đảm bảo rằng bạn đang sử dụng Rust nightly, thay vì Rust stable mặc định của bạn. Điều này rất hữu ích khi bạn có nhiều dự án Rust!

### Quy Trình RFC và Các Nhóm

Vậy làm thế nào để bạn tìm hiểu về các tính năng mới này? Mô hình phát triển của Rust tuân theo một _quy trình Request For Comments (RFC)_. Nếu bạn muốn có cải tiến trong Rust, bạn có thể viết một đề xuất, được gọi là RFC.

Bất kỳ ai cũng có thể viết RFC để cải thiện Rust, và các đề xuất được xem xét và thảo luận bởi nhóm Rust, bao gồm nhiều nhóm con theo chủ đề. Có danh sách đầy đủ các nhóm [trên trang web của Rust](https://www.rust-lang.org/governance), bao gồm các nhóm cho mỗi lĩnh vực của dự án: thiết kế ngôn ngữ, triển khai trình biên dịch, cơ sở hạ tầng, tài liệu, và nhiều hơn nữa. Nhóm thích hợp đọc đề xuất và các bình luận, viết một số bình luận của riêng họ, và cuối cùng, đạt được sự đồng thuận để chấp nhận hoặc từ chối tính năng.

Nếu tính năng được chấp nhận, một issue được mở trên kho lưu trữ Rust, và ai đó có thể implement nó. Người implement nó có thể không phải là người đề xuất tính năng đó ngay từ đầu! Khi implementation sẵn sàng, nó được đưa vào nhánh chính phía sau một feature gate, như chúng ta đã thảo luận trong phần ["Các Tính Năng Không Ổn Định"](#unstable-features)<!-- ignore -->.

Sau một thời gian, một khi các Rust developer sử dụng bản phát hành nightly đã có thể thử nghiệm tính năng mới, các thành viên nhóm sẽ thảo luận về tính năng, cách nó hoạt động trên nightly, và quyết định liệu nó có nên được đưa vào Rust stable hay không. Nếu quyết định là tiến hành, feature gate được xóa, và tính năng bây giờ được coi là stable! Nó đi theo chuyến tàu vào một bản phát hành stable mới của Rust.
