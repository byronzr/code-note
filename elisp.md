# base
+ 空列表同时看作即是一个原子，也是一个列表
+ 所有双引号括起来的文本，包括标点符号和空格，都是单个原子
+ 列表既是程序
+ 单引号`'`，既是引用，当单引号位于列表之前，它告诉 **Lisp不要对这个列表做任何操作**。
+ 一个符号可以同时具有函数定义和一个值。两者是各自独立的。
+ 一个以(p)结尾的函数，通常是用来测试真假的，例如：bufferp

```elisp
    (set 'flowers '(test test2 test))
```
    '号的作用是使flower不被求值,如果使用set而不对第一个符号加引号的话，flowers将被求值，所以会可能出错。
    emacs 通常在求值列表中，由内向外逐层求值

# 关于 "." 的强记
```elisp
(setq test  '(a  (b  (c  (d . (e . nil))))))
;; (a (b (c (d e))))
(setq test  '(a . (b . (c . (d . (e . nil))))))
;; (a b c d e)
```

# (defun) 
> 定义函数
> (&optional arg) 将函数参数定义为可选
> (&rest args) 任意数量的参数

``` lisp
(defun name-of-function(buffer start end)
    "documentation..."
    (interactive "BAppend to buffer:\nr") 
    ;; buffer 通过B传递，而B后面的字符串提供了提示作为。
    ;; start,end通过r传递。而交互参数通过\n分隔了。
    body-of-function...)
```

1. 符号名。 
2. 参数列表，无参数要传空列表（）。
3. 描述文档。
4. 一个使函数成为交互函数的表达式。
5. 函数主体。
6. 
---

### (message FORMAT-STRING &rest ARGS) 
> 支持 %d / %s 

### (thing-at-point ) 
```
(thing-at-point THING &optional NO-PROPERTIES)

Return the THING at point.
THING should be a symbol specifying a type of syntactic entity.
Possibilities include ‘symbol’, ‘list’, ‘sexp’, ‘defun’,
‘filename’, ‘url’, ‘email’, ‘word’, ‘sentence’, ‘whitespace’,
‘line’, ‘number’, and ‘page’.

When the optional argument NO-PROPERTIES is non-nil,
strip text properties from the return value.

See the file ‘thingatpt.el’ for documentation on how to define
a symbol as a valid THING.
```

### (set)  
> 赋值与绑定,lisp会自动对原子进行求值,所以必须第一个参数普遍添加单引号


### (setq)  
> 除了自动为参数一进行单引号处理，还能多次赋值。将（参数N）赋值给参数（N-1）

### (buffer-name)
> 缓冲区名称

### (buffer-file-name)
> 缓冲区完整文件路径名

### (current-buffer)
> 获得缓冲区

### (other-buffer)
> 最近使用过的缓冲区

### (switch-to-buffer (other-buffer)) 
> 切换缓冲区，需要指定一个缓冲区名称

### (set-buffer)
> 转移工作缓冲区，但并不会显示在工作区域

### (buffer-size)
> 当前缓冲区大小

### (point)
> 返回光标位置

### (point-min)
> 最小可能值，通常为 1

### (point-max)
> 最大可能值


​    
### (interactive)
> 交互函数表达式，多个控制符以\n分隔。可无参数

