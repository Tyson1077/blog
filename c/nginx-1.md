date: 22:46 2013/7/27
title: nginx学习1

最近看了下nginx源码。有点头大。首先是我没用过nginx，第二是我不会c语言。

因此想借着看nginx源码学习下c语言。之所以学习nginx是因为nginx非常实用，而且代码写得好，并且已经有不少的源码分析博客。不会的话至少有个参考。

nginx有模块这个概念，可以自定义模块，这是入口点。

nginx模块的基本原理我总结了一下，基本就是靠用函数当参数，在特定地方调用函数。

写过js的一定对这个非常熟悉。因为js的cps风格就是不断的用函数做参数进行回调的。

nginx也可以叫回调，反正也是基于epoll和kqueue的嘛。

学习了一下c，发现c实现回调函数是用函数指针的。整个nginx使用模块的方法大概如下

    #include <stdio.h>

    void init(char *str) {
      printf("init: %s\n\n", str);
    }

    void bye(char *str) {
      printf("bye: %s\n\n", str);
    }

    struct ngx_module_t {
      char *name;
      int type;
      void (*init)(char *);
      void (*handle)(char *);
      void (*bye)(char *);
    } ngx = {
      "my_module",
      3,
      &init,
      NULL,
      &bye
    };
    
    int main() {
      if (ngx.init)
        ngx.init("welcome");
      printf("I'm handle the request\n");
      if (ngx.handle)
        ngx.handle("handle it!\n");
      if (ngx.bye)
        ngx.bye("have finished!");
      return 1;
    }

基本就是说你要给nginx一个struct，里面有你定的type， name啥的，同时要定义你的事件触发函数。

因为nginx整个模块是一个数组， 每执行到一个可以触发事件的地方，都会遍历所有模块，也就是那个结构体，然后访问模块结构体中的指定函数，比如刚进来的时候触发init事件，那就遍历所有模块，调用其init的函数。原理很简单。

虽然还没看具体代码，不过应该充斥着

    if (ngx.xxx) {
        ngx.xxx( ... )
    }

这样的代码。

话说c的声明，调用实在是太罗嗦了。用c真的会局限人的思维，程序员需要花太多时间在c语言本身上。

明天继续，争取写一个hello world模块出来。