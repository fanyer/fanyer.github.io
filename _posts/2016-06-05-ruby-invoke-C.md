---
layout: post
title:  "Ruby 使用 Fiddle 调用 C 函数"
date:   2016-06-05 21:38:55 +0800
categories: jekyll update
---


写一个c函数

```
// split.c

double split(double num)
{
  double ret = 0;
  ret = num / 2;
  return ret;
}
```

编译成动态库

```
gcc -o libsplit.so -shared split.c
```

在 split.rb 里调用 libsplit.so 里的 split 函数
```
require 'fiddle'

# Open the file
libsplit = Fiddle.dlopen('./libsplit.so')

# Load the `split` function
split = Fiddle::Function.new(
    libsplit['split'],
    [Fiddle::TYPE_DOUBLE],
    Fiddle::TYPE_DOUBLE
)

# Call the `split` function
puts split.call(10) # => 5
```
* Fiddle.dlopen，与c中调用动态链接库方法名相同dlopen
* Fiddle::Function.new 参数为 函数名，参数，返回值

还可以通过 Fiddle::Importer mixin提供的DSL

```
module Test
  extend Fiddle::Importer
  dlload './libsplit.so'
  extern 'double split(double)'
end
```

puts Test.split(10) # => 5
