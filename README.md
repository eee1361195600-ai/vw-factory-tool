# vw-factory-tool

**Factory 多账号切号器 / 会话管理工具**：Windows 版 `vwFactory.exe`，macOS 版 `vwFactory-osx-arm64.zip` / `vwFactory-osx-x64.zip`（见下文「macOS 版」）。
为每个 Factory 账号维护一套独立的本地环境，让多个账号可以**同时打开、互不干扰**，并提供跨账号的会话查看、重命名、删除与云端会话恢复。

> Windows 版适用于 Windows 10/11 x64，依赖系统自带的 .NET Framework 4.8；macOS 版自带运行时。两者都需要已安装 Factory 官方桌面客户端。

## 功能

### 多账号并行
- 一个账号一张卡片，一个账号一个独立窗口，可同时打开多个 Factory。
- 每个账号拥有独立的 Factory 数据目录（`FACTORY_HOME_OVERRIDE`）和浏览器数据目录，登录状态、会话、`hostId` 互不混用。
- 首次打开新账号时在官方页面登录即可；同一邮箱不能重复添加。
- 网页登录完成后点「返回 Factory」，会准确回到**发起登录的那个账号窗口**（工具自己接管 `factory-desktop:` 协议并按前台窗口 / 最近选中账号路由，官方客户端覆盖注册后会自动修正）。

### 额度与凭据
- 卡片显示各账号剩余额度，可在 Standard / Droid Core 两种视图间切换。
- 一键刷新所有账号的额度和凭据有效期；凭据快过期时自动通过官方 droid 续期。
- 每 6 小时自动刷新一次所有账号凭据，保持登录不过期（refresh token 本身被吊销/长期失效时仍需重新登录）。

### 自动重新登录（邮箱验证码）
- refresh token 彻底失效（官方 droid 续期失败）时，可通过 WorkOS device flow + 邮箱验证码自动重新登录：打开账号右键菜单的「重新登录（自动接码）」，或用 `vwFactory.exe --relogin <profileId>` 命令行触发；刷新额度时遇到续期失败也会自动触发一次（同账号 30 分钟内至多一次，全局不并发）。
- 要求（目前仅 Windows）：自动接码依赖随 exe 部署的 `tools\factory_relogin.exe`，无需 Python；首次运行会自动下载浏览器组件（约 150MB，存于 `tools\ms-playwright`）。如需改用本机的 kling console / 指纹浏览器数据源，在 `relogin.json` 里填 `KlingConsoleDir` / `FingerprintDir` 即可覆盖。验证码取自 Outlook 邮箱，凭据两种来源：账号自己的 Outlook 账密+rt（优先），或 kling workflow console 的 `data\workflow.db` 同名账号（仅当 `KlingConsoleDir` 已配置时回退）。
- Outlook 账密行格式：`email----password----client_id----refresh_token`（也接受 `:` / `|` / 制表符分隔）。可右键账号卡片「导入/覆盖 Outlook 账号…」保存（DPAPI 加密存于 `数据目录\outlook\<id>.bin`，仅当前 Windows 用户可解）；「复制 Outlook 账密+rt」可复制该行。卡片上「有rt / 无rt」徽标指示是否已导入。
- 「添加账号」对话框可直接粘贴 Outlook 账号行：留空备注名时以邮箱为名，保存后立即自动登录该账号（弹浏览器收验证码完成 WorkOS device flow），无需手动打开 Factory。也可用 `vwFactory.exe --import-outlook <profileId> "<行>"` 命令行导入。
- 邮箱取到验证码时若 Microsoft 轮换了 refresh_token，新 rt 会自动写回本地凭据。
- 设置文件 `%LOCALAPPDATA%\vwFactory\relogin.json`（首次运行自动创建）：`Enabled`、`PythonPath`、`KlingConsoleDir`、`FingerprintDir`、`Proxy`、`Headless`，全部可留空使用默认值。

