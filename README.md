Netcup Monitor Pro 📊

Netcup Monitor Pro 是专为 Netcup RS/VPS 用户打造的智能化流量监控与自动化流控面板。

它解决了 Netcup 用户最大的痛点：限速（Throttling）后的自动化处理。通过对接 Netcup 官方 SOAP API，它能精准识别服务器状态，并根据状态自动指挥 qBittorrent 和 Vertex 进行流量规避和恢复。

✨ 核心功能

1. 🛡️ 智能限速策略 (核心逻辑)

当检测到服务器被 Netcup 限速时，系统会根据种子分类执行精细化操作，而非简单粗暴的停止所有任务：

HR (Hit & Run) 保护机制：

做种中 (Seeding) 的 HR 种子：不删除、不暂停。自动将上传速度限制在安全范围（默认 10KB/s），确保持续做种以满足站点考核，同时避免消耗过多被限速的宝贵带宽。

下载中 (Downloading) 的 HR 种子：自动暂停，防止在限速期间通过龟速下载占用连接数。

空间与带宽释放：

非保留分类：直接删除。用于清理非必要种子，释放磁盘空间。

保留分类 (Keep)：自动暂停。用于长期保种的资源，进入休眠状态，等待限速解除。

2. ⚡ 自动恢复模式

当 SOAP API 检测到限速解除（恢复高速）时：

自动恢复所有暂停的种子。

自动解除所有种子的速度限制（恢复全速上传）。

3. 🔗 Vertex 深度联动

智能选机：自动读取 Vertex 的 RSS 规则，并将下载服务器列表动态更新为当前未限速的机器。

自动重启：支持在更新规则后自动重启 Vertex 容器（需 Docker Socket 权限），确保规则即时生效。

4. 📊 现代化监控面板

流量趋势：可视化展示过去 7 天的流量消耗趋势。

健康报告：记录每日限速时长、日均限速分析。

多端适配：基于 Bootstrap 5 的响应式设计，手机端完美管理。

5. 🔔 多渠道通知

支持 Telegram Bot、企业微信 (Webhook) 和 企业微信 (应用) 推送每日状态简报。

🚀 部署指南 (Docker Compose)

1. 准备工作

确保服务器已安装 Docker 和 Docker Compose。

2. 创建配置文件

在服务器上创建目录并编写 docker-compose.yml：

mkdir -p /root/netcup-monitor/data
cd /root/netcup-monitor
nano docker-compose.yml


3. Docker Compose 配置 (含自定义端口)

将以下内容复制到 docker-compose.yml 中。请注意 ports 部分，您可以根据需要修改冒号左边的端口号。

version: '3.8'

services:
  netcup-monitor:
    image: ghcr.io/agonie0v0/netcup-monitor:latest
    container_name: netcup-monitor
    restart: unless-stopped
    
    # --- 网络与端口配置 ---
    # 格式为 "宿主机端口:容器端口"
    # 如果您想通过 8080 端口访问，请改为 "8080:5000"
    ports:
      - "5000:5000"
    
    volumes:
      # 数据目录，存放数据库和配置文件
      - ./data:/app/data
      # 挂载宿主机时区
      - /etc/localtime:/etc/localtime:ro
      # (可选) 挂载 Docker Socket，仅在需要脚本自动重启 Vertex 容器时需要
      - /var/run/docker.sock:/var/run/docker.sock
    
    environment:
      - TZ=Asia/Shanghai
      # (可选) 自定义 Flask Session 密钥，增加安全性
      # - SECRET_KEY=your_secure_random_string


4. 启动服务

docker-compose up -d


访问地址：http://你的IP:5000 (如果您修改了端口，请使用修改后的端口)。

⚙️ 初始化配置

首次访问面板后，请点击右上角的 “登录”（默认无密码），进入设置页面完成以下配置：

1. 基础连接

qBittorrent：填写地址、用户名、密码。

Netcup SCP 账号：必须填写。前往 Netcup SCP 获取 Customer ID 和 密码。这是判断限速的唯一依据。

2. 策略配置 (关键)

保留分类 (Keep Categories)：

填写如 Keep, PTER。这些分类的种子在限速时会被暂停（而非删除）。

HR 保护配置：

受保护分类：填写涉及考核的分类，如 HDSky, U2。

上传限制：设置一个较小的值（如 10 KB/s）。限速期间，处于这些分类且正在做种的任务将被限制为此速度。

3. Vertex 联动 (可选)

填写 Vertex 的地址和账号。

监控 RSS ID：在 Vertex 的 RSS 列表中找到规则 ID（例如 4c145005），多个 ID 用逗号分隔。

⚠️ 免责声明

数据安全：本项目涉及对 qBittorrent 种子的删除操作。请务必正确配置“保留分类”和“不托管”选项，防止误删重要数据。

账号安全：Netcup SCP 账号密码仅保存在您本地的 data/config.json 中，不会上传至任何第三方服务器。

责任限制：作者不对因配置错误、软件 bug 或 Netcup 策略变更导致的数据丢失或账号封禁承担责任。

🛠️ 常见问题

Q: 为什么状态显示 "Unknown"？
A: 请检查 SCP 账号密码是否正确，或者 Netcup API 是否暂时不可用。

Q: "不托管"模式是什么意思？
A: 在实例配置中勾选“不托管”后，脚本只会监控该机器的流量和限速状态并生成图表，绝对不会对 QB 执行暂停、删除或限速操作，适合用于观察或重要机器。

Q: 如何更新镜像？
A: 运行 docker-compose pull && docker-compose up -d。
