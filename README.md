# GML 练习记录站

一个**单文件**的 GameMaker / GML 练习记录本：把写过的 GML 代码、踩过的坑、用过的官方 API 记下来，
还能让 AI 直接读这个页面、帮你把一句「玩家无敌时间不对」定位到具体文件和行号。

- **打开就能用**：整个站点就是一个 `index.html`，双击即开，不需要服务器、不需要注册、不联网也能用。
- **数据在你自己浏览器里**：localStorage 存储，页面里没有任何上报逻辑。
- **给 AI 用的接口**：页面内置一份 AI skill，并暴露 `window.gml.cmd(...)` 命令接口，AI 不必去抠 DOM。

在线版：<https://basetier-gml.github.io/gml-practice-log/>

---

## 快速开始

```bash
# 1. 直接打开（推荐）
open index.html          # macOS
xdg-open index.html      # Linux
start index.html         # Windows

# 2. 或者起个本地静态服务
python3 -m http.server 8080
# 然后访问 http://localhost:8080/
```

打开是**空站**（不带任何记录）。两个入口：

- **菜单 ▾ →「新手教程」**：一键加载 6 条用法说明记录 —— 三种用法各一条、API 库怎么用、
  站点怎么操作，外加一条「点菜单里的清空，正式开始使用」。
  说明会**按当前界面语言**加载（中文 / English 各一版），而且**就是打卡记录**，
  和你自己写的记录长得一样；看完在菜单里点「清空缓存」删掉即可。
- **「＋ 打卡一次」**：直接开始记自己的练习。

### 菜单 ▾ 里有什么

| 菜单项 | 作用 |
|---|---|
| 改名字 | 改昵称 |
| 换头像 | 换头像（只存在本机） |
| 存档 | 导出整站存档 |
| 读档 | 导入整站存档（只有覆盖，两次确认） |
| **清空缓存** | 清掉这台设备的全部数据 + 浏览器缓存（两次点击确认） |
| **新手教程** | 加载 6 条用法说明记录（按界面语言） |
| 切换到 English | 中英切换，切换后加载的新手教程也换成对应语言 |
| 分享打卡记录截图 | 出 1080 长图 |

菜单从上到下的顺序就是上面这个顺序（语言切换排在「分享」上面）。

⚠ **清空缓存是破坏性操作**：会删掉全部记录、API 库、昵称头像和设置，并清空浏览器缓存。
第一次点只会变成「再点一次确认清空」，4 秒后自动解除。
清空后刷新**不会**把新手教程灌回来（会打上「已清空」标记）。

---

## 三种工作模式

站点支持三种用法，三条纪律各有侧重：

### ① 打卡模式 —— 录一份写好的成品

把已经做完的东西录进来：描述 + 代码 + 用到的官方 API。

```js
await window.gml.cmd('checkin', { name:'俯视角射击 demo（成品）', requirement:'…', hint:'…' })
await window.gml.cmd('addModule', { type:'脚本', value:'// 文件名\n<代码>' })
await window.gml.cmd('addApi',   { code:'draw_text', desc:'画一行字' })
await window.gml.cmd('refApi',   { code:'draw_text' })
await window.gml.cmd('finish')
```

- 官方 API：**登记 + 引用**。
- 自写类 / obj：**只登记它自己**（`addApi`），**不做跨记录引用** —— 成品没有「按环节演进」的过程。

### ② 日志模式 —— 按功能环节记开发过程

一个功能环节 = 一条记录，**测通过了才打卡**。这时额外有「定义它的环 / 用到它的环」之分：

- 自写类 `class_hp`：**定义它的环 + 用到它的那些环，都要 `refApi` 引用它**；
- 被创建的对象 `obj_player` 同样登记并被引用，方便反查「哪个 obj 被哪些记录用过」；
- 类的 static 方法（`class_gun.fire`）**默认不单独登记**，只有出现在自己文件之外（类外复用）才登记；
- **每打一次卡立刻 `export` 存一次档**（防中途丢），写完用 `get` 读回核对。

### ③ 教程模式 —— 一个操作步骤一条记录

把一段教程/课程整理成可跟着做的步骤，每步一条记录：

- **题目要求只写要求**（要做成什么样 + 怎么验收），**不写操作步骤、不贴代码**；
- **操作步骤 = 多段「操作描述」载体 + 代码 / 对象载体**，按步骤顺序**交错**排列
  （先说这一步做什么，紧接着给这一段的代码）；
- **操作描述只说动作，不贴代码**：赋值语句和 API 调用一律归代码 / 对象载体；
- **正文要主动分段**：一个意思一行，别让浏览器在句子中间折断。

### 三条通用纪律

1. **先测通、再打卡** —— 先打卡后补测，记录里就混进了没验证过的环节；
2. **每打一次卡立刻 `export`**；
3. **写完 `get` 读回核对** —— 写没写错，机器说了算。

---

## 命令接口（给 AI 用）

页面加载完就有 `window.gml.cmd(name, args)`，返回 `{ ok:true, ... }` 或 `{ ok:false, error:'…' }`。

