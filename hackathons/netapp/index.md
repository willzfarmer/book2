# NetApp

Brian McKean from NetApp presented to the class a data analysis problem he'd
like to get some help on.

# Overview

Listen to his presentation carefully. Write down the answers to the following
questions to show your team's understanding of the basics of the problem.

## What is the problem?

> What errors are there with the data acquisition process?

## Why is the problem important?

> We need to verify the data integrity.

## What dataset has been made available?

| Date | System Serial | Controller | Obs. Time | Base Time | Delta | Release | FW | SW |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

## What specific questions are being raised?

1. > Are things getting better over time?
2. > What percent of data is "good"?
3. > Is the "good" data random?

# Q/A session

We will run a Q/A session. Before the session, compile a list of questions you
want to ask. Then, teams will take turn to ask Brian follow up questions.
Each team gets to ask one question each time. Write down the questions your team
wanted to ask and the answers you received. If another team happens to ask the
same question, simply write down the answer you heard.

## Is there any more data? (More columns?)

> No, those are the only columns.

This implies that the problem is actually looking at the issues with the data
collection process, not the write-problem.

## What is the ideal behavior?

> Ideally, there is one entry that spans a day from each system once per day.

# Approach

Based on the information you've obtained during the Q/A session, come up with
plan how your team will tackle this problem.

[Examine the Python Analysis Here](NetAppDataAnalysis.html)

## How should the dataset be imported into Tableau?


## How should the work be distributed among the team members?


## What types of charts or visualizations to use to support the answer?


## What questions may be too complex for Tableau and may require custom scripts to be written?

# Python Approach

## Set Up Imports


```python
import numpy as np
import pandas as pd
import bqplot.pyplot as blt
import matplotlib.pyplot as plt

import time
import datetime as dt

import pysparkling

conn = pysparkling.Context()

from ipywidgets import interact

%matplotlib inline
```

## Load Data

We're going to use the `pandas` module to load the data, as its csv stuff is really nice.


```python
data = pd.read_csv('netapp.csv', header=0,
                   names=['date', 'system', 'controller', 'obs_time',
                          'base_time', 'delta', 'release', 'fw', 'sw'],
                   dtype={'system': object})
```

Now we want to look at the what the rows look like.


```python
data.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>system</th>
      <th>controller</th>
      <th>obs_time</th>
      <th>base_time</th>
      <th>delta</th>
      <th>release</th>
      <th>fw</th>
      <th>sw</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01/01/2015</td>
      <td>1016FG000254</td>
      <td>A</td>
      <td>1420149619</td>
      <td>1420063224</td>
      <td>86395</td>
      <td>Galena</td>
      <td>07.86.34.00</td>
      <td>10.86.0G00.0028</td>
    </tr>
    <tr>
      <th>1</th>
      <td>01/01/2015</td>
      <td>1016FG000432</td>
      <td>A</td>
      <td>1420125162</td>
      <td>1420091715</td>
      <td>33447</td>
      <td>Galena</td>
      <td>07.86.36.30</td>
      <td>10.86.0G00.0048</td>
    </tr>
    <tr>
      <th>2</th>
      <td>01/01/2015</td>
      <td>1017FG000057</td>
      <td>A</td>
      <td>1420102145</td>
      <td>1420015747</td>
      <td>86398</td>
      <td>Galena</td>
      <td>07.86.29.00</td>
      <td>10.86.0G08.0017</td>
    </tr>
    <tr>
      <th>3</th>
      <td>01/01/2015</td>
      <td>1030FG000606</td>
      <td>A</td>
      <td>1420111809</td>
      <td>1420044906</td>
      <td>66903</td>
      <td>Galena</td>
      <td>07.86.45.30</td>
      <td>10.86.0G00.0048</td>
    </tr>
    <tr>
      <th>4</th>
      <td>01/01/2015</td>
      <td>1030FG000619</td>
      <td>A</td>
      <td>1420112025</td>
      <td>1420111359</td>
      <td>666</td>
      <td>Galena</td>
      <td>07.86.45.30</td>
      <td>10.86.0G00.0048</td>
    </tr>
  </tbody>
</table>
</div>



## Analyze Data

We will first examine the integrity of the overall dataset.

### Delta Time Distribution

This first plot looks at the time distribution of the deltas. Ideally we would see a flat line at approx. 86,000. We don't see that.


```python
plt.figure(figsize=(16, 8))
plt.scatter(data['obs_time'], data['delta'], s=1)
plt.xlabel('Linux Epoch Time')
plt.ylabel('Deltas')
plt.show()
```


![png](NetAppDataAnalysis_files/NetAppDataAnalysis_7_0.png)


### Mean and Standard Deviation

We now examine the mean and standard deviation of the deltas for each system. First we generate our data. We will be setting a standard deviation threshold of one hour. Any system data that falls in the deviation we will call "good".


```python
thresh = 3600
grouping = data.groupby('system')
size = len(grouping)
stat_data = np.zeros((size, 4))
i = 0
for key, group in grouping:
    deltas = group['delta']
    stat_data[i][0] = deltas.mean()
    stat_data[i][1] = deltas.std()
    stat_data[i][2] = len(deltas)
    stat_data[i][3] = (255 if deltas.std() <= thresh else 0)
    i += 1
