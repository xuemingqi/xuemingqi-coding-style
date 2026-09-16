# Xuemingqi Coding Style

个人编码规范，涵盖通用代码风格、目录组织、Java Spring Web 与 MyBatis-Plus 实践，并保留完整的 Skill 结构。

核心原则：编写最小、清晰、可扩展的代码。消除重复，提取真正可复用的行为，通过命名、控制流与目录归属表达意图，避免过度抽象。

## 规范目录

| 文件 | 内容 |
| --- | --- |
| [SKILL.md](SKILL.md) | 适用范围、优先级、工作流程与检查清单 |
| [通用编码规范](references/code-style.md) | 命名、复用、类型边界、依赖注入、控制流、注释、格式与测试 |
| [目录组织规范](references/directory-style.md) | 组织轴、职责归属、包内聚与共享代码边界 |
| [Java Spring Web 规范](references/java-spring-web.md) | 身份上下文、JWT 与 Redis、OpenFeign、配置、模型与时间语义 |
| [Java Spring 与 MyBatis-Plus 规范](references/java-spring-mybatis.md) | 业务与数据库 Service 分层、Entity、Mapper、Lambda 查询与事务 |
| [Skill 展示配置](agents/openai.yaml) | 名称、说明与默认提示词 |

## 适用范围

在编写、修改或重构源代码（包括测试代码）时使用。只读分析、解释、审查、运行已有测试以及纯目录整理无需加载；任务转为代码修改时，再加载相关规范。

这些规范作为个人默认约定使用。发生冲突时，依次遵循：

1. 用户当前明确要求。
2. 仓库规则、格式化工具、静态检查与生成代码规则。
3. 相邻代码中一致的约定。
4. 本仓库提供的个人默认规范。

## 阅读方式

先阅读 [SKILL.md](SKILL.md) 与 [通用编码规范](references/code-style.md)，再按任务需要阅读目录组织或对应技术栈的规范。规范正文保留原始英文内容，本 README 提供中文导航。

## 多端安装与同步

将仓库直接克隆到个人 Skill 目录，让安装目录同时成为 Git 工作目录：

```sh
mkdir -p ~/.agents/skills
git clone https://github.com/xuemingqi/xuemingqi-coding-style.git ~/.agents/skills/xuemingqi-coding-style
```

已有同名目录时，先保留并比较本地修改，不要直接覆盖。个人 Skill 目录采用 [Codex 官方文档](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills) 中的用户级位置。

- **平时使用**：只读本地规范，不检查更新，不自动拉取。
- **让代理修改规范**：修改完成后，同步发布到此仓库，并核对远端内容。
- **主动要求更新**：在对应端执行拉取，更新后的文件直接作为本地 Skill 使用。
- **自行编辑文件**：保存本身不会自动上传，需要明确要求代理同步，或自行提交并推送。

主动拉取时执行：

```sh
git -C ~/.agents/skills/xuemingqi-coding-style status --short
git -C ~/.agents/skills/xuemingqi-coding-style pull --ff-only origin main
```

若有未提交修改或分支分歧，先处理本地工作再拉取。完整流程见 [GitHub 同步规则](references/github-sync.md)。
