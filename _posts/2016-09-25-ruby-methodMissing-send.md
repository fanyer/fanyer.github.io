---
layout: post
title:  "ruby 元编程  method_missing send"
date:   2016-09-25 20:48:52 +0800
categories: jekyll update
---


* method_missing，顾名思义，在方法找不到时被调用。有了这个强大元编程工具，我们就能创建动态的方法，比如ActiveRecord中的动态finder

```
class Legislator
  # Pretend this is a real implementation
  def find(conditions = {})
  end

  # Define on self, since it's  a class method
  def self.method_missing(method_sym, *arguments, &block)
    # the first argument is a Symbol, so you need to_s it if you want to pattern match
    if method_sym.to_s =~ /^find_by_(.*)$/
      find($1.to_sym => arguments.first)
    else
      super
    end
  end
end
```


* send，也是一个动态方法调用的强大工具，它的作用的将一个方法以参数的形式传递给对象。
```
class Box
  def open_1
    puts "open box"
  end

  def open_2
    puts "open lock and open box"
  end

  def open_3
    puts "It's a open box"
  end

  def open_4
    puts "I can't open box"
  end

  def open_5
    puts "Oh shit box!"
  end
end

box = Box.new

box.send("open_#{num}")
```
