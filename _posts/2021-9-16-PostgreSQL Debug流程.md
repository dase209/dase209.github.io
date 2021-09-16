---
layout: post
title: PostgreSQL Debug流程
---

# 流程

pg的debug执行(以版本REL_13_STABLE为例子):
1. git clone https://github.com/postgres/postgres
2. `git checkout REL_13_STABLE` 切换到稳定版本
3. `sudo apt install flex bison` 安装一下需要安装的包
4. `./configure --enable-debug --prefix=/usr/share/postgresql/13`，删除src/Makefile.global里CFLAGS的-O2。确保编译的是可以debug的版本
5. `make -j 8 && sudo make install` 编译
6. `cd /usr/share/postgresql/13 && sudo mkdir data && sudo touch logfile` 创建文件
7. `sudo chown -R postgres:postgres data && sudo chown postgres:postgres logfile`更改数据目录和logfile为pg的用户和用户组
8. `sudo mkdir /var/run/postgresql && sudo chown -R postgres:postgres /var/run/postgresql`创建unix socket
9. `sudo su postgres`
10. `./bin/initdb -D data`
11. 修改`data/postgresql.conf`中的`unix_socket_directories`选项为`/var/run/postgresql`
12. `./bin/pg_ctl -D data -l logfile start`
13. `cat logfile`确认数据库启动成功
14. `exit`退出postgres 这个user
15. 打开另一个窗口，`ps -aux | grep postgres`找到pg主进程(字样为`postgres: postgres postgres [local] idle`)的`PID`
16. `sudo gdb -tui attach {PID}` tui参数会启动gdb的user interface([TUI模式的快捷键](https://sourceware.org/gdb/current/onlinedocs/gdb/TUI-Keys.html))，显示当前的代码位置和断点
17. 开始你愉快的debug, gdb info functions 可以帮你快速找到你要打断点的函数.基本每次都需要执行pg_parse_query,所以你不知道函数在哪里就给这个打断点就好了