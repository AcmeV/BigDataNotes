# HDFS 基本操作
## 一、HDFS Shell 交互
### 1. 基本结构
```shell
hadoop fs [-command ]
hdfs dfs [-command]
```

### 2. 基本命令
```shell
hadoop fs -help # 帮助命令
hadoop fs -ls [-R] path # 递归的展示目录
hadoop fs -mkdir [-p] path # 创建目录（-p表示递归创建目录）
hadoop fs -rm [-r] file_path # 删除文件（-r删除文件夹）
```
### 3. 文件双传和下载
#### (1) 本地 -> HDFS
```shell
hadoop fs -put sourceFileOrDir targetDir #将sourceFileOrDir复制到HDFS的targetDir下
hadoop fs -moveFromLocalOrDir sourceFileOrDir targetDir #将sourceFileOrDir剪切到HDFS的targetDir下
hadoop fs -appendToFile sourceFileOrDir targetFile #将sourceFileOrDir追加到HDFS的targetFile尾部
```
#### (2) HDFS -> HDFS
```shell
hadoop fs -cp #mv、chown、chgrp、chmod、mkdir、du、df、cat、rm、du、df、tail和linux的命令用法类似
hadoop fs -setrep count FileOrDir #将HDFS上的FileOrDir设置count个副本
```
#### (3) HDFS -> 本地
```shell
hadoop fs -get sourceFileOrDir targetDir #将HDFS上的sourceFileOrDir下载到本地的targetDir
hadoop fs -getmerge sourceDir targetFile #将HDFS上的sourceDir里的所有文件合并到本地的targetFile
```

## 二、HDFS Java API
### 1. 准备工作
#### (1) 解压对应版本的hadoop jar并配置好环境变量
![hadoop环境变量配置](../pic/hadoop_env_path.png "hadoop env")
#### (2) 导入依赖
```xml
<dependencies>
    <dependency>
      <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-common</artifactId>
      <version>2.7.2</version>
    </dependency>
    
    <dependency>
      <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-client</artifactId>
      <version>2.7.2</version>
    </dependency>
    
    <dependency>
      <groupId>org.apache.hadoop</groupId>
      <artifactId>hadoop-hdfs</artifactId>
      <version>2.7.2</version>
    </dependency>
</dependencies>
```
### 2. Java API
```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

import java.io.*;
import java.net.URI;

public class HDFSClient{
    public void main(){
        /*
                准备工作
         */
        // 连接Hadoop集群
        URI uri = URI.create("hdfs://hadoop100:9000");
        Configuration configuration = new Configuration();
        // 设置配置信息：出现相同的配置项时，优先选择API中的配置，其次是hdfs-site.xml中的配置，再其次是服务器默认配置
        configuration.setInt("dfs.replication", 1);
        //创建FileSystem对象
        FileSystem fileSystem = FileSystem.get(uri, configuration, "root");

        /*
                HDFS JAVA API
         */
        // 创建文件夹
        fileSystem.mkdirs(new Path("/user/root"));
        
        // 从本地copy文件
        fileSystem.copyFromLocalFile(new Path("src/main/resources/mytest.txt"), new Path("/user/root/"));
        
        // 查看文件内容
        FSDataInputStream stream = fileSystem.open(new Path("/user/root/mytest.txt"));
        BufferedReader reader =
                new BufferedReader(new InputStreamReader(stream));
        String line = "";
        while ((line = reader.readLine()) != null)
            System.out.println(line);
        reader.close();
        
        // 在文件尾追加内容
        FSDataOutputStream append = fileSystem.append(new Path("/user/root/mytest.txt"));
        BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(append));
        writer.write("append string");
        writer.close();
        
        // 关闭连接
        fileSystem.close();
    }
}
```