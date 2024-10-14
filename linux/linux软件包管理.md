 **软件包管理**

# RPM包

## 概念阐述

> 何为rpm？

**rpm软件包**

`RPM` :  RPM包管理员（简称**RPM**，全称为**The RPM Package Manager** 此名词既可能指.rpm的文件格式软件包，也可能指其本身的软件包管理器(RPM Package Manager)。

最早由Red Hat研制，现在也由开源社区开发。 目前是GNU/Linux下软件包资源最丰富的软件包类型之一。

> 软件包管理系统是在电脑中自动安装、配制、卸载和升级软件包的工具组合，在各种系统软件和应用软件的安 装管理中均有广泛应用。

 在Linux发行版中，几乎每一个发行版都有自己的软件包管理系统。

**常见的有：** 

- 管理deb软件包的dpkg以及它的前端apt（使用于Debian、Ubuntu）。 
- RPM包管理员以及它的前端dnf（使用于Fedora）、
- 前端yum（使用于Red Hat Enterprise Linux）、
- 前端 ZYpp（使用于openSUSE）
- 前端urpmi（使用于Mandriva Linux、Mageia）等。

> 使用软件包管理系统将大大简化在Linux发行版中安装软件的过程。 

 RPM软件包分为二进制包（Binary）、源代码包（Source）和Delta包 三种。

二进制包可以直接安装在计算机中，而源代码包将会由RPM自动编译、安装。源代码包经常以src.rpm作为后缀名。 

---

## 代表性rpm

下面聊聊两个代表性的rpm。



### **yum**

`yum`: Yum（全称为 Yellow dog Updater, Modified）是一个在 **Fedora 、RedHat 和 CentOS**中的Shell前端软件包管理器。

> 基于RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。 

常用命令：

```shell
#安装软件
yum install foo-x.x.x.rpm
#删除软件：
yum remove foo-x.x.x.rpm
#yum erase foo-x.x.x.rpm

#升级软件：
yum update foo
#yum update foo

#查询信息：
yum info foo
```



### **apt**

`apt`: **linux软件包管理工具**。用于在 Ubuntu, Debian , Linux 发行版上安装、更新、删除和管理 **deb 软件包。**

> apt是Debian及其衍生产品的主要命令行包管理器，它提供了用于搜索、管理和查询的命令行工具、有关软件包的信息以及对所有功能的低级访问 

 作为操作的一部分，APT使用一个文件列出可获得软件包的镜像站点地址，这个文件就是`/etc/apt/sources.list`。