```

We can plot these results.


```python
fig, axarr = plt.subplots(1, 2, figsize=(16, 8))
axarr[0].scatter(stat_data[:, 0], stat_data[:, 1],
                 c=stat_data[:, 3], cmap=plt.cm.coolwarm,
                 alpha=0.2)
axarr[1].scatter(stat_data[:, 0], stat_data[:, 1],
                 c=stat_data[:, 3], cmap=plt.cm.coolwarm,
                 alpha=0.2)
domain = np.arange(-1e7, 7e7)
axarr[0].plot(domain, thresh * np.ones(len(domain)), 'k--')
axarr[1].plot(domain, thresh * np.ones(len(domain)), 'k--')
axarr[0].set_xlabel('Mean Delta Time')
axarr[0].set_ylabel('Standard Deviation')
axarr[1].set_xlabel('Mean Delta Time')
axarr[1].set_ylabel('Standard Deviation')
axarr[1].set_ylim(0, thresh * 2)
plt.show()
```


![png](NetAppDataAnalysis_files/NetAppDataAnalysis_11_0.png)


From this we can interpret several things.

1. The standard deviation of each system is terrible. Only a very small set of systems have consistently "good" data. This means that most systems have bad data.
2. On the other hand, most systems are clustered at the (relative) origin. This is helpful, but many systems are still bad.

Now that we know many systems result in poor data consistency, we can branch out and examine just the rows that we consider "valid", i.e., those rows in which the `delta` field is approximately 86,000 (the length of a day in seconds).

## Good Row Analysis

We start by pulling out only the valid rows. (We also get the "bad" rows, but we'll use those later.)


```python
clean_raw_data = data.values[np.where(np.abs(data.values[:, 5] - 86400) <= 3600)[0]]
dirty_raw_data = data.values[np.where(np.abs(data.values[:, 5] - 86400) > 3600)[0]]
```


```python
clean_data = pd.DataFrame(clean_raw_data,
                        columns=['date', 'system', 'controller', 'obs_time',
                                 'base_time', 'delta', 'release', 'fw', 'sw'])
print(len(clean_data))
clean_data.head()
```

    533203





<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>system</th>
      <th>controller</th>
      <th>obs_time</th>
      <th>base_time</th>
      <th>delta</th>
      <th>release</th>
      <th>fw</th>
      <th>sw</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01/01/2015</td>
      <td>1016FG000254</td>
      <td>A</td>
      <td>1.42015e+09</td>
      <td>1.420063e+09</td>
      <td>86395</td>
      <td>Galena</td>
      <td>07.86.34.00</td>
      <td>10.86.0G00.0028</td>
    </tr>
    <tr>
      <th>1</th>
      <td>01/01/2015</td>
      <td>1017FG000057</td>
      <td>A</td>
      <td>1.420102e+09</td>
      <td>1.420016e+09</td>
      <td>86398</td>
      <td>Galena</td>
      <td>07.86.29.00</td>
      <td>10.86.0G08.0017</td>
    </tr>
    <tr>
      <th>2</th>
      <td>01/01/2015</td>
      <td>1032FG000284</td>
      <td>A</td>
      <td>1.420105e+09</td>
      <td>1.420018e+09</td>
      <td>86395</td>
      <td>Galena</td>
      <td>07.86.32.00</td>
      <td>10.86.0G00.0024</td>
    </tr>
    <tr>
      <th>3</th>
      <td>01/01/2015</td>
      <td>1032FG000288</td>
      <td>B</td>
      <td>1.420074e+09</td>
      <td>1.419988e+09</td>
      <td>86396</td>
      <td>Galena</td>
      <td>07.86.34.00</td>
      <td>10.86.0G00.0013</td>
    </tr>
    <tr>
      <th>4</th>
      <td>01/01/2015</td>
      <td>1032FG000308</td>
      <td>A</td>
      <td>1.420151e+09</td>
      <td>1.420065e+09</td>
      <td>86396</td>
      <td>Galena</td>
      <td>07.86.32.00</td>
      <td>10.86.0G00.0024</td>
    </tr>
  </tbody>
</table>
</div>



We see that we've immediately removed half of the rows. This is arguably good. Notice the term "arguably"...

### Reduction Comparison

We'll start by looking at which systems "lost" the most data. We will do this by comparing the system groupings.


```python
original_grouping = data.groupby('system')
clean_grouping = clean_data.groupby('system')
keys = data['system'].unique()
```

We will now extract the difference from this array.


```python
key_groups = np.zeros((len(keys), 2))
i = 0
for k in keys:
    key_groups[i][0] = len(original_grouping.get_group(k))
    try:
        key_groups[i][1] = len(clean_grouping.get_group(k))
    except KeyError:
        pass
    i += 1
