# 传递参数

## 设置权限

chmod +x file.sh

## 传递参数

./file.sh parameter1 ...

## 特殊字符

* $# 传递到脚本的参数个数
* $* 以一个单字符串的形式显示所有向脚本传递的参数（接收全部的参数并且作为一整块输出）
* $$ 脚本运行的当前进程ID号
* $! 后台运行的最后一个进程的ID号
* $@ 和 $* 相同，但是使用时加引号，并在引号中返回每个参数（不同的是逐个调用参数输出）
* $- 显示使用的当前选项
* $? 显示最后命令的推出状态，0表示没有错误，其他值表明有错误