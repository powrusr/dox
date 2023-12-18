# pandas

## get df info
```python
df.head(10)
df.tail(6)
df.info()  # dtype columns non-null count
df.shape() # dimensions
df.sample(3) # random rows
df.corr()  # correlations between columns
df.describe() # statistics (of numerical columns)
df.describe(include="all") # to also include categorical columns
df.columns
df.dtypes
df.max()  # get max for every column
df["Year"].value_counts()
```

df.sort_values(by="last_updated",ascending=True)

market cap = price * circulating supply

df.current_price.mul(df.circulating_supply).sub(df.market_cap)
df.market_cap_rank
df.market_cap.value_counts()

zero_cap = df.loc[df.market_cap==0, ["current_price", "circulating_supply"]]
(zero_cap==0).sum() # true false, then sum



## calculations
```python
# mul(tiply) has fill_value for missing data
df["market_share"] = df.market_cap.div(total_market).mul(100)
# random integers
np.random.randint(1,101,10)
values = range(0, 500, 5)
nums = pd.Series(data=values)
# sum with don't skip na values
numbers.sum(skipna=False)
# must hold a min of 3 values present in series
numbers.sum(min_count=3)
# rolling sum of values 
numbers.cumsum()
df.market_share.cumsum().head(50)
# get percentage change from last value in series (ffill by default for missing values)
# fill_method ="bfill" replace with next value observed
numbers.pct_change()
# mean = avg value in the series
numbers.mean()
# median, middle number in sorted series
numbers.median()
# measure variation in the data
numbers.std()
# describe gives count,mean,std,min,25,50,75,max values
numbers.describe()
# give only te unique values in series or nunique
numbers.unique()
# arithmitic
s1 // 4
s1.floordiv(4)
s1 == s2 !=
s1.eq(s2) ne()
in & not in
```
```python
# tips as % of total bill
tips["tip_pct"] = tips["tip"] / tips["total_bill"]
tips["tip_pct"] = tips["tip_pct"].round(2) # 2 decimals
```


## plotting

### pyplot
```python
import matplotlib.pyplot as plt

df.iloc[0:100].market_share.cumsum().plot(figsize=(12,8))
plt.title("crypto market - Cum Market Share", fontsize=15)
plt.xlabel("No of Coins (Market Cap Rank)", fontsize=12)
plt.ylabel("Bum Market Share (%)", fontsize=12)
plt.show()
```

### seaborn

```python
import seaborn as sns
mpg = sns.load_dataset('mpg')
mpg['mpg'].plot()  # linechart
mpg['mpg'].plot(kind="hist")  # histogram
mpg['mpg'].plot(kind="box")  # box plot
mpg['mpg'].value_counts().plot(kind="barh")
mpg['mpg'].value_counts().plot(kind="pie")
mpg['mpg'].plot(x="weight", y="mpg", kind="scatter")
```

## renaming and dropping columns
### rename column
```python
nba.columns = ["Team", "Position", "DoB", "Pay"]
nba.rename(columns = { "DoB": "Birthday"})
nba = nba.rename(columns = { "DoB": "Birthday"})  # to make permanent
```
### drop column
```python
tips.drop("tip_pct", axis="columns", inplace=True)
tips.columns
```
### rename index
```python
# set another column as index of our df
nba.set_index("Team").head()
# reset_index moves current index to new column
nba.reset_index().head()
nba.reset_index().set_index("Team").head()
# rename index column
df.rename(columns={'Unnamed: 0': 'index'}, inplace=True)
```

## sorting

### sort_values
```python
# Sort by total bill
tips.sort_values(by=['total_bill'])

# Total bill, descending
tips.sort_values(by=['total_bill'], ascending=False)
tips.sort_values(by=['total_bill'], ascending=False, inplace=True)

# Time ascending, total bill descending
tips.sort_values(by=['time', 'total_bill'], ascending=[True, False])

# put nan values first when sorting
battles.sort_values(na_position="first")
# sort index
pokemon.sort_index(ascending=True)
stocks.nlargest(5)
stocks.nsmallest(5)

# sorting by multiple columns
# sort by Team first & then by Name column
nba.sort_values(by=["Team", "Name"])

# always sort first = faster lookup speed
# w slicing
nba.sort_index().loc["Otto Porter":"Patrick Beverly"]
```

```python
def highest_lowest(n, by, ascending=False):
	df2 = df.sort_values(by=by,ascending=ascending).head(n).copy()
	return HTML(df.to_html(escape=False))

highest_lowest(20, "market_cap", ascending=False) # 20 highest market_cap showing img's
highest_lowest(20, "market_share") # 20 highest market_share
highest_lowest(20, "current_price", ascending=False) # 20 highest price
highest_lowest(20, "circulating_supply", ascending=False) # 20 highest supply
```

### rank

Rank - what to do when you have equal values in a column - aka sorting with collision detection

```python
df_price = df.sort_values("price", ascending=False)
df_price[["id", "host_name", "price"]].head(5)
   
df_price["price_rank"] = df_price.price.rank(method="average", ascending=False)  # rank based on avg
df_price[["id", "host_name", "price", "price_rank"]].head(5)
```


## filtering

```python
# ~ helpful to just get the opposite
below_100k = employees["Salary"] < 100000
under_100k_earners = employees[below_100k]
100k_earners = employees[~below_100k]
# == eg
# != ne
# < lt
# <= le
# > gt
# >= ge

# isin (no need to create a ton of helpvars)
all_stars_team = ["Sales", "Legal", "Marketing"]
on_all_star_team = employees["Team"].isin(all_stars_team)

# between (works for dates & strings too)
between_80k_and_90k = employees["Salary"].between(80000, 90000) # 90k excluded
employees[between_80k_and_90k].head()

# isnull & notnull
has_name = employees["First Name"].notnull()
employees[has_name].tail()
```

```python
and_operator = df[(df['Sales'] > 300) & (df['Units'] > 20)]
or_operator = df[(df['Sales'] > 300) | (df['Units'] > 20)]
```

using regex

```python
th = df[df['Region'].str.contains('th$')]
```

```python
# Filtering tip amount greater than 5
tips_over_5 = tips['tip'] > 5

tips[tips_over_5]

# Filtering tip > 5 AND time = 'Dinner'
dinner_tips_over_5 = (tips['tip'] > 5) & (tips['time'] == 'Dinner')
tips[dinner_tips_over_5]

tips.query('tip > 5')
tips.query('tip > 5 and time == "Dinner"')

some_limit = 85
df.query('column > @some_limit')
df.query('column == 3')
df.query('column == 3 & survived == 1 & age > 40')
df.query('column == 3 & survived == 1 & (age > 40 | age < 10)')


df.filter(["Name", "Age"]
new_df = df.filter(["Age", "Name"]

df.filter(["Name"]).Name.str.contains("Miss.")  # returs bool series
df.filter(["Name"]).Name.str.contains("Miss.").value_counts()  # easy count
df[df.filter(["Age", "Name"]).Name.str.contains("Mister") == True]

df[df.Name.str.contains("Mister") == True].filter(["Age", "Name"])

# use copy!
df_fast = df.query('Time < 10').copy()
```

### mask helpers

```python
# between
   print(df.loc[df.price.between(100, 200), "price"].head())
# isin
df.loc[df.price.isin([100, 200]), "price"].head()  # is price 100 or 200

# you can apply mask on entire df not just column
df == "John"
(df == "John").any())  # any column that contains "john"

# now get rows! by changing default axis
(df == "John").any(axis=1)  # any column that contains "john"
```

