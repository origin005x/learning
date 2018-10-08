# Android 屏幕适配教程及适配机制介绍

## 概述

android屏幕适配有很多方面，例如，drawable，values等，开发者根据产品的实际情况和自己的经验，可以选择性的从其中几个方面入手。
本文主要介绍一种基于分辨率等比例缩放的适配方案。

# 几个必须知道的概念
## 屏幕尺寸
屏幕尺寸是指手机屏幕对角线的英寸数。
## 屏幕分辨率
屏幕分辨率是指屏幕宽高像素数。
## 屏幕密度
屏幕密度是指手机屏幕对角线上单位英寸内的像素数。
## 代码中用的尺寸单位（dp,dip,sp,px）
这些单位跟我们写代码才是最相关的。
- dp即dip，规定，密度为160的屏幕上，1像素对应的尺寸为1dp。320密度的屏幕上，1像素对应0.5dp，以此类推。在密度为160的屏幕上，1英寸有160个像素，那么1px对应的尺寸 = 1/160英寸。所以说dp是个物理尺寸，跟像素无关。所以，100dp的尺寸在不同手机上显示出来，物理尺寸看上去基本是一样的。
- SP 即 Scale-independent Pixel，即与缩放无关的抽象像素。sp和dp很类似但唯一的区别是，Android系统允许用户自定义文字尺寸大小（小、正常、大、超大等等），当文字尺寸是“正常”时，1sp=1dp=0.00625英寸，而当文字尺寸是“大”或“超大”时，1sp>1dp=0.00625英寸
- px，即像素，无需多说

## Android适配机制
1. drawable,在工程里可以在res目录下创建一系列的带后缀的drawable文件夹，例如drawable-hdpi，drawable-xhdpi等等。
 - drawable的适配机制是，系统会先到后缀与设备匹配的drawable目录下找对应的图片，当找不到的时候会去‘更高’一级的目录去找，再找不到，继续往高一级的找，再找不到就退而求其次去低一级的找，依次类推。
例如，在密度为xxhdpi的手机上运行app，会去drawable-xxhdpi目录下找图片资源，找不到就去drawable-xxxhdpi找，如果没有比drawable-xxxhdpi更高的，则再找不到就去drawable-xhdpi找，再找不到就去drawable-hdpi找，直到找到对应的图片资源，当找到后，系统会按密度对图片做缩放处理，然后再显示到屏幕上，所以如果图片放的目录不对的话，有可能造成图片模糊。
- layout目录，layout目录也是可以加后缀的，通常是带分辨率后缀（当然也可以加其他后缀，详见android官网，这里只讨论常用的后缀），例如， layout-land-1024x720，layout-1280x720，layout-1920x1080等等。
 - layout目录的适配机制是，从“高往低”找最接近的尺寸目录，例如手机是1920x1080分辨率的，但是如果无此layout目录那么便会低一级的layout-1280x720找布局（而不会去高一级的layout-2560x1440找），依次类推，直至找到layout不带后缀的目录为止，如果还没有，就会报错。
  - 所以考虑以下场景：
    - 原本我们的布局文件目录只有layout一个，没有其他带后缀的layout目录
    - 实际测试中发现的布局在960x540手机上有问题。
   那么有些人可能会想到加个layout-960x540目录，然后在此目录下做特殊处理。那么问题来了，加了这个目录之后，layout目录就有两个，layout无后缀和layout-960x540。当在1920x1080手机上运行程序时，按照适配机制，系统会使用layout-960x540目录下的布局文件，而我们当初的初衷是只希望layout-960x540目录下的布局文件在960x540的手机上使用，所以这种情况下布局肯定会有问题。
    ** 因此，千万注意上面这种场景，不要随意添加‘layout-分辨率’的这种目录，除非把各种主流分辨率都添加一遍。碰到这种问题，最好从dimens文件入手做适配，后面会讲到。**
- values目录之dimens文件，为了适配不同尺寸的手机，我们可以创建多个values目录，然后在其中定义dimens尺寸，例如values-1280x720，values-1196x720等等。
  - dimens适配的机制是，先找跟设备对应的values目录下的dimens文件中的尺寸定义，找不到则往低一级的找，比如，在1280x720分辨率的手机上，如果app中没有创建values-1280x720目录，而只有values-1920x1080、values-1196x720目录和默认的values目录，那么系统会去优先去values-1196x720的目录下找对应的尺寸。如果找不到，则去默认的values找，再找不到就报错（不会去1920x1080目录找）。
总结：
  1.drawable适配过程：找与设备密度对应的目录下的图片-》往更高质量的找-》退而求其次找低质量的
  2.layout适配过程：找与设备对应的目录，找不到则从比设备分辨率低一级的目录开始依次往下找。
  3.values适配过程：同layout。

#基于分辨率等比例缩放的适配方案示例
先看两张图：

