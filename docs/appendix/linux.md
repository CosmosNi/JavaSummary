# linux指令大全

- 查找文件：find  / -name filename.txt
- 查看一个程序是否运行：ps -ef|grep tomcat
- 终止线程：kill -9 19979
- 查看文件，包括隐藏文件：ls -al
- 当前工作目录： pwd
- 复制文件包括子文件到指定目录：cp -r sourceFolder targetFolder
- 创建目录：mkdir newfolder
- 删除目录（空目录）：rmdir deleteEmptyFolder
- 删除文件包括子文件：rm -rf deleteFile
- 移动文件：mv /temp/movefile /targetFolder
- 切换用户：su -username
- 修改文件权限：chmod 777 file.java
  //file.java的权限-rwxrwxrwx，r表示读、w表示写、x表示可执行
- 压缩文件：tar -czf test.tar.gz /test1 /test2
- 列出压缩文件列表：tar -tzf test.tar.gz
- 解压文件：tar -xvzf test.tar.gz
- 查看文件头10行：head -n 10 example.txt
- 查看文件尾10行：tail -n 10 example.txt
- 查看日志文件：tail -f exmaple.log
- 查看系统当前时间：date
- 解压zip文件：unzip -oq
- 查看线程个数：ps -Lf 端口号|wc -l