## groupby

```python
tips_grouped = tips.groupby('day')
tips_grouped
<pandas.core.groupby.generic.DataFrameGroupBy object at 0x00000167F284FC40>
tips_grouped.mean()

tips_grouped[['total_bill']].mean()

tips.groupby('time')[['tip']].max()

tips.groupby(['day', 'time'])[['tip']].max()

# market share over time
total_mcap = df.groupby("date").market_cap.sum()
```
```python
# group rows into buckets based on shared values in a column

# eg isolate fruit & vegetable rows into sep groups to perform calculations
groups = supermarket.groupby("Type") # pandas.core.groupby.generic.DataFrameGroupBy object
groups.get_group("Fruit")

# excels at aggregates
groups.mean()  # calculate mean per group
"""
	    Price
Type
Fruit	    1.4554
Vegetable   0.8800
"""
in_retail = fortune["Sector"] == "Retailing"
retail_companies = fortune[in_retail]
retail_companies["Revenues"].mean() # has tbd for every sector!
# ->
sectors = fortune.groupby("Sector")
sectors.mean()
fortune["Sector"].nunique() # 21 -> number of df's created for sectors
# view sectors
sectors.size()
"""
Aerospace	25
Apparel		35
Chemicals	33
	...
"""
# view sectors as dictionaries of keys with row index positions
sectors.groups
sectors.first()
sectors.last()
# extract row N within group
sectors.nth(0) # first row in groups
sectors.head(2) # first 2 in each group
# all rows from group
sectors.get_group("Energy")
sectors.sum() # show sum|mean|.. per sector
# show total revenue per sector
sectors["Revenues"].sum()
```

### aggregate

```python
# calculate multiple aggregations per sector
aggregations = {
	"Revenues": "min",
	"Profits": "max",
	"Employees": "mean"
}
sectors.agg(aggregations).head()
# return companies with 5 largest profits
fortune.nlargest(n=5, columns="Profits")

# groupby.apply
def get_largest_row(df):
	return df.nlargest(1, "Revenues")  # only largest one

sectors.apply(get_largest_row) # calculates for each sector (df)
```

### multiple groups

```python
# when combo serves as best identifier for a group
sector_and_industry = fortune.groupby(by=["Sector", "Industry"])
sector_and_industry.size()

# use tuples to get all rows that belong to that group
sectors_and_industry.get_group(("Wholesalers", "Health Care"))
```

```python
store_day = df.groupby(["Store", "DayOfWeek"], as_index=False).mean()
# instead of reset_index -> as_index=False
```
### binning

```python
# eg cut sales made
df.groupby("Sales").mean().shape  # (21734, 6)
df.Sales.describe()  # check up to where your sales go to know where you should cut
"""
count    1.017209e+06
mean     5.773819e+03
std      3.849926e+03
min      0.000000e+00
25%      3.727000e+03
50%      5.744000e+03
75%      7.856000e+03
max      4.155100e+04
Name: Sales, dtype: float64
"""
bins = [0, 2000, 4000, 6000, 8000, 10000, 50000]
cuts = pd.cut(df.Sales, bins, include_lowest=True)
# cut is not attached to the dataframe itself.. , include sales with 0!

# add cuts back into original dataframe
df["SalesGroup"] = cuts
df.groupby(["Store", "SalesGroup"]).DayOfWeek.value_counts()
df.groupby(["Store", "SalesGroup"]).DayOfWeek.value_counts().unstack(fill_value=0)
# fill_value=0 (instead of NaN)
"""
DayOfWeek                  1   2   3   4   5    6    7
Store SalesGroup                                      
1     (-0.001, 2000.0]     6   1   3  11   6    0  134
      (2000.0, 4000.0]    28  42  42  43  28   13    0
      (4000.0, 6000.0]    70  80  83  72  91  109    0
      (6000.0, 8000.0]    26  12   7   9   9   10    0
      (8000.0, 10000.0]    4   0   0   0   1    2    0
...                       ..  ..  ..  ..  ..  ...  ...
1115  (2000.0, 4000.0]    15  17  15   9   0    0    0
      (4000.0, 6000.0]    34  54  65  64  61   36    0
      (6000.0, 8000.0]    30  52  39  39  50   80    0
      (8000.0, 10000.0]   37   9  12  10  11   16    0
      (10000.0, 50000.0]  12   2   1   2   7    2    0

[6263 rows x 7 columns]
"""
df.groupby(["Store", "SalesGroup", "DayOfWeek"]).count()
"""
                                    Date  Sales  Customers  Open  Promo  StateHoliday  SchoolHoliday
Store SalesGroup         DayOfWeek                                                                  
1     (-0.001, 2000.0]   1             6      6          6     6      6             6              6
                         2             1      1          1     1      1             1              1
                         3             3      3          3     3      3             3              3
                         4            11     11         11    11     11            11             11
                         5             6      6          6     6      6             6              6
...                                  ...    ...        ...   ...    ...           ...            ...
1115  (10000.0, 50000.0] 3             1      1          1     1      1             1              1
                         4             2      2          2     2      2             2              2
                         5             7      7          7     7      7             7              7
                         6             2      2          2     2      2             2              2
                         7             0      0          0     0      0             0              0

[46830 rows x 7 columns]
"""
```
```python
# count values that are in certain ranges
buckets = [0, 200, 400, 600, 800, 1000]
stocks.value_counts(bins=buckets)
# ( = excluded, ] bracket = included
stocks.value_counts(bins=3)
```

## merge & concatenate

### concat

```python
pd.concat(objs=[groups1, groups2], ingore_index=True) # to get std index
# assign a key G1 & G2 to multiIndex (G1 + regular old row index)
pd.concat(objs=[groups1, groups2], keys=["G1", "G2"])

groups1.head()
#   group_id	name 		category_id		city_id
#0  6388	NYC Pit Bull		26 		10001
categories.head()
#	category_id		category_name
# 0 	1 			Arts & Culture
cities.head()
#  id 		city		state 	zip
#0 7093		West New York	   NJ	7093
```

### merge

