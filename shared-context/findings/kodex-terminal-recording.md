# 真实终端录制

## 正式站点当前路线

- 正式资产为 `KodexDocs/docs/public/recordings/` 的 12 段录制，不再使用旧 probe 充当当前版本演示；现均自动播放。
- 3 段原生 CLI 实录：`KodexDocs/scripts/record-native.py` 使用隔离 Home、禁网 bubblewrap、独立 tmux 与 asciinema；输出 `.cast`、原终端抓屏及来源 sidecar。
- 另 3 段原生 CLI：`record-live.py` 载入真实特性历史的英文副本，执行片段在明确授权下只读绑定认证、联网调用真实模型；结果与索引复用独立演示会话。原始 Home/Session 不挂载。
- 再新增 2 段原生 CLI：`record-integrations.py` 在独立禁网 Home 中录制 Hooks 管理和真实本地 MCP 协议服务，包含持久化快照与请求核验。见[详细记录](docs-integrations-recording.md)。
- 4 段组件录制：测试驱动生产提问、建议、终端条目和标签组件，使用 Mosaic TRUECOLOR 渲染导出，不从纯文本拼 ANSI。设置 `KODEX_DOCS_RECORDINGS_DIR` 才写文件；每个关键帧展示 2 秒，是编辑时间线而非模型耗时。
- 当前源码基线为 `89205188d5da39f36b983a0bfb57752977feb21a`，另有 Revert 确认框 private → internal 的测试可见性变更；CLI 已重新构建。不是已发布版本。准确的 CLI、源码变更、导出器、录制器和每段资产哈希见[manifest](../../KodexDocs/docs/public/recordings/manifest.json)。
- 当前执行片段包含实际模型运行与压缩结果，合并结果片段在可丢弃文件系统历史中真实执行 Fork/Revert；提问、审批与终端句柄的独立片段仍为 fixture，不能声称启动子 Agent 或杀死真实进程。
- 复用命令见 [README](../../KodexDocs/README.md)；逐项覆盖、实际运行检查与未展示项见[补录验收](docs-showcase-refinement-2026-09-08.md)。无提交、上传或发布。

## 初次 probe：已验证路线（历史记录）

