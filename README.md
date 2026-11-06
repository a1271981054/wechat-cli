# wechat-cli · 本机微信只读检索与首次接入 Skill

面向 Codex 的 `wechat-cli` Skill，以及它使用的 **Rion WeChat CLI / Reader** 源码。提供聊天检索、联系人与会话读取、上下文查询、本地导出，以及独立、需明确授权的首次接入助手。

本仓库从 [Rion-Wu-tech/wechat-intelligence-hub](https://github.com/Rion-Wu-tech/wechat-intelligence-hub) 拆分相关组件，保留来源、AGPL-3.0 许可与 wxkey 相关 MIT 许可。公开版本只包含代码、文档和虚构测试，不包含使用者的配置、密钥、数据库或聊天记录。

## 安装到 Codex

需要本机 Git、Bash 与 Python 3。下列命令适用于 macOS/Linux 的安装入口；Windows 的已有材料读取与首次获取支持请先看 [Windows 指引](references/windows-access.md)。

目标目录应不存在。已有同名 Skill 或 CLI 时先检查版本与配置，安装器默认拒绝覆盖；不要为重装删除私有材料。

```bash
skill_dir="${CODEX_HOME:-$HOME/.codex}/skills/wechat-cli"
git clone https://github.com/a1271981054/wechat-cli.git "$skill_dir"
bash "$skill_dir/projects/rion-wechat-reader/install.sh" \
  --prefix "${CODEX_HOME:-$HOME/.codex}" --with-sqlcipher
"$skill_dir/scripts/reader.sh" self-test --require-sqlcipher
```

安装器在独立运行环境安装 SQLCipher 与 Zstandard，并提供 `rion-wechat-cli`、兼容别名 `rion-wechat-reader` 和可选助手 `rion-wechat-access`。安装不启动密钥获取、不操作微信，也不要求填写微信密码。

在新 Codex 会话中调用：

```text
使用 $wechat-cli 检查本机微信接入状态。
已有配置和访问材料先复用，按我的问题读取最小范围的聊天；
如果还未接入，先定位缺少的条件，再给出具体下一步。
```

只安装独立 CLI 时，可以从克隆目录运行：

```bash
bash projects/rion-wechat-reader/install.sh --prefix "$HOME/.local" --with-sqlcipher
"$HOME/.local/bin/rion-wechat-cli" self-test --require-sqlcipher
```

## 常用入口

```bash
reader="${CODEX_HOME:-$HOME/.codex}/skills/wechat-cli/scripts/reader.sh"
access="${CODEX_HOME:-$HOME/.codex}/skills/wechat-cli/scripts/access.sh"
"$reader" self-test --require-sqlcipher
"$reader" access-plan --pretty
"$access" diagnose --support-summary
```

`self-test` 使用虚构临时数据库。`access-plan` 与 `diagnose` 用于接入检查；退出码 0 或顶层 `ok:true` 不代表真实聊天可读。读取具体联系人前解析歧义，涉及“最新”时刷新实际数据。仅有通知预览时必须标明覆盖不完整。

## 能力与边界

| 组件 | 能力 |
|---|---|
| Reader | 会话、联系人、历史/时间线、搜索、上下文、群成员、本地导出，以及配置已支持的收藏/朋友圈/资源索引 |
| 材料导入 | 验证本人已有路径型 key、schema-2 salt-key 等材料，检查账号、数据库覆盖与权限 |
| 接入助手 | 安全诊断、材料归一化与配置、恢复状态检查；确实需要首次获取时可调度已审查的外部工具 |
| 回复建议 | 根据原消息核对最后发送者与语气，生成未发送草稿 |

Reader 只读数据，不发送、转发或删除微信消息。首次获取是独立的实验性流程，可能涉及管理员授权、微信退出/重开和临时副本，必须依据用户对具体操作的授权执行；不附带获取工具二进制。Windows 自动获取不由本仓库提供。

数据库 key、文本可读、媒体路径可定位、媒体内容可理解、官方微信恢复是不同验收项；本机记录也不等于手机全部历史。

## 缺少有效解密材料时

先验证已有材料、账号/目录、实际解释器依赖和权限，避免把所有错误归为 key 缺失。`diagnose --source ... --support-summary` 的候选结果位于 `data.material_preview.state/error_code`，与活动配置的 `data.state` 分开判断。

- [五步首次接入](references/access-onboarding.md)
- [故障诊断与恢复](references/access-troubleshooting.md)
- [实验性接入助手](references/experimental-access.md)
- [Windows 分阶段接入](references/windows-access.md)
- [配套安装与解密排障 Skill](https://github.com/a1271981054/wechat-hub-setup-recovery)：提供真实降权、登录会话等问题的固定基线补丁及脱敏案例。

## 仓库结构

```text
SKILL.md                         Codex Skill 主入口
scripts/                         Reader / 接入助手的 Skill 包装入口
references/                      查询、接入、排障及 Windows 指引
projects/rion-wechat-reader/      可独立安装的 Reader、助手、文档与测试
providers/wxkey/                 可选候选补丁和准备工具；不自动下载或执行
LICENSE                          上游 AGPL-3.0 许可
```

仓库根目录同时是可发现的 Skill 目录。包装入口已适配这个独立布局；原有显式 CLI 覆盖、PATH 入口和安装后的运行环境仍按原顺序解析。

## 测试

使用带 SQLCipher/Zstandard 的隔离 Python，从仓库根运行：

```bash
python3 -m unittest discover -s projects/rion-wechat-reader/tests -p 'test_*.py'
python3 -m unittest discover -s providers/wxkey/tests -p 'test_*.py'
```

依赖缺失或平台不匹配的测试会跳过，不能把跳过称为能力通过。测试使用虚构数据或模拟系统调用；真实微信接入另需授权和实际文本抽检。不要把直播/实机对照脚本当成普通测试自动执行。

## 公开包验证记录（2026-11-06）

- Reader 测试共 161 项，其中 159 项通过、2 项跳过；provider 准备工具的 3 项测试通过。
- 在临时 prefix 安装 CLI，验证 Skill 包装入口、源码回退入口及已有安装拒绝覆盖。
- 使用现有依赖环境完成虚构数据库 SQLCipher/Zstandard 自检；本轮未重新验证网络下载依赖的过程。
- Skill frontmatter、文档相对链接、Shell 语法与公开文件清单检查通过。

这些检查使用隔离测试数据，没有重新接入微信或获取真实密钥；不能作为新机器实读或 Windows 真机获取的证明。

## 来源与本地修复

源代码基线为上游提交 [`fb6a42c34160d2ffa66ed5f114f8d8cdabe7f5f9`](https://github.com/Rion-Wu-tech/wechat-intelligence-hub/tree/fb6a42c34160d2ffa66ed5f114f8d8cdabe7f5f9)，Reader 标识为 `0.9.2-preview.2`。本仓库保留了本地已核对的接入助手修复：完整 worker 通过 `launchctl asuser` 进入调用者登录会话、恢复检查兼容 macOS 负 UID，并保留身份核验、恢复锁和失败停止逻辑。

Skill 包装脚本仅调整独立仓库布局的源码回退路径；文档中的相对链接已重定位。Reader 与接入助手核心在本次拆分中保持与核对过的本机源码一致。上游通用候选的成熟度和单机成功案例分开记录，不承诺任意微信版本一键接入。

[AGPL-3.0 许可](LICENSE)覆盖对应上游组件；wxkey 来源的许可文本保留在 [providers/wxkey/LICENSE.upstream](providers/wxkey/LICENSE.upstream)。
