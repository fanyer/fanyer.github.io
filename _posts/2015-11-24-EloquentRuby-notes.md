---
layout: post
title:  "Eloquent Ruby (Chapter 1) 手记"
date:   2015-11-24 20:48:52 +0800
categories: jekyll update
---
### 1. 代码的优美从缩进开始：
Ruby的代码缩进约定： 2个空格。 只用2个空格。
### 2. Go Easy on the Comments
注释的时候悠着点儿

Ruby的注释很简单：#符号后面的任何代码都是注释。

但是真正的问题是：什么时候该注释，多少注释才够？

好的Ruby代码是不需要去写注释的。代码应该能够自己解释自己。当需要注释的时候，最好去解释，这些代码如何去用就够了。而不是你为什么要写它，以及它的算法，或者是你如何使它越来越快。只需要告诉我们，如何去用，能记住下面注释例子就更好了：

```
# Class that models a plain text document, complete with title
# and author:
#
# doc Document.new( 'Hamlet', 'Shakespeare', 'To be or...' )
# puts doc.title
# puts doc.author
# puts doc.content
#
# Document instances know how to parse their content into words:
#
# puts doc.words
# puts doc.word_count
```

### 3. Camels for Classes, Snakes Everywhere Else
给类命名使用驼峰式，其他都用蛇式（下划线链接的小写字母: `like_this`）

对于常量，则全部大写: `UPPERCASE`
### 4. Parentheses Are Optional but Are Occasionally
Forbidden

括号是可选的，但有时不需要。当方法没有参数，或者只有一个参数，或者只有一个条件的时候， 不需要括号， 是为了保证我们代码的干净。

### 5.   Folding Up Those Lines
某些简单的方法，完全可以写为一行。比如：
class DocumentException < Exception; end
逗号分隔。


### 6.  Folding Up Those Code Blocks
同样的， Ruby的blocks也可以写为一行：
```
10.times { |n| puts "The number is #{n}" }
```
当然，你也可以使用`do;end`, 但是这两种形式的block本质上是一样的，Ruby并不关心你使用哪种。如果你block里就一句代码，就用`{}`, 如果是多句，就用`do;end`.


### 7. Staying Out of Trouble
当代码太长的时候， 该用do;end就用, 该换行就换行，该上括号就上括号。可读性最重要。