字符 | description
----|---
* | 星号用于只读缓冲区的调用（测试当前缓冲区是否只读而中断，并警告）
b | 一个必须存在的缓冲区名字
B | 一个可以不存在的缓冲区名字
f | 一个已经存在的文件名字
r | 位点与标记，小的在前面。这是唯一定义两个连续参数而不是一个参数的控制符
* | 只读 buffer
@ | Select the window mentioned in the first mouse event in the key sequence that invoked this command. Special.
^| If the command was invoked through shift-translation, set the mark and activate the region temporarily, or extend an already active region, before the command is run. If the command was invoked without shift-translation, and the region is temporarily active, deactivate the region before the command is run. Special.
a|A function name (i.e., a symbol satisfying fboundp). Existing, Completion, Prompt.
b|（默认为当前buffer name）
B|一个不存在的的buffer name,如果存在，则进行确认
c|A character. The cursor does not move into the echo area. Prompt.
C|A command name (i.e., a symbol satisfying commandp). Existing, Completion, Prompt.
d|The position of point, as an integer (see Point). No I/O.
D|A directory name. The default is the current default directory of the current buffer, default-directory (see File Name Expansion). Existing, Completion, Default, Prompt.
e|The first or next non-keyboard event in the key sequence that invoked the command. More precisely, ‘e’ gets events that are lists, so you can look at the data in the lists. See Input Events. No I/O.You use ‘e’ for mouse events and for special system events (see Misc Events). The event list that the command receives depends on the event. See Input Events, which describes the forms of the list for each event in the corresponding subsections.You can use ‘e’ more than once in a single command’s interactive specification. If the key sequence that invoked the command has n events that are lists, the nth ‘e’ provides the nth such event. Events that are not lists, such as function keys and ASCII characters, do not count where ‘e’ is concerned.
f|A file name of an existing file (see File Names). The default directory is default-directory. Existing, Completion, Default, Prompt.
F|A file name. The file need not exist. Completion, Default, Prompt.
G|A file name. The file need not exist. If the user enters just a directory name, then the value is just that directory name, with no file name within the directory added. Completion, Default, Prompt.
i|An irrelevant argument. This code always supplies nil as the argument’s value. No I/O.
k|A key sequence (see Key Sequences). This keeps reading events until a command (or undefined command) is found in the current key maps. The key sequence argument is represented as a string or vector. The cursor does not move into the echo area. Prompt.If ‘k’ reads a key sequence that ends with a down-event, it also reads and discards the following up-event. You can get access to that up-event with the ‘U’ code character.This kind of input is used by commands such as describe-key and global-set-key.
K|A key sequence, whose definition you intend to change. This works like ‘k’, except that it suppresses, for the last input event in the key sequence, the conversions that are normally used (when necessary) to convert an undefined key into a defined one.
m|The position of the mark, as an integer. No I/O.
M|Arbitrary text, read in the minibuffer using the current buffer’s input method, and returned as a string (see Input Methods in The GNU Emacs Manual). Prompt.
n|数值 ‘n’ never uses the prefix argument. Prompt.
N|The numeric prefix argument; but if there is no prefix argument, read a number as with n. The value is always a number. See Prefix Command Arguments. Prompt.
p|The numeric prefix argument. (Note that this ‘p’ is lower case.) No I/O.
P|The raw prefix argument. (Note that this ‘P’ is upper case.) No I/O.
r|Point and the mark, as two numeric arguments, smallest first. This is the only code letter that specifies two successive arguments rather than one. No I/O.
s|字符串
S|An interned symbol whose name is read in the minibuffer. Terminate the input with either C-j or RET. Other characters that normally terminate a symbol (e.g., whitespace, parentheses and brackets) do not do so here. Prompt.
U|A key sequence or nil. Can be used after a ‘k’ or ‘K’ argument to get the up-event that was discarded (if any) after ‘k’ or ‘K’ read a down-event. If no up-event has been discarded, ‘U’ provides nil as the argument. No I/O.
v|A variable declared to be a user option (i.e., satisfying the predicate custom-variable-p). This reads the variable using read-variable. See Definition of read-variable. Existing, Completion, Prompt.
x|A Lisp object, specified with its read syntax, terminated with a C-j or RET. The object is not evaluated. See Object from Minibuffer. Prompt.
X|A Lisp form’s value. ‘X’ reads as ‘x’ does, then evaluates the form so that its value becomes the argument for the command. Prompt.
z|A coding system name (a symbol). If the user enters null input, the argument value is nil. See Coding Systems. Completion, Existing, Prompt.
Z|A coding system name (a symbol)—but only if this command has a prefix argument. With no prefix argument, ‘Z’ provides nil as the argument value. Completion, Existing, Prompt.

### (let)
> 局部变量，多个赋值,let 对第一参数中的各项列表求值（赋值），所以如果不是值对求值时，单个变量被赋值为nil

``` lisp
(let ((variable value)
      (varibale value)
      ...)
      body...)
```
> 注意变量列表界限

``` lisp
(let ((oldbuf (current-buffer)))
...
)
```

> 未初化变量自动绑定 nil

``` lisp
(let ((variable value)
      variable
      varibale
      ...)
      body...)
```

### (let*)
> 虽然与let相似，但赋值过程中后面的变量可以使用前面变量已经设置的值。

```lisp
(let* ((foo 7)                         ;; 如果取消*号下一行 bar 无法完成解释
       (bar (* 3 foo)))                ;; 如果取消编号解释出错
  (message "'bar' is %d." bar))
```


### (if)
function | description
---|---
string= | 判断字符串是否相等
equal/eq | 判断对象是否相等(结构与内容)
>、<、>=、<= | 数值判断

