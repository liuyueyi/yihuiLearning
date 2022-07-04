---
title: 211120-Java实现Gif图转字符动图
tags:
  - JDK
categories:
  - Java
  - JDK
date: 2021-11-20 21:14:32
keywords:
  - Java
  - JDK
  - BufferedImage
---

前面介绍了两篇基于jdk实现图片灰度处理、转字符图片的操作，接下来我们在将之前的能力扩展一下，支持将一个gif图灰度化或者转gif字符图

<!-- more -->

本文的实现主要在前面两篇文章的基础上来实现，推荐没有看过的小伙伴也可以瞅一眼

- [Java实现图片灰度化](https://blog.hhui.top/hexblog/2021/11/12/211112-Java%E5%AE%9E%E7%8E%B0%E5%9B%BE%E7%89%87%E7%81%B0%E5%BA%A6%E5%8C%96/)
- [Java实现图片转字符图片示例demo](http://blog.hhui.top/hexblog/2021/11/16/211116-Java%E5%AE%9E%E7%8E%B0%E5%9B%BE%E7%89%87%E8%BD%AC%E5%AD%97%E7%AC%A6%E5%9B%BE%E7%89%87%E7%A4%BA%E4%BE%8Bdemo/)

单张图的灰度化与转字符实现之后，gif图的实现就简单多了；gif图无非是多张图组合而成，将每一张图转换之后，再重新组装成gif图就完事了

这里我们使用的gif工具类来自于[https://github.com/liuyueyi/quick-media/tree/master/plugins/base-plugin/src/main/java/com/github/hui/quick/plugin/base/gif](https://github.com/liuyueyi/quick-media/tree/master/plugins/base-plugin/src/main/java/com/github/hui/quick/plugin/base/gif)

核心关键类为`GifEncode`与`GifDecode`；借助它来实现gif图的加载与保存

首先我们将上篇博文中的转字符图的方法抽取一下

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

private BufferedImage parseImg(BufferedImage img) {
    int w = img.getWidth(), h = img.getHeight();
    // 创建新的灰度图片画板
    BufferedImage out = new BufferedImage(w, h, img.getType());
    Graphics2D g2d = out.createGraphics();
    g2d.setColor(null);
    g2d.fillRect(0, 0, w, h);

    int size = 12;
    Font font = new Font("宋体", Font.BOLD, size);
    g2d.setFont(font);
    for (int x = 0; x < w; x += size) {
        for (int y = 0; y < h; y += size) {
            Color avgColor = getAverage(img, x, y, size, size);
            g2d.setColor(avgColor);
            g2d.drawString("灰", x, y);
        }
    }
    g2d.dispose();
    return out;
}
```

接着就是Gif的操作了

```java
@Test
public void testRender() throws IOException {
    String file = "https://c-ssl.duitang.com/uploads/item/201707/11/20170711194634_nTiK5.thumb.1000_0.gif";
    // 从网络上下载图片
    GifDecoder decoder = new GifDecoder();
    decoder.read(FileReadUtil.getStreamByFileName(file));

	// 这里是核心的转换逻辑
    List<ImmutablePair<BufferedImage, Integer>> frames = new ArrayList<>();
    for (int i = 0; i < decoder.getFrameCount(); i++) {
        BufferedImage img = decoder.getFrame(i);
        frames.add(ImmutablePair.of(parseImg(img), decoder.getDelay(i)));
    }

	// 下面是保存gif图
    File save = new File("/tmp/out2.gif");
    ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
    GifHelper.saveGif(frames, outputStream);
    FileOutputStream out = new FileOutputStream(save);
    out.write(outputStream.toByteArray());
    out.flush();
    out.close();
    System.out.printf("渲染完成");
}
```

上图转换成功之后，输出如下

![](/imgs/211120/00.gif)

如果希望输出图片更像原图，可以修改上面的fontSize，比如上面用的是12，可以调整成8，6等值，根据实际情况进行选择

有的小伙伴可能会说了，动漫的gif图转换之后相似度还可以，那么真实人物图转换之后呢？

接下来我们借助开源项目 [https://github.com/liuyueyi/quick-media](https://github.com/liuyueyi/quick-media) 来迅速的实现一个gif图转换

> 下图来自网络，有兴趣的自己打开查看，就不贴上了😏）
> http://n.sinaimg.cn/sinacn/w390h219/20171231/0ac1-fyqefvw5238474.gif

```java
@Test
public void testGif() throws Exception {
    String img = "http://n.sinaimg.cn/sinacn/w390h219/20171231/0ac1-fyqefvw5238474.gif";
    ImgPixelWrapper.build().setSourceImg(img)
            .setBlockSize(7)
            .setPixelType(PixelStyleEnum.CHAR_COLOR)
            // 生成的gif图放大为原来的两倍
            .setRate(2d)
            // 支持设置字体
            .setFontStyle(Font.BOLD)
            // 这里设置生成字符图中的字符集
            .setChars("灰")
            .build()
            .asFile(prefix + "/out3.gif");
    System.out.println("--------");
}
```

![](/imgs/211120/01.gif)

最后提个小问题，gif图都能生成字符图了，那么视频也可以生成字符视频么？