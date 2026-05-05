# How pandemic severity relates to market performance: the relationship between COVID severity and stock prices of pharmaceutical companies


## Contributors: 
- Bowen Zhao
- Yuhan Zhou


## Summary

In this project, we explored the relationship between a global health catastrophe, COVID-19, and the resilience of capital markets. Specifically, we investigate how the severity of the COVID-19 pandemic—measured through infection and mortality rates—influenced the financial performance of the United States health sector during the critical window between January 1, 2021, to January 1, 2022. This period is particularly significant as it included the period of the vaccines, the emergence of the Delta and Omicron variants, and a shift in market sentiment from panic to strategic long-term valuation of healthcare equities.

The motivation behind this study stems from the nature of the COVID outbreak. Unlike traditional economic shocks driven by housing or credit bubbles, this was a biological catalyst that restructured global priorities fundamentally. For the healthcare and pharmaceutical industry, the pandemic presented a reality that a moral obligation to innovate and a massive commercial opportunity. We wanted to quantify this impact. As the threat of the virus grew more dreadful in the eyes of the public, we want to discover in the project that whether the stock market respond in a linear fashion to the rising death counts, or did it anticipate medical breakthroughs before they reflected in health statistics.

Therefore, our research centralizes on the question: What is the statistical and predictive relationship between U.S. COVID-19 case/death counts and the stock price fluctuations of major pharmaceutical firms during 2021?

**Methodology and Technical Execution**

In order to answer this, our team developed a analytical pipeline designed to merge different data streams. We utilized the Johns Hopkins University COVID-19 Repository for high-resolution health data and the yfinance API to extract historical financial time-series data (including Open, High, Low, and Closing prices).

The execution required some data engineering. To acheive this, we employed SQL for relational mapping and Python’s Pandas library for initial data cleaning. A primary challenge evolved in data cleaning and normalization, which we had to standardize namings of the daily records and align the seven-day-a-week health data with the five-day-a-week trading schedule of the New York Stock Exchange. In addition to simple correlation, we implemented machine learning models via scikit-learn to identify the sensitivity of these stocks, thereby attempt to see if pandemic-driven news cycles could serve as a leading indicator for stock volatility.

**Findings and Gaps**
- PLACEHOLDER FOR UNFINISHED FINDING PART!!!
We constructed three OLS regression models using daily stock returns as the dependent variable and three pandemic-related metrics as independent variables: `Severity_Index`, `Infection_Momentum`, and `Mortality_Momentum`. The results across all three companies reveal that COVID-19 metrics have very limited explanatory power on daily stock returns.

However, the study has some constraints. The reliance on country-level aggregated data created a regional gap, which obscured how localized policy between states shifts or vaccination rates in specific states could have influenced broader market trends. In addition, we identified a gap that, while correlations exist, they are influenced by latent variables such as government stimulus packages and broader shifts in interest rates.

Ultimately, this project highlights that during a global health crisis, the pharmaceutical sector functions as a highly speculative asset class. The results underscore the necessity for investors and policymakers to look beyond raw health statistics and consider the broader socio-economic ecosystem when predicting market behavior in times of biological volatility.



## Data Profile

### **COVID-19 Data Repository (JHU CSSE)**

The data is sourced from the Johns Hopkins University Center for Systems Science and Engineering (CSSE). It consists of daily CSV reports of the pandemic's status globally and within the U.S.
- Format: Daily CSV files (Relational mapping via Date).  
- Key Variables:
Geographic Identifiers: Province_State, Country_Region, Lat, Long_, FIPS (unique US county codes), UID, ISO3.

Public Health Metrics: Confirmed (total cases), Deaths (total fatalities), Recovered, and Active (calculated as Confirmed - Deaths - Recovered).

Analytical Rates: Incident_Rate (cases per 100k), Case_Fatality_Ratio (percentage of confirmed cases resulting in death), and Mortality_Rate.

