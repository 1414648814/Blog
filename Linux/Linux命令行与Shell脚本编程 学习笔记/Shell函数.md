```sh
hello_fun(){
	echo "hello world"
	echo "$1" # 第一个参数，其中第0个参数为文件本身
}

hello_fun 1

```

在函数名 `hello_fun` 前面加上 function也是同样的效果；

需要注意的地方是

* 在函数调用之前，需要先声明函数（Shell是逐行执行）
* 获得参数方法需要通过 $0...$n，其中$0代表文件本身

例子

```sh
#!/bin/sh
declare num=1000;
 
uname()
{
    echo "test!";
    ((num++));
    return 100;
}
testvar()
{
    local num=10;
    ((num++));
    echo $num;
 
}
 
uname;
echo $?
echo $(uname)
echo $num;
testvar;
echo $num;

```

结果为：

![](http://odwv9d2u8.bkt.clouddn.com/17-4-5/43969552-file_1491327743884_c2cc.png)

可以发现的是

* $? 显示最后命令的退出状态，所以返回了100
* `echo $(uname)` 这段语句并没有将num进行累加，所以要执行函数还是要单独的执行 `uname`
* 局部变量使用local，全局变量使用declare