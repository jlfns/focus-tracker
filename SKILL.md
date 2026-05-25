---
name: focus-tracker
description: >
  持久化关注列表管理器。将关心的事项存入本地 JSON 文件，随时一键批量查询。
  触发词："关注XXX"（添加）、"取消关注XXX"（删除）、"我关注了哪些"（查看列表）、"看看我关注的"/"帮我查一下关注的"/"查关注的"（批量联网执行）。
  自动分类为 18 个子类 × 3 策略组（数值查询/信息聚合/行动决策）。
  存储路径：{用户主目录}/.workbuddy/focus-tracker/focus-list.json（跨平台自适应）
---

# Focus Tracker · 关注追踪器

持久化关注列表管理器。维护一个本地 JSON 文件，逐项添加，一键批量查询。

## 存储

- **数据目录**：`{用户主目录}/.workbuddy/focus-tracker/`
- **数据文件**：`{用户主目录}/.workbuddy/focus-tracker/focus-list.json`
- **路径解析**：
  - Windows：`%USERPROFILE%\.workbuddy\focus-tracker\focus-list.json`（例如 `C:\Users\gq\.workbuddy\focus-tracker\`）
  - macOS / Linux：`~/.workbuddy/focus-tracker/focus-list.json`（例如 `/home/gq/.workbuddy/focus-tracker/`）
  - 执行时先通过系统环境变量（`USERPROFILE` 或 `HOME`）获取主目录，拼接相对路径
- **首次使用**：目录不存在则递归创建，文件不存在则写入初始值 `[]`
- **格式**：JSON 数组，每项结构如下：

```json
[
  {
    "id": 1,
    "keyword": "黄金价格",
    "category": "finance",
    "sub_category": "金融行情",
    "handler": "numeric-handler",
    "added_at": "2026-05-25",
    "last_run": null
  }
]
```

## 五大命令

### 1. 添加关注

触发：用户说"关注XXX" / "加上XXX" / "帮我关注XXX"

流程：
1. 提取关键词（去掉"关注"/"加上"前缀）
2. 用 `references/rules.md` 做关键词分类（纯规则，不调模型）
3. 读取当前 focus-list.json
4. 已存在同关键词项 → 提示用户，跳过
5. 追加新项，ID 自增
6. 写回 focus-list.json
7. 回复确认："✅ 已添加 [分类标签] {keyword} → {sub_category} | 当前关注 N 项"

示例："关注黄金价格" → "✅ 已添加 📊 黄金价格 → 金融行情 | 当前关注 3 项"

### 2. 取消关注

触发：用户说"取消关注XXX" / "删除XXX关注"

流程：
1. 提取关键词，读取 focus-list.json
2. **模糊匹配**：在列表中查找 keyword 包含该词的项（如"取消关注黄金" → 匹配"黄金价格"）
3. 命中多项目 → 列出所有匹配项，让用户确认删哪个
4. 命中单项目 → 直接确认后删除
5. 无匹配 → 提示未找到
6. 写回 focus-list.json

### 3. 更新关注

触发：用户说"更新关注XXX" / "修改XXX关注为YYY"

流程：
1. 提取旧关键词和新关键词
2. 按旧关键词模糊匹配找到对应项
3. 如果命中多项目 → 列出让用户选
4. 对新关键词重新分类
5. 更新 keyword、category、sub_category、handler 字段
6. 写回并确认

### 4. 查看列表

触发：用户说"我关注了哪些" / "关注列表" / "看看我关注了什么"

流程：
1. 读取 focus-list.json
2. 逐项展示 ID、关键词、分类、添加日期、上次执行
3. 空列表 → 提示"关注列表为空，用「关注XXX」添加"

### 5. 批量执行

触发：用户说"看看我关注的" / "帮我查一下关注的" / "查关注的" / "跑一下关注的"

流程：
1. 读取 focus-list.json
2. 空列表 → 提示为空
3. **先列出全部关注项，询问用户："确认执行以上 N 项吗？"**
4. 等待确认后再执行
5. 逐项：联网搜索 → 加载对应 Handler → 执行 Prompt → 纠错验证 → 输出结果
6. 执行完成后，将每项的 `last_run` 更新为当日日期，写回 focus-list.json
7. 各项独立执行，结果顺序呈现

**关键规则：执行前必须先展示列表并请求确认，绝不能跳过。**

### 6. 清空关注

触发：用户说"清空关注"

流程：
1. 请求确认
2. 确认后将 focus-list.json 写为 `[]`

## 分类逻辑

使用 `references/rules.md` 中的 18 个子类关键词规则。

分类流程（直接执行，不调模型）：
1. 预处理：去空格、转小写
2. 遍历所有规则，若关键词出现在输入中 → 命中
3. 多个关键词命中时，取**最长**的那个
4. **价格意图优先**：当 tech 类关键词（显卡/GPU/CPU/处理器/RTX）与 ecommerce 类关键词（价格/多少钱/比价/买/便宜/折扣）同时命中 → 优先取 ecommerce，因为用户想比价而非看新闻
5. 无命中 → 默认 `news`（新闻热点, aggregate-handler）

## Handler 分发（批量执行时）

按每项的 handler 字段加载对应参考文件：

| Handler | 参考文件 | 策略组 |
|---------|---------|--------|
| numeric-handler | references/numeric-handler.md | A: 数值查询 |
| aggregate-handler | references/aggregate-handler.md | B: 信息聚合 |
| decision-handler | references/decision-handler.md | C: 行动决策 |

按参考文件中的 Prompt 模板执行，注入 `{keyword}`、`{sub_category}`、`{current_date}`（当日日期），联网搜索获取实时数据，应用纠错验证，输出结构化结果。

## 并发执行

关注项之间无依赖，独立运行。联网搜索尽量并行发起，结果顺序呈现。
