## Unsafe Rust

Tất cả code mà chúng ta đã thảo luận cho đến nay đều có các đảm bảo bộ nhớ an toàn của Rust được thực thi tại thời gian compile. Tuy nhiên, Rust có một ngôn ngữ thứ hai ẩn bên trong nó mà không thực thi các đảm bảo bộ nhớ an toàn này: Nó được gọi là _unsafe Rust_ và hoạt động giống như Rust thông thường nhưng cung cấp cho chúng ta những siêu năng lực bổ sung.

Unsafe Rust tồn tại vì bản chất của phân tích tĩnh là bảo thủ. Khi trình biên dịch cố gắng xác định xem code có duy trì các đảm bảo không, tốt hơn là nó nên từ chối một số chương trình hợp lệ hơn là chấp nhận một số chương trình không hợp lệ. Mặc dù code _có thể_ ổn định, nếu trình biên dịch Rust không có đủ thông tin để tự tin, nó sẽ từ chối code. Trong những trường hợp này, bạn có thể sử dụng code unsafe để nói với trình biên dịch, "Hãy tin tôi, tôi biết tôi đang làm gì." Tuy nhiên, hãy cảnh báo rằng bạn sử dụng unsafe Rust với rủi ro của chính bạn: Nếu bạn sử dụng code unsafe không chính xác, các vấn đề có thể xảy ra do không an toàn bộ nhớ, chẳng hạn như dereference con trỏ null.

Một lý do khác mà Rust có một đối tác unsafe là phần cứng máy tính cơ bản về bản chất là không an toàn. Nếu Rust không cho phép bạn thực hiện các hoạt động không an toàn, bạn không thể thực hiện các tác vụ nhất định. Rust cần phải cho phép bạn thực hiện lập trình hệ thống cấp thấp, chẳng hạn như tương tác trực tiếp với hệ điều hành hoặc thậm chí viết hệ điều hành của riêng bạn. Làm việc với lập trình hệ thống cấp thấp là một trong những mục tiêu của ngôn ngữ. Hãy khám phá những gì chúng ta có thể làm với unsafe Rust và cách làm điều đó.

<!-- Old headings. Do not remove or links may break. -->

<a id="unsafe-superpowers"></a>

### Thực Hiện Unsafe Superpowers

Để chuyển sang unsafe Rust, sử dụng từ khóa `unsafe` và sau đó bắt đầu một khối mới chứa code unsafe. Bạn có thể thực hiện năm hành động trong unsafe Rust mà bạn không thể trong safe Rust, chúng ta gọi đây là _unsafe superpowers_. Những siêu năng lực đó bao gồm khả năng:

1. Dereference một raw pointer.
1. Gọi một function hoặc method unsafe.
1. Truy cập hoặc sửa đổi một biến static có thể thay đổi.
1. Implement một trait unsafe.
1. Truy cập các field của `union`s.

Điều quan trọng cần hiểu là `unsafe` không tắt borrow checker hay vô hiệu hóa bất kỳ kiểm tra an toàn nào khác của Rust: Nếu bạn sử dụng một reference trong code unsafe, nó sẽ vẫn được kiểm tra. Từ khóa `unsafe` chỉ cung cấp cho bạn quyền truy cập vào năm tính năng này mà sau đó không được kiểm tra bởi trình biên dịch để đảm bảo bộ nhớ an toàn. Bạn sẽ vẫn có được một số mức độ an toàn bên trong một khối unsafe.

Ngoài ra, `unsafe` không có nghĩa là code bên trong khối nhất thiết phải nguy hiểm hoặc nó chắc chắn sẽ có các vấn đề về bộ nhớ an toàn: Ý định là như một lập trình viên, bạn sẽ đảm bảo rằng code bên trong khối `unsafe` sẽ truy cập bộ nhớ theo một cách hợp lệ.

Mọi người đều có khả năng mắc lỗi và những sai lầm sẽ xảy ra, nhưng bằng cách yêu cầu năm hoạt động unsafe này nằm trong các khối được chú thích bằng `unsafe`, bạn sẽ biết rằng bất kỳ lỗi nào liên quan đến bộ nhớ an toàn phải nằm trong khối `unsafe`. Giữ các khối `unsafe` nhỏ; bạn sẽ cảm thấy biết ơn sau này khi bạn điều tra các lỗi bộ nhớ.

