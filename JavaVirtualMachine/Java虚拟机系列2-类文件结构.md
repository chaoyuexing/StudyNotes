# Java虚拟机系列2-类文件结构
我们现在提到的虚拟机都把它和Sun HotSpot VM等同.

![Java技术体系](http://xingshaonian.oss-cn-hangzhou.aliyuncs.com/img/JavaRepertoire.jpg)

## 特性
- 平台无关性: Java虚拟机就是一个虚拟的计算机,有自己完善的硬件体系,堆栈,寄存器,对应的指令集,所有的都是有自己一套虚拟的规则
- 语言无关性: 语言无关性是指 JVM虚拟机执行的.class二进制文件,不管是 kotlin, Groovy,Scala等可以运行在JVM虚拟机上的语言,如果足够牛逼 ,写一个.c文件编译成.class文件的 编译器,c语言也是可以运行在JVM虚拟机上的

- JIT 编译 ,just in time的缩写. java是一门解释型语言,首先编译成与平台无关的Java字节码文件,通过JIT技术,jvm发现某个方法或代码块运行频繁的时候,认为是热点代码,然后JIT会把它编译成本地机器相关的机器码,并进行优化,然后缓存,以备下次使用. JIT技术也是实现热重载的重要原因之一  

## .class文件

.class是虚拟机实现了语言无关性的重要基础之一,虚拟机并不关心class的来源是什么语言,只关心class文件.

### class文件存储结构
class文件是一组以8位字节为基础单位的二进制流文件,class文件采用一种类似于C语言结构体的伪结构体来存储数据

-  1:无符号数: 基本的数据类型,u1、u2、u4、u8 来分别代表 1 个字节、2 个字节、4 个字节和 8 个字节的无符号数，无符号数可以用来描述数字、索引引用、数量值或者字符串（UTF-8 编码）
-  2:表:表是由多个无符号数或者其他表作为数据项构成的复合数据类型，class文件中所有的表都以“_info”结尾。其实，整个 Class 文件本质上就是一张表。

### 魔数和版本号

- 每个class文件头4个字节称为魔数,U4类型  4个字节,在Class文件中只有一个，判断当前文件是否是class文件

 ```
 	CA FA BA BE
 ```
 上面就是魔数,充分体现了程序员的浪漫情节
 
- 版本号 : U2  2个字节，在Class文件中只有一个  同时还有一个副版本号 2个字节。 紧跟在魔数后面,副版本号在前,主版本号在后
 

### 常量池

常量池紧跟在版本号后面.首先用2个字节来表示常量池的大小,可以理解为class文件之中的资源仓库. 最重要

1:常量池的大小。FF FF  65535 - 1 , -1是因为Java的常量池是由1开始的,设计者将0空出来是有特殊考虑的,是为了满足某些指向常量池,但是不引用任何一个常量池项目的含义.

2:存放数据类型:

- 文本字符串,声明为final的常量值
- 类和接口的全限定名
- 字段的名称和描述符
- 方法的名称和描述符

#### 常量池中的常量项结构总表

| 常量                                          |  项目                      | 类型                               | 描述           |
| -------------------------- | --------------- | -------------------- | --------- |
| CONSTANT_Utf8-Info 	          | tag <br/>	length <br/>   bytes | u1 <br/> u2 <br/> u1  | 值为0x01  ,UTF-8编码字符串表  <br/>    UTF-8编码的字符串占用的字节数    <br/>长度为length的UTF-8编码的字符串   |
| CONSTANT_Integer-Info          | tag   <br/>  bytes | u1 <br/> u4 | 值为0x03 ,整形常量表   <br/> 按照高位在前存储的int值 |
| CONSTANT_folat-Info             | tag  <br/>  bytes  | u1 <br/> u4  | 值为0x04  ,浮点型常量表   <br/>    按照高位在前存储的float值  |
| CONSTANT_Long-Info            | tag  <br/> bytes   | u1 <br/> u8 | 值为0x05   ,长整形常量表   <br/>   按照高位在前存储的long值  |
| CONSTANT_Double-Info         | tag <br/>    bytes | u1 <br/> u8 | 值为0x06    ,双精度浮点型常量表   <br/> 按照高位在前存储的double值    |
| CONSTANT_Class-Info           | tag <br/> index     | u1 <br/> u2 | 值为0x07      ,类/接口引用表 <br/> 指向全限定名常量项的索引    |
| CONSTANT_String-Info          | tag  <br/> index    | u1 <br/> u2 | 值为0x08     ,字符串类型字面量  <br/> 指向全限定名常量项的索引   |
| CONSTANT_Fieldref-Info         | tag   <br/>  index  <br/> index  | u1 <br/> u2 <br/> u2 | 值为0x09      ,字段引用表   <br/> 指向声明字段的类或者接口描述符的索引   <br/> 指向声明字段描述符CONSTANT_Name-AndType-info的索引     |
| CONSTANT_Methodref-Info     |     tag   <br/> index  <br/> index | u1 <br/> u2 <br/> u2 | 值为0x10      ,类的方法引用表  <br/> 指向声明方法的类描述符的索引  <br/> 指向声明名称及类型的描述符CONSTANT_Name-AndType-info的索引  |
| CONSTANT_Interface-Methodref-Info     |     tag  <br/> index <br/> index   | u1  <br/> u2 <br/> u2 | 值为0x11     ,接口的方法引用表  <br/> 指向声明方法的接口描述符的索引   <br/>  指向声明名称及类型的描述符CONSTANT_Name-AndType-info的索引   |
| CONSTANT_Name-AndType-Info     |     tag   <br/> index  <br/> index | u1 <br/> u2 <br/> u2 | 值为0x12    ,字段或方法的名称和类型表   <br/> 指向该字段或方法名称常量项的索引  <br/> 指向改字段或方法描述符常量的索引  |
| CONSTANT_Methodref-Handle-Info     |     tag  <br/> index <br/> index    | u1  <br/> u2  <br/> u2 | 值为0x15,方法句柄表  <br/> 值必须在1~9之间(包括1和9),它决定了方法句柄的类型,方法句柄类型的值表示方法句柄的字节码行为 <br/>  值必须是对常量池的有效索引   |
| CONSTANT_Methodref-Type-Info     |     tag  <br/> index     | u1 <br/> u2 | 值为0x16   ,方法类型表 <br/> 值必须是对常量池的有效索引 ,常量池在索引处的项必须是Utf-8_info结构,表示方法的描述符   |CONSTANT_Invoke-Dynamic-Info   |     tag  <br/> bootstrp_method-attr-index <br/>  name_and-type+index   | u1 <br/> u2 <br/ > u2 | 值为0x18  ,动态方法调用表 <br/ > 值必须是对当前class文件中引导方法表bootstarp_methods[]数组的有效索引 <br/>  值必须是对常量池的有效索引,索引处结构必须是NameAndType-info结构  |


常量池之所以说最繁琐,就是因为14中常量类型都有自己的结构,给一个例子.

### 访问标志

在常量池结束之后,紧跟着的两个字节代表访问标志,例如 是class 还是接口,是否为public 类型,abstract修饰,类是否final.
例如 0021标识为一个public的类

| 标志名称                                    |  标志值                  | 含义                               | 
| -------------------------- | --------------- | -------------------- | 
| ACC_PUBLIC      	| 0x0001   | 是否为public类型         |
| ACC_FINAL         	| 0x0010   | 是否为final,只能修饰类  |
| ACC_SUPER       	| 0x0020   | 是否允许使用invokespecial字节码指令的新语意, invokespecial指令的语意在JDK1.0.2发生改变,JDK1.0.2之后编译出来的类这个标志都为真|
| ACC_INTERFACE	| 0x0200  | 标识为一个接口   |
| ACC_ABSTRACT     | 0x0400  | 是否为抽象类,接口类型为假 |
| ACC_SYNTHETIC    | 0x1000   | 标识这个类并非由用户代码产生的 |
| ACC_ANNOTATION | 0x2000 | 标识这是一个注解 |
| ACC_ENUM		| 0x4000 | 标识这是一个枚举 |

### 类索引,父类索引与接口索引集合

类索引,父类索引和接口索引集合都按顺序排列在访问标志之后,

- 类索引: u2  2个字节  例如00 05 指向常量池的第五个常量 
- 父类索引:  类索引之后紧跟的就是父类索引， 例如00 06指向常量池的第六个常量.
- 接口索引集合: 紧跟着就是接口索引计数器 例如 00 02 就是代表实现了2个接口,紧跟在后面的 00 04 ,00 05代表实现的接口在常量池中的位置.

### 字段表集合

字段表用于描述接口或者类中声明的变量. 包含了`类级变量`以及`实例级变量`,不包含方法内声明的局部变量

- 字段表表结构

| 标志名称                                    |  标志值                  | 含义                               |      类型      |
| -------------------------- | --------------- | -------------------- |  --------|
| access_flags <br/> 字段的访问标志 |  ACC_PUBLIC  (00 01)<br/> ACC-PRIVATE   (00 02)<br/>  ACC-PROTECTED     (00 04)<br/> ACC-STATIC   (00 08) <br/> ACC-FINAL    (00 10)  <br/>ACC-VOLATILE   (00 40) <br/> ACC-TRANSIENT   (00 80) <br/> ACC-SYNTHETIC   (10 00) <br/> ACC-ENUM   (40 00) | 字段是否public <br/> 字段是否private <br/> 字段是否protected <br/> 字段是否static <br/> 字段是否final <br/> 字段是否volatile <br/> 字段是否transient <br/> 字段是否由编译器自动产生 <br/> 字段是否enmu | u2 |
| name_index <br/> 字段的名称索引 |                     |  对常量池的引用    | u2 |
| descriptor_index <br/> 字段的描述索引 | B <br/> C <br/> D <br/>  F <BR/ > I <BR/> J <BR/> S <BR/> Z <BR/> V<BR/> L   |  基本类型byte <br/>  基本类型char <br/> 基本类型double  <br/> 基本类型float <br/> 基本类型int <br/> 基本类型long <br/> 基本类型short <br/> 基本类型boolean <br/> 特殊类型void <br/> 对象类型,如Object | u2 |
| attributes_count |   | 属性计数器,例如private int i 没有额外的属性,属性计数器为0,如果使用 final 修饰,就会存在constantVlaue属性 | u2 |
| attributes |  | 如果attributes_count为0,数量为attributes-count  |  attribute_info |

### 方法表集合

方法表和字段表的存储几乎采用了一致的方式.

| 名称                           |  类型               | 数量           |
| ------------------| ------------ |  -------- |
| access_flags 	 | u2 | 1 |
| name_index  	 | u2 | 1 |
| descriptor_index  | u2 | 1 |
| attributes_count  | u2 | 1 |
| attributes  | attribute_info | attributes_count |

### 属性表集合

属性表在上面的文档中出现了很多次,在class文件,字段表,方法表都可以携带自己的属性表集合

虚拟机也定义了一些属性

| 属性名称                  |  标志值                  | 含义                                |      类型    |
| ---------------- | --------------- | -------------------- |  --------|
| code | 方法表 | Java代码编译成的字节码指令 |
| ConstantValut | 字段表 | final关键字定义的常量 |
| Deprecated | 类 , 方法表 ,字段表 | 被声明为deprecated的方法和字段 |
| Exceptions  | 方法表 | 方法抛出的异常 |
| EnclosingMethod | 类文件 | 仅当一个类为局部类或者匿名类时才拥有这个属性,这个属性用来表示这个类所在外围方法|

上面的这个属性表并不全,对于每个属性,它的名称需要从常量池引用一个utf8_info类型的常量表示,属性值的结构都有自己的一个表结构.

- code属性表

| 属性名称                  |  数量                |     类型    |
| ---------------- | ------------ |  --------|
| attribute_name-index |  1   |  u2 |
| attribute_length |  1   |  u4 |
| max_stack |  1   |  u2 |
| max_locals |  1   |  u2 |
| code_length |  1   |  u4 |
| code | code_length | u1 |
| exception_table-length | 1 | u2 |
| exception_table | exception_info | exception_table-length |
| attributes_count | 1 | u2 |
| attributes | attributes_info | attributes_count |