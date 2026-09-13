---
title: "后端面试高频重点（三）"
slug: "interview-key-3"
date: 2026-09-13T12:00:00+08:00
draft: false   # true=草稿，构建默认忽略
tags: ["数据库", "框架", "c++工具链", "软件工程", "Linux"]
categories: ["面试八股"]
summary: "本系列内容以面试和工作高频知识点为引扩展全貌，涵盖后端几乎所有常用知识点。"
toc: true
comments: true
description: "计算机面试高频重点（三）"
---

- 本部分内容为个人整理，以面试和工作高频知识点为引扩展全貌，涵盖后端几乎所有常用知识点，将根据面试进度推进不断补充
- ⭐⭐⭐⭐⭐：极高频考点，工作中也常用，必须掌握
- ⭐⭐⭐⭐：高频考点，必须掌握
- ⭐⭐⭐：工作中偶尔用，尽量掌握
- ⭐⭐：对面试意义不大，了解即可
# 数据库
- ⭐⭐⭐数据库常用概念：
	- 模式：结构元数据
	- 主键：表标识每一行的唯一字段，不NULL，不重复不更新
	- 外键：本表引用另一张表的主键
	- DML：insert/update/delete
	- 临时表：本连接优化器自动内存生成的临时表，子查询、视图
	- 快照读（select）：数据某一时刻的快照，MVCC
	- 当前读（select...for update/DML）：数据最新版本，锁
- ⭐⭐⭐数据库三大范式：
	- 1NF：列不可再分
	- 2NF：非主键完全依赖主键
	- 3NF：非主键直接依赖主键
- ⭐⭐⭐⭐⭐事务的ACID特性
	- A 原子性：全部成功或全部失败
	- C 一致性：业务数据状态合法
	- I 隔离性：多个事务之间互相隔离
	- D 持久性：commit 之后修改永久保存
- ⭐⭐⭐⭐⭐缓存穿透、击穿、雪崩？怎么解决？
	- 穿透：key未命中，查询数据库，拦截非法参数
	- 击穿：热点key过期，数据库背压，逻辑过期
	- 雪崩：大量key过期，数据库背压，过期时间加偏移，保证Redis高可用，限流
- ⭐⭐⭐⭐⭐MySQL引擎类型与区别：
	- InnoDB：支持事务、外键；行锁+表锁；.frm/.ibd；聚簇；磁盘；自增持久化；无行数
	- MyISAM：不支持事务、外键；表锁；.frm/.myd/.myi；非聚簇；磁盘；自增内存，有行数
	- Memory：不支持事务、外键；表锁；.frm；非B+；内存；自增内存，有行数
- ⭐⭐MySQL推荐字符集：utf8mb4_unicode_ci
- ⭐⭐建库建表避免报错：IF NOT EXISTS/IF EXISTS
- ⭐⭐数据类型选型：
	- 主键 id：BIGINT AUTO_INCREMENT
	- 状态标记：TINYINT UNSIGNED
	- 金额：DECIMAL (10,2)
	- 短文本、名称：VARCHAR (合理长度)
	- 长文本文章：TEXT
	- 时间戳：DATETIME DEFAULT CURRENT_TIMESTAMP
	- 更新时间：DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
- ⭐⭐⭐⭐char与varchar区别：
	- char(n)：定长
	- varchar(n)：变长，n只是逻辑限制
- ⭐⭐int与varchar区别：
	- int：整数
	- varchar：字符串
- ⭐⭐⭐⭐in与exists区别：
	- in：子查询为主，结果在外层匹配（适合大子查询）
	- exist：父查询为主，结果在内层匹配（适合小子查询）
- ⭐⭐⭐⭐INNER JOIN与LEFT JOIN区别：
	- INNER JOIN：只返回两边匹配上的数据
	- LEFT JOIN：左边表全部保留，右表匹配不到填 NULL，如果用WHERE会退化为INNER JOIN
	- 扩展：还有CROSS JOIN笛卡尔积和FULL JOIN匹配不上填NULL
