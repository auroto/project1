# 电商用户转化漏斗诊断与 A/B 实验效果分析

基于 200 万条行为事件的电商数据集，使用 Python 完成数据清洗、漏斗诊断、A/B/C 实验分析与渠道分层，定位转化瓶颈并评估实验版本效果。

---

## 一、项目背景与目标

电商平台上线了两版结算页改动（Variant_A / Variant_B），需要通过数据回答三个问题：

1. 用户在哪个环节流失最严重？
2. 两个实验版本相对 Control 是否有提升？提升是否显著？
3. 不同获客渠道的转化效率是否有差异？

---

## 二、数据说明

数据来源：Kaggle - [Marketing and E-commerce Analytics Dataset](数据集链接)](https://www.kaggle.com/datasets/geethasagarbonthu/marketing-and-e-commerce-analytics-dataset)

包含 5 张业务表：

| 表名 | 行数 | 说明 |
| :--- | :--- | :--- |
| customers | 100,000 | 用户属性：国家、年龄、性别、忠诚度、获客渠道 |
| events | 2,000,000 | 行为事件：view / click / add_to_cart / purchase / bounce |
| transactions | 103,127 | 交易记录：数量、折扣率、收入、退款标记 |
| products | 2,000 | 商品属性：品类、品牌、价格、是否高端 |
| campaigns | 50 | 营销活动：渠道、目标、时间、目标人群 |

**数据质量问题与处理：**

- `traffic_source` 存在大小写不一致（10 类 → 合并为 5 类）
- `gross_revenue` 存在负值，对应退款记录（3,029 条），分析时剔除
- `events.product_id` 缺失对应 bounce 事件，业务上合理，未做删除
- `transactions.product_id` 缺失对应退款记录，同样保留

---

## 三、分析方法

**技术栈：** Python（Pandas / NumPy / Matplotlib / Seaborn / SciPy）

**分析步骤：**

1. **数据清洗**：统一渠道大小写、区分退款记录、确认缺失值业务含义
2. **漏斗分析**：按 `session_id` 构建会话级漏斗（view → click → add_to_cart → purchase）
3. **A/B/C 实验分析**：事件级 + 会话级双口径对比，卡方检验验证显著性
4. **渠道分层**：按获客渠道对比购买转化率
5. **可视化**：漏斗对比图、渠道购买率图

---

## 四、核心发现

### 4.1 漏斗诊断：两大瓶颈

![漏斗对比图](output/funnel_by_experiment.png)

| 环节 | 转化率 | 流失率 |
| :--- | :--- | :--- |
| view → click | 43.4% | **56.6%** |
| click → add_to_cart | 80.0% | 20.0% |
| add_to_cart → purchase | 41.2% | **58.8%** |
| view → purchase | 14.3% | 85.7% |

**结论：** view→click 是最大瓶颈（流失 56.6%），cart→purchase 是第二大瓶颈（流失 58.8%）。

### 4.2 A/B/C 实验：Variant_B 显著领先

**关键转折：** 初步按用户级对比时，发现 Control 购买率 43%，实验组仅 19%，方向反常识。进一步检查发现 **99.96% 的用户跨组**，说明实验分组是事件级而非用户级，用户级对比无效。改用事件级与会话级双口径重新分析。

**会话级结果：**

| 组 | 会话数 | 购买会话 | 购买率 | 相对 Control |
| :--- | :--- | :--- | :--- | :--- |
| Control | 316,596 | 44,625 | 14.10% | — |
| Variant_A | 105,523 | 15,071 | 14.28% | +1.3% |
| **Variant_B** | 104,783 | 15,620 | **14.91%** | **+5.7%** |

**卡方检验：** chi² = 42.36，p < 0.001，三组差异显著。

**事件级交叉验证：** Variant_B 在 cart→purchase 环节转化率 44.76%，相对 Control（33.27%）提升 **34.5%**。

**结论：** Variant_B 是明确赢家，提升集中在结算环节（cart→purchase），对前序环节无影响。

### 4.3 渠道分析：差异有限

![渠道购买率](output/channel_purchase_rate.png)

| 渠道 | 用户数 | 购买率 |
| :--- | :--- | :--- |
| Email | 14,992 | 65.15% |
| Paid Search | 20,013 | 65.08% |
| Social | 14,946 | 64.65% |
| Direct | 9,946 | 63.23% |
| Organic | 40,103 | 63.07% |

**结论：** 渠道间差异有限（63%-65%），核心增长点在实验版本，不在渠道优化。

---

## 五、业务建议

1. **优先全量上线 Variant_B 的结算页改动**：提升集中在 cart→purchase 环节，按会话级估算可带动整体购买率提升约 0.81 个百分点。
2. **优化商品列表页（PLP）**：view→click 是最大瓶颈，流失 56.6%，建议优化推荐逻辑与商品卡片信息。
3. **渠道投放维持现状**：各渠道转化率差异有限，不建议大幅调整预算分配。

---

## 六、项目结构
├── README.md
├── ecommerce_analysis.ipynb # 完整分析 Notebook
└── output/
├── funnel_result.csv # 漏斗各环节数值
├── ab_session_funnel.csv # 会话级 A/B/C 漏斗
├── ab_event_funnel.csv # 事件级 A/B/C 漏斗
├── channel_purchase_rate.csv # 渠道购买率
├── session_purchase_rate.csv # 各组会话级购买率
├── chi_square_result.csv # 卡方检验结果
├── summary.csv # 关键结果汇总
├── event_distribution.png # 事件类型分布
├── funnel_by_experiment.png # 三组漏斗对比图
└── channel_purchase_rate.png # 渠道购买率图


---

## 七、如何复现

1. 从 Kaggle 下载数据集：https://www.kaggle.com/datasets/geethasagarbonthu/marketing-and-e-commerce-analytics-dataset
2. 安装依赖：`pip install pandas numpy matplotlib seaborn scipy`
3. 打开 `ecommerce_analysis.ipynb`，按顺序执行所有单元格
4. 输出文件会自动保存到 `output/` 目录

---

## 八、项目亮点

- **方法严谨**：发现用户跨组后，改用事件级与会话级双口径交叉验证，避免错误归因
- **数据敏感**：识别渠道大小写不一致、退款导致的负收入、缺失值业务含义
- **业务落地**：结论直接指向可执行建议，并量化预期提升幅度