Để cô lập code unsafe càng nhiều càng tốt, tốt nhất là bao quanh code như vậy trong một trừu tượng hóa an toàn và cung cấp một API an toàn, chúng ta sẽ thảo luận sau trong chương khi chúng ta kiểm tra các function và method unsafe. Các phần của thư viện chuẩn được thực hiện như các trừu tượng hóa an toàn so với code unsafe đã được kiểm toán. Bao quanh code unsafe trong một trừu tượng hóa an toàn ngăn chặn việc sử dụng `unsafe` rò rỉ vào tất cả các nơi mà bạn hoặc người dùng của bạn có thể muốn sử dụng chức năng được triển khai bằng code unsafe, vì sử dụng một trừu tượng hóa an toàn là an toàn.

Hãy xem xét từng năm unsafe superpowers lần lượt. Chúng ta cũng sẽ xem xét một số trừu tượng hóa cung cấp một giao diện an toàn cho code unsafe.

### Dereferencing một Raw Pointer

Trong Chương 4, trong phần ["Dangling References"][dangling-references]<!-- ignore -->, chúng ta đã đề cập rằng trình biên dịch đảm bảo rằng các reference luôn có giá trị. Unsafe Rust có hai loại mới được gọi là _raw pointers_ tương tự như các reference. Giống như các reference, raw pointers có thể bất biến hoặc có thể thay đổi và được viết dưới dạng `*const T` và `*mut T`, tương ứng. Dấu sao không phải là toán tử dereference; nó là một phần của tên kiểu. Trong bối cảnh của raw pointers, _bất biến_ có nghĩa là con trỏ không thể được gán trực tiếp sau khi được dereference.

Khác với các reference và smart pointers, raw pointers:

- Được phép bỏ qua các quy tắc mượn bằng cách có cả các pointer bất biến và có thể thay đổi hoặc nhiều con trỏ có thể thay đổi ở cùng một vị trí
- Không được đảm bảo để trỏ đến bộ nhớ hợp lệ
- Được phép là null
- Không thực hiện bất kỳ dọn dẹp tự động nào

Bằng cách thoát khỏi việc Rust thực thi các đảm bảo này, bạn có thể từ bỏ sự an toàn được đảm bảo để đổi lấy hiệu suất cao hơn hoặc khả năng giao diện với một ngôn ngữ hoặc phần cứng khác mà các đảm bảo của Rust không áp dụng.

Listing 20-1 menunjukkan cara membuat raw pointer baku tidak berubah dan dapat diubah.

<Listing number="20-1" caption="Creating raw pointers with the raw borrow operators">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-01/src/main.rs:here}}
```

</Listing>

Lưu ý rằng chúng ta không bao gồm từ khóa `unsafe` trong code này. Chúng ta có thể tạo raw pointers trong code an toàn; chúng ta chỉ không thể dereference raw pointers bên ngoài một khối unsafe, như bạn sẽ thấy trong một chút.

Chúng ta đã tạo raw pointers bằng cách sử dụng các toán tử borrow thô: `&raw const num` tạo một `*const i32` immutable raw pointer, và `&raw mut num` tạo một `*mut i32` mutable raw pointer. Vì chúng ta đã tạo chúng trực tiếp từ một biến cục bộ, chúng ta biết rằng những raw pointers cụ thể này hợp lệ, nhưng chúng ta không thể đưa ra giả định đó về bất kỳ raw pointer nào.

Để chứng minh điều này, tiếp theo chúng ta sẽ tạo một raw pointer mà chúng ta không thể chắc chắn tính hợp lệ của nó, sử dụng từ khóa `as` để cast một giá trị thay vì sử dụng toán tử borrow thô. Listing 20-2 menunjukkan cara membuat raw pointer ke lokasi memori arbitrer. Cố gắng sử dụng memori arbitrer tidak terdefinisi: Mungkin ada data di alamat itu atau mungkin tidak, trình biên dịch mungkin mengoptimalkan code sehingga tidak ada akses memori, atau program mungkin berakhir dengan kesalahan segmentasi. Biasanya, tidak ada alasan bagus untuk menulis code seperti ini, terutama dalam kasus di mana Anda dapat menggunakan toán tuk borrow thô sebagai gantinya, tetapi ini dimungkinkan.

<Listing number="20-2" caption="Creating a raw pointer to an arbitrary memory address">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-02/src/main.rs:here}}
```

