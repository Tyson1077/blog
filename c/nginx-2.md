title: nginx学习2
date: 9:29 2013/7/28

今天尝试写一个模块

ngx_http_module_t
-------

这里面初始化所有8个成员全部是函数指针，同样是可选的，用NULL放弃使用回调。

### create_loc_conf

此函数接受的参数是`ngx_conf_t *`， 也就是nginx配置结构体的指针

返回结果会被放在`ctx->loc_conf[mi]`中，mi是module index的缩写

返回一个有字符串和整数的结构体, 

作用是告诉总的配置文件, 我需要一个啥结构体, 保存哪些信息

这个函数貌似不重要..不管了

### merge_loc_conf

可以不写,和上面一样,没啥用

ngx_command_t
----------

    struct ngx_command_s {
        ngx_str_t             name;
        ngx_uint_t            type;
        char               *(*set)(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);
        ngx_uint_t            conf;
        ngx_uint_t            offset;
        void                 *post;
    };

其中主要是set这一个回调函数

set函数给这个模块加上了handler.

ngx_http_handler_pt handler

handler函数
-------

核心部分

昨天没写出个模块,因为有太多很杂的东西了.

其实和大部分web程序一样, handler才是核心内容.

ngx_http_get_module_loc_conf(r, ngx_http_hi_module)这个函数可以(应该是把配置文件中的信息给r)

假设返回结果放到cglcf中了.

然后就是设置content-type,这永远都是必须的,注意nginx设置字符串全部是用data和len两个属性.

比如

    r->headers_out.content_type.len = sizeof("text/html");
    r->headers_out.content_type.data = (u_char *) "text/html";

当然也要设置状态码.

    r->headers_out.status = NGX_HTTP_OK; // 这当然就是200

设置content_length_n, 不得不说我在nodejs和django中并没有发现返回数据的时候要写content-length.我猜这个content_length_n可能并不是http首部content-length.设置长度很简单

    r->headers_out.content_lenght_n = cglcf->ecdata.len;

设置buffer.

    b = ngx_pcalloc(r->pool, sizeof(ngx_buf_t));

b里面是什么东西呢,我也看不懂,大概是一些起始和末尾位置,chain位置.

设置out, 就是output啦.

    out.buf = b; 
    out.next = NULL; // out可以是一个buffer chain,这里表示后面没有了

调整b的pos地址,指向需要返回字符串的初始位置.

    b->pos = cglcf->ecdata.data; // ecdata是网上抄的例子中的,实际自己取名

设置结束地址

    b->last = cglcf->ecdata.data + (cglcf->ecdata.len)

可以把buffer放入memory

    b->memory = 1;
    b->last_buf = 1; // 这是最后一个buffer,我不知道这跟上面的next = NULL有啥区别


跟上面的发送头部一样先把头部发出去, 头部都存在r->headers_out中

    rc = ngx_http_send_header(r);

如果成功的话, 继续发送output

    return ngx_http_output_filter(r, &out);

这个filter啥意思呢?

其实filter有两个,一个是header filter,在r传进来之前就filter了header.

另一个filter就是这个body filter,是传输body的开始函数.

filter函数是链, 可以把它传给比如gzip这样的filter

交给filter, handler的任务就完成了.

总结一下

1. 先设置头部,必要的有content-type, status, content-length.

2. 设置要返回的buffer, 是通过设置buf的pos来定位字符串的.

3. 发送头部

4. 交给filter

明天计划写一个静态文件模块.

