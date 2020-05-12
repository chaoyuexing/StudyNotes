# Android内存优化

- 1: 内存不足,会触发频繁GC,触发频繁GC,GC会引发Stop World.导致卡顿

- 2:内存不足,系统开始杀死大量应用,来满足前台进程,甚至直接GC.导致应用崩溃.


### GC (Garbage Collection)

是不是所有的GC都会导致Stop World,Android系统在5.0以后使用ART虚拟机,ART虚拟机对GC机制进行了大量的优化,首先我们看下Android的堆内存分区

![Android内存分区] (http://xingshaonian.oss-cn-hangzhou.aliyuncs.com/img/AndroidHeapMemory.jpg)

##### 1 Young Generation

由一个Eden区和两个Survivor区组成，程序中生成的大部分新的对象都在Eden区中，当Eden区满时，还存活的对象将被复制到其中一个Survivor区，当此Survivor区满时，此区存活的对象又被复制到另一个Survivor区，当这个Survivor区也满时，会将其中存活的对象复制到年老代。



### 确定手机内存大小

```
        ActivityManager manager = (ActivityManager) getSystemService(Context.ACTIVITY_SERVICE);
        ActivityManager.MemoryInfo info = new ActivityManager.MemoryInfo();
        manager.getMemoryInfo(info);
        if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.JELLY_BEAN) {
            long totalMem = info.totalMem;
            Log.d("MainActivity", "totalMem::" + (totalMem);
        }
```

以上代码获取到手机内核大致可以能够使用的最大内存,

https://blog.csdn.net/luoshengyang/article/details/42555483
https://mp.weixin.qq.com/s?__biz=MzI1MTA1MzM2Nw==&mid=400021278&idx=1&sn=0e971807eb0e9dcc1a81853189a092f3&scene=0&key=b410d3164f5f798eafd870697d352ac86e0e54b9605b5fcd2c6a62268c16080ee291069627f13ed906cc2f39706b6a54&ascene=0&uin=NzY0MTg2ODU%3D&devicetype=iMac+MacBookPro11%2C1+OSX+OSX+10.10.5+build(14F27)&version=11000003&pass_ticket=nhSGhYD4LC9FWvUPv26Y7AdIzqEDu8FTImf2AKlyrCk%3D
https://blog.csdn.net/qq_20798591/article/details/105010818