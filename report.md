# Data Analytics
## Customer Segmentation using RFM Analysis

- **Team**: 3
- **Lecture**: Chap Chanpiseth
- **Members**: Sim Lyheng, Tan Chesthareah

---
### Explore

#### Library Used
```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import squarify
```

#### Understand the Data
```python
df = pd.read_csv('rfm_data_orders.rda (2).csv')
df.head()
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
      <th>Unnamed: 0</th>
      <th>customer_id</th>
      <th>order_date</th>
      <th>revenue</th>
      <th>first_name</th>
      <th>last_name</th>
      <th>email</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>Brion Stark</td>
      <td>2004-12-20</td>
      <td>32</td>
      <td>Brion</td>
      <td>Stark</td>
      <td>brion_stark@rfmail.com</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>Ethyl Botsford</td>
      <td>2005-05-02</td>
      <td>36</td>
      <td>Ethyl</td>
      <td>Botsford</td>
      <td>ethyl_botsford@rfmail.com</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>Hosteen Jacobi</td>
      <td>2004-03-06</td>
      <td>116</td>
      <td>Hosteen</td>
      <td>Jacobi</td>
      <td>hosteen_jacobi@rfmail.com</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>Edw Frami</td>
      <td>2006-03-15</td>
      <td>99</td>
      <td>Edw</td>
      <td>Frami</td>
      <td>edw_frami@rfmail.com</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>Josef Lemke</td>
      <td>2006-08-14</td>
      <td>76</td>
      <td>Josef</td>
      <td>Lemke</td>
      <td>josef_lemke@rfmail.com</td>
    </tr>
  </tbody>
</table>
</div>

```
RangeIndex: 4906 entries, 0 to 4905
Data columns (total 7 columns):
     Column       Non-Null Count  Dtype 
---  ------       --------------  ----- 
 0   Unnamed: 0   4906 non-null   int64 
 1   customer_id  4906 non-null   object
 2   order_date   4906 non-null   object
 3   revenue      4906 non-null   int64 
 4   first_name   4906 non-null   object
 5   last_name    4906 non-null   object
 6   email        4906 non-null   object
dtypes: int64(2), object(5)
```

#### Rename header 
Replace ```Unnamed:0``` to ```cusomter_id``` and ```customer_id``` to ```customer_name```

```python
# Rename the columns
df = df.rename(columns={'Unnamed: 0': 'customer-id', 'customer_id' : 'customer_name'})
```
#### Check for duplicate 
```python
 # Check duplicates for email
email_duplicates = df['email'].duplicated().sum()
print(f"Duplicate emails: {email_duplicates}")

# Check duplicates for customer_name
name_duplicates = df['customer_name'].duplicated().sum()
print(f"Duplicate names: {name_duplicates}")
```

```
[Output]:
Duplicate emails: 3911
Duplicate names: 3911
```
Emails and names are one to one. So there is no mistake in names and emails.

#### Drop unnecessary columns
```python
df = df.drop(columns=['first_name', 'last_name','email'])
```
#### Check for missing value
```python
check_missing = df.isnull().sum()
check_missing
```

```
[Output]:
customer-id      0
customer_name    0
order_date       0
revenue          0
dtype: int64
```

#### Convert the date to Date/DateTime format
```python
df['order_date'] = pd.to_datetime(df['order_date'])
```
#### Convert the revenue to numeric
Since Revenue already numeric (int64). 

#### Sort the Value
```python
df = df.sort_values(by=['customer_name', 'order_date'])
df
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
      <th>customer-id</th>
      <th>customer_name</th>
      <th>order_date</th>
      <th>revenue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3760</th>
      <td>3760</td>
      <td>Abbey O'Reilly</td>
      <td>2005-07-16</td>
      <td>47</td>
    </tr>
    <tr>
      <th>4418</th>
      <td>4418</td>
      <td>Abbey O'Reilly</td>
      <td>2005-09-04</td>
      <td>152</td>
    </tr>
    <tr>
      <th>4180</th>
      <td>4180</td>
      <td>Abbey O'Reilly</td>
      <td>2005-11-03</td>
      <td>41</td>
    </tr>
    <tr>
      <th>313</th>
      <td>313</td>
      <td>Abbey O'Reilly</td>
      <td>2005-12-24</td>
      <td>145</td>
    </tr>
    <tr>
      <th>1691</th>
      <td>1691</td>
      <td>Abbey O'Reilly</td>
      <td>2006-01-06</td>
      <td>67</td>
    </tr>
  </tbody>
</table>
</div>

