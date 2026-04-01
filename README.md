## 新手小白也能0基础docker compose部署新版独角Dujiao-Next  


### 前言
- 感谢独角（Dujiao-Next）作者 assimon 的开源项目！

- 本篇教程将指导你使用1Panel面板（开源）结合 Docker 环境，优雅地部署新版独角。
相比于社区文档中使用 aaPanel 手动下载编译好的二进制文件部署，本篇采用的 宝塔+Docker部署 方案，在后续的版本更新维护上会更加方便快捷。

- 手动去安装Redis和Postgresql数据库应用好处：
如果你有其他web项目或者程序已经在使用1Panel面板  以及 Redis和Postgresql  
我们这里docker-compose编排文件在部署独角项目时就不需要去单独拉取创建Redis和Postgresql容器

后续你有新的项目需要Redis和postgresql  也都可以直接复用连接。

注意⚠️
本篇教程是1Panel面板 docker compose部署教程，和宝塔面板天差地别的不同！请不要弄混了docker-compose.postgres.yml和config.yml文件！


### 提前准备
1. 一台vps（服务器），一个域名
服务器推荐2核2G运行内存最好，美国/香港或其他国外地区，如果你需要既便宜又足够正常使用的服务器  这里推荐RN商家的
RN这家服务器商家是一家很老的美国商家了，对于我们 科学上网 的同学来说既便宜又足够正常项目所需的

2026年服务器活动价格表：

1 GB RAM KVM VPS   

1x vCPU Core   

1 GB RAM    

24 GB SSD  

2 TB Bandwidth  

Price: $11.29/Year  （11.29U一年）

---------------------------------------------

2 GB RAM KVM VPS （推荐该款/热门款）← 

1x vCPU Core

2 GB RAM

40 GB SSD     （40GB  纯SSD 超大硬盘内存）

3.5 TB Bandwidth

Price: $18.29/Year（18.29U一年）

---------------------------------------------

3.5 GB RAM KVM VPS （精英款）

2x vCPU Cores

3.5 GB RAM

65 GB SSD  （65GB  纯SSD 超大硬盘内存）

7 TB Bandwidth

Price: $32.49/Year（32.49U一年）

---------------------------------------------

4 GB RAM KVM VPS （土豪款）

3x vCPU Cores

4 GB RAM

105 GB SSD （105GB  纯SSD 超大硬盘内存）

9 TB Bandwidth

Price: $43.88/Year（43.88U一年）


直达购买链接我会放在本篇教程文本最下面，点击链接你只需要邮箱注册账号然后登录就行，不需要实名也不需要你填任何个人信息。
因为是美国商家，如果打开是英文  浏览器都有全页面翻译功能，翻译成中文或者使用商家自带的中文语言即可

下单购买服务器的时候，我们可以选择操作系统：ubuntu24版本、ubuntu22版本  或者  Debian12版本、Debian13版本

本篇教程以Debian12系统为例，服务器也是使用的RN商家，你也可以登录商家给的管理地址随时重新安装操作系统。

RN家的服务器统一管理地址基本都是：https://nerdvm.racknerd.com/login.php


购买服务器后，等待商家发给我们的服务器信息，记录下来：
- 我们的服务器IP
- 服务器根密码
- 服务器管理地址登录账号，登录密码


3. 解析好域名（建议托管到cloudflare  同时创建15年的免费证书）

- 本篇教程为例：docker compose分域名部署方式

所以我们在解析域名的时候解析三个域名，例如：

用户前端域名：shop.vfaka.cn

后台管理域名：admin.vfaka.cn

API后端域名：api.vfaka.cn


4. 在电脑桌面上准备好服务器终端连接工具，本篇教程使用的工具：FinalShell
FinalShell下载地址：https://www.hostbuf.com/t/988.html

也是很多人使用的免费工具

我们打开工具，添加我们购买的服务器进行连接，成功连接到服务器后执行命令安装1Panel可视化面板
1Panel面板安装命令