</Listing>

Ingat bahwa kami dapat membuat raw pointers dalam kode aman, tetapi kami tidak dapat dereference raw pointers dan membaca data yang ditunjuk. Dalam Listing 20-3, kami menggunakan toán tử dereference `*` pada raw pointer yang memerlukan blok `unsafe`.

<Listing number="20-3" caption="Dereferencing raw pointers within an `unsafe` block">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-03/src/main.rs:here}}
```

</Listing>

Membuat pointer tidak menimbulkan bahaya; hanya ketika kami mencoba mengakses nilai yang ditunjuknya, kami mungkin berakhir dengan nilai yang tidak valid.

Perhatikan juga bahwa di Listings 20-1 dan 20-3, kami membuat `*const i32` dan `*mut i32` raw pointers yang keduanya menunjuk ke lokasi memori yang sama, tempat `num` disimpan. Jika kami mencoba membuat reference baku tidak dapat diubah dan dapat diubah untuk `num`, code tidak akan dikompilasi karena aturan kepemilikan Rust tidak memungkinkan reference yang dapat diubah pada waktu yang sama dengan reference baku tidak dapat diubah. Dengan raw pointers, kami dapat membuat pointer yang dapat diubah dan pointer baku tidak dapat diubah ke lokasi yang sama dan mengubah data melalui pointer yang dapat diubah, yang dapat membuat race condition. Hati-hati!

Dengan semua bahaya ini, mengapa Anda akan pernah menggunakan raw pointers? Satu kasus penggunaan utama adalah ketika menghubungkan dengan kode C, seperti yang akan Anda lihat di bagian berikutnya. Kasus lain adalah ketika membangun abstraksi aman yang borrow checker tidak mengerti. Kami akan memperkenalkan function unsafe dan kemudian melihat contoh abstraksi aman yang menggunakan code unsafe.

### Calling an Unsafe Function or Method

Jenis operasi kedua yang dapat Anda lakukan dalam blok unsafe adalah memanggil function unsafe. Function dan method unsafe terlihat persis seperti function dan method biasa, tetapi mereka memiliki `unsafe` tambahan sebelum sisa definisi. Kata kunci `unsafe` dalam konteks ini menunjukkan bahwa function memiliki persyaratan yang perlu kami penuhi ketika kami memanggil function ini, karena Rust tidak dapat menjamin bahwa kami telah memenuhi persyaratan ini. Dengan memanggil function unsafe dalam blok `unsafe`, kami mengatakan bahwa kami telah membaca dokumentasi function ini dan kami bertanggung jawab untuk memenuhi kontrak function.

Berikut adalah function unsafe bernama `dangerous` yang tidak melakukan apa pun di tubuhnya:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-01-unsafe-fn/src/main.rs:here}}
```

Kami harus memanggil function `dangerous` dalam blok `unsafe` yang terpisah. Jika kami mencoba memanggil `dangerous` tanpa blok `unsafe`, kami akan mendapatkan kesalahan:

```console
{{#include ../listings/ch20-advanced-features/output-only-01-missing-unsafe/output.txt}}
```

Dengan blok `unsafe`, kami menyatakan kepada Rust bahwa kami telah membaca dokumentasi function, kami memahami cara menggunakannya dengan benar, dan kami telah memverifikasi bahwa kami memenuhi kontrak function.

Untuk melakukan operasi unsafe dalam tubuh function `unsafe`, Anda masih perlu menggunakan blok `unsafe`, sama seperti dalam function biasa, dan trình biên dịch akan memperingatkan Anda jika Anda lupa. Ini membantu kami menjaga blok `unsafe` sekecil mungkin, karena operasi unsafe mungkin tidak diperlukan di seluruh tubuh function.

#### Creating a Safe Abstraction over Unsafe Code

Hanya karena function berisi code unsafe tidak berarti kami perlu menandai seluruh function sebagai unsafe. Sebenarnya, membungkus code unsafe dalam function aman adalah abstraksi umum. Sebagai contoh, mari kita pelajari function `split_at_mut` dari thư viện chuẩn, yang memerlukan beberapa code unsafe. Kami akan mengeksplorasi bagaimana kami dapat mengimplementasikannya. Method aman ini didefinisikan pada slice yang dapat diubah: Ia mengambil satu slice dan membuatnya menjadi dua dengan membagi slice pada indeks yang diberikan sebagai argumen. Listing 20-4 menunjukkan cara menggunakan `split_at_mut`.

