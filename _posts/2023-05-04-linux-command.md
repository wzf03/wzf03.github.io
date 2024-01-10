---
title: "Linux 命令行工具"
date: 2023-05-04 08:58:24 08:00
categories: [Linux]
tags: [linux, commandline]
image: assets/img/covers/linux.jpg
---

## 使用命令帮助

### 查看命令简要说明

```bash
# 简要说明命令作用
$ whatis command
# 正则匹配
$ whatis -w "loca*"
# 更加详细的说明文档
$ info command
```

注意，至少在archlinux上，使用whatis命令需要先安装`man-db`包并运行`sudo mandb`。

### 使用`man`

```bash
$ man command
# 指定分类
$ man 3 printf
# 根据命令中部分关键字来查询命令
$ man -k keyword
```

man页面的分类标识

```plain
(1)、用户可以操作的命令或者是可执行文件
(2)、系统核心可调用的函数与工具等
(3)、一些常用的函数与数据库
(4)、设备文件的说明
(5)、设置文件或者某些文件的格式
(6)、游戏
(7)、惯例与协议等。例如Linux标准文件系统、网络协议、ASCⅡ，码等说明内容
(8)、系统管理员可用的管理条令
(9)、与内核有关的文件
```

### 查看路径

```bash
# binary文件所在路径
$ which command
# 程序的搜索路径
$ whereis command
```

## 文件及目录管理

### 创建和删除

```bash
# 创建
$ mkdir dir
# 删除
$ rm path
# 删除非空目录
$ rm -rf path
# 删除日志
$ rm *log
$ find . -name "*log" -exec rm {};
# 移动
$ mv src dest
# 复制
$ cp src dest
# 复制目录
$ cp src dest -r
# 查看当前目录下文件个数
$ find . | wc -l
```

### 目录切换

```bash
# 切换目录
$ cd path
# 切换到上一位置
$ cd -
# 切换到home目录
$ cd
$ cd -
# 显示当前路径
$ pwd
```

### 列出目录项

```bash
# 基本
$ ls
# 按时间排序，列表形式
$ ls -lrt
# 列出隐藏文件
$ ls -a
```

### 查找目录及文件

```bash
# 基本, 可以使用正则表达式
$ find . -name "name"
# 显示文件信息
$ find . -name "core*" | xargs file
# 使用建立索引库的查询
$ locate string
# 更新索引库
$ updatedb
```

### 查看文件内容

```bash
# 直接查看内容
$ cat file
# 显示行号
$ cat -n
# 按页显示
$ more file
$ ls -al | more
# 看前10行
$ head -10 file_path
# 看后10行
$ tail -10 file_path
# 查看文件差异
$ diff file1 file2
# 动态显示文本最新信息
$ tail -f file
# 检查文件具体类型
$ file file
```

### 查找文件内容

```bash
egrep 'string' file_path
```

### 文件与目录权限修改

```bash
# 改变文件所有者
$ chown user path
# 改变文件属性(4读，2写，1执行)
$ chmod mode file_path
# 递归修改
$ chown -R user path
# 增加脚本可执行权限
$ chmod +x script_path
```

### 文件别名

```bash
# 硬链接；删除一个，仍能找到
$ ln file fileAgain
# 符号链接；删除源另外一个无法使用
$ ln -s file fileAgain
```

### 管道和重定向

- 批处理命令连接：`|`
- 串联：`;`
- 前面成功则执行后面一条：`&&`
- 前面失败则后一条执行：`||`

```bash
# 输出重定向
$ command > file
# 输入重定向
$ command < file
# 追加重定向
$ command >> file
# Here Document，将两个delimiter之间的内容作为输入传递给command
$ command << delimiter
  document
  delimiter
# 文件描述符重定向，0为标准输入，1为标准输出，2为标准错误输出
n> file
n>> file
# 将输出文件m和n合并
n>&m
# 输入文件合并
n<&m
```

### 设置环境变量

```bash
export PATH=path:$PATH
```