![800x480分辨率.png](http://upload-images.jianshu.io/upload_images/2659843-c45118f35c9fcff6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![1920X1200分辨率.png](http://upload-images.jianshu.io/upload_images/2659843-23c2638d9730850a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面两张图中的登录按钮的尺寸是100dp，分别在800x480的设备上和1920x1200的设备上运行效果。可以看出来，第一张图按钮的尺寸跟整体布局还比较协调，但是第二张图中的按钮宽度显得特别的小，与整体布局很不协调。（注意，两个图中的按钮，在实际设备上的物理尺寸是差不多的，但是为了看起来方面，我将两张图都等比例压缩了，但是，这样不影响我们看按钮与整体尺寸比例关系）。

在实际界面开发当中，一般流程是这样的：
1.UI设计师基于854x480手机设计了一套图，并给出了标注。
2.开发人员将标注转换为dp写到布局文件中。
3.在854x480分辨率的手机上测试界面效果，不出意外应该是跟设计图基本一致的。
但是，如果将程序运行到1920x1080分辨率，但是密度为240（hdpi）的设备上，那就会出现类似上面第二张图中的效果（当然实际开发中会把上图中的按钮的宽度设置为match_parent，这里只是为了说明问题）。
界面布局简单了还好，我们可以通过调整，使用相对布局以及match_parent等手段将界面调整的比较好看。但是如果界面元素比较多了，有时候我们必须给某个控件设置一个固定尺寸，所以肯定避免不了这种问题的出现。

所以UI设计师可能会提出如下几个要求：
1.设计师基于720P（1280x720）设计一套图，并标注
2.研发开发出来的实际效果在720P手机上必须跟标注一致（要求高的公司甚至精确到像素）
3.如果在480P、1080p手机上，则实际尺寸按标注等比例缩放。比如在720p上的标注是100px，那么显示到1080p手机上必须是100*1080/720 = 150px。
4.对于非标准分辨率来说，比如1184x720(手机底部带虚拟按键的那种），则需要在高度上做调整，给控件的纵向间隙做些调整。

所以针对如上要求，结合android屏幕适配的机制，可以采用如下方案来适配：
1. 创建不同分辨率的values文件夹，在其中分别创建dimens.xml：

![不同目录的dimens文件.png](http://upload-images.jianshu.io/upload_images/2659843-9f73e1a04e9ab85e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 由于我们的标注是基于1280x720分辨率标注的，所以在values-1280x720目录下的dimens.xml里定义诸如下面这些尺寸：

![values-1280x720.png](http://upload-images.jianshu.io/upload_images/2659843-f1227239b855611b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到，我对名字的命名有一些规则，即，宽度值用‘w_像素数值’的格式命名，高度值用‘h_像素数值’的格式命名。
3. 根据标注编写布局文件，假设标注中‘发送验证码’按钮的宽度为180px
![代码片段.png](http://upload-images.jianshu.io/upload_images/2659843-c2ecf68cc4dcf408.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在720分辨率手机上对应的效果：
![720p.png](http://upload-images.jianshu.io/upload_images/2659843-77e8686cf042f9d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
4.根据values-1280x720目录下的dimens文件内容，转换为其他分辨率下的尺寸，然后放到对应分辨率的values目录下。当然这种转换手动转会累死。我写了一个转换工具（一个java类），转换起来很方便。

好了。来看下效果，**重点关注‘发送验证码’按钮**，为了方便看这个按钮占据整个布局的比例，我把几张图都缩放为差不多大小：
1.1280x720分辨率：

![720p.png](http://upload-images.jianshu.io/upload_images/2659843-7653c73a037c9c77.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2.1920x1080分辨率：

![1080new.png](http://upload-images.jianshu.io/upload_images/2659843-ed138d21aa674396.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


3.2560x1440：

![2k_new.png](http://upload-images.jianshu.io/upload_images/2659843-50cf9cfc3dfd3d43.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
上面的方式就是按分辨率等比例缩放的适配方式，我们称其为第一种适配方式，另外还有第二种适配方式，就是我们常用的把宽高用dp来定义的方式。
如果把‘发送验证码’按钮的宽度使用dp来定义，按照180px的标注，那转换为dp按钮宽度应设置为90dp，那么效果如下：
1.720分辨率，密度320dpi，跟上面用px的效果一样。
2.2560x1440分辨率，密度560dpi（由于用了dp，所以密度要关注下）：

![2k_usedp.png](http://upload-images.jianshu.io/upload_images/2659843-fd9a7301b23b5a72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看出按钮的宽度占整个屏幕宽度的比变小了。分析下原因，按第一种方式，即宽度按像素设定的方式，app运行时，@dimen/w_180px在2560x1440分辨率的手机上会去找values-2560x1440目录下的dimens文件，而此文件中的尺寸定义已经事先被我们按等比例缩放好了，即缩放后的像素= 180 * 1440/720 = 360px  
若按第二种方式，即宽度按dp设定的方式，app运行时，90dp在560dpi密度的手机上，实际显示的像素=90*560/160 = 315px。
那么按钮宽度占屏幕宽度的比例315：1440小于360：1440所以，所以看起来占用比例小了。

总结下，如果要适配各种分辨率的手机，并且想让控件在屏幕中所占用的比例不变，那么就需要用第一种方式来适配。这种方式下，无论屏幕的密度是多少，控件显示出来后在屏幕中占用的比例都不会变。   
也就是说，如果按这种方式做的话，由于主流分辨率的宽高比基本一致，因此在任何主流分辨率手机上运行后截图，然后再缩放为720宽度，那么界面中的元素位置及大小一定是跟标注中一致的。

以上是对屏幕适配的一些理解，及实际测试过后的一些结论，希望对大家有帮助。
另外，尺寸转换的工具见附件，用java swing组件做了个界面（比较low，凑合用吧）：
[尺寸转换工具](http://pan.baidu.com/s/1hrD1NEk)

使用说明：
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2659843-3649eada0b38e5c7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![尺寸转换工具使用.gif](http://upload-images.jianshu.io/upload_images/2659843-e065cf4867d57403.gif?imageMogr2/auto-orient/strip)
