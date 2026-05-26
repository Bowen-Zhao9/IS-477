# COVID-19 严重程度与医药股市场表现分析

这个项目研究 2021 年美国 COVID-19 疫情严重程度与主要医药、医疗健康公司股票表现之间的关系。项目将 Johns Hopkins University CSSE COVID-19 日度疫情数据与 `yfinance` 获取的股票时间序列数据进行整合，分析确诊病例、死亡病例、疫情动量等公共卫生指标是否能够解释或预测 CVS、Johnson & Johnson（JNJ）和 AbbVie（ABBV）等公司的股价波动。

本项目的目标不是构建一个可直接用于投资决策的交易模型，而是展示一个完整的数据分析流程：如何从多个异构数据源中提取、清洗、对齐和融合时间序列数据，并通过可视化与回归模型探索公共卫生危机和资本市场表现之间的统计关系。

## 项目背景

COVID-19 是一次由公共卫生事件引发的全球性冲击。与传统金融危机不同，疫情对医药和医疗健康行业的影响具有双重性：一方面，感染和死亡人数上升会增加社会风险和市场不确定性；另一方面，疫苗、药品、医疗服务和保险需求也可能提升相关公司的市场预期。

因此，本项目关注的问题是：当疫情在美国持续发展时，医药股是否会随着病例和死亡数据变化而出现可观测的价格反应？这种关系是否足够稳定，能够通过统计模型捕捉？项目以 2021 年 1 月 1 日至 2022 年 1 月 1 日为研究窗口，因为这一时期包含疫苗推广、Delta 与 Omicron 变种传播，以及市场从早期恐慌转向长期估值调整的过程。

## Notebook 做了什么

这个 notebook 展示了一个从原始数据到分析结论的端到端流程：

1. 读取 Johns Hopkins CSSE 提供的美国 COVID-19 日度 CSV 报告。
2. 筛选 2021 年数据，并将分散的每日文件合并为一个连续时间序列。
3. 将州级疫情数据聚合为美国全国层面的每日新增确诊和新增死亡数据。
4. 处理累计数据，使用差分方法计算每日新增病例和死亡人数。
5. 使用 `yfinance` 获取 CVS、JNJ 和 ABBV 的历史股票数据。
6. 对齐疫情数据的每日频率与股票市场的交易日频率，处理周末和节假日造成的时间错位。
7. 构造 7 日和 14 日滚动均值、感染动量、死亡动量与综合 `Severity_Index`。
8. 将原始股价转换为日收益率，减少非平稳时间序列对模型的影响。
9. 使用 OLS 回归模型检验疫情指标与股票收益率之间的统计关系。
10. 生成时间序列图，对比疫情波动与三家公司股价变化趋势。

## 使用的数据

- **Johns Hopkins University CSSE COVID-19 数据集**：提供美国各州每日累计确诊、死亡、住院、检测等疫情指标。本项目主要使用确诊和死亡数据，并通过差分计算每日新增值。
- **yfinance 股票数据**：通过 Yahoo Finance 获取 CVS、JNJ 和 ABBV 在 2021 年的历史行情数据，包括收盘价、成交量等字段。本项目主要使用收盘价计算日收益率。

两个数据源的核心挑战在于时间粒度不同。疫情数据每天更新，而股票市场只在交易日开放。项目通过交易日映射和聚合逻辑，将周末累积的疫情信息映射到下一个交易日，尽量模拟投资者在市场重新开盘时接收信息的方式。

## 展示的能力

- 使用 Python 和 pandas 批量读取、合并和清洗 CSV 文件
- 处理时间序列数据中的日期格式、累计值和缺失值
- 将区域级公共卫生数据聚合到全国层面
- 使用 `yfinance` 获取金融市场历史数据
- 对齐公共卫生数据与金融交易日数据
- 设计疫情严重程度指标和动量特征
- 使用 rolling average、Z-score 和 percentage return 进行特征工程
- 使用 `statsmodels` 构建 OLS 回归模型
- 使用 matplotlib 和 seaborn 进行趋势可视化
- 解释统计结果、模型限制和潜在混杂变量

## 主要发现

项目构建了三个 OLS 回归模型，分别以 CVS、JNJ 和 ABBV 的日收益率为因变量，以 `Severity_Index`、`Infection_Momentum` 和 `Mortality_Momentum` 为自变量。整体结果显示，COVID-19 日度指标对三家公司股票日收益率的解释力较弱。

- **CVS**：模型几乎没有解释力，说明疫情严重程度指标与 CVS 的日收益率之间没有明显统计关系。
- **JNJ**：`Infection_Momentum` 显示出边缘显著性，可能暗示短期感染增长与 JNJ 收益率之间存在较弱联系。
- **ABBV**：`Mortality_Momentum` 接近边缘显著，并呈负向关系，说明死亡人数上升可能对 ABBV 日收益率产生轻微压力。

从可视化结果看，三家公司股价在 2021 年整体呈现不同程度的上涨或波动，但这些趋势并不能简单归因于每日疫情指标。更可能的解释是，市场已经提前消化了部分疫情信息，同时受到疫苗新闻、行业预期、宏观政策、利率环境和整体市场情绪等因素影响。

