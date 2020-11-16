#### jdk安装(linux)
```bash
# 1.上传安装包到目录：/usr/local/java，并解压: tar -zxvf jdk-8u231-linux-x64.tar.gz
# 2.修改JDK 环境变量
  vi /etc/profile
# 3.在文件空白处加入如下命令
  JAVA_HOME=/usr/local/java/jdk1.8.0_231
  JRE_HOME=/usr/local/java/jdk1.8.0_231/jre
  PATH=$PATH:$JAVA_HOME/bin
  export PATH JAVA_HOME JRE_HOME
# 4.使修改生效
  source /etc/profile
# 5.验证JDK 是否安装成功
  java -version
```
