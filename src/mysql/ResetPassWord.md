# 01 忘记密码

* 打开终端输入：sudo mysqld_safe --skip-grant-tables
* 打开一个新的终端，输入：mysql -u root
* 终端输入 use mysql
* 修改密码：UPDATE mysql.user SET authentication_string=PASSWORD('你的密码') WHERE User='root';
