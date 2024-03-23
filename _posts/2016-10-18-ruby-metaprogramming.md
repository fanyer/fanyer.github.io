---
layout: post
title:  "ruby DSL"
date:   2016-10-18 20:48:52 +0800
categories: jekyll update
---
[Creating a Ruby DSL](https://www.leighhalliday.com/creating-ruby-dsl)归纳笔记

借用 Creating a Ruby DSL 这篇文章中的例子，假设我们想写一段 HTML 代码:
```
<html>
  <body>
    <div id="container">
      <ul class="pretty">
        <li class="active">Item 1</li>
        <li>Item 2</li>
      </ul>
    </div>
  </body>
</html>
```
但又感觉手写代码太麻烦，希望简化它，所以使用一个自创的 DSL:

```
html = HTMLMaker.new.document do
  body do
    div id: "container" do
      ul class: "pretty" do
        li "Item 1", class: :active
        li "Item 2"
      end
    end
  end
end
# 这个 html 变量是一个字符串，值就是上面的 HTML 文档
```

在实际开发时，元编程通常以两种形式体现出他的威力:

1. 提供反射的功能，通过 API 来提供对运行时环境的访问和修改能力
提供执行字符串格式代码的能力
2. 在 Ruby 中，我们可以随意为任何一个类添加、修改甚至删除方法。调用不存在方法时，可以统一进行转发:

```
class TestMissing
  def method_missing(m, *args, &block)
    puts "方法名：#{m}，参数：#{args}，闭包：#{block}"
  end
end

TestMissing.new.say "Hello", "World" do
  puts "Hello, world"
end
# 方法名：say，参数：["Hello", "World"]，闭包：#<Proc:0x007feeea03cb00@t.ruby:7>
```

#### 然后如何利用这个特性来实现dsl

首先我们要定义一个 HTMLMaker 类，并且把 document 方法作为入口。这个方法接收一个闭包，闭包中调用 body 函数，这个函数也提供了闭包，闭包中调用了 div 方法，并且有一个参数 id: "container"……

可见这其实是一个递归调用，无论是 body 还是 div，他们对应着 HTML 标签，其实都是一些并列的方法，方法可以接受若干个键值对，也就是 HTML 中标签的属性，最后再跟上一个闭包用来创建隶属于自己的子标签。

如果不用 Ruby，我们需要事先知道所有的 HTML 标签名，然后进行匹配，可想工作量有多大。而在 Ruby 中，他们都是并列关系，可以统一转发到 method_missing 方法中，获取方法名、参数和闭包。

我们首先解析参数，配合方法名拼凑出当前标签的字符串，然后递归调用闭包即可，核心代码如下:

```
def method_missing(m, *args, &block)
    tag(m, args, &block)
end

def tag(html_tag, args, &block)
  # indent 用来记录行首的空格缩进
  # options 表示解析后的 HTML 属性，比如 id="container"， content 则是标签中的内容
  html << "\n#{indent}<#{html_tag}#{options}>#{content}"
  if block_given? # 如果传递了闭包，递归执行
    instance_eval(&block) # 递归执行闭包
    html << "\n#{indent}"
  end
  html << "</#{html_tag}>"
end
```

这里的 instance_eval 也是一种元编程，表示以当前实例为上下文，执行闭包，具体用途可以参考这篇文章: [Eval, module_eval, and instance_eval](https://4loc.wordpress.com/2009/05/29/eval-module_eval-and-instance_eval/).
