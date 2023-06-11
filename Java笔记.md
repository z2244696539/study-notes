### JodaTime

#### String 转 Date DateTIme

```java
DateTimeFormatter formatter = DateTimeFormat.forPattern("yyyy-MM-dd HH:mm:ss");
Date date = DateTime.parse(map.get("gmt_payment"), formatter).toDate();
```

### ==和equals

==:对于基本类型，比较的是值是否相等；对于引用类型，比较的是地址是否相等
equals：比较的是对象是否相等（不能比较基本类型，因为equals是Object超类中的方法，而Object是所有类的父类）

== 在基本数据类型：值内容, 引用类型时：地址
equals 重写：值内容 ， equals不重写：地址

## Java标识符

- 标识符由数字（0~9）和字母（A~Z 和 a~z）、美元符号（$）、下划线（_）
- 标识符的第一个符号为字母、下划线和美元符号，后面可以是任何字母、数字、美元符号或下划线。

## 类名和Java文件

- 如果一个类中的类修饰符不为public，则类名和文件名没有任何联系
- 一个Java文件可以有多个类，但是只能有一个public修饰的类且与文件名名称相同

## 循环

- 循环条件要以表达式的方式来体现，计算的结果是boolean类型，不能在条件里定义变量
- 循环条件必须是 boolean类型，int不能转成boolean
- 计数器如果是多个变量，用逗号分隔就可以了

~~~
while (int i<7){循环条件要以表达式的方式来体现，计算的结果是boolean类型，不能在条件里定义变量
	i++;
	system.out.println ( "i is "+i);
}
int j=3;
while(j) {
	System.out.println (" jis "+j);
}
int j=0;
for(int k=0; j + k !=10;j++,k++) { 
	system.out.println (" jis "+j + "k is"+ k);
}

~~~

