+++
date = "2015-05-16T04:00:52+08:00"
draft = false
title = "block operation in emacs"
categories = ["emacs"]
slug = "first"
+++
之前用vim（ xcode 没有比较好的 emacs 插件）写iOS代码的时候比较喜欢的一个功能就是块操作，比如对于每行都加上`@property`等关键字。
现在开始用Emacs写一点Go的代码，发现Emacs的块操作更加的方便，比如对于选中某几行，然后执行`c-x c-u r t`的命令，然后输入你想加入的前缀即可。
如果想对于每一行加入按顺序标记数字，可以用`c-x c-u r N`。更多的块操作可以猛击[这里](https://www.gnu.org/software/emacs/manual/html_node/emacs/Rectangles.html)。

还有我比较喜欢的一个vim功能而emacs里面没有的是`%`可以跳转到匹配的括号，虽然可以用`c-m-f`和`c-m-b`来代替，但是毕竟对于一个常用的功能，按三个键不是那么的方便。
在EmacsWiki中找到相应的[解决办法](http://www.emacswiki.org/emacs/NavigatingParentheses)，只需要插入如下代码到你的emacs配置文件中即可
```lisp
(global-set-key "%" 'match-paren)
(defun match-paren (arg)
  "Go to the matching paren if on a paren, otherwise insert %."
  (interactive "p")
  (cond ((looking-at "\\s\(") (forward-list 1) (backward-char 1))
        ((looking-at "\\s\)") (forward-char 1) (backward-list 1))
        (t (self-insert-command (or arg 1)))))
```
