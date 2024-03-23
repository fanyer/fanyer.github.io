---
layout: post
title:  "commonjs异步模块"
date:   2016-07-24 13:48:52 +0800
categories: jekyll update
---
如何包装基础性的异步模块，有几种方案：

### 1.包装到对象属性上
```
var Wrapper = function(){
  this.foo = "bar";
  this.init();
};
Wrapper.prototype.init = function(){
  var wrapper = this;  
  async.function(function(response) {
    wrapper.foo = "foobar";
  });
}
module.exports = new Wrapper();
```

### 2.Promise
按规范来import和export即可


## 3. MQ
用zeroMQ之类的进行队列投递
