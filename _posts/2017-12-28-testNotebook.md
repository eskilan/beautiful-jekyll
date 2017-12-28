---
layout: post
title: Testing a notebook
tags: [test]
---


We begin by loading all required libraries


```python
import pandas as pd
import numpy as np
import os
```

See what is in the directory


```python
print(os.listdir())
```

    ['sample.py', 'Restaurant_prediction.ipynb', 'sample_submission.csv', 'hashtable_btb.csv', 'air_store_info.csv', 'hpg_store_info.csv', 'ideas.txt', 'Untitled1.ipynb', 'Untitled.ipynb', 'hpg_reserve.csv', '.ipynb_checkpoints', 'air_visit_data.csv', 'Untitled.py', 'date_info.csv', 'air_reserve.csv', 'store_id_relation.csv']


We load the data


```python
air_store_info = pd.read_csv('air_store_info.csv')
air_store_info.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>air_store_id</th>
      <th>air_genre_name</th>
      <th>air_area_name</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>air_0f0cdeee6c9bf3d7</td>
      <td>Italian/French</td>
      <td>Hyōgo-ken Kōbe-shi Kumoidōri</td>
      <td>34.695124</td>
      <td>135.197852</td>
    </tr>
    <tr>
      <th>1</th>
      <td>air_7cc17a324ae5c7dc</td>
      <td>Italian/French</td>
      <td>Hyōgo-ken Kōbe-shi Kumoidōri</td>
      <td>34.695124</td>
      <td>135.197852</td>
    </tr>
    <tr>
      <th>2</th>
      <td>air_fee8dcf4d619598e</td>
      <td>Italian/French</td>
      <td>Hyōgo-ken Kōbe-shi Kumoidōri</td>
      <td>34.695124</td>
      <td>135.197852</td>
    </tr>
    <tr>
      <th>3</th>
      <td>air_a17f0778617c76e2</td>
      <td>Italian/French</td>
      <td>Hyōgo-ken Kōbe-shi Kumoidōri</td>
      <td>34.695124</td>
      <td>135.197852</td>
    </tr>
    <tr>
      <th>4</th>
      <td>air_83db5aff8f50478e</td>
      <td>Italian/French</td>
      <td>Tōkyō-to Minato-ku Shibakōen</td>
      <td>35.658068</td>
      <td>139.751599</td>
    </tr>
  </tbody>
</table>
</div>



The file contains store_id, genre, area name, and coordinates. Let's try a new one


```python
hpg_store_info = pd.read_csv('hpg_store_info.csv')
hpg_store_info.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>hpg_store_id</th>
      <th>hpg_genre_name</th>
      <th>hpg_area_name</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>hpg_6622b62385aec8bf</td>
      <td>Japanese style</td>
      <td>Tōkyō-to Setagaya-ku Taishidō</td>
      <td>35.643675</td>
      <td>139.668221</td>
    </tr>
    <tr>
      <th>1</th>
      <td>hpg_e9e068dd49c5fa00</td>
      <td>Japanese style</td>
      <td>Tōkyō-to Setagaya-ku Taishidō</td>
      <td>35.643675</td>
      <td>139.668221</td>
    </tr>
    <tr>
      <th>2</th>
      <td>hpg_2976f7acb4b3a3bc</td>
      <td>Japanese style</td>
      <td>Tōkyō-to Setagaya-ku Taishidō</td>
      <td>35.643675</td>
      <td>139.668221</td>
    </tr>
    <tr>
      <th>3</th>
      <td>hpg_e51a522e098f024c</td>
      <td>Japanese style</td>
      <td>Tōkyō-to Setagaya-ku Taishidō</td>
      <td>35.643675</td>
      <td>139.668221</td>
    </tr>
    <tr>
      <th>4</th>
      <td>hpg_e3d0e1519894f275</td>
      <td>Japanese style</td>
      <td>Tōkyō-to Setagaya-ku Taishidō</td>
      <td>35.643675</td>
      <td>139.668221</td>
    </tr>
  </tbody>
</table>
</div>



This is the same but for the reservation system. Let's now look at the hpg_reserve and air_reserve sheets


