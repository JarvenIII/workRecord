# 我的PHP

## 基础
- 双引号和单引号的区别
- [$_REQUEST, $_GET, $_POST](http://www.jb51.net/article/28957.htm)
- $GLOBALS 所有的变量都放在里面
- $FILE 上传文件使用
- $SERVER 系统环境变量
- [$SESSION $COOKIE](http://blog.csdn.net/sayigood/article/details/4850480) 会话控制
- post, put, get, delete
- echo, print, print_r, var_dump()
- 获得客户端ip $_SERVER["REMOTE_ADDR"] 获得服务器ip gethostbyname('www.baidu.com')
- 


## 数据库优化
### MySql
- 选取合适的字段属性，减少它的宽度,尽量设置为NOTNULL
- 索引，优化查询语句
- 事务
- 使用join代替子查询
- 使用union代替临时表
- 锁定表，优化事务

## 适应大流量
- 代码优化，尽量较少IO
- 提升服务器性能
- 多主机分流
- 数据库读写分离
- 控制大文件的上传下载
