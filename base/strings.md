## 字符串的赋值
在PHP中，字符串的赋值虽然只有一行，其实包含了两步，一是声明变量，二是赋值给变量，同一个变量可以任意重新赋值。
```php
$str = 'Hello World!';
$str = 'hia';
```

Go语言实现上述两步也可以用一行语句解决，就是通过标识`var`赋值时同时声明变量，切记等号右侧的字符串不能用单引号，对变量的后续赋值也不能再重新声明，否则会报错。除此之外，定义的变量不使用也会报错，从这点来看，Go还是比PHP严格很多的，规避了很多在开发阶段产生的性能问题。
```go
var str = "Hello World!"
str = "hia"
```

关于声明，Go提供了一种简化方式，不需要在行首写var，只需将等号左侧加上一个冒号就好了，切记这只是替代了声明语句，它并不会像PHP那样用一个赋值符号来统一所有的赋值操作。

```go
str := "Hello World!"
str = "hia"
```

## 字符串的输出
PHP中的输出非常简单，一个echo就搞定了。
```php
<?php
    echo 'Hello World!';
?>
```

而Go不一样的是，调用它的输出函数前需要先引入包`fmt`，这个包提供了非常全面的输入输出函数，如果只是输出普通字符串，那么和PHP对标的函数就是`Print`了，从这点来看，Go更有一种万物皆对象的感觉。
```go
import "fmt"

func main() {
	fmt.Print("Hello world!")
}
```

在PHP中还有一个格式化输出函数`sprintf`，可以用占位符替换字符串。
```php
echo sprintf('name:%s', '平也');  //name:平也
```
在Go中也有同名同功能的字符串格式化函数。
```go
fmt.Print(fmt.Sprintf("name:%s", "平也"))
```
官方提供的默认占位符有以下几种，感兴趣的同学可以自行了解。
```
bool:                    %t
int, int8 etc.:          %d
uint, uint8 etc.:        %d, %#x if printed with %#v
float32, complex64, etc: %g
string:                  %s
chan:                    %p
pointer:                 %p
```

## 字符串的相关操作
### 字符串长度

在PHP中通过`strlen`计算字符串长度。
```php
echo strlen('平也');  //output: 6
```
在Go中也有类似函数`len`。
```go
fmt.Print(len("平也"))   //output: 6
```
因为统计的是ASCII字符个数或字节长度，所以两个汉字被认定为长度6，如果要统计汉字的数量，可以使用如下方法，但要先引入`unicode/utf8`包。
```go
import (
	"fmt"
	"unicode/utf8"
)

func main() {
	fmt.Print(utf8.RuneCountInString("平也"))    //output: 2
}
```

### 字符串截取

PHP有一个`substr`函数用来截取任意一段字符串。
```php
echo substr('hello,world', 0, 3); //output: hel
```
Go中的写法有些特别，它是将字符串当做数组，截取其中的某段字符，比较麻烦的是，在PHP中可以将第二个参数设置为负数进行反向取值，但是Go无法做到。
```go
str := "hello,world"
fmt.Print(str[0:3])  //output: hel
```

### 字符串搜索

PHP中使用`strpos`查询某个字符串出现的位置。
```php
echo strpos('hello,world', 'l'); //output: 2
```
Go中需要先引入`strings`包，再调用`Index`函数来实现。
```go
fmt.Print(strings.Index("hello,world", "l")) //output: 2
```

### 字符串替换

PHP中替换字符串使用`str_replace`内置函数。
```php
echo str_replace('world', 'girl', 'hello,world'); //output: hello,girl
```
Go中依然需要使用`strings`包中的函数`Replace`，不同的是，第四个参数是必填的，它代表替换的次数，可以为0，代表不替换，但没什么意义。还有就是字符串在PHP中放在第三个参数，在Go中是第一个参数。
```go
fmt.Print(strings.Replace("hello,world", "world", "girl", 1)) //output: hello,girl
```

### 字符串连接

在PHP中最经典的就是用点来连接字符串。
```php
echo 'hello' . ',' . 'world'; //output: hello,world
```
在Go中用加号来连接字符串。
```go
fmt.Print("hello" + "," + "world") //output: hello,world
```
除此之外，还可以使用`strings`包中的`Join`函数连接，这种写法非常类似与PHP中的数组拼接字符串函数`implode`。
```go
str := []string{"hello", "world"}
fmt.Print(strings.Join(str, ",")) //output: hello,world
```

### 字符串编码

PHP中使用内置函数`base64_encode`来进行编码。
```php
echo base64_encode('hello, world'); //output: aGVsbG8sIHdvcmxk
```
在Go中要先引入`encoding/base64`包，并定义一个切片，再通过`StdEncoding.EncodeToString`函数对切片编码，比PHP要复杂一些。
```go
import (
	"encoding/base64"
	"fmt"
)

func main() {
	str := []byte("hello, world")
	fmt.Print(base64.StdEncoding.EncodeToString(str))
}
```