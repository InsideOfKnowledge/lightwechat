# lightwechat

#### 介绍

lightwechat 是一款高性能、轻量级的开源即时通讯软件，目前包括服务器端、pc 端。

#### 功能列表

目前已经实现了如下功能（这里只列举网络相关的功能，其他客户端已经实现的功能不统计在列）：

- 注册
- 登录
- 查找好友、查找群
- 添加好友、添加群
- 好友列表、群列表、最近会话
- 单人聊天功能（包括发文字、表情、窗口抖动、离线文件）
- 群组功能（包括发文字、表情）
- 群发消息
- 修改密码
- 修改个人信息（自定义昵称、签名、个性头像等个人信息）
- 自动升级功能
- 断线重连
- 客户端还有很多细节功能，比如头像有三种显示模式、好友上线动画、聊天记录、聊天自动回复功能等，有兴趣的同学可以自己探索尝试一下吧，这里就不截图了。

#### 编译和安装

##### 服务器端的编译与安装

###### 依赖的开发工具

1. 安装cmake、makefile和gcc。

作系统是 Linux，推荐的版本是 CentOS 7.0 以上。服务器代码使用纯C++11开发，所以您的gcc/g++版本必须至少在4.7以上，推荐的版本是4.8.5。另外，使用cmake和makefile工具进行项目管理和编译，因此您需要安装cmake和makefile工具。

2. 安装 mysql。

   使用的数据库是mysql，如果您使用的是CentOS 7.0及以上系统，需要安装 **mariadb-server**、**mariadb-client** 和 **mariadb-devel**。如果您使用的是其他版本的linux系统，请安装 **mysql-server**、**mysql-client** 和 **mysql-devel**。

   聊天服务 chatserver 会使用到 mysql，mysql 的库名（默认库名叫 lightchat）、登陆用户名和密码配置在 `lightchatserver/etc/chatserver.conf` 文件中。

   首次启动聊天服务chatserver时，程序会自动检测是否存在lightchat这样的库，如果不存在则创建之，并检测相应的数据表是否存在，如果不存在则创建它们。所以，无需手动创建对应的库和表。当然，不排除由于不同的mysql版本对应的SQL语法有细微差别，可能建表会失败，这个时候你可能需要手动建表，建表语句在**lightchatserver/table.sql**中。flamingo目前使用的四个表分别是：

   |        表名         |        用途说明        |
   | :-----------------: | :--------------------: |
   |       t_user        |       用户信息表       |
   | t_user_relationship | 好友关系及群成员信息表 |
   |      t_chatmsg      |       聊天记录表       |

   

###### 编译方法

1. 进入 `lightchatserver` 目录，执行：

   ```
   cmake .
   ```

   如果没有错误，会产生一个 Makefile 文件。

2. 执行如下命令生成可执行文件：**chatserver**、**fileserver**、**imgserver**。

   ```shell
   make
   ```

   

   各个服务的作用如下：

   |  服务名称  |                           用途说明                           |
   | :--------: | :----------------------------------------------------------: |
   | chatserver |        聊天服务，用于基本的用户注册、登陆、聊天功能。        |
   | fileserver | 文件服务，用于聊天窗口中的离线文件的发送和客户端自动升级包的下载。 |
   | imgserver  | 图片服务，用于聊天中图片文件的发送与用户自定义头像的上传与下载。 |

   

   ###### 启动方式

   **chatserver**、**fileserver**、**imgserver** 这三个服务是相互独立的，可以直接在 Linux 中单独启动，也可以以守护进程的形式启动，守护进程形式启动命令：

   ```shell
   ./chatserver -d
   ./fileserver -d
   ./imgserver -d
   ```

   

   ##### 使用的端口说明

   可以使用一下 `lsof` 命令查看端口是否处于正常侦听状态：

   ```shell
   [zhangz@localhost lightchatserver]$ lsof -i -Pn
   COMMAND     PID    USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
   chatserve 18540 zhangz   19u  IPv4 1129031      0t0  TCP *:20000 (LISTEN)
   chatserve 18540 zhangz   27u  IPv4 1129032      0t0  TCP *:8888 (LISTEN)
   chatserve 18540 zhangz   35u  IPv4 1129033      0t0  TCP *:12345 (LISTEN)
   fileserve 18552 zhangz   25u  IPv4 1130523      0t0  TCP *:20001 (LISTEN)
   imgserver 18562 zhangz   25u  IPv4 1131971      0t0  TCP *:20002 (LISTEN)
   ```

   

   | 端口号 |  所属服务  |                           用途说明                           |
   | :----: | :--------: | :----------------------------------------------------------: |
   | 20000  | chatserver |                           聊天端口                           |
   |  8888  | chatserver | 聊天服务的监控端口，可以使用telnet或nc命令远程连接至聊天服务去查看服务器的一些实时状态信息 |
   | 12345  | chatserver | 聊天服务的http端口，聊天服务不仅支持长连接，也支持http短连接。 |
   | 20001  | fileserver |                         文件服务端口                         |
   | 20002  | imgserver  |                         图片服务端口                         |

   上表中的8888端口可以使用nc命令，使用方式： `nc -v ip port` 去连接，其中port这里就是 8888，-v 选项是 nc 命令支持的选项，表示要求显示的信息丰富一点。示例：

   ```
   [zhangz@localhost lightchatserver]$ nc -v 127.0.0.1 8888
   Ncat: Version 6.40 ( http://nmap.org/ncat )
   Ncat: Connected to 127.0.0.1:8888.
   1. help-show help info
   2. ul-show online user list
   3. su-show userinfo specified by userid: su [userid]
   ul
   No user online.
   su
   please specify userid.
   help
   1. help-show help info
   2. ul-show online user list
   3. su-show userinfo specified by userid: su [userid]
   ```

   连接成功以后，你可以根据命令说明输入相应的命令来查看服务器的状态信息，目前只实现三个命令，即上面显示的 help、ul、su，您可以继续增加更多的命令。

   

   **提醒一下：** 服务器代码不仅是一款即时通讯软件的服务器代码，同时也是一款通用的 C++11 服务器框架。

   

   #### pc客户端编译和安装

   使用Visual Studio 2019打开**lightchatclient/**目录下的**lightchat.sln**即可编译。这个解决方案包括三个工程项目，说明如下：

   |   项目名称    |                 项目说明                  |
   | :-----------: | :---------------------------------------: |
   | lightchat.sln |           lightchat pc版主程序            |
   |  CatchScreen  | lightchat聊天中用的抓屏程序，类似QQ截图。 |
   |  iUpdateAuto  |      用户pc端自动升级的zip解压工具。      |

​      生成后的程序位于目录**lightchatclient\Bin**目录下，其中客户端与服务器的连接配置信息位于**lightchatclient\Bin\config\lightchat.ini**文件中，直接打开**lightchat.exe**即可启动lightchat pc端。







