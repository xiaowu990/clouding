# RUN的流程
![[image-20200610160609037.png]]


# 原理
Docker**是怎么工作的**？

Docker是一个Client-Server结构的系统，Docker的守护进程运行在主机上。通过Socket从客户端访问！

Docker-Server接收到Docker-Client的指令，就会执行这个命令！
![[image-20200610161147612.png]]