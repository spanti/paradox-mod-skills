# Paradox Mod Skills

面向 **Crusader Kings III** 和 **Victoria 3** 模组开发的 Claude Code 技能合集。技能会优先查阅本机游戏文档、原版示例和新鲜日志，并区分静态检查与游戏内验证结果。

本仓库采用“根目录单插件 + 自托管 marketplace”结构，可直接通过 `claude plugin install` 安装，无需 MCP 服务或后台进程。

## 包含的技能

| 技能 | 用途 |
| --- | --- |
| [ck3-modding](skills/ck3-modding/SKILL.md) | CK3 脚本、事件、决议、本地化、兼容性与调试 |
| [ck3-gui](skills/ck3-gui/SKILL.md) | CK3 原生窗口、HUD、PdxGui 布局、数据绑定与交互 |
| [ck3-playtest](skills/ck3-playtest/SKILL.md) | CK3 游戏内检查清单、视觉验证、回归测试与证据记录 |
| [vic3-modding](skills/vic3-modding/SKILL.md) | Victoria 3 脚本、经济、政治、GUI、本地化与文档查询 |

安装后，技能带有插件命名空间，例如 `paradox-mod-skills:ck3-modding`。Claude 可根据任务自动调用，也可以由你明确指定。

## 安装

### 从 GitHub 安装

仓库发布后，将 `spanti` 替换为实际的 GitHub 用户名或组织名，然后执行：

```bash
claude plugin marketplace add spanti/paradox-mod-skills
claude plugin install paradox-mod-skills@paradox-mod-skills
```

第一条命令注册本仓库自带的 `.claude-plugin/marketplace.json`，第二条命令安装插件。

### 从本地目录安装

在本仓库根目录执行：

```bash
claude plugin marketplace add ./
claude plugin install paradox-mod-skills@paradox-mod-skills
```

也可以使用较新版本 Claude Code 支持的单命令形式：

```bash
claude plugin install paradox-mod-skills --marketplace ./
```

安装后重启 Claude Code 会话，并检查插件及技能清单：

```bash
claude plugin list
claude plugin details paradox-mod-skills
```

### 使用 skills.sh 安装（其他 Agent 或可编辑副本）

本仓库符合 Agent Skills 标准目录结构，`skills` CLI 可以直接发现并复制全部四个技能。仓库发布后，Claude Code 用户可执行：

```bash
npx skills@latest add spanti/paradox-mod-skills -a claude-code
```

将最后的 `claude-code` 替换为 `cursor`、`opencode`、`devin`、`windsurf`、`amp`、`pi` 等 Agent 名称，即可安装到对应客户端；省略 `-a claude-code` 则会在终端中交互选择。注意 Claude Code 的 Agent 标识是 `claude-code`，不是 `claude`。

这种方式会复制可编辑的技能文件，适合二次开发，但不会自动更新。它与 Claude Code 插件安装方式应二选一，避免同一技能重复加载。更新副本时运行 `npx skills@latest update`；如果上游新增了技能，重新运行上述 `add` 命令。

## 更新与卸载

```bash
# 更新 marketplace 元数据并升级插件
claude plugin marketplace update paradox-mod-skills
claude plugin update paradox-mod-skills@paradox-mod-skills

# 卸载插件（保留 marketplace，便于以后重新安装）
claude plugin uninstall paradox-mod-skills@paradox-mod-skills

# 如不再需要，移除 marketplace
claude plugin marketplace remove paradox-mod-skills
```

发布新版本时，递增 `.claude-plugin/plugin.json` 中的 `version`，让用户能够获取更新。

## 仓库结构

```text
paradox-mod-skills/
├── .claude-plugin/
│   ├── plugin.json          # 插件元数据与公开技能列表
│   └── marketplace.json     # 自托管 marketplace，source 指向仓库根目录
├── skills/
│   ├── ck3-modding/
│   │   └── SKILL.md
│   ├── ck3-gui/
│   │   └── SKILL.md
│   ├── ck3-playtest/
│   │   └── SKILL.md
│   └── vic3-modding/
│       └── SKILL.md
├── LICENSE
└── README.md
```

每个技能目录还可以包含 `references/`、`scripts/` 和其他按需加载的资源。不要把技能组件放进 `.claude-plugin/`；该目录只保存插件和 marketplace 清单。

## 添加或修改技能

1. 在 `skills/<skill-name>/SKILL.md` 中编写技能，并在 frontmatter 中提供 `name` 和 `description`。
2. 将技能目录添加到 `.claude-plugin/plugin.json` 的 `skills` 数组；发布时同步递增 `version`。
3. 在 README 的技能表中补充用途说明。
4. 在仓库根目录运行严格校验：

   ```bash
   claude plugin validate . --strict
   ```

5. 通过“本地目录安装”流程确认插件能够安装，且 `claude plugin details paradox-mod-skills` 显示预期技能。

## 来源与许可证

技能内容参考 [JDeffner/paradox-ai-modding](https://github.com/JDeffner/paradox-ai-modding)，本仓库将其整理为扁平的根插件结构。插件打包方式参考 [mattpocock/skills 的 ADR](https://github.com/mattpocock/skills/tree/main/.agents/adr)。

本项目使用 [GPL-3.0](LICENSE) 许可证。游戏、模组项目、本机路径和验证工具由使用者提供；静态校验通过不代表游戏内行为已经验证。