- ⭐⭐⭐⭐ON、WHERE、HAVING 区别：
	- ON：表联结条件过滤
	- WHERE：结果行过滤（分组前）
	- HAVING：分组后行过滤
- ⭐⭐⭐⭐UNION、UNION ALL区别：
	- UNION：自动去重
	- UNION ALL：不去重
- ⭐⭐⭐⭐DROP、DELETE、TRUNCATE区别：
	- DELETE：删除行
	- DROP：删除表
	- TRUNCATE：清空表
- ⭐⭐⭐自增与UUID：
	- 自增：自带，空间小，有序，不安全，分布式不友好
	- UUID：应用层，空间大，无序，安全，分布式友好
- ⭐⭐count(\*)、count(1)、count(列名)的区别：
	- count(\*)：推荐，包含NULL
	- count(1)：同count(\*)
	- count(列名)：不包含NULL
- ⭐⭐⭐⭐索引类型：
	- 聚簇：主键索引
	- 非聚簇：普通、唯一（本字段不可重复）、复合、文本、哈希
- ⭐⭐⭐⭐⭐聚簇与非聚簇区别：
	- 聚簇：叶子节点有行数据
	- 非聚簇：叶子节点只有指针和本列值
- ⭐⭐⭐⭐⭐索引的数据结构？B+树的具体结构与优势？
	- B+树：m阶key数 \[ceil(m/2)-1,m-1\]；查询、插入（已满分裂）、删除（不足合并）
	- 优势：操作都是O(h)，h一般不大，性能高；磁盘IO少；范围查询强
- ⭐⭐⭐⭐B树与B+树区别：
	- B树：所有节点都有key和数据，全树都是数据，叶子间独立
	- B+树：非叶子只有key，数据节点只有叶子，叶子间有链表
- ⭐⭐⭐⭐⭐回表查询与索引覆盖：
	- 回表查询：非聚簇索引到主键然后根据主键到聚簇索引返回完整行
	- 索引覆盖：全部字段可以直接在非聚簇里拿到
- ⭐⭐⭐⭐⭐索引遵循的最左前缀匹配原则，失效情况？
	- 从左到右依次匹配，遇到范围停止，运算、函数、类型转换、or、<>会失效
- ⭐⭐⭐视图概念？
	- SELECT 查询语句，虚拟临时表，重用sql语句，只用于检索
- ⭐⭐⭐触发器概念？
	- 绑定在表上的特殊存储过程（回调函数），发生DML时自动触发
- ⭐⭐⭐⭐⭐一致性问题？
	- 脏读：读到别人还没提交的数据
	- 不可重复读：重复读时中间被修改
	- 幻读：重复读时中间被增删
- ⭐⭐⭐⭐⭐事务隔离级别？
	- RU：啥保证没有
	- RC：禁止脏读；MVCC（每次select生成快照）；行锁
	- RR：禁止脏读、不可重复读、幻读；MVCC（第一次select生成快照）；临键锁
	- S：禁止脏读、不可重复读、幻读；全部变成当前读；临键锁
- ⭐⭐⭐⭐⭐MVCC介绍？
	- 行字段：最近修改事务ID、回滚指针、隐藏主键
	- undo log：版本链，purge异步清理
	- 快照：当前未提交事务集、本事务ID
	- 可见性判断：行事务ID小于集合或不在集合才可见，不可见找上个版本
	- 脏读：行事务ID在集合内
	- 不可重复读：行事务ID在集合内（如果第二次select生成的话那就不对了，就不在集合内了）
	- MVCC不能阻止幻读！但是因为操作的是旧版本的快照没影响
- ⭐⭐⭐⭐⭐行锁与表锁？
	- 表锁：表级操作
	- 意向锁：准备加行锁就先加意向锁防止表锁，同类不冲突
	- 行锁：行级操作，读写锁，支持共享和排他
	- 临键锁：锁住间隙和当前行，防止幻读
- 乐观锁与悲观锁？
	- 乐观锁：版本号没变才更新否则重试
	- 悲观锁：当前读锁（临键锁）
