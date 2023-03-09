## Ambari集成Flink

### 下载ambari-flink-service服务
```bash
# 在ambari-server节点获取版本号
VERSION=`hdp-select status hadoop-client | sed 's/hadoop-client - \([0-9]\.[0-9]\).*/\1/'`
# 下载ambari-flink-service服务到 ambari-server 资源目录下，如果无法访问外网可以下载下来传到目的目录
sudo git clone https://github.com/abajwa-hw/ambari-flink-service.git /var/lib/ambari-server/resources/stacks/HDP/$VERSION/services/FLINK
cd /var/lib/ambari-server/resources/stacks/HDP/$VERSION/services/FLINK/
```

### 修改配置文件

1. metainfo.xml 修改版本号
    ```xml
    <name>FLINK</name>
    <displayName>Flink</displayName>
    <comment>Apache Flink is a streaming ...</comment>
    <version>1.10.3</version>
    ```

2. configuration/flink-ambari-config.xml 指定Flink安装包地址
    将Flink安装包下载到服务器`flink_release_file`对应的地址，如果需要自动下载就配置`flink_download_url`
    ```xml
    <property>
        <name>flink_release_file</name>
        <value>/tmp/flink-1.14.6-bin-scala_2.11.tgz</value>
        <description>Decompress .tgz to flink_install_dir when setup_prebuilt is true</description>
    </property>

    <property>
        <name>flink_download_url</name>
        <value>https://archive.apache.org/dist/flink/flink-1.14.6/flink-1.14.6-bin-scala_2.11.tgz</value>
        <description>Snapshot download location. Downloaded when setup_prebuilt is true</description>
    </property>
    ```

### 创建用户
```bash
groupadd flink
useradd -d /home/flink -g flink flink
```

### 新增Flink服务
执行`ambari-server restart`命令重启ambari-server服务随后在web页面查看是否有Flink对应的服务
![Image](screenshots/hava-flink.png?raw=true)

然后添加服务并选择需要安装的服务器，随后修改关键配置信息
![Image](screenshots/edit-configuation.png?raw=true)

检查没问题之后开始安装
![Image](screenshots/deploy.png?raw=true)

检查安装状态
![Image](screenshots/in-progress.png?raw=true)
![Image](screenshots/install-sucessful.png?raw=true)

安装完成
![Image](screenshots/flink-status.png?raw=true)

Flink Dashboard，有一个问题是它的地址取决于启动Flink时使用的是哪个NodeManager的Container，这是随机的，所以快速链接有时并不管用。而且提交单作业的时候这里是监控不到的，在Yarn界面中的ApplicationMaster地址中可以看到作业对应的Dashboard
![Image](screenshots/flink-dashboard.png?raw=true)

安装完成后如需在服务器执行Flink命令还需执行下面的命令
```bash
# 3.1.0.0-78换成自己的版本目录
sudo ln -s /usr/hdp/3.1.0.0-78/flink/bin/flink /bin/flink
# 必须指定HADOOP_CLASSPATH到当前会话的环境变量，否则执行flink命令会报错
echo "export HADOOP_CLASSPATH=`hadoop classpath`" >> ~/.bashrc
```

### 测试
```bash
flink run -m yarn-cluster examples/batch/WordCount.jar
```
![Image](screenshots/flink-test.png?raw=true)