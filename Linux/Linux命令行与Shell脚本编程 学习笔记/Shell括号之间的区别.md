

# 前言

初次学习Shell，对于括号的使用肯定很困惑，所以我打算将其整理成一篇文章

# 单括号

## { }

* 表达变量的值，在不引起歧义的时候可以省略大括号

    例子：
    
    ```sh
    var=1
    echo ${var}
    # 或者echo $var
    
    ```

* `(command1; command2; command3;)` 新开多条命令来执行，各个命令之间用分号隔开，最后一个命令必须要分号来隔开；

## ( )

* `(command1; command2; command3)` 命令组 新开多条命令来执行，各个命令之间用分号隔开，最后一个命令后面可以没有分号；

* 初始化数组
    
    例子：
    
    ```sh
    array=(1 2 3 4)
    
    ```

## [ ]

* 字符串或是数字的比较，可用的运算符只有 `==` 和 `!=` ，比如 `[[ ]]` 里面介绍的；

* 通过下标获取到数组中对应的元素
    
    例子

    ```sh
    
    arr=("a" "b" "c")

    echo ${arr[0]} #输出第一个的内容

    echo ${arr[@]} #输出全部的内容
    
    ```

# 双括号


## (( ))

* `$((exp))` 和`expr exp`效果相同，计算数学表达式exp的数值；计算逻辑运算（常用于算术运算比较，双括号中的变量可以不使用$，支持多个表达式用 `,` 来隔开），exp里面只要符合c语言语法即可，前面的 `$` 是在返回值给变量的时候才加上，如果只是元算可以不用加；

    例子：
    
    ```sh
    var=$(( 1+2 ))
    echo $var
    var=`expr 2 + 2`
    echo $var
    
    ```
    
    结果输出为3和4

## [[ ]]

* 判断结构，将判断语句放在双括号中，如果不想双括号，可以使用多个单裤好，常用于字符串的比较

    例子：
    
    ```sh

    a=10
    if [[ $a != 1 && $a != 2 ]]
    then
        echo "not 1 and not 2"
    fi
        
    if [ $a -ne 1 ] && [ $a != 2 ]
    then
    	echo "not 1 and not 2"
    fi
    
    if [ $a -ne 1 -a $a != 2 ]
    then
    	echo "not 1 and not 2"
    fi
    
    ```
    
* 支持字符串的模式匹配，字符串比较时可以把右边的作为一个**模式**，而不仅仅是一个字符串；
    
    例子
    
```sh

if [[ "hell"=="hello" ]]
then
	printf "not equal\n"
else
	printf "equal\n"
fi

if [[ hello==hell? ]]
then
	printf "pattern true\n"
else
	printf "pattern false\n"
fi

```

    输出结果为
    not equal
    pattern true
