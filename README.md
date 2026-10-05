# moban-web-source-demo

AnyReader（墨伴）**网络书源**与**书源订阅**的格式示例仓库。这里没有可编译的代码，只有可以直接导入 App、也可以照着仿写的 JSON。

> ⚠️ 示例中的域名（`www.example-biquge.com`、`m.example-jieqi.com`）是**占位地址**，无法真正联网看书；选择器取自笔趣阁 / 杰奇这类常见模板。仿写时按文末「改成你自己的书源」替换即可。

## 目录结构

```
moban-web-source-demo/
├── single/
│   ├── example-source.json        # 单书源：GET 搜索 + 发现分类（字段最全，推荐先看）
│   └── example-source-post.json   # 单书源：POST 搜索 + GBK 编码 + 自定义请求头
└── subscription/
    └── sources.json               # 书源订阅：裸数组形态（含上面两个源）
```

## 快速使用

### 方式一：添加书源订阅（一次导入，可更新）

1. 打开 App：**我的 → 书源管理**；
2. 点击右下角悬浮按钮，选择 **添加书源订阅**；
3. 粘贴订阅文件 URL（裸数组，多个书源放在一个 JSON 数组里）：

`https://raw.githubusercontent.com/Jason-wam/moban-web-source-demo/main/subscription/sources.json`

添加后会立即拉取一次并自动导入其中的书源；之后可在订阅上「立即更新」。

> 国内访问 GitHub 较慢时，可在 URL 前加任意可用的 GitHub 加速前缀，或改用自建 Gitee 镜像。

### 方式二：导入单个书源（JSON 文件）

三种途径任选其一：

1. **直接选择文件**：在 **书源管理** 页点右下角悬浮按钮 → **导入书源文件（JSON）**，选择 `single/` 下下载好的 JSON 文件；
2. **系统「打开方式 / 分享」**：在文件管理器中选中书源 JSON（可多选），用「打开方式 / 分享」发送到墨伴，自动解析导入；
3. **浏览器粘贴**：在书源管理页顶栏打开书源管理服务开关，同一 Wi‑Fi 下用浏览器访问 `http://手机IP:8894`，点 **导入** 粘贴 JSON 文本。

以上均支持单条对象、数组或带 `sources` 字段的订阅包，同源重复自动跳过。

> 不想用 JSON 时，也可以在书源管理页直接点 **新建书源**，用表单逐字段填写。

## JSON 字段说明

### 顶层字段

| 字段 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- |
| `ruleVersion` | 否 | `1` | 规则格式版本 |
| `bookSourceName` | 是 | | 书源显示名 |
| `bookSourceUrl` | 是 | | 站点根地址，作为书源唯一标识（会规范化：补 scheme、小写 host、去末尾斜杠） |
| `bookSourceGroup` | 否 | `""` | 分组名 |
| `enabled` | 否 | `true` | 是否启用 |
| `enabledExplore` | 否 | `true` | 是否启用「发现」 |
| `weight` | 否 | `0` | 权重，越大在列表中越靠前 |
| `charset` | 否 | 跟随响应头 | 强制编码，老站常见 `GBK` |
| `icon` | 否 | 自动抓 favicon | 图标：图片 URL 或 base64 |
| `bookSourceComment` | 否 | `""` | 书源简介 |
| `headers` | 否 | `{}` | 自定义请求头，如 `User-Agent` |
| `search` | 否 | 无搜索 | 搜索规则，缺省时搜索该源会提示「不支持」 |
| `explore` | 否 | `[]` | 发现分类列表 |
| `detail` | 是 | | 详情页规则 |
| `toc` | 是 | | 目录规则 |
| `content` | 是 | | 正文规则 |

