# catalog.json schema 2

`tools/normalize_catalog.py` 生成，App 解析。字段缺省值写在括号里；未列出的字段 App 必须忽略（向前兼容）。

```jsonc
{
  "schema": 2,
  "updated": "2026-10-11",                 // 目录发布日期
  "currencies": [
    {"id": "chase_ur", "label": "Chase Ultimate Rewards", "default_cpp": 1.5}
    // cpp = cents per point，用户可在设置里覆盖；cash 类 currency 的 default_cpp 为 1.0
  ],
  "categories": [
    {"id": "hotels", "label_zh": "酒店", "parent": "travel_general",  // parent 可为 null
     "aliases": ["hotel", "lodging", "住宿"]},   // 可选（[]，categories 和 brands 都可写）：查卡搜索框的
                                           // 同义词，中英文都行。搜索词不分大小写，是 label_zh、id、aliases
                                           // 任一个的子串就算匹配；也匹配直接上级类别的 label_zh 和 aliases
                                           // （「酒店」「hotel」都列出万豪）。旧名 search_terms（v1.0 目录的
                                           // everything_else 上有）App 仍照 aliases 搜，新目录不再写
    {"id": "rent", "label_zh": "房租与房贷", "parent": null,
     "no_base_fallback": true},            // 可选（false）：这一类（及其子类、子品牌）不按 base_rate 算，
                                           // 没有专属行的卡不参与排序
    {"id": "everything_else", "label_zh": "其他消费", "parent": null,
     "aliases": ["其他", "一般", "other"]}   // 兜底类别：查它时每张卡按 base_rate 排序，另加显式的
                                           // everything_else 行（目录里的这类行在转换时并进 base_rate，
                                           // 用户的 custom_earn 可以挂在这里）
  ],
  "brands": [
    {"id": "brand:marriott", "label_zh": "万豪", "parent": "hotels",   // parent 必须是 categories 里的 id
     "aliases": ["marriott bonvoy", "bonvoy"]},
    {"id": "brand:costco", "label_zh": "好市多", "parent": "shopping",
     "accepted_networks": ["visa"]}        // 可选（null = 都收）：商家收的卡组织，其余卡不参与排序
  ],
  "benefit_families": [                    // 福利页的分区（只供浏览，不合计金额）；必须有 other
    {"id": "rides_delivery", "label_zh": "叫车外卖"},
    {"id": "other", "label_zh": "其他"}
  ],
  "benefit_types": [                       // 一个类型 = 完全同一种福利（跨卡）。只有同一笔消费或同一个余额
                                           // 能吃下其中任一张卡的这项福利（同一商家或计划、同一机制）时才同类型：
                                           // 几张 Platinum 的 Uber Cash 进同一个 Uber Cash 余额，同类型；
                                           // Uber One 是会员费报销，另一类型。金额合计、组级记录、「我能用几份」、
                                           // 提醒都只在一个类型内算，从不跨类型
    {"id": "uber_cash", "label_zh": "Uber Cash", "benefit_family": "rides_delivery"},
    {"id": "uber_one", "label_zh": "Uber One 会员", "benefit_family": "rides_delivery"},
    {"id": "trusted_traveler", "label_zh": "Global Entry / TSA PreCheck", "benefit_family": "travel_id",
     "note_zh": "一次申请管 4–5 年，每人只需要一张卡报销"},   // note_zh 可选：显示在组标题下
    {"id": "other", "label_zh": "其他", "benefit_family": "other"}    // 必须有 other
  ],
  "cards": [
    {
      "id": "chase-sapphire-reserve",      // 稳定 id，永不复用
      "name": "Chase Sapphire Reserve",
      "issuer": "Chase",
      "network": "visa",                   // 可选（null = 未知）：visa | mastercard | amex | discover
      "annual_fee": 795,
      "currency": "chase_ur",              // currencies[].id
      "base_rate": {"rate": 1, "unit": "x"},   // unit: "x" = 每美元点数，"%" = 现金返现百分比
      "earn": [
        {
          "key": "dining",                 // 卡内唯一且跨版本稳定；本地状态（上限、反馈）按它挂
          "category": "dining",            // categories[].id 或 brands[].id
          "label": "Dining",
          "rate": 3, "unit": "x",
          "channel": "any",                // any | direct | issuer_portal
          "cap": null,                     // 或 {"amount": 1500, "period": "month|quarter|year|cardmember_year|promo_period", "after_cap_rate": 1,
                                           //     "group": "gas_ev"}
                                           // group 可选：同一张卡里 group 相同的行共用这一个上限（amount、period 必须相同，
                                           // after_cap_rate 可以不同）；任一行标了用满，整组都按用满算
          "valid_from": null, "valid_to": null,   // "YYYY-MM-DD"，含端点
          "notes": "",
          "confidence": "issuer_verified", // issuer_verified | guide_verified | third_party_only | user_reported
          "via": "Chase Travel portal"     // 可选：调研原文条件；channel 非 any 时 App 用它当条件说明
        }
      ],
      "rotating": [
        {
          "key": "2026Q4",
          "quarter": "2026Q4",             // 本地日期所在自然季度
          "categories": ["dining", "entertainment", "utilities"],
          "labels": ["Restaurants", "..."],
          "rate": 5, "unit": "%",
          "cap": {"amount": 1500, "period": "quarter", "after_cap_rate": 1},
          "activation_required": true,
          "confidence": "guide_verified"
        }
      ],
      "credits": [
        {
          "key": "uber-cash",
          "name": "Uber Cash",
          "benefit_type": "uber_cash",     // benefit_types[].id，由 conversion_hints.json 指定；桌面校验缺了就失败，
                                           // App 遇到缺失或未知的类型按 other 处理
          "stack": "per_card",             // per_card：一个人能用每张卡的这项福利（几张卡的 Uber Cash 都进同一余额）
                                           // once：一个人只能用一张卡的（一个账户只有一个 CLEAR / Uber One / Walmart+ 会员）
                                           // unknown：没查实。由 conversion_hints.json 的 credits.<key>.stack 指定，
                                           // why_stack 写理由和查过的来源；桌面校验缺了就失败，同一类型的福利必须一致。
                                           // App 遇到缺失或其他值按 unknown 处理
          "amount": 15,
          "period": "month",               // month | quarter | half_year | year | four_years | per_stay
          "resets": "calendar",            // calendar | cardmember_year | null（four_years、per_stay 为 null）
          "enrollment_required": true,     // true | false | null（来源未说明）
          "valid_from": null,              // 可选，"YYYY-MM-DD"：之前不显示；resets 为 null 或 valid_to 也有（限时活动）时，
                                           // 周期从 valid_from 起算，不按日历（AA Turo：2025-10-19 至 2026-10-18 是一期）
          "valid_to": null,
          "windows": [                     // 可选：分段不等长的限时福利，每段各 amount；有它就不用 period 推算周期，
            {"from": "2026-06-04", "to": "2026-12-31"}   // 不在任何一段里的日子不显示。须按时间排序、不重叠、
          ],                               // 落在 valid_from..valid_to 内
          "month_amounts": {"12": 35},     // 可选：只用于 period "month"、没有 windows 的福利。键是月份 "1".."12"
                                           // （不带前导零），值是那个月这一期的金额；没列的月份用 amount。
                                           // 本期金额、剩余金额、「用完」都按它算（Uber Cash：每月 $15，12 月 $35）
          "notes": "December gets an extra $20",
          "category_hint": "rideshare",    // 可为 null
          "issuer_tracker_match": [        // 可选：发卡行导出文件（data/issuer-export.schema.json）里哪些 tracker
            {"issuer": "amex", "pattern": "\\buber\\s*cash\\b"}   // 名对应这条福利。pattern 是不区分大小写的正则，
          ],                               // 在 tracker 名里搜（不要求整串匹配）；同卡同一 tracker 名只许命中一条福利
          "confidence": "user_reported"    // 可选：只在这条福利比整卡更不确定时写（档位同 earn 行），由
                                           // conversion_hints.json 的 credits.<key>.confidence 指定。user_reported 的
                                           // 福利不单独决定导入时推出的周年月（见下文「发卡行数据」的周年月一条）
        }
      ],
      "certificates": [ {"key": "...", "name": "...", "trigger": "...", "notes": ""} ],
      "other_benefits": ["..."],
      "confidence": "issuer_verified",     // 整卡最低档
      "sources": [ {"url": "...", "kind": "issuer|issuer_pdf|guide|third_party|user_report", "fetched": "YYYY-MM-DD"} ],
      "open_questions": ["..."],
      "watch": {"url": "...", "phrase": "...", "kind": "issuer|guide", "renders_without_js": true},
      "watch_guide": {"url": "https://www.uscreditcardguide.com/...", "phrase": "..."}   // 可选：该卡的美卡指南页，tools/watch.py 用
    }
  ]
}
```

