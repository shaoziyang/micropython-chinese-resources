# re（简单正则表达式）

re 模块实现正则表达式操作。正则表达式的语法支持 CPython 的 re 模块子集 (实际是 POSIX 扩展正则表达式的子集)。

支持操作符:

* `.`：匹配任意字符
* `[...]`：匹配字符集，支持单个字符和字符范围，包括否定集（例如 [^a-c]）。
* `^`： 匹配字符串开始
* `$`： 匹配字符串结束
* `?`： 匹配前一个子模式 0 次或 1 次
* `*`： 匹配前一个子模式 0 次或多次
* `+`： 匹配前一个子模式 1 次或多次
* `??`：`?` 的非贪婪版本，匹配零次或一次，优先匹配零次。
* `*?`：`*`的非贪婪版本，匹配零次或多次，优先匹配最短的可能。
* `+?`：`+`的非贪婪版本，匹配一次或多次，优先匹配最短的可能。
* `|`：匹配此运算符左侧或右侧的子模式。
* `(...)`：分组。每个组都是捕获组（可以通过 `match.group()` 方法访问它捕获的子字符串）。
* `\d`：匹配数字，相当于[0-9]。
* `\D`：匹配非数字，相当于[^0-9]。
* `\s`：匹配空格，相当于[ \t-\r]。
* `\S`：匹配非空格，相当于 [^\t-\r]。
* `\w`：匹配"单词字符"（仅 ASCII）。相当于[A-Za-z0-9_]。
* `\W`：匹配非"单词字符"（仅 ASCII）。相当于[^A-Za-z0-9_]。
* `\`：转义字符。除上面列出的字符外，反斜杠后面的任何其他字符都是字面意思。例如，`\*`等效于字符 `*`（不被视为`*`运算符）。注意，`\r`、`\n` 等不被专门处理，将等效于字母 `r`、`n` 等。因此，不建议将原始Python 字符串 (r"")用于正则表达式。例如，r"\r\n"用作正则表达式时等效于"rn"。若要匹配 LF 后的 CR 字符，请使用"\r\n"。

目前不支持的功能：

* 重复计数 `({m,n})`。
* 命名组 `((?P<name>...))`。
* 非捕获组 `((?:...))`。
* 更高级的断言 `(\b, \B)`。
* 特殊字符转义，如\r，\n，使用 Python 自己的转义代替。

示例：

```py
import re

# As re doesn't support escapes itself, use of r"" strings is not
# recommended.
regex = re.compile("[\r\n]")

regex.split("line1\rline2\nline3\r\n")
# Result:
# ['line1', 'line2', 'line3', '', '']
```
	
##  函数

re 模块的相关函数：

- re.`compile`(regex_str [, flags])

  编译正则表达式，返回 `regex` 对象。
<br><br>

- re.`match`(regex_str, string)

  编译 `regex_str` 并根据 `string` 进行匹配。匹配总是从 `string` 的起始位置开始。
<br><br>

- re.`search`(regex_str, string)

  编译 `regex_str` 并在 `string` 中搜索。不同于匹配，它搜索第一个匹配位置的正则表达式字符串 (结果可能是 0，如果无匹配)。
<br><br>	
	
- re.`sub`(regex_str, replace, string, count=0, flags=0 , / )

  编译 `regex_str` 并在 `string` 中搜索，用 `replace` 替换所有匹配项，然后返回新字符串。
  
  `replace` 可以是字符串或函数。如果是字符串，则可以使用 `\number` 和 `\g<number>` 形式的转义序列来扩展为相应的捕获组（对于未匹配的组则扩展为空字符串）。如果 replace 是函数，则它必须接受单个参数（匹配对象）并返回替换字符串。
如果指定了 `count` 且不为零，则替换将在完成指定次数后停止。`flags` 参数会被忽略。

  这个函数是否可用取决于 MicroPython 移植版的设定。
<br><br>

- re.`DEBUG`

  标志参数，显示已编译表达式的调试信息。（可用性取决于 MicroPython 移植版的设定。）


##  Regex 对象

regex 对象是已编译的正则表达式，它是使用 re.compile() 创建的。regex 对象支持下面的方法：

- regex.`match`(string)
- regex.`search`(string)
- regex.`sub`(replace, string, count=0, flags=0, / )

  与模块级函数 `match()`、`search()`和 `sub()`的功能和用法类似。若需将同一正则表达式应用于多个字符串，使用编译后的正则表达式对象的方法会高效得多。
<br><br>

- regex.`split`(string, max_split=-1, / )

  使用正则表达式拆分字符串，如果指定 `max_split` 参数，最多拆分 `max_split` 个元素。返回字符串列表（如果指定了 `max_split`，则列表最多包含 `max_split+1` 个元素）。


## 匹配对象

匹配对象是 `match()` 和 `search()` 方法的返回值，并传递给 `sub()`中的替换函数。

- match.`group`(index)

  返回匹配的（子）字符串。index 为 0 时返回整个匹配，1 及以上返回各个捕获组。仅支持数字组索引。
<br><br>

以下几个函数是否可用取决于 MicroPython 移植版的设定。

- match.`groups()

  返回包含匹配中所有捕获组的子字符串元组。
<br><br>

- match.`start`([index])
- match.`end`([index])

  返回原始字符串中匹配的子字符串或捕获组的起始或结束位置索引。index 默认为整个匹配，否则选择指定的捕获组。
<br><br>

- match.`span`([index])

  返回一个 2 元组 (`match.start(index)`, `match.end(index)`)。

