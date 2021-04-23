## String

### 中文编码

#### char型变量中能不能存储一个中文汉字

char型变量是用来存储Unicode编码的字符的，unicode编码字符集中包含了汉字，所以，char型变量中可以存储汉字啦。不过，如果某个特殊的汉字没有被包含在unicode编码字符集中，那么，这个char型变量中就不能存储这个特殊汉字。

> 补充说明：
>
> 一个字符的 Unicode 编码是确定的。但是在实际传输过程中，由于不同系统平台的设计不一定一致，以及出于节省空间的目的，对 Unicode 编码的实现方式有所不同。Unicode 的实现方式称为Unicode转换格式（Unicode Translation Format，简称为 UTF）。
>
> Unicode编码占用两个字节，char类型的变量也是占用两个字节。

 

> utf-8 中汉字占三个字节

 

```java
public void test(){
    char cha = '我';
    System.out.println(cha);  //我
    //通过Integer中的toBinaryString方法转char为二进制
    String binaryString = Integer.toBinaryString(cha);
    System.out.println(binaryString);  //1100010  00010001
}
```