## 校验规则（normalize 失败即非零退出）

- `id`、`earn[].key`、`rotating[].key`、`credits[].key` 在各自范围内唯一。
- `currency` 在 `currencies` 里；`earn[].category` / `rotating[].categories[]` 在 `categories` 或 `brands` 里；每个 brand 的 `parent` 在 `categories` 里；`categories` 的 parent 链无环。
- `unit` 只能是 `x` 或 `%`；`channel` 只能是三值之一。
- 同一张卡里品牌行回报低于其父类行时打 warning（不失败）。
- 每张卡有 `base_rate`。
- `network` 有值时只能是 `visa`、`mastercard`、`amex`、`discover`；没有时打 warning。`accepted_networks` 是这四个值组成的非空列表；`no_base_fallback` 只能写在 categories 上，取 true / false。
- 同一张卡里 `cap.group` 相同的行，`cap.amount` 和 `cap.period` 必须相同；组只有一行时打 warning。
- `credits[].windows` 每段有 `from`、`to`，按时间排序、不重叠，落在 `valid_from..valid_to` 内。
- `credits[].month_amounts` 只能写在 `period` 为 `month`、没有 `windows` 的福利上；键是 `"1"`..`"12"`，值是正数。
- `aliases`（及旧名 `search_terms`）有值时是非空字符串组成的列表。
- `benefit_types` id 唯一、有 `label_zh`、包含 `other`；每条 `credits[].benefit_type` 都必须是其中之一。
- `benefit_families` id 唯一、有 `label_zh`、包含 `other`；每个 `benefit_types[].benefit_family` 都必须是其中之一。
- 每条 `credits[].stack` 是 `per_card`、`once`、`unknown` 之一；同一 `benefit_type` 的福利 `stack` 必须相同（「我能用几份」按类型设）。
- `credits[].issuer_tracker_match` 是非空列表，每项恰好有 `issuer`（`[a-z][a-z0-9_]*`，同导出文件的 `issuer`）和 `pattern` 两个键；`pattern` 须能编译（Python `re.IGNORECASE`），且不能用 Python 和 Java 写法不同的记号（`(?P`、`\A`、`\Z`、`\z`、`\p{`），因为 App 用 Java 正则跑同一串；同一福利里不重复。App 侧校验 `issuer` 格式和能否编译，不过就整份目录不收。不互相串的保证靠 `tools/issuer_tracker_samples.json`：每个样例 tracker 名在该卡上恰好命中预期那一条（或一条都不中），`tools/test_tools.py`（Python）和 `IssuerTrackerSamplesTest`（JVM）各跑一遍。

