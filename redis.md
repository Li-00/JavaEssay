---
title: "redis"
tags: ""
---

## 重要操作

2022.1.21安装redis地址在   /usr/local/redis/bin        本机LiunxIP:  192.168.231.128      ifconfig

**bin下运行**  ./redis-server                       /usr/local/redis/bin/redis.conf   **配置文目录**

/etc/systemd/system目录下  redis-service文件  配置redis开机自启



导入DB失败因为数据库版本问题（DB数据为8以上数据库导出数据）  utf8mb4_0900_ai_ci替换为utf8_general_ci    utf8mb4替换为utf8

**考虑到项目中驱动都是用的8以上，准备将本地mysql升级**



```tex
[Unit]
Description=redis-server
After=network.target

[Service] 
Type=forking
ExecStart=/usr/local/redis/bin/redis-server /usr/local/redis/bin/redis.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

## redis操作命令

```shell
#分别是启动 停止 重启  开机自启

systemctl start redis.service

systemctl stop redis.service

systemctl restart redis.service

systemctl enable redis.service

#查询
ps -ef | grep redis

#关闭防火墙
systemctl stop firewalld
```

### 常用命令

```shell
#切换库
SELECT [index]
#删库
FLUSHALL
#层级set与普通set
set home:user:1 zzy      set name zzy

```



## redis配置

Redis支持很多的参数，但都有默认值。
●daemonize默认情况下，redis 不是在后台运行的，如果需要在后台运行，把该项的值更改为yes。
●bind指定Redis只接收来自于该IP地址的请求。
●port监听端口，默认为6379.
●databases设置数据库的个数，默认使用的数据库是0。
●save设置Redis进行数据库镜像的频率,
●dbfilename镜像备份文件的文件名。
●dir数据库镜像备份的文件放置的路径。
●requirepass设置客户端连接后进行任何其他指定前需要使用的密码。
●maxclients限制同时连接的客户数量。
●maxmemory设置redis能够使用的最大内存。

**已设置**：在redis.conf中配置 bind 0.0.0.0   使得windows系统可以连接    requirepass密码设置为  123456    

```shell
/requirepass   找关键字
```

[protected-mode设置](https://blog.csdn.net/u012925172/article/details/84921861)   设置为no允许外网连接



# 项目

![img](https://gitee.com/zy0098/image_repo/raw/master/20220122162555.png)

# 遇到的问题及解决

## 1.23

- Navicat导入DB报错，因为DB文件是MySQL 8以上导出文件   本地使用的是5.7.30  

  解决：因为项目中依赖使用的是8.0以上    准备将本地MySQL升级为8.0以上**（待解决）**  [参考博文](https://www.dkewl.com/course/detail500.html)

- Maven运行报错

> 注: 某些输入文件使用了未经检查或不安全的操作。
> 注: 有关详细信息, 请使用 -Xlint:unchecked 重新编译。

因为本地编译环境有问题 更改本地jdk编译版本为8   [参考博文](https://www.cnblogs.com/shanyingyuyan/p/13500560.html)


 1.30
 ms-oauth2-server  在http://localhost:8082/oauth/token 地址测试获取token时必须先填写用户认证，其信息在子项目配置文件中定义
 ms-diners  

2.1
ms-diners    启动报错提示 程序包javax.xml.bind.annotation不存在
原因是版本问题，项目中java是1.8的     
[博客园](https://www.cnblogs.com/FlyAway2013/p/10908340.html)

JWT(Json Web Token)

[jwt概念](https://juejin.cn/post/6977280069437751310)   
[与session对比](https://www.jianshu.com/p/576dbf44b2ae)

网关gateway   系统diners   认证中心oauth
在客户端发出请求连接到网关判断是否有令牌  有的话直接执行操作
  > 没有的话返回提示登录的信息
  
  客户端发出包含用户账户密码的登录请求信息，登录URL在网关白名单不再被过滤可以调用系统的登录业务 校验用户信息完成后 进而远程调用认证中心生成token并将其存储在redis并返回给客户端
  
![image.png](https://boostnote.io/api/teams/H0QI9a3wv/files/22b7d610d7ba80b2847f49546a38ca316643fbb4ff127f31bb94bceb5be9b3a0-image.png)

通过JWT实现了SSO（单点登录）
携带Token向网关发出请求，获取token并调用认证中心进行验证
> 有效则放行请求使其调用业务方法进行业务处理
   无则拒绝访问

![image.png](https://boostnote.io/api/teams/H0QI9a3wv/files/c1a6cd5fc6e77298c7e7dccbbf93c4022b3d8cd54f3697d4f988b75a8f311bba-image.png)


[发验证码注册用户](http://localhost/diners/send) 调用接口传参手机号，检验非空后查询redis中是否有该手机号对应的验证码 ，如没有则生成6为随机码然后发送短信发送成功将verifyCode存入如redis，失效时长60s
注册完成直接调用登录接口传参登录


```java

  /**
     * 根据手机号查询是否已生成验证码
     *
     * @param phone
     * @return
     */
    private boolean checkCodeIsExpired(String phone) {
        String key = RedisKeyConstant.verify_code.getKey() + phone;
        String code = redisTemplate.opsForValue().get(key);
        return StrUtil.isBlank(code) ? true : false;
    }



