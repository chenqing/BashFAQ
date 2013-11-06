# `翻译` 为什么在bash下读取文件行不用 `for`

> 许多人都认为从文本文件中读取一行一行的应该用`for循环`。这最多也就算个笨拙和低效的方案，而且有时候还会出错。那么取代的我们应该使用`while`，下面就说下为什么：

### 首先我们看看用正确的方式所实现的结果

```
$ ls
10.file  1.file  1.txt	2.file	3.file	4.file
$ cat 1.txt
hello world

*

$ while IFS= read -r aline; do echo "$aline" ; done < 1.txt
hello world

*

```

### 我们现在用`for`来试试：

```
$ for i in $(<1.txt) ;do echo "$i" ;done
hello
world
10.file
1.file
1.txt
2.file
3.file
4.file
....

```

你也看到了，太出乎我们的意料了。首先，一行的两个字段变成2行了，其次，空行没有输出，再者，最后一行的`*` 给`glob（匹配或者扩展通配符）`了，一行一行的输出来了。

接下来，我们尝试使用`IFS（内部字段分隔符）`来避免一行给分割成两行，使用`set -f` 关闭bash的`glob`功能。

```
$ FS=$'\n';set -f ; for i in $(<1.txt) ;do echo "$i" ;done ;set +f; unset IFS
hello world
*
```

现在这句命令比`while`长了一大截了，另外那个空行还是没有出来哦。因为`IFS=$'\n'（或者其他空白符）`会导致shell把多个空白的行压缩成1个，换句话说，塔跳过空行。

目前还没有对此的解决办法。如果使用`IFS`来分割换行符就没办法保留空行。

另外一个问题就是，`IFS`可能也会影响你循环体的部分，如果你循环体要求的不是上面说`IFS`呢，那就更扯淡了。

最后一个使用`for`的问题就是效率低下。`while`一次只读取一行，而`$(<file)`就一下子把整个文件读到内存中去了。小文件还行，你要是一个很大的文件呢？(Bash就会用一个string来保存文件的内容，用另外一个string来保存分割出来的结果，那么使用的内存就会是文件大小的两倍了)。

### Oh，稍等一下：

```
arr=($(cmd))
or

IFS=$'\n'; arr=($(cat file))
or

printf ... $(printf ...)

```

上面的也都会出现前面诉说的问题哦。

 