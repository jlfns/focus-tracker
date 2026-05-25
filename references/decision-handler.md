# C: 行动决策 Handler (decision-handler)

策略：跨平台对比 → 历史参照 → 带验证的推荐。

## Prompt 模板

执行以下 Prompt。替换 `{keyword}`、`{sub_category}`、`{current_date}`（当日真实日期，YYYY-MM-DD 格式）。

**执行前必须先联网搜索**：搜索「{keyword} 价格 当前年份」，搜多个平台（京东/淘宝/拼多多 等）做跨平台比价。

```
你是一个专业的行动决策助手。请根据用户关注的内容，结合联网搜索结果，提供可操作的建议和对比信息。

## 当前日期
{current_date}

## 关注项信息
- 关键词：{keyword}
- 子类别：{sub_category}

## 输出要求
请以 JSON 格式输出，严格遵循以下结构：
{
  "recommendation": "核心建议（一句话）",
  "action": "买/等/比/投/观望",
  "options": [
    {
      "name": "选项名称（商品/职位/房源/课程等）",
      "price": "价格（含货币单位）",
      "platform": "来源平台",
      "url": "链接（如有）",
      "availability": "有货/缺货/已下架/可报名",
      "updated_at": "价格更新时间（ISO 8601）"
    }
  ],
  "best_option": "最优选项名称",
  "reason": "推荐理由",
  "extra": {
    "price_trend": "涨/跌/稳",
    "urgency": "高/中/低",
    "freshness": "fresh/stale/unknown",
    "note": "备注说明"
  }
}

## 重要约束
1. 必须对比每个选项的 updated_at 与当前日期 {current_date}。若大部分价格早于 {current_date}，freshness 标为 "stale" 并在 note 中说明
2. 无法提供接近 {current_date} 的最新价格时，在 note 中注明"知识截止至 YYYY-MM，价格可能已变动"
3. 不得编造当前日期之后的数据
4. 至少提供 2 个可比较选项
5. 每个选项必须包含价格、平台、可用性和更新时间
6. 仅输出 JSON
```

## 纠错验证：价格交叉验证 + 时效性检查

解析 LLM 返回的 JSON，执行以下验证：

1. **跨平台价格检查**：取所有选项中的最低价，计算其他选项偏差 = `|选项价 - 最低价| / 最低价`
2. **时效性检查**：检查 updated_at，超过 24 小时 → 标记可能过期
3. **可用性检查**：任何选项"缺货"或"已下架" → 标记
4. **判定**：
   - 全部通过 → ✅ 理由："多平台验证通过，信息新鲜"
   - 价格偏差 3%-10% → ⚠️ 理由："平台间价格存在差异，建议多方比价"
   - 数据过期（>24h）→ ⚠️ 理由："部分信息已过 {n} 小时，可能已变动"
   - 选项不可用 → ❌ 理由："部分选项已缺货/下架"
   - 仅单一选项 → ⚠️ 理由："仅单一选项，无法交叉对比"

## 输出格式

```
🛒 {keyword} | {sub_category}
━━━━━━━━━━━━━━━━━━━━━━
建议：{recommendation}
行动：{action}
最优选择：{best_option}
理由：{reason}
━━━━━━━━━━━━━━━━━━━━━━
选项对比：
  🥇 {option1}: {price} | {platform} | {availability}
  🥈 {option2}: {price} | {platform} | {availability}
━━━━━━━━━━━━━━━━━━━━━━
价格趋势：{price_trend} | 紧迫度：{urgency}
验证：{✅/⚠️/❌} {reason}
```

⚠️/❌ 时追加相关警告（过期数据、缺货等）。
