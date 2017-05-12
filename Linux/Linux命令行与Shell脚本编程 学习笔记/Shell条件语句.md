# 条件语句

## if

```sh
# if
if condition
then
    command  
fi

# if else
if condition
then
    command
else
    command
fi

# if elif
if condition
then
    command
elif condition
then
    command
else
    command
fi

```

需要注意的是

* elif下面还有个then

## for

```sh
# 第一种表达方式

for v in item1 item2 item3 itemN
do
    command
done

# 第二种表达方式

for (( i=0;i<=10;i++ ))
do
    command
done

```

* 和前面if里面的then fi不相同的是，这里使用do done
* in 列表中可以是字符串，也可以是数字

## while

```sh
while condition
do
    command
done

```

例如

```sh
#!/bin/sh
int=1
while(( $int<=5 ))
do
        echo $int
        let "int++"
done

```

需要注意的是，这里面的let命令，它用于执行一个或者多个表达式，变量中不必使用 `$` 来表示变量，如果不想这么用，按照下面这么用也是可以的：

```sh
int=1
while(( $int<=10 ))
do
        int=$((int+1))
        echo $int
        ((int++))
        echo $int
        printf "\n"
done

```

## until

顾名思义，until循环是指执行一系列命令直到条件为真则停止

```sh
until condition
do
    command
done

```

需要注意的是，判断条件发生在末尾，也就是说至少会执行一次command

## case

多选择语句，类似其它语言中的switch

```sh
case val in
    mode1)
        command
        ;;
    mode2)
        command
        ;;
    *)
        command
        ;;
esac

```

需要注意的是

* 每个模式后面必需以 `)` 来结束
* 在每个选择语句后面都需要加上**两**个 `;`
* mode 为自己填写的模式
* 最后的选择语句和default类似，为 `*)`
* 结束标记 `esac` 为case的反转

例子：

```sh
read int
case $int in
1) 
	echo '选择1'
	;;
3|4|5)
	echo '选择3,4,5'
	;;
*)
	echo '选择了其它'
	;;
esac

```

## break 和 continue

和其它语言中的作用是一样的

## 无限循环

```sh
# 第一种表达方式

while :
do
    command
done

# 第二种表达方式

while true
do
    command
done

# 第三种表达方式

for (( ; ; ))
do
    command
done

```

最后一种方法不知道为什么显示不了高亮，但是执行起来是正确的

