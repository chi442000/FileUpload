# FileUpload


### Contents
- [Khái niệm FileUpload](https://github.com/chi442000/fileupload#Concept)
- [Nguyên nhân](https://github.com/chi442000/fileupload#reason)
- [Mức độ nguy hiểm ](https://github.com/chi442000/fileupload#dangerousness)
- [Khai thác lỗ hổng](https://github.com/chi442000/fileupload#exploit)
    - [In-band SQLi](https://github.com/chi442000/SQLi#In-band-SQLi)
    - [Inferential SQLi (Blind SQLi)](https://github.com/chi442000/SQLi#Inferential-SQLi)
    - [Out-of-band SQLi](https://github.com/chi442000/SQLi#out-of-band-sqli)
- [Các phòng chống SQLi](https://github.com/chi442000/SQLi#prevention)
- [Kết luận](https://github.com/chi442000/XSS#in-conclusion)


### Concept
- Lỗ hổng File Upload xảy ra khi máy chỉ web cho phép người dùng upload file lên hệ thống mà không xác thực đầy đủ những thứ như tên, loại, nội dung hoặc kích thước của chúng. Khi không có những hạn chế đối với file thì attacker có thể lợi dụng điều đó để thực thi các kịch bản tấn công, thậm chí là thực thi mã từ xa.

- Web shell là một tập lệnh độc hại cho phép attacker thực hiện các lệnh tùy ý trên máy chủ web từ xa chỉ bằng cách gửi các yêu cấp HTTP đến đúng điểm cuối. Nếu có thể upload web shell thành công, bạn có toàn quyền kiểm soát máy chủ.

### Reason
 
Lỗ hổng chạy Script file upload xảy ra khi thỏa mãn những điều kiện sau:

- File upload được lưu vào public directory

- Có thể upload các file script

Đối với uploader, khi tạo ra một ứng dụng phù hợp hai điều kiện trên thì sẽ trở thành nguyên nhân sinh ra lỗ hổng. Vì thế, việc không thỏa mãn ít nhất một trong hai điều kiện nêu trên trở thành đối sách giải quyết của lỗ hổng này.

### Dangerousness

Ảnh hưởng của lỗ hổng file upload phụ thuộc vào hai yếu tố chính:

- Phần của file mà trang web không thể xác định được đúng cách

- Những hạn chế được áp dụng lên file sau khi đã upload thành công

Trường hợp kích thước của tệp nằm ngoài ngưỡng dự kiến có thể kích hoạt tấn công DoS, attacker có thể tấn công lấp đầy không gian đĩa có sẵn.

Trường file được upload không được xác thực đúng cách có thể cho phép attacker ghi đè các tệp quan trọng bằng cách tải lên tệp có cùng tên. Nếu máy chủ cũng dễ bị truy cập thư mục thì attacker có thể upload file vào những vị trí không xác định được.

Trường hợp tệ nhất, loại file không được xác định đúng cách và cấu hình máy chủ cho phép file được thực thi dưới dạng mã (như .php hay .jsp). Trong trường hợp này, attacker có thể upload file có chức năng như một web shell và có thể cho attacker có toàn quyền kiểm soát máy chú.

Tóm lại, lỗ hổng file upload xảy ra có thể:

- Làm lộ các thông tin nội bộ, chẳng hạn như đường dẫn của máy chủ

- Là tiền đề dẫn đến các lỗ hổng khác

- Khiến attacker chiếm quyền điều khiển hệ thống

### Exploit
Đây là một số cách bypass mình đã tìm hiểu được và đã tạo được lab demo
- Không lọc dữ liệu đầu vào (Có thể tải lên bất kì file nào)
  Chúng ta có thể up lên 1 file shell bất kỳ và khai thác trực tiếp. Có một lưu ý đặc biệt đó là file thực thi phải phù hợp với Web server đang sử dụng. Bạn không thể up 1 file .jsp với Apache đúng không? À, có, nhưng nó sẽ không thực thi được. Hãy thử code 1 form upload đơn giản và up shell viết bằng .jsp xem thử đi.
- Bypass Check Header
  Đoạn code sử dụng sẽ trông như thế này:
  ![example](1.png)
Để bypass qua filter này, hãy dùng burp suite sửa content type thành image/...

Ví dụ web cho phép bạn upload file .png, hãy up một cái shell lên, chặn cái request đó lại và sửa:

+Content-type: image/png+
Sau đó forward nó là được
- Bypass Check the blacklisted
Blacklist là danh sách những file bị cấm tải lên. Giả sử đoạn code đó sẽ trông như sau:
![example](2.png)
Bypass qua filter này có thể Upload file có đuôi .php3, .php4, .php5, .pHp, ... Lúc đó tôi đã tự đặt ra câu hỏi là tại sao lại có những số 3 4 5 (ở các tài liệu đọc được), thế 7 8 9 10 thì có được không? Một câu hỏi giúp ta tìm hiểu nhiều hơn, nên tôi sẽ không trả lời nó ở đây.
- Bypass Check the whitelisted
Ngược lại với Blacklist thì Whitelist sẽ là danh sách những file được cho phép tải lên. Đây là filter khó bypass nhất
![example](3.png)
Đúng thế đó. Tôi đã thử nhiểu cách như là sử dụng double extension (Ví dụ như web chỉ cho bạn up file .png thì bạn thêm .png vào file shell, shell.php.png -> shell.png.php), sử dụng null byte (shell.php%00.jpg) nhưng đều không được. Nó có nhiều lý do. Với việc sử dụng code PHP7 thì một số cách bypass kia đã không thể thực hiện được. Nếu bạn tạo lab về lỗ hổng này có thể sử dụng một phiên bản PHP thấp hơn. Tuy nhiên thì tôi vẫn cần nhắc đến việc sử dụng 2 cách này để bypass vì đa số các web vẫn được code bằng PHP5, và có thể bị dính lỗ hổng này.

Một cách khác, đó là cấu hình file .htaccess. Có lẽ đây là cách duy nhất để bypass whitelist với PHP cao cấp
- Bypass check image giả hay thật
Việc xác định định dạng file ngoài việc dựa vào extension, người ta còn dựa vào signature file. Việc xác định signature file sẽ dựa vào 8 bits đầu của file. Code của nó sẽ trông như thế này.
![example](4.png)
Để bypass qua trường hợp này, hãy thêm một đoạn định dạng vào đầu file shell. Trong trường hợp web upload cho phép upload gif thì thêm GIF89a vào đầu tiên, đánh lừa rằng một file gif đã được tải lên:
![example](5.png)



#### **In-band SQLi**

Đây là dạng tấn công phổ biến nhất và cũng dễ để khai thác lỗ hổng SQL Injection nhất
Xảy ra khi hacker có thể tổ chức tấn công và thu thập kết quả trực tiếp trên cùng một kênh liên lạc
In-Band SQLi chia làm 2 loại chính:
- Error-based SQLi
- Union-based SQLi

*Error-based SQLi*
- Là một kỹ thuật tấn công SQL Injection dựa vào thông báo lỗi được trả về từ Database Server có chứa thông tin về cấu trúc của cơ sở dữ liệu.
- Trong một vài trường hợp, chỉ một mình Error-based là đủ cho hacker có thể liệt kê được các thuộc tính của cơ sở dữ liệu

![example](2.png)

*Union-based SQLi*

Là một kỹ thuật tấn công SQL Injection dựa vào sức mạnh của toán tử UNION trong ngôn ngữ SQL cho phép tổng hợp kết quả của 2 hay nhiều câu truy vấn SELECTION trong cùng 1 kết quả và được trả về như một phần của HTTP response

![example](3.png)

#### Inferential SQLi

- Không giống như In-band SQLi, Inferential SQL Injection tốn nhiều thời gian hơn cho việc tấn công do không có bất kì dữ liệu nào được thực sự trả về thông qua web application và hacker thì không thể theo dõi kết quả trực tiếp như kiểu tấn công In-band
- Thay vào đó, kẻ tấn công sẽ cố gắng xây dựng lại cấu trúc cơ sở dữ liệu bằng việc gửi đi các payloads, dựa vào kết quả phản hồi của web application và kết quả hành vi của database server.
- Có 2 dạng tấn công chính:

*Blind-boolean-based*

*Blind-time-based SQLi*

**Blind-boolean-based**

- Là kĩ thuật tấn công SQL Injection dựa vào việc gửi các truy vấn tới cơ sở dữ liệu bắt buộc ứng dụng trả về các kết quả khác nhau phụ thuộc vào câu truy vấn là True hay False.
- Tuỳ thuộc kết quả trả về của câu truy vấn mà HTTP reponse có thể thay đổi, hoặc giữ nguyên
- Kiểu tấn công này thường chậm (đặc biệt với cơ sở dữ liệu có kích thước lớn) do người tấn công cần phải liệt kê từng dữ liệu, hoặc mò từng kí tự

![example](4.png)


**Time-based Blind SQLi**
- Time-base Blind SQLi là kĩ thuật tấn công dựa vào việc gửi những câu truy vấn tới cơ sở dữ liệu và buộc cơ sở dữ liệu phải chờ một khoảng thời gian (thường tính bằng giây) trước khi phản hồi.
- Thời gian phản hồi (ngay lập tức hay trễ theo khoảng thời gian được set) cho phép kẻ tấn công suy đoán kết quả truy vấn là TRUE hay FALSE
- Kiểu tấn công này cũng tốn nhiều thời gian tương tự như Boolean-based SQLi

#### Out-of-band SQLi

- Out-of-band SQLi không phải dạng tấn công phổ biến, chủ yếu bởi vì nó phụ thuộc vào các tính năng được bật trên Database Server được sở dụng bởi Web Application.
- Kiểu tấn công này xảy ra khi hacker không thể trực tiếp tấn công và thu thập kết quả trực tiếp trên cùng một kênh (In-band SQLi), và đặc biệt là việc phản hồi từ server là không ổn định
- Kiểu tấn công này phụ thuộc vào khả năng server thực hiện các request DNS hoặc HTTP để chuyển dữ liệu cho kẻ tấn công.
- Ví dụ như câu lệnh xp_dirtree trên Microsoft SQL Server có thể sử dụng để thực hiện DNS request tới một server khác do kẻ tấn công kiểm soát, hoặc Oracle Database’s UTL HTTP Package có thể sử dụng để gửi HTTP request từ SQL và PL/SQL tới server do kẻ tấn công làm chủ


### Prevention 

Mặc dù SQL rất nguy hại nhưng cũng dễ phòng chống. Gần đây, hầu như chúng ta ít viết SQL thuần mà toàn sử dụng ORM (Object-Relational Mapping) framework. Các framework web này sẽ tự tạo câu lệnh SQL nên hacker cũng khó tấn công hơn.

Tuy nhiên, có rất nhiều site vẫn sử dụng SQL thuần để truy cập dữ liệu. Đây chính là mồi ngon cho hacker. Để bảo vệ bản thân trước SQL Injection, ta có thể thực hiện các biện pháp sau.

- Lọc dữ liệu từ người dùng: Cách phòng chống này tương tự như XSS. Ta sử dụng filter để lọc các kí tự đặc biệt (; ” ‘) hoặc các từ khoá (SELECT, UNION) do người dùng nhập vào. Nên sử dụng thư viện/function được cung cấp bởi framework. Viết lại từ đầu vừa tốn thời gian vừa dễ sơ sót.
- Không cộng chuỗi để tạo SQL: Sử dụng parameter thay vì cộng chuỗi. Nếu dữ liệu truyền vào không hợp pháp, SQL Engine sẽ tự động báo lỗi, ta không cần dùng code để check.
- Không hiển thị exception, message lỗi: Hacker dựa vào message lỗi để tìm ra cấu trúc database. Khi có lỗi, ta chỉ hiện thông báo lỗi chứ đừng hiển thị đầy đủ thông tin về lỗi, tránh hacker lợi dụng.
- Phân quyền rõ ràng trong DB: Nếu chỉ truy cập dữ liệu từ một số bảng, hãy tạo một account trong DB, gán quyền truy cập cho account đó chứ đừng dùng account root hay sa. Lúc này, dù hacker có inject được sql cũng không thể đọc dữ liệu từ các bảng chính, sửa hay xoá dữ liệu.
- Backup dữ liệu thường xuyên: Các cụ có câu “cẩn tắc vô áy náy”. Dữ liệu phải thường xuyên được backup để nếu có bị hacker xoá thì ta vẫn có thể khôi phục được. Còn nếu cả dữ liệu backup cũng bị xoá luôn thì … chúc mừng bạn, update CV rồi tìm cách chuyển công ty thôi!
Mặc dù loại tấn công này được coi là một trong những loại nguy hiểm và rủi ro nhất, nhưng vẫn nên chuẩn bị một kế hoạch ngăn ngừa. Bởi vì sự phổ biến của cuộc tấn công này, có khá nhiều cách để ngăn chặn nó.


###  In conclusion