### `search` / `explore`（列表页）

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `title` | 仅 `explore` 必填 | 分类显示名（搜索段忽略） |
| `url` | 是 | 列表页 URL 模板 |
| `method` | 否 | `GET`（默认）/ `POST` |
| `body` | POST 时 | 请求体模板 |
| `pageStart` | 否 | 首页页码，默认 `1` |
| `list` | 是 | 列表项选择器，命中多个节点，每项进入相对作用域 |
| `name` | 是 | 书名选择器 |
| `author` | 否 | 作者 |
| `detailUrl` | 是 | 书籍详情链接 |
| `coverUrl` | 否 | 封面 |
| `intro` | 否 | 简介 |
| `latestChapter` | 否 | 最新章节 |
| `nextUrl` | 否 | 页内「下一页」链接；为空时按 URL 中的 `{{page}}` 翻页 |

### `detail`（详情页）

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `url` | 否 | 详情页地址模板；留空表示直接用搜索给出的 `detailUrl` |
| `name` | 是 | 书名 |
| `author` / `coverUrl` / `intro` | 否 | 作者 / 封面 / 简介 |
| `tocUrl` | 否 | 目录页链接**规则**；留空表示目录就在详情页 |

### `toc`（目录）

| 字段 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- |
| `list` | 是 | | 章节项选择器 |
| `name` | 否 | `@text` | 章节标题 |
| `chapterUrl` | 否 | `@attr(href)` | 章节链接 |
| `isVolume` | 否 | `""` | 分卷判定（布尔出口），命中的条目会被跳过 |
| `nextUrl` | 否 | `""` | 目录多页的下一页规则 |

### `content`（正文）

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `content` | 是 | 正文容器：字符串或字符串数组（多个候选，按序取首个非空），出口必须为 `@html` |
| `nextUrl` | 否 | 章内分页「下一页」规则 |
| `remove` | 否 | 取到正文 HTML 后删除的节点选择器 |
| `replace` | 否 | 正则净化对 `[正则, 替换串]`，按序执行 |
| `imageStyle` | 否 | 当前固定 `text` |

## 选择器语法

规则字符串由 Document 框架解释：

- **默认 CSS（jsoup）**；也可用前缀切换引擎：`@XPath:` / `//` 走 XPath，`@Json:` / `$.` 走 JSONPath，`@Regex:` 走正则，`@Raw:` 为常量。
- `>` 链接成链式步骤；`A||B` 取首个非空结果；`A&&B` 合并结果。
- 属性出口：`@text`、`@attr(key)`、`@html`、`@outerHtml`、`@ownText`、`@hasClass(x)`、`@hasAttr(x)` 等。
- 索引控制：`@first`、`@last`、`@index(n)`、`@range(a..b)`。
- 文本后处理：`@trim`、`@clearText(待删文本)`、`@replaceText(旧,新)`、`@uppercase`、`@lowercase`。

URL / 请求体模板变量：

| 变量 | 含义 |
| --- | --- |
| `{{key}}` / `{{keyword}}` | 搜索关键词（按站点编码做 URL encode） |
| `{{page}}` | 页码 |
| `{{baseUrl}}` | 书源根地址 |

## 改成你自己的书源

1. 复制 `single/example-source.json`，改 `bookSourceName` 和 `bookSourceUrl`。
2. 用浏览器打开站点搜索页，观察提交后的地址（或表单），确定关键词参数与分页参数，改 `search.url`（或 `method=POST` + `body`）。
3. 在搜索结果页右键检查元素，确定列表项节点，填 `search.list` 及 `name` / `detailUrl` 等字段。
4. 依次打开详情页、目录页、正文页，用同样方法核对 `detail` / `toc` / `content`。
5. 页面乱码时加 `"charset": "GBK"`；正文有广告就往 `content.remove` / `content.replace` 里加规则。
6. 在 App 的书源管理里测试该规则，确认搜索 → 详情 → 目录 → 正文全链路。
7. 要做成订阅：把多条书源放进一个 JSON 数组，文件传到 GitHub/Gitee，把原始 URL 交给用户添加为订阅；以后更新该文件即可，用户在订阅上「立即更新」获取变化。
