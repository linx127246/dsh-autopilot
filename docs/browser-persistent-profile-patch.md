# dsh-tool-browser 持久化登录态补丁说明

## 问题

dsh-tool-browser 插件默认使用 `chromium.launch({ channel: "msedge", headless })`——**临时 profile**。
每次浏览器实例重启（关闭全部标签页/DSH 重启/插件重载），所有网站的登录态（cookie）全部丢失，
用户需要重新登录小红书/抖音/GitHub/豆包等。

实测：2026-02-16 关闭全部标签页后重开豆包，登录态丢失（页面重新出现登录按钮）。

## 修改点（lib/index.js，4 处）

文件：`~/.dsh/profiles/web/node_modules/dsh-tool-browser/lib/index.js`

1. **import**：`import { tmpdir } from "node:os"` → `import { homedir, tmpdir } from "node:os"`

2. **BrowserSession 构造函数**：新增 `this.context = null;`

3. **launchBrowser()**：`chromium.launch({ channel, headless })` →
   `chromium.launchPersistentContext(userDataDir, { channel, headless, viewport })`
   其中 `userDataDir = join(homedir(), ".dsh", "browser-profile")`（先 `mkdirSync` recursive）

4. **ensurePage()**：`this.browser = await this.launchBrowser()` →
   `this.context = await this.launchBrowser(); this.browser = this.context.browser();`
   检查条件加 `this.context === null`

5. **openTab()**：`this.browser.newPage({ viewport })` → `this.context.newPage()`（viewport 已在 launch 时设置）

6. **close()**：`this.browser.close()` → `this.context.close()`，同时清 `this.context = null`

## 如何重打补丁（插件升级后）

```powershell
# 1. 确认升级后回到旧代码（launch 无持久化）
Select-String -Path "C:\Users\lxx10\.dsh\profiles\web\node_modules\dsh-tool-browser\lib\index.js" -Pattern "launchPersistentContext"

# 2. 按上述修改点重新编辑（或用本文件的 diff 指导）
# 3. 语法检查
node --check "C:\Users\lxx10\.dsh\profiles\web\node_modules\dsh-tool-browser\lib\index.js"

# 4. 重启 DSH 使补丁生效，然后验证：
#    - 登录一个网站 → 关掉全部标签页 → 重开 → 登录态应保持
#    - ~/.dsh/browser-profile 目录应存在
```

## 验证清单（补丁生效后）

- [ ] `~/.dsh/browser-profile` 目录存在（首次 launch 自动创建）
- [ ] 登录豆包 → 关闭全部标签页 → 重开豆包 → 无需重新登录
- [ ] 其他网站（小红书/抖音/GitHub）登录态同样保持

## 备注

- 持久 profile 目录在 `~/.dsh/browser-profile`，如要彻底重置浏览器（清空所有登录态），删除该目录即可
- 本补丁与 vision_clipboard 补丁同理：**插件升级后会被覆盖，需重新打**
