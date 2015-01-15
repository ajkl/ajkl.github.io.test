---
layout: post
title: Dataframes
---

I have been playing a lot lately with [Julia](http://julialang.org/).
In the past I have used python and R a lot for my data analysis tasks. I always tend to have some cheatsheet in front of me for the basic dataframe operations in pandas and R. Its hard to make that context switch in the syntax from one language to another.
With Julia, my brain was just not ready to accept another set of syntax.
So I thought a quick reference comparing the basic dataframe manipulation syntax for all 3 languages would be nice.
Most of my data tasks start with some ETL tasks on dataframes. So this is more of a reference for my use in future but can be useful to others too who face the same situation.

I took this awesome tutorial by [Greg Reda](http://www.gregreda.com/) on ["Working with DataFrames"](http://www.gregreda.com/2013/10/26/working-with-pandas-dataframes/) and tried to port the example to R and Julia.

All the code from this post can be found at this [github repo dataframes-compare](https://github.com/ajkl/dataframes-cheatsheet).

We start with the [MovieLens 100k dataset](http://grouplens.org/datasets/movielens/). 

This data set consists of:

* 100,000 ratings (1-5) from 943 users on 1682 movies. 
* Each user has rated at least 20 movies. 
* Simple demographic info for the users (age, gender, occupation, zip)

## Data import
Lets start with loading the data into our dataframe. The methods to read the data from a flat file are pretty simple and straight forward in all 3, the only tricky part is when adding column names in Julia. But there are ways to get around.

#### Julia

{% highlight julia %}
using DataFrames
u_col_names=[symbol("user_id"), symbol("age"), symbol("sex"), symbol("occupation"), symbol("zip_code")]
{% endhighlight %}

Another way to do the same without adding each entry as a symbol. Thanks to [Jubobs](http://stackoverflow.com/users/2541573/jubobs) from this [stackoverflow post](http://stackoverflow.com/questions/27629206/why-does-this-list-comprehension-return-an-arrayany-1-instead-of-an-arraysymb/27629609#27629609) for the suggestion.

{% highlight julia %}
col_names=["user_id", "age", "sex", "occupation", "zip_code"]
u_col_names=map(symbol, col_names)
users = DataFrames.readtable("data/ml-100k/u.user", separator='|', header=false, names=u_col_names)
{% endhighlight %}

Edit: Probably the easiest way to do this is (as pointed out by lot of people on HN and in the comments section here)

{% highlight julia %}
u_col_names=[:user_id,  :age,  :sex,  :occupation,  :zip_code]
users = DataFrames.readtable("data/ml-100k/u.user", separator='|', header=false, names=u_col_names)
{% endhighlight %}

There is no way right now to load selective columns from a file. The next IO version will hopefully have that mechanism.
[Stackoverflow question selecting-columns-while-importing-data-with-dataframes-readtable](http://stackoverflow.com/questions/27628366/selecting-columns-while-importing-data-with-dataframes-readtable)

#### Python

{% highlight python %}
import pandas as pd

u_col_names = ['user_id', 'age', 'sex', 'occuptation', 'zip_code']
users = pd.read_csv('data/ml-100k/u.user', sep = '|', names=u_col_names)
{% endhighlight %}
Let's only load the first five columns of the file with usecols
{% highlight python %}
m_col_names = ['movie_id', 'title', 'release_date', 'video_release_date', 'imdb_url']
movies = pd.read_csv('data/ml-100k/u.item', sep='|', names=m_col_names, usecols=range(5))
{% endhighlight %}

#### R

{% highlight r %}
u_col_names <- c('user_id', 'age', 'sex', 'occupation', 'zip_code')
users <- read.csv('data/ml-100k/u.user', sep='|', col.names=u_col_names, header=FALSE)
{% endhighlight %}
Loading the first 5 columns only
{% highlight r %}
m_col_names = c('movie_id', 'title', 'release_date', 'video_release_date', 'imdb_url')
movies = read.table('data/ml-100k/u.item', sep='|', colClasses=c("integer", "character", "factor", "factor", "character", rep("NULL", 19)), quote="")
{% endhighlight %}
[Stackoverflow question only-read-limited-number-of-columns-in-r](http://stackoverflow.com/questions/5788117/only-read-limited-number-of-columns-in-r) also quotes in strings cause importing errors so you need quote="" 
Cannot specify col.names in read.table as we are skipping 19 columns. R will complain with "more columns than column names"
{% highlight r %}
colnames(movies) <- m_col_names
{% endhighlight %}

If you notice closely R and python accept string literals in single quotes ' but Julia treats is as a character literal. I personally feel thats how it should be if I think about how C/C++ treats string literals.

## Sanity check
Now that we have the data loaded in our dataframes, its usually good practice to see the classes/types of each column. We will also try to get the summary statistics for the columns of our dataframe.

#### Julia
{% highlight julia %}
eltypes(users)

	5-element Array{Type{T<:Top},1}:
	 Int64     
	 Int64     
	 UTF8String
	 UTF8String
	 UTF8String


describe(users)

	user_id
	Min      1.0
	1st Qu.  236.5
	Median   472.0
	Mean     472.0
	3rd Qu.  707.5
	Max      943.0
	NAs      0
	NA%      0.0%
	
	age
	Min      7.0
	1st Qu.  25.0
	Median   31.0
	Mean     34.05196182396607
	3rd Qu.  43.0
	Max      73.0
	NAs      0
	NA%      0.0%
	
	sex
	Length  943
	Type    UTF8String
	NAs     0
	NA%     0.0%
	Unique  2
	
	occupation
	Length  943
	Type    UTF8String
	NAs     0
	NA%     0.0%
	Unique  21
	
	zip_code
	Length  943
	Type    UTF8String
	NAs     0
	NA%     0.0%
	Unique  795
{% endhighlight %}

#### Python
{% highlight julia %}
type(users)

	pandas.core.frame.DataFrame

users.dtypes

	user_id         int64
	age             int64
	sex            object
	occuptation    object
	zip_code       object
	dtype: object

users.describe

		user_id		age
	count	943.000000	943.000000
	mean	472.000000	34.051962
	std	272.364951	12.192740
	min	1.000000	7.000000
	25%	236.500000	25.000000
	50%	472.000000	31.000000
	75%	707.500000	43.000000
	max	943.000000	73.000000
{% endhighlight %}

#### R
{% highlight r %}
class(users)

	[1] "data.frame"

lapply(users, class)

	$user_id
	[1] "integer"
	
	$age
	[1] "integer"
	
	$sex
	[1] "factor"
	
	$occupation
	[1] "factor"
	
	$zip_code
	[1] "factor"
	
str(users)

	'data.frame':	943 obs. of  5 variables:
	'data.frame':	943 obs. of  5 variables:
	 $ user_id   : int  1 2 3 4 5 6 7 8 9 10 ...
	 $ age       : int  24 53 23 24 33 42 57 36 29 53 ...
	 $ sex       : Factor w/ 2 levels "F","M": 2 1 2 2 1 2 2 2 2 2 ...
	 $ occupation: Factor w/ 21 levels "administrator",..: 20 14 21 20 14 7 1 1 19 10 ...
	 $ zip_code  : Factor w/ 795 levels "00000","01002",..: 623 690 271 332 134 759 652 49 2 643 ...
	 $ user_id   : int  1 2 3 4 5 6 7 8 9 10 ...
	 $ age       : int  24 53 23 24 33 42 57 36 29 53 ...
	 $ sex       : Factor w/ 2 levels "F","M": 2 1 2 2 1 2 2 2 2 2 ...
	 $ occupation: Factor w/ 21 levels "administrator",..: 20 14 21 20 14 7 1 1 19 10 ...
	 $ zip_code  : Factor w/ 795 levels "00000","01002",..: 623 690 271 332 134 759 652 49 2 643 ...

summary(users)

	    user_id           age        sex             occupation     zip_code  
	 Min.   :  2.0   Min.   : 7.00   F:273   student      :196   55414  :  9  
	 1st Qu.:237.2   1st Qu.:25.00   M:669   other        :105   55105  :  6  
	 Median :472.5   Median :31.00           educator     : 95   10003  :  5  
	 Mean   :472.5   Mean   :34.06           administrator: 79   20009  :  5  
	 3rd Qu.:707.8   3rd Qu.:43.00           engineer     : 67   55337  :  5  
	 Max.   :943.0   Max.   :73.00           programmer   : 66   27514  :  4  
	                                         (Other)      :334   (Other):908  
	    user_id           age        sex             occupation     zip_code  
	 Min.   :  2.0   Min.   : 7.00   F:273   student      :196   55414  :  9  
	 1st Qu.:237.2   1st Qu.:25.00   M:669   other        :105   55105  :  6  
	 Median :472.5   Median :31.00           educator     : 95   10003  :  5  
	 Mean   :472.5   Mean   :34.06           administrator: 79   20009  :  5  
	 3rd Qu.:707.8   3rd Qu.:43.00           engineer     : 67   55337  :  5  
	 Max.   :943.0   Max.   :73.00           programmer   : 66   27514  :  4  
	                                         (Other)      :334   (Other):908  							
{% endhighlight %}

We can see that the ratio is around 1:2.5 between female and male users and most of them (if you exclude "other") are students. Minimum age is 7 and it goes till 73 as the max.

## Subsetting

Lets try to query our data now and filter it with some conditions. 

### Head/Tail
We will warmup with some top/bottom values to get a feel of the data we have at hand.


#### Julia
{% highlight julia %}
head(users)

	6x5 DataFrame
	| Row | user_id | age | sex | occupation   | zip_code |
	|-----|---------|-----|-----|--------------|----------|
	| 1   | 1       | 24  | "M" | "technician" | "85711"  |
	| 2   | 2       | 53  | "F" | "other"      | "94043"  |
	| 3   | 3       | 23  | "M" | "writer"     | "32067"  |
	| 4   | 4       | 24  | "M" | "technician" | "43537"  |
	| 5   | 5       | 33  | "F" | "other"      | "15213"  |
	| 6   | 6       | 42  | "M" | "executive"  | "98101"  |

tail(users)

	6x5 DataFrame
	| Row | user_id | age | sex | occupation      | zip_code |
	|-----|---------|-----|-----|-----------------|----------|
	| 1   | 938     | 38  | "F" | "technician"    | "55038"  |
	| 2   | 939     | 26  | "F" | "student"       | "33319"  |
	| 3   | 940     | 32  | "M" | "administrator" | "02215"  |
	| 4   | 941     | 20  | "M" | "student"       | "97229"  |
	| 5   | 942     | 48  | "F" | "librarian"     | "78209"  |
	| 6   | 943     | 22  | "M" | "student"       | "77841"  |
{% endhighlight %}

If you want to select a custom number of rows in head or tail you can pass it as a param

{% highlight julia %}
head(users, 3)

	3x5 DataFrame
	| Row | user_id | age | sex | occupation   | zip_code |
	|-----|---------|-----|-----|--------------|----------|
	| 1   | 1       | 24  | "M" | "technician" | "85711"  |
	| 2   | 2       | 53  | "F" | "other"      | "94043"  |
	| 3   | 3       | 23  | "M" | "writer"     | "32067"  |
{% endhighlight %}

I wont paste the output of these for python and R as they are similar to what Julia shows..

#### Python
{% highlight python %}
users.head()
users.tail()
users.head(3)
{% endhighlight %}

#### R
{% highlight r %}
head(movies)
tail(movies)
head(movies, n=3)
{% endhighlight %}

### Row subset

Lets try to get all the rows from 50th row to the 55th row

#### Julia
{% highlight julia %}
users[50:55,:]

	6x5 DataFrame
	| Row | user_id | age | sex | occupation   | zip_code |
	|-----|---------|-----|-----|--------------|----------|
	| 1   | 50      | 21  | "M" | "writer"     | "52245"  |
	| 2   | 51      | 28  | "M" | "educator"   | "16509"  |
	| 3   | 52      | 18  | "F" | "student"    | "55105"  |
	| 4   | 53      | 26  | "M" | "programmer" | "55414"  |
	| 5   | 54      | 22  | "M" | "executive"  | "66315"  |
	| 6   | 55      | 37  | "M" | "programmer" | "01331"  |
{% endhighlight %}

#### Python
{% highlight python %}
users[49:55]
{% endhighlight %}

#### R
{% highlight r %}
users[50:55,]
{% endhighlight %}
Notice that Julia and R have 1-based indexing whereas python has 0-based indexing. 
Also python slicing a:b is a(included) and b(excluded) so you have to do 49:55 to get rows 50-55 in python.

### Column subset

You can select a single column by column name.

#### Julia
{% highlight julia %}
head(users[:occupation])

	6-element DataArray{UTF8String,1}:
	 "technician"
	 "other"     
	 "writer"    
	 "technician"
	 "other"     
	 "executive" 
{% endhighlight %}

#### Python
{% highlight python %}
users['occupation'].head()
{% endhighlight %}

#### R
{% highlight r %}
head(users$occupation)
{% endhighlight %}

Multiple column selection works by passing a vector of column names

#### Julia
{% highlight julia %}
head(users[:,[:occupation, :sex, :age]])

	6x3 DataFrame
	| Row | occupation   | sex | age |
	|-----|--------------|-----|-----|
	| 1   | "technician" | "M" | 24  |
	| 2   | "other"      | "F" | 53  |
	| 3   | "writer"     | "M" | 23  |
	| 4   | "technician" | "M" | 24  |
	| 5   | "other"      | "F" | 33  |
	| 6   | "executive"  | "M" | 42  |
{% endhighlight %}

#### Python
{% highlight python %}
users[['occupation', 'sex', 'age']].head()
{% endhighlight %}

#### R
{% highlight r %}
head(users[,c('occupation', 'sex', 'age')])
{% endhighlight %}

### Query / Conditional subset

Subsetting a dataframe based on querying a column for a condition can be achieved like this -

#### Julia
{% highlight julia %}
users[users[:occupation] .== "writer", :]
	45x5 DataFrame
	| Row | user_id | age | sex | occupation | zip_code |
	|-----|---------|-----|-----|------------|----------|
	| 1   | 3       | 23  | "M" | "writer"   | "32067"  |
	| 2   | 21      | 26  | "M" | "writer"   | "30068"  |
	| 3   | 22      | 25  | "M" | "writer"   | "40206"  |
	| 4   | 28      | 32  | "M" | "writer"   | "55369"  |
	| 5   | 50      | 21  | "M" | "writer"   | "52245"  |
	⋮
	| 43  | 853     | 49  | "M" | "writer"   | "40515"  |
	| 44  | 896     | 28  | "M" | "writer"   | "91505"  |
	| 45  | 911     | 37  | "F" | "writer"   | "53210"  |
{% endhighlight %}

#### Python
{% highlight python %}
users[users.occupation == 'writer']
{% endhighlight %}

There is another way in python, by using the query construct of the dataframe. 
{% highlight python %}
users.query('occupation=="writer"')
{% endhighlight %}

#### R
{% highlight r %}
users[users$occupation == 'writer',]
{% endhighlight %}

Notice the subtle changes in all these examples. For example in the last query subsetting, for python you dont need to specify "select all columns" by adding a ' ," ' as you do in Julia or a ', ' as you do in R.

I will try to write a followup post on Joins on Dataframes in these 3. 

Discuss this post on [hacker news](https://news.ycombinator.com/item?id=8794276).