### 分发给他人
- 只需打包 `vwFactory.exe` + `tools\` 目录（含 `factory_relogin.exe`），解压后双击 `vwFactory.exe` 即可使用；自动接码首次运行会自动下载浏览器组件（约 150MB）。
- 账号数据目录在 `%LOCALAPPDATA%\vwFactory\`（账号配置 `profiles.json`、自动登录设置 `relogin.json`）；Outlook 账密按账号保存在 `profiles.json` 同级目录下 `outlook\<id>.bin`（DPAPI 加密）。
- 登录组件为随 exe 部署的 `tools\factory_relogin.exe`；成功后会校验新令牌邮箱与账号名一致再写回凭据文件，避免写错账号。
- 每小时自动「检测续接」：对登录凭据 2 小时内到期的账号调用官方 droid 强制续期；续期失败且已导入 rt 的账号自动重新登录一次；未临期与无 rt 的账号静默跳过。顶部「检测续接」按钮可立即手动执行（不受 30 分钟节流限制），状态栏汇总「续接 N，跳过 M（未临期 X / 无rt Y），失败 Z」，手动模式下失败项会弹窗列出。

### 账号管理（卡片右键）
- 编辑名称、重新登录。
- 删除账号：**只移除卡片，保留本机会话文件**（路径会在提示中给出）。之后用同一邮箱再登录，会自动接续保留的会话记录和原来的 `hostId`，不会产生"同一账号两套环境"的问题。

### 会话中心（跨账号）
- 汇总本机所有账号的历史会话，支持搜索、只读预览。
- 重命名（同步修改云端标题）、删除（先归档云端会话，成功后再把本机文件移入回收站，可从回收站恢复）。
- 生成"接续材料"（去掉推理、工具参数和工具结果的可见文本），方便在别的会话/账号里继续。

### 卡片「▤ 会话」云端会话列表
- 直接从云端拉取该账号的全部会话，并标注「本机可用」或「仅云端」。
- 右键：重命名、归档/删除、复制续接提示词。
- **克隆为本机新会话**：对「仅云端」的会话（例如在别的机器 / 已删除的环境里创建的），拉取云端完整消息，用当前账号本机的 `hostId` 在云端新建会话并逐条原样上传，同时写入本机会话文件，之后就能在本机直接打开并继续对话。
  - 不调用模型、不消耗额度（只是搬运已有消息）。
  - 原会话保留不动；上传中途失败会自动归档半成品新会话且不写本机文件。
  - 云端不保存的本地事件（如 todo 状态、回合结果）无法恢复，仅迁移消息本身。
- **迁移到其他账号**：与克隆机制相同，但新会话建在另一个账号下——用源账号拉取云端消息，以目标账号的凭据在云端新建会话并逐条上传，同时写入目标账号的本机目录（用其 `hostId`）。默认迁移成功后删除源账号的原会话（云端归档 + 删本机文件）以保持一致；确认框可取消勾选以保留原会话。两个账号都需已登录，且目标账号需在本机打开过一次。之后在目标账号的 Factory 里打开即可继续对话。

## 使用

1. 下载 `vwFactory.exe`、同目录的 `vwFactory.exe.config` 和 `tools\` 目录（含 `factory_relogin.exe`，仅自动接码需要）放到任意文件夹。
2. 双击运行，工具会检测本机 Factory 客户端状态。
3. 点击「添加账号」：只填备注名或邮箱 → 点击卡片，在弹出的官方页面完成登录；粘贴 Outlook 账号行（`email----password----client_id----refresh_token`）→ 自动接码登录，无需手动操作。
4. 再添加其他账号，点击各自卡片即可并行打开。

所有数据存放在 `%LOCALAPPDATA%\vwFactory\` 下（各账号的 profile、Factory home、浏览器数据、DPAPI 加密的 Outlook 凭据），程序不会把任何凭据上传到第三方。网络访问：额度与云端会话请求发往 `api.factory.ai`；自动接码时会访问 Factory/WorkOS 登录页、Outlook 邮箱（微软登录与 IMAP）以收取验证码，首次运行还会下载浏览器组件到 `tools\ms-playwright`。

## macOS 版

macOS 版是同一套业务逻辑的跨平台版本（.NET 8 + Avalonia 11，自带运行时）。

1. 从仓库根目录下载并解压 `vwFactory-osx-arm64.zip`（Apple Silicon，M1 及以后）或 `vwFactory-osx-x64.zip`（Intel），把 `vwFactory.app` 拖到「应用程序」。应用自带运行时，不需要另外安装 .NET。
2. 应用仅 ad-hoc 签名：首次打开请右键 → 打开，或执行 `xattr -dr com.apple.quarantine vwFactory.app`。
3. 首次读取 Keychain 中的 Factory 登录密钥（服务名 `Factory CLI`）时系统会弹窗，请选择「始终允许」。
4. 数据存放在 `~/Library/Application Support/vwFactory/`（`profiles.json` 为账号配置，`profiles/<id>/home` 为各账号的 Factory home，`browser-data/<id>` 为各账号的浏览器数据）。
5. 协议接管（`factory-desktop:` 登录回调）需要以 `.app` 方式运行；应用启动后会把自己注册为该协议的默认处理程序，并把回调转发给前台 / 最近选择的那个 Factory 窗口。

已在 macOS 26（Apple Silicon，Factory 0.184）上实测：检测 Factory.app、添加账号并以隔离 home / `--user-data-dir` 并行启动多个 Factory 窗口、卡片切回窗口、右键重命名 / 删除账号、会话中心列出本机会话、`factory-desktop://` 回调转发、本机文件移入废纸篓、重启后状态保留。未实测（本机没有 Factory 账号）：读取 Keychain 密钥解密登录态、额度刷新、云端会话列表 / 重命名 / 归档 / 克隆，以及 Intel 包的实际运行。

## 说明与限制

- 云端会话的重命名 / 归档 / 克隆使用的是 Factory 客户端自身所用的内部接口（`/api/sessions/*`），并非公开 API，Factory 升级后有可能变化。
- Factory 云端只提供"归档"，没有公开的永久删除；本工具的"删除"= 云端归档 + 本机文件移入回收站。
- 卸载：结束程序后删除 exe 和 `%LOCALAPPDATA%\vwFactory\` 即可。
