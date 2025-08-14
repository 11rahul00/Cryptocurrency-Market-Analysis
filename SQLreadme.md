
````markdown
# 📊 Cryptocurrency SQL Analysis

This project analyzes cryptocurrency market data from **CoinMarketCap** for the years **2017** and **2018**.  
It explores market capitalization, volume trends, price changes, volatility, and correlations using advanced SQL queries.

---

## 📂 Dataset Structure

### Table: `cmc2017` & `cmc2018`

| Column Name          | Data Type  | Description |
|----------------------|-----------|-------------|
| id                   | TEXT      | Unique coin identifier |
| name                 | TEXT      | Coin name |
| symbol               | TEXT      | Ticker symbol |
| rank                 | INTEGER   | Rank by market cap |
| market_cap_usd       | NUMERIC   | Market capitalization in USD |
| "24h_volume_usd"     | NUMERIC   | 24-hour trading volume in USD |
| percent_change_24h   | NUMERIC   | % change in last 24 hours |
| percent_change_7d    | NUMERIC   | % change in last 7 days |

### Table: `daily_crypto_data` *(for Q19)*

| Column Name          | Data Type  | Description |
|----------------------|-----------|-------------|
| symbol               | TEXT      | Ticker symbol |
| date                 | DATE      | Trading date |
| "24h_volume_usd"     | NUMERIC   | 24-hour trading volume in USD |

---

## 🔍 Queries & Explanations

### **Q1: Top 10 Coins by Market Cap (2017)**
```sql
SELECT name, market_cap_usd
FROM cmc2017
ORDER BY market_cap_usd DESC
LIMIT 10;
````

Shows the largest cryptocurrencies by market cap in 2017.

---

### **Q2: Top 10 Coins by Market Cap (2018)**

```sql
SELECT name, market_cap_usd
FROM cmc2018
ORDER BY market_cap_usd DESC
LIMIT 10;
```

Compares to Q1 to track dominance changes in 2018.

---

### **Q3: Coins with Over 50% Weekly Growth (2018)**

```sql
SELECT name, percent_change_7d
FROM cmc2018
WHERE percent_change_7d > 50
ORDER BY percent_change_7d DESC;
```

Finds high-growth coins for short-term investment insights.

---

### **Q4: Count of Coins Missing Market Cap (2017)**

```sql
SELECT COUNT(*) AS null_market_cap
FROM cmc2017
WHERE market_cap_usd IS NULL;
```

Identifies data gaps for cleaning and quality checks.

---

### **Q5: Market Cap Share of Top 10 vs Rest (2018)**

```sql
SELECT 
 ROUND(SUM(CASE WHEN rank <= 10 THEN market_cap_usd ELSE 0 END) * 100.0 / SUM(market_cap_usd), 2) AS top10_pct,
 ROUND(SUM(CASE WHEN rank > 10 THEN market_cap_usd ELSE 0 END) * 100.0 / SUM(market_cap_usd), 2) AS rest_pct
FROM cmc2018;
```

Shows market concentration among top players.

---

### **Q6: Rank Coins by 24h Volume (2018)**

```sql
SELECT name, "24h_volume_usd",
       RANK() OVER (ORDER BY "24h_volume_usd" DESC) AS vol_rank
FROM cmc2018;
```

Ranks coins by liquidity for trading activity insights.

---

### **Q7: Total Market Cap by Symbol (2018)**

```sql
SELECT symbol, SUM(market_cap_usd) AS total_market_cap
FROM cmc2018
GROUP BY symbol
ORDER BY total_market_cap DESC;
```

Aggregates market cap for coins sharing the same ticker.

---

### **Q8: Biggest Gainers (24h) – 2018**

```sql
SELECT name, percent_change_24h
FROM cmc2018
ORDER BY percent_change_24h DESC
LIMIT 10;
```

Lists short-term top performers.

---

### **Q9: Biggest Losers (7d) – 2018**

```sql
SELECT name, percent_change_7d
FROM cmc2018
ORDER BY percent_change_7d ASC
LIMIT 10;
```

Identifies worst weekly performers.

---

### **Q10: Year-over-Year Market Cap Growth**

```sql
SELECT c17.name,
       ROUND((c18.market_cap_usd - c17.market_cap_usd) / c17.market_cap_usd * 100, 2) AS pct_growth
FROM cmc2017 c17
JOIN cmc2018 c18 ON LOWER(c17.id) = LOWER(c18.id)
WHERE c17.market_cap_usd IS NOT NULL
  AND c18.market_cap_usd IS NOT NULL
ORDER BY pct_growth DESC
LIMIT 10;
```

Finds coins with the highest YoY market cap growth.

---

### **Q11: Largest YoY Increase in 24h Volume**

```sql
SELECT c17.name,
       ROUND((c18."24h_volume_usd" - c17."24h_volume_usd") / c17."24h_volume_usd" * 100, 2) AS pct_increase
FROM cmc2017 c17
JOIN cmc2018 c18 ON LOWER(c17.id) = LOWER(c18.id)
WHERE c17."24h_volume_usd" IS NOT NULL
  AND c18."24h_volume_usd" IS NOT NULL
