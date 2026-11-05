# sing-box v1.13.8：REALITY 指纹兼容补丁

路由器目标为 JCG Q30 Pro、ImmortalWrt 25：完整 ARM64 musl 内核，使用 UPX 压缩，沿用裸核加 Web 面板。桌面目标按 Windows x64 构建：完整默认功能，不使用 UPX。

## 修改范围

运行时代码只改 `common/tls/reality_client.go`：新增 26 行、删除 11 行，净增 15 行。另新增本说明和 `.github/workflows/build-personal-mlkem.yml`；相对官方 v1.13.8 共涉及三个文件。

原版无条件删除 `X25519MLKEM768`；修改后，在指纹同时提供 supported group 和 key share 时保留，只有组合不完整时才过滤并重建握手。原 REALITY 认证和依赖版本保持不变。节点指纹使用 `chrome`。

路由器读取 v1.13.8 原始 `release/DEFAULT_BUILD_TAGS` 并追加 `with_musl`；Windows 读取原始 `release/DEFAULT_BUILD_TAGS_WINDOWS`。两者均保留全部官方默认功能，包括 NaiveProxy、gVisor、QUIC、WireGuard 和 Clash API。Windows 包附带必需的 `libcronet.dll`。依赖文件和原始功能标签文件保持不变。

两种构建沿用官方常规链接选项，去除调试信息；只有路由器可执行文件经过 UPX 压缩。

## 云端编译

工作流的两个构建任务并行执行：路由器准备官方 musl 工具链、编译、UPX 压缩；桌面直接编译 Windows x64、不做 UPX。各自上传产物，没有额外测试或面板检查。

先在 GitHub fork `SagerNet/sing-box`，打开 fork 的 Actions 页面并启用工作流。再打开 Git Bash，将 `YOUR_GITHUB_USER` 替换为自己的 GitHub 用户名后执行：

```bash
cd "C:/Users/80552/pi-cwd/20261017/sing-box-q30pro"
git remote set-url origin https://github.com/YOUR_GITHUB_USER/sing-box.git
git add common/tls/reality_client.go \
  .github/workflows/build-personal-mlkem.yml \
  README.personal-mlkem.zh-CN.md
git commit -m "Fix REALITY fingerprint compatibility on v1.13.8"
git push -u origin fix/reality-mlkem-v1.13.8
```

推送上述分支会触发 **Build 1.13.8 REALITY MLKEM**。打开 fork 的 Actions，进入此次运行，在 Artifacts 下载对应文件：

| Artifact | 内容 |
| --- | --- |
| `sing-box-1.13.8-mlkem-linux-arm64-musl-upx` | 路由器用的完整 UPX 压缩版 `sing-box` |
| `sing-box-1.13.8-mlkem-windows-amd64` | 桌面用的未加壳 `sing-box.exe` 和 `libcronet.dll` |

在电脑解开下载的 ZIP。路由器上传其中的 `sing-box`，该文件可以直接执行，无需路由器安装 UPX。桌面将 `sing-box.exe` 和 `libcronet.dll` 放在同一目录使用；ZIP 只是传输打包，桌面可执行文件没有 UPX 压缩。

完整 musl 产物仍待云端编译，大小以此次构建结果为准。此前约 20 MB 可作参考；此前的精简产物不用于本方案。

## 查看与官方源码的 diff

运行时代码的全部差异：

```bash
git diff v1.13.8 -- common/tls/reality_client.go
```

提交后查看包括构建工作流和本说明在内的全部差异：

```bash
git diff v1.13.8 HEAD
```

## 裸核加 Web 面板

沿用原配置、启动参数、Clash API 地址和面板文件目录。备份旧核，停止原进程，替换内核后按原方式启动即可。产物版本标记为 `1.13.8-mlkem`。