> 客户端在进行安装或升级时先要查询DEB索引清单，从而可以获知所有具有[依赖关系](https://baike.baidu.com/item/依赖关系/9489181?fromModule=lemma_inlink)的软件包，并一同下载到客户端以便安装。
>
> 当客户端需要安装、升级或删除某个软件包时，客户端计算机取得DEB索引清单[压缩文件](https://baike.baidu.com/item/压缩文件/7121310?fromModule=lemma_inlink)后，会将其解压置放于/var/state/apt/lists/，而客户端使用[apt-get](https://baike.baidu.com/item/apt-get?fromModule=lemma_inlink) install或apt-get upgrade命令的时候，就会将这个文件夹内的数据和客户端计算机内的DEB数据库比对，知道哪些DEB已安装、未安装或是可以升级的。

**一些常用命令：**

```shell
#安装删除
sudo apt install package_name
sudo apt remove package_name

#要列出所有可用的软件包，请使用以下命令：
sudo apt list
#该命令将输出所有包的列表，包括有关包的版本和体系结构的信息。要了解是否安装了特定的软件包，可以使用grep命令过滤输出。 [1]
sudo apt list | grep package_name
#要仅列出已安装的软件包，请键入： [1]
sudo apt list --installed
#在实际升级软件包之前，获取可升级软件包的列表可能很有用： [1]
sudo apt list --upgradeable
```



> apt 与 rpm 一样是 linux 软件包管理器。 

## rpm操作命令

### 命名与依赖性

==RPM包命名规则==

> **例子：**
> `Httpd-2.2.15.el6.centos.1.i686.rpm`
>
> **其中：**
>
> Httpd：软件包包名
> 2.2.15：软件版本
> 15：软件发布的次数
> el6.centos：适合的Linux平台
> i686：适合的硬件平台 noarch 表示任何硬件平台都可以安装

rpm rpm包扩展名
如果自己组建rpm包，都以rpm结尾，这样更加清晰，其他管理员可以明白。

> 注意：`Httpd-2.2.15.el6.centos.1.i686.rpm`为包全名，`Httpd `为包名。这两者有明显区别的，Linux系统命令严格区分两者

RPM包依赖性。

- 树形依赖：a→b→c
- 环形依赖：a→b→c→a
- 环形依赖需要把a,b,c三个同时安装
- 模块依赖：模块依赖查询网站：www.rpmfind.net

> 如果安装时遇到问题，出现依赖性错误的话：若被依赖文件以.so.[数字]结尾，则为库依赖，需要直接安装这个软件，错误会自动解决。
> 安装这个包时需要进入网站 www.rpmfind.net 查询被依赖文件



### 安装升级与卸载

- 包全名：操作的包是没有安装的软件包时，使用包全名。

  > 注意路径

- 包名：操作已经安装好的安装的软件包时，使用包名。

  > 包名默认在搜索/var/lib/rpm中的数据库

```shell
rpm -ivh 包全名 
#RPM安装
```

**参数：**

- -i 安装（install）
- -v 显示详细安装信息（verbose）
- -h 显示进度（hash）
- –nodeps 不检测依赖性 一般不用，安装时都得显示依赖性

> 注意：安装一定要用包全名

```shell
rpm -Uvh 包全名 
#RPM包升级
```

**参数：**

- -U 升级
- rpm -e 包名
- -e 卸载
- –nodeps 不检查依赖性

### 查询

```shell
#-q 查询（query）
rpm -q 包名 #查询是否安装

#-a 所有（all）
rpm -qa #查询所有安装的包

rpm -qa | grep [关键字]
#查询所有含义关键字的包。

```

`| `为管道符 。作用是：对于管道符两侧，**左边命令的输出**能作为**右边命令的输入**。

> 注意：
> 1、管道命令只处理前一个命令正确输出，不处理错误输出。
> 2、管道命令右边命令，必须能够接收标准输入流命令才行。

查询安装过软件包详细信息

```shell
rpm -qi 包名 
#-i 查询软件信息（information）
#-p 查询未安装包信息（package）
rpm -qip 包全名
#查询没安装过软件包详细信息 
#因为包没有安装所以得加包全名，因为包在生产好的时候他的信息就已经生成，所以可以查到没安装好的包的信息。
```

查询包中文件安装位置

```shell
rpm -ql 包名 
#-l 列表（list）
#-p 查询未安装包信息（package）
```

查询系统文件属于哪个RPM包

```shell
rpm -qf 系统文件名 
#-f 查询系统文件属于哪个软件包（file）
```

查询软件包的依赖性

```shell
rpm -qR 包名 
#-R 查询软件包的依赖性（requires）
#-p 查询未安装包信息（package）
```



### 校验和文件提取

```shell
rpm -V 已安装的包名 RPM包校验
```

- -V 校验指定RPM包中的文件（verify）

> 例子：`rpm -V httpd `
>
> 显示： S.5……T. c /etc/httpd/conf/heepd.conf

**验证内容**中的8个信息的具体内容：

- S：文件大小是否改变
- M：文件的类型或文件的权限(rwx)是否被改变
- 5：文件MD5校验和是否改变(可以看成文件内容是否改变) MD5是进行文件完整性验证的
- D：设备的中，从代码是否改变
- L：文件路径是否改变
- U：文件的属主(所有者)是否改变
- G：文件的属组是否改变
- T：文件的修改时间是否改变

**文件类型：**

- c：配置文件（config file）

- d：普通文档（documentation）

- g：“鬼”文件（ghost file），

  > 很少见，意识是该文件不应该被这个RPM包 包含。

- l：授权文件（license file）

- r：描述文件（read me）

```shell
rpm2cpio 包全名 | cpio -idv.文件绝对路径
```

> `rpm2cpio` 是将rpm包转换为cpio格式的命令
> `cpio` 是一个标准工具，他用于创建软件档案文件和从档案文件中提取文件



# 源码包

## 源码包和rpm的区别

### **概念不同**

==源码包==

**源码包**：**可以看到源代码**，但是安装时间较慢。像脚本安装包类似Windows安装软件。 可以看作是是写了安装界面的源码包

- 优点：
  1. 开源，如果有足够的能力，可以修改源代码
  2. 可以自由选择所需的功能
  3. 软件是编译安装，所以更适合自己的系统，使用更加稳定也效率更高
  4. 卸载方便，直接删除安装目录
- 缺点：
  1. 安装过程步骤较多，尤其安装较大的软件集合时（如LAMP环境搭建），容易出现拼写错误
  2. 编译过程时间较长，安装比二进制安装时间长
  3. 因为是编译安装，安装过程中一旦报错新手很难解决。

==二进制包==

**二进制包**：RPM包，系统默认包，厂商已经进行了编译，看不到源代码，但是安装时间较快

- 优点：

  1. 包管理系统简单，只通过几个命令就可以实现包的安装，升级，查询和卸载。
  2. 安装速度比源码包安装快得多。

- 缺点：

  1. 经过编译，不再可以看到源代码。

  2. 功能选择不如源码包灵活。

  3. 依赖性 依赖性指的是要想安装A包就得先安装B包，要想安装B包又得先安装C包，所以只能以CBA的顺序。安装RPM包，删除的时候得按ABC顺序删除安装包，基本上所有的RPM包全有依赖性。

     ****

### 安装位置不同

- **RPM包默认安装路径(绝大部分软件文件安装位置)**:
  - `/etc  `安装文件安装目录
  - `/usr/bin/  `可执行的命令安装目录
  - `/usr/lib`  程序所使用的函数库保存位置
  - `/usr/share/doc/` 基本的软件使用手册保存位置
  - `/usr/share/man` 帮助文件保存位置

RPM包默认安装路径(绝大部分软件文件安装位置):

- **源码包安装位置：**

  - 安装在指定位置当中，一般是`/usr/local/软件名/` 。

  

**安装位置不同带来的影响**:

**RPM包：**

RPM包安装的服务可以使用系统服务管理命令(service)来管理，例如RPM包安装的Apache的启动方法是：

```shell
/etc/rc.d/init.d./httpd start
service httpd start #（红帽系列专有命令，如果没有只能靠/etc/rc.d/init.d./启动）
```

RPM包的启动文件全在`/etc/rc.d/init.d./`里，service 会搜索RPM包所有的安装路径，所以 service 才能启动RPM包软件。

但是service启动不了源码包软件，因为源码包在`/usr/local`里，和RPM包不一样。

**源码包**

源码包启动使用**绝对路径加start**。

## 源码包的安装

这里举例（安装Apache）说明一下：

**安装准备**

安装C语言编译器
使用命令：

```shell
yum -y install gcc gcc-c++ autoconf pcre pcre-devel make automake
```

> **执行命令前要下载源码包**：
> http://mirror.bit.edu.cn/apache/httpd/
> 选择任何版本，比如：
> `httpd-2.4.43.tar.bz2 `版本

**注意事项**

- 源码包保存位置：/usr/local/src
- 软件安装位置：/usr/local

然后看看安装过程有没有报错：
安装过程停止，并出现error，warning或者no提示，即发生错误

> 把电脑本机下载好的源码包传输到Linux系统或者服务器上。
> Windows下载WinSCP软件进行本机与虚拟机或者远程传输
> Mac连接远程阿里云服务器，终端使用命令 scp进行传输：

```shell
scp -r localfile.txt username@192.168.0.1:/home/username/
```
> 其中，
> １）scp是命令，-r是参数
> ２）localfile.txt 是文件的路径和文件名
> ３）username是服务器账号，一般为root
> ４）192.168.0.1是要上传的服务器ip地址
> ５）/home/username/是要拷入的文件夹路径，一般为/root 家目录
> 例子：
> `scp -r /Users/yangyangyang/Desktop/httpd-2.4.43.tar.bz2 root@47.95.5.171:/root`

1. **解压下载到源码包**
   使用命令` tar -jxvf httpd-2.4.43.tar.bz2`
   如果是tar.gz压缩包可以使用 tar -zxvf 命令

2. **进入解压缩目录**
   输入命令：

   ```shell
   cd httpd-2.4.43
   ```

   > INSTALL：安装说明
   > README：使用说明

   进入安装说明：

   ```shell
   vi INSTALL
   $ ./configure --prefix=PREFIX 编译前准备
   $ make 进行编译
   $ make install 编译安装
   $ PREFIX/bin/apachectl star 启动命令
   ```

   > 这些为详细的安装步骤，其中：
   > ./configure为软件配置与检查 我们也称编译前准备
   > 1.定义需要的功能选项。
   > 2.检测系统环境是否符合安装要求
   > 3.把定义好的功能选项和检测系统环境的信息都写入Makefile文件，用于后续的编辑。

3. **定义安装路径**
   退出之后输入命令：`./configure --prefix=/usr/local/apache2`

   > 如果报错显示：
   > 进以下网址寻求解决办法
   > http://www.cnblogs.com/yuzhaokai0523/p/4382974.html

4. **完成定义**
   输入命令：`make `进行编译

5. **完成编译**
   输入命令：`make install `编译安装

6. **启动**
   输入命令

   ```shell
   /usr/local/apache2/bin/apachectl start
   $ PREFIX/bin/apachectl star #$pREFIX为软件安装路径
   ```

   

---

# 第三种包：脚本包

 - 脚本安装包并不是独立的软件包类型，常见安装的是源码包。
 - 是人为把安装过程写成了自动安装的脚步，只要执行脚本，定义简单的参赛，就可以完成安装。
 - 非常类似于Windows下软件的安装方式。

使用绝对路径直接回车安装。

> linux中.sh文件即脚本文件，一般都是bash脚本。 