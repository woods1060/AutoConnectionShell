# AutoConnectionShell
西安邮电大学移动应用开发实验室，校园网自动拨号脚本
原理：校园网使用http协议，进行身份验证。在完成登录后，使用DHCP（动态ip分配）为实验室分配一个ip。通过网络转发协议进行外网访问。
当网络处于未验证状态，将会触发302重定向到网络登录页（172.18.0.3/a.htm）,正常验证将会返回正常页面。
## 一、自动连接脚本
**AutoConnectShell.py**
### 1.脚本逻辑
使用循环，每5秒，访问一次百度首页（https://www.baidu.com ）,通过监控返回值是否为200，判定网络连接是否断开。
断开后通过模拟http请求进行登录尝试，尝试验证后，再次验证百度首页。如果登录失败，1秒后将再次触发重连，直至连接成功。
登录成功后，继续进行每5秒的网络连接测试。
### 2.日志处理
当系统在进行脚本启动，网络断开，重连成功时，都会写入日志中。日志中将记录事件内容和详细时间点。
- 日志路径：./log
- 日志分割及命名：日志按照月份进行自动分割。eg：AutoConnectionLog-2019-06.log

## 二、日志清理脚本
**delAutoConnectionLog.py**

由于自动连接脚本，在出现网络波动时，可能会出现短时间大量日志写入操作。
为防止系统因日志容量过大，导致运行缓慢或宕机的情况，使用脚本定期清除连接日志。
- 脚本运行周期：目前该脚本，已使用linux-crontab自动化运维，同时加入开机自启动项。
  - 自动执行周期每月1号0点0分。
  - 同时该清理脚本已经挂载到/etc/rc.local中，每次机器启动，将会自动执行一次清理脚本。
- 清理规则：
  - 1.系统保留1-2月网络脚本日志，清除过期日志。
  - 2.为保证日志清理正常，防止linux-crontab失效，每次删除时，将会对过去两年的日志都进行删除尝试。
- 清理日志：AutoConnectionDel.log中记录删除尝试，和删除成功的信息和具体时间。

## 三、连接脚本的守护脚本
**start.sh**

为防止自动连接脚本因异常退出或被意外杀掉，导致实验室断网。
使用shell守护脚本，监控AutoConnectShell运行情况，如果异常，重启AutoConnectShell。
使用linux-crontab对该shell脚本定期执行（每分钟1次），守护自动连接脚本正常运行。

## 四、启动命令
python StartShell.py


