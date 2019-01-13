---
title: 'Peterman’s Ineptitude: Web Scraping with Pandas Read_HTML'
date: 2019-01-13
permalink: /posts/2019/1/web-scraping-pandas-nathan-peterman/
tags:
  - pandas
  - NFL
  - Nathan Peterman
  - QBR
---

On December 19th, the Oakland Raiders shocked the football community by signing Nathan Peterman to their practice squad after a terrible year with the Buffalo Bills.  Advanced stats aren’t really needed to describe how poorly Peterman played. A 54.3% completion percentage, a 30.7 quarterback rating, and a 1/7 TD/INT ratio did just fine demonstrating how bad he was.

I was curious nonetheless to see how Peterman’s 2018 QBR of 6.5 ranked since 2006.  [Pro Football Reference](https://www.pro-football-reference.com/players/P/PeteNa00.htm) has the stat by year for Peterman but no easy way to sort historical QBR in ascending order.  [ESPN’s website]( http://www.espn.com/nfl/qbr) does allow for sorting but the limit for qualifying is too high for Peterman’s 106 Total QB Plays.  

My goal was to combine the qualified and unqualified leaderboards in order to determine who had the worst QBR with at least 100 Total QB Plays. It’s an arbitrary cutoff, but it’s a nice round number and it will get players who played close to as much as Peterman without including RB and WR.

In order to complete the analysis, we’ll need to a) combine the qualified and unqualified leaderboards, b) filter for at least 100 plays, and c) sort by Total QBR in ascending order.

To do so, we’ll need to grab every page for both the qualified and unqualified leaderboards.  There are 8 pages on the qualified leaderboard and 29 pages on the unqualified leaderboards.  We will set up two for loops to loop through 1 through 8, and 1 through 29.  Because python uses [0-based indexing](http://python-history.blogspot.com/2013/10/why-python-uses-0-based-indexing.html), we’ll use the python range function to create an iterator that starts at 1 and ends at 8.

For each n in our iterator we will use the pandas read_html function to return all HTML formatted tables on each page.  As there is only one, we will grab the first element [0] of that list and append to our dataframes (all_unq_qbr: unqualified, all_qual_qbr: qualified).

```python
import pandas as pd

all_unq_qbr = pd.DataFrame()

for n in range(1,30):
    df = pd.read_html(f'http://www.espn.com/nfl/qbr/_/type/alltime-season/page/{n}/order/true/qualified/false')[0]
    all_unq_qbr = all_unq_qbr.append(df)
```

```python
all_qual_qbr = pd.DataFrame()

for n in range(1,9):
    df = pd.read_html(f'http://www.espn.com/nfl/qbr/_/type/alltime-season/page/{n}/order/true/qualified/true')[0]
    all_qual_qbr = all_qual_qbr.append(df)
```

Next we’ll concatenate the two dataframes together and rename the columns.  

```python
all_qbr = pd.concat([all_unq_qbr, all_qual_qbr]).reset_index(drop=True)
```

```python
all_qbr.columns = ['RK',
  'PLAYER',
  'YEAR',
  'PTS ADDED',
  'PASS',
  'RUN',
  'PENALTY',
  'TOTAL EPA',
  'QB_PLAYS',
  'RAW QBR',
  'TOTAL QBR']
```


We also need to remove the sub headers that are included in the tables by filtering to remove rows that include “YEAR” under the “YEAR” column. 

```python
all_qbr = all_qbr.query('YEAR!="YEAR"').reset_index(drop=True)
```

Because of the sub headers, all columns are formatted as strings.  In order to fix this, we can use the astype method to reformat QB_PLAYS as an integer and TOTAL_QBR as a float.

```python
all_qbr['QB_PLAYS'] = all_qbr['QB_PLAYS'].astype(float)
all_qbr['TOTAL QBR'] = all_qbr['TOTAL QBR'].astype(float)


```


Now we can answer the question!

By [using method chaining]( https://tomaugspurger.github.io/method-chaining) we will

1. Filter QB_PLAYS to only include players with greater than or equal to 100 QB Plays
2. Drop players that are duplicated
3. Sort values by Total QBR in ascending order
4. Select the columns Player, Year, QB_Plays, and Total_QBR
5. Reset the index
6. Show the top 10 

```python
(all_qbr
.query("QB_PLAYS>=100")
.drop_duplicates(subset=['PLAYER','YEAR'])
[['PLAYER','YEAR','QB_PLAYS','TOTAL QBR']]
.sort_values('TOTAL QBR', ascending=True)
.reset_index(drop=True).head(10))
```

Per ESPN's Total QBR, Nathan Peterman had the second-worst QB season since 2006 for all players with at least 100 QB Plays. Only Blaine Gabbert's 2013 season ranks worse. 


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
      <th>PLAYER</th>
      <th>YEAR</th>
      <th>QB_PLAYS</th>
      <th>TOTAL_QBR</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Blaine Gabbert, JAX</td>
      <td>2013</td>
      <td>110.0</td>
      <td>4.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Nathan Peterman, BUF/OAK</td>
      <td>2018</td>
      <td>106.0</td>
      <td>8.7</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Ryan Lindley, ARI</td>
      <td>2012</td>
      <td>196.0</td>
      <td>10.2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Tarvaris Jackson, MIN</td>
      <td>2006</td>
      <td>110.0</td>
      <td>11.5</td>
    </tr>
    <tr>
      <th>4</th>
      <td>John Beck, MIA</td>
      <td>2007</td>
      <td>128.0</td>
      <td>13.3</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Jimmy Clausen, CAR</td>
      <td>2010</td>
      <td>378.0</td>
      <td>13.8</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Bryce Petty, NYJ</td>
      <td>2016</td>
      <td>161.0</td>
      <td>15.9</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Kerry Collins, IND</td>
      <td>2011</td>
      <td>109.0</td>
      <td>16.8</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Keith Null, LAR</td>
      <td>2009</td>
      <td>148.0</td>
      <td>16.9</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Alex Smith, SF</td>
      <td>2007</td>
      <td>234.0</td>
      <td>18.2</td>
    </tr>
  </tbody>
</table>
</div>

If we relax the QB plays filter to 75 plays, Peterman ranks third since 2006, with Byron Leftwich's 2007 season with Atlanta at 6.6 slightly worse.

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
      <th>PLAYER</th>
      <th>YEAR</th>
      <th>QB_PLAYS</th>
      <th>TOTAL_QBR</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Blaine Gabbert, JAX</td>
      <td>2013</td>
      <td>110</td>
      <td>4.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Byron Leftwich, ATL</td>
      <td>2007</td>
      <td>75</td>
      <td>6.6</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Nathan Peterman, BUF/OAK</td>
      <td>2018</td>
      <td>106</td>
      <td>8.7</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Max Hall, ARI</td>
      <td>2010</td>
      <td>96</td>
      <td>8.8</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Ryan Lindley, ARI</td>
      <td>2012</td>
      <td>196</td>
      <td>10.2</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Tarvaris Jackson, MIN</td>
      <td>2006</td>
      <td>110</td>
      <td>11.5</td>
    </tr>
    <tr>
      <th>6</th>
      <td>John Beck, MIA</td>
      <td>2007</td>
      <td>128</td>
      <td>13.3</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Jimmy Clausen, CAR</td>
      <td>2010</td>
      <td>378</td>
      <td>13.8</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Bryce Petty, NYJ</td>
      <td>2016</td>
      <td>161</td>
      <td>15.9</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Kerry Collins, IND</td>
      <td>2011</td>
      <td>109</td>
      <td>16.8</td>
    </tr>
  </tbody>
</table>
</div>
