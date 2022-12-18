> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/TH_NUM/article/details/123746115)

Docker 空间占用查看
-------------

1.13 版本后 Docker 提供给了一些 System 命令，例如 [df 命令](https://so.csdn.net/so/search?q=df%E5%91%BD%E4%BB%A4&spm=1001.2101.3001.7020)查看 Docker 占用分布：

```
$ docker system df
TYPE                TOTAL     ACTIVE    SIZE          RECLAIMABLE
Images              3         0         5.373GB       5.373GB (100%)
Containers          0         0         0B            0B
Local Volumes       0         0         0B            0B
```

```
$ docker system df -v 命令可以进一步查看空间占用细节，以确定是哪个镜像、容器或本地卷占用过高空间
```

清理方法
----

### 1 自动清理命令

`docker system prune`可对空间进行自动清理。  
该命令所清理的对象如下：

```
已停止的容器
未被任何容器使用的卷
未被任何容器所关联的网络
所有悬空的镜像
```

```
已使用的镜像：指所有已被容器（包括stop的）关联的镜像，也就是docker ps -a所看到的所有容器对应的image。
未引用镜像：没有被分配或使用在容器中的镜像
悬空镜像（dangling image）：未配置任何Tag（也就是无法被引用）的镜像。通常是由于镜像编译过程中未指定-t参数配置Tag导致的。
```

`docker system prune`后可以加额外的参数，如：  
`docker system prune -a` ： 一并清除所有未被使用的镜像和悬空镜像。  
`docker system prune -f` ： 用以强制删除，不提示信息。

另外除了 system 级别的，还有针对容器或是镜像级别的删除命令：

```
docker image prune：删除悬空的镜像。

docker container prune：删除无用的容器。
      --默认情况下docker container prune命令会清理掉所有处于stopped状态的容器
      --如果不想那么残忍统统都删掉，也可以使用--filter标志来筛选出不希望被清理掉的容器。例子：清除掉所有停掉的容器，但24内创建的除外：
      --$ docker container prune --filter "until=24h"  

docker volume prune：删除无用的卷。
docker network prune：删除无用的网络
```

手动清除
----

对于悬空镜像和未使用镜像可以使用手动进行个别删除：  
1、  
1.1 删除所有悬空镜像，不删除未使用镜像：

```
docker rmi $(docker images -f "dangling=true" -q)
```

1.2 docker 如何删除＜none＞镜像

```
docker rmi -f $(docker images | grep "none" | awk '{print $3}')
```

2、删除所有未使用镜像和悬空镜像

```
docker rmi $(docker images -q)
```

3、清理卷  
如果卷占用空间过高，可以清除一些不使用的卷，包括一些未被任何容器调用的卷（-v 详细信息中若显示 LINKS = 0，则是未被调用）：  
删除所有未被容器引用的卷：

```
docker volume rm $(docker volume ls -qf dangling=true)
```

4、容器清理  
如果发现是容器占用过高的空间，可以手动删除一些：  
删除所有已退出的容器：

```
docker rm -v $(docker ps -aq -f status=exited)
```

删除所有状态为 dead 的容器

```
docker rm -v $(docker ps -aq -f status=dead)
```