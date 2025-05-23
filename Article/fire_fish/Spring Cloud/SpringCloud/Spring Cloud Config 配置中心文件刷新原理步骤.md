本文描述了在 git 仓库修改了配置之后，新的配置是如何刷新到各个微服务的步骤

> 前言：
>
> 1、假设现有有 3 个微服务，1 个是 配置中心，另外 2 个是普通微服务，服务名称分别如下：
>
> > 配置中心：confi-server
> >
> > 用户服务：user-service
> >
> > 商品服务：product-service

# 1. 第一次启动时

1、首先配置中心必须先启动，配置中心第一次启动直接从 git 拉取 master 分支上的配置保存到本地某个文件夹

2、用户服务 user-service 通过配置文件指定要哪个文件如：dev 分支的 application-dev.yml 文件，启动然后访问配置中心，但是配置中心只有 master 分支的配置没有 dev 分支的配置，随后配置中心访问 git 拉取 dev 分支的配置保存到本地，并返回给 user-service 服务

> 配置中心退出后会删除本地保存的配置文件

3、商品服务 product-service 同样按照 user-service 的方式（不过此时配置中心本地已经缓存了不需要访问 git 了）

此时的配置中心的角色类似于配置文件的保存的地方并返回配置给其它服务

> 疑问：假设有2个 user-service 服务，一个用dev分支，一个用master分支，这样是不是会引起配置中心 config-server 频繁切换去git上拉取配置呢？
>
> 答案：通过观察配置中心的本地文件，发现确实会发生切换，但是即使是100人的开发小组有的人切换到开发环境有的人切换到测试环境，切换也不会至于太频繁，这点姑且暂时可以忍耐！

# 2. 后续直接在 git 修改配置时

1、当 git 发生文件修改时，配置中心不会更新本地文件，没有任何作为，那么新修改的配置文件怎么达到各个服务中呢？

2、需要在各个服务中全部调用 `刷新配置` ，即调用如 `http://localhost:9001/actuator/refresh` ，调用该接口的意义就是间接告诉配置中心快点去 git 拉取新的配置然后保存到本地然后给我新的配置

> 每个服务都调用一下也是不小的工作量，特别是服务特别多的时候，不过这就是最原始的配置中心的缺点嘛，后面的新技术当然要修复了

# 3. 参考资料

自行调试配置中心的本地文件夹

> 1、在刚启动时会拉取 master 分支
>
> 2、在用户服务或商品服务启动时会把它们要求的配置给它们（如果没有就去git上拉取，哪怕是切换）
>
> 3、在调用刷新接口时会拉取新的配置给对应的服务
