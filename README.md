# Inflation-Data-Scraper
Inflation data are relevant in times of large monetary payouts to show the risk of demonetization. Furthermore, the gold price correlates with the real interest rate, which is determined by the official bank rate and the inflation rate.


Consumer price inflation cannot be predicted into the future therefore you should never base your MONEY investment on the inflation rate


<img src= "InflationRate-Month united-states.png" width="800">

How is inflation caused?
Money is created only when loans are made possible and or when central banks buy bonds themselves.

Wages: when the economy is running, more jobs are needed, the more labor in demand, the better the negotiating chances to get high wages.

More retirees lead to more inflation: less available workforce, with the same production input. Workers have to perform more, therefore higher wages. 

Technological progress: Personnel input can be saved, this also leads to inflation.





To download the following data I found a website where monthly inflation data is listed in tables. Using a Python script, the monthly inflation data for different countries and given years are downloaded, saved to a CSV file and then visualized.

# Initialization

````python

from pandas_datareader import data as pdr
import yfinance as yf
import matplotlib.pyplot as plt
from datetime import date
from dateutil.relativedelta import relativedelta
import pandas as pd
import numpy as np
import os, sys
import time
import os
from os import makedirs
import shutil
from currency_converter import CurrencyConverter
import quandl as ql
from bs4 import BeautifulSoup
import requests
import re
import datetime
import matplotlib.pyplot as plt
import datetime
%matplotlib inline


UnixTime = int(time.time())
currentdate = datetime.datetime.fromtimestamp(UnixTime).strftime('%Y-%m-%d')

Grundpfad = 'C:/*****/Scraping_InflationsData/'
URL = 'https://www.inflation.eu/en/inflation-rates/LAND/historic-inflation/cpi-inflation-LAND-JAHR.aspx'
````

# Webscraping

````python
############ Variables ##########
Countries = ['germany', 'great-britain', 'france', 'united-states']
Jahre = list(range(1956, 2021))

data = {
'Year': [],
'Month': [],
'Day': [],
'Country': [],
'Inflation-Rate': []}
df = pd.DataFrame(data)

for Country in Countries:
    for Jahr in Jahre:
        #print(Jahr)
        neue_URL = URL.replace("JAHR", str(Jahr))
        neue_URL = neue_URL.replace("LAND", Country)
        ## get Source-Code
        response = requests.get(neue_URL).text
        soup = BeautifulSoup(response, 'html.parser')

        # find a few relevant markers
        soup2 = str(soup)
        ind1=soup2.find('<table cellpadding="2" cellspacing="0" style="width:100%;border:1px solid #CCCCCC;">') # gebe Zahl aus ab wann in dem Char soup der Begriff fällt
        ind2=soup2.find('<h2 style="margin:16px 0px 4px 0px;">Historic CPI inflation ') # gebe Zahl aus ab wann in dem Char soup der Begriff fällt
        a = soup2[ind1:ind2] # zeige soup2 von marker 1 bis marker 2 an | find char in api char

        # make a beautifulsoup file again
        soup3 = BeautifulSoup(a)

        # extract table content
        Month = 1
        for tr in soup3.find_all('tr')[1:]: #header [1:]
            tds = tr.find_all('td')
            #print(str(Jahr) + '-' + str(Month))
            #print(tds[4].text[:-3]) # zeige spaltenweise [4] Inhalte an    
            data = {
            'Year': [Jahr][0],
            'Month': [Month][0],
            'Day': [1][0],
            'Country': [Country][0],
            'Inflation-Rate': [tds[4].text[:-3]][0]}
            df = df.append(data, ignore_index=True)
            Month = Month + 1

fmt = '%1.0f', '%1.0f', '%1.0f', '%s', '%s'
Datenpfad = Grundpfad + currentdate + '_Inflationsdaten.csv'
np.savetxt(Datenpfad, df, delimiter=",", fmt=fmt)
````

