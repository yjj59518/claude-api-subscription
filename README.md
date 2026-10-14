# Claude API和Claude Code订阅区别一次讲清：按Token付费和包月套餐怎么选？Pro有必要买吗？为什么API Key会覆盖订阅计费？（附开通方式与立减$1优惠入口）

先把结论放在最前面：Claude API 和 Claude Code 背后跑的是同一批 Claude 模型，但它们根本不是一个层面的东西。API 是“原料”——你自己的程序直接向 Anthropic 的接口发请求，按 token 计费；Claude Code 是“工具”——Anthropic 官方的编程 agent，能读你的代码仓库、改文件、跑命令，付费方式既可以是 Pro/Max 订阅，也可以挂 API Key。

由此可以推出一条很实用的判断规则：**你自己在写代码，就用订阅跑 Claude Code；你的程序在调 Claude，就用 API 拿 Key。** 如果要把 agent 做成产品交给别人用，只能走 API Key 或云凭证，不能搭你个人订阅的便车。

大部分人搜这个问题，其实是被两件事困扰：一是不知道该把钱花在哪边，二是明明付着 Max 的月费，账单却在按 token 扣 API 的钱。下面按顺序说清楚。

## 先分清楚：API 是接口，Claude Code 是工具

Claude API 是最底层的模型访问方式。你在 Anthropic Console 创建一个 API Key，自己的代码通过 Messages API 发请求，用多少 token 付多少钱；也可以不直连 Anthropic，改走 Amazon Bedrock、Google Cloud、Microsoft Foundry 这类云平台接入。

Claude Code 不是另一个模型。你在里面选 Sonnet 5 还是 Opus 5.5，背后请求的都是和 API 完全相同的模型，区别全在模型外面套的那一层：agent 循环、文件读写、命令执行、权限管理、会话和上下文管理都是现成的，你只管提需求。

它的使用位置也比很多人以为的广。终端是主阵地，但 VS Code、Cursor 等 VS Code 系 IDE、JetBrains 系列（IntelliJ、PyCharm）都支持，桌面应用和网页端同样覆盖，而且都算在同一份订阅额度里。

|  | Claude API | Claude Code |
| --- | --- | --- |
| 本质 | 通过 Messages API 和各类 SDK 的编程访问 | 能读仓库、改文件、跑命令的编程 agent |
| 谁来搭 agent 循环 | 你自己：写提示词、定义工具、处理重试 | 官方做好：工具、权限、会话、上下文管理内置 |
| 计费 | 按 token，走 Console 或云平台账单 | Pro/Max/Team/Enterprise 套餐额度，或挂 Key 按 token |
| 适用条款 | 商业条款 | Free/Pro/Max 按消费者条款；API 与 Team/Enterprise 按商业条款 |
| 用量在哪看 | Console 的 Usage 页面 | 会话内 /usage 查看套餐额度条 |

顺带说一个常见误会：很多人嘴里的"Claude Code API"其实混着两个意思。有人指“用 API Key 跑 Claude Code”，这是凭证选择问题；有人指“在自己的程序里调用 Claude Code”，这是 `claude -p` 命令和 Agent SDK 的事。两种都存在，后文都会讲到。

## 订阅怎么收费：目前全部套餐一览