<Listing number="20-4" caption="Using the safe `split_at_mut` function">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-04/src/main.rs:here}}
```

</Listing>

Kami tidak dapat mengimplementasikan function ini hanya menggunakan safe Rust. Percobaan mungkin terlihat seperti Listing 20-5, yang tidak akan dikompilasi. Untuk kesederhanaan, kami akan mengimplementasikan `split_at_mut` sebagai function daripada method dan hanya untuk slice nilai `i32` daripada jenis generic `T`.

<Listing number="20-5" caption="An attempted implementation of `split_at_mut` using only safe Rust">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-05/src/main.rs:here}}
```

</Listing>

Function ini pertama kali mendapatkan panjang total slice. Kemudian, ia menyatakan bahwa indeks yang diberikan sebagai parameter berada dalam slice dengan memeriksa apakah kurang dari atau sama dengan panjang. Pernyataan berarti bahwa jika kami melewatkan indeks yang lebih besar dari panjang untuk membagi slice, function akan panik sebelum mencoba menggunakan indeks itu.

Kemudian, kami mengembalikan dua slice yang dapat diubah dalam tupel: satu dari awal slice asli ke indeks `mid` dan yang lain dari `mid` ke akhir slice.

Ketika kami mencoba mengompilasi code di Listing 20-5, kami akan mendapatkan kesalahan:

```console
{{#include ../listings/ch20-advanced-features/listing-20-05/output.txt}}
```

Borrow checker Rust tidak dapat memahami bahwa kami meminjam bagian berbeda dari slice; ia hanya tahu bahwa kami meminjam dari slice yang sama dua kali. Meminjam bagian berbeda dari slice pada dasarnya aman karena dua slice tidak tumpang tindih, tetapi Rust tidak cukup pintar untuk mengetahui hal ini. Ketika kami tahu code aman, tetapi Rust tidak, saatnya menggunakan code unsafe.

Listing 20-6 menunjukkan cara menggunakan blok `unsafe`, raw pointer, dan beberapa panggilan ke function unsafe untuk membuat implementasi `split_at_mut` berfungsi.

<Listing number="20-6" caption="Using unsafe code in the implementation of the `split_at_mut` function">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-06/src/main.rs:here}}
```

</Listing>

Ingat dari bagian ["The Slice Type"][the-slice-type]<!-- ignore --> di Chap 4 bahwa slice adalah pointer ke beberapa data dan panjang slice. Kami menggunakan method `len` untuk mendapatkan panjang slice dan method `as_mut_ptr` untuk mengakses raw pointer dari slice. Dalam hal ini, karena kami memiliki slice yang dapat diubah untuk nilai `i32`, `as_mut_ptr` mengembalikan raw pointer dengan jenis `*mut i32`, yang kami simpan dalam variabel `ptr`.

Kami menjaga pernyataan bahwa indeks `mid` berada dalam slice. Kemudian, kami sampai pada code unsafe: Function `slice::from_raw_parts_mut` mengambil raw pointer dan panjang, dan membuat slice. Kami menggunakan function ini untuk membuat slice yang dimulai dari `ptr` dan memiliki panjang `mid` item. Kemudian, kami memanggil method `add` pada `ptr` dengan `mid` sebagai argumen untuk mendapatkan raw pointer yang dimulai pada `mid`, dan kami membuat slice menggunakan pointer itu dan jumlah item yang tersisa setelah `mid` sebagai panjang.

Function `slice::from_raw_parts_mut` tidak aman karena mengambil raw pointer dan harus dipercaya bahwa pointer ini valid. Method `add` pada raw pointers juga tidak aman karena harus dipercaya bahwa lokasi offset juga merupakan pointer valid. Oleh karena itu, kami harus menempatkan blok `unsafe` di sekitar panggilan kami ke `slice::from_raw_parts_mut` dan `add` sehingga kami dapat memanggilnya. Dengan melihat code dan dengan menambahkan pernyataan bahwa `mid` harus kurang dari atau sama dengan `len`, kami dapat mengatakan bahwa semua raw pointers yang digunakan dalam blok `unsafe` akan menjadi pointer valid ke data dalam slice. Ini adalah penggunaan `unsafe` yang dapat diterima dan sesuai.

Perhatikan bahwa kami tidak perlu menandai function `split_at_mut` yang dihasilkan sebagai `unsafe`, dan kami dapat memanggil function ini dari safe Rust. Kami telah membuat abstraksi aman untuk code unsafe dengan implementasi function yang menggunakan code `unsafe` dengan cara yang aman, karena hanya membuat pointer valid dari data yang function ini memiliki akses.

Sebaliknya, penggunaan `slice::from_raw_parts_mut` dalam Listing 20-7 mungkin akan crash ketika slice digunakan. Code ini mengambil lokasi memori arbitrer dan membuat slice dengan panjang 10.000 item.

<Listing number="20-7" caption="Creating a slice from an arbitrary memory location">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-07/src/main.rs:here}}
```

