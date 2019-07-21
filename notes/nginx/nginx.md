nginx：日志管理
默认的日志格式：
ip + 用户认证（可能为空） + 访问时间 + 请求格式（GET/POST）+ 请求状态 + 发送多少字节 + 上一个页面是什么 + 用户代理（浏览器格式类型）+ （代理服务器）

配置访问日志：
access_log	/path/文件名	main(默认格式）;

日志定时切割：
date -d yesterday +%Y-%m-%d 获取昨天的日期并以%Y-%m-%d显示

shell脚本
#bin/bush
LOGPATH = log文件路径
BASEPATH = 备份到的路径
filename = $BASEPATH/($date -d yesterday +%Y-%m-%d).log

mv $LOGPATH $BASEPATH
touch $LOGPATH

kill -USER1 | cat pid

crontab -e 
*/1 * * * * sh /shell脚本path

location 语法：
location [=|~|~*|^~] pattern{}		= 精准匹配 ~ 正则匹配 location pattern{} 一般字符串匹配 最长匹配方式
正则会检查一般匹配，如果一般匹配匹配成功，但正则也有匹配，将会立即覆盖 

rewrite 重写（重定向）：
if + 空格 + （条件）
条件类型：= 判断相等 ~ 判断正则 ~* 不区分大小写的正则 -f -d -e 判断是否为文件、目录、是否存在
$remote_addr 获取IP
$http_user_agent 获取浏览器类型
rewrite (重定向并不会改变请求的网址）==》但会拿到重定向后的网址内容返回
set 设置变量 可达到多次判断重定向

正则表达式支持反向引用 ==》带有{}的正则表达式需要用“”括起来	正则表达式中所有的（）中的值都能被记录下来，在后面使用$*进行引用
例如：
rewrite "goods-(\d{1, 7})\.html" /goods.php?id=$1	$1 是前面正则表达式中的第一个（）中的记录值

压缩与解压 ==》gzip
http协议 ==》声明请求头可以接受 accept-encoding: gzip deflate sdch （压缩方法）
注意：	二进制文件不建议压缩，压缩率很小且消耗大量CPU资源
	太小的文件不建议压缩
gzip 常用参数：
	gzip on/off		是否开启gzip
	gzip_buffers 32 4K | 16 8K	缓冲（压缩在内存中缓冲几块，每块多大）
	gzip_comp_level [1-9]	压缩级别（推荐为 6 ）压缩级别越高，压缩的文件就越小。但同时消耗的CPU计算资源越大
	gzip_disable 正则匹配UA	什么样的URI不进行gzip压缩
	gzip_min_length 4*1024	开始压缩的最小长度（太小压缩意义不大）
	gzip_http_version 1.0|1.1	开始压缩的http协议版本（可不设置，几乎全为 1.1 协议）
	gzip_proxied		设置请求者代理服务器，应该如何压缩内容
	gzip_types text/ application/	对那些类型的文件进行压缩 如：txt，xml，html，css，js
	gzip_vary on/off		是否传输gzip压缩的标志
常用推荐配置：
	gzip		on;
	gzip_buffers	32 4K;
	gzip_comp_level	6;
	gzip_min_length	4*1024;	单位字节
	gzip_type		text/css text/xml application/x-javascript	（类型获取方式：可以去官网查或者在 conf 下的 mime.types 中）

expires 缓存设置提高网站性能
对于网站图片，一旦发布基本不会发生改变，能够在用户访问后，缓存到用户的浏览器端，较长时间保存
设置过期时间：
	在 location 或 if 中：
		expires 30s; 30m; 2h; 30d;	分别为30秒、30分钟、2小时、30天
注意：在静态文件的location中直接添加即可，可保证不向服务器发起请求。过期后回向服务器发起一次请求，若未改变返回304，拉取缓存信息
304缓存的原理：浏览器向服务器发起请求时，同时相应etag标签和last_modified_since两个标签，下次请求时，头文件发送这两个标签，若标签未变化返回标签，发生变化返回新的标签和文件信息

nginx 反向代理 + 负载均衡
proxy 反向代理	协议 + proxy_pass IP/域名	（常说的“动静分离”）
	proxy_set_header X-Forwarded-For	$remote_addr	将客户端的 IP 设置在头信息中，保证日志中能够拿到真正的客户端 IP 而不仅仅时代理服务器的 IP （upstream转发前的本机 IP）
upstream 负载均衡	
将多台反向代理服务器绑定在一起形成一个虚拟服务器 ==》upstream 起一个组名
upstream 组名 {
	server	127.0.0.1:8080 weight=1 max_fails=2 fail_timeout=3;
	server	127.0.0.1:8081 weight=1 max_fails=2 fail_timeout=3;
	weight 连接权重，数字越大越容易被命中；max_fails 最大连接失败所允许的次数；fail_timeout 每次连接失败的超时时间（单位秒）
}

nginx 和 memcached（memcached 是一个高性能的分布式内存对象缓存系统）

nginx 第三方模块的安装（一致性哈希==》ngx_http_upstream_constistent_hash）：
nginx -V		可以查看旧版本的安装参数信息
configure --help	可以查看编译参数
--add-module=PATH	安装第三方文件
