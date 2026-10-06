# mctier-linux-web-git（Arch Linux）

为 [MCTier Linux Web](https://github.com/Copper0SO4/MCTier_Linux_Web) 提供 Arch Linux / pacman 源码构建包。安装后的启动命令是 **`mctier`**。沿用本地 Rust Web 服务与浏览器界面，不使用 Electron 或 Linux Tauri 窗口。

## 安装

先安装构建工具（联网下载 npm/Cargo 依赖，需要一定磁盘空间）：

```bash
sudo pacman -S --needed base-devel git rust nodejs npm

git clone https://github.com/Copper0SO4/mctier-linux-web-git.git
cd mctier-linux-web-git
makepkg -si
```

用普通用户运行 `makepkg`；安装依赖和生成的软件包时由 pacman 请求系统授权。当前只支持 **x86_64**。本仓库提供打包文件，尚未宣称已提交 AUR，不能直接假定 `yay -S mctier-linux-web-git` 可用。

包名为 `mctier-linux-web-git`，从 Linux Web 仓库 **master 开发分支**获取最新源码；不直接追踪官方 MCTier master，不固定在3.9.0发行ZIP。版本由Linux服务版本、Git提交数量及短哈希组成。要更新：

```bash
git pull --ff-only
makepkg -si
```

即使打包文件没有变化，再次运行 makepkg 也会更新 Git 源码。构建只使用随源码保存的 npm/Cargo 锁文件；EasyTier core/CLI 固定2.5.0，压缩包与两个二进制均核对SHA-256。若Linux适配的来源检查或补丁失败，构建停止，需维护者对照上游修正，不自动忽略检查。

## Rust/LLVM 工具链错误

构建前先检查 `rustc -vV` 和 `cargo --version`。如果前者直接报 `symbol lookup error`、`undefined symbol ... version LLVM_23.1`，说明编译器本身加载动态库失败，还没有编译应用。包依赖已安装不等于系统工具链可以运行；当前开发机上也能复现此问题。此前软件包构建使用的是官方独立Rust1.90.0，不能算作系统Rust验证通过。

PKGBUILD现在会在解压核心/npm安装之前检查编译器并明确报错。可选处理：由用户正常完整升级系统并复核Rust、LLVM及gcc-libs的一致性；不要仅单独升级LLVM或手工替换系统动态库。另一条路径是使用经过来源/校验核实的独立Rust工具链（本项目已验证1.90.0），在当前命令中指定其bin目录：

```bash
PATH="/path/to/verified-rust/bin:$PATH" makepkg -si
# 或在本打包仓库中：
PATH="/path/to/verified-rust/bin:$PATH" yay -Bi .
```

将占位路径换成实际工具链路径，须包含cargo和rustc；不需要修改全局PATH或移除系统Rust。本包不会自动下载安装另一套工具链。系统升级也不会由PKGBUILD执行。

npm的esbuild安装脚本被阻止提示不是上述Rust失败原因。当前验证过的锁文件包含平台esbuild包，默认npm设置下已成功构建；若之后出现真正的esbuild缺失错误，需单独检查，勿为了这条提示放开所有依赖脚本。

## 启动

```bash
mctier
```

请在终端运行 `mctier`。启动器以普通用户运行本地服务，就绪后尝试打开默认浏览器。使用 **Chrome/Chromium** 访问：

<http://127.0.0.1:14700>

启动命令会保持在终端前台，Ctrl+C停止服务；也可在另一终端运行 `mctier --stop`。关闭浏览器页面不会停止服务。本包不提供桌面菜单入口。安装和升级不会自行启动服务，不安装开机启动单元。14700若已被旧实例占用，启动器会报告冲突，不会误报就绪。默认手动加入大厅；如在软件设置明确启用了启动自动组网，则沿用该保存配置。麦克风与屏幕共享由用户开启。

可选浏览器和密码密钥环：

```bash
sudo pacman -S --needed chromium gnome-keyring
```

GNOME Keyring也可以由其它支持Secret Service的密钥环替代；需要用户D-Bus会话、已解锁密钥环与桌面polkit认证代理。系统需有 `/dev/net/tun`。Firefox大厅连接仍因已知WebSocket 1006断连而封锁。广告拦截扩展也可能阻止信令。

## 权限和安装路径

- `/usr/bin/mctier`：启动命令包装。
- `/usr/lib/mctier-linux-web/launcher`：原版Linux Web启动器。
- `/usr/lib/mctier-linux-web/bin/mctier-linux-web-service`：Rust服务。
- `/usr/lib/mctier-linux-web/bin/binaries/`：经过校验的EasyTier核心与CLI。
- `/usr/share/doc/mctier-linux-web-git/`：中文说明、高级网络设置与UFW/P2P排障文档。

只有核心缺少 `cap_net_admin,cap_net_raw=ep` 时，启动器才通过 `pkexec setcap` 请求系统认证并复核。由用户自己在系统窗口输入密码；不以root运行MCTier/EasyTier。安装脚本不直接设置CAP或调整防火墙。升级替换核心后，可能需要重新授权。

打包禁用strip/debug拆分，防止pacman包构建改变EasyTier二进制而导致内置哈希校验失败。capability留在安装后的系统文件上，不以普通用户拥有的构建文件作为运行核心。

本地服务默认仅监听回环。大厅/WebRTC信令和EasyTier虚拟数据链路分别判断；P2P、双向语音、屏幕共享和长期恢复仍取决于网络及实际测试。UFW/firewalld为可选组件，修改由应用内预览与确认控制。

## 维护与验证

每次修改PKGBUILD、本地源文件或依赖时，同步更新README、HANDOFF及 `.SRCINFO`：

```bash
makepkg --printsrcinfo > .SRCINFO
makepkg --verifysource
```

本仓库只维护Arch打包；协议和功能在Linux Web主仓库维护。包体构建验证与安装、真实房间测试分开记录；本轮结果见 [HANDOFF.md](HANDOFF.md)。

MCTier使用自定义源码可得、非商业许可；随包EasyTier使用LGPL-3.0，许可证安装到 `/usr/share/licenses/mctier-linux-web-git/`。此包不是完全开源许可的软件，请遵守主项目许可证。
