# 什么是反向代理？

用户访问代理服务器  ---->  代理服务器向后端真实服务器拿数据返回给用户；  
好处：**安全、灵活、负载均衡**

# 反向代理的配置
```Nginx
location ^~ /api/ {
			rewrite ^/api/(.*)$ /$1 break;
			proxy_pass http://localhost:8080;
        }
```
1. `location` 按URL路径分发请求的指令（不同的路径交给不同的处理方式）
2. `^~` 表示按前缀匹配,（此处表示匹配以`/api/`开头的路径）
3. `^.....$` 表示按正则表达式匹配,`()`表示捕获组，`$1`表示第一个捕获组
4. `rewrite` 是nginx中用来重写URL的指令（此处表示将`/api/`开头的路径重写为`/$1`，并中断后续处理）
5. `proxy_pass` 表示将请求代理到指定的后端服务器

## sites-available 与 sites-enabled

> 这是 Debian/Ubuntu 系 Nginx 的管理约定（官方原版 Nginx 没有这个结构）

- `sites-available`：存放**全部站点的原始配置文件**，放这里不会自动生效；
- `sites-enabled`：只放**软链接**，指向 sites-available 里的配置。

核心操作：**启用就加软链，停用只删软链**，无破坏性开关站点。
```nginx
# 1. 新建站点配置
nano /etc/nginx/sites-available/demo.com.conf
# 2. 启用站点：创建软链接（不要复制文件）
ln -s /etc/nginx/sites-available/demo.com.conf /etc/nginx/sites-enabled/
# 3. 禁用站点：只删除软链接
rm /etc/nginx/sites-enabled/demo.com.conf
# 4. 每次改完校验 + 重载
nginx -t
systemctl reload nginx
```

主配置 `/etc/nginx/nginx.conf` 的 http 块一般有这一行：
```nginx
include /etc/nginx/sites-enabled/*;
```
Nginx 启动 /reload 时，读取 `sites-enabled` 下所有文件（软链接会自动读到源文件）


1. 一个域名 / 子域名单独一个 conf 文件
2. **只编辑 sites-available 里面的源文件**，不要直接编辑 sites-enabled 内的链接