---
title: "Chipotle Item Analysis"
date: 2020-08-07
tags: [python, pandas, restaurants, food, pricing]
excerpt: "Simple, pivotable exploratory walk-thorugh of items, add-ons and prices at Chipotle."
comments: true
mathjax: true

---

This is a short exploratory walk through of an extract of transaction data (with no customer id but just order id) of orders at Chipotle. Main components are about how are different items priced at this Mexican Grill Cuisine and what are the common add-ons that orders include while a food lover customizes the order.


```python
from collections import Counter  # for generating frequency count using dictionary

import matplotlib.pyplot as plt
import pandas as pd

from wordcloud import (  # for generating word cloud
    STOPWORDS,
    ImageColorGenerator,
    WordCloud,
)
```


```python
data = pd.read_csv("chipotle.tsv", sep="\t")
```

### Data wrangling


```python
# replace $ and convert item_price to float
data["item_price"] = pd.Series(data["item_price"].str.replace("$", ""), dtype="float")
```


```python
data[:5]
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
      <th>order_id</th>
      <th>quantity</th>
      <th>item_name</th>
      <th>choice_description</th>
      <th>item_price</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1</td>
      <td>Chips and Fresh Tomato Salsa</td>
      <td>NaN</td>
      <td>2.39</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>1</td>
      <td>Izze</td>
      <td>[Clementine]</td>
      <td>3.39</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>1</td>
      <td>Nantucket Nectar</td>
      <td>[Apple]</td>
      <td>3.39</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>1</td>
      <td>Chips and Tomatillo-Green Chili Salsa</td>
      <td>NaN</td>
      <td>2.39</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>2</td>
      <td>Chicken Bowl</td>
      <td>[Tomatillo-Red Chili Salsa (Hot), [Black Beans...</td>
      <td>16.98</td>
    </tr>
  </tbody>
</table>
</div>



### Summary stats 


```python
print(
    "There are {} observations and {} features in this dataset. \n".format(
        data.shape[0], data.shape[1]
    )
)

print(
    "There are {} different items in this dataset such as {}... \n".format(
        len(data.item_name.unique()), ", ".join(data.item_name.unique()[0:5])
    )
)

print(
    "There are {} unique orders in this dataset. \n".format(
        len(data["order_id"].unique())
    )
)

print(
    "The total revenue generated from the orders within is about ${} USD. \n".format(
        round(sum(data["item_price"]), 0)
    )
)
```

    There are 4622 observations and 5 features in this dataset. 
    
    There are 50 different items in this dataset such as Chips and Fresh Tomato Salsa, Izze, Nantucket Nectar, Chips and Tomatillo-Green Chili Salsa, Chicken Bowl... 
    
    There are 1834 unique orders in this dataset. 
    
    The total revenue generated from the orders within is about $34500.0 USD. 
    


### Avg. Price across all orders


```python
by_order = pd.DataFrame(data.groupby(by="order_id").agg("sum").reset_index())
by_order = by_order.rename(columns={"item_price": "order_amount"})

# avg. order price
print(
    "The avg. order price is $",
    round(sum(by_order["order_amount"]) / len(by_order["order_amount"]), 2),
)
```

    The avg. order price is $ 18.81


### Avg. price paid by order


```python
by_order = pd.DataFrame(
    data.groupby(by="order_id")
    .agg({"item_price": "mean", "item_name": "count"})
    .reset_index()
)
by_order = by_order.rename(
    columns={"item_price": "avg_order_amount", "item_name": "num_items"}
)

# Distribution of avg. order amount across all orders
plt.hist(by_order["avg_order_amount"])
plt.xlabel("Avg. Order Amount in USD")
plt.ylabel("Frequency count")
plt.show()
```


![png](analysis_files/analysis_12_0.png)


### Distribution of items per order


```python
plt.hist(by_order["num_items"], bins=25)
plt.xlabel("Items per order")
plt.ylabel("Frequency count")
plt.show()
```


![png](analysis_files/analysis_14_0.png)



```python
by_item_numbers = (
    by_order.groupby(by="num_items").agg({"order_id": "count"}).reset_index()
)
by_item_numbers = by_item_numbers.rename(columns={"order_id": "num_orders"})
by_item_numbers
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
      <th>num_items</th>
      <th>num_orders</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>128</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>1012</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>484</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>134</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>44</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>14</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>4</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>5</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>2</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>1</td>
    </tr>
    <tr>
      <th>10</th>
      <td>11</td>
      <td>3</td>
    </tr>
    <tr>
      <th>11</th>
      <td>12</td>
      <td>1</td>
    </tr>
    <tr>
      <th>12</th>
      <td>14</td>
      <td>1</td>
    </tr>
    <tr>
      <th>13</th>
      <td>23</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



