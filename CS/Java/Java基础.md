
### String
[菜鸟文档-String](https://www.runoob.com/java/java-string.html)
#### 字符串长度

用于获取有关对象的信息的方法称为访问器方法。

String 类的一个访问器方法是 ` length() ` 方法，它返回字符串对象包含的字符数。

#### 连接

有两种方法
```java
string1.concat(string2)
```

或者中 `+`


#### 格式化字符串
可以用静态方法 `format()` 来返回一个 `String`

```java
String fs;
fs = String.format("浮点型变量的值为 " +
                   "%f, 整型变量的值为 " +
                   " %d, 字符串变量的值为 " +
                   " %s", floatVar, intVar, stringVar);
```


#### 获得子字符串

- Usage:
```java
String substring(int beginIndex, int endIndex)
```


#### 拆分
```java
String[] split(String regex)
// 根据给定正则表达式的匹配拆分此字符串。
```