```


```python
key_groups_difference = key_groups[:, 0] - key_groups[:, 1]
```

And now we can plot this.


```python
plt.figure(figsize=(16, 8))
plt.scatter(np.arange(len(key_groups_difference)), key_groups_difference, s=1)
plt.show()
```


![png](NetAppDataAnalysis_files/NetAppDataAnalysis_22_0.png)


We will also do a histogram.


```python
hist, bins = np.histogram(key_groups_difference, bins=200)
center = (bins[:-1] + bins[1:]) / 2
plt.figure(figsize=(16, 8))
plt.bar(center, hist, align='center')
plt.show()
```


![png](NetAppDataAnalysis_files/NetAppDataAnalysis_24_0.png)


From this we can interpret that *many* systems have no change, while many lose their data.

## Release Analysis

Now let's look at the dirty data. We will see if the software and firmware versions affect the number of bad rows.


```python
dirty_data = pd.DataFrame(dirty_raw_data,
                        columns=['date', 'system', 'controller', 'obs_time',
                                 'base_time', 'delta', 'release', 'fw', 'sw'])
print(len(dirty_data))
dirty_data.head()
```

    536778





<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>system</th>
      <th>controller</th>
      <th>obs_time</th>
      <th>base_time</th>
      <th>delta</th>
      <th>release</th>
      <th>fw</th>
      <th>sw</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01/01/2015</td>
      <td>1016FG000432</td>
      <td>A</td>
      <td>1.420125e+09</td>
      <td>1.420092e+09</td>
      <td>33447</td>
      <td>Galena</td>
      <td>07.86.36.30</td>
      <td>10.86.0G00.0048</td>
    </tr>
    <tr>
      <th>1</th>
      <td>01/01/2015</td>
      <td>1030FG000606</td>
      <td>A</td>
      <td>1.420112e+09</td>
      <td>1.420045e+09</td>
      <td>66903</td>
      <td>Galena</td>
      <td>07.86.45.30</td>
      <td>10.86.0G00.0048</td>
    </tr>
    <tr>
      <th>2</th>
      <td>01/01/2015</td>
      <td>1030FG000619</td>
      <td>A</td>
      <td>1.420112e+09</td>
      <td>1.420111e+09</td>
      <td>666</td>
      <td>Galena</td>
      <td>07.86.45.30</td>
      <td>10.86.0G00.0048</td>
    </tr>
    <tr>
      <th>3</th>
      <td>01/01/2015</td>
      <td>1030FG000621</td>
      <td>A</td>
      <td>1.420112e+09</td>
      <td>1.420111e+09</td>
      <td>1205</td>
      <td>Galena</td>
      <td>07.86.45.30</td>
      <td>10.86.0G00.0048</td>
    </tr>
    <tr>
      <th>4</th>
      <td>01/01/2015</td>
      <td>1030FG000630</td>
      <td>A</td>
      <td>1.420112e+09</td>
      <td>1.420076e+09</td>
      <td>36243</td>
      <td>Galena</td>
      <td>07.86.45.30</td>
      <td>10.86.0G00.0048</td>
    </tr>
  </tbody>
</table>
</div>



Now we group by software version. We also convert this to an array, as we're interested in the number of bad values per software version. (Note, we make some assumptions about these software versions. This sorting *may* be incorrect.)


```python
sw_groups = dirty_data.groupby('sw')
sw_data = np.array(sorted([(int(''.join(t[0].split('.')[:2]) + t[0].split('.')[-1]),
                           t[0], len(t[1]))
                          for t in list(sw_groups)],
                    key=lambda t: t[0]))