- 2026-09-08：用户已观看并认可试验效果；采用 **真实原生 CLI → asciinema → asciinema-player**，不重写 TUI。
- 使用当前 Linux 的 tmux 3.6、bubblewrap 0.11.1，另下载官方 asciinema 3.2.1；临时 npm 环境使用 asciinema-player 3.17.0。
- tmux 只负责提供固定 PTY、发送真实键鼠事件、抓取对照画面；录制来自 CLI 的真实输出，不将抓屏文字拼成假界面。
- 不需要录制服务器、模型账号或网站后端。本次没有上传、提交或修改正式网站依赖。
- [官方录制说明](https://docs.asciinema.org/manual/cli/quick-start/)、[播放器接入](https://docs.asciinema.org/manual/player/quick-start/)、[播放参数](https://docs.asciinema.org/manual/player/options/)。

## 初次 probe 结果与边界

- 短片 120×36、16.078 秒、271,574 字节；asciicast v3，94 条输出事件、1 条退出事件，没有输入事件。
- 真实场景：侧栏 hover、点击钉住后移开；简繁中文与英文多行粘贴；Ctrl+Z/Y；Settings；目录选择与 `do` 过滤；三次 Escape 依次清除过滤、关闭目录弹层、关闭设置。
- Chromium 本地回放的 10 个时间点，全部 36 行文字与当时 tmux 抓屏一致；中文相邻字定位相差两列。采样前景 `rgb(158,239,253)`、背景 `rgb(0,79,88)` 与原始真彩色转义一致。
- 播放、暂停、时间跳转、marker 跳转、结束与重播通过；1440/1024/768/390/320 px 下等比缩放且无横向溢出；没有页面异常或资源请求失败，只请求本地静态资源。
- **窄屏缩放不等于易读**：120 列在手机上文字很小；正式内容应拆短场景，按功能选择更窄的实际录制尺寸，不能只改播放器列数来假装重排。
- `.cast` 不是像素录像：不含系统鼠标指针，但包含实际 hover/点击引起的界面变化；可用播放器原生 marker 标注操作步骤。
- 本次 detached tmux 未提供终端 palette metadata；已验证应用显式真彩色，不声称默认终端底色和所有字体都像素一致。正式接入须固定字体/默认主题；本机使用 JetBrains Mono、Noto Sans Mono CJK SC，跨机器字体供应尚未验证。
- CLI 正常退出，状态码 0；退出会离开 alternate screen，末帧可能空白。封面应选运行中的真实时间点，而非退出帧。
- 这是运行与浏览器验证；未重新编译 CLI，未运行 Gradle/单元测试，也未验证 VitePress 集成或线上 Pages。

## 本地证据与来源

- 保留的本机结果在忽略目录 `out/kodex-recording-probe/`：`short.cast`、10 份 `frame-*.txt`、`verification.json`、桌面/手机截图、`verify.py` 和 `public/` 独立回放页。不是待发布网站资产，也不随 Git 克隆。
- 从 BuildKodex 根目录启动：`uv run python -m http.server 4180 --bind 127.0.0.1 --directory out/kodex-recording-probe/public`，打开 `http://127.0.0.1:4180/`；只服务 `public/`，不服务录制 Home。
- 同机复验：`uv run --with playwright python out/kodex-recording-probe/verify.py`；依赖 `/usr/bin/google-chrome` 与上述服务。
- CLI 取自 `Kodex/app/cli/build/bin/linuxX64/releaseExecutable/kodex-cli.kexe`，SHA256 `92539baaa88f4cb78a58ad73c5a23844ef49f65babb4ee968a692c80f26e6720`。
- 短片 SHA256 `f527e5dd7c578f5ab3816e4a3e64eba7ebbb147fa183641c091500f0ea3912a1`；保留原始输出，没有修改 ANSI 画面。
- 检查时 Kodex HEAD 为 `ab54349003db16e6a59c48f63b8b6caf1745441d`，但工作区有改动，已有二进制不能证明来自此提交。**正式录制需另记可追溯构建基线，不能把此试验当成当前发布版功能证明。**
- asciinema 官方 Linux musl 二进制 SHA256 `bec9781bc8f297a9d3d74ff60205599507f2abba1183578b8b2f22be4c999214`；[来源](https://github.com/asciinema/asciinema/releases/tag/v3.2.1)。

## 复用录制步骤

- 新建专用临时目录，放入 CLI 副本 `kodex`、录制器 `asciinema`，创建 `home/`、`recorder-home/`、`project/{docs,src,tests}/`。不要复用个人 Home。
- 原生 Home 的解析顺序是 USERPROFILE → HOME → HOMEDRIVE/HOMEPATH（[OsEnvironment.kt:23](../../Kodex/utils/os-environment/src/commonMain/kotlin/io/github/stream29/kodex/utils/osenvironment/OsEnvironment.kt#L23)）；只覆盖 HOME 不足以保证隔离，因此清空继承环境并禁止网络。
- 在该目录保存下面的 `launch-kodex.sh` 并赋予执行权限。本机已验证此隔离方式；其他系统需重新验证，失败不得降级到私人环境。

```bash
#!/usr/bin/env bash
set -euo pipefail
here=$(cd -- "$(dirname -- "$0")" && pwd)
exec bwrap --unshare-all --die-with-parent \
  --ro-bind /usr /usr --ro-bind /lib /lib --ro-bind /lib64 /lib64 \
  --ro-bind /etc/ssl /etc/ssl \
  --proc /proc --dev /dev --tmpfs /tmp \
  --dir /opt --ro-bind "$here/kodex" /opt/kodex \
  --bind "$here/home" /home/demo \
  --bind "$here/project" /work/demo \
  --clearenv --setenv HOME /home/demo --setenv USER demo \
  --setenv PATH /usr/bin:/bin --setenv SHELL /bin/sh \
  --setenv TERM xterm-256color --setenv COLORTERM truecolor \
  --setenv LANG C.UTF-8 --setenv TMPDIR /tmp \
  --chdir /work/demo /opt/kodex
```

- 同目录保存 `record.sh` 并赋予执行权限；不给 `-I/--capture-input`，不运行 upload。录制器也使用独立 Home。

```bash
#!/usr/bin/env bash
set -euo pipefail
here=$(cd -- "$(dirname -- "$0")" && pwd)
exec env -i PATH=/usr/bin:/bin HOME="$here/recorder-home" \
  SHELL=/bin/sh TERM=xterm-256color COLORTERM=truecolor LANG=C.UTF-8 \
  "$here/asciinema" rec \
  --command "$here/launch-kodex.sh" --window-size 120x36 \
  --capture-env TERM,COLORTERM --title 'Kodex isolated recording probe' \
  --idle-time-limit 1 --return "$here/demo.cast"
```

- 在此目录执行下列输入。坐标为 **120×36 下本次版本**，不是稳定选择器；换布局/尺寸必须重新核对。鼠标使用真实 SGR 协议，中文使用 bracketed paste。

```bash
here=$PWD
tm() { tmux -S "$here/tmux.sock" "$@"; }
send() { tm send-keys -t demo -l -- "$1"; sleep 1; }
tm -f /dev/null new-session -d -x 120 -y 36 -s demo "$here/record.sh"
tm set-option -g remain-on-exit on
sleep 2
tm capture-pane -p -t demo                 # 先确认启动完成
send $'\e[<35;2;2M'                       # hover 左栏
send $'\e[<0;2;2M\e[<0;2;2m'             # 点击钉住
send $'\e[<35;60;10M'                     # 移开，确认仍展开
send $'\e[<0;32;34M\e[<0;32;34m'         # 聚焦输入
send $'\e[200~录制示例：这是实际 Kodex 界面。\n繁體中文與 English / 123\e[201~'
tm send-keys -t demo C-z; sleep 1
tm send-keys -t demo C-y; sleep 1
send $'\e[<0;110;35M\e[<0;110;35m'       # Settings
send $'\e[<0;60;7M\e[<0;60;7m'           # Browse
send d; send o                           # 分次输入，核对 Filter: do
tm capture-pane -p -t demo
for i in 1 2 3; do tm send-keys -t demo Escape; sleep 1; done
tm send-keys -t demo C-c; sleep 1
tm list-panes -t demo -F '#{pane_dead} #{pane_dead_status}' # 本次 1 0
tm kill-server                           # 仅此专用 socket
```

- 每个关键输入后保存 `capture-pane`，播放器按相应时间跳转并逐行比较。正式自动化优先等待已知画面；本次固定停顿仅为探测。
- `--idle-time-limit` 保存播放提示，并不删改输出。验帧时覆盖 `idleTimeLimit: 1000` 保持本次原始时间线；正式启用压缩后重新核对 marker。
- [v3 事件时间是相邻事件间隔](https://docs.asciinema.org/manual/asciicast/v3/)，分析时须累计，不能按 v2 绝对时间读取。
- 审核 header、全部输出事件与示例数据；禁用输入捕获不代表输出自动脱敏。本次命令 metadata 含临时脚本路径，没有私人路径、会话或凭证。
- 保留经过检查的 `.cast` 和验证结果；停掉专用 tmux/HTTP 服务，删除临时 Home、CLI 副本、下载工具与依赖，不动已有会话。

## 初次探索时的其他途径（历史记录）

- **首选 PTY 实录**：已打通，无需产品改动；工作台、输入编辑、目录导航和设置界面可以先在此隔离环境录制。
- **Mosaic 测试渲染导出**：作为需要固定历史/表单数据时的补充。现有 [TestMosaic.kt:25](../../Kodex/Mosaic/mosaic-testing/src/commonMain/kotlin/com/jakewharton/mosaic/testing/TestMosaic.kt#L25) 支持自定义 SnapshotStrategy，[同文件:50](../../Kodex/Mosaic/mosaic-testing/src/commonMain/kotlin/com/jakewharton/mosaic/testing/TestMosaic.kt#L50) 支持键鼠/粘贴输入；[TuiDialogTest.kt:79](../../Kodex/app/view/components/src/mosaicTest/kotlin/io/github/stream29/kodex/cli/components/TuiDialogTest.kt#L79) 已使用真实组件的 TRUECOLOR 渲染。
- 默认纯文本快照会去掉颜色（[TestMosaic.kt:128](../../Kodex/Mosaic/mosaic-testing/src/commonMain/kotlin/com/jakewharton/mosaic/testing/TestMosaic.kt#L128)），不能直接拿来当彩色录制。初次探索只核对扩展点；后续实现情况见本文“正式站点当前路线”。
- 运行控制、提问表单、建议会话、历史回退等依赖特定状态：内容任务先复用现有测试 fixture 设计无私人数据的真实状态；若采用组件测试导出，标明 fixture 驱动的实际组件，不冒充在线模型操作。
- 视频录屏本轮不采用，也未实测：当前需要的终端回放已通，不为鼠标指针再引入视频处理流程。
- 正式播放器任务仍负责 Markdown 原文与播放器挂载、固定字体/默认主题、适合内容的尺寸和 Pages 静态资源检查；本次独立探测不等于正式接入完成。
