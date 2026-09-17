# mobileserver-bin-resources

MobileServer（安卓本地服务器 APP）的**运行时二进制资源仓库**。
APP 外壳本身不打包这些组件，运行时从这里下载对应 arm64 包，解压到手机 `files/server/bin/` 下独立进程运行。

## 目录结构

```
mobileserver-bin-resources/
├── manifest.json              # APP 读取的组件清单（版本/下载地址/sha256）
├── conf/                      # 各服务标准配置模板（参考/兜底用）
│   ├── nginx.conf.template
│   ├── php.ini.template
│   ├── redis.conf.template
│   └── mariadb.cnf.template
└── README.md
```

二进制 zip **不放 git 历史**（太大），通过 **GitHub Releases** 分发，见下面流程。

## 一、二进制要求（关键）

所有组件必须是 **arm64 (aarch64)、Android bionic libc** 能独立运行的版本：

| 组件 | 解压后路径 | 说明 |
|------|-----------|------|
| Nginx | `bin/nginx/nginx` | 需静态或相对路径动态库，独立进程 |
| PHP | `bin/php/php-cgi` | CGI 模式，监听 9000 |
| MariaDB | `bin/mariadb/mariadbd` | 必须连同 `share/`（字符集）和插件 so 一起打包 |
| Redis | `bin/redis/redis-server` | 选 7.2 BSD 版，避开 8.x SSPL |
| OpenList | `bin/openlist/openlist` | Go 静态编译，官方提供 android-arm64 |

> ⚠️ 普通 Linux(x86/glibc) 包不能用；Termux 的 .deb 也不能直接用（它硬编码依赖 `/data/data/com.termux/...`）。
> 正确做法是用 Android NDK 交叉编译并设 rpath 到 APP 目录，或用社区已编译好的静态包。

## 二、发布流程（每次更新组件）

1. 拿到某组件的 arm64 zip，命名如 `nginx-1.26.2-arm64.zip`
2. 计算 sha256：
   ```bash
   sha256sum nginx-1.26.2-arm64.zip
   ```
3. 在 GitHub 本仓库 → **Releases** → **Draft a new release**
   - Tag 填如 `v1.0.0`
   - 把所有组件 zip 拖到 Assets 上传
4. 打开 `manifest.json`：
   - 把对应组件的 `sha256` 替换成第 2 步算出来的值
   - 确认 `url` 里的版本号与 release tag、文件名一致
   - 提交并 push

APP 启动时读 manifest.json → 比对本地版本 → 缺失或更新就下载 → sha256 校验通过后解压。

## 三、首次接入自检

- [ ] manifest.json 能被浏览器直接打开（raw 地址可访问）
- [ ] 每个 url 在浏览器能直接下载到文件（不是 404）
- [ ] 每个 sha256 与实际 zip 一致（校验失败 APP 会拒绝安装并删除）
- [ ] 解压后 entry 路径与 manifest 的 `entry` 一致

## 协议

各二进制版权归原项目所有，遵循其原始许可（见 manifest.json 每个组件的 license 字段）。
本仓库仅以独立聚合方式分发，不修改其源码。