</Listing>

Kami tidak memiliki memori di lokasi arbitrer ini, dan tidak ada jaminan bahwa slice yang dibuat code ini berisi nilai `i32` yang valid. Mencoba menggunakan `values` seolah-olah itu adalah slice valid menghasilkan perilaku yang tidak terdefinisi.

#### Using `extern` Functions to Call External Code

Terkadang code Rust Anda mungkin perlu berinteraksi dengan code yang ditulis dalam bahasa lain. Untuk ini, Rust memiliki kata kunci `extern` yang memfasilitasi pembuatan dan penggunaan _Foreign Function Interface (FFI)_, yang merupakan cara untuk bahasa pemrograman mendefinisikan function dan memungkinkan bahasa pemrograman yang berbeda (asing) untuk memanggil function tersebut.

Listing 20-8 menunjukkan cara menyiapkan integrasi dengan function `abs` dari thư viện standar C. Function yang dideklarasikan dalam blok `extern` umumnya tidak aman untuk dipanggil dari code Rust, jadi blok `extern` juga harus ditandai `unsafe`. Alasannya adalah bahwa bahasa lain tidak menegakkan aturan dan jaminan Rust, dan Rust tidak dapat memeriksanya, jadi tanggung jawab jatuh pada programmer untuk memastikan keselamatan.

<Listing number="20-8" file-name="src/main.rs" caption="Declaring and calling an `extern` function defined in another language">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-08/src/main.rs}}
```

</Listing>

Dalam blok `unsafe extern "C"`, kami mencantumkan nama dan signature function eksternal dari bahasa lain yang ingin kami panggil. Bagian `"C"` mendefinisikan _application binary interface (ABI)_ mana yang digunakan function eksternal: ABI mendefinisikan cara memanggil function di tingkat assembly. ABI `"C"` adalah yang paling umum dan mengikuti ABI bahasa pemrograman C. Informasi tentang semua ABI yang didukung Rust tersedia di [the Rust Reference][ABI].

Setiap item yang dideklarasikan dalam blok `unsafe extern` secara implisit tidak aman. Namun, beberapa function FFI *aman* untuk dipanggil. Misalnya, function `abs` dari thư viện standar C tidak memiliki pertimbangan keselamatan memori apa pun, dan kami tahu itu dapat dipanggil dengan `i32` apa pun. Dalam kasus seperti ini, kami dapat menggunakan kata kunci `safe` untuk mengatakan bahwa function spesifik ini aman untuk dipanggil meskipun berada dalam blok `unsafe extern`. Setelah kami membuat perubahan itu, memanggilnya tidak lagi memerlukan blok `unsafe`, seperti yang ditunjukkan dalam Listing 20-9.

<Listing number="20-9" file-name="src/main.rs" caption="Explicitly marking a function as `safe` within an `unsafe extern` block and calling it safely">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-09/src/main.rs}}
```

</Listing>

Menandai function sebagai `safe` tidak membuat function itu aman secara inheren! Sebaliknya, ini adalah janji yang Anda buat kepada Rust bahwa function itu aman. Tetap menjadi tanggung jawab Anda untuk memastikan janji itu ditepati!

#### Calling Rust Functions from Other Languages