```bash
curl -sSL https://resource.1panel.pro/quick_start.sh -o quick_start.sh && bash quick_start.sh
```

后续你可能需要到的进程守护（可选）顺带执行

```bash
sudo apt-get install supervisor
```

安装好1Panel面板后，我们把面板登录信息全部复制记下来，到这里全部准备工作就已经做完了。


---

### 正式开始部署

1. 登录我们安装好的1Panel面板，登录 1Panel 面板后，按以下步骤操作：左侧菜单栏 → 应用商店

- 安装以下 3 个必须应用：
Redis	无特殊要求	需记录容器名称和自定义密码
PostgreSQL	无特殊要求	需记录容器名称
OpenResty	无特殊要求	反向代理服务

- ⚠️ 安装注意事项
不要打开外部访问权限
设置一个较长的自定义密码

- 创建数据库，记录我们的数据库信息：
数据库名称：
数据库用户：
数据库密码：

- 记录以下信息后面要用：
Redis 容器名称：
PostgreSQL 容器名称：
Redis 密码：

- 可选安装
PgAdmin（可视化数据库管理面板）
用途：查看和管理 PostgreSQL 数据库
功能：查看数据库表结构、用户信息、订单信息等

---

2. 创建站点，设置反向代理监听端口

- 创建站点操作步骤：
1Panel 左侧菜单 → 网站 → 网站 → 创建 → 选择：反向代理
填写主域名和代理地址

以此类推，完成三个站点的创建。

- 域名分配方案（推荐）
采用分域名部署方案，三端使用不同的域名或二级域名：
类型	用途	                示例	                说明
用户前端	用户浏览和购买商品	shop.vfaka.cn	用户访问地址
后台前端	管理员管理系统	admin.vfaka.cn	后台管理地址
API 后端	数据交互通信	api.vfaka.cn	接口网关地址

- 监听端口配置示例：
用户前端：shop.vfaka.cn
  监听端口：9005
  反向代理：127.0.0.1:9005

后台前端：admin.vfaka.cn
  监听端口：9006
  反向代理：127.0.0.1:9006

API 后端：api.vfaka.cn
  监听端口：9007
  反向代理：127.0.0.1:9007

---

3. 申请或上传SSL 证书
- 本篇教程是将域名托管到了cloudflare  也创建好了15年证书，我们就可以直接上传复制下来了的证书然后保存即可使用
三个站点都必须开启 HTTPS

都必须配置 SSL 证书

建议使用 Cloudflare 托管域名和证书


- 如果你没有或者不想托管到cloudflare  那么在1Panel面板这里按照以下步骤手动申请证书（90天有效期  可以开自动续约）
1Panel 左侧菜单 → 网站 → 证书 → Acme 账户 → 点击创建  → 填写邮箱 → 账号类型默认 → 密钥算法默认 → 点击确认
点击申请证书 → 选择我们的域名 可以一次性给我们需要的全部域名进行申请 → 验证方式选择：HTTP → 点击确认

- 然后回到第2步，我们刚刚创建的三个站点域名都必须要开启 HTTPS
都必须配置 SSL 证书
建议使用 Cloudflare 托管域名和证书

---

4. 前往本篇教程github开源仓库
打开教程仓库：https://github.com/zaocao/1panel-docker-compose-dujiaonext

点击绿色 Code 按钮

选择 Download ZIP 下载

保存到桌面

---

5. 上传并解压项目文件
- 目录结构创建
操作步骤：
1Panel 左侧菜单 → 点击：文件 → 任意目录位置 → 创建本次独角根目录（例如：dujiaonext）

进入 dujiaonext 根目录

上传刚才下载的 ZIP 文件并点击两下解压


- 创建 data 文件夹及子文件夹
```bash
data/
├── logs/
├── redis/
├── postgres/
└── uploads/
```

