
# 介绍


JumpServer 是广受欢迎的开源堡垒机，是符合 4A 规范的专业运维安全审计系统。JumpServer 帮助企业以更安全的方式管控和登录所有类型的资产，实现事前授权、事中监察、事后审计，满足等保合规要求。


 
# 产品特色


* 开源：零门槛，线上快速获取和安装；
* 分布式：轻松支持大规模并发访问；
* 无插件：仅需浏览器，极致的 Web Terminal 使用体验；
* 多云支持：一套系统，同时管理不同云上面的资产；
* 云端存储：审计录像云端存储，永不丢失；
* 多租户：一套系统，多个子公司和部门同时使用；
* 多应用支持：数据库，Windows 远程应用，Kubernetes。


 


# 背景


* 我们公司的windows资产，以前都是让大家连接8年前的一个商用的跳板机奇治。老旧，数据混乱
* 并且存在一个1\.4\.10的jumpserver。然后权限也非常的混乱。
* 大家连接数据库查询数据，都是用一台共用的一台windows，然后从这台windows用数据库客户端链接。并且每一个人还拥有很多的数据库账号。数据库侧 那边开通了各种账号。
* 反正就是特别的混乱
* 并且没有任何的审计，然后上次出现了一个事故，线上的数据被删除清空了。最后是从备份恢复了数据。但是也缺少了点数据。幸好数据不是特别的重要
* 之后领导就想整理这一块的东西


 


## 计划


* 重新搭建新版jumpserver，替换老的jumpserver，以及商用的奇治跳板机
* 以后windows linux资产都要通过新版的jumpserver登陆
* 数据库需要从jumpserver登陆，权限细分到单库上面


 


# 设计


* 无非就是在新版的jumpserver给用户新建各种服务器以及数据库的权限
* 并且数据库，需要去数据库里面新建各种人的账号，但是这样的话跟之前一样了。不方便管理
* 所谓有了规矩，规则的话。那么整个事情就变的 好维护，以及好管理了


 


## 服务器权限设计


* 服务器权限的话 就分两种，app和root。
* 一个普通用户的一个超级用户的
* 每一个用户从jumpserver创建一个 资产授权。后期这个用户也好管理


## 数据库权限设计


* 我设计的是给数据库那边新建一个根据用户名的账号，后面我们用这个账号给授权不同的权限。
* 然后我们给每一个人创建一个 资产授权。方便后期管理。
* 那么就会表现成 数据库有一个jump\_db\_fanlichun\_r的数据库账号
* 然后给这个jump\_db\_fanlichun\_r用户授权，比如查询，修改，等等
* 最后我们给这个用户创建一个 资产授权。名称也是jump\_db\_fanlichun\_r。


 


# 问题反思


* 如果都是人工操作的话，不免麻烦，服务器还好。但是数据库就比较麻烦，因为涉及到给数据库账号授权。那么你就得登陆数据库，然后授权。
* 那么能不能直接弄一个自动化的形式，不需要人工干预。
* 我在网上搜了，jumpserver确实有工单系统。但是这个是企业版的，我就是一个小打小闹的，企业版的要1年小几万。那肯定是不可能的，除非你们公司是大公司，对于jumpserver工单系统比较依赖
* 而且很多公司 都是自己的工单系统
* 最后我就准备自己写一个工单管理系统


 


# 展示


