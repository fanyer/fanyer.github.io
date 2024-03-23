---
layout: post
title:  "Eloquent Ruby (Chapter 3) 手记"
date:   2016-01-02 15:28:32 +0800
categories: jekyll update
---

### 1. Literal Shortcuts
Array和Hash可以用方便的字面量结构来定义：
```
poem_words = %w {twinkle  little star how i wonder}
```
或括号：
```
poem_words = %w (twinkle  little star how i wonder)
```
Hash在Ruby1.9之后，有更方便的写法：
```
book_info = {first_name: ‘Russ’, last_name: ‘Olsen’}
```

### 2. Instant Arrays and Hashes from Method Calls
当给方法传多个参数的时候，使用星号*标记，来把参数包装为一个数组。例如：
```
def add_authors( *names )
  @author + " #{names.join(' ')}"
end
```
Hash也类似，
```
def load_font( specification_hash )
  # Load a font according to specification_hash[:name] etc.
end
```
可以给这个方法传任意参数：
```
load_font( :name > 'times roman', :size > 12 )
```
还可以更进一步， 去掉括号，这样看起来更像是命令，而不像方法调用。
```
load_font :name > 'times roman', :size > 12
```

### 3.  Running Through Your Collection
```
def index_for( word )
  words.find_index { |this_word| word = this_word }
end
```
map,  inject 方法。

### 4.  Beware the Bang!
当心带叹号的方法。
还有`push`, `pop`, `delete`, 和`shift`，`unshift`也会改变数组自身。

### 5. Rely on the Order of Your Hashes
在Ruby1.9中， Array和Hash都是有序的。当你创建了一个新的hash：
```
hey_its_ordered = { first: 'mama', second: 'papa', third: 'baby' }
```
然后：
```
hey_its_ordered.each { |entry| pp entry }
#会输出：
[:first, "mama"]
[:second, "papa"]
[:third, "baby"]
```
### 6.  Staying Out of Trouble
```
array ＝ [ 0, -10, -9, 5, 9 ]
array.each_index {|i| array.delete_at(i) if array[i] < 0}
pp array
```
这段代码会输出：
```
[0, -9, 5, 9]
```
为什么呢？
因为当删除－10的时候，数组的内部索引被弄乱了，所以把-9漏掉了。
这种问题在上个例子中可能容易发现， 但是，情况并不是总是如此， 特别是当你在不同的线程中共享一个集合的时候， 危险在于， 一个线程正在修改被另一些线程迭代的集合。
如果需要不重复的集合，我们需要考虑一下Set.
```
require ‘set’
```
但是，需要注意， Set又是无序的。