### RFM Compute
```python
analysis_date = df["order_date"].max()

rfm = df.groupby("customer_name").agg(
    recency=('order_date', lambda x: (analysis_date - x.max()).days),  # Days since last purchase
    frequency=('order_date', 'count'),  # Unique order dates
    monetary=('revenue', 'sum')  # Total spent
).reset_index()

rfm
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
      <th>customer_name</th>
      <th>recency</th>
      <th>frequency</th>
      <th>monetary</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Abbey O'Reilly</td>
      <td>204</td>
      <td>6</td>
      <td>472</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Add Senger</td>
      <td>139</td>
      <td>3</td>
      <td>340</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Aden Lesch</td>
      <td>193</td>
      <td>4</td>
      <td>405</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Aden Murphy</td>
      <td>97</td>
      <td>7</td>
      <td>596</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Admiral Senger</td>
      <td>131</td>
      <td>5</td>
      <td>448</td>
    </tr>
  </tbody>
</table>
</div>


### Assign Monetary, Frequency and Recency

#### Assign Monetary Score
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
      <th>customer_name</th>
      <th>monetary</th>
      <th>Interval</th>
      <th>monetary_score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Abbey O'Reilly</td>
      <td>472</td>
      <td>&gt; 505.4 &amp; &lt;= 665.0</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Add Senger</td>
      <td>340</td>
      <td>&gt; 505.4 &amp; &lt;= 665.0</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Aden Lesch</td>
      <td>405</td>
      <td>&gt; 505.4 &amp; &lt;= 665.0</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Aden Murphy</td>
      <td>596</td>
      <td>&gt; 665.0</td>
      <td>4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Admiral Senger</td>
      <td>448</td>
      <td>&gt; 505.4 &amp; &lt;= 665.0</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>

#### Assign Frequency Score
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
      <th>customer_name</th>
      <th>frequency</th>
      <th>Interval</th>
      <th>frequency_score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Abbey O'Reilly</td>
      <td>6</td>
      <td>&lt;= 3</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Add Senger</td>
      <td>3</td>
      <td>&lt;= 3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Aden Lesch</td>
      <td>4</td>
      <td>&lt;= 3</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Aden Murphy</td>
      <td>7</td>
      <td>&lt;= 3</td>
      <td>4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Admiral Senger</td>
      <td>5</td>
      <td>&lt;= 3</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>

#### Assign Recency Score

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
      <th>customer_name</th>
      <th>recency</th>
      <th>Interval</th>
      <th>recency_score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Abbey O'Reilly</td>
      <td>204</td>
      <td>&gt; 179.0 &amp; &lt;= 295.4</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Add Senger</td>
      <td>139</td>
      <td>&gt; 113.0 &amp; &lt;= 179.0</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Aden Lesch</td>
      <td>193</td>
      <td>&gt; 179.0 &amp; &lt;= 295.4</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Aden Murphy</td>
      <td>97</td>
      <td>&lt;= 113.0</td>
      <td>5</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Admiral Senger</td>
      <td>131</td>
      <td>&gt; 113.0 &amp; &lt;= 179.0</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>

#### Marge the RFM together with computing

Formula:

$$
RFM Score = Recency Score * 100 + Frequency Score * 10 + Monetary Score
$$

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
      <th>Customer</th>
      <th>Recency (Days)</th>
      <th>R</th>
      <th>Frequency</th>
      <th>F</th>
      <th>Monetary ($)</th>
      <th>M</th>
      <th>RFM Score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Abbey O'Reilly</td>
      <td>204</td>
      <td>3</td>
      <td>6</td>
      <td>4</td>
      <td>472</td>
      <td>3</td>
      <td>343</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Add Senger</td>
      <td>139</td>
      <td>4</td>
      <td>3</td>
      <td>1</td>
      <td>340</td>
      <td>2</td>
      <td>412</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Aden Lesch</td>
      <td>193</td>
      <td>3</td>
      <td>4</td>
      <td>2</td>
      <td>405</td>
      <td>3</td>
      <td>323</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Aden Murphy</td>
      <td>97</td>
      <td>5</td>
      <td>7</td>
      <td>4</td>
      <td>596</td>
      <td>4</td>
      <td>544</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Admiral Senger</td>
      <td>131</td>
      <td>4</td>
      <td>5</td>
      <td>3</td>
      <td>448</td>
      <td>3</td>
      <td>433</td>
    </tr>
  </tbody>
</table>
</div>

### Segments
### Customer Segmentation Criteria

| Segment           | Description                                             | Recency Score | Frequency Score | Monetary Score |
|-------------------|---------------------------------------------------------|---------------|-----------------|----------------|
| Champions         | Bought recently, buy often and spend the most.          | 4–5           | 4–5             | 4–5            |
| Loyal Customers   | Spend good money. Responsive to promotions.             | 2–4           | 3–4             | 4–5            |
| Potential Loyalists | Recent customers, spent good amount, bought more than once | 3–5         | 1–3             | 1–3            |
| New Customers     | Bought more recently but not often                      | 4–5           | < 2              | < 2             |
| Promising         | Recent shoppers but haven’t spent much                  | 3–4           | < 2              | < 2             |
| Need Attention    | Above average recency, frequency & monetary values      | 3–4           | 3–4             | 3–4            |
| About to Sleep    | Below average recency, frequency & monetary values      | < 3            | < 3              | < 3             |
| At Risk           | Spent big money, purchased often but long time ago      | < 3            | 2–5             | 2–5            |
| Can’t Lose Them   | Made big purchases and often but long time ago          | < 2            | 4–5             | 4–5            |
| Hibernating       | Low spenders, low frequency and purchased long time ago | 2–3           | 2–3             | 2–3            |
| Lost              | Lowest recency, frequency & monetary values             | < 2            | < 2              | < 2             |

#### Segmentation

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
      <th>Customer</th>
      <th>Recency (Days)</th>
      <th>R</th>
      <th>Frequency</th>
      <th>F</th>
      <th>Monetary ($)</th>
      <th>M</th>
      <th>RFM Score</th>
      <th>Segment</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Abbey O'Reilly</td>
      <td>204</td>
      <td>3</td>
      <td>6</td>
      <td>4</td>
      <td>472</td>
      <td>3</td>
      <td>343</td>
      <td>Need Attention</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Add Senger</td>
      <td>139</td>
      <td>4</td>
      <td>3</td>
      <td>1</td>
      <td>340</td>
      <td>2</td>
      <td>412</td>
      <td>Potential Loyalist</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Aden Lesch</td>
      <td>193</td>
      <td>3</td>
      <td>4</td>
      <td>2</td>
      <td>405</td>
      <td>3</td>
      <td>323</td>
      <td>Potential Loyalist</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Aden Murphy</td>
      <td>97</td>
      <td>5</td>
      <td>7</td>
      <td>4</td>
      <td>596</td>
      <td>4</td>
      <td>544</td>
      <td>Champion</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Admiral Senger</td>
      <td>131</td>
      <td>4</td>
      <td>5</td>
      <td>3</td>
      <td>448</td>
      <td>3</td>
      <td>433</td>
      <td>Potential Loyalist</td>
    </tr>
  </tbody>
</table>
</div>

```python
# segment_counts = marge_rfm['Segment'].value_counts().reset_index()
segment_counts = marge_rfm['Segment'].value_counts()
 
