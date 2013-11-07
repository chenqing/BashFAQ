#BashFaq-001: 如何从一个文件(数据流,变量)一行一行(或者一个字段一个字段)的读取 

[为什么在bash下读取文件行不用 `for`](Why-you-dont-read-lines-with-for.html) 。使用`while`循环和`read` 命令来完成：

```
$  while IFS= read -r line; do
       printf '%s\n' "$line"
   done <"$file"

```

read -r 选项的是阻止转义的(\\或者\n这种)。如果不加`-r`那么所有的`\`都给你丢弃了，所以使用`read`的一个最佳实践就是加上`-r`选项。

不过你要是想拥有下面脚本中[去除开头和结尾的空白](#Trimming)的效果，那你就得把`-r`去掉了。


> `line` 在这里是一个变量名，当然你可以换一个你喜欢的名字替换`line`

`<"$file"` 告诉`while`循环要从变量`$file`中所代表的文件读取内容，当然你可以直接写一个文件的路径+文件名也是可以的。如果你的脚本就是要读取标准输入，那你就用不到重定向了。

如果你的输入源是一个变量或者参数的值，那么BASH就会使用叫`here string`的遍历里面的每一行:

```
    while read -r line; do
      printf '%s\n' "$line"
    done <<< "$var"
```

这同样也可以利用Bourne-type shell的`here document`实现同样的效果(虽然`read -r `是posix标准，而不是BASH的)：

```
    while read -r line; do
      printf '%s\n' "$line"
    done <<EOF
$var
EOF
```
你也可以使用下面的循环来排除以注释`#`开头的行：

```
    # Bash
    while read -r line; do
      [[ $line = \#* ]] && continue
      printf '%s\n' "$line"
    done < "$file"
```

如果你像处理每一行的各个字段，那么你就得使用多个`read` 变量了:

```
    # Input file has 3 columns separated by white space.
    while read -r first_name last_name phone; do
      # Only print the last name (second column)
      printf '%s\n' "$last_name"
    done < "$file"
```

如果字段分隔符不是空白符，那么你就要设置`IFS`[内部字段分割器](BASH-IFS.html)了:

```
    # Extract the username and its shell from /etc/passwd:
    while IFS=: read -r user pass uid gid gecos home shell; do
      printf '%s: %s\n' "$user" "$shell"
    done < /etc/passwd
```

如果是`tab` 就设置`IFS=$'\t'`。

你也没必要知道输入的行里面又多少个字段。如果你给的变量数比字段数多，那多余的变量的值就是空，反过来，给的变量比字段数少，那么最后一个变量将包含所有剩下的值:

```
  read -r first last junk <<< 'Bob Smith 123 Main Street Elk Grove Iowa 123-555-6789'

    # first will contain "Bob", and last will contain "Smith".
    # junk holds everything else.
```

有些人也使用一次性变量`_`来保存那些"垃圾值"（那些忽略的字段）。如果我们不在意那些字段的值跑哪儿去的话，一次性变量也可以一个`read`命令中多次使用:

```
    read -r _ _ first middle last _ <<< "$record"

    # We skip the first two fields, then read the next three.
    # Remember, the final _ can absorb any number of fields.
    # It doesn't need to be repeated there.
```

**`read`**命令是一行一行的处理数据。默认它会[去掉行开始和结尾的空白](bashfaq-067.html).如果你不想这样，那么你就得把`IFS`清空：

```
   # Exact lines, no trimming
    while IFS= read -r line; do
      printf '%s\n' "$line"
    done < "$file"
```

有时候输入是来自于一些命令而不是文件：

```
    some command | while read -r line; do
      printf '%s\n' "$line"
    done
```

这种方法对于使用一串命令处理[`find`](BASH-find.html)查找的结果特别有用:

```
   find . -type f -print0 | while IFS= read -r -d '' file; do
        mv "$file" "${file// /_}"
    done
```

上面的脚本从`find`命令一次获取一个文件名，然后使用`_`来替换空白[重命名文件名](bashfaq-030.html).

