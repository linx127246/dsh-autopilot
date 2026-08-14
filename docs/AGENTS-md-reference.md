# 工作区指令（AGENTS.md）

## 视觉能力（重要：主动使用，无需询问用户）

本机已安装 DeepEye 视觉插件（dsh-plugin-deepeye），配置智谱免费模型 glm-4v-flash
（API Key 在环境变量 DEEPEYE_API_KEY，用户级）。你有以下工具，**做项目时应当主动使用**：

- vision_describe — 详细描述图片（布局/元素/颜色/问题）
- vision_ocr — 提取图片文字（OCR）
- vision_ask — 针对图片回答问题
- vision_layout — UI 布局结构化分析（返回 JSON）
- vision_clipboard — 读取并分析系统剪贴板里的最新图片

图片来源支持：本地路径、http(s) URL、base64 data URI。

## 项目中的主动截图工作流（用户已授权，直接做，不要问）

1. **UI 验证**：完成页面/组件/界面修改后，用截图验证渲染效果、布局、样式
2. **排错**：遇到报错、异常界面，截屏并分析定位
3. **OCR 提取**：从设计稿/文档截图提取文字
4. **读图表**：架构图、流程图、数据分析图直接理解

### 截图方法
- 用工作区脚本：`powershell -NoProfile -ExecutionPolicy Bypass -File screenshot.ps1 [可选文件名]`
  （保存到工作区 screenshots\ 目录，带时间戳）
- 或剪贴板截屏（Win+Shift+S 后）直接调 vision_clipboard
- 截图后用 vision_describe / vision_ocr 分析该文件


## 浏览器自动化（dsh-tool-browser，已安装）

你有完整的浏览器控制工具（Playwright + 本机 Edge，有头模式，操作可见）：

- browser_navigate / browser_snapshot / browser_tabs / browser_switch_tab / browser_open_tab / browser_close_tab / browser_back
- browser_click / browser_type / browser_fill_form / browser_select_option / browser_hover / browser_drag / browser_press_key
- browser_evaluate（执行页面 JS）/ browser_console_messages（读控制台报错）
- browser_take_screenshot（截图存到工作区 screenshots\）/ browser_wait_for / browser_resize

**项目中的用法**：登录网站填表单、抓取页面数据、验证页面渲染效果、调试控制台报错。
浏览器截图后可用 vision_describe 分析页面效果（视觉 + 浏览器组合）。
注意：所有会话共享同一个浏览器页面；需要隔离时用独立 profile。


## 核心工作方法论（最高优先级）：先侦察，选最优方法，再动手

**不要急着动手，先花几秒钟想清楚用什么工具/路径最合适。** 这条原则适用于一切任务，
尤其是控制外部系统（应用、浏览器、设备）时：

1. **先侦察目标的技术形态**，再选控制方式：
   - 网页/浏览器页面 → 用浏览器 DOM 工具（browser_*）或 CDP，元素级精准操作
   - 桌面应用是 Electron（目录里有 app.asar / chrome_*.pak / LICENSE.electron.txt）→
     用 CDP 远程调试（--remote-debugging-port）直接读 DOM、元素.click()，绝不猜坐标
   - 原生 Win32 应用 → 优先 UI Automation / WM 消息 / 快捷键
   - 只有无 API 的纯绘制界面（游戏、自绘 UI）→ 才用截图+视觉定位+坐标点击（最后手段）
2. **确认登录态、权限、前置条件**，避免白忙活
3. **动手前先小步验证**（探测式操作），确认路径有效再推进
4. **一条路走不通，停下来重新侦察**，不要硬磨；及时止损比坚持错误路径重要

经典反面教材：2026-08 控制哔哩哔哩桌面客户端时，未先检查应用类型，直接用了
"截图→视觉定位→坐标点击"的通用方案，视觉模型读数不准+用户切窗口导致截图不稳定，
浪费大量时间；后来发现它是 Electron，用 CDP 一次成功。先花 10 秒查安装目录就能避免。


## Superpowers Skill 挂钩（最高优先级：每个任务的第一步）

已安装 Superpowers 14 个技能（`~/.dsh/skills/`）。**规则：**

1. **任何任务的第一步**：先调用 `skill` 工具检查技能目录——只要任务与某个 skill 的
   描述匹配（哪怕只有 1% 可能），就必须先加载它再行动，没有商量余地（见 using-superpowers）
2. **创造性工作**（新功能/新项目/改行为）：先加载 `brainstorming`——先澄清需求、给设计、
   **等用户批准后才动手**（HARD-GATE：未经批准禁止写代码）
