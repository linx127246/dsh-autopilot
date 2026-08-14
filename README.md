# dsh-autopilot（AI 全自动项目交付工作流）

一套可复用的 AI Agent 技能集，让 AI 接到项目任务后**全自动**跑完：意图澄清 → 资源侦察 → 方案自检 → 并行执行 → 验证迭代 → 交付。核心诉求：除权限/账号等必要操作外，用户无需主动操作或提醒。

## 包含的技能

| 技能 | 作用 |
|---|---|
| **project-autopilot** | 六阶段全自动项目交付流水线（本仓库核心） |
| **find-skills** | 从开源技能生态（skills.sh）发现和安装现成技能（Windows 兼容版） |

## 安装

将 `skills/` 下的技能目录复制到你的 Agent 技能目录（以 Claude Code / DSH 为例）：

```bash
# 复制到用户级技能目录
cp -r skills/project-autopilot ~/.dsh/skills/
cp -r skills/find-skills ~/.dsh/skills/
```

并在工作区 `AGENTS.md` 中挂一条触发指引（可选但推荐）：

```markdown
## 项目任务挂钩（最高优先级）：先加载 project-autopilot
用户交代项目级任务（"做个X"/"开发X"/"做一个系统"）→ 第一步必须加载
`project-autopilot`，按六阶段流水线全自动推进。
```

## project-autopilot 六阶段流水线

```
项目任务 → 【分级 S/M/L】→ ①意图挖掘 → ②资源侦察 → ③方案自检
        → ④并行执行 → ⑤验证迭代 → ⑥交付 → 【复盘】
```

1. **意图挖掘**：反诘式澄清（始终尖锐，对方案不对人），三层六问，Mom Test（问过去行为），用户失联时简报草案模式防死锁
2. **资源侦察**：六维度双语信源（现有方案/开源库/技能/文档API/设计资产/用户之声-B站小红书抖音），时间盒分级，关键选型 Spike 实测，合规前移
3. **方案自检**：2-3 个真实差异方案+推荐，自检五连问，L 级派红队挑刺
4. **并行执行**：契约先行，子代理并行（并发≤2+429退避），验证不信任（贴证据+抽查+截图）
5. **验证迭代**：证据先于断言，完成度五维审查（功能/质量/体验/视觉/健壮），独立子代理扮演刁钻用户挑刺，降级必须问用户
6. **交付**：六连清单+合规复核，五段汇报，签字门（用户确认才算完成）

贯穿机制：自主原则六条 / 问题预算 / 冲突仲裁优先级表 / 需求变更管理 / 失败协议（三终态）/ 检查点恢复。

## 文档

- `docs/2026-02-16-project-autopilot-design.md` — 完整设计文档（含三路红队裁决）
- `docs/project-autopilot-pressure-scenarios.md` — TDD 压力测试记录（RED 基线 vs GREEN 验证）
- `docs/AGENTS-md-reference.md` — 工作区 AGENTS.md 参考副本（路径已脱敏为 %USERPROFILE%）

## 灾难恢复指南（DSH 破坏性更新 / 换机器 / 重装系统）

本仓库是整套工作流的"备份箱"。恢复三步：

```bash
# 1. 克隆仓库
git clone https://github.com/linx127246/dsh-autopilot.git

# 2. 恢复技能（技能目录路径以新版 DSH 为准，先侦察新目录约定）
cp -r skills/project-autopilot ~/.dsh/skills/
cp -r skills/find-skills ~/.dsh/skills/

# 3. 恢复工作区方法论（按 docs/AGENTS-md-reference.md 重建 AGENTS.md，
#    把 %USERPROFILE% 替换回你的用户目录）
```

### 恢复后必须适配的 3 处（环境相关）

1. **技能目录路径**：新版 DSH 若改了技能目录约定，先侦察再复制（`~/.dsh/skills` 是当前版本约定）
2. **npm 缓存路径**：`SKILL.md` 操作层里的 `$env:npm_config_cache = "...\.npm-cache"` 是沙箱环境路径，换环境后改成你机器上可写的目录
3. **浏览器登录态**：小红书/抖音侦察需要在新环境的自动化浏览器里重新登录一次

### 恢复后建议

- 跑一次项目任务验证技能生效（先小步验证）
- 技能里"操作层细节"章节如与实际环境不符，顺手更新并 push 回本仓库

## 开发方法

本项目技能按 [Superpowers writing-skills](https://github.com/obra/superpowers) 的 TDD 法开发：先跑无技能基线（RED）抓失败点，写技能后重跑验证（GREEN），REFACTOR 堵漏洞。测试场景与实测结果见 `docs/`。

## License

MIT
