# java_基础01常见坑
常见坑整理

## 对象池
==比较的是引用，不是内容

让人更混乱的是，有时候用==是能比较内容的。如果你有两个一样的不可变值，JVM会尝试引用 同一个对象 。

String s1 = "Hi", s2 = "Hi";

Integer a = 12, b = 12;

这两个例子中用到了对象池，所以最后引用 的是同样的对象。s1==s2和a==b都是返回true,JVM已经把两个引用 都指向了同一个对象。然而，如果稍微改下代码，JVM没有把对象放到池里的话，==就会返回false，可能会让你意想不到。这个时候你得用equals 了。

```
String s3 = new String(s1);  
Integer c = -222, d = -222;  
s1 == s2      // is true  
s1 == s3      // is false  
s1.equals(s3) // is true  
a == b        // is true  
c == d        // is false (different objects were created)  
c.equals(d)   // is true  
```
对于整型来说，对象池缓存的范围是-128到127（还有可能更高）。


## Integer类有缓存
这个功能点也是面试的高频热点之一，稍不注意，也有可能被带入沟里，我们看看下面这段代码：

```
public static void main(String[] args){
   Integer a = 100;
   Integer b = 100;
   Integer c = 200;
   Integer d = 200;
   System.out.println(a==b);
   System.out.println(c==d);
}
```
上面的代码竟然输出：

```
true
false
```

## Object.toString那些不为人知的事情
Object.toString那些不为人知的事情

toString的默认行为是打印类的的内部名 称还有对象的hashCode。 上面已经提到，hashCode并不是内存地址，尽管它是用16进制打印的。同样的，类名，尤其是数组的类名，更容易让人头晕。比如说String[]的 名称是[Ljava.lang.String; 这个[表明它是个数组，L说明它是Java语言（Language）创建的类，并不是基础类型比如byte这些，顺便提一下byte内部名称是B。;号标 识类的结束。比如你有个这样的数组：

String[] words = { “Hello”, “World” };

System.out.println(words);

输出会是这样：

[Ljava.lang.String;@45ee12a7

很不幸你只知道这是个对象数组。如果你只有一个Object实例words，这样是不够的，你得调用下Arrays.toString(words)。这极其恶劣地破坏了封装的原则，在StackOverflow上面这也是最常见的一类问题。

我问过Oracle的好些个工程师，从他们的反馈感觉，这个问题目前很难解决。

## 逻辑运算符的“短路”现象
使用逻辑运算符时，我们会遇到“短路”的现象：一旦能够确定整个表达式的值，就不会计算余下的部分了，当然，这个功能点其实是非常有用的，但对于初学者来说，可能会感觉比较惊讶，使用不当就会产生“坑爹”后果。比如下面的代码：

```
int num = 1;  
System.out.println(false && ((num++)==1));  
System.out.println(num);  
```
就会输出false和1，因为逻辑与&&的前半部分为false，不管后半部分为true还是false，整个表达式都会返回false，所以就不会再计算后面的部分了，如果把false改成true，那么后半部分就会得到执行，num也就变成2了。


##  Java注释能够识别Unicode
先看看代码：
```
public static void main(String[] args){
    // \u000d System.out.println("Hello World!");
}
```
乍一看，代码都被注释掉了，当然不会输出任何东西，然而，它还是输出每个程序员都倍感亲切的Hello World，这是因为，unicode解码发生在代码编译之前，编译器将\u样式的代码进行文本转义，即使是注释也是这样，然后\u000a被转换成\n换行符，所以println代码得以正常执行。

这样的功能着实“坑爹”，极其违反常识，它必须要上榜，必须要荣登状元的位置。


## 参考
Java中常见的坑：http://doc.okbase.net/flycars001/archive/86504.html

Java 语言中十大“陷阱”你遇到过几个？：https://blog.csdn.net/weixin_42784331/article/details/104303549

