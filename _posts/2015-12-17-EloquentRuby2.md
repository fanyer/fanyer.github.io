---
layout: post
title:  "Eloquent Ruby (Chapter 2) 手记"
date:   2015-12-17 21:28:52 +0800
categories: jekyll update
---

### 1.  If, Unless, While, and Until
基于unless的语句有2个好处： 第一， 它比if not要短。第二， 一旦你习惯用它， 你可以轻松的去理解它。前提是你需要熟悉它，如果不熟悉， 你就像穿错了鞋。 但是对于大多数的程序员来说， 已经习惯于unless。
### 2. Use the Modifier Forms Where Appropriate
你可以使用：
```
@title ＝ new_title unless @read_only
```
也可以使用
```
@title  ＝ new_title if @writable
```
同样：
你可以使用  
```
document.print_next_page while document.pages_available?
```
也可以使用      
```
document.print_next_page until document.printed?
```
 你可以灵活的使用它，但是要使他们像这样易读：Do this if that.

### 3.  Use each, Not for
 `for`是基于`each`实现的， 为什么你不直接用`each`呢？ 抛弃`for`吧。
### 4.  A Case of Programming Logic
  `case`语句是用`===`操作符来做比较的。尽量使用`case`吧。
### 5.  Staying Out of Trouble
Ruby只认false和nil为false
记住：
```
puts 'Sorry Dennis Ritchie, but 0 is true!' if 0
puts 'Sorry but "false" is not false' if 'false'
#  均会打印
```