## 项目限制

本项目仍然存在一些限制：

- 疫情数据最终聚合到全国层面，无法捕捉州级政策、地区感染差异和地方医疗资源压力。
- 模型主要使用确诊和死亡数据，未纳入疫苗接种率、政策变化、新闻情绪或宏观经济变量。
- OLS 回归适合解释线性关系，但可能无法捕捉金融市场中的非线性和滞后反应。
- 股票价格受多种因素影响，疫情指标和股价之间的相关性不应被解释为直接因果关系。
- `yfinance` 数据适合教学和研究演示，但不应视为高频交易或生产级金融数据源。

## 后续改进方向

可以进一步扩展：

- 加入疫苗接种率、政府政策、利率、通胀和市场指数等控制变量。
- 扩大股票池，比较疫苗公司、药企、医疗服务、保险和设备供应商等不同子行业。
- 使用新闻标题或社交媒体文本进行情绪分析，衡量市场对疫情新闻的反应。
- 将全国层面疫情数据拆分为州级或区域级数据，分析地域差异。
- 尝试 Random Forest、XGBoost、LSTM 等非线性或时间序列预测模型。
- 将 notebook 流程封装为可复用的数据管道，提高可复现性和扩展性。

---
# COVID-19 / Pharmaceutical Stock Analysis Pipeline

A reproducible Snakemake workflow that investigates the relationship between
U.S. COVID-19 severity and the daily performance of three pharmaceutical
stocks (CVS, JNJ, ABBV) over 2021.

## Project structure

```
.
├── Snakefile                       # Workflow definition (DAG)
├── run_all.sh                      # Snakemake-free fallback runner
├── requirements.txt                # Python dependencies
├── config/
│   └── config.yaml                 # All tunable parameters
├── scripts/
│   ├── 01_load_covid.py            # Concatenate JHU CSSE CSVs
│   ├── 02_clean_covid.py           # Daily diffs + national aggregation
│   ├── 03_fetch_stocks.py          # Per-ticker yfinance download
│   ├── 04_integrate.py             # Next-trading-day merge + clip
│   ├── 05_feature_engineering.py   # Rolling, Z-score, momentum, severity
│   ├── 06_analyze.py               # Per-ticker OLS regressions
│   └── 07_visualize.py             # Multi-panel price-vs-cases plot
├── csse_covid_19_daily_reports_us/ # Input data (NOT generated)
├── data/processed/                 # Intermediate CSVs (generated)
├── results/
│   ├── analysis/                   # Regression summaries (generated)
│   └── figures/                    # Plots (generated)
└── logs/                           # Per-rule run logs (generated)
```

## Workflow DAG

```
csse CSVs ─▶ load_covid ─▶ clean_covid ──┐
                                         ├─▶ integrate ─▶ features ─┬─▶ analyze
yfinance ─▶ fetch_stock × N ─────────────┘                          └─▶ visualize
```

## How to run

### With Snakemake (recommended)

```bash
pip install -r requirements.txt
snakemake --cores 1            # full pipeline
snakemake -np                  # dry-run preview
snakemake --forceall --cores 1 # rebuild everything
snakemake clean                # wipe generated artifacts
```

### Without Snakemake

```bash
pip install -r requirements.txt
bash run_all.sh
```

## Configuration

All tunable parameters live in `config/config.yaml`:

- `tickers` — list of stock symbols to analyze
- `start_date` / `end_date` — range passed to yfinance
- `year_filter` — substring used to select COVID CSVs
- `infection_window` / `mortality_window` — rolling-mean windows
- `infection_weight` / `mortality_weight` — composite-index weights

Editing the YAML and re-running `snakemake` re-executes only the downstream
rules whose inputs changed.

## Outputs

| File                                       | Description                                |
|--------------------------------------------|--------------------------------------------|
| `data/processed/covid_raw_combined.csv`    | All daily JHU rows for the chosen year     |
| `data/processed/covid_state_daily.csv`     | Cleaned state-level series with daily diffs|
| `data/processed/covid_us_daily.csv`        | National daily New_Confirmed / New_Deaths  |
| `data/processed/stock_<TICKER>.csv`        | Per-ticker Close / Volume                  |
| `data/processed/integrated.csv`            | COVID + stocks aligned to trading days     |
| `data/processed/features.csv`              | Adds rolling, Z, momentum, severity, returns|
| `results/analysis/ols_<TICKER>.txt`        | Full statsmodels OLS summary               |
| `results/analysis/regression_results.csv`  | Tidy table of all coefficients across stocks|
| `results/figures/stocks_vs_cases.png`      | Multi-panel price-vs-cases visualization   |

## Data source

Place the JHU CSSE daily-report CSVs in
`csse_covid_19_daily_reports_us/` (one per day, named `MM-DD-YYYY.csv`).
They can be obtained from
<https://github.com/CSSEGISandData/COVID-19/tree/master/csse_covid_19_data/csse_covid_19_daily_reports_us>.

Stock prices are pulled from the public Yahoo Finance API by `yfinance`
and require an internet connection at runtime.