- ⭐⭐⭐⭐⭐MySQL主从同步实现？
	- binlog
	- 主库dump线程发送，从库IO线程接收写入relay-log，sql线程读取重放
	- 异步复制主库发完就返回，半同步至少等待一个从库结果才返回
- ⭐⭐⭐⭐日志类型？
	- error_log：错误日志
	- slow_query_log：过慢查询
	- binlog：事务日志
	- redo log：物理页修改，启动时自动重放恢复
	- undo log：MVCC基础，purge线程回收
- ⭐⭐⭐百万级别数据库内数据的删除过程：逻辑删除
	- 命中行加行锁
	- B + 树标记删除
	- 写 Undo Log
	- 写 Redo Log
	- 事务提交
	- Purge 线程异步清理UNDO日志
	- 空间回收
- ⭐⭐⭐⭐⭐SQL查询慢的原因？数据库有哪些优化方法？
	- 索引级：多建，索引覆盖，WHERE优化为索引友好，少用order by / group by
	- 语句级：大查询分解，不用\*，避免大量join（主表选对），少用子查询
	- 事务锁级：避免长事务，少用当前读，降低隔离级别
	- 设计级：字段拆分，大文本拆分，避免类型转换
	- 业务级：合理参数，冷热分离，主从分离（主写从读），业务分库分表，水平分表（哈希分片，分布式主键、排序、事务问题）
- ⭐⭐⭐⭐⭐Redis双写一致性？更新数据时保证DB和缓存一致，不存在强一致
	- 先更新DB再更新缓存，永久不一致
	- 先更新缓存再更新DB，永久不一致
	- 先更新DB再删除缓存（Cache Aside，推荐）：大概率可以解决，也有可能出现问题
		- 延迟双删：更新DB-删除缓存-延时100ms-删除缓存
	- 先删缓存再更新DB：永久不一致
- ⭐⭐Redis快速原因？
	- 高性能数据结构
	- 内存操作
	- 无锁、非阻塞、IO多路复用
- ⭐⭐Redis对象类型？底层结构？场景？
	- OBJ_STRING：SDS字符串
	- OBJ_LIST：链表
	- OBJ_HASH：哈希表
	- OBJ_SET（无序集合）：哈希表
	- OBJ_ZSET（有序集合）：跳表
- ⭐⭐⭐⭐过期删除和内存淘汰：
	- expires过期字典：key，value是时间戳
	- 惰性删除：访问key的时候检查是否过期，过期删除
	- 定期删除：没隔一段时间，抽取一部分 expires 里的 过期key删除
	- 内存淘汰：分配内存满了，直接淘汰部分key（和有没有过期无关），LRU、LFU、random
- ⭐⭐⭐⭐⭐两种持久化方式区别？
	- RDB：全量数据二进制快照，BGSAVE，文件小
	- AOF（优先）：记录每条写命令，落盘策略：always、everysec、no（操作系统管理），文件大
	- AOF重写：重新生成当前情况最简命令
	- 两种持久化不能同时进行
- ⭐⭐⭐⭐⭐主从复制介绍？
	- 主写从读，从是主的客户端，异步转发写命令给从
	- 复制缓冲区：RDB生成期间的新命令
	- 全量同步：主发送RDB给从，再发送复制缓冲区命令
	- 增量同步：不发送RDB直接补发offset后复制缓冲区的命令
	- 从会定时发送心跳检测确定主存活
	- 缺少故障转移机制，需要手动把从提升为主
- ⭐⭐⭐⭐⭐哨兵介绍？
	- 特殊模式运行的Redis进程，不存储业务数据
	- 监控主从存活，事件广播，负责故障转移
	- 主观下线：某一个哨兵发现心跳超时；客观下线：广播询问，超半数哨兵的回答都是下线
	- 故障转移：投票选leader哨兵，过滤从节点，升级主节点，旧主恢复变从，广播新主
- ⭐⭐⭐脑裂问题？
	- 网络分区，客户端能写入旧主，但是旧主已经和整个集群隔离，出现了新主
	- 解决：主检测可用从数量小于 N拒绝接收写
