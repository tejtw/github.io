<span id="menu"></span>
# Universe Analysis－using get_universe

分析股票池的產業分布與成交金額

## 選單

1. [分析臺灣50指數成份股公司的產業分佈](#臺灣50)
2. [分析臺灣中型100指數成份股公司的產業分佈](#臺灣100)
3. [分析臺灣高股息指數成份股公司的產業分佈](#臺灣高股息)
4. [分析電子工業公司的產業分佈](#電子工業)
5. [分析上市ETF成交金額](#上市ETF)

```
import tejapi
import os
import numpy as np
import pandas as pd

# set tej_key and base
os.environ['TEJAPI_KEY'] = "your key" 
os.environ['TEJAPI_BASE'] = "https://api.tej.com.tw"

# set date
start = end = '2024-05-31'

from matplotlib import pyplot as plt
plt.rc("font",family='MicroSoft YaHei',weight="bold")

import TejToolAPI
from zipline.sources.TEJ_Api_Data import get_universe
from zipline.utils.calendar_utils import get_calendar
```

利用`get_universe`取得台灣50指數成份股

```
tw50_ = get_universe(start, end, idx_id='IX0002')
```
![alt text](image-26.png)

```
tw50_ 
```
![alt text](image-27.png)

`getUniverseSector`：繪製股票池產業分佈柱狀圖與圓餅圖

```
def plot_sector_counts(sector_counts):
    
    # create bar chart of number of companies in each sector    
    from matplotlib import pyplot as plt
    plt.rc("font",family='MicroSoft YaHei',weight="normal")
    
    from matplotlib.ticker import MaxNLocator
    import matplotlib.ticker as ticker
        
    plt.figure(figsize=(12, 15), dpi=100)
    
    bar = plt.subplot2grid((5,5), (0,0), rowspan=2, colspan=5)
    pie = plt.subplot2grid((5,5), (2,0), rowspan=3, colspan=5)
    
    # Bar chart
    sector_counts.plot(
        kind='barh',        
        color='b',
#         rot=90,
        grid=True,
        fontsize=12,
        ax=bar,
    )

    plt.gca().yaxis.set_major_locator(ticker.MaxNLocator(integer=True))
    bar.set_title('股票池產業分布家數')
    bar.set_xlabel('家數')     

    
    num = list(sector_counts.values)
    explode = [0.1 if x == max(num) else 0 for x in num]
    
    # Pie chart
    sector_counts.plot(
        kind='pie', 
        colormap='Set3', 
        autopct='%.1f %%', # '%.2f %%'
        fontsize=12,
        ax=pie,
        labeldistance=1.1,
        pctdistance=0.9,
        explode = explode
    ) 
    pie.set_ylabel('')      
    pie.set_title('股票池產業分布占比 - %')
    
    
    plt.tight_layout(pad=5);
    
def getUniverseSector(start_date,
                      end_date,
                      trading_calendar=get_calendar('TEJ_XTAI'),
                      **kwargs):
    
    tickers = get_universe(start_date,
                           end_date,
                           trading_calendar = trading_calendar,
                           **kwargs)
        
    df_sector = TejToolAPI.get_history_data(ticker=tickers,
                                            columns=['Industry'], transfer_to_chinese=True,
                                            start = start_date,
                                            end = end_date)   

    counts = (df_sector.groupby('主產業別_中文').size())
    _c =[]
    counts.index = [ x.split(' ')[1]  if len(x)>0 else ' ' for x in counts.index]
    
    plot_sector_counts(counts[counts>0].sort_values(ascending=False)) 
```

<span id="臺灣50"></span>
# 分析臺灣50指數成份股公司的產業分佈
[Return to Menu](#menu)

```
getUniverseSector(start, end, idx_id='IX0002')
```
![alt text](image-28.png)
![alt text](image-29.png)

<span id="臺灣100"></span>
# 分析臺灣中型100指數成份股公司的產業分佈
[Return to Menu](#menu)

```
getUniverseSector(start, end, idx_id='IX0003')
```
![alt text](image-30.png)
![alt text](image-31.png)

<span id="臺灣高股息"></span>
# 分析臺灣高股息指數成份股公司的產業分佈
[Return to Menu](#menu)

```
getUniverseSector(start, end, idx_id='IX0006')
```
![alt text](image-32.png)
![alt text](image-33.png)

<span id="電子工業"></span>
# 分析電子工業公司的產業分佈
[Return to Menu](#menu)

```
getUniverseSector(start, end, main_ind_c='M2300 電子工業')
```
![alt text](image-34.png)
![alt text](image-35.png)

<span id="上市ETF"></span>
# 分析上市ETF成交金額
[Return to Menu](#menu)

```
etf = get_universe(start, end, stktp_c=['ETF', '國外ETF'], mkt=['TWSE'])
```

```
df_amount = TejToolAPI.get_history_data(ticker=etf, 
                                        columns=['Value_Dollars'], 
                                        transfer_to_chinese=False,
                                        start = '2023-01-01',
                                        end = end
                                        )  
```

```
df_top = (df_amount.
          set_index(['coid','mdate']).
          unstack('coid').
          rolling(30).
          mean().
          iloc[-1].
          sort_values(ascending=False)['Value_Dollars'] #['成交金額_元']
         )
```

```
df_top = (df_top.to_frame().
          join(tejapi.get('TWN/APISTOCK')[['coid','stk_name']].
               set_index('coid')).
          set_index('stk_name').iloc[:,0]
         )
```

```
plt.figure(figsize=(8, 12), dpi=150)
    
bar = plt.subplot2grid((5,5), (0,0), rowspan=2, colspan=5)
    
df_top.nlargest(20).plot(
        kind='barh',        
        color='b',
#         rot=90,
        grid=True,
        ax=bar
    )
   
bar.set_xlabel('TWD')
bar.set_ylabel('')
bar.set_title('上市ETF 過去30日的平均成交金額 Top20（{}）'.format(df_amount.mdate.max().strftime('%Y-%m-%d')))
```

![alt text](image-36.png)
[Return to Menu](#menu)