---
layout: post
title:  "某段ruby元编程"
date:   2016-07-10 22:48:52 +0800
categories: jekyll update
---
可以用irb（Ruby的REPL程序）实际运行：

```
class A
  [:scope, :show_snippets, :search_results, :search_objects].each do |name|
    define_method name do
      search
      instance_variable_get "@#{name}"
    end
  end

  def search
    return if @searched
    @scope, @show_snippets, @search_results, @search_objects = 1,2,3,4
    @searched = true
  end

  def call(*attrs)
    attrs.map { |a| send a}
  end
end

a = A.new
[a.scope, a.show_snippets, a.search_results, a.search_objects] # [1, 2, 3, 4]
a.(:scope, :show_snippets, :search_results, :search_objects) # [1, 2, 3, 4]
```

解释一下：

对给定的4个属性名定义相应的getter，并且在获得属性前要先执行幂等的search方法来生成属性值。define_method接受方法名和代码块来定义方法，instance_variable_get根据名字获取对象的成员变量。
定义一个call方法来支持a.(:attr1, :attr2)这种批调用语法。对*attrs每个属性名a调用send a，相当于把方法名发送给对象(send是任何对象都拥有的方法)，让它执行。

**等价的普通Ruby代码如下：**

```
class A
  def scope
    search
    @scope
  end

  def show_snippets
    search
    @show_snippets
  end

  def search_results
    search
    @search_results
  end

  def search_objects
    search
    @search_objects
  end

  def search
    return if @searched
    @scope, @show_snippets, @search_results, @search_objects = 1,2,3,4
    @searched = true
  end
end

a = A.new
[a.scope, a.show_snippets, a.search_results, a.search_objects]
```
