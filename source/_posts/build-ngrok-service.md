---
date: 2019-05-24 14:01:58
title: ngrok 服务的搭建过程
tags: 

- ngrok
- 服务
- 示例
- 笔记

---

前段时间为了做微信的相关功能， 需要在开发机器上直接调试， 所以搞了个内网穿透， 今天把搭建过程记录一下。 
<!-- more -->

#### 大纲

- 一、 如何用一分钟实现ngrok内网穿透
- 二、 ngrok的作用
- 三、 材料
- 四、 搭建过程
    - 事前准备
    - 下载ngrok源码
    - ssl证书
    - 安装编译环境
    - 编译
    - 运行
- 五、 注意事项

#### 一、 如何用一分钟实现ngrok内网穿透

安装ngrok， 开启服务， 完成。 本次分享到此结束。 

  

#### 二、 ngrok的作用

网络十分发达， 但是大部分网络以局域网的形式存在， 无法直接访问。 所以需要内网穿透

#### 三、 需要用到的材料

- 一台具有公网ip的服务器
- 一个域名， 有备案最好（示例域名： tunnel.emok.top）
- 一堆依赖的工具， 包括但不限于： 
    - git
    - gcc
    - wget

#### 四、 搭建过程

目前搭建的方案大都是 ： 

1. 事前准备》2. 安装编译环境 》 3. 下载源码 》 4. 获取证书 》 5. 编译 》 6. 运行。 

其中2-4步是可以并行进行， 不需要按顺序的。 流程图如下： 

```
graph TB
A[-事前准备 - ]
B1[-ngrok源码 - ]
B2[-ssl证书 - ]
B3[-编译环境 - ]
C[-编译ngrok - ]
D[-运行服务端 - ]
A-- > B1
A-- > B2
A-- > B3
B1-- > C
B2-- > C
B3-- > C
C-- > D
```

##### 事前准备

事前准备分为两个步骤， 一个是域名解析， 一个是依赖安装。 
域名解析需要到域名服务商那里设置用于ngrok的域名的服务器地址解析， 以及该域名子域名的解析。 
依赖安装是指搭建过程中需要用到的一些底层工具， 包括Git、 gcc、 wget等。 

```
#
CentOS 7
yum -y install zlib-devel openssl-devel perl hg cpio expat-devel gettext-devel curl curl-devel perl-ExtUtils-MakeMaker hg wget gcc gcc-c++
```

##### 下载ngrok源码

下载ngrok的源码是通过Git工具到Github上下载的， 一般Linux默认有Git工具， 如果没有， 可以自己安装（安装方式不限于apt、 yum、 下载安装）。 

```
#
CentOS
yum –y install git
```

