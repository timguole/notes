# sed 基础

### 基本命令格式

```shell
sed -f script-file INPUT-FILE ...
sed -e 'cmd1; cmd2; ...' INPUT-FILE ...
shell-cmd | sed -e 'cmd1; cmd2; ...'
shell-cmd | sed -f script-file
sed -e 'cmd' -e 'cmd' ... INPUT-FILE
```

常用选项：

-n/--quiet/--silent：不输出（只有p命令才输出）

-r/-E：使用扩展的正则表达式

-e：后面参数为sed要执行的命令；该选项可以重复多次

-f：后面参数为sed要执行的命令文件

-i[SUFFIX]：直接修改文件；如果提供SUFFIX，则创建一个以SUFFIX为后缀的备份文件

--follow-symlinks：如果需要修改目标文件，并且目标文件是一个链接，则需要使用该选项；否则链接文件会变成一个独立的普通文件。

### sed模式空间

在此，“sed命令”是指sed程序的命令，比如：a，s，d等。sed命令操作的对象是模式空间。默认情况下，由换行符分割的每一行输入是一个模式空间，sed每次读取一行。g/G/h/H/n/N等sed命令用于暂存模式空间的内容，从暂存中复制内容到模式空间，将多行合并为一个模式空间后再操作

### sed 命令地址

可以使用行号或者正则表达式作为命令的地址。使用逗号分隔的两个地址表示一个地址范围。正则表达式使用斜线（/）包围。

```shell
1 # the first line
1,3 # from the 1st to the 3rd line
/^foo/ # line starting with 'foo'
/foo/,/bar/ # from line containing 'foo' to line containing 'bar'
2,/foo/ # from the 2nd to the line containing 'foo'
```

### sed 命令

```shell
2afoobar # append a line after the 2nd line. The text is 'foobar'

# append two lines after the 2nd line
2afoobar1\
foobar2

# change the 2nd line to foobar
2cfoobar

2d # delete the 2nd line
2rfile.txt # append the content of file.txt after the 2nd line

s/foo/bar/ # replace the first occurence of foo with bar
s/foo/bar/g # replace all occurences of foo with bar

# replace foobar with foobaz. back-reference is from \1 to \9
s/(foo)bar/\1baz/

2p # print the 2nd line. With option -n, only the 2nd line is printed out.

# join line starting with 'foo' with the following line
# e.g.:
#	foo
#	bar
# becomes
#	foo bar
/^foo/N; s/\n//
```

### 使用不同的分隔符

```shell
# use | as delimiter
sed '\|^foo|abar'
sed '\|^foo|,\|^bar|d'
```