以上新增字段都是可选的：旧目录没有它们时 App 照旧工作（卡组织未知的卡不会被商家过滤掉，没有组的上限各算各的，
没有分区的类型归「其他」，没有 `stack` 的福利按 `unknown` 算）。

## 可选信息字段（桌面工具写入，App 忽略即可）

`base_rate.notes`、`earn[].rate_incl_loyalty_earn`、`earn[].covers`、`rotating[].notes`、`watch.notes`、`sources[].read_ok` / `note`、顶层 `watch_guide`。（`earn[].via` 和 `credits[].valid_from` 原先在这里，现在 App 会用，见上。）

## 目录怎么生成

`data/catalog.json` 不手改。调研草稿（schema 1，卡的 JSON 数组）放在 `data/drafts/`，按卡给的提示（行可信度、卡组织、上限组、福利分段、按月金额、福利类型和 `stack`）在 `tools/conversion_hints.json`，类别 / 品牌 / 渠道规则在 `data/taxonomy.json`。`python tools/normalize_catalog.py --rebuild` 用这三样重新生成整份目录；当前目录里的任何卡 id 或 earn / rotating / credit / certificate key 在重建结果里消失都会失败、不写文件。`--add` 只追加新卡，从不改已有的卡。

## 持有卡导入文件 my-cards.json（个人数据，不进仓库）