3. **需求明确的多步任务**：加载 `writing-plans` 写实施计划 → `executing-plans` 执行
4. **实现功能/修 bug**：加载 `test-driven-development`（先写测试）或 `systematic-debugging`（先定位根因）
5. **声称完成之前**：加载 `verification-before-completion`——先运行验证命令、确认输出，
   再下结论（证据先于断言，绝不空口说"完成了"）
6. **优先级**：进程类 skill（brainstorming / systematic-debugging）先于实现类 skill

反面教材：2026-08 钢琴块任务未先加载 brainstorming，直接写代码，导致需求（旋律来源、
用户环境适配）反复返工，用户多次不满。先走流程能省掉 80% 的返工。



## 项目任务挂钩（最高优先级）：先加载 project-autopilot

已安装 project-autopilot 技能（`~/.dsh/skills/project-autopilot/`）。**规则：**

1. **用户交代项目级任务**（多步骤、要交付物："做个X"/"开发X"/"帮我建个X"/"做一个系统"）
   → **第一步必须加载 `project-autopilot`**，按六阶段流水线全自动推进：
   意图挖掘 → 资源侦察 → 方案自检 → 并行执行 → 验证迭代 → 交付
2. 单步问答/闲聊/一次性小事 → 不触发，正常处理
3. 该技能会交叉引用 brainstorming / find-skills / writing-plans / executing-plans /
   subagent-driven-development / systematic-debugging / verification-before-completion，
   按需加载子技能，只编排不重写


## 多通道验证原则（用户强调，必须遵守）：不要单一通道空转

**验证状态/推进任务时，同时使用多个反馈通道，避免只靠一种方式反复轮询。**

典型场景（控制外部系统时）：
- 通道 A：DOM/CDP/API 查询（精确但可能因加载/导航暂时返回空）
- 通道 B：截图 + 视觉分析（看到用户看到的画面，能发现 DOM 查询漏掉的模态框、canvas、加载态）
- **DOM 查询返回空或不确定时，立刻截屏用视觉交叉验证，而不是盲目重试**
- 正向操作（点击/输入/搜索）后，用截图快速确认界面确实变了，再继续下一步

延伸到处做方案/调研：
- 不要只从一个视角看问题；技术可行性、成本、用户体验、维护成本多角度交叉验证
- 信息不确定时，用多个来源（文档、实测、搜索）互相印证，别信单一来源

反面教材：2026-08 控制哔哩哔哩客户端搜索时，只轮询 DOM 判断结果是否出现，
用户已看到搜索完成但 DOM 返回空，我反复报告"未检测到"，速度慢且体验差。
若在 DOM 返回空时立刻截图看一眼，一次就能确认。

## 做方案的流程（用户强调，必须遵守）

当用户让你"做一个方案/设计/计划"时，按以下流程走：

1. **先澄清需求**：目标、成功标准、约束（时间/预算/技术栈/平台）、边界（不做什么）、受众
2. **侦察调研**：查现有方案/开源项目，别重复造轮子；关键风险点先小实验验证
3. **多方案设计**：至少 2-3 个选项，注明权衡（成本/复杂度/可维护性/风险），明确推荐+理由
4. **关键决策点让用户拍板**：只问真正的分歧点，给选择题+默认推荐，不让用户做填空题
5. **输出结构化方案**：目标/范围 / 技术选型 / 架构 / 实施步骤 / 里程碑 / 风险应对 / 验收标准 / 迭代计划
6. **小步验证**：先 MVP 后迭代，执行中发现问题及时修正，方案是活的

原则：先花 20% 时间把需求和路看清，省后面 80% 的返工。

## 已知事项

- git push 到 GitHub：沙箱下 PortableGit 的 sh.exe 会崩（CreateFileMapping 被拒），需用 danger-full-access 权限执行；且必须配置 `git config core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"`（正斜杠！反斜杠会被 sh 吃掉）
- 免费模型有限流（429），重试即可；高峰可把 cordis.patch.yml 的 model 临时换成 glm-4v-flash
- vision_clipboard 在插件 v0.1.0 上打过本地补丁（node_modules 里的 index.mjs），
  **升级插件后需重新打补丁**（详见 deepeye-setup.md）
- 智谱免费档：glm-4v-flash（默认）、glm-4.6v-flash（更强但高峰 429 多）
- 隐私：识图会把图片发给智谱服务器；截屏会包含屏幕内容，注意场合
