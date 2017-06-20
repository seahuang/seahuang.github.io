
# iftop使用手册

Author: 海啸

Date: 2017-06-19

## 1. 简介
> iftop是Linux环境类似于top的实时流量监控工具。

![界面](http://www.ex-parrot.com/~pdw/iftop/iftop_mini.png)

[官网 http://www.ex-parrot.com/~pdw/iftop/](http://www.ex-parrot.com/~pdw/iftop/)

## 2. 安装

### 2.1 依赖
+ libpcap
+ libcurses

### 2.2 下载
> iftop最新稳定版本是0.17。旧版和为发布的最新版下载地址为：[http://www.ex-parrot.com/~pdw/iftop/download/?M=D](http://www.ex-parrot.com/~pdw/iftop/download/?M=D)

### 2.3 源码
> iftop源码管理在git: https://code.blinkace.com/pdw/iftop.git

### 2.4 安装方法一：编译安装

1。 CentOS安装依赖包
```
yum install flex byacc libpcap ncurses ncurses-devel libpcap-devel
```

2。 下载编译安装
```
wget http://www.ex-parrot.com/~pdw/iftop/download/iftop-0.17.tar.gz
tar xvfz iftop-0.17.tar.gz
cd iftop-0.17
./configure
make && make install
```

### 2.5 安装方法二：快速安装

1。 安装三方库EPEL

> 参照 [https://www.vpser.net/manage/centos-rhel-linux-third-party-source-epel.html](https://www.vpser.net/manage/centos-rhel-linux-third-party-source-epel.html)

2。 yum快速安装

```
yum install iftop
```

## 运行
1。 直接运行
```
iftop
```

2。 标准视图（端口显示关闭，每主机对显示两行）
![](http://www.ex-parrot.com/~pdw/iftop/iftop_normal.png)

3。 网络服务流量视图（主机名隐藏，显示源端口，每主机对显示一行）
![](http://www.ex-parrot.com/~pdw/iftop/iftop_ports.png)

## 3. 参数及说明
### 3.1 命令格式
```
iftop -h | [-nNpblBP] [-i interface] [-f filter code] [-F net/mask] [-G net6/mask6]
```
### 3.2 参数
```
-h 显示帮助信息;
-n 不解析主机名;
-N 显示网络服务端口，不解析服务名;
-p 混合模式，中间的列表显示的本地主机信息，出现了本机以外的IP信息;
-P 开启端口显示
-l 
-b 不显示流量图形条
-m limit 
   设置流量图形条刻度范围上限，刻度分五个大段显示。指定数字+'K','M'或'G'。例：# iftop -m 100M
-B 显示带宽率单位为：字节/秒，而不是：比特/秒
-i interface
   监听指定网络接口的报文。
-f filter code
   用过滤代码选择统计的报文。只有ip报文被统计。
-F net/mask
   显示特定网段的进出流量，如# iftop -F 10.10.1.0/24或# iftop -F 10.10.1.0/255.255.255.0
-G net6/mask6
   指定IPv6网络下的流量分析。
-c config file
   指定一个可选配置文件。如果存在默认用~/.iftoprc。
-t text output mode
   用没有重叠的文本模式并输出到标准输出界面。
```

### 3.3 显示说明
> 在运行过程中，iftop将使用全部屏幕显示网络信息。在屏幕最上面显示一个刻度，用于标识流量图形条长度对应的数量。

> 屏幕列表的主要部分，显示了每个主机对之间的发送和接收数据的速率，分别为2秒、10秒和40秒内的传输速率。符号<=和=>分别表示数据流的传输方向。例如：

       foo.example.com  =>  bar.example.com      1Kb  500b   100b
                        <=                       2Mb    2Mb    2Mb

> 在屏幕的底部显示多种汇总信息，包括最近40秒的流量峰值，开启监控后的总流量值，最近2、10、40秒平均总传输速率。

### 3.4 控制命令
> 在iftop运行界面，通过键盘命令字符（注意大小写）可以控制iftop

```
1/2/3 根据右侧显示的三列流量数据进行排序（根据10秒平均流量值）;
  b   切换是否显示平均流量图形条;
  B   切换计算2秒或10秒或40秒内的平均流量;
s/d   源/目的聚合，切换所有流量的源地址或目的地址的聚合；
S/D   端口显示命令，分别切换源或目的端口的显示，p切换端口显示的开/关。
  f   可以编辑过滤代码。可能会导致不可预料的行为。
  h   切换是否显示帮助;
j/k   可以向上或向下滚动屏幕显示的连接记录;
  l   显示过滤，输入要过滤的字符（符合POSIX扩展正则表达式）比如ip，按回车后，屏幕就只显示这个IP相关的流量信息;
  L   切换显示画面上边的刻度;刻度不同，流量图形条会有变化;
  n   切换显示本机的IP或主机名;
  N   切换显示端口号或端口服务名称;
  o   切换是否固定只显示当前的连接;
  p   切换是否显示端口信息;
  P   切换暂停/继续显示;
  q   退出监控。
  t   切换显示格式为2行分别显示发送和接收流量/1行只显示发送流量/1行只显示接收流量/1行显示汇总流量;
  T   切换是否显示每个连接的总流量;
</>   分别表示根据源或目的主机名或IP排序;
  !   可以使用shell命令！
```