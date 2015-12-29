title: Android UI学习笔记
tags:
  - Android
id: 145
categories:
  - 技术分享
date: 2011-07-21 18:57:24
---

**1\. UI****设计**

自学Android也有一段时间了，自我感觉良好。于是乎反编译了一些UI界面不错的APK学习一下。为了做对比，反编译了一个中文应用一个英文应用，不知是师出同门，还是业界标准，这两个APK的UI资源文件格式惊人的相似，现将res结构贴出来：

[![](/images/2011/07/Android-Res1.png "Android Res2")](/images/2011/07/Android-Res1.png)

<!--more-->这与我们在书本、网络、视频教程看到不一样的是，它将类型一致的常量分离出来，如colors、dimens。同时，它也将样式（styles）给分离出来，主窗体中只留下了界面组件的结构，如同写html+css一样……

将colors、dimens、styles给分离出来的好处显而易见，它能使UI设计的结构更为清晰，组织更为方便。现在给一个例子，让我们更为理性的了解一下这种模式的好处……
> &lt;!-- layout/main.xml 定义一个TextView --&gt;> 
> 
> &lt;TextView style=_"@style/StyleTest"_ /&gt;> 
> 
> &lt;!-- values/styles.xml 定义一个_StyleTest_ --&gt;> 
> 
> &lt;resources&gt;> 
> 
> &lt;style name=_"StyleTest"_&gt;> 
> 
> &lt;item name=_"android:id"_&gt;@+id/text&lt;/item&gt;> 
> 
> &lt;item name=_"android:textSize"_&gt;@dimen/text_medium&lt;/item&gt;> 
> 
> &lt;item name=_"android:textStyle"_&gt;bold&lt;/item&gt;> 
> 
> &lt;item name=_"android:textColor"_&gt;@color/title_text&lt;/item&gt;> 
> 
> &lt;item name=_"android:ellipsize"_&gt;end&lt;/item&gt;> 
> 
> &lt;item name=_"android:gravity"_&gt;center_vertical&lt;/item&gt;> 
> 
> &lt;item name=_"android:layout_width"_&gt;@dimen/text_width&lt;/item&gt;> 
> 
> &lt;item name=_"android:layout_height"_&gt;fill_parent&lt;/item&gt;> 
> 
> &lt;item name=_"android:singleLine"_&gt;true&lt;/item&gt;> 
> 
> &lt;/style&gt;> 
> 
> &lt;/resources&gt;> 
> 
> &lt;!-- values/dimens.xml 定义一个text_medium text_width --&gt;> 
> 
> &lt;resources&gt;> 
> 
> &lt;dimen name=_"text_medium"_&gt;14.0dip&lt;/dimen&gt;> 
> 
> &lt;dimen name=_"text_width"_&gt;40.0dip&lt;/dimen&gt;> 
> 
> &lt;/resources&gt;> 
> 
> &lt;!-- values/colors.xml 定义一个text_color --&gt;> 
> 
> &lt;resources&gt;> 
> 
> &lt;color name=_"text_color"_&gt;#000&lt;/color&gt;> 
> 
> &lt;/resources&gt;
将相同类型的常量分离出来后，方便开发者统一组织与管理，如统一将字号变大，统一将字体颜色修改等……将样式分离出来的好处，相信做过web开发的朋友应该能很好的理解，开发者将应用UI整体结构打好之后，剩下的工作就是一个一个给其赋属性（style）了，同时还可以方便的替换多种样式。

需要强调的一点就是，android的UI设计也支持include，这个也是今天才知道的，格式如下：

&lt;include layout=_"@layout/title_bar" _/&gt;

**2\. ****监听器模式**

在学习的时候，大部分教程写UI事件监听器都是一个这样的方式，先findViewByID(ID)；再setOnXXXListener(new OnXXXListener({ methods}))。个人不是很喜欢这样的方式，繁琐而且代码不清晰，于是乎便建立了很多很多的XXXListener.java类，管理起来更累。周爱名前辈写的《大道至简》中说到：“是懒人造就了方法。”于是乎我就在组件属性中发现了一个很熟悉的名词：OnClick，让我很是开心。这个和javaScript类似，开发者只需要在相应的Activity中加上相应的方法即可。如：
> &lt;!-- layout/XXX.xml 定义一个Button --&gt;> 
> 
> &lt;Button …> 
> 
> android:onClick=_"onButtonClicked"_> 
> 
> …/&gt;> 
> 
> // Button点击事件> 
> 
> public void onButtonClicked(View view){> 
> 
> …> 
> 
> }
是不是简单得多？方便得多？清晰得多？

**3\. ****其它问题**

**3.1 Tab****修改样式**

不管哪种语言，Tab组件一定是最麻烦的！比如我在xml中定义它的时候，需要一个TabHost，一个TabWidget，一个FrameLayout，这还不算什么，最令人觉得怪异的是，它都没有定义字体的属性，无奈只能通过代码修改，但代码的修改方式同样很诡异……
> **public** **void** setTabStyle(TabHost **tabHost**) {> 
> 
> // 还得向上转型，将TabWidget转成View才能修改！！！> 
> 
> ** for**(**int** **i** = 0; **i**&lt; **tabHost**.getTabWidget().getChildCount();**i**++){> 
> 
> View **view** = **tabHost**.getTabWidget().getChildTabViewAt(**i**);> 
> 
> ** view**.setXXX();> 
> 
> // 标题栏> 
> 
> TextView **tv** = (TextView) **view**.findViewById(android.R.id._title_);> 
> 
> ** tv**.setXXX();> 
> 
> }> 
> 
> }
不知是自己功力不够，还是本来就是这么麻烦，现在只能认了……不知你有没有好的方法？

**3.2 DatePickerDialog**

我用DatePickerDialog时，返回的月份（monthOfYear）是从0开始算起，查了一下SDK，它的解释是：monthOfYear The month that was set (0-11) for compatibility with Calendar. 以后用得注意一下……
