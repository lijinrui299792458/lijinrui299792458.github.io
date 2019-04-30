---
layout:     post
title:      Action View
subtitle:   Rails应用系列第四篇
date:       2019-05-01
author:     YunLambert
header-img: img/post-bg-rails4.png
catalog: true
tags:
    - Share
    - Rails
---

> Active View

前面一篇文章讲了一下rails的路由部分，其中涉及了部分Action Pack的只是。但实际上在找到相应的控制器去接受请求之外，还需要处理一些请求。因为控制器的工作之一是响应用户，有以下四种基本方式：
1.渲染模版，从控制器拿取数据渲染视图，生成后发送给浏览器的响应；
2.直接把字符串返回给浏览器；
3.相应Ajax请求是不给浏览器阿松任何相应，但是在任何情况下控制器都会返回一系列HTTP收不因为这是满足某些响应的需求；
4.把其他数据/下载文件发送给客户端

控制器对每个请求只能响应一次，因此在完成处理请求之前，它会检查是否已经发生过响应。如果没有，就会自动去寻找根据控制器和动作命名的模版，并自动渲染它。

## 渲染模版
模版是定义响应内容的文件。Rails原生是支持四种模版格式的：erb，嵌入式Ruby代码；builder，构建XML内容更为方便的格式；CoffeeScript模版，用于创建javascript，可以修改内容在浏览器中的表现和行为。SCSS模版，用于创建CSS样式表，控制内容在浏览器中的表现。

模版在哪里呢？render()方法会假定模版在当前应用的app/views目录中，按约定，这个目录中有针对各个控制器的子目录，用于存放各个控制器的视图。由于需要通过控制器来传数据，所以模版中可以使用控制器中的所有实例变量，也可以通过存取方法去访问控制器的session、flash、headers、logger、params、request、response等对象。

如果我们在控制器中编写这样的代码：
```ruby
def update
  @user = User.find(params[:id])
  if @user.update(user_params)
    render action: show
  end
  render template: "fix_user_errors"
end
```
虽然本意是好的，希望通过render展示视图，但是别忘了控制器对每个请求只响应一次，而且调用render并不会终止动作，所以这段程序就会执行两次render，即二次渲染的DoubleRenderError异常。如果动作中没有写render，控制器会默认去默认位置去调用相应的视图文件。
### 重定向
除了render方法之外，在这里还有一个redirect_to方法。重定向是服务器发送给客户端的一种响应，让客户端接下来尝试访问url以及用于追赶

