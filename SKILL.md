---
name: xiaohui_wechat-ai-tools
description: |
  公众号「灰常有想法」AI 工具/应用赛道生产线。
  每日 5 篇，卡兹克风格，受众：创业者/小老板/AI实用主义者。
  全自动：热点采集→选题筛选→文案生成→AI配图→草稿箱发布→飞书通知。
  触发词：AI生产线、AI文章、AI工具文章、AI选题
version: 1.0.0
category: content
triggers:
  - AI生产线
  - AI文章
  - AI工具文章
  - AI选题
  - AI赛道
---

# 公众号 AI 工具/应用赛道生产线

> **风格锚定**：卡兹克（数字生命卡兹克）— "有见识的普通人在认真聊一件打动他的事"
> **受众画像**：创业者、小老板、AI实用主义者（25-45岁，要的是能用的东西，不是技术科普）
> **日产量**：5 篇/天（Cron 22:00 批量 → 草稿箱 → 人工审核）

---

## 📋 流程总览

```
Step 1: 热点采集（topic_collector.py + web_search）
  ↓
Step 2: 选题筛选（15 候选 → 5 篇）
  ↓
Step 3: 素材深挖（每篇提取 3-5 个关键事实）
  ↓
Step 4: 文案生成（2000-4000 字/篇 × 5 篇，卡兹克风格）
  ↓
Step 5: 风格自检（四层检查）
  ↓
Step 6: AI 配图（封面 21:9 + 正文信息图）
  ↓
Step 7: 发布到草稿箱
  ↓
Step 8: 飞书通知审核
```

---

## Step 1: 热点采集

### 数据源优先级

```bash
# 1. 6551 API — Twitter/X AI 圈热推
python3 ~/.hermes/skills/xiaohui-meta-image/scripts/topic_collector.py \
  --track xiaohongshu-ai-tools --limit 20 --format json \
  --output /tmp/ai-topics-twitter.json

# 2. 6551 API — 新闻聚合
python3 ~/.hermes/skills/xiaohui-meta-image/scripts/topic_collector.py \
  --keywords "AI工具,ChatGPT,Claude,DeepSeek,AI应用,AI创业,AI变现" \
  --source news --limit 20 --format json \
  --output /tmp/ai-topics-news.json

# 3. Web Search 补充（中文科技媒体）
web_search("AI工具 实测 2026")
web_search("AI创业 变现 案例 最新")
web_search("DeepSeek Claude GPT 最新动态")
```

### 采集关键词池

```
一线热点：DeepSeek, Claude, GPT, Gemini, AI Agent, Sora, AI编程
商业应用：AI变现, AI创业, AI工具实测, AI效率, AI自动化
行业动态：AI融资, AI政策, AI裁员, AI岗位, AI公司
垂直场景：AI写作, AI设计, AI视频, AI客服, AI营销, AI教育
```

---

## Step 2: 选题筛选

### 筛选矩阵

每个候选按 3 维度打分（1-5分），总分 > 10 才入选：

| 维度 | 权重 | 判断标准 |
|------|------|---------|
| 实战价值 | ×2 | 读完能立刻用的 > 只能看的 > 纯新闻 |
| 时效性 | ×1 | 24h 内 > 3 天内 > 一周内 > 过期 |
| 情绪共鸣 | ×1 | 能引发"卧槽"/"终于有人说了"/"我也是" |

### 5 篇选题配比

```
必须包含：
  ├ 1 篇 工具实测（亲自下场用，截图+步骤+结论）
  ├ 1 篇 行业洞察（数据+趋势+独立判断）
  ├ 1 篇 商业案例（真实公司/个人的 AI 变现故事）
  └ 2 篇 灵活搭配（热点评论/教程/观点文/深度分析）

禁止：
  ├ 同一天 5 篇全是工具测评
  ├ 同一天出现 2 篇以上纯新闻转述
  └ 跟前 3 天的选题重复率 > 30%
```

### 去重检查

```bash
cat ~/.hermes/shared/wechat-ai-tools/topic-history.json 2>/dev/null || echo "[]"
# 标题相似度 > 60% 的跳过
```

---

## Step 3: 素材深挖

**选题确定后，每篇都要深挖素材，不能只靠标题写。**

```
每篇必须准备：
  ├ 核心事实 × 3-5 个（来源可追溯）
  ├ 关键数据 × 2-3 个（融资额/用户数/增长率等）
  ├ 原始来源 URL × 2-3 个
  └ 竞品/对比对象（如有）

工具：
  web_extract(原始报道URL)  → 提取关键段落
  web_search(补充搜索)      → 交叉验证数据
```

---

## Step 4: 文案生成（卡兹克风格）

### 风格规则文件

完整规则见：`references/khazix-style-rules.md`

### 5 个核心风格锚点（简版）

```
1. 活人感优先：口语化、自嘲、情绪标点（。。。 ？？？），像跟朋友聊天
2. 亲自下场：所有工具必须具体名字，禁止"某AI工具"，案例必须真实
3. 波动式节奏：长短句交替，一句话独立成段，疑问句做刹车
4. 知识随手掏：引用是"聊着聊着想起来的"，不是"下面我来科普"
5. 文化升维：从具体事件连接到更大的文化/哲学参照物
```

### 文案结构模板

