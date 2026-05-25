# Classification Rules — 18 Sub-Categories

## How to Use This File

Match user input against keywords below. Longest match wins. Default: `news` (新闻热点).

## A: 数值查询 (numeric-handler)

| category | sub_category | keywords |
|----------|-------------|----------|
| finance | 金融行情 | 黄金, 金价, 股票, 股价, 行情, 指数, A股, 美股, 港股, 基金净值, 期货, 原油, 白银, 道琼斯, 纳斯达克, 上证, 深证 |
| crypto | 加密货币 | 加密货币, 比特币, 以太坊, BTC, ETH, SOL, 币价, 挖矿, DeFi, NFT价格, 狗狗币, USDT, 币安 |
| macro | 宏观经济 | GDP, CPI, 通胀, 利率, PMI, 失业率, 经济数据, 贸易顺差, 财新PMI, M2, 通货, 降息, 加息, LPR |
| weather | 天气环境 | 天气, 温度, 降雨, PM2.5, 空气质量, AQI, 台风, 紫外线, 湿度, 风力, 雾霾, 暴雨, 高温 |
| exchange_rate | 汇率换算 | 汇率, 换汇, 美元兑, 欧元兑, 人民币汇率, 日元汇率, 英镑汇率, USD, EUR, JPY, CNY, 港币, 中间价 |
| logistics | 物流追踪 | 快递, 物流, 运单, 包裹, EMS, 顺丰, 中通, 圆通, 韵达, 配送, 快递单号, 到哪了 |

## B: 信息聚合 (aggregate-handler)

| category | sub_category | keywords |
|----------|-------------|----------|
| news | 新闻热点 | 新闻, 热点, 事件, 突发, 头条, 热搜, 最新消息, 动态, 社会, 时事 |
| tech | 科技动态 | 科技, AI, 人工智能, 芯片, 苹果, 谷歌, 微软, 发布会, WWDC, GPT, Claude, LLM, 大模型, 手机, 系统更新, 特斯拉, OpenAI, 英伟达, NVIDIA, 显卡, GPU, RTX, CPU, 处理器 |
| policy | 政策法规 | 政策, 法规, 法律, 规定, 条例, 监管, 合规, 新规, 出台, 文件, 通知, 禁令, 整改 |
| opensource | 开源项目 | 开源, GitHub, 仓库, star, fork, release, 框架, 库, 工具链, npm, pypi, 版本更新, 许可证 |
| competitive | 竞品行业 | 竞品, 行业, 市场, 份额, 对手, 赛道, 趋势, 报告, 分析, 研报, 财报 |
| academic | 学术论文 | 论文, 学术, 研究, 期刊, arXiv, 引用, 学者, 实验, NeurIPS, CVPR, ICML, ICLR, paper |

## C: 行动决策 (decision-handler)

| category | sub_category | keywords |
|----------|-------------|----------|
| ecommerce | 电商比价 | 比价, 价格对比, 最低价, 京东, 淘宝, 拼多多, 购物, 商品价格, 买, 便宜, 优惠, 折扣, 好价, 划算, 价格, 多少钱 |
| travel | 票务出行 | 机票, 火车票, 酒店, 出行, 旅游, 航班, 高铁, 签证, 民宿, 自由行, 跟团, 攻略, 去玩 |
| job | 招聘求职 | 招聘, 求职, 岗位, 面试, 薪资, Offer, 跳槽, 内推, 简历, BOSS直聘, 猎头 |
| rental | 房产租金 | 租房, 房价, 租金, 二手房, 新房, 房贷, 楼盘, 中介, 链家, 安居客, 均价 |
| discount | 优惠补贴 | 补贴, 满减, 优惠券, 红包, 返利, 薅羊毛, 信用卡, 免息, 福利, 政府补贴, 权益 |
| course | 课程资源 | 课程, 教程, 培训, 学习, 网课, 视频课, 考证, MOOC, Udemy, Coursera, 限免, 资源 |

## Mixed Intent Split Patterns

Split user input on these separators to detect multiple focus items:
- `和`, `与`, `及`, `以及`, `还有`
- `、`, `,`, `;`, `；`

Each segment is classified independently.