Kami juga dapat menggunakan `extern` untuk membuat antarmuka yang memungkinkan bahasa lain memanggil function Rust. Daripada membuat seluruh blok `extern`, kami menambahkan kata kunci `extern` dan menentukan ABI yang akan digunakan tepat sebelum kata kunci `fn` untuk function yang relevan. Kami juga perlu menambahkan anotasi `#[unsafe(no_mangle)]` untuk memberitahu trình biên dịch Rust untuk tidak mengubah nama function ini. _Mangling_ adalah ketika trình biên dịch mengubah nama yang kami berikan function menjadi nama berbeda yang berisi informasi lebih untuk bagian lain dari proses kompilasi yang dikonsumsi tetapi kurang dapat dibaca oleh manusia. Setiap trình biên dịch bahasa pemrograman mengubah nama sedikit berbeda, jadi agar function Rust dapat dinamai oleh bahasa lain, kami harus menonaktifkan name mangling trình biên dịch Rust. Ini tidak aman karena mungkin ada collision nama di seluruh thư viện tanpa mangling bawaan, jadi tanggung jawab kami untuk memastikan nama yang kami pilih aman untuk diekspor tanpa mangling.

Dalam contoh berikut, kami membuat function `call_from_c` dapat diakses dari code C, setelah dikompilasi ke thư viện bersama dan ditautkan dari C:

```
#[unsafe(no_mangle)]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```

Penggunaan `extern` ini memerlukan `unsafe` hanya dalam atribut, bukan pada blok `extern`.

### Accessing or Modifying a Mutable Static Variable

Dalam buku ini, kami belum berbicara tentang variabel global, yang didukung Rust tetapi dapat bermasalah dengan aturan kepemilikan Rust. Jika dua thread mengakses variabel global yang dapat diubah yang sama, dapat menyebabkan race condition data.

Dalam Rust, variabel global disebut _static_ variables. Listing 20-10 menunjukkan contoh deklarasi dan penggunaan variabel static dengan string slice sebagai nilai.

<Listing number="20-10" file-name="src/main.rs" caption="Defining and using an immutable static variable">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-10/src/main.rs}}
```

</Listing>

Static variables mirip dengan constants, yang kami bahas dalam bagian ["Declaring Constants"][constants]<!-- ignore --> di Chap 3. Nama static variables berada dalam `SCREAMING_SNAKE_CASE` menurut konvensi. Static variables hanya dapat menyimpan reference dengan lifetime `'static`, yang berarti trình biên dịch Rust dapat mengetahui lifetime dan kami tidak diperlukan untuk menganotasinya secara eksplisit. Mengakses static variable baku tidak dapat diubah aman.

Perbedaan halus antara constants dan static variables baku tidak dapat diubah adalah bahwa nilai dalam static variable memiliki alamat tetap dalam memori. Menggunakan nilai akan selalu mengakses data yang sama. Constants, di sisi lain, diperbolehkan untuk menduplikasi data mereka setiap kali digunakan. Perbedaan lain adalah bahwa static variables dapat diubah. Mengakses dan memodifikasi static variables yang dapat diubah adalah _unsafe_. Listing 20-11 menunjukkan cara mendeklarasikan, mengakses, dan memodifikasi static variable yang dapat diubah bernama `COUNTER`.

<Listing number="20-11" file-name="src/main.rs" caption="Reading from or writing to a mutable static variable is unsafe.">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-11/src/main.rs}}
```

</Listing>

Seperti halnya variabel biasa, kami menentukan mutability menggunakan kata kunci `mut`. Setiap code yang membaca atau menulis dari `COUNTER` harus berada dalam blok `unsafe`. Code di Listing 20-11 dikompilasi dan mencetak `COUNTER: 3` seperti yang kami harapkan karena single-threaded. Memiliki multiple threads mengakses `COUNTER` kemungkinan akan menghasilkan race condition data, jadi itu adalah perilaku yang tidak terdefinisi. Oleh karena itu, kami perlu menandai seluruh function sebagai `unsafe` dan mendokumentasikan batasan keselamatan sehingga siapa pun yang memanggil function mengetahui apa yang mereka boleh dan tidak boleh lakukan dengan aman.

Setiap kali kami menulis function unsafe, adalah idiomatis untuk menulis komentar yang dimulai dengan `SAFETY` dan menjelaskan apa yang perlu dilakukan pemanggil untuk memanggil function dengan aman. Demikian pula, setiap kali kami melakukan operasi unsafe, adalah idiomatis untuk menulis komentar yang dimulai dengan `SAFETY` untuk menjelaskan bagaimana aturan keselamatan dipertahankan.

Selain itu, trình biên dịch akan menolak secara default upaya apa pun untuk membuat reference ke static variable yang dapat diubah melalui compiler lint. Anda harus secara eksplisit keluar dari proteksi lint itu dengan menambahkan anotasi `#[allow(static_mut_refs)]` atau mengakses static variable yang dapat diubah melalui raw pointer yang dibuat dengan salah satu operator borrow raw. Itu termasuk kasus di mana reference dibuat secara tidak terlihat, seperti saat digunakan dalam `println!` dalam daftar code ini. Memerlukan reference ke static variables yang dapat diubah untuk dibuat melalui raw pointers membantu membuat persyaratan keselamatan untuk menggunakannya lebih jelas.