- 设置文件夹权限
全选 config 和 data 文件夹
右键 → 权限 → 设置为 777


- 你也可以终端执行命令：
- 
```bash
mkdir -p /www/dujiao-next/{config,data/db,data/uploads,data/logs,data/redis,data/postgres}
```

不过我们都登录到1Panel面板了，手动创建也快，就不需要终端执行命令了


确保dujiaonext目录结构如下：
```bash
dujiaonext/
├── config/                          # 配置文件夹
│   └── config.yml                   # 主配置文件
├── nginx/                           # 反向代理补充配置（用完可删）
│   ├── user.conf
│   └── admin.conf
├── data/                            # 数据存储文件夹（需手动创建）
│   ├── logs/                        # 日志文件夹
│   ├── redis/                       # Redis 数据文件夹
│   ├── postgres/                    # PostgreSQL 数据文件夹
│   └── uploads/                     # 上传文件夹
├── .env                             # 环境变量配置文件
├── docker-compose.postgres.yml      # Docker 容器编排文件
└── README.md                        # 教程文件（可删除）
```

---

6. 站点的反向代理配置信息补充

- 用户前台域名（shop.vfaka.cn）反向代理配置补充
操作步骤：
左侧菜单 → 网站 → 网站
点击之前创建的 用户前端域名（shop.vfaka.cn）
选择 反向代理
点击 配置文件
在第一个标签页中，打开 dujiaonext/nginx/user.conf
按提示复制两段代码
粘贴到反向代理配置文件中的指定位置
保存


- 后台管理域名（admin.vfaka.cn）反向代理配置补充
操作步骤和上方的完全相同
这里只需要改用 dujiaonext/nginx/admin.conf 文件

--- 

7. 完成配置填写

.env 配置文件填写示例：
```bash
# 镜像标签（保持默认）
TAG=latest

# 时区（保持默认）
TZ=Asia/Shanghai

# ========== 监听端口号 ==========
# 填写你配置的 API 监听端口（示例：9007）
API_PORT=9007

# 填写你配置的用户前端监听端口（示例：9005）
USER_PORT=9005

# 填写你配置的后台前端监听端口（示例：9006）
ADMIN_PORT=9006

# ========== 管理员默认账号密码 ==========
# 默认管理员用户名（推荐改为：admin）
DJ_DEFAULT_ADMIN_USERNAME=admin

# 默认管理员密码（示例：admin123，搭建完后必须更改！）
DJ_DEFAULT_ADMIN_PASSWORD=admin123
```

📢 必读说明
管理员默认账号密码必须设置
搭建完成后第一时间必须更改



config.yml 配置文件填写示例：
```bash

# 关键配置项
# JWT 密钥配置
jwt:
  secret: "412df024-c5f7-4bdc-9550-4d8d6b09023d"  # ❌ 默认值，生产环境必须修改
  
user_jwt:
  secret: "3d2d0f71-975c-4d06-a663-d09ce6a509c5"  # ❌ 默认值，生产环境必须修改
修改方式： 使用较长且复杂的随机字符串（可"脸滚键盘"生成）

jwt:
  secret: "aB3$mK9!pL@xQ2$rS5%tU7^vW9&xY1*zC3~dE5_fG7-hI9.jK1(lM3)nO5+pQ7"
  
user_jwt:
  secret: "zY9*xW7&vU5$tS3@rQ1%pO9!mN7^kL5&jI3$hG1@fE9(dC7-bA5_`Z3+yX1*wV9"

# 为什么必须修改？ 这是系统设计的"防呆瓜"机制，简单密钥会导致部署失败



# CORS 跨域配置
cors:
  allowed_origins:
    - "https://user.vfaka.cn"      # 改成你的用户前端域名（HTTPS）
    - "http://user.vfaka.cn"       # 改成你的用户前端域名（HTTP）
    - "https://admin.vfaka.cn"     # 改成你的后台前端域名（HTTPS）
    - "http://admin.vfaka.cn"      # 改成你的后台前端域名（HTTP）


