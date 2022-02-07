
# Betting Against Beta

Implementation of Trading Algorithm Betting Against 
Beta based on Andrea Frazzini and Lasse H.Pedersen's
 research paper.
Regression on the CAPM equation is used to get Beta.

The project's aim is to generate an equally weighted portfolio.

#### Final Backtest Performance
![Logo](https://user-images.githubusercontent.com/69301816/152737059-dbae8f86-5e0d-42dc-8427-7307ba26caa2.PNG)
![App Screenshot](https://user-images.githubusercontent.com/69301816/152737690-c2d7b7e8-a647-44c1-afeb-51614a96f900.PNG)

## Motivation

- Apply learnings from the EPAT course, and
- Create Proof points of applicability and relevance of this theory in the Indian stock markets

## Documentation

#### What is Risks in Investment?

There are two types of risks

| Idiosyncratic risk | Systematic risk |
| :-------- | :------- |
| `Specific to firm` | `Market wide` |
| `Doesn't affect the whole market` | `Affect all the firm` |
| `Diversified away` | `Cannot be diversified away` |

The Systematic risk can be captured through beta.

#### Beta value Interpretation

| Beta | Interpretation |
| :-------- | :------- |
|Greater than 1| More volatile than the market|
|Close to 1|As volatile as the market|
|Between 0 and 1 |Less volatile than the market|
|0|Not correlated to the market|
|Less than 0|Negatively correlated to the market|

For example, if a stock's beta is 1.2, it's theoretically 20% more volatile than the market.

#### Calculate Beta

```bash
import statsmodels.api as sm

def calc_beta(stock_series, bechmark_series):
    model = sm.OLS(stock_series, bechmark_series)
    results = model.fit()
    return results.params[0]
```






## Strategy Rationale

- People prefer assets with higher expected returns per unit of risk
- Mutual funds and retail investor are constrained in the leverage
- They tilt allocation to high beta assets to improve returns resulting in over pricing


## Strategy Backtesting

- Read the Nifty component stock list
````bash
nifty_list = pd.read_csv('ind_nifty50list.csv')
````

- Read the price data of the stocks
````bash
data = pd.read_csv('nifty_stocks_prices.csv',index_col=0)
data.index = pd.to_datetime(data.index)
````
- Calculate Daily percentage change
````
data_pct_change = data.pct_change().dropna
````

- Calculate the beta on train data (2013 - 2017, to avoid look ahead bias) and plotting the data 
````bash
beta = pd.DataFrame(index=[0])
for ticker in data_pct_change.columns:
    beta[ticker] = calc_beta(data_pct_change.loc[:'2017',ticker],data_pct_change.loc[:'2017','^NSEI'])
beta = beta.T
beta.columns = ['beta']

bar_p = beta.plot.bar(figsize=(18,7))

plt.show()
````
![App Screenshot](https://user-images.githubusercontent.com/69301816/152636465-848193d2-ee76-4fb5-a407-e284d37e0c79.PNG)

- Generate signals and calculate strategy returns
Creating portfolio with low beta stocks 
### Alpha 1
````bash
low_beta = beta[beta.values < 0.7].index
low_beta
````
![App Screenshot](https://user-images.githubusercontent.com/69301816/152729050-db2b2738-02d3-40dc-add1-722076e70599.PNG)

````
def plot_performance(stock_list, strategy_name):
    stk_returns = data_pct_change.loc['2018':, stock_list]
    (stk_returns+1).cumprod().plot(figsize=(15,7),legend="left")
    plt.title(strategy_name)
    plt.show()

    nifty = data_pct_change.loc['2018':, '^NSEI'] #Benchmark
    portfolio = stk_returns.mean(axis=1)
    plt.title(strategy_name + ' Portfolio Performance')
    (portfolio+1).cumprod().plot(figsize=(15,7),label=strategy_name, color='purple')
    (nifty+1).cumprod().plot(figsize=(15,7),label='Nifty', color='blue')
    plt.legend()
    plt.show()
    return portfolio
````
![App Screenshot](https://user-images.githubusercontent.com/69301816/152735921-b8173460-0d43-49a1-9367-1d5565eddb0a.PNG)
![App Screenshot](https://user-images.githubusercontent.com/69301816/152735940-8e3e9d1a-149e-4426-a1af-9cd22f695313.PNG)

Portfolio performanced well compared to Nifty index,even during the downfall and the recovery was even better.

Our strategy has an advantage since it falls less when the market falls and rises more when the market rises, gives edge to low beta portfolio.

## Alpha 2

We're going to add one more filter to our strategy, which is Return on Equity.
For all stocks, we're using ROE figures till 2017(to avoid look ahead bias).
### ROE > 18
````
high_roe = nifty_list[nifty_list.ROE>18].Symbol.values
````

Combining Alpha
````
filtered_stocks = low_beta & high_roe
````

- Generate the signal based on beta and ROE

````
portfolio = plot_performance(filtered_stocks, 'Low Beta & High ROE')
````
![App Screenshot](https://user-images.githubusercontent.com/69301816/152736124-ba7c7706-9be6-48a6-a4cf-590f57aa65d1.PNG)
![App Screenshot](https://user-images.githubusercontent.com/69301816/152736136-ee4483f6-9d2e-44e9-91c7-94ee0cc0321f.PNG)


- Returns analysis using pyfolio
````
import pyfolio as pf
pf.create_simple_tear_sheet(portfolio,benchmark_rets=data_pct_change.loc['2018':, '^NSEI'])
````
![App Screenshot](https://user-images.githubusercontent.com/69301816/152737059-dbae8f86-5e0d-42dc-8427-7307ba26caa2.PNG)
![App Screenshot](https://user-images.githubusercontent.com/69301816/152737690-c2d7b7e8-a647-44c1-afeb-51614a96f900.PNG)
![App Screenshot](https://user-images.githubusercontent.com/69301816/152737894-3c62ecf7-bc59-400f-b30a-472fe2bc2950.PNG)


## Reference

 - [Nifty50 Historical Constituents Data:](https://www1.nseindia.com/products/content/equities/indices/archieve_indices.htm)
 - [Screener for Fundamental data](https://www.screener.in/)
 - [Betting Against Beta](https://www.sciencedirect.com/science/article/pii/S0304405X13002675)