numeric_sw_data = np.array(sw_data[:, 2], dtype=int)
```


```python
fw_groups = dirty_data.groupby('fw')
fw_data = np.array([(t[0], len(t[1]))
                    for t in list(fw_groups)
                    if t[0] != 'no storage_array_profile.txt'])
numeric_fw_data = np.array(fw_data[:, 1], dtype=int)
```


```python
plt.figure(figsize=(16, 8))
plt.plot(np.arange(len(numeric_sw_data)), numeric_sw_data,
            label='Software Versions')
plt.plot(np.arange(len(numeric_fw_data)), numeric_fw_data,
            label='Firmware Versions')
plt.legend()
plt.show()
```


![png](NetAppDataAnalysis_files/NetAppDataAnalysis_31_0.png)


Hey! That's perfect! That shows that the number of *bad* rows was really really high for specific versions, but have since been reduced to a lower amount. That's exactly what we wanted to see. As per request, now let's normalize this. In other words, let's examine the percent of bad rows for each release.

## Normalized Releases

First we'll get the software groups.


```python
def sw_to_key(s):
    arr = s.split('.')
    return int(''.join(arr[:2]) + arr[-1])
def fw_to_key(s):
    return int(''.join(s.split('.')))
```


```python
dirty_sw = dirty_data.groupby('sw')
total_sw = data.groupby('sw')
sw_versions = sorted([(t[0], sw_to_key(t[0])) for t in total_sw], key=lambda t: t[1])
normal_sw = np.zeros((len(sw_versions), 3), dtype=np.float128)
for i in range(len(sw_versions)):
    key = sw_versions[i][0]
    normal_sw[i][0] = sw_to_key(key)
    total_group = total_sw.get_group(key)
    normal_sw[i][2] = len(total_group)
    try:
        dirty_group = dirty_sw.get_group(key)
    except KeyError:
        normal_sw[i][1] = 0
    else:
        normal_sw[i][1] = (len(dirty_group) / len(total_group))
```

Now we can get the firmware groups.


```python
dirty_fw = dirty_data.groupby('fw')
total_fw = data.groupby('fw')
fw_versions = sorted([(t[0], fw_to_key(t[0]))
                       for t in total_fw
                       if not t[0].startswith('no storage')],
                     key=lambda t: t[1])
normal_fw = np.zeros((len(fw_versions), 3), dtype=np.float128)
for i in range(len(fw_versions)):
    key = fw_versions[i][0]
    normal_fw[i][0] = fw_to_key(key)
    total_group = total_fw.get_group(key)
    normal_fw[i][1] = len(total_group)
    try:
        dirty_group = dirty_fw.get_group(key)
    except KeyError:
        normal_fw[i][1] = 0
    else:
        normal_fw[i][1] = (len(dirty_group) / len(total_group))
```

And now we can plot.


```python
fig, axarr = plt.subplots(2, 1, figsize=(16, 8))
axarr[0].plot(np.arange(len(normal_sw)), normal_sw[:, 1], 'b-')
axarr[0].scatter(np.arange(len(normal_sw)), normal_sw[:, 1], s=100,
                 c=normal_sw[:, 2], cmap=plt.get_cmap('Blues'))
axarr[0].set_xlim(-1, 90)
axarr[0].set_title('Normalized Software Versions')
axarr[1].plot(np.arange(len(normal_fw)), normal_fw[:, 1], 'g-')
axarr[1].scatter(np.arange(len(normal_fw)), normal_fw[:, 1], s=100,
                 c=normal_fw[:, 2], cmap=plt.get_cmap('Greens'))
axarr[1].set_title('Normalized Firmware Versions')
plt.show()
```


![png](NetAppDataAnalysis_files/NetAppDataAnalysis_38_0.png)


These plots can be interpreted by looking at the $y$ axis. In each graph, the $y$ axis corresponds to the percentage of *bad* rows for the specific software or firmware version. We can see that there appears to be no strong correlation between versions and the number of bad rows.

Each point is also shaded, where the darker the node is the more data exists for that version.


```python

```