`tools/make_holdings.py` 生成，App「导入」读取，「导出」写出同一格式（导出时附带本地福利使用、上限状态、反馈，导入时有就恢复）。

```jsonc
{
  "holdings_schema": 1,
  "exported": "2026-10-11T18:00:00-07:00",
  "instances": [
    {"id": "inst-1", "catalog_id": "amex-platinum", "nickname": "Platinum #1",
     "relationship_bonus_pct": 0, "anniversary_month": null, "notes": "",
     "bound_issuer": "amex", "issuer_suffix": "12345",          // 可选：发卡行账户绑定，两个要么都有要么都没有；
     "issuer_synced_at": "2026-10-11T12:00:00-07:00"}           // 同一 (issuer, suffix) 只能绑一张；最后一次应用的导出时间
    // relationship_bonus_pct 是百分数：25 表示 +25%（回报 ×1.25），不是 0.25
  ],
  "valuations": {"chase_ur": 1.5},          // 可选，覆盖 default_cpp
  "credit_usage": [], "cap_state": [], "rotating_activation": [],
  "custom_earn": [], "data_report": [],     // 可选，App 导出时填
  "credit_exclusions": [                    // 可选：「这张卡不追踪此项」
    {"instance_id": "inst-3", "credit_key": "uber-cash"}
  ],                                        // 列出的实例不计入福利组的合计、张数和提醒
  "credit_shares": [                        // 可选：「我能用几份」，每个 benefit_type 至多一条，shares ≥ 1
    {"benefit_type": "clear", "shares": 2}  // 例：自己和家人各一个 CLEAR 会员
  ]                                         // 没列的类型用默认值：per_card、unknown 为张数，once 为 1
}
```

福利页按 `benefit_family` 分区（区标题只有名称和最近到期日，不合计金额），区内按 `benefit_type` 分组，组内同一种卡的多张实例同一项福利叠成一个池（「Platinum ×5」）。

- 「我能用几份」：组展开后的步进器，1 到该类型追踪中的张数。默认 `per_card` 为张数、`once` 为 1、`unknown` 为张数并显示「能否叠加未确认」（用户改过就不再显示）。存在 App 的 Room 表 `credit_share`（数据库 v3，从 v2 迁移时只新建这张表，原有数据不动），导出导入随 `credit_shares`。
- 份数少于张数时只有拿到份的卡计入：本期已记录使用的卡优先，其次以前用过的，其次本期剩余多的，其次组内顺序（池按到期日、卡按持有顺序）；同一张卡在该类型的几项福利算一份。组的合计、剩余、张数后的「能用 N 份」、提醒都只算这些卡。
- 合计金额按各自本期金额（含 `month_amounts`）相加；组标题只写最近一个到期日前还剩多少（「10-31 前还剩 $75」），其他到期日的钱另起一行（「另有 $120 在 12-31 到期」）。排序按还有钱的最近到期日。
- 组级「记录使用」只在拿到份的卡里按实例顺序先填满第一张的剩余再填下一张，超出的记在最后一张拿到份的卡上；在某张没拿到份的卡上单独记录，份就挪到那张卡。

## 发卡行数据（导入文件格式见 `data/issuer-export.schema.json`）

设置 →「导入发卡行数据」（系统文件选择器，可多选）和系统分享 / 打开（`application/json`、`text/plain`、`*.json`）走同一个导入函数。文件先按 schema 校验，不合格整份不收，错误写明 JSON 路径。