Close to about 55% of the orders are 2-item orders analogous to people ordering a main item with either a side or a beverage. Appently, the next in number of orders is those with 3 items (main item + side + beverage).

### Avg. prices per item ordered

As the each item ordered in any order is customized based on choice description, let's see what are the avg. prices per item ordered.


```python
# avg. price per item ordered
by_item = pd.DataFrame(
    data.groupby(by="item_name")
    .agg({"item_price": "mean", "order_id": "count"})
    .reset_index()
)
by_item = by_item.rename(
    columns={"item_price": "avg_price_paid", "order_id": "times_ordered"}
)

by_item["revenue"] = by_item["avg_price_paid"] * by_item["times_ordered"]
```


```python
plt.figure(figsize=(10, 10))
by_item = by_item.sort_values(by="revenue", ascending=True)
plt.barh(by_item["item_name"], by_item["revenue"])
plt.xlabel("Revenue (USD)")
plt.ylabel("Food Item")
plt.title("Items ranked based on revenue generated in USD")
plt.show()
```


![png](analysis_files/analysis_20_0.png)


#### By most ordered


```python
# Top 5items by 'times_ordered'
by_item.sort_values(by=["times_ordered"], ascending=False)[:10]
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
      <th>item_name</th>
      <th>avg_price_paid</th>
      <th>times_ordered</th>
      <th>revenue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>17</th>
      <td>Chicken Bowl</td>
      <td>10.113953</td>
      <td>726</td>
      <td>7342.73</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Chicken Burrito</td>
      <td>10.082857</td>
      <td>553</td>
      <td>5575.82</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Chips and Guacamole</td>
      <td>4.595073</td>
      <td>479</td>
      <td>2201.04</td>
    </tr>
    <tr>
      <th>39</th>
      <td>Steak Burrito</td>
      <td>10.465842</td>
      <td>368</td>
      <td>3851.43</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Canned Soft Drink</td>
      <td>1.457641</td>
      <td>301</td>
      <td>438.75</td>
    </tr>
    <tr>
      <th>38</th>
      <td>Steak Bowl</td>
      <td>10.711801</td>
      <td>211</td>
      <td>2260.19</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Chips</td>
      <td>2.342844</td>
      <td>211</td>
      <td>494.34</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Bottled Water</td>
      <td>1.867654</td>
      <td>162</td>
      <td>302.56</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Chicken Soft Tacos</td>
      <td>9.635565</td>
      <td>115</td>
      <td>1108.09</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Chicken Salad Bowl</td>
      <td>11.170455</td>
      <td>110</td>
      <td>1228.75</td>
    </tr>
  </tbody>
</table>
</div>



* Canned Soft Drink is the most ordered beverage.

#### By revenue


```python
by_item.sort_values(by=["revenue"], ascending=False)[:10]
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
      <th>item_name</th>
      <th>avg_price_paid</th>
      <th>times_ordered</th>
      <th>revenue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>17</th>
      <td>Chicken Bowl</td>
      <td>10.113953</td>
      <td>726</td>
      <td>7342.73</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Chicken Burrito</td>
      <td>10.082857</td>
      <td>553</td>
      <td>5575.82</td>
    </tr>
    <tr>
      <th>39</th>
      <td>Steak Burrito</td>
      <td>10.465842</td>
      <td>368</td>
      <td>3851.43</td>
    </tr>
    <tr>
      <th>38</th>
      <td>Steak Bowl</td>
      <td>10.711801</td>
      <td>211</td>
      <td>2260.19</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Chips and Guacamole</td>
      <td>4.595073</td>
      <td>479</td>
      <td>2201.04</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Chicken Salad Bowl</td>
      <td>11.170455</td>
      <td>110</td>
      <td>1228.75</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Chicken Soft Tacos</td>
      <td>9.635565</td>
      <td>115</td>
      <td>1108.09</td>
    </tr>
    <tr>
      <th>45</th>
      <td>Veggie Burrito</td>
      <td>9.839684</td>
      <td>95</td>
      <td>934.77</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Barbacoa Burrito</td>
      <td>9.832418</td>
      <td>91</td>
      <td>894.75</td>
    </tr>
    <tr>
      <th>44</th>
      <td>Veggie Bowl</td>
      <td>10.211647</td>
      <td>85</td>
      <td>867.99</td>
    </tr>
  </tbody>
</table>
</div>



* Chicken Bowl, Chicken Burrito, Steak Burrito, Steak Bowl get us the most revenue.
* Chips and Guacamole is the most ordered as well as revenue generating sides

