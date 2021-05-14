# 01 忘记密码

* 打开终端输入：sudo mysqld_safe --skip-grant-tables
  >注意执行该命令前，必须确保系统中**没有mysql进程**
* 打开一个新的终端，输入：mysql -u root
* 终端输入 use mysql
* 修改密码：UPDATE mysql.user SET authentication_string=PASSWORD('你的密码') WHERE User='root';

> alter user user() identified by '你的密码'; 也是设置密码的命令

# 02 explain命令详解

