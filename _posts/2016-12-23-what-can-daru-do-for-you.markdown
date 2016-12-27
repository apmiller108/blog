---
layout: post
title:  "What Can Daru Do For You?"
date:   2016-12-23 21:38:35 -0500
categories: ruby 
---

At my job, I started work on a client project that was rather number intensive.  We were going to have to perform repetitive calculations over a dataset.  A colleague introduced me to the concept of data frames and the Ruby gem Daru.

### What is Daru?

It’s a data analysis tool that lets us build tabular data sets which we can then manipulate and apply linear calculations.  It's also is a data visualization tool.  It’s worth noting that Daru is quite similar in functionality to the [pandas Python library][pandas].  

You can do alot with Daru, but I’ll just be looking at a very small piece of it here.  See [Resources](#resources) section below for a link to the docs. 

### Vectors and DataFrames

#### Vector
A *Vector* is a one dimensional set of data, like an array.  When I was working with Daru, I liked to think of a *Vector* as a single column from a spreadsheet. In this example, I am mocking 7 day price history of some imaginary product. Note that the *Vector* can be named.  Also, note the numeric index.

{% highlight ruby %}
require 'daru'

prices = Array.new(7) { rand(499..599) }

vector = Daru::Vector.new prices, name: :price_cents
#=>
#<Daru::Vector(7)>
#            price_cents
#          0         505
#          1         544
#          2         552
#          3         501
#          4         534
#          5         516
#          6         541
{% endhighlight %}

#### DataFrame
A *DataFrame* on the other hand is two dimensional, like a spreadsheet.  Expanding
on the example above, we can include a date column.  We instantiate the
*DataFrame* with a hash of arrays.  Each key of the hash is a column name and
the array contains the data.  This is one of several ways to compose a
*DataFrame*.  Another way is to use *Vectors* instead of arrays - *Vector*
indicies are lined up with each other.

{% highlight ruby %}
require 'daru'
require 'date'

prices = Array.new(7) { rand(499..599)}

def dates
  dates = []
  7.times { |index| dates << Date.new(2016, 1, index + 1) }
  dates
end

df = Daru::DataFrame.new(
  { 
    date: dates, 
    price_cents: prices 
  }, 
  name: :product_prices,
  order: [:date, :price_cents]
)
#=>
#<Daru::DataFrame: product_prices (7x2)>
#                 date price_cent
#         0 2016-01-01        501
#         1 2016-01-02        591
#         2 2016-01-03        543
#         3 2016-01-04        558
#         4 2016-01-05        528
#         5 2016-01-06        552
#         6 2016-01-07        558
{% endhighlight %}

### So I have a *DataFrame*...Now what?
We can perform analysis on the data frame like finding the mean, counts, min,
and max.  Also we can find covariance and correlation bewtween *Vectors*. It's
also possible to perform SQL like queries against the data. There is also
filtering and sorting...the list goes on and on.  See the [documentation][daru
docs] for the details.  Here, I'll show a couple of examples that I found
useful.

#### Add a rolling mean column
Here is an example of adding a column to the *DataFrame* that is a rolling mean
calculation on the price column with a lookback of 7.

{% highlight ruby %}

...

df = Daru::DataFrame.new(
  { 
    date: dates, 
    price_cents: prices 
  }, 
  name: :product_prices,
  order: [:date, :price_cents]
)

# Add a column that is a rolling mean of price with a lookback of 7.
df[:r_mean] = df[:price_cents].rolling_mean(7)

p df
#=>
#<Daru::DataFrame: product_prices (14x3)>
#                 date price_cent     r_mean
#         0 2016-01-01        573        nil
#         1 2016-01-02        587        nil
#         2 2016-01-03        511        nil
#         3 2016-01-04        501        nil
#         4 2016-01-05        548        nil
#         5 2016-01-06        562        nil
#         6 2016-01-07        567 549.857142
#         7 2016-01-08        503 539.857142
#         8 2016-01-09        503 527.857142
#         9 2016-01-10        565 535.571428
#        10 2016-01-11        580 546.857142
#        11 2016-01-12        567 549.571428
#        12 2016-01-13        540 546.428571
#        13 2016-01-14        514 538.857142
{% endhighlight %}

#### Do some arithmetic.
What if we want to calculate the price difference with a lookback of 7.  Here is
one way we could solve it with the `lag` method.


{% highlight ruby %}

...

# First we could add a column that is the price_cents column shifted by 7. 
df[:prev_week_price] = df[:price_cents].lag(7)

p df
#=>
#<Daru::DataFrame: product_prices (14x3)>
#                 date price_cent prev_week_
#         0 2016-01-01        515        nil
#         1 2016-01-02        567        nil
#         2 2016-01-03        528        nil
#         3 2016-01-04        593        nil
#         4 2016-01-05        599        nil
#         5 2016-01-06        563        nil
#         6 2016-01-07        520        nil
#         7 2016-01-08        579        515
#         8 2016-01-09        551        567
#         9 2016-01-10        575        528
#        10 2016-01-11        541        593
#        11 2016-01-12        529        599
#        12 2016-01-13        594        563
#        13 2016-01-14        573        520

# Now we can calculate the difference between values for each index.
df[:weekly_change] = df[:price_cents] - df[:prev_week_price]

p df
#=>
#<Daru::DataFrame: product_prices (14x4)>
#                 date price_cent prev_week_ weekly_cha
#         0 2016-01-01        590        nil        nil
#         1 2016-01-02        515        nil        nil
#         2 2016-01-03        507        nil        nil
#         3 2016-01-04        591        nil        nil
#         4 2016-01-05        556        nil        nil
#         5 2016-01-06        534        nil        nil
#         6 2016-01-07        524        nil        nil
#         7 2016-01-08        562        590        -28
#         8 2016-01-09        544        515         29
#         9 2016-01-10        553        507         46
#        10 2016-01-11        577        591        -14
#        11 2016-01-12        530        556        -26
#        12 2016-01-13        502        534        -32
#        13 2016-01-14        511        524        -13

{% endhighlight %}

#### Join *DataFrames*
Let's say we want to compare the prices of two products.  We could join two
*DateFrames* together à la SQL.

{% highlight ruby %}

...

# Product 'a' prices
df_a = Daru::DataFrame.new(
  { 
    date: dates, 
    price_a: prices_a
  }, 
  name: :product_a_prices,
)

# Product 'b' prices
df_b = Daru::DataFrame.new(
  { 
    date: dates, 
    price_b: prices_b
  }, 
  name: :product_b_prices,
)

# We can join both DataFrames on the :date Vector
df_joins = df_a.join(df_b, on: [:date], how: :left)

#=>
#<Daru::DataFrame(14x3)>
#              price_a       date    price_b
#         0        541 2016-01-01        566
#         1        570 2016-01-02        578
#         2        561 2016-01-03        541
#         3        537 2016-01-04        535
#         4        594 2016-01-05        570
#         5        541 2016-01-06        586
#         6        536 2016-01-07        539
#         7        586 2016-01-08        586
#         8        596 2016-01-09        510
#         9        572 2016-01-10        565
#        10        536 2016-01-11        535
#        11        536 2016-01-12        574
#        12        549 2016-01-13        505
#        13        597 2016-01-14        538

{% endhighlight %}

### Conclusion

Daru is a powerful data analysis tool which I have barely scratched the surface. 
Some other things of note:

* Creating *DataFrames* from CSVs or Excel files
* Grouping and aggregating data
* Graceful handling of missing data (nils)
* Pivot tables
* Data visulization

There is much more to this library that what I have shown, so I encourage the
reader to explore more.  I have provided some links below to get started.  

### Resources

* [Daru documentation][daru docs]
* [Daru Github repo][daru repo]
* [Daru vs panda][daru vs panda]

[pandas]: http://pandas.pydata.org/
[daru docs]: http://www.rubydoc.info/gems/daru/0.1.4.1
[daru repo]: https://github.com/v0dro/daru
[daru vs panda]: https://github.com/v0dro/daru/wiki/pandas-vs-daru