- ⭐⭐⭐⭐⭐集群介绍？
	- 哈希槽，所有key映射到槽，主分担所有槽，从备份，主写，从读
	- 只能用0号数据库，不在槽会MOVED重定向
	- 槽迁移：在线迁移，以槽为最小单位迁移，如果中途查询会返回ASK 重定向
- ⭐⭐事务和LUA脚本介绍？
	- MULTI/EXEC：事务，一次性执行；**不支持回滚**
	- WATCH：乐观锁，监控 key，失败则放弃
	- Lua 脚本：**原子执行，单线程程序，执行期间其他命令阻塞**
# 框架
- ⭐⭐⭐⭐nginx有哪些功能？
	- 安全保护：虚拟主机、反向代理、HTTPS、防盗链、访问控制
	- 均衡负载：动静分离
	- 主备切换（心跳保活）
	- 数据压缩
- ⭐⭐⭐⭐⭐nginx负载均衡有哪些策略？
	- 默认轮询
	- weight加权轮询
	- ip_hash长连接保证
	- least_conn给连接数最少的
- ⭐⭐protobuf优点？
	- 编译期检查
	- 二进制体积小
	- 支持Arena内存池序列化速度快
	- 严格遵循开闭原则
- ⭐⭐⭐protobuf由哪些部分组成？
	- protoc编译器
	- libprotobuf库
- ⭐⭐⭐⭐gRPC概念？
	- 远程过程调用，**客户端像调用本地函数一样调用另一台服务器上的方法**，底层通信自动管理
- ⭐⭐⭐⭐⭐gRPC组成？
	- Protobuf（grpc_cpp_plugin插件）：消息体、客户端存根服务端服务代码
	- 客户端stub调用、服务器继承服务并实现
	- HTTP2
- ⭐⭐⭐⭐⭐gRPC通信模式有哪几类？
	- 单收发
	- 单向流
	- 双向流
- ⭐⭐⭐gRPC的功能？
	- 超时、取消
	- 元数据配置
	- 拦截器：日志、监控、鉴权、追踪
	- 也有负载均衡、安全之类的功能
# 工具链 & 软件工程
- ⭐⭐传统软件工程有哪些阶段？可行性研究-需求分析-概要设计-详细设计-编码-测试-维护
- ⭐⭐⭐介绍敏捷开发Scrum？
	- 2-4周时间盒迭代
	- 全部需求池和当前需求池
	- 看板和燃尽图
- ⭐⭐⭐⭐持续集成Devops有哪些阶段？
	- 代码管理-CI-代码安全-容器-配置服务器-产品仓库-CD-监控告警
- ⭐⭐⭐⭐介绍ER图
	- 数据库模型，实体、属性、联系
	- 1对1、1对多、多对多
![软件工程\|388](Attachments/软件工程%2014.png)
- ⭐⭐⭐⭐⭐介绍UML类图
	- 类名、属性、方法（+、-、\#、下划线static、纯虚斜体、虚函数virtual、接口`<<interface>>`）
	- 关系：继承（is a，空心三角实线）、实现（is implement of，空心三角虚线）、关联（has a，普通实线或空心菱形实线）、组合（has a，生命周期跟随使用者，实心菱形实线）、使用（use a，临时用到不是成员，普通虚线）
![471](../interview-key-3/4e613323574ce423a987db649380ff2af183b9c7.png)
- ⭐⭐⭐⭐介绍UML时序图
	- 角色、对象、生命线、执行条、消息发送接收
	- 发送实线，接收虚线、同步异步
![478](../interview-key-3/dbf6a183d4dce885a82f99fa1829d7240013e8a4.png)
- ⭐⭐⭐vim常用命令
	- 插入模式：`i`
	- 命令模式：`esc`
	- 保存退出：`:wq`
	- 强制退出：`:q!`
	- 行首：`0`
	- 行尾：`$`
	- 删除整行：`dd`
	- 向上翻整页：`ctrl+b`
	- 向下翻半页：`ctrl+d`
	- 查找：`/xxx`，`n`下一个，`N`上一个
