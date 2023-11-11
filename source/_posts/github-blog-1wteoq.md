---
title: github博客搭建
date: '2023-11-11 11:04:41'
updated: '2023-11-12 00:04:13'
permalink: /post/github-blog-1wteoq.html
comments: true
toc: true
---

# github博客搭建

参考：https://www.52pojie.cn/thread-1825164-1-1.html

# 新建仓库

只需要注意仓库名为<span style="font-weight: bold;" data-type="strong">你的ID+.github.io</span>即可，比如ID为<span style="font-weight: bold;" data-type="strong">truth</span>，那设置仓库名就是<span style="font-weight: bold;" data-type="strong">truth.github.io</span>

‍

# 设置仓库

进入仓库->setting，在左侧找到<span style="font-weight: bold;" data-type="strong">Pages</span>选项卡

​![image](assets/image-20231111122046-hgk3tnf.png)​

设置好Brach后刷新页面就可以看到这个：

​![image](assets/image-20231111122600-7kit2k3.png)​

至此，我们可以在github域名上访问仓库了，下面就是建立和美化博客的工作了

# hexo

[hexo]([https://hexo.io/zh-cn/](https://hexo.io/zh-cn/))是一款博客框架, 可以通过简单的命令部署出漂亮的博客系统。

前置需要npm和nodejs, 其中nodejs需要>=14.0, 以下操作是在本地ubuntu 22.04系统中进行：

```sh
# 安装hexo
npm install hexo -g


# 初始换blog目录
mkdir /blog && cd /blog
hexo init

# 启动web服务, 默认监听4000端口
hexo server /  hexo s

```

如此就可以在本地访问到一个简单的blog系统了。

‍

## fluid主题

[fluid](https://github.com/fluid-dev/hexo-theme-fluid)是国人开发的一款 Material Design 风格的主题，安装也很简单

```sh
# 安装fluid
cd /blog
npm install --save hexo-theme-fluid

# 下载插件配置文件
wget https://raw.githubusercontent.com/fluid-dev/hexo-theme-fluid/master/_config.yml -O /blog/_config.fluid.yml

# 编辑全局配置文件
vim /blog/_config.yml
 # theme: fluid  # 指定主题
 # language: zh-CN  # 指定语言，会影响主题显示的语言，按需修改

```

‍

## 功能测试

新建about页面，验证博客是否正常

```sh
hexo new page about
```

命令执行后会在`/blog/source/`​目录下新建一个about目录，目录下有`index.md`​的文件，为文件添加一行`layout: about`​

```md
---
title: about
date: 2020-02-23 19:20:33
layout: about
---
```

然后执行`hexo server`​，浏览器访问`ip:4000/about`​

​![image](assets/image-20231111173203-3xwtp53.png)成功！

‍

## 一键部署

hexo支持[一键部署到github](https://hexo.io/docs/one-command-deployment.html)上

### 安装git组件

```sh
# 安装组件
npm install hexo-deployer-git --save
```

### 编辑配置文件`_config.yml`​

​![image](assets/image-20231111185716-9r80pha.png)​

### 配置github连接信息

#### ssh证书

```sh
git config --global user.name "yourname" 
git config --global user.email "youremail"
ssh-keygen -t rsa -C "youremail"
```

配置后使用以下命令测试是否配置成功

```sh
ssh -T git@github.com
```

发现不太成功，原因是[家中使用的网络代理封禁了 Github 端口 22 的连接](https://blog.csdn.net/KevinHades/article/details/128848004)，解决办法：

```sh
# vim ~/.ssh/config
Host github.com
    HostName ssh.github.com
    User git
    Port 443
```

​![image](assets/image-20231111193944-tzqunhs.png)​

然后

```bash
cat ~/.ssh/id_rsa.pub
```

将上述命令得到的信息写入github设置中

​![image](assets/image-20231111190246-1qea9n7.png)​

执行

```sh
hexo clean
hexo g 		# 生成静态文件, 也就是public目录
hexo d		# 部署
```

部署还需要token，在执行`hexo d`​时输入

​![image](assets/image-20231111190631-2eetv77.png)​

#### 注册token

进入setting页面，左侧最下面找到`Developer settings`​

​![image](assets/image-20231111190803-vosmgbm.png)​

然后按照要求填写即可

![image](assets/image-20231111190845-xewy5fc.png)​

注意需要勾选repo权限选项

​![image](assets/image-20231111190929-pl8h8x2.png)​

‍

## 成功部署

部署成功的输出应如图所示

​![image](assets/image-20231111191055-93ad4sd.png)​

‍

但是后续仍然需要输入token，这不够优雅！

修改`_config.yml`​, 将`https://github.com/HolyTruth/holytruth.github.io`​改为`git@github.com:HolyTruth/holytruth.github.io`​

​![image](assets/image-20231111195849-3lffwzs.png)​

舒服了

‍