Dengan data yang dapat diubah yang dapat diakses secara global, sulit untuk memastikan bahwa tidak ada race condition data, itulah sebabnya Rust menganggap static variables yang dapat diubah sebagai tidak aman. Jika memungkinkan, lebih baik menggunakan teknik concurrency dan smart pointers thread-safe yang kami bahas di Chap 16 sehingga trình biên dịch memeriksa bahwa akses data dari thread berbeda dilakukan dengan aman.

### Implementing an Unsafe Trait

Kami dapat menggunakan `unsafe` untuk mengimplementasikan trait unsafe. Trait adalah unsafe ketika setidaknya satu method-nya memiliki beberapa invariant yang tidak dapat diverifikasi oleh trình biên dịch. Kami mendeklarasikan bahwa trait adalah `unsafe` dengan menambahkan kata kunci `unsafe` sebelum `trait` dan menandai implementasi trait sebagai `unsafe` juga, seperti yang ditunjukkan dalam Listing 20-12.

<Listing number="20-12" caption="Defining and implementing an unsafe trait">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-12/src/main.rs:here}}
```

</Listing>

Dengan menggunakan `unsafe impl`, kami menjanjikan bahwa kami akan mempertahankan invariant yang tidak dapat diverifikasi oleh trình biên dịch.

Sebagai contoh, ingat kembali marker traits `Send` dan `Sync` yang kami bahas dalam bagian ["Extensible Concurrency with `Send` and `Sync`"][send-and-sync]<!-- ignore --> di Chap 16: Trình biên dịch secara otomatis mengimplementasikan traits ini jika tipe kami terdiri sepenuhnya dari tipe lain yang mengimplementasikan `Send` dan `Sync`. Jika kami mengimplementasikan tipe yang berisi tipe yang tidak mengimplementasikan `Send` atau `Sync`, seperti raw pointers, dan kami ingin menandai tipe itu sebagai `Send` atau `Sync`, kami harus menggunakan `unsafe`. Rust tidak dapat memverifikasi bahwa tipe kami mempertahankan jaminan bahwa tipe itu dapat dikirim dengan aman di seluruh thread atau diakses dari multiple threads; oleh karena itu, kami perlu melakukan pemeriksaan tersebut secara manual dan menunjukkan demikian dengan `unsafe`.

### Accessing Fields of a Union

Tindakan terakhir yang hanya berfungsi dengan `unsafe` adalah mengakses field dari union. *union* mirip dengan `struct`, tetapi hanya satu field yang dideklarasikan digunakan dalam instance tertentu pada saat yang sama. Union terutama digunakan untuk antarmuka dengan union dalam code C. Mengakses union fields tidak aman karena Rust tidak dapat menjamin tipe data yang saat ini disimpan dalam instance union. Anda dapat mempelajari lebih lanjut tentang union di [the Rust Reference][unions].

### Using Miri to Check Unsafe Code

Ketika menulis code unsafe, Anda mungkin ingin memeriksa bahwa apa yang telah Anda tulis sebenarnya aman dan benar. Salah satu cara terbaik untuk melakukan hal tersebut adalah menggunakan Miri, alat resmi Rust untuk mendeteksi perilaku yang tidak terdefinisi. Sementara borrow checker adalah alat _static_ yang bekerja pada waktu compile, Miri adalah alat _dynamic_ yang bekerja pada waktu runtime. Ia memeriksa code Anda dengan menjalankan program Anda, atau test suite-nya, dan mendeteksi ketika Anda melanggar aturan yang dipahaminya tentang bagaimana Rust seharusnya bekerja.

Menggunakan Miri memerlukan build nightly Rust (yang kami bicarakan lebih detail di [Appendix G: How Rust is Made and "Nightly Rust"][nightly]<!-- ignore -->). Anda dapat menginstal versi nightly Rust dan alat Miri dengan mengetik `rustup +nightly component add miri`. Ini tidak mengubah versi Rust yang digunakan proyek Anda; itu hanya menambahkan alat ke sistem Anda sehingga Anda dapat menggunakannya kapan pun Anda mau. Anda dapat menjalankan Miri pada proyek dengan mengetik `cargo +nightly miri run` atau `cargo +nightly miri test`.

Sebagai contoh dari betapa bermanfaatnya hal ini, pertimbangkan apa yang terjadi ketika kami menjalankannya terhadap Listing 20-7.

```console
{{#include ../listings/ch20-advanced-features/listing-20-07/output.txt}}
```

Miri dengan benar memperingatkan kami bahwa kami mengcast integer menjadi pointer, yang mungkin menjadi masalah, tetapi Miri tidak dapat menentukan apakah masalah ada karena tidak tahu bagaimana pointer berasal. Kemudian, Miri mengembalikan error di mana Listing 20-7 memiliki perilaku yang tidak terdefinisi karena kami memiliki dangling pointer. Berkat Miri, kami sekarang tahu ada risiko perilaku yang tidak terdefinisi, dan kami dapat berpikir tentang cara membuat code aman. Dalam beberapa kasus, Miri bahkan dapat memberikan rekomendasi tentang cara memperbaiki error.

Miri tidak menangkap semuanya yang mungkin Anda lakukan dengan salah saat menulis code unsafe. Miri adalah alat dynamic analysis, jadi hanya menangkap masalah dengan code yang benar-benar berjalan. Itu berarti Anda perlu menggunakannya bersama dengan teknik testing yang baik untuk meningkatkan kepercayaan diri tentang code unsafe yang telah Anda tulis. Miri juga tidak mencakup setiap cara yang mungkin code Anda tidak sehat.

Dengan kata lain: Jika Miri _menangkap_ masalah, Anda tahu ada bug, tetapi hanya karena Miri _tidak menangkap_ bug tidak berarti tidak ada masalah. Ini dapat menangkap banyak hal. Coba jalankan pada contoh code unsafe lainnya dalam bab ini dan lihat apa yang dikatakannya!

Anda dapat mempelajari lebih lanjut tentang Miri di [its GitHub repository][miri].

<!-- Old headings. Do not remove or links may break. -->

<a id="when-to-use-unsafe-code"></a>

### Using Unsafe Code Correctly

Menggunakan `unsafe` untuk menggunakan salah satu lima superpowers yang dibahas baru-baru ini bukan hal yang salah atau bahkan tidak disetujui, tetapi lebih rumit untuk membuat code `unsafe` benar karena trình biên dịch tidak dapat membantu mempertahankan keselamatan memori. Ketika Anda memiliki alasan untuk menggunakan code `unsafe`, Anda dapat melakukannya, dan memiliki anotasi `unsafe` eksplisit membuat lebih mudah untuk melacak sumber masalah ketika itu terjadi. Setiap kali Anda menulis code unsafe, Anda dapat menggunakan Miri untuk membantu Anda lebih yakin bahwa code yang telah Anda tulis mempertahankan aturan Rust.

Untuk eksplorasi yang jauh lebih dalam tentang cara bekerja secara efektif dengan unsafe Rust, baca panduan resmi Rust untuk `unsafe`, [The Rustonomicon][nomicon].

[dangling-references]: ch04-02-references-and-borrowing.html#dangling-references
[ABI]: ../reference/items/external-blocks.html#abi
[constants]: ch03-01-variables-and-mutability.html#declaring-constants
[send-and-sync]: ch16-04-extensible-concurrency-sync-and-send.html
[the-slice-type]: ch04-03-slices.html#the-slice-type
[unions]: ../reference/items/unions.html
[miri]: https://github.com/rust-lang/miri
[editions]: appendix-05-editions.html
[nightly]: appendix-07-nightly-rust.html
[nomicon]: https://doc.rust-lang.org/nomicon/
