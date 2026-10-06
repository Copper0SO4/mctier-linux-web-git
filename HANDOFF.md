# Arch 打包维护说明

更新：2026-10-06。用户目标：使用本仓库安装MCTier Linux Web到Arch Linux，以 `mctier` 命令启动，并保留随包核心、普通用户运行和按需CAP授权边界。

本仓库只管理打包，不复制Linux Web源码。源项目为 https://github.com/Copper0SO4/MCTier_Linux_Web 的master。当前功能/限制、协议及源码上游合并以该项目HANDOFF为准；已发布ZIP与本-git源码包不是同一制品。

## 文件职责

- PKGBUILD：Git源码、随包EasyTier/许可证校验、npm/Cargo准备、Linux独立构建和pacman安装布局。
- .SRCINFO：由 `makepkg --printsrcinfo`生成，每次PKGBUILD变化同步。
- mctier.sh：/usr/bin入口，调用/usr/lib/mctier-linux-web/launcher，避免BASH_SOURCE因路径包装找错资源。
- 当前不提供desktop文件/应用菜单入口；仅支持终端启动mctier并通过Ctrl+C停止。
- mctier-linux-web.install：只给出中文首次运行/升级提示，不能增加自动setcap、服务启动或防火墙更改。
- README：安装、升级、运行需求、已知限制及许可。

EasyTier包固定2.5.0，压缩包SHA-256沿用源项目fetch-binaries.sh；解压后两项再校验。许可证来自官方同版LICENSE（LGPL-3.0）。禁止strip/debug拆分改变核心哈希。后续核心更新须同时审查源项目校验、PKGBUILD校验、随包版本与许可，不独立升级到不兼容核心。

## 本轮验证

当前宿主为Arch Linux，已具备声明的构建/运行依赖；系统Rust曾有LLVM动态链接问题，本轮构建使用此前官方校验的临时Rust1.90.0，仅改变构建进程PATH，不修改系统安装。makepkg使用--nodeps跳过pacman依赖查询/安装，完整源码构建与包体检查结果随后记录。

没有安装到系统、启动服务、修改CAP/防火墙或接入真实房间；应用菜单点击、首次安装授权、升级后重授权与跨端联机留待用户实际安装验收。CI或包体检查不得标成这些真实测试已通过。

- 第一次Vite构建出现源码目录MCTier与本地包装文件mctier的名称碰撞（路径被解析到包装文件）。将打包源文件改名mctier.sh，安装目标仍为/usr/bin/mctier；重试Vite构建通过。源码适配未更改。README.md纳入source校验，确保源码包也包含安装说明。

- 构建成功：makepkg生成 `mctier-linux-web-git-3.9.0.r379.g9f64665-1-x86_64.pkg.tar.zst`，约109MiB，源代码提交9f64665；TypeScript/Vite（566项表情）及Rust release构建通过。Shell语法、desktop文件、SRCINFO一致性和包体9个必需文件字节一致性校验通过，执行文件/目录为root:root、0755，文档0644。EasyTier保持原官方哈希，未strip。
- 包SHA-256：`0b6e3651ed6b22ad1e2c23a4f376e32d7dd4f161af6b484cf7c18f78812065c1`。验证包未执行系统安装/认证/启动；未重跑源码全套自动化。主项目31a4795仅增加Arch文档，后续-git构建版本会随Git提交增加。
- 已准备main分支提交及推送至用户指定GitHub仓库。只上传打包源文件与文档，生成的.pkg.tar.zst保持本地，不宣称已上AUR。

## 2026-10-06：移除桌面入口

用户指出缺少适合桌面入口的退出机制，要求移除desktop。删除mctier.desktop及包内菜单/图标安装，移除desktop-file-utils构建依赖与相关检查；pkgrel提升为2，重新生成README校验与SRCINFO。保留终端命令mctier，中文说明和安装提示明确Ctrl+C退出、关网页不停止服务。前面的desktop验证结果只属于历史包。此次仅重新打包已有构建产物，不重编译业务代码、不安装或启动服务；升级时pacman会撤销旧包记录的desktop和图标。
- 验证：Shell语法、SRCINFO一致性及pkgrel2重新打包通过；包体确认无desktop/菜单图标、保留/usr/bin/mctier并包含更新的终端退出说明。未安装到系统。

## 2026-10-06：系统Rust不能启动的构建故障

用户yay -Bi日志在prepare的cargo fetch处失败，rustc -vV本身退出127，要求basic_string::_M_mutate@LLVM_23.1。现场系统Rust1.99/LLVM23.1.1/gcc-libs16.2可复现，无LD_LIBRARY_PATH/LD_PRELOAD覆盖；objdump确认Rust driver要求LLVM_23.1版本符号而LLVM无对应导出，libstdc++只导出GLIBCXX_3.4.21版本。证据指向当前系统工具链二进制不匹配，尚不确定是发行打包还是本机更新状态；不能归因MCTier源码，也不自动替换系统库。独立官方Rust1.90.0（LLVM20.1.8）正常启动，之前构建实际使用该工具链。

PKGBUILD增加prepare首项Rust/Cargo启动预检，README说明实际工具链与临时PATH办法；重新计算README SHA及SRCINFO。不自动下载编译器、不全局改PATH、不升级系统或安装软件。npm esbuild脚本提示不作为此Rust故障根因，保持原npm安全设置。

- 恢复验证：仅给构建进程PATH指定/tmp/mctier-rust-toolchain/installed/bin，完整makepkg（来源校验、prepare、TypeScript/Vite、Rust release、打包）通过，产物3.9.0.r381.g8aefe58-1；makepkg更新pkgver后按惯例把pkgrel重置1。包体验证新README、mctier入口及无desktop通过。系统Rust失败预检亦现场复现，原始错误清楚显示。未执行pacman安装、pkexec或真实联机；用户可在其打包仓库用同一临时PATH搭配yay -Bi .完成安装。
- makepkg的srcdir引用警告来自Rust编译时源路径（含开发态核心fallback），不是本次失败；已安装布局优先选择服务旁随包核心，包不依赖构建目录保留。若后续消除路径警告，需在主项目独立审查，不能直接删运行检查。