或者可以下载Git安装包进行安装， 参考 [这篇文章](http://note.youdao.com/noteshare?id=52b0b40cd61272f497386c9854597906)

安装完使用克隆命令将ngrok代码克隆到某个路径

```
#
git clone[git url]
git clone https: //github.com/inconshreveable/ngrok.git
```

安装结果如图： 
![image](https://note.youdao.com/favicon.ico)  

##### 获取SSL证书

运行ngrok服务需要ssl证书的支持， 一般用于商业服务的ssl证书都是花钱购买别的机构颁发的证书， 这里我们选择自己生成一个证书， 缺点是只能用于自己的服务， 但是对我们来说足够了。 
    
首先需要安装生成证书所需的工具 open-ssl

```
#
CentOS
yum install open-ssl
```

  
  
然后为了便于管理， 我们可以新建一个目录来存放ssl证书。 

```
mkdir cert
cd cert
```

然后生成ssl证书 

> 生成SSL证书时使用的域名要和准备搭载ngrok服务的域名一致。 

```
#
ssl# 指令中的$NGROK_DOMAIN替换成准备的域名
openssl genrsa - out rootCA.key 2048
openssl req - x509 - new - nodes - key rootCA.key - subj "/CN=$NGROK_DOMAIN" - days 5000 - out rootCA.pem
openssl genrsa - out server.key 2048
openssl req - new - key server.key - subj "/CN=$NGROK_DOMAIN" - out server.csr
openssl x509 - req - in server.csr - CA rootCA.pem - CAkey rootCA.key - CAcreateserial - out server.crt - days 5000
```

生成过程中遇到 yes or no 可以直接yes， 遇到需要输入的地方也可以直接回车继续。 
生成完成后会有如图的 6 个文件： 
![image](https://note.youdao.com/favicon.ico)  

然后我们将其中的 rootCA.pem 、 server.crt 、 server.key  三个文件复制到 ngrok 的 assets 目录下， 覆盖原来的默认证书。 

```
cp rootCA.pem ngrok / assets / client / tls / ngrokroot.crt
cp server.crt ngrok / assets / server / tls / snakeoil.crt
cp server.key ngrok / assets / server / tls / snakeoil.key
```

> 如果还没下载源码就先下载源码后在执行覆盖

##### 安装编译环境

> 假设已经安装好依赖环境。 

编译的依赖环境为Golang。 
go的安装可以通过下载源码， 然后编译安装， 也可以直接下载go安装， 后者相对简单一点。 
我们为了简单， 直接下载go的压缩包， 下载下来解压就可以用。 
去官网([https://golang.org/dl/](https://note.youdao.com/))下载go1.4的安装包， 以及最新版本的压缩包， 要选择kind为Archive的。 后面会解释为什么要下载两个版本。 
下载后放在服务器上顺眼的地方， 然后运行安装， 基本都默认就可以。 
![image](https://github.com/emoker/graph-bed/raw/master/notes/58beab878313a01d6b832db1ab2613b2/wx_20190325162423.png)

> 需要说明的是， go1.4 之后的版本是需要基于go1.4 进行自编译， 所以需要安装两次go环境， 一次是安装go1.4， 第二次是安装更新的版本（一般是最新的稳定版）。 

至于为什么要安装最新的go稳定版是因为go1.4会在编译ngrok的过程中出错。 

安装完go后需要配置go的环境配置， 后续的编译要用到。 

```
export GOPATH = /usr/local / ngrok /
    export GOROOT = /usr/local / go /
    export GOROOT_BOOTSTRAP = /usr/local / go / #添加 path
export PATH = $PATH: /usr/local / go / bin
```

##### 编译

通过go语言的指令进行编译， 需要编译一个服务端和需要的客户端。 编译过程本身没有难度， 只要go环境没问题， 一般都不会报错。 

> 需要注意一点， 编译客户端的参数要和需要运行的目标系统及其位数匹配

编译服务端  
Linux 64位： 

```
#
Linux 64
cd / usr / local / go / src
GOOS = linux GOARCH = amd64
    . / make.bash
cd / usr / local / ngrok /
    GOOS = linux GOARCH = amd64
make release - server
```

  
编译客户端  
Windows 64的版本： 

```
#
Windows 64
cd / usr / local / go / src
GOOS = windows GOARCH = amd64
    . / make.bash
cd / usr / local / ngrok /
    GOOS = windows GOARCH = amd64
make release - client
```

Mac 64： 

```
#
Mac 64
cd / usr / local / go / src
GOOS = darwin GOARCH = amd64
    . / make.bash
cd / usr / local / ngrok /
    GOOS = darwin GOARCH = amd64
make release - client
```

##### 运行

运行服务端

```
#
ngrok# 指令中的$NGROK_DOMAIN替换成准备的域名
    /
    usr / local / ngrok / bin / ngrokd - domain = "$NGROK_DOMAIN" - httpAddr = ":80"
```

客户端配置文件格式  
新建配置文件ngrok.cfg， 内容如下： 

```
#
ngrok client option: ngrok.cfg
server_addr: "tunnel.hanyz.cn:4443"
trust_host_root_certs: false
```

客户端运行指令

```
#
ngrok
    . / ngrok - config = . / ngrok.cfg - subdomain abc 80
```

#### 五、 注意事项 & 解答

1. 编译安装golang的时候遇到以下报错： 

```
ngrok package net / http / httptrace: unrecognized
import path "net/http/httptrace"
```

  

> 解决： 安装go最新稳定版

2. 安装最新版golang时遇到以下报错： 

```
$GOPATH must not be set to $GOROOT
```

> 解决： 设置GOROOT

3. 安装最新版golang时遇到以下报错： 

```
Set $GOROOT_BOOTSTRAP to a working Go tree >= Go 1.4.
```

> 解决： 先安装1 .4 再安装最新版

4. 启动客户端后访问页面提示 ： tunnel not found

```
// 如果是以下面的命令启动的，建议将 -httpsAddr 参数块与 -httpAddr 参数调换位置
//由
~/ngrok/bin / ngrokd - domain = "$NGROK_DOMAIN" - httpAddr = ":80" - httpsAddr = ":443"
    //改为：
    ~/ngrok/bin / ngrokd - domain = "$NGROK_DOMAIN" - httpsAddr = ":443" - httpAddr = ":80"
// 经试验有效，但是原因未知
```

  
参考： 
[sunny博客](https://link.jianshu.com/?t=http://sunnyos.com/article-show-48.html)  
[自己服务器搭建NGROK](https://www.jianshu.com/p/406b041a7635)