### List of all the items where customization was possible


```python
df_without_na = pd.DataFrame(data.dropna())

# Items for choice_description is possible
pd.Series(df_without_na["item_name"].unique())
```




    0                      Izze
    1          Nantucket Nectar
    2              Chicken Bowl
    3             Steak Burrito
    4          Steak Soft Tacos
    5      Chicken Crispy Tacos
    6        Chicken Soft Tacos
    7           Chicken Burrito
    8               Canned Soda
    9          Barbacoa Burrito
    10         Carnitas Burrito
    11            Carnitas Bowl
    12            Barbacoa Bowl
    13       Chicken Salad Bowl
    14               Steak Bowl
    15      Barbacoa Soft Tacos
    16           Veggie Burrito
    17              Veggie Bowl
    18       Steak Crispy Tacos
    19    Barbacoa Crispy Tacos
    20        Veggie Salad Bowl
    21      Carnitas Soft Tacos
    22            Chicken Salad
    23        Canned Soft Drink
    24         Steak Salad Bowl
    25        6 Pack Soft Drink
    26                     Bowl
    27                  Burrito
    28             Crispy Tacos
    29    Carnitas Crispy Tacos
    30              Steak Salad
    31        Veggie Soft Tacos
    32      Carnitas Salad Bowl
    33      Barbacoa Salad Bowl
    34                    Salad
    35      Veggie Crispy Tacos
    36             Veggie Salad
    37           Carnitas Salad
    dtype: object




```python
# Calling list_to_string function
%run list_to_string.py
```


```python
# data transformation to string without any [] in the choice_description column
df_without_na["choice_description"] = df_without_na["choice_description"].apply(
    list_to_string
)
```


```python
text = ", ".join(cd for cd in df_without_na.choice_description)
print("There are {} choice additions in the choice descriptions.".format(len(text)))
my_list = text.split(", ")
word_count_dict = Counter(my_list)
```

    There are 210889 choice additions in the choice descriptions.


### What goes in the add-ons/choice description mostly?


```python
wordcloud = WordCloud(
    stopwords=stopwords, background_color="white", width=1600, height=1200, scale=1.5
).generate_from_frequencies(word_count_dict)

# Display the generated image
plt.figure(figsize=(10, 15))
plt.imshow(wordcloud, interpolation="bilinear")
plt.axis("off")
plt.show()
```


![png](analysis_files/analysis_33_0.png)


Just by looking at the wordcloud, a couple of insights are visible:
* In all the orders pertaining to this dataset, a bowl or a burrito has Rice, Lettuce, Cheese, Sour cream as add-ons. While this maybe trivial to some of you, expecialy Mexican descent, for someone who don't know anything about the Mexican cuisine, this is valuable information to form hypothesis. This applies to any data set one works on. In some cases, we as data analysts/scientists are pre-informed but in some cases, data itself tells us the new information we do not know till that point.
* Among Salsa's, Fresh Tomato Salsa looks to be the top added add-on and then Roasted Chili Corn Salsa
* Asbeans are common in Mexican food, both Black and Pinto beans are visible. But as a matter of fact, Black beans has lower carbs than Pinto beans  which could be a reason as all the food lovers like us in this data are preferring `Black Beans`.

### What are the items which doesn't have any choice_description (no customization)?


```python
set(data["item_name"].unique()) - set(df_without_na["item_name"].unique())
```




    {'Bottled Water',
     'Chips',
     'Chips and Fresh Tomato Salsa',
     'Chips and Guacamole',
     'Chips and Mild Fresh Tomato Salsa',
     'Chips and Roasted Chili Corn Salsa',
     'Chips and Roasted Chili-Corn Salsa',
     'Chips and Tomatillo Green Chili Salsa',
     'Chips and Tomatillo Red Chili Salsa',
     'Chips and Tomatillo-Green Chili Salsa',
     'Chips and Tomatillo-Red Chili Salsa',
     'Side of Chips'}



A good check that the sides are mostly not customizable - data sanity check. It's good to have these sanity checks once in a while all through the data analysis and modelling to ensure reliable results. But that's not the end as there can be variation depending on the region, season, etc. That's where as a data analyst/scientist, one has to use communication witha team member to clarify whether all the listed elements are not customizable. Ultimately, data tells a story partially, to make it whole, business knowledge (as simple as the above example) and team play is necessary.

This brings to the end of this simple walk-through and my plan from here on is to extend this analysis for a market basket analysis or to recommend what can a customer add as an add-on in the next order. Off to pondering whether this data is sufficient to do what struck me. If you have any ideas or loves different food cuisines, ping me - we can discuss about food or plan to implement the ideas...
