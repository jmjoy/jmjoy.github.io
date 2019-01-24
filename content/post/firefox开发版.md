+++
date = "2017-01-10T00:35:10+08:00"
draft = false
title = "firefox开发版"
categories = ["其它"]
tags = ["firefox"]
+++

今晚和同事闲聊的时候才知道有`Firefox develop editon`这回事！Firefox 也是隐藏地很深啊，一直都觉得firefox的手机调试非常烂，但是开发版的手机模拟挺不错了，和Chrome的不相上下，这下子可以彻底抛弃Chrome了。（以前我开发总是要开两个浏览器，Firefox和Chrome，Chrome主要是用来调试web app的，我投靠Firefox的主要原因只是因为Chrome不知持外部下载器而且它自身的下载太烂了。）

https://www.mozilla.org/en-US/firefox/developer/?v=a

https://www.mozilla.org/en-US/firefox/channel/desktop/?v=a

这里就是下载地方，至于启动图标嘛

http://askubuntu.com/questions/548003/how-do-i-install-the-firefox-developer-edition

这里就有个很棒的回答，我是通过手动添加`.desktop`文件的，方便快捷。

> Manually
> 
> Download from Mozilla Firefox Developer Edition webpage. Extract it with file-roller and move the folder to its final location. A good practice is to install it in /opt/ or /usr/local/.
> 
> Once you moved the files to their final location (say /opt/firefox_dev/), you can create the following file ~/.local/share/applications/firefox_dev.desktop to get a launcher with an icon distinct from normal Firefox.
 
 ```
 [Desktop Entry]
 Name=Firefox Developer 
 GenericName=Firefox Developer Edition
 Exec=/opt/firefox_dev/firefox
 Terminal=false
 Icon=/opt/firefox_dev/browser/icons/mozicon128.png
 Type=Application
 Categories=Application;Network;X-Developer;
 Comment=Firefox Developer Edition Web Browser.
 ```

-------------------------------------------------------------------------------
(2017-01-21修改) 好气啊，答案给的`.desktop`不能打开外部url，原因是`Exec`执行的`firefox`少了`%u`这个参数，今天故意对比了一下原来firefox的`.desktop`才发现的，所以下面贴出了我对比之后修改的`.desktop`。

```
[Desktop Entry]
Encoding=UTF-8
Name=Firefox Developer 
GenericName=Firefox Developer Edition
Comment=Firefox Developer Edition Web Browser.
Exec=/opt/firefox_dev/firefox %u
X-MultipleArgs=false
Terminal=false
Icon=/opt/firefox_dev/browser/icons/mozicon128.png
Type=Application
Categories=Network;WebBrowser;
MimeType=text/html;text/xml;application/xhtml+xml;application/xml;application/vnd.mozilla.xul+xml;application/rss+xml;application/rdf+xml;image/gif;image/jpeg;image/png;x-scheme-handler/http;x-scheme-handler/https;
StartupWMClass=Firefox
StartupNotify=true
```
-------------------------------------------------------------------------------

***

最后，期待2017年的servo!

http://www.oschina.net/news/80710/mozilla-plan-replace-gecko-with-servo
