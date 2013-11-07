# IFS

在(Bourne, POSIX, ksh, bash)shell中，`IFS`变量被作为输入字段分割器(或内部字段分割器)。通常情况下，一些特殊字符串被当做分隔符来分割输入行。

`IFS`的默认值是`空格` `tab` `换行`等。如果`IFS`被`unset`掉，那么它的值就是默认值。如果设置成一个空值（和unset不一样），那么就没有分隔符了。

`IFS`可以用在不同的地方，但不同的地方意思也不太一样:

* 在`read`命令中，如果后面跟着很多变量，那么`IFS`就会就会使用分割符分割输入行，给每个变量赋值一个字段(如果提供的变量少于分割得到的字段数，那么最后一个变量将包含剩余的字段). 
* 在解释没加引号[WordSplitting](BASH-WordSplitting.html)扩展的时候，`IFS`会把扩展的值分割成多个 `for i in one two three ;do echo "$i" ; done` .
* 在解释`$*`或者`${array[*]}`(注意：这里不是`@`是`*`而且还加了引号)。那么`IFS`就会作为他们的不同的值之间的分隔符，最后体现在输出中。
* `${!posix*}` 同上。
* IFS is used by `complete -W `under programmable completion.

在前两点情况中，`IFS`在处理空白符的时候有许多特殊的规则.`IFS`会把空白分隔符分割的字符串的首尾空白也会一并的删除掉,多个连续的空白符会被当做一个来对待。例如下面：

```
IFS=: read user pwhash uid gid gecos home shell \
   <<< 'statd:x:105:65534::/var/lib/nfs:/bin/false'

IFS=$' \t\n' read one two three \
   <<< '   1      2  3'
```

在第一个例子中,`gecos`会被赋值成空字符串，这也是两个`:`分隔符之间的内容。`:`不会被认为是内容的一部分，而被是认为是分隔符。在第二个例子中变量`one`的值是1，`two`的值是2，`three`的值是3，前面的空格就被修剪掉，多个空格也被合并成一个了。


一些其它的例子:

```
$ IFS= read a b c <<< 'the plain gold ring'
$ echo "=$a="
=the plain gold ring=

$ IFS=$' \t\n' read a b c <<< 'the plain gold ring'
$ echo "=$c="
=gold ring=

$ IFS=$' \t\n' read a b c <<< 'the    plain gold      ring'
$ echo "=$a= =$b= =$c="
=the= =plain= =gold      ring=
```

上面第一个例子，因为缺少分隔符，所以都给了第一个变量了。第二个例子把多余的字段都给了最后一个变量。第三个例子表明在最后一个变量中`IFS`的这些特性是不生效的（空白符的合并压缩等）。

```
$ IFS=: read a b c <<< '1:2:::3::4'
$ echo "=$a= =$b= =$c="
=1= =2= =::3::4=
```

上面是另外一个字段比变量多得例子，同样最后一个变量保存了最后的所有东西，`:`顺便都保持原样了。

```
IFS=:
set -f
for dir in $PATH; do
   ...
done
set +f
unset IFS
```

虽然上面这个例子比较简单，但还是能够说明`IFS`是如何[WordSplitting](BASH-WordSplitting.html)的。如果想要更好的了解，请看[FAQ081](bashfaq-081.html).

## 特别注意

关于`$*`和`"$*"`(还有`${array[*]}`和` ${!prefix*}`)，`IFS`在有括号和没有括号的都会有用。但是在不加括号的时候，`IFS`就不会生效或者无能为力了：

```
$ set -- one two three
$ IFS=+
$ echo $*
one two three
```

看看上面发生了什么？`+`分隔符没有出现在`echo`的输出中。

这不是重点，`IFS`认为`$*`是一个单一的字符串，但是因为没有[引号](BASH-Quotes.html),在`echo`得到之前，已经[WordSplitting](BASH-WordSplitting.html)了。

```
$ tmp=$*
$ echo "$tmp"
one+two+three
```

变量赋值跳过了单词分割([WordSplitting](BASH-WordSplitting.html))(因为这样做也没啥意义，你多个字符串放到一个变量里面)。

根据定义，单词之间的分隔符都是`IFS`的一部分，因此都会有*WorSpliting*的特性。这你没法避免除非你用一个临时变量抑制*WorSpliting*，就像上面的例子中一样。

这也是另外的一些你要多用引号的原因!