Infrastructure Data: Total_Test_Results, People_Hospitalized, Testing_Rate.

- Characteristics: This is time-series aggregate data. Since our project focuses on the U.S., we filter for Country_Region == 'US'. The data is cumulative, requiring us to calculate "Daily New Cases" by subtracting $Day_{n}$ from $Day_{n-1}$.

### **Pharmaceutical Stock Market Dataset (yfinance)**

This dataset was generated using the yfinance library, which interfaces with Yahoo Finance to pull historical market data for three specific tickers: CVS (CVS Health), JNJ (Johnson & Johnson), and ABBV (AbbVie).
- Format: Tabular time-series data (Pandas DataFrame).
- Variables:
Date: The primary key for merging with COVID-19 data.

Close: The closing price of the stock on a given day (our primary dependent variable).

Volume: The total number of shares traded (used to measure market "excitement" or volatility).

Additional Fields: While we focused on Close and Volume, the raw data also includes Open, High, Low, and Adj Close.

- Characteristics: Unlike the COVID-19 data (which is 7 days a week), stock data follows the business calendar. There is no data for weekends or federal holidays. To bridge this gap, we used "forward-filling" or "rolling averages" in our pipeline.

**Ethical or Legal Constraints**

For the JHU CSSE dataset, it is provided in aggregate form, meaning it contains no Personally Identifiable Information (PII). This circumvents HIPAA concerns, as individual patients cannot be identified. However, ethical care is required when interpreting data from marginalized communities where reporting may have been less accurate. Its licencing is provided under a Creative Commons Attribution 4.0 International (CC BY 4.0) license. It is free for educational and research use, provided that proper credit is given to the JHU CSSE.

For the Pharmaceutical stock market dataset, it is public information. Therefore, the primary ethical concern is the risk of misinformation if we were to present correlation as definitive financial advice. Our project explicitly states it is for retrospective research, not predictive trading. Its licencing is from yfinance, which is an open-source tool that scrapes public Yahoo Finance pages. Although Yahoo Finance's Terms of Service generally prohibit high-frequency commercial scraping, our use case involves a one-time retrieval of historical data for non-commercial research, which generally falls under its Fair Use.



## Data quality

For this project, we assessed the quality of the Johns Hopkins University (JHU) COVID-19 dataset and the yfinance pharmaceutical dataset based on completeness, consistency, and reliability.

### 1. Financial Data Integrity (yfinance)

The stock market data for CVS, JNJ, and ABBV was relatively robust. Upon data extraction and auditing, we observed that there is zero missing values for the primary variables of interest: Date, Close, and Volume.

We believe this high level of completeness is due to the regulated nature of financial reporting. Since we are targeting three major firms on the New York Stock Exchange, the price feeds are continuous and authoritative. The only gap in the timeline were the expected absences of data on weekends and federal holidays. Rather than treating these as missing data points, we recognized them as a structural characteristic of capital markets. To maintain synchronization with the daily health data, we implemented a forward-fill methodology, where the stock price on a Saturday or Sunday is assumed to be the closing price of the preceding Friday.

### 2. Public Health Data Quality (JHU CSSE)

The JHU COVID-19 dataset is a secondary aggregate, meaning its quality is dependent on the reporting accuracy of individual U.S. states and counties. Our assessment revealed several loopholes in null values. 

Our initial audit found 58 null values in the New_Confirmed and New_Deaths columns. Interestingly, all 58 instances occurred on January 1, 2021. Therefore, we think that this is not a failure of data collection, but because of the dataset's structure. Since January 1st marks the start of our specific observation window and the "new" cases are calculated as a delta from the previous day's cumulative total, the absence of a "Day 0" (December 31, 2020) in our specific filtered subset resulted in these initial nulls.

To resolve this without skewing the results, we cross-referenced the cumulative totals from the final day of 2020 to populate these starting values, ensuring that our time-series analysis began with a verified baseline.

