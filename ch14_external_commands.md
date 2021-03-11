# 外部命令

在Unix系统内部，您会发现许多小型的，超专业化命令，每个命令都能很好地完成一件事情。您可以将这些命令链接在一起以共同解决一个复杂的问题。如果可以从Vim内部使用这些命令，那不是很好吗？

在本章中，您将学习如何扩展Vim以使其与外部命令无缝协作。

# Bang 命令

Vim有一个Bang(`!`)命令，可以执行三件事：

1.将外部命令的STDOUT读入当前缓冲区。
2.将缓冲区的内容作为STDIN写入外部命令。
3.从Vim内部执行外部命令。


# 将STDOUT作为命令读入Vim

将外部命令的STDOUT读入当前缓冲区的语法为：

```
:r !{cmd}
```

`:r`是Vim的读命令。如果不带`!`使用它，则可以使用它来获取文件的内容。如果当前目录中有文件`file1.txt`并运行：

```
:r file1.txt
```

Vim会将`file1.txt`的内容放入当前缓冲区。

如果您运行`:r`命令，然后再执行`!`和外部命令，则该命令的输出将插入到当前缓冲区中。要获取`ls`命令的结果，请运行：

```
:r !ls
```

它返回类似：

```
file1.txt
file2.txt
file3.txt
```

您可以从`curl`命令读取数据：

```
:r !curl -s 'https://jsonplaceholder.typicode.com/todos/1'
```

r命令也接受一个地址：

```
:10r !cat file1.txt
```

现在，将在第10行之后插入来自运行`cat file.txt`的STDOUT。

# 将缓冲区内容写入外部命令

除了保存文件，您还可以使用写命令(`:w`)将当前缓冲区中的文本作为STDIN传递给外部命令。语法为：

```
:w !cmd
```

如果您具有以下表达式：

```
console.log("Hello Vim");
console.log("Vim is awesome");
```

确保在计算机中安装了[node](https://nodejs.org/en/)，然后运行：

```
:w !node
```

Vim将使用`node`执行Javascript表达式来打印“ Hello Vim”和“ Vim很棒”。

当使用`：w`命令时，Vim使用当前缓冲区中的所有文本，与global命令类似（大多数命令行命令，如果不给它传递范围，则仅对当前行执行该命令）。如果您通过`：w`来指定地址：

```
:2w !node
```

Vim只使用第二行中的文本到`node`解释器中。

`:w !node`和`:w! node`之间有一个细微但重要的区别。节点`。使用`:w !node`，您可以将当前缓冲区中的文本“写入”到外部命令`node`中。用`:w! node`，则您将强制保存文件并将其命名为"node"。

# 执行外部命令

您可以使用bang命令从Vim内部执行外部命令。语法为：

```
:!cmd
```

要以长格式查看当前目录的内容，请运行：

```
:!ls -ls
```

要终止在PID 3456上运行的进程，可以运行：

```
:!kill -9 3456
```

您可以在不离开Vim的情况下运行任何外部命令，因此您可以专注于自己的任务。

# 过滤文字

如果给`!`范围，则可用于过滤文本。假设您有：

```
hello vim
hello vim
```

让我们使用`tr` (translate)命令将当前行大写。运行：

```
:.!tr '[:lower:]' '[:upper:]'
```

结果：

```
HELLO VIM
hello vim
```

细目：
- `.!` 在当前行执行filter命令。
- `!tr '[:lower:]' '[:upper:]'` 调用`tr`命令将所有小写字符替换为大写字符。

必须传递范围以运行外部命令作为过滤器。如果您尝试在没有`.`的情况下运行上述命令(`:!tr '[:lower:]' '[:upper:]'`)，则会看到错误。

假设您需要使用awk命令删除两行的第二列：

```
:%!awk "{print $1}"
```

结果：

```
hello
hello
```

细目：
- `:%!`  在所有行上执行filter命令(`%`)。
- `awk "{print $1}"` 仅打印匹配项的第一列。在这种情况下，单词“你好”。

您可以使用链运算符（`|`）链接多个命令，就像在终端中一样。假设您有一个包含这些美味早餐的文件：

```
name price
chocolate pancake 10
buttermilk pancake 9
blueberry pancake 12
```

如果您需要根据价格对它们进行排序，并且仅以均匀的间距显示菜单，则可以运行：

```
:%!awk 'NR > 1' | sort -nk 3 | column -t
```

结果：
```
buttermilk pancake 9
chocolate pancake 10
blueberry pancake 12
```

细目：
- `:%!` 将过滤器应用于所有行（％）。
- `awk 'NR > 1'` 仅从第二行开始显示文本。
- `|`链接下一个命令。
- `sort -nk 3`使用列3（`k 3`）中的值对数字进行排序（`n`）。
- `column -t`以均匀的间距组织文本。

# 普通模式命令

在正常模式下，Vim有一个过滤运算符（`!`）。如果您有以下问候：

```
hello vim
hola vim
bonjour vim
salve vim
```

要大写当前行和下面的行，可以运行：
```
!jtr '[a-z]' '[A-Z]'
```

细目：
- `!j` 运行常规命令过滤器运算符（`!`），目标是当前行及其下方的行。回想一下，因为它是普通模式运算符，所以适用语法规则“动词+名词”。
- `tr '[a-z]' '[A-Z]'`将小写字母替换为大写字母。

filter normal命令仅适用于至少一行或更长的运动/文本对象。如果您尝试运行`!iwtr'[az]''[AZ]'`（在内部单词上执行`tr`），您会发现它在整个行上都应用了tr命令，而不是光标所在的单词开启。

# 聪明地学习外部命令

Vim不是IDE。它是一种轻量级的模式编辑器，通过设计可以高度扩展。由于这种可扩展性，您可以轻松访问系统中的任何外部命令。这样，Vim离成为IDE仅一步之遥。有人说Unix系统是有史以来的第一个IDE。

Bang 命令与您知道多少个外部命令一样有用。如果您的外部命令知识有限，请不要担心。我还有很多东西要学。以此作为持续学习的动力。每当您需要过滤文本时，请查看是否存在可以解决问题的外部命令。不必担心掌握特定命令的所有内容。只需学习完成当前任务所需的内容即可。