> nil or () ==> false

``` lisp
(if (true-or-false-test)
    (action-to-carry-out-if-test-is-true))
```


### (if-then-else)
``` lisp
(if (true-or-false-test)
    (action-to-carry-out-if-the-test-returns-true)
    (action-to-carry-out-if-the-test-returns-false))
```
### (save-excursion)
> 执行函数体内的多个表达式，最后一个表达式的值将作为返回值，如果点位（point）发生变化，则在退出后恢复。通常配合let使用。除了记录位点，还会记录工作缓冲区。

``` lisp
(let (varlist)
    (save-excursion
        (first)
        (second)
        (third)
        ...
    ))
```

### (push-mark)
> 将当前标记保存到标记环中

### (goto-char)
> 设置当前点位到一个数值上。

### (get-buffer-create)
> 创建或捕获一个指定名称的缓冲区

### (insert-buffer-substring)
> 将参数buffer的region(start,end)插入到当前buffer

### (copy-to-buffer)
> 当前选中region文本拷贝至指定缓冲区。并擦除原有缓冲区内容。

### (insert-buffer)
> 将另外一个缓冲区内容拷贝到当前缓冲区中。
``` lisp
(defun insert-buffer (buffer)
    "insert after point the contents of BUFFER."
    (interactive "*bInsert buffer:")                       ;; *b测试一个已经存在的buffer是否只读
    (or (bufferp buffer)                                   ;; 如果是一个缓冲区，返回缓冲区
        (setq buffer (get-buffer buffer)))                 ;; 如果是一个名字，查找缓冲区
    (let (start end newmark)                               ;; 创建局布变量
        (save-excursion                                    ;; 记录current-buffer的point, start /1
            (save-excursion                                ;; 记录target-buffer的point, start/2
                (set-buffer buffer)                        ;; 切换工作缓冲区
                (setq start (point-min) end (point-max)))  ;; 设置region的start,end,恢复到current-buffer的 point, end/2
            (insert-buffer-substring buffer start end)     ;; 在current-buffer 中插入target-buffer的region
            (setq newmark (point)))                        ;; 将当前插入变更好的point记录在newmark中，并恢复插入前的 point, end/1
        (push-mark newmark)))                              ;; 将newmark存入mark，并结束let有效周期。
```

### (or)
> 对多个表达式进行求值，直到第一个求值不为nil并返回其值。

### (and)
> 对多个表达式求值，如果任何一个参数值为nil，则返回nil,如果没有nil返回最后一个参数值

### (prefix-numeric-value)
> 转换前缀参数用于执行算数运算

### (forward-line)
> 将光标移动到下一行的行首，如果数数大于1，移动多行。

### (erase-buffer)
> 删除工作缓冲区的全部内容

### (save-restriction)
> 记录区域隔离状态，完成后恢复，需要注意的是，在与 save-excursion 配合使用时，一定在其之后。除非隔行。
> 官方解释为：调用 save-excursion 之后无法记录缓冲区中变窄(隔离)开启的标记。

### (what-line)
> 当前光标(point)所在行

### (narrowing)
> 隔离

### (widen)
> 取消隔离

### (car)
> 返回列表的第一（个）元素，无论是值还是列表。起源于 Contents of the Address part of the Register

### (cdr)
> 返回列表的除第一元素以外的其余部分，起源于 Contents of the Descrement part of the Register

### (cons)
> 构造列表，要注意的是两点，总是将参数 1 插入列表的第一个元素。如果为一个元素创建一个单独的列表，参数 2 应该是个空列表。

### (length)
> 查询列表的长度

### (nthcdr)
> 重复参数 1 次数的 cdr 循环版

### (setcar)
> 用参数 2 替换参数 1 列表中的第一个元素

### (setcdr)
> 用参数 2 替换参数 1 列表中的第二个之后的所有元素


### progn
> 依次对其每个参数（表达式）求值，取其操作并返回最后一个参数的值。

### (prog1)
> 与prog不同的是prog1返回的是第一次求值。其它的都是附带的效果

### search-forward
> 查找第一个字符串，并且如果找到这个字符串就移动光标位点。
> 参数 1：要查找的字符串
> 参数 2：查找限制范围（可选）
> 参数 3：失败处理 nil/message
> 重复次数

### kill-region
> 剪切region文本，并保存到kill-ring中

### delete-region
> 删除region文本，这是一个C的函数宏。

