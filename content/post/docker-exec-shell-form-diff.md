---
title: "Docker Exec Shell Form 区别"
date: "2020-02-02T21:10:18+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - docker
---
记录docker中exec form和shell form的区别,以及CMD和ENTRYPOINT区别,以及最佳实践.
<!--more-->
## exec form VS shell form

```text
# exec form
<instruction> ["executable", "param1", "param2", ...]

# shell form
<instruction> <command>
```


1. exec form 以JSON格式解析,所以命令参数必须使用""引号包裹;
2. exec form 不会invoke shell. 所以`CMD [ "echo", "$HOME" ]`中`$HOME`变量不会被替换;
3. shell form实际是执行`/bin/sh -c "<command>"`;
4. 优先使用 exec form,因为在shell form中[spawns your application in a new process and you won’t receive signals from Docker][1],在k8s中会遇到问题;
5. 在shell form也可以使用`exec <cmd>`形式;


## CMD VS ENTRYPOINT
[直接翻译SO上面的回答,比较清楚][2]
ENTRYPOINT是容器执行入口,CMD是参数设置.

Docker有默认的 ENTRYPOINT:`/bin/sh -c`,但是没有默认的CMD.但是一般镜像都会设置一个默认的CMD.(注意是**docker**的默认 ENTRYPOINT,和**镜像**的默认`CMD`,基础镜像一般不设置 ENTRYPOINT)

`docker run -i -t ubuntu bash` the ENTRYPOINT is the default `/bin/sh -c`, the image is `ubuntu` and the command is `bash`.

所以上面的命令实际上在启动容器中执行了`/bin/sh -c bash`. 不是所有场景都需要"/bin/sh"的,所以引入了 `ENTRYPOINT` and `--entrypoint`.
`docker run -i -t ubuntu`中 `ubuntu`(镜像名)后面跟的所有内容都作为参数传递给entrypoint. 这和使用`CMD`指令是完全一样的,也即是`CMD`指令可以在`docker run`中覆盖.

由于ubuntu镜像设置了默认CMD: `CMD ["bash"]`,所以`docker run -i -t ubuntu`和`docker run -i -t ubuntu bash`是完全一样的效果.

所以到此,可以总结: ENTRYPOINT 是容器的执行入口,CMD是参数设置,不过参数也可以是bash中的可执行命令(例如,`CMD ["echo","hello"]`,实际执行` /bin/sh -c "echo hello"`).


ENTRYPOINT和CMD的搭配可以实现将容器作为一个可执行文件启动,这个特性也是我们日常使用docker的主要目的.例如在dockerfile中设置:
```dockerfile
ENTRYPOINT ["/bin/cat"]
```
运行`docker run cat-img /etc/passwd`,`/etc/passwd` 是cmd, 实际执行的是`/bin/cat /etc/passwd`. 恭喜你,得到一个cat程序,假设你安装了一个linux系统,里面没有cat命令,cat-img镜像就可以实现你想要的功能.

再例如你有个redis镜像,与其运行 `docker run redis-img redis -H srv-host -u toto get key`, 
不如设置`ENTRYPOINT ["redis", "-H", "srv-host", "-u", "toto"]` 然后运行`docker run redis-img get key`.

1. Dockerfile只有最后一个CMD会生效;
2. 可以使用`docker inspect <img-id>`查看默认的`CMD`参数;
3. 取消默认ENTRYPOINT,可以在Dockerfile中设置: `ENTRYPOINT []`


[1]: <https://hynek.me/articles/docker-signals/> "docker-signals"
[2]: <https://stackoverflow.com/questions/21553353/what-is-the-difference-between-cmd-and-entrypoint-in-a-dockerfile/21564990#21564990> "docker-CMD-ENTRYPOINT"

