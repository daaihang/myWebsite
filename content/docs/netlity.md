+++
title = '使用 Netlity 加速 Github Page'
date = 2024-03-16T18:22:49+08:00
+++

使用Github Page构建静态网页可以大大降低网页部署成本。虽然Github设有CDN加速节点，但在大陆仍然加载较慢。可以使用Netlity加速访问，提升Github Page的国内使用体验。

<img src="https://cdn.jsdelivr.net/gh/daaihang/PicGo/202301310127745.png" alt="image-20230131012454959" style="zoom:33%;" />

加速后本网站实测结果（用了很多网站测试，测速结果基本一致）：

<img src="https://cdn.jsdelivr.net/gh/daaihang/PicGo/202301310130655.png" alt="" style="zoom: 33%;" />

<img src="https://cdn.jsdelivr.net/gh/daaihang/PicGo/202301310157035.png" alt="" style="zoom: 33%;" />

**更多测速结果**

<img src="https://cdn.jsdelivr.net/gh/daaihang/PicGo/202301310204970.png" style="zoom:33%;" />

<img src="https://cdn.jsdelivr.net/gh/daaihang/PicGo/202301310206217.png" style="zoom:33%;" />

<img src="https://cdn.jsdelivr.net/gh/daaihang/PicGo/202301310207495.png" style="zoom:33%;" />

<img src="https://cdn.jsdelivr.net/gh/daaihang/PicGo/202301310212887.png" style="zoom:33%;" />



在构建过程中，本网站使用Cloudflare解析域名并加速，使用Netlity再次加速Github Page。因此本网站（旧版本，版本更新见[关于](/about)）是Cloudflare、Netlity和Github Page三方加速的结果。**但在我部署前后的加速仍然比较明显**。

## 绑定仓库

前面的基础信息自行选择，并起一个用户名。只需要填框内两个项目，其他选项可不填写。

<img src="https://cdn.jsdelivr.net/gh/daaihang/PicGo/202301310238656.png" alt="" style="zoom:50%;" />

选择`Import from Git`，以导入Github上的仓库。

<img src="https://cdn.jsdelivr.net/gh/daaihang/PicGo/202301310240296.png" alt="" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/daaihang/PicGo/202301310242126.png" style="zoom:50%;" />

登入Github账号后，选择需要构建的仓库`<username>.github.io`。

<img src="https://cdn.jsdelivr.net/gh/daaihang/PicGo/202301310318721.png" style="zoom:50%;" />

选择了需要的分支后点击`Deploy site`。

<img src="https://cdn.jsdelivr.net/gh/daaihang/PicGo/202301310248008.png" style="zoom:50%;" />

## 添加自定义域名

完成后在`Domains`选项卡中点击`Add custom domain`添加自定义域名。并在域名解析服务商填写解析。

<img src="https://cdn.jsdelivr.net/gh/daaihang/PicGo/202301310251539.png" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/daaihang/PicGo/202301310252712.png" style="zoom:50%;" />

添加后后添加两个域名，需要分别设置。点击`Check DNS configuration`选项，查看需要CNAME的域名。

<img src="https://cdn.jsdelivr.net/gh/daaihang/PicGo/202301310255259.png" style="zoom:50%;" />

复制框内的二级域名或IP地址，在域名解析商设置解析。

<img src="https://cdn.jsdelivr.net/gh/daaihang/PicGo/202301310302886.png" style="zoom:50%;" />

例如，我在Cloudflare解析域名，那我就在此处添加记录，解析上面两个域名。注意需要分开解析`example.com`和`www.example.com`域名，解析类型为CNAME。

<img src="https://cdn.jsdelivr.net/gh/daaihang/PicGo/202301310307898.png" style="zoom:50%;" />

## 设置HTTPS

完成解析后在下方选项启用HTTPS，安全访问网站。（此时截图还没验证完成，验证成功后可以启用。）

<img src="https://cdn.jsdelivr.net/gh/daaihang/PicGo/202301310314163.png" style="zoom:50%;" />

## 最后

### 推荐网站：[PageSpeed Insights](https://pagespeed.web.dev/)

这个网站为谷歌旗下网站分析的网站，可以分析网站的优缺点，提出优化建议。例如采用新一代格式提供图片、适当调整图片大小、移除阻塞渲染的资源。这些建议可以帮助提高网页加载速度。同时还给出无障碍设计建议、安全建议和SEO优化建议，谷歌还是很给力的😘。

<img src="https://cdn.jsdelivr.net/gh/daaihang/PicGo/202301310324195.png" style="zoom:50%;" />

在国内使用Github Page还是比较累人的。网络环境让我们没法好好使用这个功能。但花些时间捣鼓捣鼓就可以有比较好的体验。关键还不用花钱。虽然Page服务没法弄数据库，但对个人博客来说已经够用了。