### 3. Cross-Dataset Synchronization

The most significant challenge was the relational mapping between the two datasets. We performed a join operation on the Date variable, but because the health data is provided at a county/state level and the stock data is national, we had to aggregate the JHU data into a single national daily total.

We verified the success of this aggregation by comparing our calculated national daily sums against official CDC (Centers for Disease Control and Prevention) reports. 

### 4. Final Quality Verdict

Overall, the datasets are of high quality and high fidelity. The yfinance data provides a clean reflection of market sentiment, while the JHU data—despite the initial null values on January 1st and the inherent noise of public health reporting—remains the gold standard for pandemic tracking. In conclusion, we have established a data foundation that is both statistically sound and technically reliable for investigating the intersection of public health and finance.



## Data cleaning

Our data cleaning process was executed entirely in python, utilizing a robust pipeline to transform raw, fragmented health reports and financial APIs into a refined analytical dataset. The following operations were performed to address specific structural and qualitative issues:

### 1. Longitudinal Integration of COVID-19 Data
The JHU COVID-19 repository stores data as individual daily snapshots. To create a usable timeline, we utilized the os library to iterate through the directory, filtering specifically for files containing "2021." We read each CSV into a list of DataFrames and used pd.concat() to merge them into a single global 2021 master file. For the issues address, we solved the data fragmentation issue, transforming hundreds of isolated files into a continuous longitudinal time-series.

### 2. Temporal Standardization and Delta Calculations
The raw JHU data provides cumulative totals, which are unsuitable for analyzing daily market volatility. We converted the `Date` column to `datetime` objects and sorted by `Province_State` and `Date`. We then applied `.groupby('Province_State').diff()` to the `Confirmed` and `Deaths` columns to generate `New_Confirmed` and `New_Deaths`. This corrected the cumulative bias. By calculating the daily "delta," we were able to measure the actual velocity of the pandemic rather than just the total volume, which is essential for correlation with daily stock returns.

### 3. National Aggregation and Logical Clipping
After calculating state-level changes, we needed a unified U.S. metric to compare against national stock exchanges. We grouped the data by `Date` and summed the new cases/deaths. Furthermore, we used `.clip(lower=0)` on the resulting columns. This addressed geospatial granularity and reporting anomalies. Occasionally, state-level adjustments resulted in negative daily values; "clipping" ensured that our model wasn't skewed by nonsensical negative infection counts.

### 4. Market-Syncing (The "Next Trading Day" Logic)
A critical challenge was that COVID-19 data is reported 7 days a week, while stock markets only operate on business days. We defined a custom function, `assign_trading_day`, which mapped weekend health metrics to the next available trading date. We then re-aggregated the COVID-19 stats based on these trading windows and used an `inner merge` to join with the `yfinance` data. This solved the temporal misalignment(the "weekend gap"). Instead of simply dropping weekend health data, we consolidated it into the Monday market open, reflecting how investors actually process news that accumulates over a weekend.

### 5. Normalization and Feature Engineering
To prepare the data for the Ordinary Least Squares (OLS) regression models seen in the final code blocks, we had to normalize different scales. Our tasks involves, calculating 7-day and 14-day rolling means to smooth out reporting "noise" (weekend lags). We also converted infection and mortality rates into Z-scores (`Infection_Z`, `Mortality_Z`). We then created a weighted feature ($0.3 \times \text{Infection} + 0.7 \times \text{Mortality}$) to provide a single metric of pandemic "dread.", and we converted raw stock prices into `pct_change()` (returns). These operations addressed variance and scale disparity. By converting raw numbers into standardized indices and percentage returns, we ensured that the regression model would not be biased toward variables with larger raw magnitudes (like total cases) over smaller ones (like stock price changes).

