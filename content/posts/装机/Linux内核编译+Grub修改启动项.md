>Linux内核编译参考：[https://linux.cn/article-9665-1.html](https://linux.cn/article-9665-1.html)
>Grub修改启动项参考：[https://www.cnblogs.com/open-skill/p/8295234.html](https://www.cnblogs.com/open-skill/p/8295234.html)

本人的笔记本电脑是联想拯救者Y7000，最适合的Linux内核是长期支持版：4.19.x。
![image.png](https://upload-images.jianshu.io/upload_images/18494435-4f46574e381c9eda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意i2c_hid的触摸版必须这个内核以上的才支持，5.x的内核貌似有问题。目前使用这个版本的内核较为完善，建议手动编译，Ubuntu帮忙构建的的通用内核.deb包貌似缺了某些驱动（nvidia、虚拟机等）。
这个笔记本其实对Linux不太友好，包括触摸板、wifi、Nvidia显卡等，需要好好调教才能正常使用。

## 内核编译注意事项：

### 基本顺序
1. `make xconfig`，务必设置版本的后缀（`general setup -> Local version - append to kernel release`这一项）。
1. `make`，将会编译内核和模块，具体内容参考`make help`的说明。
1. `make modules_install`，将安装内核模块。
1. `make install`，将安装内核，过程很长，需要耐心等候。如无意外将执行安装`vmlinuz`、`System.map`、`initrd.img`和更新`grub`等。

添加版本后缀后其实不要担心会覆盖原来的内核，Linux本身就支持多内核共存。如果害怕可以设置`make modules_install`和`make install`的安装目录，具体参考`make help`。
