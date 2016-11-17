# Round 1 介绍
   Hadoop-2.6.5搭建
  一.安装java
         安装java版本为1.8的。
     1.给java文件执行权限
         输入chomd +x ./jdk-8u111-linux-x64
     2.解压java文件
        tar zxvf ./jdk-8u111-linux-x64
     3.配置java环境
       vim /home/hadoop/.bashrc
       在最后添加java的路径：export JAVA_HOME=/home/hadoop/jdk-8u111
                           export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
                           export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
    4.resource ./.bashrc
  二.安装hadoop
    1.解压缩
      tar zxvf hadoop-2.6.5.bin.tar.gz
    2.在解压后的目录下新增tmp文件
      mkdir tmp   用来指定Hadoop运行时产生文件的目录
    3.配置hadoop
      使用vim修改slaves文件：我配置了一个master，两个slave
      使用vim修改core-site.xml
          <property>
            <name>fs.defaultFS</name>
            <value>hdfs://master:9000</value>
         </property>
         <property>
           <name>hadoop.tmp.dir</name>
           <value>file:/home/hadoop/hadoop-2.6.5/tmp</value>
         </property>
      使用vim修改hdfs-site.xml
          <property>
              <name>dfs.namenode.secondary.http-address</name>
              <value>master:50090</value>
         </property>
         <property>
              <name>dfs.replication</name>
              <value>2</value>
         </property>
        <property>
             <name>dfs.namenode.name.dir</name>
             <value>file:/home/hadoop/hadoop-2.6.5/tmp/dfs/name</value>
        </property>
        <property>
            <name>dfs.datanode.data.dir</name>
            <value>file:/home/hadoop/hadoop-2.6.5/tmp/dfs/data</value>
        </property>
       其中，<value>3</value>视情况修改，如果有三台slave机器，这里设置成3，如果只有1台或2台，修改成对应的值即可。换句话说，就是HDFS保存的副本的数        量。
      修改hadoop-env.sh
         新增export JAVA_HOME=/home/hadoop/jdk1.8.0_111
      修改yarn-site.xml
          第一个属性：Yarn的老大是resourcemanager的地址（master）
          第二个属性：告诉nodemanager获取数据的方式是shuffle方式
          <property>
            <name>yarn.resourcemanager.hostname</name>
            <value>master</value>
           </property>
           <property>
                 <name>yarn.nodemanager.aux-services</name>
                 <value>mapreduce_shuffle</value>
           </property>
           <property>
                <name>yarn.log-aggregation-enable</name>
                <value>true</value>
           </property>
        修改本地网络配置
          使用vim编辑/etc/hosts文件
          sudo v192.168.2.41 master
                192.168.2.40 slave1
                192.168.2.42 slave2im /etc/hosts
    三.建立互信关系  
       1.	生成公私钥
          在master机器的虚拟机命令行下输入ssh-keygen 
           一路回车，全默认
       2.	复制公钥
          复制一份master的公钥文件
          cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
          同样，在所有的slave机器上，也在命令行中输入
          ssh-keygen 
          一路回车，全默认
          所有的salve机器上
          从master机器上复制master的公钥文件
          scp master:~/.ssh/authorized_keys      /home/hadoop/.ssh/      
        3．	测试联接
            在master机器上
            分别向所有的slave机器发起联接请求
            如：
            ssh slave1
    四.启动hadoop
        1.	初始化
            在master机器上
            进入/home/hadoop/hadoop-2.6.5/bin目录
                cd /home/hadoop/hadoop-1.2.1/bin
            运行
                ./hadoop namenode –format 或者 hdfs namenode -format
            初始化hadoop的文件系统。
       2.	启动
            执行./start-all.sh
            中间过程 ，可能提示要判断是否,输入yes
            输入jps，查看进程是否都正常启动
       3.	测试系统
           输入./hadoop fs –ls /

    统计需求：
      1：完成每天的UV统计
         map部分用于数据过滤，用正则方式把每条访问记录中的ip和文件路径匹配出来，去重，在reduce里进行计数
      2：完成每天访问量Top10的Show统计
         map部分用正则方式把每条访问记录中的ip和文件路径匹配出来， 用hbase存储key文件/show...的路径，ip作为列族的值用空格拼接起来。reduce部分把各个key查询出来，把列族的拼接ip拆开放在Set里面，返回set的size，再把文件路径和ip数量拼接放在Arraylist中，用冒泡排序，取前10的/show...
      3:完成每天的次日留存统计
        把今天的去重ip保存在一个文件中，把次日的去重ip保存在一个文件中，先把今天文件中的ip读进一个set，再把次日文件中的ip往set里面读，每读一条判断set的size有没有改变，要是改变了，说明原来的set里面没有这个ip，要是没改变，说明有，则把这个ip保存在另一个文件中作为每天的次日留存统计。
      
