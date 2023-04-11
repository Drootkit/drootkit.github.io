记录一下学习shell编程中的一些代码

# 变量

```shell
#!/bin/bash

you_name="Test Me"
# 这里针对variable的赋值不能有空格，变量名称不能有空格，等号左右不能有空格
echo ${you_name}

# 利用语句命令对变量赋值
# 循环用do-done结构，for结尾用分号表示判断语句结束，同时用``来表示命令行或者是指令
for file_name in `ls`;
do
	echo $file_name
done

# 针对变量的操作
test_variable="crootkit"
readonly test_variable		# 使变量只读

echo $test_variable

# 删除变量(只读变量是不能被删除的,而且设置只读之后不能被恢复)
# unset test_variable

echo $test_variable
```

# 字符串

```shell
#!/bin/bash

# 根据shell的aaa对单引号和双引号的规则，我决定shell中一律使用双引号
name="this is a string"
echo $name

test_str="look at this \"$name\" "
echo "-e" $test_str
echo $test_str
```

## 切片

```shell
#!/bin/bash

# using ${#string} to get the length of string
str="this is a string"
echo "len of str is " ${#str}

# using ${string:i:j} to get the sub string in string
str="acdefghijklmn"
echo ${str:1:5}

# find specific string using `` & expr index $string sub_str
echo `expr index $str c`
```



# 数组

```shell
#!/bin/bash

arry=(1 2 3 4 5 6 7 8 9 10)

# using ${array[@]} to print all array elements
echo ${arry[@]}

# for array, you must use full ${} to print it
arry[0]="123"
arry[1]=999999999999
echo ${arry[@]}

# get elements nunmber [*] or [@]
echo ${#arry[@]}
echo ${#arry[*]}

# delet specific left str by #
# delet specific right str by %
str="http://aabbccddeeffjjhhiiggkk1122334455"
# delet // left str
echo ${str#*//}
# delet // right str
echo ${str%//*}

# if you want delet to point char ,you can use double # or %
echo ${str%%a*}

# using declare arry just like the dictionary
declare -A dict_arry=(["one"]=1 ["two"]=2 ["three"]=3 ["fore"]=4)

echo ${dict_arry['one']}
echo ${dict_arry[@]}
echo ${#dict_arry[*]}
```

# 计算

```shell
#!/bin/bash

# full expressoins should be include by ``
# using 'expr' to add support to shell, because native shell not support normal calculate
# space is required between expreesions and operations
var=`expr $1 + $2`
echo $1 "add" $2 "equal" $var

# conditional expressions must be included by [], and space between expressions and operations
echo $[ $1 == $2 ]	# return 0 or 1

if [ $var == 24 ]
then
	echo "$1 * $2 == 144"
fi

# -eq -ne…… 这些关系运算符只支持数字，不支持字符串
n1="abcdef"
n2="abcdef"
if [ $n1 == $n2 ]
then
	echo "equal"
else
	echo "not equal"
fi

# bool operations 
# -a = and
# -o = or
# ! = not

# 逻辑运算符
# && ||

# string opre:
# -n -z check if the strlen = 0
str="abcd"
if [ -z str ]
then
	echo "zero"
else
	echo "no 0"
fi

```



# 控制流

## 判断

### if-else

```shell
#!/bin/bash

# 利用if判断有多少参数，程序本身自己的启动命令算¥0，但是程序不会将其计算在参数列表内
if [ $# == 1 ]
then
	echo you have one para $*
elif [ $# == 2 ]
then
	echo you have two para $@
else
	echo you have many para $*
fi


#(())test
if (( $# > 3 ));
then
	echo dayu 3 ge para
fi

if [ $# > 3 ]
then
	echo above 3 para
fi
```

### 利用test

```shell
#!/bin/bash

# test 用来判断条件是否成立

n1=$1
n2=$2

if test $n1 -eq $n2
then
	echo 两个参数相同
else
	echo 两个参数不同
fi

if [ $n1 -eq $n2 ]
then
	echo ==
else
	echo !=
fi

s1="abcdefg"
s2="1234567"
readonly $s1

if test $s1 = $s2
then
	echo equal
else
	echo no equal
fi

# 和上面的方法一样，同样可以用来检查文件的一些问题
```

### case-esac

