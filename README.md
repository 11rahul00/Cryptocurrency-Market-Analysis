
## 📚 About the Project

This project analyzes a full snapshot of the cryptocurrency market as of **December 6, 2017**, using data scraped from CoinMarketCap's public API. We explore trends in market capitalization, volatility, and the classification of cryptos by size.

> ⚠️ **Disclaimer**: This is **not financial advice**. Cryptocurrencies are volatile and speculative. Historical performance is not indicative of future results.

---

## 🗂️ Dataset

* 📁 **File**: `datasets/coinmarketcap_06122017.csv`
* 📌 **Source**: [CoinMarketCap](https://coinmarketcap.com) (via public API, which is no longer available as of 2020)

| Column Name          | Description                          |
| -------------------- | ------------------------------------ |
| `id`                 | Cryptocurrency name (e.g., bitcoin)  |
| `market_cap_usd`     | Market capitalization in USD         |
| `percent_change_24h` | 24-hour price change (%)             |
| `percent_change_7d`  | 7-day price change (%)               |
| `...`                | Other metadata fields (not analyzed) |

---

## 🧰 Tech Stack

* 🐍 Python 3
* 🐼 Pandas
* 📈 Matplotlib
* 📊 Jupyter Notebook

---

## ⚙️ Setup Instructions

```bash
pip install pandas matplotlib jupyter
```

Then open the notebook:

```bash
jupyter notebook
```

---

## ✅ Tasks & Code Summary

### 🔹 1. Load and Inspect Data

```python
import pandas as pd
import matplotlib.pyplot as plt

%matplotlib inline
%config InlineBackend.figure_format = 'svg' 
plt.style.use('fivethirtyeight')

dec6 = pd.read_csv('datasets/coinmarketcap_06122017.csv')
market_cap_raw = dec6[['id', 'market_cap_usd']]
market_cap_raw.count()
```

---

### 🔹 2. Filter Out Coins Without Market Cap

```python
cap = market_cap_raw.query('market_cap_usd > 0')
cap.count()
```

---

### 🔹 3. Bitcoin vs. Other Top Coins

```python
cap10 = cap[:10].set_index('id')
cap10 = cap10.assign(market_cap_perc = lambda x: (x.market_cap_usd / cap.market_cap_usd.sum()) * 100)

ax = cap10.market_cap_perc.plot.bar(title='Top 10 market capitalization')
ax.set_ylabel('% of total cap')
```

---

### 🔹 4. Improve Plot Readability (log scale, color coding)

```python
COLORS = ['orange', 'green', 'orange', 'cyan', 'cyan', 'blue', 'silver', 'orange', 'red', 'green']
ax = cap10.market_cap_usd.plot.bar(title='Top 10 market capitalization', logy=True, color=COLORS)
ax.set_ylabel('USD')
ax.set_xlabel('')
```

---

### 🔹 5. Explore Volatility (24h & 7d Change)

```python
volatility = dec6[['id', 'percent_change_24h', 'percent_change_7d']]
volatility = volatility.set_index('id').dropna().sort_values('percent_change_24h')
volatility.head()
```

---

### 🔹 6. Top 10 Winners & Losers (24h)

```python
def top10_subplot(volatility_series, title):
    fig, axes = plt.subplots(nrows=1, ncols=2, figsize=(10, 4))
    fig.suptitle(title, y=1.05)

    volatility_series[:10].plot.bar(color="darkred", ax=axes[0])
    axes[0].set_ylabel('% change')

    volatility_series[-10:].plot.bar(color="darkblue", ax=axes[1])
    return fig, axes

fig, ax = top10_subplot(volatility['percent_change_24h'], "24 hours top losers and winners")
```

---

### 🔹 7. Weekly Winners & Losers

```python
volatility7d = volatility.sort_values('percent_change_7d')
fig, ax = top10_subplot(volatility7d['percent_change_7d'], "Weekly top losers and winners")
```

---

### 🔹 8. Large-Cap Cryptos

```python
largecaps = cap.query('market_cap_usd > 1E+10')
print(largecaps)
```

---

### 🔹 9. Market Cap Segmentation

```python
def capcount(query_string):
    return cap.query(query_string).count().id

LABELS = ["biggish", "micro", "nano"]
values = [
    capcount("market_cap_usd > 3E+8"),
    capcount("market_cap_usd >= 5E+7 & market_cap_usd < 3E+8"),
    capcount("market_cap_usd < 5E+7")
]

plt.bar(range(len(values)), values, tick_label=LABELS)
```

---

## 🧠 Key Insights

* **Bitcoin** held over **\$200B** market cap on Dec 6, 2017
* Some coins gained or lost **500%+ in a week**, highlighting extreme volatility
* Most cryptocurrencies had **very small** market capitalizations
* High volatility correlates with **low market cap** (i.e., high risk)

---

## 📈 Future Enhancements

* Track evolution of top coins over time (2017–present)
* Add clustering to group coins by volatility or tech similarity
* Use modern APIs (e.g., CoinGecko) for up-to-date analysis
* Interactive dashboard using **Plotly** or **Streamlit**

---

Would you like this turned into:

* 📓 A ready-to-run `.ipynb` notebook?
* 🌐 An interactive dashboard?
* 📦 A public GitHub project template?

Let me know and I’ll generate/export it for you.
