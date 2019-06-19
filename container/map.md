## 映射的定义

初识映射会很懵，因为在PHP中没有映射类型的定义。其实没那么复杂，任何复杂的类型在PHP中都可以用数组表示，映射也不例外。
```php
$array['name'] = '平也';
$array['sex'] = '1';
$array['age'] = '10';

//output
Array
(
    [name] => 平也
    [sex] => 1
    [age] => 10
)
```

映射其实就是有key有value的数组，在Go中的赋值也很类似，但需要提前声明该映射类型的键与值的类型，确保所有的键和值的赋值类型统一，否则会报错。
```go
array := make(map[string]string)
array["name"] = "平也"
array["sex"] = "1"
array["age"] = "10"
fmt.Print(array) //output map[age:10 name:平也 sex:1]
```

在PHP中还有一种初始化数组的方法，就是将所有要存储的键与值赋值给变量。
```php
$array = [
	'name' => '平也',
	'sex' => '1',
	'age' => '10'
];
```

在Go中也有类似的初始化方法，但切记统一键与值的数据类型。
```go
array := map[string]string{
	"name": "平也",
	"sex":  "1",
	"age":  "10",
}
```

## 映射的遍历

在PHP中其实就是遍历数组的操作，foreach即可。
```php
$array = [
	'name' => '平也',
	'sex' => '1',
	'age' => '10'
];

foreach ($array as $key => $value) {
	print_r($array);
}

//output
Array
(
    [name] => 平也
    [sex] => 1
    [age] => 10
)
Array
(
    [name] => 平也
    [sex] => 1
    [age] => 10
)
Array
(
    [name] => 平也
    [sex] => 1
    [age] => 10
)
```

在Go中也可以像遍历数组那样遍历map，依然使用range关键字。
```go
array := map[string]string{
	"name": "平也",
	"sex":  "1",
	"age":  "10",
}
for v, k := range array {
	fmt.Print(k, v)
}
```

上篇文章讲到遍历时可以通过下划线来忽略键或值，如果只遍历键，下划线也可以省略。
```go
array := map[string]string{
	"name": "平也",
	"sex":  "1",
	"age":  "10",
}
for k := range array {
	fmt.Print(k)
}
//output sexagename
```

## 映射的取值

PHP中可以直接通过读数组的key来取值。
```php
$array = ['name' => 'pingye'];
echo $array['name']; //output pingye
```

在Go中的操作是一样的，与PHP不同的是，如果取了不存在的key，Go中默认输出空值，在PHP中就会产生warning警告。
```go
array := map[string]string{
	"name": "pingye",
	"sex":  "1",
	"age":  "10",
}
fmt.Print(array["name"]) //pingye
```

## 映射元素的删除

在PHP中的unset可以删除任何你想删除的数组元素，非常好用。
```php
$array = [
	'name' => '平也',
	'sex' => '1',
	'age' => '10'
];
unset($array['name']);
print_r($array);

//output
Array
(
    [sex] => 1
    [age] => 10
)
```

在Go中通过delete函数来删除map中的元素。
```go
array := map[string]string{
	"name": "pingye",
	"sex":  "1",
	"age":  "10",
}
delete(array, "name")
fmt.Print(array) //output map[age:10 sex:1]
```

## 清空map元素

在PHP中好像从来没有注意过是否把数组清空，很抱歉，我能想到的清空数组方法就是把空数组赋值给它。
```php
$array = [
	'name' => '平也',
	'sex' => '1',
	'age' => '10'
];
$array = [];
print_r($array);
//output
Array
(
)
```

然而，在Go中也没有提供清空map的函数，重新make一个map就行了，原来的map会被Go的垃圾回收机制清除掉，甚至比写一个清空的函数效率还高。