```shell
#!/bin/bash

# 类似于C的switch-case结构，除了语法基本上一样
echo -n input a number: 
read input_num
case $input_num in 
	1)
		echo you 1
		;;
	2)
		echo you 2
		let input_num++
		;;
	3)
		echo you 3
		;;
	4|5|6|7|8|9|10)
		echo you many
		;;
	*)
		echo you miss
		;;
esac
# 执行完匹配就结束了，并不会一直执行下去
```

## 循环

```shell
#!/bin/bash

# 循环的间隔默认是通过 "空格" 
for i in 1 2 3 4 5 6 7
do
	echo -e  $i "\n"
done

str="ab cd ef gh ij kl mn"
for c in $str
do
	echo $c ', '
done

# 使用let来执行表达式，比如let b=9+1之类的东西，不用带¥标记变量
sum=10
while (( $sum < 100 ))
do
	let "sum += 1"
	printf "%d\n" $sum
done

# 通过<<COMMENT #############COMMENT的结构来实现多行注释
<<COMMENT
# using -n cancle \n
echo -n input your name: 
while read Name
do
	echo hello $Name
done

# 通过不写条件构造无线循环
#<<COMMENT
while :
do
	echo -n "123"
done

while true
do 
	echo -n "@222"
done

for (( ; ; ))
do
	echo "33445"
done
COMMENT

# until 循环，该循环和while相反，他会一直执行到条件为真，是一个从假到真多过程。
num=100
until num=10
do
	let num--
done
echo $num
# 跳出循环的指令依旧是break和continue两个指令，用法和其他程序语言基本一致。
```

# 函数

```shell
#!/bin/bash
# 函数返回值只能是一个整数 0-255，默认最后一句的结果是返回值，或者用return，主函数用$?来表示返回值

first_func()
{
	printf "%s" "this is my first func in shell"
}

first_func
echo the retuen is $?

add_func()
{
	echo the ten para is ${10}
	echo the ten para is $10
	
	#这两种计算都是可以的
	# return $(( $1 + $2 ))
	return `expr $1 + $2`
}
# 函数的参数其实和程序的参数是一样的
add_func 12 12 1 1 1 2 2 2 2 2 3 3 3
echo the res is $?
```

## 参数

```shell
#!/bin/bash


# using $n to use para
# first para is this shell
# usr para begin from second
echo "测试shell添加启动参数"
echo "first : $0"
echo "second : $1"
echo "third : $2"

# using $# to get num of para
echo "total" $#

#using $* to print all para
echo "all para" $*

# using $$ to get current PID
echo $$

# using $* the all argu will be analyse to a string
for argumet in "$*"; 
do
	echo $argumet
done

# using $@, the argu will be analyse one by one
for argu in "$@";
do 
	echo $argu
done
```

# print

```shell
#!/bin/bash

# 在shell里，echo和printf的功能基本类似，但是根据教程说，printf的可移植性更高
# printf format_string [arguments]
# %作为格式替换符，类似于c语言；-代表左对齐；数字代表宽度

printf "%-10s %-12s %-12s\n" 我是大傻逼 我在轻化2001 我78kg重
printf "%-10s %-12s %-12s\n" abc def 123aaaaaaa
printf "%-10s %-12s %-12s\n" 1 2 3

# 当参数多余格式字符串,会重用最后一个格式化字符输出
printf "%s %d %f\n" abc 123 4.1234 abcdefh

# using %b to use \
# 如果这里用\s那么就会将n直接输出，但是%B可以解决这个问题
printf "test <%b>" "A\nb"
```

# 文件

读文件直接用cat命令；写文件直接用重定位 >> 

```shell
#!/bin/bash

# check the unix attributes
file_name="testfile"

if [ -e file_name ]
then
	if [ -x file_name ]
	then
		echo $file_name is excutable
	fi
else
	echo "no such a file"
fi
# 类似这种格式，判断符号太多了，先用先查

```

# 重定位

```shell
#!/bin/bash

# 正常的重定位
echo this is a string > redirct_file

# 追加
echo add the new string >> redirct_file

# 如果你不想让命令输出到屏幕上，可以用/dev/null文件，输入到这个文件的内容都会被抛弃
echo aaaaaaaaaaaaaaaaaaaaaaaaaa > /dev/null

# stdin 0 ; stdout 1
# 将报错信息也就是stderr（2）重定向到testfile
a=10
let b=a/0 2>>testfile

```