### 6. Handling Missing Financial Data
In the final stages, we audited the merged dataset for remaining nulls. We used `.dropna()` specifically on our feature set (`Severity_Index`, `Infection_Momentum`) before fitting the OLS model. This ensured mathematical validity for our statistical models, as scikit-learn and statsmodels cannot process `NaN` values during regression fitting.


## Findings

Our final results for the project includes a OLS analysis report and a stock price vs. new confirmed cases comparison graph for the three companies. 

### OLS Regression Results
 
We constructed three OLS regression models using daily stock returns as the dependent variable and three pandemic-related metrics as independent variables: `Severity_Index`, `Infection_Momentum`, and `Mortality_Momentum`. The results across all three companies reveal that COVID-19 metrics have very limited explanatory power on daily stock returns.
 
- **CVS**: The model produced an R-squared of 0.000 with an F-statistic p-value of 0.994, indicating no statistically significant relationship between pandemic severity and daily returns.
- **JNJ**: The model performed slightly better with an R-squared of 0.030, and `Infection_Momentum` showed marginal significance (p = 0.073), suggesting that short-term surges in infection may have a weak positive association with JNJ returns.
- **ABBV**: The model yielded an R-squared of 0.014, with `Mortality_Momentum` showing borderline significance (p = 0.081) and a negative coefficient (-0.0030), implying that rising death tolls may slightly suppress ABBV returns.
Across all three models, none of the coefficients reached the conventional 5% significance threshold, suggesting that daily pandemic metrics alone are insufficient to predict pharmaceutical stock returns.
 
### Visual Analysis
 
The dual-axis time series plots reveal interesting patterns that complement the regression findings:
 
- **CVS** stock price exhibited a strong upward trend throughout 2021, rising from approximately $59 in January to nearly $90 by year-end, with the most pronounced acceleration occurring during the Delta and Omicron surges in late 2021.
- **JNJ** displayed more volatile behavior, peaking around $157 mid-year before declining and recovering, suggesting that JNJ was more sensitive to vaccine-related news cycles given its role as a vaccine manufacturer.
- **ABBV** showed steady appreciation from $86 to $115, with a notable jump coinciding with the Omicron wave in December 2021.

### Interpretation
 
The divergence between weak statistical significance and visually apparent co-movement suggests that while pharmaceutical stocks broadly benefited during the pandemic period, daily fluctuations in COVID metrics do not directly drive daily returns. Instead, longer-term trends, investor expectations about vaccine demand, and broader market conditions likely play more substantial roles. The high kurtosis values (5.484 for CVS and 8.928 for ABBV) and significant Jarque-Bera statistics indicate non-normal return distributions with fat tails, which is consistent with the heightened market volatility characteristic of the pandemic era.



## Future work

While our current model provides a strong foundation using OLS regression and basic feature engineering, there are several ways to expand this research to provide a more granular understanding of the market's mechanics.

### 1. Sentiment Analysis and the "News Cycle" Gap

Our current analysis relies on quantitative health metrics (cases and deaths) as a proxy for the pandemic's severity. However, market volatility is often driven as much by perception as by reality.

#### Future Direction

A logical next step would be to integrate Natural Language Processing (NLP). By scraping headlines from financial news outlets (e.g., Bloomberg, Reuters) or social media (Twitter/X) and performing sentiment analysis, we could quantify the "fear index" of the market. This would allow us to see if a negative news cycle about a vaccine's efficacy has a more immediate impact on stock prices than the actual mortality rates reported by JHU.

### 2. Expanding the Ticker Universe
Currently, our study focuses on three major players: CVS, JNJ, and ABBV. While these are representative, they do not capture the full diversity of the healthcare sector.

#### Future Direction

 Future work should include a wider array of firms, categorized by their role in the pandemic. We could compare "Vaccine Pioneers" (Moderna, Pfizer) against "Equipment Providers" (Thermo Fisher, Danaher) and "Traditional Providers" (UnitedHealth). Using a Clustering Algorithm could help identify which sub-sectors were most resilient and which were most sensitive to specific pandemic milestones.