ORDER BY pct_increase DESC
LIMIT 5;
```

Highlights the most traded coins in relative growth.

---

### **Q12: Coins Entered Top 20 in 2018**

```sql
SELECT id, name
FROM cmc2018
WHERE id NOT IN (
    SELECT id FROM cmc2017
    ORDER BY market_cap_usd DESC LIMIT 20
)
ORDER BY market_cap_usd DESC
LIMIT 20;
```

Identifies newcomers to the top 20.

---

### **Q12b: Coins Dropped from Top 20 in 2018**

```sql
SELECT id, name
FROM cmc2017
WHERE id NOT IN (
    SELECT id FROM cmc2018
    ORDER BY market_cap_usd DESC LIMIT 20
)
ORDER BY market_cap_usd DESC
LIMIT 20;
```

Shows coins that lost dominance.

---

### **Q13: Moving Average of 7-Day Returns**

```sql
SELECT name, percent_change_7d,
       ROUND(AVG(percent_change_7d) OVER (
             ORDER BY percent_change_7d
             ROWS BETWEEN 4 PRECEDING AND CURRENT ROW
       ), 2) AS moving_avg_5
FROM cmc2018
ORDER BY percent_change_7d DESC;
```

Calculates a rolling 5-coin average for weekly returns.

---

### **Q14: Median 7-Day Return by Year**

```sql
SELECT year,
       PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY percent_change_7d) AS median_7d_return
FROM (
    SELECT percent_change_7d, 2017 AS year FROM cmc2017
    UNION ALL
    SELECT percent_change_7d, 2018 AS year FROM cmc2018
) combined
GROUP BY year;
```

Provides a better central measure of weekly returns.

---

### **Q15: Top 5 Coins by Volatility (24h % Change)**

```sql
SELECT name,
       symbol,
       ROUND(STDDEV_SAMP(percent_change_24h), 2) AS stddev_24h
FROM cmc2018
WHERE percent_change_24h IS NOT NULL
GROUP BY name, symbol
ORDER BY stddev_24h DESC
LIMIT 5;
```

Measures price volatility to assess risk.

---

### **Q16: Correlation between Market Cap and Volume**

```sql
WITH categorized AS (
    SELECT *,
           CASE
               WHEN market_cap_usd >= 10000000000 THEN 'Large Cap'
               WHEN market_cap_usd >= 1000000000  THEN 'Mid Cap'
               ELSE 'Small Cap'
           END AS cap_category
    FROM cmc2018
)
SELECT cap_category,
       ROUND(
         (SUM((market_cap_usd - (SELECT AVG(market_cap_usd) FROM categorized WHERE cap_category = c.cap_category)) *
              ("24h_volume_usd" - (SELECT AVG("24h_volume_usd") FROM categorized WHERE cap_category = c.cap_category))) /
          ((COUNT(*) - 1) * (STDDEV_SAMP(market_cap_usd) * STDDEV_SAMP("24h_volume_usd")))
       , 3) AS correlation
FROM categorized c
GROUP BY cap_category;
```

Shows relationship between liquidity and size.

---

### **Q17: Ranking Coins by Average Weekly Return**

```sql
SELECT name,
       ROUND(AVG(percent_change_7d), 2) AS avg_weekly_return,
       RANK() OVER (ORDER BY AVG(percent_change_7d) DESC) AS rank_by_return
FROM cmc2018
WHERE percent_change_7d IS NOT NULL
GROUP BY name
ORDER BY avg_weekly_return DESC
LIMIT 10;
```

Ranks coins based on average performance.

---

### **Q18: Clusters of Coins with Similar Weekly Returns**

```sql
SELECT a.name AS coin_a,
       b.name AS coin_b,
       ABS(a.percent_change_7d - b.percent_change_7d) AS diff_pct
FROM cmc2018 a
JOIN cmc2018 b ON a.name < b.name
WHERE a.percent_change_7d IS NOT NULL
  AND b.percent_change_7d IS NOT NULL
  AND ABS(a.percent_change_7d - b.percent_change_7d) <= 10
ORDER BY diff_pct ASC
LIMIT 20;
```

Groups coins with close weekly performances.

---

### **Q19: Rolling 7-Day Trading Volume**

```sql
WITH top10 AS (
    SELECT symbol
    FROM cmc2018
    ORDER BY market_cap_usd DESC
    LIMIT 10
),
rolling AS (
    SELECT t.symbol,
           t.date,
           SUM(t."24h_volume_usd") OVER (
             PARTITION BY t.symbol
             ORDER BY t.date
             ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
           ) AS rolling_7d_volume
    FROM daily_crypto_data t
    JOIN top10 ON t.symbol = top10.symbol
)
SELECT *
FROM rolling
ORDER BY symbol, date;
```

Tracks short-term liquidity trends for the largest coins.

---

## 📌 Summary

These queries cover:

* Market structure analysis
* Price and volume movements
* Risk & volatility measures
* Trend and correlation insights

They can be extended for:

* Multi-year comparisons
* Real-time dashboards
* Investment screening tools