/**
     * 发送验证码
     *
     * @param phone
     */
    public void send(String phone) {
        // 检查非空
        AssertUtil.isNotEmpty(phone, "手机号不能为空");
        // 根据手机号查询是否已生成验证码，已生成直接返回
        if (!checkCodeIsExpired(phone)) {
            return;
        }
        // 生成 6 位验证码
        String code = RandomUtil.randomNumbers(6);
        // 调用短信服务发送短信
        // 发送成功，将 code 保存至 Redis，失效时间 60s
        String key = RedisKeyConstant.verify_code.getKey() + phone;
        redisTemplate.opsForValue().set(key, code, 60, TimeUnit.SECONDS);
    }


```


[以Json形式发送用户名、密码、手机号、验证码注册用户](http://localhost/diners/register)

```java
 /**
     * 用户注册
     *
     * @param dinersDTO
     * @param path
     * @return
     */
    public ResultInfo register(DinersDTO dinersDTO, String path) {
        // 参数非空校验
        String username = dinersDTO.getUsername();
        AssertUtil.isNotEmpty(username, "请输入用户名");
        String password = dinersDTO.getPassword();
        AssertUtil.isNotEmpty(password, "请输入密码");
        String phone = dinersDTO.getPhone();
        AssertUtil.isNotEmpty(phone, "请输入手机号");
        String verifyCode = dinersDTO.getVerifyCode();
        AssertUtil.isNotEmpty(verifyCode, "请输入验证码");
        // 获取验证码
        String code = sendVerifyCodeService.getCodeByPhone(phone);
        // 验证是否过期
        AssertUtil.isNotEmpty(code, "验证码已过期，请重新发送");
        // 验证码一致性校验
        AssertUtil.isTrue(!dinersDTO.getVerifyCode().equals(code), "验证码不一致，请重新输入");
        // 验证用户名是否已注册
        Diners diners = dinersMapper.selectByUsername(username.trim());
        AssertUtil.isTrue(diners != null, "用户名已存在，请重新输入");
        // 注册
        // 密码加密
        dinersDTO.setPassword(DigestUtil.md5Hex(password.trim()));
        dinersMapper.save(dinersDTO);
        // 自动登录
        return signIn(username.trim(), password.trim(), path);
    }

 /**
     * 根据手机号获取验证码
     *
     * @param phone
     * @return
     */
    public String getCodeByPhone(String phone) {
        String key = RedisKeyConstant.verify_code.getKey() + phone;
        return redisTemplate.opsForValue().get(key);
    }
```




# other

 Redis 6.2.0 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 10215-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io   

### 状态

linux系统默认IP地址 127.0.0.1 端口6379	
更改bind 0.0.0.0
防火墙已关闭，保护模式已关闭

### windows系统

inet 192.168.255.128  netmask 255.255.255.0  broadcast 192.168.255.255


windows系统telnet连接linux地址（IP地址+端口）telnet 192.168.255.128 6379

redis连接密码 root



### linux系统：

查询linux系统使用的IP地址，端口等信息    ifconfig

查询redis进程使用的端口号    	ps -ef|grep redis
杀死redis进程（其他进程也适用）	kill -9 ****（*为上面查到的端口号）


redis的bin目录下启动redis命令    redis-server conf/redis.conf


指定IP地址开启redis进程    redis-cli -h 192.168.255.128

关闭防火墙		systemctl stop firewalld
查询防火墙状态		systemctl status firewalld 

### 使用iptable服务工具配置管理端口（开放或关闭等）

安装iptable服务工具----先检查是否安装了	iptables service iptables status
安装iptables		yum install -y iptables
升级iptables		yum update iptables 

安装iptables-servic      yum install iptables-services 

禁用/停止自带的firewalld服务  systemctl stop firewalld

编辑iptables防火墙，设置放开和关闭的端口等  vi /etc/sysconfig/iptables

重启iptables服务使配置生效	  systemctl restart iptables
查看iptables运行状态   systemctl status iptables

设置iptables开机自启动		systemctl enable iptables
禁止firewall服务开机自启动	systemctl disable firewalld

————————————————
版权声明：本文为CSDN博主「Jayden_1209」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_43966259/article/details/104765798
