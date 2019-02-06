# PyBackup
> 自动备份站点目录和数据库


可以备份本地站点及数据库(mysql,mssql)，和远程FTP站点及远程数据库(mysql,mssql),

同步备份文件到Ftp,阿里云Oss,腾讯云Cos,OneDrive,Email,还有本地，并删除指定天数的旧备份文件


### 需要的环境
```
python3以及python3中的库oss2和cos-python-sdk-v5
windows上需要7z.exe，mysqldump.exe,SQLCMD.exe,SQLCMD.rll 在本程序已中自带有无需下载
linux上需要安装p7zip(确保7za命令可以执行),mysqldump是安装mysql时附带的,sqlcmd这个是mssql的命令行工具,linux上也有https://docs.microsoft.com/zh-cn/sql/linux/sql-server-linux-setup-tools?view=sql-server-2017 暂时不整理 
如果以下一键包安装出现问题，在安装python3成功后自行安装依赖库
pip3 install --upgrade pip
pip3 install oss2 cos-python-sdk-v5
注：如果linux下也想备份远程mssql(SqlServer)的话自行参考https://docs.microsoft.com/zh-cn/sql/linux/sql-server-linux-setup-tools?view=sql-server-2017安装sqlcmd工具
```
### 安装
Linux一键配置环境

```sh
wget --no-check-certificate https://raw.githubusercontent.com/LoneKingCode/PyBackup/master/PyBackup/backup/plugin/init.sh -O pybackup.sh && bash pybackup.sh
执行完本程序存放在脚本同目录
安装完输入python3 -h 和pip3 -h  还有7za 查看结果 判断是否安装成功
然后自行设置计划任务 使用crontab或者面板内自带计划
本程序目录/plugin/cron.sh为使用crontab设置计划任务 自行修改里面路径后执行sh cron.sh 即可
backup.py为主程序 设置的命令为 python3 yourPath/backup.py
```


Windows

```sh
首先下载安装python3.6.3 https://www.python.org/ftp/python/3.6.3/python-3.6.3.exe
安装完命令提示符中输入python3 -h 和pip3 -h 命令看结果 判断是否安装成功
然后下载本程序并解压 https://github.com/LoneKingCode/PyBackup/releases
然后修改本程序目录中/backup/plugin/init.bat 批处理文件中这一行命令
schtasks /create /tn "backup_web_db" /ru system /tr "python3 /yourPath/backup.py" /sc DAILY /st 01:00
修改backup_web_db为计划任务名称,修改yourPath为文件实际位置,DATLY为每日,01:00 凌晨一点执行
想计划为其他周期自行百度schtasks或者windows添加计划任务
修改完成保存后 双击 init.bat运行即可
init.bat作用是安装python3依赖库及设置计划任务
```

