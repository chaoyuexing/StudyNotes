# Android IPC 机制

android 中如何指定多进程,在androidManifest文件注册4大组件时添加process属性,指定对应进程名称即可,开启多进程非常简单,但是多进程之间的通信问题是一件值得学习和研究的事情.

## 序列化
为什么需要序列化,当对象需要在内存中传递,就需要序列化,这里有个疑问,为什么我们用Intent传递基本数据类型时不需要手动序列化,因为在这些基本数据类型的类中,已经帮我们实现了Serializable接口,不需要我们手动序列化.

### Seralizable

Seralizable实现起来非常简单,只需要实现Seralizable接口即可, Seralizable序列化是通过I/O流的方式去序列化,即将对象转化为16进制的字符串,适合做持久化存储,存储到存储设备上.

### Parcelable

Parcelable接口的实现类是可以通过Parcel写入和恢复数据的,并且必须要有一个非空的静态变量 CREATOR,而且还给了一个例子,这样我们写起来就比较简单了,但是简单的使用并不是我们的最终目的.


## Bindle