![](https://img2024.cnblogs.com/blog/1257808/202412/1257808-20241209170515430-488325002.png)


![](https://img2024.cnblogs.com/blog/1257808/202412/1257808-20241209170915644-10929903.png)


![](https://img2024.cnblogs.com/blog/1257808/202412/1257808-20241209170930006-208792588.png)


 


![](https://img2024.cnblogs.com/blog/1257808/202412/1257808-20241209171023901-763960410.png)


 


# 开源代码介绍


1. 是在jumpserver(v3\.10\.9\)上面二次开发出了一个简单的工单申请
2. 用户可以申请服务器和mysql库的权限
3. mysql 权限可以细分到库表
4. 自动创建授权，不用人为干预

 


# 软件架构



jumpserver core代码
jumpserver lina代码
1. 我这个是基于jumpserver v3\.10\.9 开发的。
2. 大家可以试试别的版本，基本上只涉及到几个接口
3. 只要这几个接口不变，我这个简单版的工单申请 就可以运行的
4. 只需要在你的版本代码上面 新增几个接口就行
5. 然后前端新增页面就行



 


# 代码接口介绍




```
# 我新增的接口主要有以下几个

# 都是在这个文件里面
jumpserver-ticket/jumpserver-v3.10.9/apps/perms/urls/user_permission.py

# 具体接口


# 自己新增的 工单申请
# 这个接口是获取全部的资产的 
path('mytickets/getassets/', api.mytickets.get_all_node),

# 这个是创建申请服务器的工单接口
path('mytickets/apply/', api.perm_apply.perm_application),

# 这个是创建申请mysql数据库权限的接口
path('mytickets/applydb/', api.perm_apply.perm_application_db),

# 这个是查看自己的工单申请接口
path('mytickets/myapplication/', api.mytickets.my_application),

# 这个是管理员查看并审批 用户申请的接口
path('mytickets/myapproval/', api.mytickets.my_myapproval),

# 对接工单系统

# 这两个接口是因为 我们本身就有工单系统平台。
# 然后我给我们的开发专门写的两个接口

# 这个是创建服务器权限的接口
path('mytickets/createauthnodes/', api.mytickets.create_auth_nodes),
# 这个是创建mysql数据库权限的接口
path('mytickets/createauthmysql/', api.mytickets.create_auth_mysqls),
```


 


# 安装教程



安装的话 就按照官网的安装就行
大概步骤的话：


1. 从官网下载代码，修改代码（下载我的代码）
2. 先编译core代码，docker build \-f Dockerfile\-ce \-t jumpserver/mycore\-ce\-v3\-2:v3\.10\.9 .    最终会编译出一个镜像
3. 编译lina前端代码，yarn build。 最终会编译出一个lina目录


 


部署core：


[?](https://github.com):[milou云加速器官网](https://jiechuangmoxing.com)

| 123456789101112 | `# 进入docker compose文件夹``cd` `/opt/jumpserver-installer-v3``.10.8``/compose` `# 批量替换core镜像``sed` `-i` `"s/mycore-ce-v3-1/mycore-ce-v3-2/g"` `*``cd` `..` `# 停止服务``.``/jmsctl``.sh stop` `# 启动服务``.``/jmsctl``.sh start` |
| --- | --- |



 


部署lina:


[?](https://github.com)

| 1234567891011121314 | `# 因为前端打包出来的是一个lina文件夹``# 所以你可以再写一个dockerfile，以原始的lina镜像为基础，把这个文件给编译进去` `# 我采用了一种比较最简单的方式 直接把文件复制进去，然后重启nginx``# 但是缺点就是容器重启了 那么你修改的lina代码就没有了，得再复制一遍``# 方式有很多种，看你采用哪种都行` `docker` `cp` `lina  7309df137aff:``/tmp/lina``docker` `exec` `-it 7309df137aff` `bash``rm` `-rf lina``mv` `/tmp/lina` `.` `nginx -t``nginx -s reload` |
| --- | --- |



 




这样就部署好了
 


# 修改配置


1. 下载我的代码
2. 修改连接数据库的账号密码，你得有数据库账号，并且权限是。或者你用root账号也行
3. 你在全局文件里搜 db\_user 的行，然后把用户名密码 改成你有权限的用户
4. 然后需要修改调用你本身jumpserver接口的 token。因为我没有完全去读和jumpserver的 代码，所以我这就是取巧。直接在代码里面调用接口全局文件搜索 admin\_token 的行。然后把token替换成的你token。
5. 具体怎么获取token，查看jumpserver官方文档，要选择private\_token的方式。[jumpserver官方文档创建token](https://github.com)
6. 然后再按照我上面的安装步骤 就可以了

 


# 开源地址


[jumpserver\-Ticket](https://github.com)


https://gitee.com/ccsang/jumpserver\-ticket 


# 结语



如果有大佬感兴趣的，咱们可以一起交流。
或者有什么问题的，请留言。
就是平时写的一个小功能，愿对大家有用。
一起进步，一起成长！

 


