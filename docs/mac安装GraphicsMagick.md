#GraphicsMagick使用
> 什么是 GraphicsMagick？ 图像处理的瑞士军刀。GraphicsMagick 是图像处理的瑞士军刀。由基本包中源代码的 267K 物理行（根据 David A. Wheeler 的 SLOCCount）（或 1,225K，包括第 3 方库）组成，它提供了强大而高效的工具和库集合，支持读取、编写和操作超过 88 种主要格式的图像，包括 DPX、GIF、JPEG、JPEG-2000、PNG、PDF、PNM 和 TIFF 等重要格式


###安装Homebrew
mac包管理器，可以帮助您安装 GraphicsMagick 和其他软件包，已经安装可以跳过这一步

	/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

 
###安装GraphicsMagick

 查找GraphicsMagick 版本
 
	 brew search graphicsmagick

```
mayundeMacBook-Pro ~ % brew search graphicsmagick
==> Formulae
graphicsmagick
```

安装GraphicsMagick，这可能需要几分钟时间
	
	brew install graphicsmagick

检查 GraphicsMagick 是否已成功安装
	
	gm version
```
mayundeMacBook-Pro ~ % gm version
GraphicsMagick 1.3.40 2023-01-14 Q16 http://www.GraphicsMagick.org/
Copyright (C) 2002-2023 GraphicsMagick Group.
Additional copyrights and licenses apply to this software.
See http://www.GraphicsMagick.org/www/Copyright.html for details.
```

## GraphicsMagick图片相关操作
[官方文档](http://www.graphicsmagick.org/GraphicsMagick.html)
####获取图片分配率、大小
后面的地址可以是本地地址也可以是互联网地址

	gm  identify -format  %wx%h,%b http://resources1.deepdraw.biz/img/default_videoHome.jpg

```
输出 750x1000,23.5Ki
```
	
####缩放图片
	gm convert  http://resources1.deepdraw.biz/img/default_color.jpg -resize 50%  /Users/hk/Downloads/kjkk2.jpg

####裁剪图片
基于百分比进行裁剪
	
	gm convert /Users/hk/Downloads/demo.jpg -crop 50%x50%+10%+10% /Users/hk/Downloads/demo.jpg	
基于像素坐标进行裁剪

	gm convert /Users/hk/Downloads/demo.jpg -crop 800x600+10+10 /Users/hk/Downloads/demo.jpg
裁剪并调整图片大小

	gm convert /Users/hk/Downloads/demo.jpg  -crop 800x600+10+10 -resize 400x300 /Users/hk/Downloads/demo.jpg 

####图片打上水印

设置水印图片的透明度为百分之50%

	gm composite -dissolve 50% -gravity center watermark.png input.jpg output.jpg
	
####调整图像的质量
将图片压缩为80%的质量，清晰度会降低

	gm convert -quality 80 input.jpg output.jpg
#### 图片转换
图片转换为PNG格式

	gm convert -format png input.jpg output.png
#### 图片旋转
将图片旋转90度

	gm convert -rotate 90 input.jpg output.jpg
####高斯模糊
	gm convert -blur 0x8 input.jpg output.jpg
#### 生成gif

将三张名为 1.png、2.png 和 3.png 的图片合成为一个动画 GIF，每张图片之间间隔 100 毫秒并输出为 demo.gif

	gm convert -delay 100 1.png 2.png 3.png -loop 0 demo.gif
	
`-delay `参数指定每张图片之间的间隔时间，`-loop 0 `参数指定无限循环播放 GIF 动画，demo.gif 是输出的 GIF 文件。

此外，还可以通过 `-dispose `参数指定 GIF 动画的帧间处理方式。常见的帧间处理方式有 `background`、`previous`、`none` 和 `restore`，我们可以通过指定这个参数来设置帧间处理方式。

例如：

	gm convert -delay 100 -dispose previous frame1.png frame2.png frame3.png -loop 0 animation.gif 
	
可以指定帧间处理方式为 `previous`，表示帧间不清除并将前一帧作为背景
##集成到SpringBoot

###引入相关sdk
maven

```
<dependency>
    <groupId>com.sharneng</groupId>
    <artifactId>gm4java</artifactId>
    <version>1.1.1</version>
</dependency>
```

### 构建GraphicsMagick进程池
配置相关进程池信息注入到Spirng容器中

```
    //从yml读取自定义的配置信息
    @Autowired
    private  GMPoolProperties gmPoolProperties;

    @Bean
    public PooledGMService PooledGMService() {
        GMConnectionPoolConfig config = new GMConnectionPoolConfig();
        config.setMaxActive(gmPoolProperties.getMaxActive());
        config.setMaxIdle(gmPoolProperties.getMaxIdle());
        config.setMinIdle(gmPoolProperties.getMinIdle());
        config.setMinEvictableIdleTimeMillis(gmPoolProperties.getMinEvictableIdleTimeMillis());
        config.setWhenExhaustedAction(gmPoolProperties.getWhenExhaustedAction());
        config.setMaxWait(gmPoolProperties.getMaxWait());
        config.setTestWhileIdle(gmPoolProperties.isTestWhileIdle());
        config.setTimeBetweenEvictionRunsMillis(gmPoolProperties.getTimeBetweenEvictionRunsMillis());
        log.info("init gm pool finished");

        return new PooledGMService(config);
    }
```
### 运行相关图片操作
获取图片信息

```
    @Autowired
    PooledGMService pooledGMService;

    @Test
    void test() throws Exception {
        String imageInfo = pooledGMService.execute("identify -format %wx%h,%b", "/image/default.jpg");
        System.out.println(imageInfo);
    }
```