---
## Output csv file

    
| YEAR | MONTH | DAY | Country | InflationRate |
|     :---:      | :---:      | :---:      | :---:      | :---:  | 
| 1956 | 1 | 1 | germany | 1.25 |
| 1956 | 2 | 1 | germany | 2.19 |
| 1956 | 2 | 1 | germany | 3.76 |
| ... | ... | ... | ... | ... | ... |
| 2020 | 12 | 1 | united-states | 1.36 |



# Visualization of monthly data 

````python
import pandas as pd
Datenpfad = Grundpfad + currentdate + '_Inflationsdaten.csv'
# df = pd.read_csv(filename, sep='\t', encoding = 'utf-16',nrows=10,  index_col="CASE") #Filename, Tabulator, Textcodierung, lade nur die ersten 10 Reihen ein, nehme Case als Index
df = pd.read_csv(Datenpfad, sep=',', header=None)
df.columns = ['Year', 'Month', 'Day', 'Country', 'InflationRate']

#### neue Zeitachse
date = pd.to_datetime(dict(year=df.Year, month=df.Month,  day=df.Day)) # convert coloums format into python understandable datetimeframe
df.set_index(date, inplace = True) # set new index



###### country sorted
Countries = ['germany', 'great-britain', 'france', 'united-states']
Country = Countries[3]

###### plot 
fig, ax1 = plt.subplots(figsize=(20,4.5))
ax1.grid(color='black', linestyle='--', linewidth=0.05)
ax1.title.set_text('Monthly Inflation-Rate in '+ Country)
ax1.set_xlabel('date')
ax1.set_ylabel('Inflationsrate')
ax1.plot(df[(df["Country"] == Country)].InflationRate, label='Inflation-Rate')
fig.tight_layout()  # otherwise the right y-label is slightly clipped
plt.savefig("InflationRate-Month " + Country + ".png", bbox_inches='tight')
````

<img src= "InflationRate-Month united-states.png" width="800">
<img src= "InflationRate-Month france.png" width="800">
<img src= "InflationRate-Month germany.png" width="800">
<img src= "InflationRate-Month great-britain.png" width="800">

# Visualization annual data

In addition to monthly inflation rates, annual inflation rates can also be determined

```` python
import pandas as pd
Datenpfad = Grundpfad + currentdate + '_Inflationsdaten.csv'
Datenpfad = Grundpfad + '2021-02-11_Inflationsdaten.csv'
# df = pd.read_csv(filename, sep='\t', encoding = 'utf-16',nrows=10,  index_col="CASE") #Filename, Tabulator, Textcodierung, lade nur die ersten 10 Reihen ein, nehme Case als Index
df = pd.read_csv(Datenpfad, sep=',', header=None)
df.columns = ['Year', 'Month', 'Day', 'Country', 'InflationRate']

data = {
'Year': [],
'Country': [],
'InflationRateYear': []}
df_year = pd.DataFrame(data)

for Country in Countries:
    for Jahr in Jahre:
        new_df = df.loc[(df['Year'] == Jahr) & (df['Country'] == Country)]
        data = {
        'Year': [Jahr][0],
        'Country': [Country][0],
        'InflationRateYear': [new_df.InflationRate.mean()][0]}
        df_year = df_year.append(data, ignore_index=True)

fmt = '%1.0f', '%s', '%1.2f'
Datenpfad = Grundpfad + currentdate + '_Inflationsdaten_Year.csv'
np.savetxt(Datenpfad, df_year, delimiter=",", fmt=fmt)


###### nach ländern sortieren
Countries = ['germany', 'great-britain', 'france', 'united-states']
Country = Countries[3]

###### plotten 
fig, ax1 = plt.subplots(figsize=(20,4.5))
ax1.grid(color='black', linestyle='--', linewidth=0.1)
ax1.title.set_text('Yearly Inflation-Rate in '+ Country)
ax1.set_xlabel('date')
ax1.set_ylabel('Inflationsrate')
ax1.plot(df_year[(df_year["Country"] == Country)].Year, df_year[(df_year["Country"] == Country)].InflationRateYear, label='Inflation-Rate')
fig.tight_layout()  # otherwise the right y-label is slightly clipped
plt.savefig("InflationRate-Year " + Country + ".png", bbox_inches='tight')
````

<img src= "InflationRate-Year united-states.png" width="800">



