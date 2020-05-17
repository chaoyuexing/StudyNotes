# 像使用Activity一样使用Navigation

## Navigation是什么

Navigation(导航)是Android Jetpack框架中管理Fragmen的一个框架.

## 怎么使用

在`res`目录下创建`Navigation`文件夹,如下图所示

![创建目录](http://xingshaonian.oss-cn-hangzhou.aliyuncs.com/img/navigation_create.jpg)

然后在代码目录创建类继承Fragment以后就可以在`Navigation`目录下的xml文件添加你想要的Fragment了,然后在`activity`的布局文件中添加fragment

```
    <fragment
        android:id="@+id/nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:defaultNavHost="true"     
        app:layout_constraintBottom_toTopOf="@id/nav_view"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:navGraph="@navigation/mobile_navigation"   --指定使用的Navigation.xml文件/>
```
接下来看下Navigation.xml文件

![创建目录](http://xingshaonian.oss-cn-hangzhou.aliyuncs.com/img/navigation_example.jpg)

添加的fragment会自动的生成一个ID

```
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/mobile_navigation"
    app:startDestination="@+id/navigation_home" >  ---指定启动的默认的展示的fragmentID

    <fragment
        android:id="@+id/navigation_home" ---自动生成的ID
        android:name="com.example.navigation.ui.home.HomeFragment"
        android:label="@string/title_home"
        tools:layout="@layout/fragment_home" />

    <fragment
        android:id="@+id/navigation_dashboard"
        android:name="com.example.navigation.ui.dashboard.DashboardFragment"
        android:label="@string/title_dashboard"
        tools:layout="@layout/fragment_dashboard" />

    <fragment
        android:id="@+id/navigation_notifications"
        android:name="com.example.navigation.ui.notifications.NotificationsFragment"
        android:label="@string/title_notifications"
        tools:layout="@layout/fragment_notifications" />
</navigation>
```

以上就是入门怎么使用,建议直接在AndroidStudio中创建模板代码查看熟悉,以上代码出自于模板代码,在关键处加了文字描述.

![创建目录](http://xingshaonian.oss-cn-hangzhou.aliyuncs.com/img/navigation_template_create.jpg)


## StartActivity一样跳转

#### 1 直接指定要跳转的`FragmentID`,ID是在假如Navigation.xml文件中生成的ID

```
root.findViewById<Button>(R.id.start_one_btn).setOnClickListener {
    findNavController().navigate(R.id.homoOneFragment)
}
```
#### 2 使用action进行跳转

```
root.findViewById<Button>(R.id.start_one_btn).setOnClickListener {
    findNavController().navigate(R.id.action_navigation_home_to_homoOneFragment)
}
```

action 从哪里来,在Navigation.xml文件中将两个Fragment拉上线,如下图,系统会自动生成代码
![创建action](http://xingshaonian.oss-cn-hangzhou.aliyuncs.com/img/navigation_fragment_link.jpg)

```
<fragment
    android:id="@+id/navigation_home"
    android:name="com.example.navigation.ui.home.HomeFragment"
    android:label="@string/title_home"
    tools:layout="@layout/fragment_home" >
    <action
        android:id="@+id/action_navigation_home_to_homoOneFragment"
        app:destination="@id/homoOneFragment" /> 目标地点
</fragment>
```

#### 3 携带数据跳转

```
root.findViewById<Button>(R.id.start_one_btn).setOnClickListener {
    val bundle = Bundle()
    bundle.putString("parm","This is Home One Fragment")
    findNavController().navigate(R.id.action_navigation_home_to_homoOneFragment,bundle)
}
```

#### 4:使用Safe Args传递参数

```
root.findViewById<Button>(R.id.start_one_btn).setOnClickListener {
    val direction = HomeFragmentDirections.actionNavigationHomeToHomoOneFragment("2")
    findNavController().navigate(direction)
}
```
看到这里大家可能有点懵,这是啥玩意,下面贴出代码

在目的地Fragment定义argument,还可以顺便给出默认值

```
<fragment
    android:id="@+id/homoOneFragment"
    android:name="com.example.navigation.ui.home.HomoOneFragment"
    android:label="HomoOneFragment" >
    <argument
        android:name="parm"
        app:argType="string"
        android:defaultValue="1"/>
</fragment>
```
在最顶层的`build.gradle`添加`classpath "androidx.navigation:navigation-safe-args-gradle-plugin:2.2.0"` 

在app的`build.gradle`添加 `apply plugin: 'androidx.navigation.safeargs.kotlin'`

Java使用`apply plugin: "androidx.navigation.safeargs"`

添加过后系统会在`⁨app⁩ ▸ ⁨build⁩ ▸ ⁨generated⁩ ▸ ⁨source⁩ ▸ ⁨navigation-args⁩ ▸ ⁨debug⁩ `中生成模板代码,然后你就可以使用上面的代码进行传递参数了,本质没什么区别,只是系统为我们生成了大量的模板代码.

`但是可以保证空安全,提升代码下限,建议使用这种方式`

## startActivityForResult 获取目的地页面的返回参数

例如A跳转到B,B需要回传一些参数,在B的back回调事件中添加

```
root.findViewById<Button>(R.id.back).setOnClickListener {
    val bundle = Bundle()
    val aa = "122323"
    bundle.putString("parm", aa)
    findNavController().setGraph(R.navigation.mobile_navigation, bundle)
}
```
在A中

```
Log.d("HomeFragment","arguments[\"parm\"]"+ (arguments?.get("parm") ?: "null"))
```

这个部分的资料非常少,后面再仔细研究研究,补充一下.

## 跳转嵌套

A跳转到B,B到C,C到A 是我们在日常工作中经常遇到的场景,如下图
![创建action](http://xingshaonian.oss-cn-hangzhou.aliyuncs.com/img/navigation_nest_start.jpg)

```
<fragment
    android:id="@+id/homeTwoFragment"
    android:name="com.example.navigation.ui.home.HomeTwoFragment"
    android:label="HomeTwoFragment" >
    <action
        android:id="@+id/action_homeTwoFragment_to_navigation_home"
        app:popUpTo="@id/navigation_home" /> 将 homeTwo跳转到home的action destination 修改为 popUpto
</fragment>
```
Kotlin代码

```
root.findViewById<Button>(R.id.back_home).setOnClickListener {
    findNavController().navigate(R.id.action_homeTwoFragment_to_navigation_home)
}
```
这样就可以跳转回直接返回首页并将homeOne也移出栈了

或者直接指定返回栈ID,推荐上面方式,更清晰,不容易出错,提升代码下限

```
root.findViewById<Button>(R.id.back_home).setOnClickListener {
        findNavController().popBackStack(R.id.navigation_home, false)
}
```