```
─── 开头（200-400字）─── 四选一：
  A. 叙事启动：「故事是这样的。{一句话交代起因}」
  B. 荒诞事实：「{直接抛出让人"？？？"的事实}」
  C. 热点破题：「{最近被xxx刷屏了}」
  D. 好奇心驱动：「{最近刷到了一个xxx，很有意思}」

─── 正文（1500-3000字）───
  [核心论述] 工具实测/数据分析/商业拆解
  [知识随手掏] 自然插入 1-2 个跨领域知识点
  [独立判断] 「我自己的感受是...」「坦率的讲...」

─── 结尾（200-400字）─── 五选一：
  A. 引用收尾
  B. 哲思余韵
  C. 行动呼吁（推荐最多用）
  D. 信念宣言
  E. 回环呼应

─── 固定尾部 ───
  以上，觉得有启发的话，点个「在看」分享给更多人。
  有问题或者想交流的，评论区见。
```

### 文案质量标准

- **字数**：2000-4000 字（不硬凑，内容够了就收）
- **具体名字**：工具名/公司名/人名，必须准确具体
- **数据引用**：每篇至少 2 个可验证的数据点
- **独立观点**：每篇至少 1 个明确的个人判断（"我觉得""我的看法是"）
- **口语化词组**：全文至少用 8 个不同的口语化转场词

### 禁忌清单（硬性规则）

```
禁用词：说白了、本质上、综上所述、值得注意的是、让我们来看看、不可否认
禁用标点：冒号「：」→用逗号、破折号「——」→用逗号句号
禁用结构：连续 bullet point > 3 个、大段加粗、宏大叙事开场
禁用开头：「在当今AI时代」「随着技术的发展」
禁用内容：假设性例子、空泛工具名、教科书式科普、知识性情绪描述
```

### 四层自检（每篇写完必跑）

```
L1 硬性规则：禁用词 + 禁用标点 + 套话扫描 + 工具名检查
L2 风格一致：开头模式 + 节奏波动 + 口语化密度（≥8个词组）
L3 内容质量：观点支撑 + 知识输出方式 + 数据准确性
L4 活人感终审：温度感 + 独特性 + 姿态（朋友 vs 导师）
```

---

## Step 5: AI 配图

### 封面图

```
尺寸：21:9（900×383）
风格：科技感 + 未来感，但不冷冰冰
色调：深蓝/渐变紫/赛博朋克，但保持温暖质感
元素：与文章主题相关的 AI/科技意象
工具：wechat-cover-master skill → pipeline_image_gen.py
右侧留白：必须，用于排版标题
```

### 正文配图

```
数量：每篇 1-3 张
类型优先级：
  1. 信息图/对比表（HTML+CSS 渲染，适合工具测评）
  2. 工具截图（如果是实测类文章）
  3. AI 生成配图（Nano Banana 2）
```

### 封面 Prompt 模板

```
"Modern tech illustration, {主题关键词}, clean design,
 gradient blue-purple tones, warm futuristic atmosphere,
 subtle glow effects, professional but approachable,
 right side empty for text overlay, 21:9 aspect ratio"
```

---

## Step 6: 发布到草稿箱

```bash
# 使用 baoyu-post-to-wechat 浏览器模式
bun ~/.hermes/skills/baoyu-post-to-wechat/scripts/wechat-article.ts \
  --markdown {article.md} \
  --theme grace \
  --cover {cover_image_path}
```

**重要**：5 篇全部进草稿箱，绝不自动发布。

---

## Step 7: 飞书通知

```
🤖 AI 赛道今日产出（{日期}）
━━━━━━━━━━━━━━━━━━━━━━
1️⃣ [工具实测] {标题} | {字数}字
2️⃣ [行业洞察] {标题} | {字数}字
3️⃣ [商业案例] {标题} | {字数}字
4️⃣ [{类型}] {标题} | {字数}字
5️⃣ [{类型}] {标题} | {字数}字
━━━━━━━━━━━━━━━━━━━━━━
📸 配图: 封面 ×5 + 正文 ×{N}
📮 状态: 已进草稿箱，待审核
🔥 今日最佳: #{最高分篇号}
```

---

## Step 8: 更新历史记录

```python
# 追加到选题历史
import json, os
path = os.path.expanduser("~/.hermes/shared/wechat-ai-tools/topic-history.json")
history = json.load(open(path)) if os.path.exists(path) else []
for article in today_articles:
    history.append({
        "date": "YYYY-MM-DD",
        "title": article["title"],
        "category": article["type"],  # 工具实测/行业洞察/商业案例/...
        "word_count": article["word_count"],
        "sources": article["source_urls"]
    })
json.dump(history, open(path, "w"), ensure_ascii=False, indent=2)
```

---

## 📎 References

| 文件 | 用途 |
|------|------|
| `references/khazix-style-rules.md` | 卡兹克完整写作风格规则手册 |

## ⚠️ Pitfalls

1. **5 篇/天是高强度**：质量不够宁可减量，绝不水文
2. **数据必须可验证**：引用的融资额/用户数必须有来源，编造数据是大忌
3. **工具名必须准确**：写错工具名/版本号会被读者喷
4. **不要全是正面评价**：工具有缺点就直说，真诚是卡兹克风格的核心
5. **热点有时效性**：超过 7 天的热点要换角度，不能原样追
6. **避免同质化**：5 篇之间要有区分度，标题不能看起来像同一篇
7. **配图不要用通用素材图**：每篇的封面必须跟内容相关，不能 5 篇用差不多的图