| 套餐 | 价格 | 计费周期 | Claude Code | 额度机制与限制 | 开通方式 |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | — | 不包含 | 基础聊天额度，与各端共享 | 官网直接注册 |
| Pro | $20/月；年付折合 $17/月（一次付 $200） | 月付或年付 | 包含 | 5 小时会话窗口 + 周限额，与聊天共享 | Anthropic 官方订阅渠道 |
| Max 5x | $100/月 | 月付 | 包含 | 约为 Pro 的 5 倍用量 | Anthropic 官方订阅渠道 |
| Max 20x | $200/月 | 月付 | 包含 | 约为 Pro 的 20 倍用量；Fable 另有单独周限额 | Anthropic 官方订阅渠道 |
| Claude API（官方按量） | 按 token 计费 | 按量 | 可作为凭证使用 | 用多少付多少；agent 用法消耗高 | Console 创建 Key 并充值 |
| 第三方 API 通道（本文合作平台） | 充值按量使用，优惠码 ACCPAY 立减 $1 | 按量 | 作 API 凭证使用 | 具体规则以平台页面为准 | [ 开通入口（立减 $1）](https://bewild.ai/?code=ACCPAY) |

说明一下：价格均不含税；表中订阅套餐需在 Anthropic 官方渠道办理，文中优惠入口指向本文合作平台，用途是按量开通 API 额度，规则以平台页面为准。

订阅额度有几个机制值得单独记一下：

- 限制是“5 小时滚动会话窗口 + 每周限额”的双层结构，Max 套餐对 Fable 还有一条单独的周限额。
- 订阅额度和网页、桌面、手机端的聊天共享同一个池子，在 IDE 里用 Claude Code 也算在内，快到限额时会有提示。
- 额度见底时你有三个选项：等重置、升级套餐，或者开启 usage credits 按 API 价格继续用。

usage credits 这个词后面还会再出现，它和 Console 里的 API credits 是两回事，混起来容易花双份钱，这里先埋个伏笔。

## API 按量付费：钱怎么算

API 完全按 token 计费。参考量级（具体以官方定价页实时价格为准）：中端的 Sonnet 档模型大约输入 $3、输出 $15 每百万 tokens；旗舰的 Opus 档大约 $5 / $25；轻量的 Haiku 档大约 $1 / $5。

听起来不贵，但 agent 式用法是个 token 吞金兽。每读一个文件、每收到一次工具返回、每段代码 diff 都会进入上下文。一个专注的编码小时可能烧掉几百万输入 tokens；干同样的活，agent 用法的 token 消耗可以是单次普通 API 调用的 5 到 20 倍。

两个省钱的开关值得打开：一是 prompt caching，重复出现的上下文（系统提示、常用文件）从缓存读取时按大幅折扣计费；二是 Batch API，非实时任务走批处理大约能省一半。

## 最容易踩的坑：API Key 会覆盖订阅

> 如果系统环境变量里躺着 ANTHROPIC_API_KEY，Claude Code 会优先用它计费——哪怕你付着 Max 的月费，会话也在按 token 扣 Console 账户的钱。

这不是玄学，Claude Code 检查凭证有固定顺序，大致从高到低是：云平台凭证（Bedrock/Vertex/Foundry 环境变量）→ ANTHROPIC_AUTH_TOKEN → ANTHROPIC_API_KEY → apiKeyHelper 脚本 → CLAUDE_CODE_OAUTH_TOKEN → Anthropic 账户联合凭证 → 你的订阅登录。订阅排在最后。

典型事故长这样：之前某个项目把 API Key 写进了 shell 配置文件，你某次被询问时随手批准了，之后所有会话都在按 token 计费。交互模式里可以用 /config 的 "Use custom API key" 开关改回来；但在 `-p` 非交互模式下没有任何提示，环境里只要有 Key 就一定用 Key。

恢复方法不复杂：

bash
unset ANTHROPIC_API_KEY
claude
# 会话内运行 /status


/status 会显示当前生效的登录方式，如果同时配置了订阅和 Key，它也会标出哪个没在被使用。记得把 shell 配置文件里的那行 Key 删掉，不然下个终端窗口它又回来了。另外桌面应用和网页端的云端会话不读这些环境变量，直接走 claude.ai 登录，不用担心。

再说说两种容易混淆的"credits"：

- **Usage credits**：订阅套餐的“溢出额度”，超出套餐限额后按 API 价格继续用，在 claude.ai 的 Settings > Usage 里管理。
- **Console API credits**：给 API Key 充值的钱，付的是你自己的 API 调用，对 Pro/Max 的限额没有任何加成。

到达限额时，Claude Code 会弹选项问你要不要用 API credits 继续。想严格守住订阅预算，就拒绝这个选项、等额度重置，平时用 /status 盯余量；如果压根不想看到这类提示，可以先 `claude logout`，再只用订阅凭证 `claude login`，登录过程中别添加任何 Console 凭证。

## 怎么选：按场景对号入座

1. **自己写代码、重构、调试**：Claude Code + 订阅。日常强度 Pro 够用，经常打满限额、仓库比较大，就升 Max 5x 或 Max 20x。
2. **给自己的应用加 Claude 功能**（聊天、摘要、信息抽取、分类这类）：Messages API + Key。这条路的钱订阅付不了，只能按量。
3. **把带文件和 shell 工具的 agent 发布给外部用户**：必须用 API Key 或云凭证。Anthropic 的政策写得很明白：第三方不能向用户提供 claude.ai 登录，也不能替用户把请求路由到 Free/Pro/Max 凭证上，而且官方保留不预告就执行的权利。
4. **CI 或脚本自动化**：用 `claude -p` 或 Agent SDK（Python/TypeScript）。想用订阅跑流水线，可以通过 `claude setup-token` 生成 OAuth token 挂到 Pro/Max 额度上，但要注意 `--bare` 模式不读它，而且官方说未来 `-p` 会默认进入 bare 模式——依赖订阅 token 的流水线最好提前准备好 API Key 的备用方案。

如果你落在第 2、3 类，或者是第 1 类但搞不定订阅的支付环节，那就属于 API 按量付费的场景。

## 开通 API 的两条路

官方路径：注册 Anthropic Console → 创建 API Key → 充值额度。功能最全、计费透明，但需要绑一张能用的海外支付方式，这一步卡住了不少人。

第三方通道：通过提供 Claude API 额度的平台充值使用，配置好 ANTHROPIC_API_KEY（或 ANTHROPIC_AUTH_TOKEN 加 ANTHROPIC_BASE_URL）之后，Claude Code 和你自己的程序都能直接跑，用法不变。挑平台时重点看两件事：你要用的模型是否可用、计费规则是否清晰。

本文合作平台 bewild.ai 目前有立减活动：[👉 从这个入口进入，使用优惠码 ACCPAY 开通可立减 $1](https://bewild.ai/?code=ACCPAY)。一美元不多，反正不要钱，顺手领上总没坏处。

## 常见问题

**买了 Pro 订阅，用 Claude Code 还要另买 API 额度吗？**

不用。Pro 及以上付费套餐都已包含 Claude Code，和聊天共享同一份额度。只有 Free 套餐不包含它。

**为什么我的 Claude Code 没走订阅，扣的是 API 的钱？**

十有八九是环境变量里的 ANTHROPIC_API_KEY 在起作用，它的优先级高于订阅登录。unset 掉、重开终端，再用 /status 确认一下当前登录方式即可。

**Claude Code 能当成 API 用吗？**

它本身不是 HTTP 端点。想在程序里用它的能力，路子有两条：`claude -p` 命令行调用，或者 Agent SDK 的 Python/TypeScript 包。注意 `--bare` 模式只认 API Key，不读订阅凭证。如果连 agent 运行时都不想自己维护，Anthropic 还有托管的 Managed Agents，通过 API 配置、由官方负责跑 agent 循环。

**订阅和 API 能同时用吗？**

能。Claude Code 可以在订阅和 Key 之间切换，很多重度用户的做法就是订阅打底、不够的部分再按量。关键是搞清楚当前生效的到底是哪个凭证，/status 一查便知。

**哪种方式更省钱？**

看用量结构。轻量、低频的调用，按 token 付费可能比 $20 月费划算；重度 agent 使用，订阅的固定上限明显更可控——照“一小时几百万输入 tokens”的消耗速度，一个月纯按量跑下来的账单超过顶配 Max 的 $200 并不意外。先估一下自己的用量，再决定钱花在哪边。
