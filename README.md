# Tổng khái SQL Injection (SQLi):

Trước hết , bạn cần phải hiểu được SQL Injection là gì ?Vì sao lại đặt là SQLi ?

SQL là ngôn ngữ lập trình dùng để quản lý các dữ liệu trong Cơ sở dữ liệu , thực chất cũng chỉ là một ngôn ngữ lập trình như C++ ,PHP ,... nhưng nó lại được thực hiện lệnh trong cơ sở dữ liệu 

Injection được dịch ra là tiêm . Nói tóm lại SQL Injection là tấn công vào trong cơ sở dữ liệu , mà cơ sở dữ liệu là nơi mà Server lưu trữ những thông tin cá nhân như password , username , ... dạng dạng thế

Về cấu trúc của database thì bạn hình dung là trong database có thể chứa nhiều bảng khác nhau , giả sử một database tên `product` sẽ có thể chứa những bảng(table) là `price` , `amount` ,... 

Trong bảng thì đây

Ví dụ cho dễ hình dung hơn , bạn phải đăng nhập username và password ở 1 trang web , khi bạn đăng nhập đúng thì trang web sẽ trả lại bạn thông tin của account đó 

Bạn nhập vào `username: Nguyen Kiet` mà bạn biết là có tồn tại  và một bất khẩu bất kì `password: abcxyz` thì câu lệnh SQL ở phía Server sẽ chạy câu lệnh SQL như sau:

`SELECT * FORM nguoi_dung WHERE username = 'Nguyen Kiet' AND password = 'abcxyz'`

đây là cú pháp tiêu chuẩn của SQL lấy tất cả dữ liệu từ bảng (table) **nguoi_dung** với điều kiện là `**username** = 'Nguyen#  Kiet'` và `**password** = 'abcxyz'` và tất nhiên nó sẽ không trả về gì cả bởi vì trong bảng `nguoi_dung` không có hàng nào với cột `username = Nguyen Kiet` và `password = abcxyz` cả 

Vậy nếu để phải biết cả username và password thì điều đó quá khó khăn , giả sử lúc này phía Back-End nó tồn tại lỗ hổng SQLi , khi bạn nhập vào ô username ở front-end là **test** thì nó sẽ lấy chữ **test** đó ghép vào chuỗi `SELECT * FORM nguoi_dung WHERE username = 'test'` thế này 

Vậy thì sẽ thế nào nếu chúng ta nhập vào `test'-- ` . Lúc này câu lệnh SQL thực thi sẽ là `SELECT * FORM nguoi_dung WHERE username = 'test'-- AND password = 'abcxyz'` .

Ở đây dấu `--` tức là tất cả những ký tự ở phía sau nó đều là comment và Server sẽ không quan tâm đến những comment đó vậy thì lúc này nó chỉ còn thực thi `SELECT * FORM nguoi_dung WHERE username = 'test'` . Server lúc này nó sẽ lục trong database tìm coi dòng nào mà có `username = test` không thôi , không cần quan tâm password ứng với `username:test` đó là gì cả 

Đó là một ví dụ điển hình về SQLi giúp bạn có thể hình dung một cách tổng quát , nói tóm lại , nó sẽ tấn công vào cơ sở dữ liệu và lấy tất cả thông tin trong cái cơ sở dữ liệu đó ra 


# Các dạng SQL Injection thường gặp:

**1. In-band SQL Injection** 

Trước tiên , In-band có nghĩa ở đây là cùng một kênh , tức là Dữ liệu đi vào (Input/payload) và dữ liệu đi ra (Output/Result) sẽ đều di chuyển trên một con đường và In-band SQLi tức là bạn tiêm lệnh vào trang web và trang web sẽ hiển thị ra kết quả ở ngay chính nó , nó không trả kết quả ra một web khác 

Ở trong In-band SQLi nó sẽ có 2 kỹ thuật khai thác là `UNION-based SQLi` và `Error-based SQLi` 

- `UNION-based SQLi` : đúng như tên gọi của nó , kĩ thuật này sử dụng toán tử `UNION` đê gộp kết quả trả về của hai câu lệnh truy vấn khác nhau lại 

Ví dụ: Ở đây chúng ta có 1 trang web là `shoppe.com` , và trang web này có 2 bảng để lưu một là sản phảm (table name: product) , hai là tài khoản người dùng (table name: users) , khi chúng ta truy cập vào đường dẫn để xem chi tiết một sản phẩm , ví dụ `shoppe.com/item?id=10` chẳng hạn 

Lúc này hệ thống back-end sẽ chạy câu lệnh SQL để lấy thông tin sản phẩm với id=10 ấy : `SELECT ten_san_pham, gia_tien FORM product WHERE id = 10` 

Kết quả màn hình sẽ in ra là: `Tên sản phẩm: Áo thun; Giá tiền: 100.000 Đồng`

Vấn đề xảy ra ở đây , Trang web này bị lỗi SQL Injection, Chúng ta sẽ không nhập số 10 đơn thuần mà sử dụng thêm `UNION` để lấy thêm dữ liệu từ bảng `users` , vậy chúng ta có thể lấy bằng cách nào?

ví dụ nếu chúng ta thực thi câu lệnh: `SELECT ten_san_pham, gia_tien FORM product WHERE id = 10 UNION SELECT username,password FROM users` thì lúc này màn hình sẽ in ra kết quả: `Dòng 1: Áo thun 100.000 Đồng ; Dòng 2: admin 123456; Dòng 3:user1 111111;.....`

Dòng 1: chính là kết quả từ câu lệnh SELECT đầu tiên , từ Dòng 2 trở đi , chính là kết quả trả về từ câu lệnh SELECT thứ 2 , lấy username và password từ bảng `users` 

Và `UNION-based SQLi` chính là thế , bằng cách sử dụng `UNION` chúng ta đã lừa trang web hiển thị danh sách tài khoản và mật khẩu từ bảng `users` tại nơi mà vốn dĩ chỉ dành để hiển thị tên sản phẩm và giá . Đó là bản chất của gộp kết quả 
