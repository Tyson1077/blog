date: 22:46 2013/7/27
title: nginxѧϰ1

���������nginxԴ�롣�е�ͷ����������û�ù�nginx���ڶ����Ҳ���c���ԡ�

�������ſ�nginxԴ��ѧϰ��c���ԡ�֮����ѧϰnginx����Ϊnginx�ǳ�ʵ�ã����Ҵ���д�úã������Ѿ��в��ٵ�Դ��������͡�����Ļ������и��ο���

nginx��ģ�������������Զ���ģ�飬������ڵ㡣

nginxģ��Ļ���ԭ�����ܽ���һ�£��������ǿ��ú��������������ض��ط����ú�����

д��js��һ��������ǳ���Ϥ����Ϊjs��cps�����ǲ��ϵ��ú������������лص��ġ�

nginxҲ���Խлص�������Ҳ�ǻ���epoll��kqueue���

ѧϰ��һ��c������cʵ�ֻص��������ú���ָ��ġ�����nginxʹ��ģ��ķ����������

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

��������˵��Ҫ��nginxһ��struct���������㶨��type�� nameɶ�ģ�ͬʱҪ��������¼�����������

��Ϊnginx����ģ����һ�����飬 ÿִ�е�һ�����Դ����¼��ĵط��������������ģ�飬Ҳ�����Ǹ��ṹ�壬Ȼ�����ģ��ṹ���е�ָ������������ս�����ʱ�򴥷�init�¼����Ǿͱ�������ģ�飬������init�ĺ�����ԭ��ܼ򵥡�

��Ȼ��û��������룬����Ӧ�ó����

    if (ngx.xxx) {
        ngx.xxx( ... )
    }

�����Ĵ��롣

��˵c������������ʵ����̫�����ˡ���c��Ļ�����˵�˼ά������Ա��Ҫ��̫��ʱ����c���Ա����ϡ�

�����������ȡдһ��hello worldģ�������