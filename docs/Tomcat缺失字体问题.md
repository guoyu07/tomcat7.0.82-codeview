# Tomcat缺失字体问题

本篇主要介绍解决`线上环境Tomcat缺失字体的问题`，如果知道操作系统中字体作用以及工作原理的同学，看到这个问题可能会觉得这个问题是不成立的，因为`Tomcat`不存在字体与不是字体的问题，它只是一个容器而已，存在字体问题都是因为里面的代码使用了字体库，但是环境中没有才会导致，所以和是不是`Tomcat`没有关系，`控制台`程序也会有问题，这是`JRE`的问题。字体问题和编码问题不是同一类问题，出现乱码时该有的位置都是奇怪的特殊字符，缺少字体时会存在一个一个方块无法显示。




----------------


## 一、问题现象描述

最近在工作过程中接到前方同学的提问，在`Windows`环境中写的一个生成图片水印的代码运行的好好的，但是放在`Linux 的Docker容器中`发现中文位置全都是方块,很明显是缺少字体的问题。现象如下图所示

![](images/缺少字体1.png)

问题代码如下
```java
import javax.imageio.ImageIO;
import java.awt.*;
import java.awt.font.*;
import java.awt.geom.Rectangle2D;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import java.io.FileInputStream;
public class AppsData {
  //args[0]  水印的文字内容
  //args[1]  水印图片的文件存储位置
  //args[2]  自定义字体位置
  public static void main(String[] args) throws IOException,FontFormatException {
    //Font font=Font.createFont(Font.TRUETYPE_FONT,new File(args[2]));
    Font font=new Font("宋体",1,24);
    GraphicsEnvironment ge =GraphicsEnvironment.getLocalGraphicsEnvironment();
    System.out.print(ge.registerFont(font));
    int with=300;
    int hight=300;
    BufferedImage image=new BufferedImage(with,hight,BufferedImage.TYPE_INT_ARGB);
    Graphics2D g2d=image.createGraphics();
    image=g2d.getDeviceConfiguration().createCompatibleImage(with,hight,3);
    g2d.dispose();
    g2d=image.createGraphics();
    g2d.setColor(new Color(10));
    g2d.setStroke(new BasicStroke(1.0F));
    FontRenderContext context=g2d.getFontRenderContext();
    Rectangle2D bounds=font.getStringBounds(args[0],context);
    double x=(with-bounds.getWidth())/2.0;
    double y=(hight-bounds.getHeight())/2.0;
    double ascent=-bounds.getCenterY();
    double baseY=y+ascent;
    g2d.rotate(Math.toRadians(-45.0D),with/2,hight/2);
    g2d.drawString(args[0],(int)x,(int)baseY);
    g2d.dispose();
    ImageIO.write(image,"png",new File(args[1]));
  }
}
```
代码主要部分就是调用了`JDK`的图形库，使用`Graphics2D`类来画一个图像，水印使用`宋体`字体。针对前方人员反馈的问题来看，最简单的解决方法就是在当前环境中添加一个字符集即可，对于`Centos`来说直接扔到`/usr/share/fonts/chinese`这个路径下即可(如果目录没有直接新建)，完了重启计算机，对于`Ububtu或者Arch`来说可能就在其他地方了，如果是使用的`Docker`的环境，对于`alpine`镜像可能又是另外一个位置，我们总不可能把所有的系统都熟悉一遍，或者要求所有的环境都添加一个字体库吧。如果能够我们的程序中自带这个字体就好了，这样随便你是什么系统什么环境我都使用我自己的字体，这才是最好的解决方案。所以摆在我们面前的就是两个方法解决这个问题：

- 在一个环境中的字体读取位置添加字体，例如在`操作系统的字体库`或者在`JRE`的字体库中
- 我们程序中自己自带字体库，使用我们指定位置的字体文件，例如读取一个目录下的字体，这样我们还可以使用我们自己优化过的字体更加的美观

