- **绑定**：每个 `(issuer, suffix)` 绑到一张持有卡，存在该实例上（`bound_issuer`、`issuer_suffix`）。没绑的账户先过绑定步骤：按 `product` 和目录卡名给建议，默认「跳过」，从不自动绑。删卡或「解除绑定」后该尾号回到未绑定。
- **福利用量**：`kind: credit` 的 tracker 按目录 `credits[].issuer_tracker_match` 找福利，恰好命中一条才写。写进 `credit_usage`，`source` 为 `issuer:<issuer>`，`synced_at` 为文件的 `exported_at`，`period_start` / `period_end` / `period_amount` 是 tracker 自己的周期和金额。发卡行的数优先：同期的手动记录保留，福利页显示「手动记录 $X（已被同步覆盖）」。tracker 周期和目录推算的不一致时信 tracker（覆盖今天的那一期按 tracker 的起止和金额算）并在导入结果里列出。没命中、命中多条、不是美元、没有 `spent` 的 tracker 都列在结果里，不静默丢。
  - **按卡周年重置的福利**（`resets` 或 `period` 为 `cardmember_year`）：目录按月算周期，发卡行按开卡日算。tracker 周期长度等于目录周期长度（允许差一天：2 月 29 日的周年，Chase 脚本在平年改成 2 月 28 日，得到 2027-02-28 至 2028-02-28；以及结束在周年日当天的 2026-03-17 至 2027-03-17）、且开始月份等于目录那一期的开始月份（周年月未知时任何月份都算）时视为同一期，不列为不一致。
  - **只给一端日期**：按目录周期长度补另一端，补出来的周期照 tracker 自己的周期对待，结果里写「Chase 没给结束日期，按目录周期补成 X 至 Y」；按卡周年重置的年度福利补出来的开始月份和已设的周年月不同时也列出（周年月不改）。两端都没给（或福利有 `windows`、没有固定长度）时按目录在 `exported_at` 那天的那一期记，同样列出。
  - **周年月**：实例 `anniversary_month` 为空时，看这个账户里每一条按卡周年重置、周期为 12 个月（同上允许差一天）且日期来自 tracker 自己（两端都给，或给一端补全；按目录那一期记的不算）的 tracker：开始月份全部相同才用它填上，来源取第一条目录没标 `confidence: user_reported` 的福利（`setAnniversaryIfUnset`，只在为空时写，用户设过的不覆盖），导入结果「周年月」一节写明来源（「已按 Chase「Annual travel credit」的周期（2026-03-17 起）把周年月设为 3 月，可在卡详情改」）。月份不一致时不写，同一节列出每条 tracker 名和开始月份；只有 `user_reported` 的福利给了周期时也不写，同样说明。
  - 导入结果「周期或金额：和目录不一致或按目录补全」一节列出以上各种情况。
- **消费进度**（`kind: spend_progress`）存 Room 表 `spend_progress`，卡详情页显示进度条。**Offers** 存 `issuer_offer`，每个 `(issuer, suffix)` 整份替换，过期的不显示；查卡搜索词是某个商家名的子串（不分大小写）时排名上方显示「Offers」块，搜索词就是当前排序类别（或不匹配任何类别、按「其他消费」或之后点的类别排）时计入排序（品牌排序只计该品牌商家的 offer，其他排序只计搜索词在商家名词首出现的 offer）；「Offer」页按商家跨卡列出全部（`DESIGN.md`「Offers」）。
- **发卡行抬头**：每个发卡行最新一份已应用文件（至少应用到一个账户）的 `exported_at`、`source`、`warnings`（最多 50 条）存在 DataStore 的 `issuer_snapshots_json`（`{"amex": {...}, "chase": {...}}`），只保留 `exported_at` 最新的那份；不进持有卡导出文件，下次导入发卡行文件就回来。
- 同一文件再导一次不改任何东西；比该卡 `issuer_synced_at` 旧的文件对那张卡跳过。导入结果页顺序：脚本自己的 `warnings` 在最上，然后计数、未绑定账户、没对上的 tracker、周期或金额不一致。
- **导出导入持有卡文件**：带绑定（实例上的三个字段）和发卡行写的 `credit_usage` 行（带 `source` 等四个字段）；**不带** offers 和消费进度——这两样是快照，下次导入发卡行文件就回来。
- 数据库 v4：从 v3 迁移时给 `card_instance` 加 `bound_issuer`、`issuer_suffix`、`issuer_synced_at`，给 `credit_usage` 加 `source`、`synced_at`、`period_end`、`period_amount`（都可为空，旧行即未绑定、手动记录），新建 `issuer_offer`、`spend_progress` 两张表；原有数据不动。

## key 生成规则（保证跨版本稳定）

- `earn`：`<category>` ，`channel` 非 `any` 时加 `@<channel>`；同卡仍重名时加 `#2`、`#3`。
- `credits`：`name` 转小写 kebab-case。
- 已发布的 key 不改名；字段改值不影响 key。
