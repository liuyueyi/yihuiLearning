---
title: 211121-Java实现图片转字符输出示例demo
tags:
  - JDK
categories:
  - Java
  - JDK
date: 2021-11-21 21:58:03
keywords:
  - Java
  - JDK
  - BufferedImage
---

前面几篇博文介绍了使用jdk来对图片做一些有意思的转换，接下来我们再介绍一个有意思的玩法，直接根据图片，输出一个二维字符数组，实现用字符来实现绘画的场景

<!-- more -->

各位小伙伴可能都有看到过一些有趣的注释，比如大佛，美女之类的，通关本文，相信你也很可以很简单的实现类似的场景

关键实现，在前面的文章中其实也说到了，下面是超链

- [Java实现图片灰度化](https://blog.hhui.top/hexblog/2021/11/12/211112-Java%E5%AE%9E%E7%8E%B0%E5%9B%BE%E7%89%87%E7%81%B0%E5%BA%A6%E5%8C%96/)
- [Java实现图片转字符图片示例demo](https://blog.hhui.top/hexblog/2021/11/16/211116-Java%E5%AE%9E%E7%8E%B0%E5%9B%BE%E7%89%87%E8%BD%AC%E5%AD%97%E7%AC%A6%E5%9B%BE%E7%89%87%E7%A4%BA%E4%BE%8Bdemo/)
- [Java实现Gif图转字符动图](https://blog.hhui.top/hexblog/2021/11/20/211120-Java%E5%AE%9E%E7%8E%B0Gif%E5%9B%BE%E8%BD%AC%E5%AD%97%E7%AC%A6%E5%8A%A8%E5%9B%BE/)

接下来我们需要做的就是将之前转成字符图片输出的地方稍微改一下，根据当前色颜色，来选择合适的替换字符保存下来

所以关键的实现在于，如何根据颜色来选择字符

```java
// 这个字符来自于github搜索结果，下面将最后一个从原来的点号改成了空格，即白色时，不输出字符
private static final String DEFAULT_CHAR_SET = "$@B%8&WM#*oahkbdpqwmZO0QLCJUYXzcvunxrjft/\\|()1{}[]?-_+~<>i!lI;:,\\\"^`' ";

/**
 * 基于颜色的灰度值，获取对应的字符
 * @param g
 * @return
 */
public static char toChar(Color g) {
    double gray = 0.299 * g.getRed() + 0.578 * g.getGreen() + 0.114 * g.getBlue();
    return DEFAULT_CHAR_SET.charAt((int) (gray / 255 * DEFAULT_CHAR_SET.length()));
}
```

接下来我们针对之前的方法，稍微改造一下

```java
Color getAverage(BufferedImage image, int x, int y, int w, int h) {
    int red = 0;
    int green = 0;
    int blue = 0;

    int size = 0;
    for (int i = y; (i < h + y) && (i < image.getHeight()); i++) {
        for (int j = x; (j < w + x) && (j < image.getWidth()); j++) {
            int color = image.getRGB(j, i);
            red += ((color & 0xff0000) >> 16);
            green += ((color & 0xff00) >> 8);
            blue += (color & 0x0000ff);
            ++size;
        }
    }

    red = Math.round(red / (float) size);
    green = Math.round(green / (float) size);
    blue = Math.round(blue / (float) size);
    return new Color(red, green, blue);
}

private void parseChars(BufferedImage img) {
    int w = img.getWidth(), h = img.getHeight();
    // 这个size可用来控制精度，越小则越像原图
    int size = 4;
    List<List<String>> list = new ArrayList<>();
    for (int y = 0; y < h; y += size) {
        List<String> line = new ArrayList<>();
        for (int x = 0; x < w; x += size) {
            Color avgColor = getAverage(img, x, y, size, size);
            line.add(String.valueOf(toChar(avgColor)));
        }
        list.add(line);
    }

    System.out.println("---------------------- 开始 ------------------------");
    for (List<String> line: list) {
        for (String s: line) {
            System.out.print(s + " ");
        }
        System.out.println();
    }
    System.out.println("---------------------- 结束 ------------------------");
}
```

注意上面的实现，需要重点注意的是原图的遍历方式，一层一层的遍历，即外部是y轴，内部循环是x轴

接下来看一下测试case

```java
@Test
public void testChars() throws Exception{
    String file = "http://pic.dphydh.com/pic/newspic/2017-12-13/505831-1.png";
    BufferedImage img = ImageLoadUtil.getImageByPath(file);
    // 缩放一下图片为300x300，方便对输出字符截图
    img = GraphicUtil.scaleImg(300,300, img);
    parseChars(img);
    System.out.println("---over------");
}
```


实际输出如下(实际输出结果与皮神还是很像的)

![](/imgs/211121/00.jpg)

```text
---------------------- 开始 ------------------------
                                                                                          l m                                                         
                                                                                        ' b $ I                                                       
                                                                                        f $ $ [                                                       
                                                                                      \ 8 $ $ f                                                       
  i ~ ,                                                                               x $ $ $ u                                                       
  _ $ $ a X } ^                                                                     ' W $ $ $ c                                                       
    c $ $ $ $ B L ] '                                                               } q $ $ $ z                                                       
    ` d $ $ $ $ $ 0 r ( "                                                           t < U $ $ c                                                       
      , * $ $ $ $ z < + j | `                                                     \ t < < O $ n                                                       
        l W $ $ $ U < < < ~ t [                                                   { + < < _ W f                                                       
          > & $ $ 0 < < < < < - j ~                                               \ < < < < n (                                                       
            ! # $ k < < < < < < < ( t `                                           j < < < < ] ?                                                       
              : k B 1 + < < < < < < + n !                                       > } < < < < \ i                                                       
                ^ C z ( [ ~ < < < < < < f ]                                     1 < < < < < u `                                                       
                    1 v ( ) ? < < < < < < | {                                   ( < < < < < u                                                         
                      I v / ( 1 + < < < < < 1 }   ' l > i "                     \ < < < < ~ x                                                         
                          1 v ( ( [ ~ < < < < z r z t | \ n z f ( + '           t < < < < ? /                                                         
                            " / v | ) ? < < < < < < < < < < < < < ] f ) ^     " \ < < < < | ?                                                         
                                " ) n v / ~ < < < < < < < < < < < < < ~ j [   l 1 < < < + z ^                                                         
                                      - | < < < < < < < < < < < < < < < < ] j ~ { < < < [ u                                                           
                                    ' f < < < < < < < < < < < < < < < < < < ~ z | < < ~ ( /                                                           
                                    ] + < < < < < < < < < < < < < < < < < < < < n - < ? n i                                                           
                                  ' | < < < < < < < < < < < < < < < < < < < < < < { ~ ) v                                                             
                                  ) + < ] 0 w f < < < < < < < < < < < < < < < < < < ? / 1                                                             
                                  x < ~ * @ " | [ < < < < < < < < < < < < < < < < < } c '                                                             
                                i ( < } $ $ x w \ < < < < < < < < < < < < < < < < < / -                                                               
                                / < < + % $ $ 8 _ < < < < < < < < < < < < < < < < < { >                                                               
                              _ q f < < ( q m } < < < < < < < < < < < < { \ ~ < < < } <                                                               
                            " O U Z < < < < < < < < < < n f < < < < < n r [ h + < < \ "                                                               
                            j U U 0 } < < < < _ < < < < ~ ~ < < < < < M u ( $ r < < t                                                                 
                            U U U Q ( < < < < { Y v Y 0 Z } < < < < < * $ $ $ x < < |                                                                 
                            J U U O [ < < < < < # * # # o a t < < < < j $ $ # _ < < )                                                                 
                            Y U U O < < < < < < W b q q k # # O r \ < < [ / + < < } <                                                                 
                            x U L r < < < < < < d U c c C w * o L < < < < < < < < f '                                                                 
                            } Q n < < < < < < < J x x x x z w W [ < < < < < < < < f         : + [ { ,                                                 
                            ^ f < < < < < < < < L x x x x x J Y < < < < < < _ Y O Y   ] \ j \ [ _ } -                                                 
                              / < < < < < < < < L x x x x x C _ < < < < < - Z U U Z j ? < < < < < \ v >                                               
                              l | < < < < < < < Y x x x x X | < < < < < < J U U U z < < < < < < < + ) (                                               
                        i ) t / L | + < < < < < c x x x c x < < < < < < ) L U U C t < < < < < < < < f !                                               
                    ? x ( < < < < ? f x j ~ < < 1 J x Y x < < < < < < < u U U U 0 < < < < < < < < + r                                                 
                  1 { < < < < < < < < < _ x \ < < t v 1 < < < < < < < < n U U Z [ < < < < < < < v x _                                                 
                < ) < < < < < < < < < < < < { u ] < < < < < < < < < < < - 0 m { < < < < < < < } n | / n r ( / \ 1 } i '                               
                / < < < < < < < < < < < < < < ~ U < < < < < < < < < < < | c _ < < < < < < < - n < < < < < < < < < < { f / ( ] ^                       
                | < < < < < < < < < < < < < < < ) n ] _ ~ < < < < < / n 1 < < < < < < < ~ [ L + < < < < < < < < < < < < < < _ t / 1 l                 
                t < < < < < < < < < < < < < < < v j ( ( ) - < < ~ } + < < < < < < < < _ 1 Q j < < < < < < < < < < < < < < < < < < < ) f ( :           
                ) + < < < < < < < < < < < < < < c [ ] ? ~ < < < < < < < < < < < < ~ } ( c t [ < < < < < < < < < < < < < < < < < < < < < ~ / t <       
                ^ x { } ] ] - _ ~ < < < ~ _ ~ r c < < < < < < < < < < < < < < < - ) ( u < \ < < 1 - < < < < < < < < < < < < < < < < < < < < < \ )     
                  < c ( ( ( ( ( ( ( ) ) / Y n n < < < < < < < < < < < < < < ~ { ( ( v i   f < _ ( ( 1 _ < < < < < < < < < < < < < < < < < < + j '     
                    ! v n ( ( ( ( r X c f ~ < < < < < < < < < < < < < < < ~ 1 ( ( c ;     r < ] ( ( ( ( } + < < < < < < < < < < < < < < < + f '       
                        ] X x u u | + < < < < < < < < < < < < < < < < < < { ( t n ^     \ f < { ( ( ( ( ( ( [ ~ < < < < < < < < < < < < + / '         
                        t ~ < < < < < < < < < < < < < < < < < < < < < < - ( v {         ? ] < ) ( ( ( ( ( ( ( ) ? < < < < < < < < < < + \             
                        u < < < < < < < < < < < < < < < < < < < < < < < { w :           \ < + ( ( ( | ( ( ( ( ( ( { ~ < < < < < < < ~ (               
                      i \ < < < < < < < < < < < < < < < < < < < < < < < ~ n             j < - ( ( / x t c n ( ( ( ( ) - < < < < < ~ (                 
                      t < < < < < < < < < < < < < < < < < < < < < < < < < n   + ] :     x < [ ( ( v '     ! | n X \ ( ( [ < < < ~ /                   
                      u < < < < < < < < < < < < < < < < < < < < < < < < < x   ( 1 | r t ( < { ( x +             " { j z r { ~ ~ f '                   
                    ~ 1 < < < < < < < < < < < < < < < < < < < < < < < < < t   ] t 1 + < < < ( ( x                     ' ~ / n u ^                     
                    t < < < < < < < < < < < < < < < < < < < < < < < < < < \   i n ( ( ? < + ( v :                                                     
                  ' r < < < < < < < < < < < < < < < < < < < < < < < < < < /   " z ( ( ( } ] | \                                                       
                  _ [ < < < < < < < < < < < < < < < < < < < < < < < < < < t     L t \ Y u z z `                                                       
                  / < < < < < < < < < < < < < < < < < < < < < < < < < < < / ;   Z Q Q \   , I                                                         
                ' f < < < < < < < < < < < < < < < < < < < < < < < < < < < + # 0 d d d }                                                               
                ] ? < < < < < < < < < < < < < < < < < < < < < < < < < < < < Z d d d b ?                                                               
                \ < < < < < < < < < < < < < < < < < < < < < < < < < < < < < u o b d k ~                                                               
                j < < < < < < < < < < < < < < < < < < < < < < < < < < < < < ] | ? u d :                                                               
                j < < < < < < < < < < < < < < < < < < < < < < < < < < < < < < j                                                                       
                r < ~ < < < < < < < < < < < < < < < < < < < < < < < < < < < < /                                                                       
                / + ( ( { ? + < < < < < < < < < _ ? ] ] ] ] ] - + < < < < < < f                                                                       
                i f ( ( ( ( ( 1 { } [ [ [ } 1 ( ( ( ( ( ( ( ( ( ( ) } _ < < < /                                                                       
                  ] X ( ( ( ( ( ( ( ( ( ( ( ( ( ( ( ( ( ( ( ( ( ( ( ( ( } < < |                                                                       
              : ( f ) v u ( ( \ j u c c u x J d m C J | ( ( ( ( ( ( ( ( ( [ r \                                                                       
          ` \ J t ] ~ { ( ( n X r , '         ` < ( j c C z r ( ( ( ( ( | n z }                                                                       
          u X z r } / u c f \ 1                           l ] j z J Y Y n ) } } v _                                                                   
          + r ( ) ( - I                                         I n c \ + < < ] L Z u ^                                                               
                                                                    ' i ] } | ( - x O w                                                               
                                                                              ; 1 \ z J                                                               
---------------------- 结束 ------------------------
```


虽说上面这个是输出了字符图，从结果上看也比价像，但是需要注意的是，若图片的背景非白色，主角不是那么突出的场景，通过上面的方式输出的结果可能就不太友好了，解决办法当然就是识别背景，识别主体，针对主体元素进行转换（这个过程后面有机会再介绍）


接下来我们借助开源项目 [https://github.com/liuyueyi/quick-media](https://github.com/liuyueyi/quick-media) 来迅速的实现字符图输出

以一个冰雪女王的转换图来验证下效果

```java
String file = "http://5b0988e595225.cdn.sohucs.com/images/20200410/76499041d3b144b58d6ed83f307df8a3.jpeg";
BufferedImage res = ImgPixelWrapper.build().setSourceImg(file).setBlockSize(4).setPixelType(PixelStyleEnum.CHAR_BLACK).build().asBufferedImg();
```

![](/imgs/211121/01.jpg)