- ⭐⭐⭐⭐⭐gcc或者clang构建的全命令
	- `g++/clang [源文件/静态库] [编译参数] [-D宏名称] [-I库头文件] [-L库.o文件] [-l链接库名] -o [可执行文件]`
	- 参数：`-Wall `、`-Werror`、`-g`、`-O0`
	- 动态库参数：`-shared -fpic`
	- 动态库在系统路径`/usr/lib`找或设置rpath
- ⭐⭐⭐⭐clang与gcc的区别
	- 报错提示比gcc更友好
	- 有静态语法检查：`--analyze`
- ⭐⭐⭐⭐⭐cmake构建的全命令：用作用域管理依赖关系
	- target_compile_options
	- target_include_directories
	- target_link_directories是链接阶段，rpath是运行阶段，rpath才是真正生效的那个
	- target_link_libraries
``` cmake
cmake_minimum_required(VERSION 3.16)
project(demo LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Debug CACHE STRING "" FORCE)
endif()

add_executable(my_app main.cpp)

# 直接引入外部静态库 .a 文件
target_sources(my_app PRIVATE "/opt/lib/mystaticlib.a")

# 编译警告选项（仅GCC/Clang生效）
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(my_app PRIVATE -g -O0 -Wall -Wextra)
endif()

# -I 头文件路径
target_include_directories(my_app PRIVATE "/usr/local/include")

# -L 库搜索路径（现代CMake尽量少用，这里对应原g++命令）
target_link_directories(my_app PRIVATE "/usr/local/lib")

# -lxxx 链接库
target_link_libraries(my_app
    PRIVATE
        grpc++
        grpc
        gpr
        protobuf
        mysqlclient
        pthread
)
```
- ⭐⭐⭐cmake编译指令
``` bash
#生成makefile
cmake -B build -DCMAKE_BUILD_TYPE=Debug -DCMAKE_PREFIX_PATH=/opt/grpc
#用make编译
cmake --build build
#运行程序
./build/my_app
# 测试
ctest --test-dir build --output-on-failure
# 安装
cmake --install build --prefix "${PWD}/install"
```
- ⭐⭐⭐⭐⭐cmake依赖配置方法
	- find_package（优先）：config模式，全部自动生成，自动生成目标直接link
	- pkg_check_modules：三方库pkg‑config，读取`.pc` 文件，最新版`IMPORTED_TARGET`都能自动生成，自动生成目标直接link
	- FetchContent：网上下载，`FetchContent_Declare`，`FetchContent_MakeAvailable(protobuf)`，全部自动生成，自动生成目标直接link
	- `REQUIRED`：找不到直接报错终止 cmake，不继续构建
	- rpath配置：构建时一般不用手写，安装时配置如下
``` cmake
set_target_properties(my_app PROPERTIES
    BUILD_RPATH_USE_ORIGIN ON
    INSTALL_RPATH "$ORIGIN/../lib"
)
```
- ⭐⭐cmake自定义命令执行：只有target发生变动才运行`add_custom_command`
- ⭐⭐怎么配置ctest？
	- `enable_testing()`
	- `add_test(NAME math_test COMMAND test_math)`
	- `ctest -j4`
- ⭐⭐⭐valgrind用来做什么的？检查内存泄露`valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes ./app`
- ⭐⭐⭐⭐⭐有哪些调试方法？
	- gdb断点调试
	- 打印调试
	- 日志文件
	- coredump
	- 静态语法检测（clang）
	- valgrind
	- 远程调试
- ⭐⭐⭐⭐⭐make的主要结构
``` make
.PHONY 每个目标
目标: 依赖文件列表（空格分割）
	命令（开头必须是【Tab】，空格会报错！）
```
- ⭐⭐doctest的基本使用：
	- `#define DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN #定义这个宏后框架自动生成main`
	- `#include <doctest/doctest.h>`
	- `TEST_CASE("基础加法测试") {CHECK_FALSE(1 + 1 == 3);}`
	- 可执行文件注册到ctest
- ⭐⭐curl的基本使用：
	- GET：`curl -i http://127.0.0.1:8080/hello`
	- POST：`curl -X POST --json`
	- 自带header：`curl -H`
	- 发送cookie：`curl -b`
