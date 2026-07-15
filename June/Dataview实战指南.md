---
date: 2026-07-03
tags: [Obsidian, Dataview, 教程]
type: guide
status: complete
---

# Dataview 实战指南：用查询盘活你的知识库

> 本文承接 [[June/markdown基础语法#16. 笔记属性（Properties / YAML Frontmatter）]]，深入讲解如何用 Dataview 插件将笔记属性转化为**动态查询**，打造可检索、可汇总、可分析的知识库。

---

## 一、Dataview 是什么？

**Dataview** 是 Obsidian 最强大的插件之一。它让你像操作**数据库**一样管理笔记：

- 把每篇笔记看作数据库中的**一条记录**
- 把笔记的 **属性（Properties）** 看作数据库的**字段**
- 用 `dataview` 代码块编写 **SQL 风格的查询语句**
- 查询结果**自动更新**——新增或修改笔记后，查询实时刷新

---

## 二、核心概念：属性 → 字段

### 映射规则

你在笔记顶部的 YAML 属性会自动成为 Dataview 可识别的字段：

```yaml
---
title: 基金投资入门        # → 字段 title = "基金投资入门"
date: 2025-07-01          # → 字段 date = 2025-07-01 (日期类型)
rating: 8                 # → 字段 rating = 8 (数字类型)
tags: [投资, 基金, 入门]  # → 字段 tags = ["投资", "基金", "入门"] (列表)
status: 已完成             # → 字段 status = "已完成"
source: 某课程             # → 字段 source = "某课程"
---
```

### 特殊的内置字段

Dataview 自动为每篇笔记生成以下字段，不需要自己声明：

| 内置字段 | 含义 | 类型 |
|---------|------|------|
| `file.name` | 笔记文件名（不含扩展名） | 文本 |
| `file.path` | 笔记完整路径 | 文本 |
| `file.size` | 笔记大小（字节） | 数字 |
| `file.ctime` | 笔记创建时间 | 日期 |
| `file.mtime` | 笔记最后修改时间 | 日期 |
| `file.tags` | 笔记中所有标签 | 列表 |
| `file.etags` | 显式标签（`#tag` 语法） | 列表 |
| `file.aliases` | 笔记别名 | 列表 |
| `file.links` | 笔记中的出链 | 列表 |
| `file.outlinks` | 笔记中的外部链接 | 列表 |
| `file.folder` | 笔记所在文件夹 | 文本 |
| `file.day` | 笔记名中的日期（如 `2025-07-01` 格式） | 日期 |

> 💡 `file.day` 很实用：如果你的笔记命名是 `2025-07-01_周三.md`，Dataview 会自动把 `2025-07-01` 部分提取为日期。

---

## 三、四大查询类型

### 1. TABLE — 表格视图（最常用）

以表格形式展示数据，可自定义列：

```dataview
TABLE title, date, rating, status
FROM "读书笔记"
SORT date DESC
```

```markdown
1. `TABLE` (表格)：
    
    - 这是你要**展现的形式**。意思是，把查出来的结果以“表格”的形式呈现出来。如果写 `LIST`，就会变成简单的列表。
        
    - `title, date, rating, status`：这是你**要在表格里显示的列**。相当于告诉软件：“请把每篇笔记的标题、日期、评分、状态这 4 项数据提取出来，列成表格。”
        
2. `FROM "读书笔记"` (数据来源)：
    
    - 这是你要查询的位置。意思是，只去你笔记库里名为 **“读书笔记”** 的这个**文件夹**（或标签）里找笔记。
        
    - （注：`"读书笔记"` 加了双引号，通常代表这是一个**文件夹**名称。如果是 `#读书笔记` 则代表查找带有该标签的笔记）。
        
3. `SORT date DESC` (排序)：
    
    - 这是排序规则。
        
    - `SORT date`：按日期（date）字段排序。
        
    - `DESC`：**降序**（Descending）。也就是最新的笔记排在最前面。如果想要最老的排前面，则写 `ASC`
```

### 2. LIST — 列表视图

以列表形式展示，适合快速浏览：

```dataview
LIST FROM "投资笔记"
WHERE status = "进行中"
```

### 3. TASK — 任务视图

汇总所有笔记中的 `- [ ]` 任务（结合属性过滤）：

```dataview
TASK FROM "学习计划"
WHERE contains(tags, "基金")
```

### 4. CALENDAR — 日历视图

将笔记按日期投射到日历上：

```dataview
CALENDAR file.day
FROM "日报"
```

---

## 四、FROM：指定搜索范围

| 写法 | 含义 | 示例 |
|------|------|------|
| `FROM "文件夹"` | 该文件夹下的所有笔记 | `FROM "June"` |
| `FROM #tag` | 带有该标签的笔记 | `FROM #投资` |
| `FROM "文件夹" OR #tag` | 满足任一条件 | `FROM "读书笔记" OR #基金` |
| `FROM -"文件夹"` | 排除该文件夹 | `FROM -"attachment"` |
| `FROM ""` | 所有笔记 | `FROM ""` |

---

## 五、WHERE：条件过滤（核心）

WHERE 是最常用、最强大的部分。以下按条件类型分类讲解：

### 5.1 文本条件

```dataview
TABLE title, status
WHERE status = "进行中"
```

```dataview
TABLE title, source
WHERE source != "待定"
```

### 5.2 包含条件（列表字段）

`tags` 是列表类型，用 `contains` 检查是否包含某个值：

```dataview
TABLE title, tags
WHERE contains(tags, "基金")
```

检查多个标签（必须同时包含）：

```dataview
TABLE title
WHERE contains(tags, "基金") AND contains(tags, "入门")
```

### 5.3 数字条件

```dataview
TABLE title, rating
WHERE rating >= 8
SORT rating DESC
```

### 5.4 日期条件

```dataview
TABLE title, date
WHERE date >= date(2025-07-01)
SORT date ASC
```

最近 7 天：

```dataview
TABLE title, date
WHERE date >= date(today) - dur(7 days)
SORT date DESC
```

本月：

```dataview
TABLE title, date
WHERE date >= date(today).startofmonth
SORT date ASC
```

### 5.5 组合条件（AND / OR）

```dataview
TABLE title, date, rating, status
WHERE contains(tags, "基金")
  AND rating >= 7
  AND status != "已归档"
SORT rating DESC
```

### 5.6 文件路径条件

只看某个子文件夹：

```dataview
TABLE title
WHERE contains(file.path, "学习笔记/投资")
```

排除某个文件夹：

```dataview
TABLE title
WHERE file.folder != "attachment"
```

### 5.7 反向链接条件

找出"被某篇笔记引用"的笔记：

```dataview
TABLE title
WHERE contains(file.outlinks, [[基金投资入门]])
```

---

## 六、SORT：排序

```dataview
TABLE title, date, rating
WHERE status = "已完成"
SORT rating DESC           # 评分从高到低
SORT date ASC              # 日期从旧到新（当评分相同时）
```

> 可以指定多个排序字段，从左到右依次生效。

---

## 七、GROUP BY：分组聚合

### 按状态分组

```dataview
TABLE rows.file.link AS "笔记", rows.rating AS "评分"
WHERE contains(tags, "投资")
GROUP BY status
SORT rows.file.name ASC
```

### 按标签分组

```dataview
TABLE rows.file.link AS "相关笔记"
FLATTEN tags
GROUP BY tags
```

### 按月份分组

```dataview
TABLE rows.file.link AS "笔记"
WHERE date
GROUP BY date.year + "-" + date.month AS "月份"
SORT "月份" DESC
```

---

## 八、FLATTEN：展开列表

当你有一个列表字段（如 `tags`），需要"拆开"处理时用 FLATTEN：

```dataview
TABLE file.link, rating
FLATTEN tags
WHERE contains(tags, "基金")
```

---

## 九、实战场景（投资学习知识库）

以下场景全部围绕你的保险库设计，可直接复制使用。

### 场景 1：读书笔记评分排行榜

假设你的读书笔记都有统一的属性：

```yaml
---
title: 纳瓦尔宝典
author: Naval Ravikant
category: 理财观念
rating: 9
date_finished: 2025-06-15
tags: [读书笔记, 理财观念, 财富自由]
status: 已完成
---
```

查询语句：

```dataview
TABLE author AS "作者", rating AS "评分", date_finished AS "读完日期"
FROM #读书笔记
WHERE status = "已完成"
SORT rating DESC
```

### 场景 2：进行中的学习任务看板

```dataview
TABLE date AS "创建日期", tags AS "标签"
FROM ""
WHERE status = "进行中"
SORT date ASC
```

### 场景 3：按分类汇总所有笔记

```dataview
TABLE rows.file.link AS "笔记列表"
FROM ""
WHERE category
GROUP BY category AS "分类"
SORT rows.file.name ASC
```

### 场景 4：某只基金的学习历史

给每篇关于特定基金的笔记加属性：

```yaml
---
fund: 沪深300
type: 指数基金
tag: 定投
---
```

查询：

```dataview
TABLE date, type, tag
FROM ""
WHERE fund = "沪深300"
SORT date ASC
```

### 场景 5：近期新增/修改的笔记

```dataview
TABLE file.mtime AS "最后修改"
FROM ""
WHERE file.mtime >= date(today) - dur(7 days)
SORT file.mtime DESC
```

### 场景 6：自定义每日复盘模板

```yaml
---
date: 2025-07-01
mood: 3
learned: 理解了夏普比率的意义
todo_status: 完成80%
tags: [日报, 基金, 学习]
---
```

查询：

```dataview
TABLE mood AS "心情(1-5)", learned AS "今日收获", todo_status AS "进度"
FROM "日报"
WHERE date
SORT date DESC
LIMIT 10
```

---

## 十、DataviewJS：更灵活的查询

当基础 Dataview 无法满足需求时，可以用 **DataviewJS**（代码块用 `dataviewjs`）：

```dataviewjs
// 获取所有读书笔记
const pages = dv.pages('#读书笔记')
  .where(p => p.rating >= 7)
  .sort(p => p.rating);

// 遍历并自定义输出
for (let p of pages) {
  dv.paragraph(`**${p.file.link}** — 评分: ${p.rating}/10`);
}

// 统计
dv.span(`共 ${pages.length} 篇高质量读书笔记`);
```

### 常用 JS 方法

```dataviewjs
// 按状态计数
const counts = dv.pages('')
  .where(p => p.status)
  .groupBy(p => p.status);

dv.table(["状态", "数量"], counts.map(g => [g.key, g.rows.length]));

// 标签云
const tagCount = {};
dv.pages('').forEach(p => {
  (p.tags || []).forEach(t => {
    tagCount[t] = (tagCount[t] || 0) + 1;
  });
});
dv.paragraph(JSON.stringify(tagCount, null, 2));
```

---

## 十一、性能优化建议

| 建议 | 说明 |
|------|------|
| **限制 FROM 范围** | 尽量指定文件夹，不要写 `FROM ""`（查全部） |
| **加 LIMIT** | 测试时加 `LIMIT 20` 防止卡顿 |
| **不要过度使用 JS** | DataviewJS 比基础查询慢，够用就用基础语法 |
| **避免复杂 FLATTEN** | 展开大量嵌套数据会拖慢性能 |
| **升级到 Dataview 0.5+** | 新版本性能大幅优化，支持增量索引 |

---

## 十二、常见错误排查

| 错误现象 | 原因 | 解决 |
|---------|------|------|
| 查询无结果 | 文件夹路径写错 | 检查 FROM 路径大小写 |
| 字段显示为空 | 属性名拼写不一致 | 确保 `date:` 和 `WHERE date` 拼写完全一致 |
| 日期不显示 | 日期格式不对 | 用 `YYYY-MM-DD` 标准格式 |
| 排序不对 | 字段类型不匹配 | 确保数字字段没有加引号（`rating: 8` 而非 `rating: "8"`） |
| 代码块不渲染 | Dataview 插件未启用 | 设置 → 第三方插件 → 启用 Dataview |

---

## 总结速查表

```dataview
TABLE 用途, 示例
FROM ""
WHERE contains(tags, "速查")
```

| 你想要 | 用这个 |
|--------|--------|
| 列出某文件夹笔记 | `FROM "文件夹"` |
| 按字段过滤 | `WHERE rating >= 8` |
| 检查标签 | `WHERE contains(tags, "基金")` |
| 排序结果 | `SORT date DESC` |
| 限制数量 | `LIMIT 10` |
| 分组统计 | `GROUP BY status` |
| 展开列表 | `FLATTEN tags` |
| 日历视图 | `CALENDAR file.day` |
| 动态图表 | 用 DataviewJS + 图表库 |

---

> 📌 **下一步建议**：可以创建一份 `_模板/笔记属性模板.md`，里面预设好常用属性字段，每次新建笔记时直接复制，保证所有笔记属性统一，让 Dataview 查询更精确。