### Bash快捷输入或删除

```plain
Ctl-U   删除光标到行首的所有字符,在某些设置下,删除全行
Ctl-W   删除当前光标到前边的最近一个空格之间的字符
Ctl-H   backspace,删除光标前边的字符
Ctl-R   匹配最相近的一个文件，然后输出
```

## 文本处理

### `find` 命令

```bash
find [path] [expression]
```

- path 可以是一个或者多个路径

- expression 用于指定查找的条件

  - -name pattern: 按文件名查找，支持`*`和`?`
  - -type type: 按文件类型查找
    - `f` 普通文件
    - `d` 目录
    - `l` 符号链接
  - -size [+-]size[cwbkMG]: 按文件大小查找
    - `+-` 表示大于或小于指定大小
    - `cwbkMG` 单位，c（字节）、w（字数）、b（块数）、k（KB）、M（MB）、G（GB）
  - 按修改时间查找，使用`+-`表示在指定时间前或后
    - -amin n: n分钟内被访问
    - -atime n: n天内被访问
    - -cmin n: n分钟内状态发生变化
    - -ctime n: n天内状态发生变化
    - -mmin n: n分钟内被修改
    - -mtime n: n天内被修改
  - -user username: 按文件所有者查找
  - -group groupname: 按文件所属组查找
  - -maxdepth n: 指定搜索深度
  - -regex pattern: 使用正则表达式查找
  - -iregex pattern: 忽略大小写的正则
  - `!` 否定参数，如`find . ! -name "*.txt" -print`查找所有非txt文本

- 后续动作

  - -delete 删除

  - -exec 执行，其中`{}`会被替换为对应的文件名，`\;`是结束标志

    - ```bash
      # 变更所有权
      $ find . -type f -user root -exec chown weber {} \;
      # 将找到的文件复制到另外一个目录
      find . -type f -mtime +10 -name "*.txt" -exec cp {} OLD \;
      ```

  - -print 打印，是`find`的默认输出选项，默认使用`\n`作为定界符

    - -print0 使用`\0`分隔

### `grep` 文本搜索

```bash
grep match_pattern file
```

常用参数

- -o 只输出匹配的文本行
- -v 只输出没有匹配的文本行
- -c 统计文件中包含文本的次数
- -n 打印匹配的行号
- -i 搜索时忽略大小写
- -l 只打印文件名
- -Z 输出以0为结尾符的文件名

```bash
# 在多级目录中递归搜索, -r不跟随符号链接，-R跟随符号链接
$ grep "class" . -R -n
# 匹配多个模式
$ grep -e "class" -e "virtual" file
```

### `xargs` 命令行参数转换

```bash
# 将多行输出转化为单行输出
$ cat file.txt | xargs
# 将单行转化为多行输出, -n 指定每行显示的字段数
$ cat single.txt | xargs -n 3
```

参数

- -d 定义定界符，默认为空格，多行则为`\n`
- -n 指定输出为多行
- -I {} 指定替换字符串
- -0 指定0为输入定界符

```bash
cat file.txt | xargs -I {} ./command.sh -p {} -1

#统计程序行数
find source_dir -type f -name "*.cpp" -print0 | xargs -0 wc -l

#redis通过string存储数据，通过set存储索引，需要通过索引来查询出所有的值：
./redis-cli smembers $1  | awk '{print $1}' | xargs -I {} ./redis-cli get {}
```

### `sort` 排序

- -n 按数字进行排序
- -d 按字典序进行排序
- -r 逆序排序
- -k N 指定按第N列排序
- -b 忽略开头的空格

```bash
sort -nrk 1 data.txt
sort -bd data // 忽略像空格之类的前导空白字符
```

### `uniq` 消除重复行

```bash
# 消除重复行
sort unsort.txt | uniq
# 统计各行在文件中出现的次数
sort unsort.txt | uniq -c
# 找出重复行
sort unsort.txt | uniq -d
```

- -s 开始位置
- -w 比较字符数