# PostgreSQL 数据库配置
database:
  driver: postgres
  # 格式：host=容器名 user=用户名 password=密码 dbname=数据库名 port=5432 sslmode=disable TimeZone=Asia/Shanghai
  dsn: host=1Panel-postgresql-VIxa user=dujiaonext password=abc123 dbname=dujiaonext port=5432 sslmode=disable TimeZone=Asia/Shanghai
需要修改的部分：

host=你的PostgreSQL容器名称
user=你的数据库用户
password=你的数据库密码
dbname=你的数据库名称
# 注意：参数之间必须有一个空格


# Redis 配置
redis:
  enabled: true
  host: redis           # 改成你的 Redis 容器名称
  port: 6379            # 保持默认
  password: abc123      # 改成你的 Redis 密码


# 需要修改的部分：
host=你的Redis容器名称
password=你的Redis密码


配置完成，点击：保存
```

---

8. 执行启动命令以及后续版本更新命令

操作步骤：
在 1Panel 文件管理器中，回到 dujiaonext 根目录
点击右上角 终端 按钮

如果是第一次打开：
输入服务器密码进行连接测试
保存连接

手动退回到 dujiaonext 根目录
再次点击 终端

也可以使用SSH连接工具  打开前面我们安装1Panel面板时用的连接工具
cd到我们独角项目根目录，也就是dujiaonext目录下，执行命令
```bash
cd /opt/dujiaonext
```

然后执行启动命令
```bash
docker compose --env-file .env -f docker-compose.postgres.yml up -d
```

执行过程
按 回车键 执行
等待自动拉取和创建容器
所有输出显示 绿色 = 成功



后续独角Dujiao-Next版本更新升级
我们只需要在dujiaonext目录下 点击终端（或者通过连接工具cd到dujiaonext目录）  执行两条命令
先拉取独角Dujiao-Next最新版本镜像命令
```bash
docker compose -f docker-compose.postgres.yml pull
```

拉取成功后再执行重启命令
```bash	
docker compose --env-file .env -f docker-compose.postgres.yml up -d
```

正常更新基本不会出错，全绿即更新完成！




### 主要提示，必须看
首次登录后台的必须操作
🔐 更改默认账号密码
重要性等级：⭐⭐⭐⭐⭐ 必做

为什么必须更改？
安全风险极高：恶意人员会使用爬虫扫描后台域名
容易被破解：默认账号密码较弱，几秒钟就能破解
后果严重：黑客进入后会：
挂跳转链接
篡改支付配置
盗取数据

第一时间必须更改默认的账号和密码！
第一时间必须更改默认的账号和密码！
第一时间必须更改默认的账号和密码！
（重要的事情说三遍！）




### QA 常见情况

容器	出错概率	说明
用户前端	低	基本不会出错
后台前端	低	基本不会出错
API 后端	高	易报错，需检查配置


查看 API 错误日志
日志排查方式一：1Panel 容器管理
左侧菜单 → 容器
找到 API 容器
右拉 → 日志 → 查看具体报错


日志排查方式二：本地日志文件
位置：dujiaonext/data/logs/app.log
包含完整错误信息
经验：严格按教程操作，基本不会出错


报错状态码 已知问题分析：
Q1：后台登录时，账号密码正确，登录提示错误：“限流服务不可用”？
A：Redis 没有连接成功，导致 API 没有跑起来。重点检查 Redis 相关配置（如密码，Redis容器名称，配置是否少或多了空格）。

Q2：容器全部启动成功，启动时也没有报错，账号密码正确，登录时提示：“405 错误”？
A：需要检查的地方：
config.yml配置文件当中的跨域配置没填对也就是 CORS 跨域配置。一个填的是你的用户前端域名，另一个应该是 admin 后台管理域名。
也可能是你反向代理补充配置信息时：后台管理域名的反向代理配置可能没设置正确。