# Rename columns for clarity
segment_counts.columns = ['Segment', 'Customer Count']

segment_counts
```

```
[Output]:
Segment
Potential Loyalist    277
Champion              159
At Risk               148
Others                128
Loyal Customer        126
Lost                   75
About to Sleep         65
Need Attention         17
Name: count, dtype: int64
```

### Visualitaiton
This Treemap is showing about the cutomer segmentation based on the RFM. Color given based on the rank of the segmentation.

![alt text]([image-11.png](https://github.com/salarymakage/DA_RFM/blob/main/image/image-11.png?raw=true))

#### Median Recency
This graph tells you how many days ago, on average, customers in each group like buying.
 
![alt text](image-12.png)

#### Median Frequency
This graph shows how many times, on average, customers in each group like buying.

![alt text](image-13.png)

#### Median Monetary Value
This graph shows the average amount of money, in dollars, that customers in each group spend.

![alt text](image-14.png)

#### Heat Map

![alt text](image-4.png)

#### Bar Chart
This chart helps you see how much money customers spend based on how recently and how often they interact. It shows that most customers, no matter their recency or frequency, tend to spend less (Monetary Score 1), but some groups (like R=1, F=5) have a few who spend more.

![alt text](image-5.png)

#### Histogram
These graphs give a quick look at how recent, frequent, and valuable customers are. Most are active recently (100-200 days), interact a few times (4-6), and spend a little (200-400 dollars).
![alt text](image-6.png)

#### Customers by Orders
This graph show about the number customer orders 
![alt text](image-15.png)

#### Recency vs Monetary Value

![alt text](image-8.png)

#### Frequency vs Monetary Value

![alt text](image-17.png)

#### Recency vs Frequency

![alt text](image-16.png)
