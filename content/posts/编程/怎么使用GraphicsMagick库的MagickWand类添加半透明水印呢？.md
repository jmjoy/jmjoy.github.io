---
title: "怎么使用GraphicsMagick库的MagickWand类添加半透明水印呢？"
date: 2020-04-05T20:25:01+08:00
description: ""
featured_image: "/static/images/怎么使用GraphicsMagick库的MagickWand类添加半透明水印呢/GraphicsMagick-Logo.png"
categories: ["编程"]
tags: ["GraphicsMagick", "MagickWand", "C语言"]
---

## 前言

最近做图片处理功能，参考了各种图片处理库的对比资料，决定使用`GraphicsMagick`这个库，并且为了方便使用`MagickWand`这个struct。

基本上一些常用的图片处理功能都没啥问题，却被`半透明图片水印`这个功能卡住了。这个功能很常用也很重要啊，但是无论是中文还是英文资料，都找不到关于用`MagickWand`这个struct怎么去处理的线索，硬生生地让我纠结了几天的时间，终于在今天下午的某个时刻，抱着试试的心态，给解决了。

## 解决方案

下面就说说解决方案吧：

1. 首先，通过搜索引擎，得知命令行处理`半透明图片水印`的办法：

    ```bash
    /usr/bin/gm composite -geometry +500+450 -dissolve 30 watermark.png input.jpg output.jpg
    ```

1. 既然命令行可以，那就说明这个功能是可以做的，通过查看这个命令处理的源代码，找到了关键的处理代码：

   http://hg.code.sf.net/p/graphicsmagick/code/file/560875f67e82/magick/command.c#l2971

   ```c
   static MagickPassFail CompositeImageList(ImageInfo *image_info,Image **image,
   Image *composite_image,Image *mask_image,CompositeOptions *option_info,
   ExceptionInfo *exception)
   {
   // ...
             if (option_info->compose == DissolveCompositeOp)
             {
                 register PixelPacket
                 *q;

                 /*
                 Create mattes for dissolve.
                 */
                 if (!composite_image->matte)
                   SetImageOpacity(composite_image,OpaqueOpacity);
                 for (y=0; y < (long) composite_image->rows; y++)
                 {
                   q=GetImagePixels(composite_image,0,y,composite_image->columns,1);
                   if (q == (PixelPacket *) NULL)
                       break;
                   for (x=0; x < (long) composite_image->columns; x++)
                   {
                       q->opacity=(Quantum)
                       ((((unsigned long) MaxRGB-q->opacity)*option_info->dissolve)/100.0);
                       q++;
                   }
                   if (!SyncImagePixels(composite_image))
                       break;
                 }
             }
   // ...
              status&=CompositeImage(*image,option_info->compose,
                composite_image,geometry.x,geometry.y);
   }
   ```

   思路是先将水印图片，按设置的透明度参数，先设置成半透明，注意`SetImageOpacity`这个步骤是不能少的。然后再使用`CompositeImage`这个方法和`DissolveCompositeOp`这个方式，将水印图片覆盖到原来图片之上。

1. 但是这些方法应用的struct是`Image`而不是`MagickWand`，`MagickWand`是`Image`的一个用户使用友好的封装，`MagickWand`也提供了`MagickCompositeImage`这个方法，和`CompositeImage`是类似的。但是设置透明度这里却不知道怎么搞，遂放弃通过`MagickWand`设置透明度。

1. 查找`MagickWand`的文档找不到获取`Image`的方法，但是通过查看`MagickWand`的定义发现了`Image`字段。

   http://hg.code.sf.net/p/graphicsmagick/code/file/560875f67e82/wand/magick_wand.c#l108

   ```c
   /*
     Typedef declarations.
   */
   struct _MagickWand
   {
     char
       id[MaxTextExtent];
   
     ExceptionInfo
       exception;
   
     ImageInfo
       *image_info;
   
     QuantizeInfo
       *quantize_info;
   
     Image
       *image,             /* Current working image */
       *images;            /* Whole image list */
   
     unsigned int
       iterator;
   
     unsigned long
       signature;
   };
   ```

   可惜定义是在.c文件而不是.h文件，或者是作者不想暴露里面的字段给用户，但是为了解决需求，也只能破坏一下封装了。


1. 将上述的`struct _MagickWand`定义给copy到项目的.h文件里面，先通过`watermark->image`获取到水印图片`MagickWand`的`image`字段，按照设置水印透明度的代码将水印图片处理好，然后就可以用`MagickWand`的`MagickCompositeImage`方法去将水印图片覆盖到原图片了。

   完整代码参考以下：

   ```c
   MagickWand *input;
   MagickWand *watermark;

   //...

   Image *composite_image = watermark->image;

   register PixelPacket *q;
   
   /*
   Create mattes for dissolve.
   */
   if (!composite_image->matte)
     SetImageOpacity(composite_image,OpaqueOpacity);
   for (y=0; y < (long) composite_image->rows; y++)
   {
     q=GetImagePixels(composite_image,0,y,composite_image->columns,1);
     if (q == (PixelPacket *) NULL)
         break;
     for (x=0; x < (long) composite_image->columns; x++)
     {
         q->opacity=(Quantum)
         ((((unsigned long) MaxRGB-q->opacity)*option_info->dissolve)/100.0);
         q++;
     }
     if (!SyncImagePixels(composite_image))
         break;
   }

   MagickCompositeImage(input, watermark, DissolveCompositeOp, 500, 450);

   ```

## 后话

事实上，我是通过Rust的ffi调用`GraphicsMagick`的C函数来处理这个功能的，目前没有找到完善的`GraphicsMagick` binding库，所以只能自己来搞了，后面整理好了再开源出来。
