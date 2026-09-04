# 校园网自动认证助手（AutoCampusNet）

一个基于 Go 的 Windows 校园网自动认证工具，支持系统托盘常驻、掉线自动重连、Web 配置页与开机自启。

## 功能特性

- 自动检测校园网在线状态，掉线后自动重新认证
- 系统托盘常驻运行，支持快速操作
- 本地 Web 配置界面（`http://localhost:8080`）
- 支持手动触发认证与状态检查
- 支持开机自启（Windows 注册表）
- 单实例运行（重复启动会直接打开配置页）
- 运行日志落盘（便于排错）
- 支持 IPv4 / IPv6 参数组装（含 IPv6 展开编码）

## 运行环境

- 操作系统：Windows 10 / 11
- Go 版本：1.20+

> 说明：项目包含 Windows 相关 API 调用（互斥锁、注册表、控制台窗口处理），主要面向 Windows 使用。

## 快速开始

### 方式一：使用预编译程序

1. 获取 `campus-net-auth.exe`
2. 双击运行
3. 首次运行会自动打开配置页面
4. 填写账号密码并保存
5. 程序将转入托盘后台运行

### 方式二：源码构建

```bash
go mod download
go build -ldflags "-H windowsgui" -o campus-net-auth.exe
```

或直接执行根目录脚本：

```bat
build.bat
```

## 安装与卸载脚本

仓库根目录提供了便捷脚本：

- `install.bat`：创建桌面快捷方式并写入开机启动项
- `uninstall.bat`：停止进程、删除启动项和本地数据目录

## 使用说明

### 首次配置

在浏览器打开 `http://localhost:8080`（首次运行会自动打开），填写：

- `account`：校园网账号
- `password`：校园网密码
- `check_interval`：检查间隔（秒，建议 30，范围 10~300）

高级配置可展开设置：

- `check_url`：在线状态检查接口
- `login_url`：认证接口模板（支持占位符替换）
- `auto_start`：是否开机自启

### 托盘菜单

- **⚙️ 配置**：打开 Web 配置页面
- **🌐 网络状态**：立即检查当前连接状态
- **🚀 开机自启**：切换开机自启
- **退出**：关闭程序

## 配置文件

默认路径：`%USERPROFILE%\CampusNetAuth\config.json`

示例：

```json
{
  "account": "2021001234",
  "password": "your_password",
  "check_url": "http://10.10.102.50:801/eportal/portal/online_list",
  "login_url": "http://10.10.102.50:801/eportal/portal/login?callback=dr1005&login_method=1&user_account=%2C0%2C{account}%40unicom&user_password={password}&wlan_user_ip={wlan_user_ip}&wlan_user_ipv6={wlan_user_ipv6}&wlan_user_mac=000000000000&wlan_ac_ip=&wlan_ac_name=&jsVersion=4.1.3&terminal_type=1",
  "auto_start": true,
  "check_interval": 30
}
```

`login_url` 支持以下占位符：

- `{account}`：账号（URL 编码）
- `{password}`：密码（URL 编码）
- `{wlan_user_ip}`：本机 IPv4
- `{wlan_user_ipv6}`：本机 IPv6（展开并编码）

## 本地数据目录

程序运行后会创建：

```text
%USERPROFILE%\CampusNetAuth\
├── config.json  # 配置文件
└── app.log      # 运行日志
```

## Web API（本地）

- `GET /api/config`：读取配置
- `POST /api/config`：保存配置
- `GET /api/status`：查询在线状态
- `GET|POST /api/auth`：手动认证
- `POST /api/exit`：退出程序

## 项目结构

```text
AutoCampusNet/
├── main.go
├── go.mod
├── go.sum
├── README.md
├── INSTALL.md
├── CONFIG.md
├── USAGE_EXAMPLES.md
├── PROJECT_STRUCTURE.md
├── build.bat
├── install.bat
├── uninstall.bat
├── templates/
│   └── config.html
└── static/
    └── style.css
```

## 常见问题

### 1) 程序启动后看不到窗口

程序默认托盘常驻，并会隐藏控制台窗口；请在系统托盘区域查找图标。

### 2) 认证失败

- 检查账号密码是否正确
- 检查 `check_url` / `login_url` 是否仍与学校认证接口一致
- 查看 `%USERPROFILE%\CampusNetAuth\app.log` 获取详细错误信息

### 3) 开机自启未生效

- 检查是否成功写入注册表启动项
- 重新通过托盘菜单切换一次“开机自启”

## 安全说明

- 配置文件中会保存账号密码，请妥善保护本机账户安全
- 默认认证流程基于校园网现有接口，是否加密取决于学校网络环境

## 相关文档

- [安装部署指南](./INSTALL.md)
- [配置说明](./CONFIG.md)
- [使用示例](./USAGE_EXAMPLES.md)
- [项目结构](./PROJECT_STRUCTURE.md)

## 贡献

欢迎通过 Issue / Pull Request 提交建议与改进。
