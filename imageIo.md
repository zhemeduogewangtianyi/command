记录一次ImageIO合成图片的坑
背景图 + 二维码 + 字体合成为一张新的图片，用ImageIO类踩到了一个大坑，这类需求一定要注意以下几点：
1：图片纬度：
1.1：图片的大小一定要压缩，加入你手中拿到了一张1MB这么大的图片，就应该警醒了！！谁给你图片找谁给你压缩。。。不然byte[]的大小远超10万字节！效率绝对低到你怀疑人生。
1.2：图片的像素，一定要够用就行。像素点少了，效率自然也会高。
2：技术栈维度：
1：ImageIO.write(byte[] bytes) 写图片
1.1：ImageIO在写图片的时候，会用临时文件记录你的字要放在图片的哪个像素点，当前写到了哪个像素点了，这样在用户访问量大的情况下会分分钟把磁盘写满，让开发人员再次怀疑人生（这是为啥呢。。）。
1.2：接下来说说ImageIO.useCache(false)，这个东西说是把临时文件放内存，能够快速解决你磁盘写满的问题，他说的没错，确实能解决你临时文件写满磁盘的问题，但是呢，他把临时文件给你放内存里面去了！没错！内存里面。还不如磁盘写满。。因为内存满了机器就挂了！！！
2：Font对象：
1：Font字体使用的时候一定要谨慎，要多去问问产品用什么样的字体（微软雅黑爱告状），这时候产品给你说了一个字体，本地用起来一点问题没有，然后放到了服务器上就会遇到另外一个坑，服务器上没有字体包。这个需要在服务器上下载字体包，不是什么大问题需要注意
2：Font对象一定要做成一个静态的或者单例，通过这个构造方法过多次创建Font的话会出现 Can't create output stream!
 public static Font createFont(int fontFormat, InputStream fontStream)
        throws java.awt.FontFormatException, java.io.IOException {...}
最简单的解决方案：
private static Font setFont(int size){
    ...
    return createFont(Font.PLAIN , inputStream)
}
private static Font staticFont = setFont(34);
3：如何避免上述的坑？
3.1：多思考，多问问，多调研，多压测。
3.2：JAVA 图像处理库 Thumbnails
<dependency>
  <groupId>net.coobird</groupId>
  <artifactId>thumbnailator</artifactId>
  <version>0.4.8</version>
</dependency>
加载图片源：
File file = new File("/Users/qiangzi/data/img","beauty.jpg");
Builder<File> builder = Thumbnails.of(file);
缩放:
builder = builder.scale(0.9); 
质量压缩:
builder.outputQuality(0.9); //参数是浮点数，0-1之间
剪裁:
builder.sourceRegion(100, 100, 300, 300);    
builder.sourceRegion(Positions.CENTER, 200, 200);
根据宽度来缩放:
builder.width(500);
根据高度来缩放:
builder.height(500);
在调整尺寸时保持比例:
builder.keepAspectRatio(true);  //默认为true，如果要剪裁到特定的比例，设为false即可
根据宽度和高度进行缩放:
builder.size(600, 700);
1. //如果设置了keepAspectRatio(true)，将按比例进行缩放，否则将强制按尺寸输出
2. 缩放策略：
3. 如果宽度缩放比>高度缩放比就以宽度来缩放，反之以高度缩放
将图片放入内存:
1. File file2 = new File("/Users/qiangzi/data/img","logo.png");
2. BufferedImage bufferedImage = Thumbnails.of(file2).scale(1.0).outputQuality(1.0).asBufferedImage();
3. *必须要指定scale
加水印:
1. builder.watermark(Positions.BOTTOM_RIGHT, bufferedImage, 1.0f);
2. //第一个参数是加水印的位置
3. //第二个参数是要加水印的图片
4. //第三个参数是水印图片的透明度
5. 经过测试：gif图片的彩色会变成黑白，所以尽量使用jpg或png图片吧
输出图片，不管对图片进行什么操作，只有输出才能看到效果
1. builder.toFile(File file);
*注意：scale、width|height、size三者不能同时共存，但必须要有一个
//这里baos为了看上去方便这么写，一定要在finally里面关闭流
ByteArrayOutputStream baos = new ByteArrayOutputStream(1024);
Thumbnails.of(CacheUtil.NEW_INSTANCE.getImgs(Constants.BACK_PICTURE_KEY))
    .size(750, 1334)
    .outputFormat(Constants.JPG)
    .watermark((enclosingWidth, enclosingHeight, width, height, insetLeft, insetRight, insetTop, insetBottom) -> {
        int x1 = (enclosingWidth / 2) - (width / 2);
        int y1 = 970;
        return new Point(x1, y1);
    }, rqCode, 1.0f)
    .watermark((enclosingWidth, enclosingHeight, width, height, insetLeft, insetRight, insetTop, insetBottom) -> {
        int x1 = (enclosingWidth / 2) - (width / 2) + 51;
        int y1 = 875;
        return new Point(x1, y1);
    }, image, 1.0f).toOutputStream(baos);
附录位置参数
1. Positions.TOP_LEFT
1. Positions.TOP_CENTER
1. Positions.TOP_RIGHT
1. Positions.CENTER_LEFT
1. Positions.CENTER
1. Positions.CENTER_RIGHT
1. Positions.BOTTOM_LEFT
1. Positions.BOTTOM_CENTER
1. Positions.BOTTOM_RIGHT
3.3：OSS也带有相关的图片处理功能
3.4：这类需求尽量不要放在服务端，服务端在生成图片时卡一下，就说明服务不稳定，客户端页面卡一下，用户会感觉该换手机了。（打个比方）
