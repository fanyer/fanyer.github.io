---
layout: post
title:  "ruby RegExp"
date:   2016-01-07 19:34:12 +0800
categories: jekyll update
---

Ruby对正则表达式支持非常好，所以对我经常使用到的做一个总结，包括Ruby中正则的写法，匹配的方法，替换，分组匹配等。

### 1.Ruby中正则表达式的写法
主要有三种

* 在`//`之间，要进行转义
* 在`%r{}`内，不用进行转义
* `Regexp.new()`内，不用进行转义

`/mm\/dd/`，`Regexp.new(“mm/dd”)`，`%r{mm/dd}`三者效果相同，实质都是新建了一个`Regexp`的类。

### 2.匹配的两种方法
`=~`肯定匹配, `!~`否定匹配。

`=~`表达式返回匹配到的位置索引，失败返回nil，符号左右内容可交换

`regexp#match(str)`，返回`MatchData`，一个数组，从0开始，
`match.pre_match`返回匹配前内容，`match.post_match`返回匹配后内容
```
/cat/ =~ "dog and cat" 	#返回8
mt = /cat/.match("bigcatcomes")

"#{mt.pre_match}->#{mt[0]}<-#{mt.post_match}" #返回big->cat<-comes
```
### 3. 替换
很多时候匹配是为了替换，Ruby中进行正则替换非常简单，两个方法即可搞定，`sub()`+`gsub()`。
`sub`只替换第一次匹配，`gsub`（g:global）会替换所有的匹配，没有匹配到返回原字符串的copy

```
str = "ABDADA"
new_str = str.sub(/A/, "*") 	#返回"*BDADA"
new_str2 = str.gsub(/A/, "*")	#返回"*BD*D*"
```
如果想修改原始字符串用`sub!()`和`gsub!()`，没有匹配到返回nil。

方法后面还可以跟`block`，对匹配的字符串进行操作

```
a.gsub(/[aeiou]/) {|vowel| vowel.upcase } # => "qUIck brOwn fOx"
```
### 4. 分组匹配
Ruby的分组匹配与其它语言差别不大，分组匹配表达式是对要进行分组的内容加`()`。
对于匹配到的结果，可以用系统变量`#$1`，`#$2`…索引，也可用`matchData`数组来索引

```
md = /(\d\d):(\d\d)(..)/.match("12:50am") # md为一个MatchData对象
puts "Hour is #$1, minute #$2"
puts "Hour is #{md[1]}, minute #{md[2]}"
```
### 5. 匹配所有
`regexp#match()`只能匹配一次，如果想匹配所有要用`regexp#scan()`
用法示例：

```
"abcabcabz".scan(%r{abc}).each {|item| puts item} # 输出2行abc
```
### 6. 贪婪匹配vs懒惰匹配
这两种匹配属于标准正则表达式内容，与Ruby没关，但新手如果不明白匹配时会发生莫名其妙的错误，所以特别总结一下。

贪婪匹配：尽可能多匹配，正则默认是贪婪匹配。

eg: `a.*b`它将会匹配最长的以`a`开始，以`b`结束的字符串。对于`aabab`的匹配结果是`aabab`。

懒惰匹配：尽可能少匹配。

eg：`a.*?b`对于`aabab`的匹配结果是`aab`和`ab`。

一般是在原来表达式结尾加`?`就由贪婪匹配变成了懒惰匹配。常用的懒惰限定符有（去年最后的问题就是贪婪匹配）：

* `?`重复任意次，但尽可能少重复
* `+?`重复1次或更多次，但尽可能少重复
* `??`重复0次或1次，但尽可能少重复
* `{n,m}?`重复n到m次，但尽可能少重复
* `{n,}?`重复n次以上，但尽可能少重复
