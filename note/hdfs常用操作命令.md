[toc]





####  列出所有文件

```
hadoop fs - ls
```

或者 

```
hdfs dfs -ls
```



#### 进入某个目录

(hdfs中没有cd命令哦，而是要使用ls + 完整路径 进入某个目录中）

```
hdfs dfs -ls /user/username/app1/subdir/
```



#### 新建文件夹

```
hdfs dfs -mkdir <folder name>
```



#### 创建文件

```
hdfs dfs -touchz <file_path>
```



#### 上传文件

```
fs put <from source_path_and_file> <to dest_path_and_file>
```



#### 下载文件

```
fs get <from source_path_and_file> <to dest_path_and_file>
```



#### 查看文件夹里面每个文件的大小

**du:** It will give the size of each file in director

```
hdfs dfs -du <dirName>
```



#### 查看文件夹总大小

**dus:**: This command will give the total size of directory/file.

```
hdfs dfs -dus <dirName>
```


