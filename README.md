# stock

import tushare as ts
import pandas as pd
from pandas import DataFrame
import numpy as np
import datetime
import time

from datetime import timedelta
import matplotlib.pyplot as plt
from mpl_finance import candlestick_ochl
from matplotlib.pylab import date2num
import matplotlib as mpl
import seaborn as sns
from matplotlib import ticker

result_list = []
#获取数据接口
pro = ts.pro_api('9b1829f5b0a8901e73f4297d03e12db5ef6550d8b99b8d0417b8eb6f')
#获取上市日期大于一年的股票列表
d = timedelta(days=-365)
curr_time = datetime.datetime.now()
curr_time = curr_time+d
curr_time_str = curr_time.strftime('%Y%m%d')
df_stock_list = pro.query('stock_basic', exchange='', list_status='L', fields='ts_code,symbol,name,area,industry,list_date')
df_stock_list_1 = df_stock_list[df_stock_list['list_date'] < curr_time_str]
print(df_stock_list_1.head())

ts_code_list = df_stock_list_1.head(1000)

df_scanned_list = pd.read_excel('scanned_list.xlsx')

def stock_judging(ts_code,start_date):
    df = pro.daily(ts_code= ts_code, start_date=curr_time_str)
    df = df.sort_values(by='trade_date', ascending=True)
    df['trade_date2'] = df['trade_date'].copy()
    df['trade_date'] = pd.to_datetime(df['trade_date']).map(date2num)
    df['dates'] = np.arange(0, len(df))

    df['30'] = df.close.rolling(30).mean()
    df['72'] = df.close.rolling(72).mean()
    df['144'] = df.close.rolling(144).mean()
    df['250'] = df.close.rolling(250).mean()

    df_1 = df[-1:]
    df_2 = df[-2:]
    if len(df_1) > 0:
        print(df_1.ix[0, ['ts_code']])
        if bool(df_1.ix[0, ['close']].values > df_2.ix[0, ['close']].values):
            if df_1.ix[0, ['30']].values > df_1.ix[0, ['72']].values:
                if df_1.ix[0, ['72']].values > df_1.ix[0, ['144']].values:
                    if df_1.ix[0, ['144']].values > df_1.ix[0, ['250']].values:
                        return df_1.iloc[0, 0].values


for ts_code in ts_code_list['ts_code']:
    result = stock_judging(ts_code, curr_time_str)
    if result:
        result_list.append(result)
    #DataFrame(ts_code).to_excel('scanned_list.xlsx', sheet_name='sheet1', index=False, header=True)

    print(result_list)
    time.sleep(30)
print(result_list)
'''     
df = pro.index_daily(ts_code='399300.SZ', start_date='20190101')

df = df.sort_values(by='trade_date', ascending=True)
df['trade_date2'] = df['trade_date'].copy()
df['trade_date'] = pd.to_datetime(df['trade_date']).map(date2num)
df['dates'] = np.arange(0, len(df))

df['30'] = df.close.rolling(30).mean()
df['72'] = df.close.rolling(72).mean()
df['144'] = df.close.rolling(144).mean()
df['250'] = df.close.rolling(250).mean()


df_1 = df[-1:]
df_2 = df[-2:]
print(df_1)
print(df_1.iloc[0,3])

if df_1.iloc[0,3]> df_2.iloc[0,2]:
    if df_1['30'] > df_1['72']:
        if df_1['72'] > df_1['144']:
            if df_1['144'] > df_1['250']:
                print (df_1.iloc[0,0])





print(stock_judging('399300.SZ','20190101'))

'''

'''
sns.set()
mpl.rcParams['font.family'] = 'sans-serif'
mpl.rcParams['font.sans-serif'] = 'SimHei'


def format_date(x,pos):
    if x<0 or x>len(date_tickers)-1:
        return ''
    return date_tickers[int(x)]
date_tickers = df.trade_date2.values
fig, ax = plt.subplots(figsize=(10, 5))
ax.xaxis.set_major_formatter(ticker.FuncFormatter(format_date))
#调用方法，绘制K线图
candlestick_ochl(ax = ax,
                 quotes=df[['dates', 'open', 'close', 'high', 'low']].values,
                 width=0.7,
                 colorup='red',
                 colordown='green',
                 alpha=0.7)
#df['close'].rolling(window=3).mean().plot(color='red', label='3')
#df['close'].rolling(window=5).mean().plot(color='blue', label='5')
#df['close'].rolling(window=10).mean().plot(color='green', label='10')
for ma in ['30', '72', '144', '250']:
    plt.plot(df['dates'], df[ma])
plt.legend() #绘制图例
#设置X轴的标签
plt.xticks(range(len(df.index.values)), df.index.values, rotation=30 )

plt.title('399300')
plt.show()
'''
