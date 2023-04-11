### 为什么要用Docker
* 更高效地利用系统资源
* 更快速的启动时间
* 一致的运行环境
* 持续交付和部署
* 更轻松地迁移
* 更轻松地维护和扩展

### 容器标准
* OCI： Open Container Initiative
* OCI主要定义两个规范
* Runtime Specification: 文件系统包如何解压至硬盘，供运行时运行
* Image Specification：如何通过构建系统打包、生成镜像清单、文件系统序列化、镜像配置

### 容器主要特征
* 安全性
* 隔离性
* 便携性
* 可配额

### Linux Namespace
* PID namespace
* Net namespace:
* ipc namespace: interprocess communication
* Mnt namespace
* Its namespace
* User namespace

### Cgroups
Control Groups 是对一个或一组进程进行资源控制和监控的机制

### 文件系统
* Union FS, 解决了文件分发问题
* 写操作：由于镜像具有共享特征，所以对容器可写层的操作需要依赖存储驱动提供的写时复制和用时分配的机制，以此来支持对容器可写层的修改，进而提高对存储和内容的资源的利用率
	* 写时复制
	* 用时分配
* 容器存储驱动：Overlay2, Overlay 只有两层： upper层和lower层

### 多进程的容器镜像
选择适当的init进程：需要捕获SIGTERM信号并完成子进程的优雅终止；负责清理退出的子进程以避免僵尸进程