[docs merge](https://pandas.pydata.org/pandas-docs/stable/user_guide/merging.html)

- LEFT OUTER JOIN: use keys from left frame only
- RIGHT OUTER JOIN: use keys from right frame only
- FULL OUTER JOIN: use union of keys from both frames
- INNER JOIN: use intersection of keys from both frames

```python
   pd.merge(
       left,
       right,
       how="inner",
       on=None,
       left_on=None,
       right_on=None,
       left_index=False,
       right_index=False,
       sort=True,
       suffixes=("_x", "_y"),
       copy=True,
       indicator=False,
       validate=None,
   )
```

- **left**: A DataFrame or named Series object
- **right**: Another DataFrame or named Series object
- **on**: Column or index level names to join on. Must be found in both the left and right DataFrame and/or Series objects. If not passed and left_index and right_index are False, the intersection of the columns in the DataFrames and/or Series will be inferred to be the join keys
- **left_on**: Columns or index levels from the left DataFrame or Series to use as keys. Can either be column names, index level names, or arrays with length equal to the length of the DataFrame or Series
- **right_on**: Columns or index levels from the right DataFrame or Series to use as keys. Can either be column names, index level names, or arrays with length equal to the length of the DataFrame or Series
- **left_index**: If True, use the index (row labels) from the left DataFrame or Series as its join key(s). In the case of a DataFrame or Series with a MultiIndex (hierarchical), the number of levels must match the number of join keys from the right DataFrame or Series
- **right_index**: Same usage as left_index for the right DataFrame or Series
- **how**: One of 'left', 'right', 'outer', 'inner'. Defaults to inner. See below for more detailed description of each method
- **sort**: Sort the result DataFrame by the join keys in lexicographical order. Defaults to True, setting to False will improve performance substantially in many cases
- **suffixes**: A tuple of string suffixes to apply to overlapping columns. Defaults to ('_x', '_y')
- **copy**: Always copy data (default True) from the passed DataFrame or named Series objects, even when reindexing is not necessary. Cannot be avoided in many cases but may improve performance / memory usage. The cases where copying can be avoided are somewhat pathological but this option is provided nonetheless
- **indicator**: Add a column to the output DataFrame called _merge with information on the source of each row. _merge is Categorical-type and takes on a value of left_only for observations whose merge key only appears in 'left' DataFrame or Series, right_only for observations whose merge key only appears in 'right' DataFrame or Series, and both if the observation’s merge key is found in both
- **validate**: string, default None. If specified, checks if merge is of specified type

    - **one_to_one** or **1:1**: checks if merge keys are unique in both left and right datasets
    - **one_to_many** or **1:m**: checks if merge keys are unique in left dataset
    - **many_to_one** or **m:1**: checks if merge keys are unique in right dataset
    - **many_to_many** or **m:m**: allowed, but does not result in checks


```python
returns = pd.read_excel('returns.xlsx')
returns.head()
"""
Order ID	Returned
0	CA-2016-142398	Yes
1	CA-2017-136651	Yes
2	US-2014-164763	Yes
3	US-2016-152051	Yes
4	CA-2017-154074	Yes
"""
orders_2 = pd.read_excel('orders_2.xlsx')
orders_2.head()
"""
Row ID	Order ID	Order Date	Category	Product Name	Sales	Quantity	Discount	Profit
0	7679	CA-2014-133592	42004	Office Supplies	Xerox 1970	14.94	3	0.0	7.0218
1	6373	CA-2017-143035	43011	Office Supplies	Quartet Alpha White Chalk, 12/Pack	6.63	3	0.0	3.1161
2	8424	CA-2017-150091	43020	Furniture	DAX Two-Tone Silver Metal Document Frame	40.48	2	0.0	17.4064
3	5333	CA-2016-123120	42617	Office Supplies	Acme Preferred Stainless Steel Scissors	22.72	4	0.0	6.5888
4	5596	CA-2017-160325	43002	Technology	Wi-Ex zBoost YX540 Cellular Phone Signal Booster	437.85	3	0.0	131.3550
"""
```

#### left (outer) join

```python
left_join = orders_1.merge(returns, how='left')
left_join.head(12)

"""
Order ID	Order Date	Category	Product Name	Sales	Quantity	Discount	Profit	Returned
0	CA-2017-145429	42937	Office Supplies	GBC Premium Transparent Covers with Diagonal L...	50.352	3	0.2	17.6232	NaN
1	CA-2015-102015	42259	Office Supplies	Tyvek Top-Opening Peel & Seel Envelopes, Plai...	27.180	1	0.0	12.7746	NaN
2	CA-2017-135860	43070	Office Supplies	Avery Durable Slant Ring Binders, No Labels	15.920	4	0.0	7.4824	NaN
3	CA-2017-128363	42960	Office Supplies	Colored Envelopes	14.760	5	0.2	4.7970	NaN
4	CA-2015-142692	42300	Office Supplies	Avery Non-Stick Binders	3.592	1	0.2	1.1225	NaN
5	CA-2017-156237	42992	Technology	Ricoh - Ink Collector Unit for GX3000 Series P...	12.585	1	0.7	-18.0385	NaN
6	CA-2017-117212	42792	Office Supplies	Kensington 6 Outlet Guardian Standard Surge Pr...	81.920	4	0.0	22.1184	NaN
7	US-2015-164175	42124	Furniture	Global Value Mid-Back Manager's Chair, Gray	213.115	5	0.3	-15.2225	NaN
8	CA-2014-119032	41970	Office Supplies	Staples	3.760	2	0.0	1.3160	NaN
9	CA-2017-153787	42874	Office Supplies	Belkin Premiere Surge Master II 8-outlet surge...	97.160	2	0.0	28.1764	NaN
10	CA-2016-121370	42688	Furniture	Global Ergonomic Managers Chair	380.058	3	0.3	-21.7176	NaN
11	CA-2016-109365	42677	Office Supplies	Avery Durable Plastic 1" Binders	43.584	12	0.2	15.7992	Yes
"""

left_join.shape
(53, 9)
```

#### inner join

```python
orders_1.shape
(53, 8)
inner_join = orders_1.merge(returns, how='inner')
inner_join.head(12)
"""
Order ID	Order Date	Category	Product Name	Sales	Quantity	Discount	Profit	Returned
0	CA-2016-109365	42677	Office Supplies	Avery Durable Plastic 1" Binders	43.584	12	0.2	15.7992	Yes
1	CA-2016-151561	42614	Technology	Enermax Aurora Lite Keyboard	468.900	6	0.0	206.3160	Yes
2	US-2016-103674	42710	Office Supplies	Ibico Recycled Linen-Style Covers	437.472	14	0.2	153.1152	Yes
3	CA-2016-146157	42695	Technology	Lenovo 17-Key USB Numeric Keypad	81.576	3	0.2	2.0394	Yes
4	CA-2017-152310	42959	Office Supplies	Xerox 1937	192.160	4	0.0	92.2368	Yes
"""
inner_join.shape
(5, 9)
```

#### concat

```python
orders_append = pd.concat([orders_1, orders_2])
orders_append.shape[0]
110
# are row counts the same? 
orders_append.shape[0] == orders_1.shape[0] + orders_2.shape[0]
True
```

#### summary
```python
# join can only target rows with shared values between both data sets

# left join (A 'groups' is focus, add B 'categories' with same cat_id's)
groups.merge(categories, how="left", on="category_id")

# inner join (only categories data that exists in both df's)
groups.merge(categories, how="inner", on="category_id")

# outer join (combine all values from both df's)
groups.merge(cities, how="outer", left_on="city_id", right_on="id")

# merging on index labels
# right_index=True -> look for city_id in cities df
groups.merge(cities, how="left", left_on="city_id",right_index=True)

# indicator=True -> adds left_ or right_ to string
```


#### full outer join

```python
df_a = pd.DataFrame({"A": ["x", "y", "z"], "B": [1, 2, 3]})
df_b = pd.DataFrame({"A": ["u", "v", "x"], "C": [5.0, 4.0, 3.0]})


pd.merge(df_a, df_b, how="inner", on='A')
"""
   A  B    C
0  x  1  3.0
"""

pd.merge(df_a, df_b, how="outer", on='A')
"""
   A    B    C
0  x  1.0  3.0
1  y  2.0  NaN
2  z  3.0  NaN
3  u  NaN  5.0
4  v  NaN  4.0
"""
```

#### duplicate keys

```python
df_c = pd.DataFrame({"A": ["x", "x", "z"], "B": [1, 2, 3]})
df_d = pd.DataFrame({"A": ["u", "x", "x"], "C": [5.0, 4.0, 3.0]})

pd.merge(df_c, df_d, on='A')
"""
   A  B    C
0  x  1  4.0   x1 * x4.0
1  x  1  3.0   x2 * x3.0
2  x  2  4.0   dito
3  x  2  3.0   dito
"""
```

#### duplicate columns

```python
df_e = pd.DataFrame({"A": ["x", "y", "z"], "B": [1, 2, 3]})
df_f = pd.DataFrame({"A": ["u", "v", "x"], "B": [5.0, 4.0, 3.0]})
print(pd.merge(df_e, df_f, on="A"))
"""
   A  B_x  B_y
0  x    1  3.0
"""
df_m = pd.merge(df_e, df_f, on="A", suffixes=("_left", "_right"))
"""
   A  B_left  B_right
0  x       1      3.0
"""
df_m.rename(columns={"B_left": "Hello", "B_right": "Obi_wan_kenobi"})
"""
   A  Hello  Obi_wan_kenobi
0  x      1             3.0
"""
```

#### validating

Users can use the **validate** argument to automatically check whether there are unexpected duplicates in their merge keys. Key uniqueness is checked before merge operations and so should protect against memory overflows. Checking key uniqueness is also a good way to ensure user data structures are as expected

```python
left = pd.DataFrame({"A": [1, 2], "B": [1, 2]})
right = pd.DataFrame({"A": [4, 5, 6], "B": [2, 2, 2]})
result = pd.merge(left, right, on="B", how="outer", validate="one_to_one")
# MergeError: Merge keys are not unique in right dataset; not a one-to-one merge
```

If the user is aware of the duplicates in the right DataFrame but wants to ensure there are no duplicates in the left DataFrame, one can use the **validate='one_to_many'** argument instead, which will not raise an exception

```python
pd.merge(left, right, on="B", how="outer", validate="one_to_many")
"""
   A_x  B  A_y
0    1  1  NaN
1    2  2  4.0
2    2  2  5.0
3    2  2  6.0
"""
```

```python
pd.merge(df_s, df_p, left_on="id", right_on="student_id", validate="one_to_one")
```

#### merge indicator

```python
df1 = pd.DataFrame({"col1": [0, 1], "col_left": ["a", "b"]})
df2 = pd.DataFrame({"col1": [1, 2, 2], "col_right": [2, 2, 2]})
pd.merge(df1, df2, on="col1", how="outer", indicator=True)
"""
   col1 col_left  col_right      _merge
0     0        a        NaN   left_only
1     1        b        2.0        both
2     2      NaN        2.0  right_only
3     2      NaN        2.0  right_only
"""
```
The indicator argument will also accept string arguments, in which case the indicator function will use the value of the passed string as the name for the indicator column

#### composite keys

```python
df_1 = pd.DataFrame({"year": [2000, 2000, 2001, 2001], "sem": [1, 2, 1, 2], "fee": [200, 200, 200, 200]})
df_2 = pd.DataFrame({"year": [2000, 2000, 2001, 2001], "sem": [1, 2, 1, 2],
                     "student": [1, 2, 2, 3], "discount": [0.1, 0.2, 0.2, 1.0]})
df_3 = pd.DataFrame({"student": [1, 2, 3, 4, 5]})

combined = pd.merge(df_1, df_2, on=["year", "sem"])
"""
   year  sem  fee  student  discount
0  2000    1  200        1       0.1
1  2000    2  200        2       0.2
2  2001    1  200        2       0.2
3  2001    2  200        3       1.0
"""

combined["due_fee"] = combined.fee * (1 - combined.discount)
"""
   year  sem  fee  student  discount  due_fee
0  2000    1  200        1       0.1    180.0
1  2000    2  200        2       0.2    160.0
2  2001    1  200        2       0.2    160.0
3  2001    2  200        3       1.0      0.0
"""

# after merging we don't need key column (axis=1) anymore
pd.merge(df_3, df_2, on="student", how="left")
"""
   student    year  sem  discount
0        1  2000.0  1.0       0.1
1        2  2000.0  2.0       0.2
2        2  2001.0  1.0       0.2
3        3  2001.0  2.0       1.0
4        4     NaN  NaN       NaN
5        5     NaN  NaN       NaN
"""

# cross join aka cartesian product
df_1["key"], df_3["key"] = 1, 1
df_cross = pd.merge(df_1, df_3, on="key").drop("key", axis=1)
"""
    year  sem  fee  student
0   2000    1  200        1
1   2000    1  200        2
2   2000    1  200        3
3   2000    1  200        4
4   2000    1  200        5
..   ...  ...  ...      ...
15  2001    2  200        1
16  2001    2  200        2
17  2001    2  200        3
18  2001    2  200        4
19  2001    2  200        5
"""

all_fees = pd.merge(df_cross, df_2, on=["student", "year", "sem"], how="left")
"""
    year  sem  fee  student  discount
0   2000    1  200        1       0.1
1   2000    1  200        2       NaN
2   2000    1  200        3       NaN
3   2000    1  200        4       NaN
4   2000    1  200        5       NaN
..   ...  ...  ...      ...       ...
15  2001    2  200        1       NaN
16  2001    2  200        2       NaN
17  2001    2  200        3       1.0
18  2001    2  200        4       NaN
19  2001    2  200        5       NaN
"""

all_fees.discount.fillna(0, inplace=True)
all_fees["due"] = all_fees.fee * (1 - all_fees.discount)
all_fees
"""
    year  sem  fee  student  discount    due
0   2000    1  200        1       0.1  180.0
1   2000    1  200        2       0.0  200.0
2   2000    1  200        3       0.0  200.0
3   2000    1  200        4       0.0  200.0
4   2000    1  200        5       0.0  200.0
..   ...  ...  ...      ...       ...    ...
15  2001    2  200        1       0.0  200.0
16  2001    2  200        2       0.0  200.0
17  2001    2  200        3       1.0    0.0
18  2001    2  200        4       0.0  200.0
19  2001    2  200        5       0.0  200.0
"""
```

### merge helper functions

#### pd.merge_ordered

```python
# ffill = forward fill (take previous value if NaN)
pd.merge_ordered(df_be, df_nl, on="date", suffixes=("_be", "_nl"), fill_method="ffill").head(10)
```

#### pd.merge_asof

A merge_asof() is similar to an ordered left-join except that we match on nearest key rather than equal keys. For each row in the left DataFrame, we select the last row in the right DataFrame whose on key is less than the left’s key. Both DataFrames must be sorted by the key

https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.asof.html

get closest or partial match

```python
   df_both = pd.merge_ordered(df_be, df_nl, on="date", suffixes=("_be", "_nl"), fill_method="ffill")
   df_all = pd.merge_asof(df_both, df_bris, on="date").rename(columns={"temperature": "temperature_bxl"})
   df_all.plot("date", ["temperature_be", "temperature_nl", "temperature_bxl"])
   # on="date", tolerance=pd.Timedelta("14 days"), direction="neares"
```

#### tolerance

```python
In [144]: pd.merge_asof(
   .....:     trades,
   .....:     quotes,
   .....:     on="time",
   .....:     by="ticker",
   .....:     tolerance=pd.Timedelta("10ms"),
   .....:     allow_exact_matches=False,
   .....: )
   .....: 
Out[144]: 
                     time ticker   price  quantity    bid    ask
0 2016-05-25 13:30:00.023   MSFT   51.95        75    NaN    NaN
1 2016-05-25 13:30:00.038   MSFT   51.95       155  51.97  51.98
2 2016-05-25 13:30:00.048   GOOG  720.77       100    NaN    NaN
3 2016-05-25 13:30:00.048   GOOG  720.92       100    NaN    NaN
4 2016-05-25 13:30:00.048   AAPL   98.00       100    NaN    NaN
```

## missing values

- **df.dropna()**: drop all or some of the rows that have missing values
- **df.fillna()**: replace the rows that have missing values with that you pass in

### isna notna

```python
df.info()
df.max_supply.notna().sum()
df.max_supply.isna().sum()

df[df.max_supply.notna()]
df.loc[df.max_supply.notna()]
```

```python
penguins.isna().sum()
'''
species               0
island                0
bill_length_mm        2
bill_depth_mm         2
flipper_length_mm     2
body_mass_g           2
sex                  11
dtype: int64
'''

penguins.isna().sum()/len(penguins)
'''
species              0.000000
island               0.000000
bill_length_mm       0.005814
bill_depth_mm        0.005814
flipper_length_mm    0.005814
body_mass_g          0.005814
sex                  0.031977
dtype: float64
'''
```

### dropna

```python
df.dropna().info()  # rows you would have when no NaN values

# you can say only drop it when a row has at least 3 NaN's in it
df.dropna(thresh=3).info()

# only drop events that have "last_review" = Nan
df.dropna(subset=["last_review"]).info()  # by default we operate on row by row basis

df.dropna(axis=1).info()  # you now dropped the entire review column cuz it had a NaN in it
   
penguins.dropna(inplace=True)
penguins.isna().sum()/len(penguins)
'''
species              0.0
island               0.0
bill_length_mm       0.0
bill_depth_mm        0.0
flipper_length_mm    0.0
body_mass_g          0.0
sex                  0.0
dtype: float64
'''
```

```python
employees.dropna(how="all") # rows where all are NaN
employees.dropna(how="any") # rows where any are NaN
employees.dropna(subset=["Gender"]) # remove rows when Nan in Gender column
employees.dropna(how="any", tresh=4) # must have a min of 4 non-null values
```

### isnull notnull

```python
null = df[df.isnull().any(axis=1)]
null = df[df['Units'].isnull()]
notnull = df[df['Units'].notnull()]

```

### fillna

```python
test_fix = df.Sales.transform(lambda x: x.fillna(x.mean()))


# df.fillna(0)
# if eg you have number_of_items = 0 -> you want total_purchased to be 0 instead of NaN
df.fillna(0) # replaces every nan with a 0
# for time-series data you can use method like backfill forwardfill (eg fill with previous finite value)


employees["Mgmt"] = employees["Mgmt"].astype(bool)  # nans become true so beware
# remove Nans eg with fillna & then you'll be able to convert float to ints
employees["Salary"].fillna(0).tail()  # 0 just an example
employees["Salary"].fillna(0).astype(int).tail()
```

## duplicates

```python
employees["Team"].duplicated(keep="first")  # keep first or last occurence

first_person_in_team = ~employees["Team"].duplicated()
employees[first_person_in_team]

employees.drop_duplicates()  # drop if all values are equal

# last employee on team
employees.drop_duplicates(subset=["Team"], keep="last")

# exclude all rows with duplicate values with keep=False
# first names that occur only ones:
employees.drop_duplicates(subset=["First Name"], keep=False)
```

## replace

```python
df.replace("John", "Jono").head(1)  # on every row and every column by default
df.replace("John", "Jono", limit=1)  # John is on 1st row, set a replacement limit
df.host_name.replace({"John": "Jono", "James": "Jamey"})  # on every row and every column by default
```

## tresholding (outliers)

```python
# plt.hist(df.price, log=True)
plt.hist(df.price.clip(upper=1000))
df_copy.loc[df_copy.price > 1000, "price"] = 1000  # identical to .clip(upper=1000)
```

## dtype conversion

### to int
```python

# always convert a string to float first b4 converting to an int
print(astro_df.age_at_zarya.astype("str").astype("float").astype("int"))  # first astype("str") for demo only
```
### to categorical

```python
print(astro_df["Military Rank"].unique())
# [nan 'Colonel' 'Lieutenant Colonel' 'Captain' 'Major General' 'Commander' 'Lieutenant Commander' 'Brigadier General' 'Major' 'Lieutenant General' 'Chief Warrant Officer' 'Rear Admiral' 'Vice Admiral']
print(astro_df["Military Rank"].dtype)  # object
# turn military into a categorical with "category"
astro_df["Military Rank"].astype("category")
astro_df["Military Rank"] = astro_df["Military Rank"].astype("category")  # overwrite orginal mil rank
print(astro_df["Military Rank"].dtype)  # category -> no longer a generic object
```

## timeseries

[timeseries docs](https://pandas.pydata.org/pandas-docs/stable/user_guide/timeseries.html#time-date-components)

```python
pd.read_csv('mydata.csv',index_col='date')

# create series, not a dataframe with squeeze=True
pd.read_csv("pokemon.csv", index_col="Pokemon", squeeze=True)

# read as datetime & not string
pd.read_csv("stocks.csv", index_col="Date", parse_dates=["Date"])

# usecols to only import these columns
pd.read_csv("stocks.csv", index_col="Date", parse_dates=["Date"], usecols= ["Startdate", "Date"], squeeze=True)
```

```python
#! pip install vega_datasets
import pandas as pd
from vega_datasets import local_data

seattle = local_data.seattle_weather()
seattle.head()
'''
date	precipitation	temp_max	temp_min	wind	weather
0	2012-01-01	0.0	12.8	5.0	4.7	drizzle
1	2012-01-02	10.9	10.6	2.8	4.5	rain
2	2012-01-03	0.8	11.7	7.2	2.3	rain
3	2012-01-04	20.3	12.2	5.6	4.7	rain
4	2012-01-05	1.3	8.9	2.8	6.1	rain
'''

seattle['date'].dt.month
'''
0        1
1        1
2        1
3        1
4        1
        ..
1456    12
1457    12
1458    12
1459    12
1460    12
Name: date, Length: 1461, dtype: int64
'''

seattle['month_name'] = seattle['date'].dt.month_name()
seattle.head()
'''
date	precipitation	temp_max	temp_min	wind	weather	month_name
0	2012-01-01	0.0	12.8	5.0	4.7	drizzle	January
1	2012-01-02	10.9	10.6	2.8	4.5	rain	January
2	2012-01-03	0.8	11.7	7.2	2.3	rain	January
3	2012-01-04	20.3	12.2	5.6	4.7	rain	January
4	2012-01-05	1.3	8.9	2.8	6.1	rain	January
'''

seattle.set_index(['date'], inplace=True)
seattle.head()
'''
precipitation	temp_max	temp_min	wind	weather	month_name
date						
2012-01-01	0.0	12.8	5.0	4.7	drizzle	January
2012-01-02	10.9	10.6	2.8	4.5	rain	January
2012-01-03	0.8	11.7	7.2	2.3	rain	January
2012-01-04	20.3	12.2	5.6	4.7	rain	January
2012-01-05	1.3	8.9	2.8	6.1	rain	January
'''

seattle['precipitation'].resample('2W').mean()
'''
date
2012-01-01    0.000000
2012-01-15    3.607143
2012-01-29    8.385714
2012-02-12    2.057143
2012-02-26    4.607143
Freq: 2W-SUN, Name: precipitation, dtype: float64
'''

seattle.reset_index(inplace=True)
seattle.head()

'''
date	precipitation	temp_max	temp_min	wind	weather	month_name
0	2012-01-01	0.0	12.8	5.0	4.7	drizzle	January
1	2012-01-02	10.9	10.6	2.8	4.5	rain	January
2	2012-01-03	0.8	11.7	7.2	2.3	rain	January
3	2012-01-04	20.3	12.2	5.6	4.7	rain	January
4	2012-01-05	1.3	8.9	2.8	6.1	rain	January
'''
```

### timestamp

```python
# Pandas uses Timestamp object ipv Datetime
pd.Timestamp("2015-03-31") # string becomes timestamp
pd.Timestamp("2015/03/31") # string becomes timestamp

# Timestamp('2015-03-31 00:00:00')
my_time = pd.Timestamp(dt.datetime(2000, 2, 3, 21, 35, 22))

# Timestamp('2000-02-03 21:35:22')
my_time.year  # immutable!
my_time.second  # immutable!
```

### datetimeindex

```python
string_dates = ["2018/01/02","2016/04/12","2009/09/07"]
dt_index = pd.DatetimeIndex(data=string_dates)

s = pd.Series(data=[100, 200, 300], index=dt_index)
s.sort_index()
```
### to datetime

```python
disney = pd.read_csv("disney.csv", parse_dates=["Date"])
dt_index = pd.to_datetime(string_dates)
disney["Date"] = pd.to_datetime(disney["Date"])
```

### using dt object
```python
disney["Date"].dt.day.head(3) # returns day series
disney["Date"].dt.dayofweek.head()
disney["Date"].dt.day_name.head()

disney["DayofWeek"] = disney["Date"].dt.day_name()

# group based on weekday
group = disney.groupby("DayofWeek")
group.mean()
"""
		High 	Low 	Open 	Close
DayofWeek
Friday		23.77	23.21 	23.45 	23.55
Monday	 ..
Thursday ..
Tuesday  ..
"""

disney["Date"].dt.month_name().head()

# dates that fell on start of quarter boolean
disney["Date"].dt.is_quarter_start.head() # on 01/01..
disney["Date"].dt.is_quarter_end.head() # on 31/03..

dates_at_month_start = disney["Date"].dt.is_month_start
dates_at_month_end = disney["Date"].dt.is_month_end

disney[dates_at_month_start].head()
disney[dates_at_month_end].head()

dates_at_year_start = disney["Date"].dt.is_year_start
```

### date offsetting

```python
# adding & subtracting durations of time
pd.DateOffset(years=3, months=4, days=5)
(disney["Date"] + pd.DateOffset(days=5))  # add 5 days correction

df = pd.DataFrame(data=["2023-03-28T17:05.12"],columns=["date"], dtype='M8[ns]')
df.date.at[0] + pd.DateOffset(months=1)
Timestamp('2023-04-28 17:05:07')

# round date to end of month
(disney["Date"] + pd.offsets.MonthEnd()) # 30/06 become 31/07 can't round to same date
# use - to go back
pd.offsets.BMonthEnd() # BusinessMonthEnd
```

### timedelta

[timedelta docs](https://pandas.pydata.org/pandas-docs/stable/user_guide/timedeltas.html)

```python
dt.timedelta(
	weeks=8,
	days=6,
	hours=3,
	minutes=58,
	seconds=12
	)
# Output: datetime.timedelta(days=62, seconds=14292)
```
```python
# Timedelta object (distance between two durations) note capital T
duration = pd.Timedelta(
	days=8,
	hours=7,
	minutes=6,
	seconds=5)
	
duration = pd.to_timedelta("3 hours, 5 minutes, 12 seconds")

pd.to_timedelta(5, unit="hour")

pd.to_timedelta([5, 10, 15], unit="day")
# TimedeltaIndex(['5 days', '10 days', '15 days']), dtype='timedelta64[ns]', freq=None)

pd.Timestamp("1999-02-05") - pd.Timestamp("1998-05-24")
# Timedelta('257 days 00:00:00')

# packages that took +1 year to deliver
(deliveries["duration"] > pd.Timedelta(days=365)) # or
(deliveries["duration"] > "365 days")
```

### datetime filtering

```python
s3 = pd.Series(list('abcde'), pd.date_range('now', periods=5, freq='M'))
s3
'''
   2021-01-31 16:41:31.879768    a
   2021-02-28 16:41:31.879768    b
   2021-03-31 16:41:31.879768    c
   2021-04-30 16:41:31.879768    d
   2021-05-31 16:41:31.879768    e
'''
```

```python
# get row where index = datetime
# row_of_interest = df[ column_Series == date_given]
result_row = df[df["timestamp"] == "2023-01-05"]

# get row where columns are datetime
date_in_columns_bool = pd.to_datetime("2023-01-05").date() in us_yields_df.columns

give_index = us_yields_df.columns.tolist().index(pd.to_datetime("2023-01-05").date())

give_row = us_yields_df.loc[:, pd.to_datetime("2023-01-05").date()]
```

```python
df.loc['2014-01-01':'2014-02-01']
df.query('20130101 < date < 20130201')
# using datetime object
df.loc[datetime.date(year=2014,month=1,day=1):datetime.date(year=2014,month=2,day=1)]
# using date
# recommended to use Timestamp rather than date
df[(df['date'] > datetime.date(2016,1,1)) & (df['date'] < datetime.date(2016,3,1))] 
df[(df['date'] > pd.Timestamp(2016,1,1)) & (df['date'] < pd.Timestamp(2016,3,1))]
# if already applied pd.to_datetime just do
df = df[(df['Date'] > "2018-01-01") & (df['Date'] < "2019-07-01")]

value_to_check = pd.Timestamp(date.today().year, 1, 1)
filter_mask = df['date_column'] < value_to_check
filtered_df = df[filter_mask]

# if the dates are in the index then simply:
df.loc['20160101':'20160301']

df.query('date > 20190515071320')
df.query('date > "20190515071320"')
df.query('date > "2019-05-15 07:13:20"')

df.loc[df.index.month == 3]
df.loc[df.ColumnName.dt.month == 3]

# datetime based index descending order
df.loc['2021-08-01':'2021-08-31']
# returns empty

df.loc['2021-08-31':'2021-08-01']
# returns expected data
```
```python
start = datetime.date(2021,1,1).strftime('%Y%m%d')
end = datetime.date(2021,1,3).strftime('%Y%m%d')
df.query(f"{start} < MyDate < {end}")

# 60 days from today
after_60d = pd.to_datetime('today').date() + datetime.timedelta(days=60)
# filter date col less than 60 days date
df[df['date_col'] < after_60d]
```

```python
date_filter = df[df['Date'] > '2020-05-01']
date_filter2 = df[df['Date'] == '2020-05-01']
date_filter3 = df[(df['Date'] >= '2020-05-01') & (df['Date'] < '2020-06-01')]

df['Months'] = df['Date'].dt.month
df['Weekday'] = df['Date'].dt.weekday
df['Year'] = df['Date'].dt.year
df['Weekday Name'] = df['Date'].dt.weekday_name

tuesdays = df[df['Date'].dt.weekday == 2]
```

## resample

allows you to resample a dataset with a timeseries index
The method accepts a periodicity that you want to resample to, such as 'W' for week or 'H' for hour

Since you’ll want to provide some method by which to invent your data, you can chain in another method, such as .mean(), to resample with that aggregation function

Let’s resample our hourly data to daily data

The process of resampling refers to changing the frequency of your data. You have two main methods available when you want to resample your timeseries data:

**Upsampling**: increasing the frequency of your data, such as from hours to minutes
**Downsampling**: decreasing the frequency of your data, such as from hours to days
Both methods require you to invent data, since the data points don’t actually exist

```python
# Resampling an Entire DataFrame with the Same Method
df = df.resample('D').mean()

# Resampling Data with Different Methods
df = df.resample('D').agg({
    'Close Price': 'last',
    'High Price': 'max',
    'Low Price': 'min',
    'Open Price': 'first',
    'Volume': 'sum'
})
```

## apply

apply works on a row / column basis of a DataFrame

```python
"""
eg: df.apply(np.square)

Apply: It is used when you want to apply a function along the axis of a dataframe
accepts a Series whose index is either column (axis=0) or row (axis=1)
apply works on df and series too
map works on series only """

random_data = np.round(np.random.random(size=(4, 3)), 2)  # 2 decimal points
dfx = pd.DataFrame(random_data, columns=["A", "B", "C"])
print(dfx.head())
dfx.apply(lambda x: 1 + np.abs(x))  # apply needs vectorized function
print(dfx.A.apply(np.abs))  # abs makes neg number positive

# apply requires a vectorized function!
# def double_if_positive(x):
#  if x > 0:
#      return 2 * x  # x here is an array, not a single number!
#  return x

def double_if_positive(x):
    x = x.copy()  # prevents you from mutating x datasource
    x[x > 0] *= 2  # masked array  x[condition]
    return x

dfx.apply(double_if_positive, raw=True)  # raw=False
# Panda Series so 1 col at a time, raw=True fast on simple calculations
# ValueError: The truth value of a Series is ambiguous. Use a.empty, a.bool(), a.item(), a.any() or a.all().
```
## map apply mapapply

- map is defined on Series only
- applymap is defined on DataFrames only
- apply is defined on both

### map

map works element-wise on a Series

```python
# map operates on Series & uses dictionary input not array input
series = pd.Series(["Steve", "Alex", "Jess", "Mark"])
series.map({"Steve": "Stephen"})
"""
0    Stephen
1        NaN
2        NaN
3        NaN
dtype: object
"""
# unlike apply (needs vectorized function) this goes over each element 1 at a time
series.map(lambda fn: f"I am {fn}")
```

### vectorized functions

```python

# https://medium.com/codex/vectorizing-functions-in-numpy-d00dc5e58f65

sky_series = pd.Series(["Obi-Wan Kenobi", "Luke Skywalker", "Han Solo", "Leia Organa"])
"""
0 [Obi-Wan, Kenobi]
1 [Luke, Skywalker]
2 [Han, Solo]
3 [Leia, Organa]
dtype: object
"""

# user defined functions
data2 = np.random.normal(10, 2, size=(100000, 2))
df_extra1 = pd.DataFrame(data2, columns=["x", "y"])

hypot = (df_extra1.x ** 2 + df_extra1.y ** 2) ** 0.5

# non vectorized -> takes 16 seconds
def hypot_bad(x, y):
    return np.sqrt(x**2 + y**2)


for index, (x, y) in df_extra1.iterrows():
    h1.append(hypot_bad(x, y))


# vectorized -> quick 13ms
def hypot_good(row_of_xs, row_of_ys):  # feed it with rows of data not element by element
    return np.sqrt(row_of_xs ** 2 + row_of_ys ** 2)


h2 = hypot_good(df_extra1.x, df_extra1.y)
```

### applymap

applymap works element-wise on a DataFrame

```python
from pycoingecko import CoinGeckoAPI
import pandas as pd
pd.set_option("display.float_format",lambda x: "%.2f" % x)

df.nunique()
data_btc = cg.get_coin_market_chart_by_id(id="bitcoin",vs_currency="usd",days="max",interval="daily")

btc_history = pd.DataFrame(data_btc).applymap(lambda x: x[1]) # ignore elem 0 datetime

btc_dates = pd.to_datetime(pd.DataFrame(data_btc).prices.apply(lambda x: x[0]),unit="ms")
df.index = btc_dates
```


### transform

transform -> like apply but has to return a series with same size as the input
[read more](https://stackoverflow.com/questions/27517425/apply-vs-transform-on-a-group-object)

```python
# instead of NaN fill with mean value of the series
test_fix = df.Sales.transform(lambda x: x.fillna(x.mean()))
```


## aggregate

```python
# sales are heavily dependant on day of the week, so use that intelligence with groupby cuts

# different aggregates for different columns
# aggregate statistics functions can be used to calc stats on a column of a DataFrame Eg df.columnName.mean()
# aggregate statistic functions can be applied across multiple rows by using a groupby function
# give the mean for the Sales & Customers column
df.groupby(["Store", "DayOfWeek", ]).agg({"Sales": "mean", "Customers": np.mean})  # str for common funcs works too

# do multiple things with 1 column (pass multiple things through a list)
df.groupby(["Store", "DayOfWeek", ]).agg({"Sales": ["mean", "max", "min"], "Customers": "count"})
"""
                          Sales              Customers
                           mean    max   min     count
   Store DayOfWeek
   1     1          5177.968750   9528  3414       128
         2          4685.626866   7959  2362       134
         3          4555.712121   7821  2605       132
         4          4457.838710   7785  2462       124
         5          4726.480620   8414  3198       129
"""
monte_carlo_uncertainty = lambda x: np.std(x) / np.sqrt(x.size)
df2 = df.groupby(["Store", "DayOfWeek", ]).agg({"Sales": ["mean", monte_carlo_uncertainty], "Customers": "count"})
df2
"""
               		Sales             Customers
                           mean  <lambda_0>     count
   Store DayOfWeek
   1     1          5177.968750  108.777349       128
         2          4685.626866   88.507530       134
"""
```

### override column names using tuple


```python

df2 = df.groupby(["Store", "DayOfWeek", ]).agg(
    {
      "Sales": [("SalesMean", "mean"),
	        ("SalesUncert", monte_carlo_uncertainty)
	       ],
      "Customers": "count"
    })
df2.head()
# notice the nested multi-column layout
"""
                Sales             Customers
                      SalesMean SalesUncert     count
   Store DayOfWeek
   1     1          5177.968750  108.777349       128
         2          4685.626866   88.507530       134
         3          4555.712121   79.957721       132
"""
```

```python
# new way of using aggregates (recommended) won't work with lambda as all functions need to have a name
# => only 1 layer of columns
def mc_uncert(x):
    return np.std(x) / np.sqrt(x.size)

dfagg = df.groupby(["Store", "DayOfWeek"])
dfagg.agg(
       SalesMean=("Sales", "mean"),
       SalesUncert=("Sales", mc_uncert)
   ).reset_index().head()
   """
      Store  DayOfWeek    SalesMean  SalesUncert
   0      1          1  5177.968750   108.777349
   1      1          2  4685.626866    88.507530
   2      1          3  4555.712121    79.957721
   3      1          4  4457.838710    82.332880
   4      1          5  4726.480620    80.066588
"""
```

## configure options

### pandas

```python
pd.describe_option("display.max_rows")  # default = 60
pd.describe_option("max_col") # get info on display.max_columns & max_colwidth

pd.get_option("display.max_rows")
pd.set_option("display.max_rows", 6)
# same as
pd.options.display.max_rows # 60
pd.options.display.max_rows = 6
# reset
pd.reset_option("display.max_rows")
# precision floats
pd.describe_option("display.precision")
pd.set_option("display.precision", 2)
pd.options.display.precision = 2

pd.describe_option("display.max_colwidth") 
# only print if value >= 0.25 otherwise print 0
pd.set_option("display.chop_threshold", 0.25)

# settings for one cell
with pd.option_context(
	"display.max_columns", 5,
	"display.max_rows", 10,
	"display.precision", 3
):
	display(df)  # notebook function display
```

### ipython
```python
from IPython.display import HTML
pd.options.display.max_colwidth = 200

HTML(df.head(20).to_html(escape=False))



pd.set_option("display.float_format", lambda x: "%.2f" % x)
df = pd.read_csv("panel_data.csv", parse_dates=["date"])
```

## optimize memory usage

```python
# convert to category where you can, check with nunique()
employees.nunique()
employees["Gender"] = employees["Gender"].astype("category")
```


## string manipulations

```python
# remove whitespace around text
employees["First Name"].str.strip()
inspections["Name"].str.lstrip()
for column in inspections.columns:
	inspections[column] = inspections[column].str.strip()
str.lower()
str.upper()
str.capitalize()  # cap first letter
str.title() # caps first letter of each word (countries names cities locations)
```

```python
# replace text
inspections = inspections.replace(to_replace="All", value="Risk 4 (Extreme)")
inspections["Risk"].str[8:].head()
"""
High)
Medium)
Low)
High)
High)
"""

# regex replace digits with an asterisk
# 0  	1256 Trace Ports Apt. 419
customers["Street"].str.replace("\d{4,}", "*", regex=True)
```

```python
inspections["Risk"].str[8:-1].head()  # remove ) too
inspections["Risk"].str.slice(8).str.replace(")", "").head()  # remove ) too
inspections["Name"].str.lower().str.contains("pizza").head() # consistency w lower first
inspections["Name"].str.lower().str.startswith("tacos").head()
inspections["Name"].str.lower().str.endswith("tacos").head()

# expand=True returns dataframe
customers["Address"].str.split(", ", expand=True)


# adjust what should be used as column index & header
# eg here state,city,street = multiIndex tuple
# Category cols service or culture
neighborhoods=pd.read_csv("nbs.csv", index_col=[0,1,2], header=[0,1])

# to get city
neighborhoods.index.get_level_values(1)
neighborhoods.index.get_level_values("City")

# name multi index column index:
neighborhoods.columns.names = ["Category", "Subcategory"]
neighborhoods.columns.get_level_values(0)
neighborhoods.columns.get_level_values("Category")
# Index(['Culture','Culture','Services','Services'],dtype='object', name='Category')
```

## reshaping & pivoting

```python
# use Date column as pivot, calc mean with aggfunc param
sales.pivot_table(index="Date", aggfunc="mean")

sales.pivot_table(index="Date", aggfunc="sum")

# onlu sum up Revenue values
sales.pivot_table(index="Date", values="Revenue", aggfunc="sum")

# revenue per salesman, replace NaN with 0
sales.pivot_table(index="Date", columns="Name", values="Revenue", aggfunc="sum", fill_value=0)
# name      	Creed	Jim	   Bart
# Date
# 2021-01-01	4444    1234   3456
# 2021-01-01	   0    5434   1456

# date + salesman subtotal margins=True (add totals for each row and column)
sales.pivot_table(index="Date", columns="Name", values="Revenue", aggfunc="sum",
	fill_value=0, margins=True, margins_name="Total")

# aggfunc="count" count no of sales rows for each Date x Employee

# max, min, std, median, size (same as count)
sales.pivot_table(
	index="Date",
	columns="Name",
	values="Revenue",
	fill_value=0,
	aggfunc=["sum", "count"]
	)

sales.pivot_table(
	index="Date",
	columns="Name",
	values=["Revenue", "Expenses"],
	fill_value=0,
	aggfunc={"Revenue":"min", "Expenses": "max"}
	)
```

## (un)stack melt explode

```python
# move date from column axis to row axis
df.stack()
# row to col axis
df.unstack()

# melting = wide data set -> narrow data set
video_game_sales.melt(id_vars="Name", value_vars="NA")

regional_sales_columns = ["NA", "EU", "JP", "Other"]
video_game_sales.melt(id_vars="Name", value_vars=regional_sales_columns)

# explode a list of walues
recipes.explode("Ingredients") # requires a Series of Lists
```

## importing/exporting

### json

```python
# importing json
nobel = pd.read_json("nobel.json")

pd.json_normalize(data=chemistry_2019) # still dicts in laureates
pd.json_normalize(data=chemistry_2019, record_path="laureates") # norm this column

# preserve year & category columns!
pd.json_normalize(data=chemistry_2019,
	record_path="laureates",
	meta=["year", "category"])

# add key, value if key doesn't exist
cheese_consumption.setdefault("France", 100)

# if no laureates key exist, add one (= no error at importing)
def add_laureates_key(entry):
	entry.setdefault("laureates", [])

nobel["prizes"].apply(add_laureates_key)

winners = pd.json_normalize(
	data=nobel["prizes"],
	record_path="laureates",
	meta=["year", "category"]
	)
```

```python
# exporting json
winners.head(2).to_json(orient="records") # json arr of k,v objects
winners.head(2).to_json(orient="split") # seperate column, index & data keys
# other orient options: index, columns, values, table
winners.head(2).to_json("winners.json", orient="records") # will overwrite!

# save you from dld manually
url = "https://data.nyc.us/api/views/345345/rows.csv"
baby_names = pd.read_csv(url)

baby_names.to_csv("data/baby_names.csv", index=False)
# only these columns
baby_names.to_csv("data/baby_names.csv", index=False,
		columns=["Gender", "Child's First Name", "Count"])
```

### read excel workbooks

```python
# install xlrd and openpyxl libs
conda install xlrd openpyxl

pd.read_excel("Single Worksheet.xlsx")
pd.read_excel(
	io="Single Worksheet.xlsx",
	usecols=["City", "First Name", "Last Name"],
	index_col = "City"
	)

# squeeze=True to force a series object
pd.read_excel("Multiple Worksheets.xlsx", sheet_name=0) # first worksheet

pd.read_excel("Multiple Worksheets.xlsx", sheet_name="Data 1")

# import all worksheets
pd.read_excel("Multiple Worksheets.xlsx", sheet_name=None)

# each sheet = a df
workbook["Data 2"]

# specified worksheet only
pd.read_excel("Multiple Worksheets.xlsx", sheet_name=["Data 1", "Data 3"])
pd.read_excel("Multiple Worksheets.xlsx", sheet_name=[1,3])
```

### exporting excel workbook

```python
# create ExcelWriter object
excel_file = pd.ExcelWriter("data/BabyNames.xlsx")
boys.to_excel(
	excel_writer=excel_file,
	sheet_name="Boys",
	index=False,  # exclude df's index
	columns=["Child's First Name", "Count", "Rank"]
	)
excel_file.save()
```

## pyarrow

```python
pd.Series(["foo", "bar", "foobar"], dtype="string[pyarrow]")
pd.options.mode.dtype_backend = "pyarrow"  # srting = string & niet object

# pyarrow is good at handling missing values
# None doesn't turn int64 into a float dtype, None become <NA> instead of NaN
df_arrow = pd.read_csv("names.csv", engine="pyarrow", use_nullable_dtypes=True)
```