- ⭐⭐⭐⭐⭐git常用命令
	- `git init`
	- `git config --global user.name "你的名字"`
	- `git config --global user.email "你的邮箱"`
	- `git remote add origin https://github.com/xxx/demo.git`
	- `git clone https://github.com/xxx/demo.git my-project`
	- `git pull origin main`
	- `git switch -c feature/login`
	- `git add .`
	- `git commit -m "feat: 登录功能"`
	- `git push -u origin feature/login`
	- `git branch -d feature/login`
- ⭐⭐⭐⭐怎么避免不想提交的：.gitignore
- ⭐⭐⭐⭐怎么查看提交记录：`git log --oneline`
- ⭐⭐⭐⭐怎么撤回：工作区`git restore`/本地库`git reset/`远程库`git revert`
- ⭐⭐⭐⭐⭐怎么合并？冲突怎么解决？
	- merge：保留原有分支，分支分叉
	- rebase：只有当前分支，单一分支，在最新版本上重放之前的提交
	- 冲突：手动修改重新提交
- ⭐⭐⭐怎么设置版本标签？`git tag v1.0.0 git push origin --tags`
- ⭐⭐⭐git actions配置文件在哪：`.github/workflows/xxx.yml`
- ⭐⭐clang-format基本使用：
	- 配置文件：项目根下`.clang-format`
``` bash
#一键格式化
clang-format -i src/**/*.cpp src/**/*.h
#检查是否合规
clang-format --dry-run -Werror <file>
```
- ⭐⭐⭐⭐⭐gdb常用命令
	- 要`-g -O0`关闭编译器优化
	- 运行：`r`
	- 设置断点：`b 行号或函数名`
	- 单步跳过：`n`
	- 单步进入：`s`
	- 运行：`c`
- ⭐⭐⭐⭐⭐gdb运行时调试
	- 找到pid
	- 挂载进程：`gdb -p <pid>`
	- 调用栈：`bt`
	- 打印线程：`info threads`
	- 实时监视：`watch 变量名`
- ⭐⭐License有什么注意点？MIT这种没有传染性，GPL有传染性
# Linux
- ⭐⭐⭐⭐⭐常用基础命令有哪些？
	- 文件操作：cd、ls、pwd、mkdir、rm -rf、mv、touch、cat、less、tail -f
	- 查找：find /home -name "hh"
	- 管道过滤：`| grep `
	- 压缩解压：压缩`tar -zcvf`、解压`tar -zxvf`
	- 进程：`ps aux`、kill -9、top
	- 网络：ping、ifconfig、nc、`netstat -tulnp`
	- 权限修改：`chmod 754 test.txt`
	- 服务管理：`systemctl status/start/stop/restart/enable/disable servicename`
	- 刷新配置：source
	- 挂载磁盘：`mount /dev/sdb1 /mnt/data`
	- 防火墙：`firewall-cmd --list-all`
	- 下载包：yum list/install/remove
	- 日志：rsyslog服务
- ⭐⭐⭐⭐⭐有哪些常用目录？
	- /home、/bin、/usr/bin、/opt、/etc、/var/log、/usr/local
- ⭐⭐⭐⭐⭐etc有哪些配置？
	- `/etc/passwd`（用户信息）、`/etc/shadow`、`/etc/group`、`/etc/gshadow`
- ⭐⭐⭐⭐权限管理介绍
	- 所有者｜所属组｜其他
	- 读｜写｜执行
	- 可以十六进制转十进制数字表示
- ⭐⭐⭐⭐⭐定时任务介绍
	- 系统：`/etc/crontab`
	- 用户：`/var/spool/cron/用户名`
	- 查看：`crontab -l`
	- 文件格式：\*、\,、\-、\/n，分 时 日 月 周
- ⭐⭐⭐⭐系统的环境变量在哪？
	- 系统：/etc/profile
	- 用户：~/.bashrc
	- export将当前变量导出为环境变量，子进程可读取，关了shell就没了