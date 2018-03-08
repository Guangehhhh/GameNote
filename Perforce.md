# 公司内网

# Perforce
- 学习
- Server
  - server代表perforce服务器的地址，也就是p4v客户端需要去连的这个地址
  - 安装p4v时会有一个默认的“localhost:1666”
- Depot
  - 代码仓库。在P4V里它表现是一个树型目录，代表着服务器上的文件及目录状态
  - 查看所有在版控服务器上的文件状态

- Workspace
  - 对应的版控文件列表
  - 它是基于当前主机的一个虚拟客户端，P4V是一个用于连接服务器和使用perforce的客户端工具

- 文件状态指示
  - 黄色三角
    - 表示该文件不是最新的
  - 右上角蓝色对号
    - 表示该文件被另一个用户检出(checkout)，鼠标悬浮可以查看用户信息
  - 左上角红色对号
    - 表示该文件正在被自己检出
- History
  - 查看文件修改历史
- Pending ChangeLists
  - 查看检出文件列表
- Submited Changelists
  -  查看历史提交信息
- Branch Mappings
  - 查看分支信息
- Labels
  - 查看标签信息


- Perforce 的最基本操作
  - 获取文件最新版本(Get Latest Revision)
  - 检出(chekcout)
  - 加锁(Lock)
  - 修改
  - 查看差异(Diff)
  - 丢弃修改(Revert)
  - 提交(submit)
  - 解决冲突(Resolve)