### copy-region-as-kill
> 将region文本拷贝到kill-ring中

### kill-ring
### kill-ring-yank-pointer
### (yank-pop)
### (rotate-yank-pointer)

### (while)
> 循环，需要注意的是。与现在的高级语言不同（）并不能保证变量的生命周期所以，需要嵌套在let中。

### (cond)
> 类似switch case 的功能。
```lisp
(defun mtest (number)  ;; function start
  (cond ((< number 0)  ;; switch start and case 1 condition
	 "<")          ;; case 1 then part
	((> number 0)  ;; case 2 condition
	 ">")          ;; case 2 then part
	((= number 0)  ;; case 3 the condition
	 "=")))        ;; case 3 then part
	 
(print "the mtest return string :%s " (mtest 0))
(print "the mtest return string :%s " (mtest 1))
(print "the mtest return string :%s " (mtest -1))
```

### (eobp)
> 测试是否为缓冲区结尾 ( End of Buffer P )

### (looking-at)
> 当紧跟在位点之后的文本与传递给looking-at 函数作为其参量的正则表达式匹配时 lokking-at 函数返回 "真"

### （append）
> append 将两个列表连接成一个列表与cons插入不同。表现形式如下。
```lisp
(append '(1 2 3 4) (5 6 7 8))
=> (1 2 3 4 5 6 7 8)

(cons '(1 2 3 4) '(5 6 7 8))
=> ((1 2 3 4) 5 6 7 8)
```

### (directory-files)
> 1: 目录名
> 2: 是否返回绝对路径
> 3: 用于匹配的正则表达式
```lisp
(directory-files "../lisp" t "\\.el$")
```

### (nreverse)
> 破坏性的反转列表

```lisp
(nreverse '(1 2 3 4))
=> (4 3 2 1)
```


# 正则表达式


### (sentence-end)
``` lisp
"[.?!][]\"')}]*\\($\\|  \\|	\\)[
]*"
```

    相当的恶心的正则，实在因为年代久远了。
    举个例子，在“（“，在 re-search-forword 中可以直接匹配“（”。
    
    (\) 单杠是回可匹配的字符转换为特殊符号，
    在 re-search-forward 中，\(s\|n\) 括号有了分组，坚线有了“或”
    
    (\\) 双斜杠，可将单扛转义为可匹配字符。
        在re-search-forward 中，\\(s\\|n\\) 正好匹配”上两行“的字符串。


```
"[.?!][]\"')]*\\($\\| $\\|\t\\|  \\)[ \t\n]*"       // 匹配句子的结束

[.?!][]\"')]*                                       //  . ? ! ] " ' ) 出现0获n次
\\($\\| $\\|\t\\|  \\)[ \t\n]*  					// $ 已经结尾
  													// ( )$ 有一个空格的结尾
  													// \t 
  													// ( )空格
  													// 再连续出现 ( ) \t \n 
  													// 上述情况出现0到n次

 可以看出
 // []....] 紧跟在第一位是一样的
 // ( ) 空格很直白，不是\S
 // \\( 括号要转意成分组: 
 // 分析 
 // 1: \( 不存在这个类似 \t 的 特殊字符. 
 // 2: \\ 转义成了普通的转义字符，类似字符串中使用的方法。
 // 3：\\( 终于成为了在re正则中，意义上的分组模式了
 // 4: \\| 同理
```
### (re-search-forward)
> 以正则表达式方式向前搜索。

------

### (setq default-major-mode 'text-mode)

    设置主模式


### (add-hook 'text-mode-hook 'turn-on-auto-fill)

    增加自动换行

### (setq-default indent-tabs-mode nil)

    设置有效于缓充区的值（可能）

### (global-set-key "\C-cw" 'compare-windows)

    compare-windows 
    将当前缓冲区中的文本与下一个窗口中的文本进行比较。
    它将位点置于每个窗口的开始进行比较，
    只要能够匹配就尽可能使位点移过整个缓冲区。
    （然而我并不知道有什么极其重要的作用，翻译垃圾。）


### (occur)

    用正则匹配所有行，并将结果以菜单方式显示在occur缓冲区

### (global-unset-key "\C-xf")

    解除帮定按钮

### (define-key textinfo-mode-map "\C-c\C-cg" 'texinfo-insert-@group)

    在模式中定义快捷键


### (setq debug-on-error t)

    遇错时，进入调试模式

### (debug-on-entry)

    M-x debug-on-entry RET triangle-bugged RET 
    读取函数时，进入调试器。