| 命令 | 作用 | 参数 |
|---|---|---|
| `state` | 看当前状态（昵称 / 设置 / 记录列表 / API 列表 / 当前打开的练习） | — |
| `checkin` | 打卡一次，建一条记录 | `{name, requirement, hint}` |
| `set` | 改当前记录的字段 | `{name?, requirement?, hint?}` |
| `addModule` | 加一个代码载体 | `{type, value}`，`type` ∈ 脚本 / 对象 / 操作描述 / SVG绘图 / 超链接 / 着色器 |
| `addObject` | 加对象载体 | `{name, fields:[], events:[{name, code}]}` |
| `addApi` | 登记一个 API 条目 | `{code, desc, params?}` |
| `refApi` / `unrefApi` | 引用 / 取消引用一个 API | `{code}` |
| `get` | 读回一条记录全文 | `{id}` 或 `{name}` |
| `find` | 把一句反馈定位到具体代码位置 | `{q}` |
| `remove` | 删一条记录（破坏性，必须指明 id 或 name） | `{id}` 或 `{name}` |
| `export` / `import` | 整站存档导出 / 导入（导入只有覆盖，两次确认） | `import: {json}` |
| `exportPractice` | 导出某一条练习（含它引用的 API） | `{id}`（**不传就是当前打开的那条**） |
| `shareLink` | 生成分享链接（网址 + `#gmlp=<压缩包>`） | — |
| `screenshot` | 生成练习过程长图（1440px，二维码固定在开头） | — |
| `setNick` / `setAvatar` | 改昵称 / 头像 | `{name}` / `{dataUrl}` |
| `lang` | 切换中英 | `'zh'` / `'en'`（或 `{lang:'en'}`） |
| `finish` / `close` | 结束编辑（弹层不关） / 关闭弹层 | — |
| `clear` | 清空整站（**只在用户明说重开时用**） | — |

### `find` —— 把反馈定位到代码

```js
await window.gml.cmd('find', { q:'cd_max' })
// → { ok:true, q:'cd_max', total:3,
//     records:[{ id, name, hits:[{ where:'class_gun.gml > fire', file:'class_gun.gml', line:28, text:'…' }] }],
//     apiHits:[{ code:'class_gun', desc:'…', where:'code' }] }
```

子串匹配、大小写不敏感，同时搜要求 / 提示 / 脚本载体 / 对象事件 / API 库。
返回的 `where` 优先给「文件 > 类 > 方法」的路径，`file` 是工程里的文件名，`line` 是行号。

---

## 内置 AI skill

页面里内嵌了一份完整的 AI 使用说明（`#gml-ai-skill`，markdown 原文），与仓库里的 `AI-SKILL.md` 同源同内容。

```bash
# 取出来单独读
python3 - <<'EOF'
import re, io
h = io.open('index.html', encoding='utf-8').read()
m = re.search(r'<script type="text/markdown" id="gml-ai-skill">(.*?)</script>', h, re.S)
print(m.group(1))
EOF
```

skill 里写死了这次迭代出来的口径（登记与引用纪律、static 方法口径、obj 登记、组合模式、
一类一记录、find 流程、主动分段规则等），AI 读完就能按同一套规矩干活。

---

## 数据与隐私

- **全部存在浏览器 localStorage**，键名前缀 `gml-practice-log-v2:`，分 `user` / `settings` / `records` / `days` / `api` / `apiIndex` 六片。
- **没有任何上报 / 统计 / 广告代码**，断网可用。
- 导出是纯 JSON 存档（`{app, version, exportedAt, counts, data}`），可以直接备份、可以贴给 AI 看。
- 换浏览器 / 换设备：用 **导出 → 导入** 搬过去（导入只有覆盖，会提示两次）。
- 昵称和头像**不会被语言切换替换**（不会被 i18n 遍历器误伤）。

---

## 目录结构

```
gml-practice-log/
├── index.html          # 站点本体（单文件，含内嵌 skill）
├── AI-SKILL.md         # 给 AI 的 skill（与页内块同源）
└── README.md           # 本文件
```

带记录的示例站（各自内嵌一份种子存档）：

```
gml-practice-log-mode-checkin/    # 打卡模式示例
gml-practice-log-mode-log/         # 日志模式示例
gml-practice-log-mode-tutorial/    # 教程模式示例（B 站 GameMaker 教程整理的 9 条记录）
```

---

## 部署

纯静态站点，扔哪都能跑：

- **GitHub Pages**：把 `index.html` 覆盖上传到仓库根目录即可；
- **对象存储 / CDN**：上传 `index.html` 即可，不需要构建；
- **子目录部署**：站点没有写死绝对路径，放在 `/foo/bar/` 下也能正常工作。

> 上线前建议先在本地用 `file://` 打开确认一遍（存档是浏览器本地的，换域名不会自动同步数据）。

---

## 开发与测试

仓库里带了一套 Playwright 回归测试（沙箱内 `python3 work/test_xxx.py` 直接跑）：

| 套件 | 覆盖 |
|---|---|
| `work/test_modes.py` | **三种工作模式**的完整链路与各自纪律 |
| `work/test_regression.py` | 站点基础行为 |
| `work/test_backup.py` / `test_practice_backup.py` | 整站 / 单条存档 |
| `work/test_storage_split.py` | 存储分片与迁移 |
| `work/test_bruteforce.py` | 脏数据、超大数据、多标签并发、配额写满 |
| `work/test_ai_skill.py` / `test_gml_cmd.py` / `test_find_cmd.py` | skill 一致性、命令接口、find |

Playwright 需要显式指定浏览器路径：

```bash
PW=/root/.cache/ms-playwright/chromium-1243/chrome-linux/chrome
python3 work/test_modes.py
```

---

## License

MIT