### 3. Global Comparative Analysis
The scope of this project was intentionally limited to the United States. However, the pharmaceutical industry is global, and different countries had vastly different regulatory responses and vaccination timelines.

#### Future Direction

 We could expand the dataset to include the European (EMA) and Asian markets. Comparing the sensitivity of the FTSE 100 or DAX pharmaceutical stocks against the S&P 500 would reveal how different government intervention strategies (e.g., lockdowns vs. early vaccine mandates) impacted investor confidence across borders.

### 4. Advanced Predictive Modeling
Our current project uses regression to find relationships. While useful for finding correlation, it is less effective at forecasting.

#### Future Direction
We would like to implement Long Short-Term Memory (LSTM) neural networks. LSTMs are specifically designed for time-series forecasting and could potentially identify non-linear patterns that OLS regression misses. By feeding the model historical case data, policy change dates, and stock returns, we could attempt to build a "Crisis Stress-Test" model to predict how healthcare stocks might behave in the early stages of a future pandemic.

### 5. Incorporating Macroeconomic Covariates
As noted in our research gaps, stock prices are influenced by more than just health data—inflation, interest rates, and government stimulus all played a role in 2021.

#### Future Direction
 Future iterations should include macroeconomic indicators (such as the 10-year Treasury yield or the Consumer Price Index) as control variables. This would allow us to isolate the "COVID effect" from the "Stimulus effect," providing a much cleaner look at the true sensitivity of the pharmaceutical sector to biological threats.


## Challenges

Building this analytical pipeline required navigating several methodological challenges, each of which shaped the final structure of our analysis. 

The first hurdle was data aggregation. Because the Johns Hopkins COVID-19 repository provides daily snapshots distributed across hundreds of separate CSV files, we needed to programmatically join these datasets into a unified time series before any analysis could begin. We accomplished this by iterating through the daily report directory, filtering for files within our 2021 study window, and concatenating them into a single dataframe. From there, regional state-level records were aggregated upward into national-level daily totals, giving us a coherent timeline of pandemic severity across the United States.

The second challenge concerned reconciling the temporal mismatch between public health data and financial market data. While COVID-19 metrics are reported every day of the year, U.S. stock markets are closed on weekends and federal holidays, creating systematic gaps in the financial time series. Rather than forward-filling stock prices into non-trading days—which would artificially smooth volatility and distort correlations—we adopted a trading-day aggregation approach, in which COVID metrics from closed-market days were rolled forward into the next available trading day. This preserved the integrity of the financial data while ensuring no pandemic information was lost from the merged dataset.

The third methodological decision involved constructing a meaningful Severity Index from the cleaned COVID metrics. Because raw daily case and death counts are extremely noisy, with substantial day-of-week reporting effects, we applied rolling averages to smooth short-term fluctuations and reveal underlying trends. These smoothed values were then converted into Z-scores, which standardized the metrics onto a common scale and allowed confirmed cases and deaths to be combined into a single weighted index that could be directly compared against other engineered features.

A parallel transformation was required on the financial side. Raw closing prices for CVS, JNJ, and ABBV are non-stationary and operate on different absolute scales, making cross-stock comparison difficult and regression results unreliable. To address this, we converted prices into daily percentage returns using `pct_change()`, producing a stationary series that more directly reflects how pharmaceutical companies responded to pandemic developments on a day-to-day basis.

Finally, the regression results themselves require careful interpretation. The models produced near-zero R² values and largely non-significant coefficients, which initially appeared disappointing. However, this outcome is consistent with efficient market theory, since by 2021 most COVID-related information had already been priced into pharmaceutical equities, suggesting the null result is itself a substantive analytical finding rather than a modeling failure.


## Reproducing

Here are the seqence to reproduce the results