```python
hpg_reserve = pd.read_csv('hpg_reserve.csv')
hpg_reserve.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>hpg_store_id</th>
      <th>visit_datetime</th>
      <th>reserve_datetime</th>
      <th>reserve_visitors</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>hpg_c63f6f42e088e50f</td>
      <td>2016-01-01 11:00:00</td>
      <td>2016-01-01 09:00:00</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>hpg_dac72789163a3f47</td>
      <td>2016-01-01 13:00:00</td>
      <td>2016-01-01 06:00:00</td>
      <td>3</td>
    </tr>
    <tr>
      <th>2</th>
      <td>hpg_c8e24dcf51ca1eb5</td>
      <td>2016-01-01 16:00:00</td>
      <td>2016-01-01 14:00:00</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>hpg_24bb207e5fd49d4a</td>
      <td>2016-01-01 17:00:00</td>
      <td>2016-01-01 11:00:00</td>
      <td>5</td>
    </tr>
    <tr>
      <th>4</th>
      <td>hpg_25291c542ebb3bc2</td>
      <td>2016-01-01 17:00:00</td>
      <td>2016-01-01 03:00:00</td>
      <td>13</td>
    </tr>
  </tbody>
</table>
</div>




```python
air_reserve = pd.read_csv('air_reserve.csv')
air_reserve.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>air_store_id</th>
      <th>visit_datetime</th>
      <th>reserve_datetime</th>
      <th>reserve_visitors</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>air_877f79706adbfb06</td>
      <td>2016-01-01 19:00:00</td>
      <td>2016-01-01 16:00:00</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>air_db4b38ebe7a7ceff</td>
      <td>2016-01-01 19:00:00</td>
      <td>2016-01-01 19:00:00</td>
      <td>3</td>
    </tr>
    <tr>
      <th>2</th>
      <td>air_db4b38ebe7a7ceff</td>
      <td>2016-01-01 19:00:00</td>
      <td>2016-01-01 19:00:00</td>
      <td>6</td>
    </tr>
    <tr>
      <th>3</th>
      <td>air_877f79706adbfb06</td>
      <td>2016-01-01 20:00:00</td>
      <td>2016-01-01 16:00:00</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>air_db80363d35f10926</td>
      <td>2016-01-01 20:00:00</td>
      <td>2016-01-01 01:00:00</td>
      <td>5</td>
    </tr>
  </tbody>
</table>
</div>



Let's now look at the remaining data frames. Date info shows the calendar date with corresponding day of week and holiday flag


```python
date_info = pd.read_csv('date_info.csv')
date_info.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>calendar_date</th>
      <th>day_of_week</th>
      <th>holiday_flg</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2016-01-01</td>
      <td>Friday</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2016-01-02</td>
      <td>Saturday</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2016-01-03</td>
      <td>Sunday</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2016-01-04</td>
      <td>Monday</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2016-01-05</td>
      <td>Tuesday</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
store_id_relation = pd.read_csv('store_id_relation.csv')
store_id_relation.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>air_store_id</th>
      <th>hpg_store_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>air_63b13c56b7201bd9</td>
      <td>hpg_4bc649e72e2a239a</td>
    </tr>
    <tr>
      <th>1</th>
      <td>air_a24bf50c3e90d583</td>
      <td>hpg_c34b496d0305a809</td>
    </tr>
    <tr>
      <th>2</th>
      <td>air_c7f78b4f3cba33ff</td>
      <td>hpg_cd8ae0d9bbd58ff9</td>
    </tr>
    <tr>
      <th>3</th>
      <td>air_947eb2cae4f3e8f2</td>
      <td>hpg_de24ea49dc25d6b8</td>
    </tr>
    <tr>
      <th>4</th>
      <td>air_965b2e0cf4119003</td>
      <td>hpg_653238a84804d8e7</td>
    </tr>
  </tbody>
</table>
</div>




```python
store_id_relation.shape
```




    (150, 2)



The last dataframe to load is called "air_visit_data"


```python
air_visit_data = pd.read_csv('air_visit_data.csv')
air_visit_data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>air_store_id</th>
      <th>visit_date</th>
      <th>visitors</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>air_ba937bf13d40fb24</td>
      <td>2016-01-13</td>
      <td>25</td>
    </tr>
    <tr>
      <th>1</th>
      <td>air_ba937bf13d40fb24</td>
      <td>2016-01-14</td>
      <td>32</td>
    </tr>
    <tr>
      <th>2</th>
      <td>air_ba937bf13d40fb24</td>
      <td>2016-01-15</td>
      <td>29</td>
    </tr>
    <tr>
      <th>3</th>
      <td>air_ba937bf13d40fb24</td>
      <td>2016-01-16</td>
      <td>22</td>
    </tr>
    <tr>
      <th>4</th>
      <td>air_ba937bf13d40fb24</td>
      <td>2016-01-18</td>
      <td>6</td>
    </tr>
  </tbody>
