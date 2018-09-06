## mangodb安装流程(Mac)
### 使用homebrew安装
####1.更新Homebrew
```
brew update
```
####2.安装mongoDB包
```
brew install mongodb
```

###把MongoDB跑起来
####1.创建data文件夹
mongod进程默认会在这个文件夹下进行操作

```
mkdir -p /data/db
```
实际操作的时候要切到mongodb对应版本的文件夹下，然后这个命令我没跑成功（就是文件夹没建起来），最后是手动创建的...
####2.为data文件夹设置读写权限
####3.将mongodb跑起来
**不指定路径**
如果环境变量PATH已经包含了mongod的路径直接敲
`mongod`

**指定mongod的路径**

PATH没有指定，就要手动输入mongod的全路径
```
<path to binary>/mongod
```

**没有使用默认的data路径**

用`--dbpath`选项指定自定义的data文件夹的路径
```
mongod --dbpath <path to data directory>
```

**实际使用的时候可能权限不够要加`sudo`**
####4.验证mongodb已经启动
查看命令行窗口有如下输出：
```
[initandlisten] waiting for connections on port 27017
```
####5.使用mongodb
使用`mongo --host`指定监听的域名和端口，如下例监听本地`127.0.0.1`地址和mongod默认端口`27017`
```
mongo --host 127.0.0.1:27017
```

如果想关闭数据库，在命令行键入`Ctrl+C`即可

更多详情查询[官网](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x)

******
  ***Enjoy Yourself!***

		




