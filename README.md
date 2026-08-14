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

## 开发方法

本项目技能按 [Superpowers writing-skills](https://github.com/obra/superpowers) 的 TDD 法开发：先跑无技能基线（RED）抓失败点，写技能后重跑验证（GREEN），REFACTOR 堵漏洞。测试场景与实测结果见 `docs/`。

## License

MIT