</table>
</div>




```python
print('There are %d stores in air' % air_store_info.shape[0])
print('There are %d stores in hpg' % hpg_store_info.shape[0])
```

    There are 829 stores in air
    There are 4690 stores in hpg



```python
whos
```

    Variable            Type         Data/Info
    ------------------------------------------
    air_reserve         DataFrame                   air_store_<...>n[92378 rows x 4 columns]
    air_store_info      DataFrame                 air_store_id<...>n\n[829 rows x 5 columns]
    air_visit_data      DataFrame                    air_store<...>[252108 rows x 3 columns]
    date_info           DataFrame        calendar_date day_of_<...>n\n[517 rows x 3 columns]
    hpg_reserve         DataFrame                     hpg_stor<...>2000320 rows x 4 columns]
    hpg_store_info      DataFrame                  hpg_store_i<...>\n[4690 rows x 5 columns]
    np                  module       <module 'numpy' from '/ho<...>kages/numpy/__init__.py'>
    os                  module       <module 'os' from '/home/<...>da3/lib/python3.6/os.py'>
    pd                  module       <module 'pandas' from '/h<...>ages/pandas/__init__.py'>
    store_id_relation   DataFrame                 air_store_id<...>n\n[150 rows x 2 columns]


Let's take a look at the sample submission

sample_submission = pd.read_csv('sample_submission.csv')
sample_submission.head(50)

We can merge the hpg and air "reservation" data into a single reservations table and join hpg ids into "air ids"

We select the the relevant hpg reservations which are those that have air stores


```python
indices = hpg_reserve.hpg_store_id.isin(store_id_relation.hpg_store_id)
```


```python
hpg_reserve_toJoin = hpg_reserve[indices]
hpg_reserve_toJoin['res_sys']='hpg'
air_reserve_toJoin = air_reserve
air_reserve_toJoin['res_sys']='air'
```

    /home/ilan/anaconda3/lib/python3.6/site-packages/ipykernel_launcher.py:2: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      


Substitute hpg store id for air store id


```python
newIds = []
count = 0
for index, row in hpg_reserve_toJoin.iterrows():
    newIds.append(store_id_relation.air_store_id[row.hpg_store_id == store_id_relation.hpg_store_id].values[0])
airIDs = pd.Series(newIds)
```

We join the reservations into a single data frame


```python
hpg_reserve_toJoin.drop('hpg_store_id',axis=1,inplace=True)
hpg_reserve_toJoin['air_store_id'] = airIDs.values
```

    /home/ilan/anaconda3/lib/python3.6/site-packages/ipykernel_launcher.py:1: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      """Entry point for launching an IPython kernel.
    /home/ilan/anaconda3/lib/python3.6/site-packages/ipykernel_launcher.py:2: SettingWithCopyWarning: 
    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead
    
    See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
      



```python
reservations = air_reserve_toJoin.append(hpg_reserve_toJoin)
reservations.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 120561 entries, 0 to 2000313
    Data columns (total 5 columns):
    air_store_id        120561 non-null object
    res_sys             120561 non-null object
    reserve_datetime    120561 non-null object
    reserve_visitors    120561 non-null int64
    visit_datetime      120561 non-null object
    dtypes: int64(1), object(4)
    memory usage: 5.5+ MB



```python

```

    Interactive namespace is empty.