- Prerequisites and Environment Setup
Before starting, ensure you have a Python environment (3.8 or higher) ready. You will need to install the following libraries via `pip`:
* `pandas` and `numpy` for data manipulation.
* `yfinance` to pull market data.
* `statsmodels` for the regression analysis.
* `matplotlib` and `seaborn` for generating the visualizations.

- Gathering the Raw Data
* **Health Data:** Download the `csse_covid_19_daily_reports_us` folder from the Johns Hopkins University GitHub repository. We specifically filtered for files with "2021" in the filename.
* **Stock Data:** Use the `yfinance` library to download historical data for tickers **"CVS"**, **"JNJ"**, and **"ABBV"** with a start date of `2021-01-01` and an end date of `2022-01-01`.

- The Cleaning Pipeline
The most critical part of reproducing our results is the way the data is cleaned. Follow these steps in order:
1.  **Merging Files:** Use `os.listdir` to loop through the JHU folder, append all 2021 CSVs into a list, and use `pd.concat` to create one big master dataframe.
2.  **Calculating New Cases:** Since the raw data is cumulative, you must sort by `Province_State` and `Date`, then use `.groupby().diff()` to find the actual **daily** new cases and deaths. 
3.  **National Totals:** Group by the `Date` column and `.sum()` the values to get a single daily count for the entire United States. Use `.clip(lower=0)` to fix any negative numbers caused by state reporting adjustments.

- 4. Syncing the Datasets (The "Weekend Gap")
Because the virus spreads every day but the stock market closes on weekends, you cannot do a simple join.
* Define a function (`assign_trading_day`) that looks at a COVID date and finds the **next available** trading date. 
* Re-aggregate your COVID numbers based on these trading days so that weekend virus data is "captured" and moved to the following Monday.
* Perform an `inner merge` on the `Date` column between your adjusted COVID data and the stock price data.

- 5. Creating Features and Running the Model
* **Smoothing:** Calculate a **7-day rolling average** for new infections and a **14-day rolling average** for mortality to remove the "noise" from irregular weekend reporting.
* **Standardization:** Convert these rolling averages into **Z-scores** (subtract the mean and divide by the standard deviation).
* **Severity Index:** Create a custom "Severity Index" variable by weighting the Infection Z-score at 30% and the Mortality Z-score at 70%.
* **Regression:** Use `sm.OLS()` from the `statsmodels` library. Set your `Severity_Index` and `Momentum` variables as your **X** (independent variables) and the stock's daily percentage return as your **y** (dependent variable). 




## References

### Data Sources
 
Dong, E., Du, H., & Gardner, L. (2020). An interactive web-based dashboard to track COVID-19 in real time. *The Lancet Infectious Diseases*, *20*(5), 533–534. https://doi.org/10.1016/S1473-3099(20)30120-1
 
Johns Hopkins University Center for Systems Science and Engineering. (2023). *COVID-19 Data Repository by the Center for Systems Science and Engineering (CSSE) at Johns Hopkins University* [Data set]. GitHub. https://github.com/CSSEGISandData/COVID-19
 
Aroussi, R. (2024). *yfinance: Download market data from Yahoo! Finance's API* (Version 0.2) [Computer software]. GitHub. https://github.com/ranaroussi/yfinance
 
---
 
### Notes on Data Use
 
The JHU CSSE COVID-19 dataset is licensed under the Creative Commons Attribution 4.0 International (CC BY 4.0) license. As required by the data provider, we attribute the data to the "COVID-19 Data Repository by the Center for Systems Science and Engineering (CSSE) at Johns Hopkins University" and cite the accompanying *Lancet Infectious Diseases* publication by Dong, Du, and Gardner (2020).
 
The yfinance library is distributed under the Apache Software License 2.0 and is an open-source tool that accesses Yahoo! Finance's publicly available APIs. yfinance is not affiliated with, endorsed by, or vetted by Yahoo, Inc., and is intended for research and educational purposes only.