## 使用方法
提前说下:
```
配置文件中[]为数组，内部元素用,来隔开
{'key':'value','key':'value'}为字典 
所有[{}] 这种结构的都可以设置为[{},{},{},]多个配置
```
修改config.py中配置
```python

#备份的站点目录
#type: local,ftp local(本地),ftp(远程站点备份)
#path: 要备份的目录  windows下记得是双斜杠\\ linux下用/就行 比如/home/wwwroot/myweb
#archive_type: 7z,zip,tar(7zip支持的类型)
#archive_password: 压缩包密码
#type为ftp时需要额外配置: host:FTP服务器地址 port:端口 username:用户名 password:密码
SITES = [{ 'type':'local','path':'D:\\wwwroot\\www.aa.com',
         'archive_type':'zip','archive_password':'123' },
         {  'type':'ftp', 'path':'/WEB',  'archive_type':'zip','archive_password':'123',
          'host':'yourFtpUrl','port':21,'username':'yourFtpUsername','password':'yourFtpPwd'}]

#备份的数据库
#type: 数据库类型 mssql,mysql
#database_name: 数据库名称
#username: 数据库用户名
#password: 数据库密码
#host: 数据库服务器地址 为空代表本地
#archive_type: 7z,zip,tar(7zip支持的类型)
#archive_password: 压缩包密码
#sqlcmd_path 或 mysqldump_path : mssql和mysql命令行工具路径 windows下无需修改 linux请设置为''
#***注意***
#设置备份远程数据库时 请在数据库中设置为备份机IP可连接或任意IP可连接(mysql添加用户且设置Host为%或指定IP)
#mysql
#linux如果mysqldump命令可以直接执行的话mysqldump_path设置为'',否则设置为实际路径,windows下设置mysqldump.exe位置,本程序plugin目录自带有(MYSQL5.7)
#mssql(Sql Server)
#linux如果SQLCMD命令可以直接执行的话sqlcmd_path设置为'',windows下设置SQLCMD.exe位置,本程序plugin目录自带有(SQLSERVER2014)
#命令行工具有需要可在plugin目录替换为你需要的版本
DATABASES = [{'type':'mssql','database_name':'yourDatabaseName','username':'sa','password':'123','host':'123.123.123.123',
              'archive_type':'zip','archive_password':'123','sqlcmd_path':os.path.join(str(ROOT_DIR),'plugin') + '\\SQLCMD.exe'},
            {'type':'mysql','database_name':'yourDatabaseName','username':'test','password':'asd','host':'aa.aa.com',
            'archive_type':'zip','archive_password':'123','mysqldump_path': os.path.join(str(ROOT_DIR),'plugin') + '\\mysqldump.exe'}]

#保留几天内的文件
KEEP_DAYS = 7

#本地备份文件保存位置
LOCAL_SAVE_PATH = {'sites':'D:\\Save\\backup\\sites','databases':'D:\\Save\\backup\\databases'}

#备份文件临时保存位置
TEMP_SAVE_PATH = 'D:\\Save\\backuptemp'

#远程备份类型 为空则只保存到本地
#可选 ftp,email,oss,cos,onedrive
#***注意***
#email的话注意附件大小 有的限制是25MB 有的是50MB
REMOTE_SAVE_TYPE = ['oss','cos','ftp','onedrive']

#FTP备份配置
#host: FTP服务器地址
#port: FTP端口
#username: FTP用户名
#password: FTP密码
#site_save_path: 站点备份文件保存的路径 确保路径已经存在
#db_save_path: 数据库备份文件保存的路径 确保路径已经存在
FTP_OPTIONS = [{'host':'yourFtpUrl','port':21,'username':'yourFtpUsername','password':'yourPwd',
                'site_save_path':'/backup/sites','db_save_path':'/backup/databases'},]

#阿里云OSS配置
#sitedir: 站点备份文件保存目录
#databasedir: 数据库文件保存目录
#url:你oss地址
#bucket: 存储桶名称
#accesskeyid: AccessKeyId
#accesskeysecret : AccessKeySecret
OSS_OPTIONS = [{'sitedir':'sites','databasedir':'databases','url':'oss-xx-xxxx.aliyuncs.com','bucket':'bucketName',
                'accesskeyid':'ASDI123ISDD12','accesskeysecret':'SAD123ASD123ASD213ASD'},]

#腾讯云COS配置
#sitedir: 站点备份文件保存目录
#databasedir: 数据库文件保存目录
#region:区域
#bucket: 存储桶名称
#accesskeyid: AccessKeyId
#accesskeysecret : AccessKeySecret
COS_OPTIONS = [{'sitedir':'sites','databasedir':'databases','region':'ap-hongkong','bucket':'bucketName',
                'accesskeyid':'ASD123ASDASD123','accesskeysecret':'ASD123ASDASD123ASD'},]

#OneDrive配置
#name: 名称
#用于区别配置文件中配置，可以设置为多个{'name':'backup1' .....},{'name':'backup2' .....}，这样的话需要你认证多次不同账户
#sitedir: 站点备份文件保存目录
#databasedir: 数据库文件保存目录
ONE_DRIVE_OPTION = [{'name':'backup1','sitedir':'sites','databasedir':'databases',}]

#默认无需修改 用于申请API访问 此处采用萌咖(MoeClub)提供的
ONE_DRIVE_CLIENT = {'client_id':'ea2b36f6-b8ad-40be-bc0f-e5e4a4a7d4fa','client_secret':'h27zG8pr8BNsLU0JbBh5AOznNS5Of5Y540l/koc7048='}

#Email备份配置

#发送配置
#host: 邮箱smtp服务器地址
#username: 用户名 password:密码 port:端口 is_ssl:是否ssl加密连接 True或者False
EMAIL_OPTIONS_SENDERS = [{'host':'smtp.AA.com','username':'AA@AA.com','password':'123446','port':465,'is_ssl':True},]

#接收邮箱
EMAIL_OPTIONS_RECEIVERS = ['receivebackup@foxmail.com',]

#7z.exe位置 利用7zip来压缩
#只有windows需要(在程序plugin目录中已经附带有了
#linux安装过"p7zip"就行
WINDOWS_7ZIP_PATH = os.path.join(str(ROOT_DIR),'plugin') + '\\7z.exe'

```

### onedrive认证方法
```
修改config.py中配置后，程序目录运行python3 auth_onedrive.py
```
![](https://i.loli.net/2019/02/06/5c5a90ad4f540.png) 
![](https://i.loli.net/2019/02/06/5c5a909812f68.png) 

```
backup.py为主程序 如果要设置计划任务 设置为 python3 yourPath/backup.py
```

## 更新历史
* 1.2
    * 支持备份到onedrive
* 1.1
    * 能用

* 1.0
    * 能用

## 关于作者

LoneKing – [@LoneKing's Blog](https://loneking.net) 


[https://github.com/LoneKingCode/PyBackup](https://github.com/LoneKingCode/PyBackup)