请注意`find`命令的`-print0`,它使用[`NUL byte`](http://zh.wikipedia.org/wiki/%E7%A9%BA%E5%AD%97%E7%AC%A6)作为文件名的分隔符。而`read`命令的`-d ''`说明会把所有内容都读到`file`变量中，直到遇到`NUL byte`.默认情况下，`find`和`read`规定都是换行符结尾。然而，如果文件名中有换行符的话，那么这个默认行为就会把文件名分割掉从而导致错误的发生。除此之外，很有必要把`IFS`清空，因为`read`会把开头和结尾的空格去掉。SEE[FAQ20](bashfaq-020.html).

把`find`的结果通过管道输入到`while`循环中把循环放到[`SubShell`](BASH-SubShell.html)中，and may therefore cause problems later on if the commands inside the body of the loop attempt to set variables which need to be used after the loop;在这种情况下，请看[FAQ24](bashfaq24.html)或者[进程替换](BASH-ProcessSubstitution.html).

```
    while read -r line; do
      printf '%s\n' "$line"
    done < <(some command)
```

如果你想把从文件读取的字段保存到[数组](bashfaq-005.html),请看[FAQ005](bashfaq-005.html).

##*我的文件坏了，缺少最后一个换行*

如果一个文件的最后一行不是换行符，那么`read`命令也会读取它，但是会返回错误并把不是换行的那部分丢给read的变量。你可以在循环之后这样处理：

```
    # Emulate cat
    while IFS= read -r line; do
      printf '%s\n' "$line"
    done < "$file"
    [[ -n $line ]] && printf %s "$line"
```

或者:

```
 # This does not work:
    printf 'line 1\ntruncated line 2' | while read -r line; do echo $line; done

    # This does not work either:
    printf 'line 1\ntruncated line 2' | while read -r line; do echo "$line"; done; [[ $line ]] && echo -n "$line"

    # This works:
    printf 'line 1\ntruncated line 2' | { while read -r line; do echo "$line"; done; [[ $line ]] && echo "$line"; }
```

在上面的第一个例子中，循环后的测试不仅没有换行符，也没有了双引号，请看[Quotes](BASH-Quotes.html)或者[Arguments](BASH-Arguments.html).

关于上面第二个例子中为啥没有按照预料中得起作用，请看[FAQ24](bashfaq-024.html).

另外，你也通过简单的给`while`循环加一个`or`的逻辑：

```
while IFS= read -r line || [[ $line ]]; do
      printf '%s\n' "$line"
    done < "$file"

    printf 'line 1\ntruncated line 2' | while read -r line || [[ $line ]]; do echo "$line"; done
    
```

### 如何避免其他命令把输入都吃了

有些命令会把所有的来自标准输入都吃光。下面的这个程序就是对此没有防范的例子:

```
    while read -r line; do
      cat > ignoredfile
      printf '%s\n' "$line"
    done < "$file"
```

上面的结果只会把第一行打印出来，其余的都到了ignoredfile中了。`cat`啜食了所有可用的输入。

一个解决方法就是使用[文件描述符](BASH-FileDescriptor.html)而不是标准输入。

```
   # Bash
    while read -r -u 9 line; do
      cat > ignoredfile
      printf '%s\n' "$line"
    done 9< "$file"

    # Note that read -u is not portable(可移植), and pretty much useless when you can use simple redirections instead:
    while read -r line <&9; do
      cat > ignoredfile
      printf '%s\n' "$line"
    done 9< "$file"
```

或者：

```
    # Bourne
    exec 9< "$file"
    while read -r line <&9; do
      cat > ignoredfile
      printf '%s\n' "$line"
    done
    exec 9<&-
```

上面的例子会等待用户输入一些东西在每一次迭代中，而不是把每一行都吃掉。

You might need this, for example, with mencoder which will accept user input if there is any, but will continue silently if there isn't. Other commands that act this way include ssh and ffmpeg.其它的解决方案将会在[FAQ89](bashfaq-089.html)中讨论。







