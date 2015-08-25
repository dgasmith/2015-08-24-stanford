
# An Introduction to Pandas

When dealing with numeric matrices and vectors in Python, [NumPy](http://www.numpy.org/) makes life a lot easier.
For more complex data, however, it leaves a lot to be desired.
If you're used to working with [data frames in R](http://www.r-tutor.com/r-introduction/data-frame), doing data analysis directly with NumPy feels like a step back.

Fortunately, some nice folks have written the [Python Data Analysis Library](http://pandas.pydata.org/) (a.k.a. pandas).
Pandas provides an R-like `DataFrame`, produces high quality plots with [matplotlib](http://matplotlib.org/), and integrates nicely with other libraries that expect NumPy arrays.

In this tutorial, we'll go through the basics of pandas using a year's worth of weather data from [Weather Underground](http://www.wunderground.com/).
Pandas has a **lot** of functionality, so we'll only be able to cover a small fraction of what you can do.
Check out the (very readable) [pandas docs](http://pandas.pydata.org/pandas-docs/stable/) if you want to learn more.

## Getting Started

Installing pandas should be an easy process if you use pip:

```
sudo pip install pandas
```

For more complex scenarios, please see the [installation instructions](http://pandas.pydata.org/pandas-docs/stable/install.html).

OK, let's get started by importing the pandas library.


    import pandas
    %matplotlib inline

Next, let's read in [our data](data/weather_year.csv).
Because it's in a CSV file, we can use pandas' `read_csv` function to pull it directly into a [DataFrame](http://pandas.pydata.org/pandas-docs/stable/dsintro.html#dataframe).


    data = pandas.read_csv("data/weather_year.csv")

We can get a summary of the DataFrame by printing the object.


    data

This gives us a lot of information. First, we can see that there are 366 rows (entries) -- a year and a day's worth of weather. Each column is printed along with however many "non-null" values are present.
We'll talk more about [null (or missing) values in pandas](http://pandas.pydata.org/pandas-docs/stable/missing_data.html) later, but for now we can note that only the "Max Gust SpeedMPH" and "Events" columns have fewer than 366 non-null values.
Lastly, the data types (dtypes) of the columns are printed at the very bottom. We can see that there are 4 `float64`, 16 `int64`, and 3 `object` columns.


    len(data)

Using `len` on a DataFrame will give you the number of rows. You can get the column names using the `columns` property.


    data.columns

Columns can be accessed in two ways. The first is using the DataFrame like a dictionary with string keys:


    data["EDT"]

You can get multiple columns out at the same time by passing in a list of strings.


    data[["EDT", "Mean TemperatureF"]]

The second way to access columns is using the dot syntax. This only works if your column name could also be a Python variable name (i.e., no spaces), and if it doesn't collide with another DataFrame property or function name (e.g., count, sum).


    data.EDT

We'll be mostly using the dot syntax here because you can auto-complete the names in IPython. The first pandas function we'll learn about is `head()`. This gives us the first 5 items in a column (or the first 5 rows in the DataFrame).


    data.EDT.head()

Passing in a number `n` gives us the first `n` items in the column. There is also a corresponding `tail()` method that gives the *last* `n` items or rows.


    data.EDT.head(10)

This also works with the dictionary syntax.


    data["Mean TemperatureF"].head()

## Exercise 1:

How would we get the second to last date (EDT) in the dataset?


    

If the data in the column is numeric, you can use `describe()` to get some stats on it.


    data["Mean TemperatureF"].describe()

## Fun with Columns

The column names in `data` are a little unweildy, so we're going to rename them. This is as easy as assigning a new list of column names to the `columns` property of the DataFrame.


    data.columns = ["date", "max_temp", "mean_temp", "min_temp", "max_dew",
                    "mean_dew", "min_dew", "max_humidity", "mean_humidity",
                    "min_humidity", "max_pressure", "mean_pressure",
                    "min_pressure", "max_visibilty", "mean_visibility",
                    "min_visibility", "max_wind", "mean_wind", "min_wind",
                    "precipitation", "cloud_cover", "events", "wind_dir"]

These should be in the same order as the original columns. Let's take another look at our DataFrame summary.


    data

Now our columns can all be accessed using the dot syntax!


    data.mean_temp.head()

There are lots useful methods on columns, such as `std()` to get the standard deviation. Most of pandas' methods will happily ignore missing values like `NaN`.


    data.mean_temp.std()

Some methods, like `plot()` and `hist()` produce plots using [matplotlib](http://matplotlib.org/). We'll go over plotting in more detail later.


    data.mean_temp.hist()

By the way, many of the column-specific methods also work on the entire DataFrame. Instead of a single number, you'll get a result for each column.


    data.std()

## Exercise 2:

What is the range of temperatures in the dataset?

*Hint: columns have `max()` and `min()` methods.*


    

## Bulk Operations with `apply()`

Methods like `sum()` and `std()` work on entire columns. We can run our own functions across all values in a column (or row) using `apply()`.

To give you an idea of how this works, let's consider the "date" column in our DataFrame (formally "EDT").


    data.date.head()

We can use the `values` property of the column to get a list of values for the column. Inspecting the first value reveals that these are strings with a particular format.


    first_date = data.date.values[0]
    first_date

The `strptime` function from the `datetime` module will make quick work of this date string. There are many [more shortcuts available](http://docs.python.org/2/library/datetime.html#strftime-and-strptime-behavior) for `strptime`.


    # Import the datetime class from the datetime module
    from datetime import datetime
    
    # Convert date string to datetime object
    datetime.strptime(first_date, "%Y-%m-%d")

Using the `apply()` method, which takes a function (**without** the parentheses), we can apply `strptime` to each value in the column. We'll overwrite the string date values with their Python `datetime` equivalents.


    # Define a function to convert strings to dates
    def string_to_date(date_string):
        return datetime.strptime(date_string, "%Y-%m-%d")
    
    # Run the function on every date string and overwrite the column
    data.date = data.date.apply(string_to_date)
    data.date.head()

Let's go one step futher. Each row in our DateFrame represents the weather from a single day. Each row in a DataFrame is associated with an *index*, which is a label that uniquely identifies a row.

Our row indices up to now have been auto-generated by pandas, and are simply integers from 0 to 365. If we use dates instead of integers for our index, we will get some extra benefits from pandas when plotting later on. Overwriting the index is as easy as assigning to the `index` property of the DataFrame.


    data.index = data.date
    data

Now we can quickly look up a row by its date with the `ix[]` property.


    data.ix[datetime(2012, 8, 19)]

With all of the dates in the index now, we no longer need the "date" column. Let's drop it.


    data = data.drop("date", axis=1)
    data.columns

Note that we need to pass in `axis=1` in order to drop a column. For more details, check out the documentation for `drop`.

## Exercise 3:

Print out the cloud cover for each day in May.

*Hint: you can make datetime objects with the `datetime(year, month, day)` function*


    datetime(2012, 5, 1)  # May 1st of 2012


    

## Handing Missing Values

Pandas considers values like `NaN` and `None` to represent missing data. The `pandas.isnull` function can be used to tell whether or not a value is missing.

Let's use `apply()` across all of the columns in our DataFrame to figure out which values are missing.


    empty = data.apply(lambda col: pandas.isnull(col))
    empty

We got back a dataframe (`empty`) with boolean values for all 22 columns and 366 rows. Inspecting the first 10 values of the "events", column we can see that there are some missing values because a `True` was returned from `pandas.isnull`.


    empty.events.head(10)

Looking at the corresponding rows in the original DataFrame reveals that pandas has filled in `NaN` for empty values in the "events" column.


    data.events.head(10)

This isn't exactly what we want. One option is to drop all rows in the DataFrame with missing "events" values.


    data.dropna(subset=["events"])

The DataFrame we get back has only 162 rows, so we can infer that there were 366 - 162 = 204 missing values in the "events" column. Note that this didn't affect `data`; we're just looking at a copy.

Instead of dropping the rows with missing values, let's fill them with empty strings (you'll see why in a moment). This is easily done with the `fillna()` function. We'll go ahead and overwrite the "events" column with empty string missing values instead of `NaN`.


    data.events = data.events.fillna("")
    data.events.head(10)

## Accessing Individual Rows

Sometimes you need to access individual rows in your DataFrame. The `irow()` function lets you grab the ith row from a DataFrame (starting from 0).


    data.irow(0)

Of course, another option is to use the index.


    data.ix[datetime(2013, 1, 1)]

You can iterate over each row in the DataFrame with `iterrows()`. Note that this function returns **both** the index and the row. Also, you must access columns in the row you get back from `iterrows()` with the dictionary syntax.


    num_rain = 0
    for idx, row in data.iterrows():
        if "Rain" in row["events"]:
            num_rain += 1
    
    "Days with rain: {0}".format(num_rain)

## Exercise 4:

Was there any November rain?

*Hint*: check out the `strftime()` function on `datetime` objects and the [documentation](http://docs.python.org/2/library/datetime.html#strftime-and-strptime-behavior)


    d = datetime(2012, 1, 1)
    d.strftime("%B")


    

## Filtering

Most of your time using pandas will likely be devoted to selecting rows of interest from a DataFrame. In addition to strings, the dictionary syntax accepts things like this:


    freezing_days = data[data.max_temp <= 32]
    freezing_days

We get back another DataFrame with fewer rows (21 in this case). This DataFrame can be filtered down even more.


    freezing_days[freezing_days.min_temp >= 20]

Or, using boolean operations, we could apply both filters to the original DataFrame at the same time.


    data[(data.max_temp <= 32) & (data.min_temp >= 20)]

It's important to understand what's really going on underneath with filtering. Let's look at what kind of object we actually get back when creating a filter.


    temp_max = data.max_temp <= 32
    type(temp_max)

This is a pandas `Series` object, which is the one-dimensional equivalent of a DataFrame. Because our DataFrame uses datetime objects for the index, we have a specialized `TimeSeries` object.

What's inside the filter?


    temp_max

Our filter is nothing more than a `Series` with a *boolean value for every item in the index*. When we "run the filter" as so:


    data[temp_max]

pandas lines up the rows of the DataFrame and the filter using the index, and then keeps the rows with a `True` filter value. That's it.

Let's create another filter.


    temp_min = data.min_temp >= 20
    temp_min

Now we can see what the boolean operations are doing. Something like `&` (**not** `and`)...


    temp_min & temp_max

...is just lining up the two filters using the index, performing a boolean AND operation, and returning the result as another `Series`.

We can do other boolean operations too, like OR:


    temp_min | temp_max

Because the result is just another `Series`, we have all of the regular pandas functions at our disposal. The `any()` function returns `True` if any value in the `Series` is `True`.


    temp_both = temp_min & temp_max
    temp_both.any()

Sometimes filters aren't so intuitive. This (sadly) doesn't work:


    try:
        data["Rain" in data.events]
    except:
        pass # "KeyError: no item named False"

We can wrap it up in an `apply()` call fairly easily, though:


    data[data.events.apply(lambda e: "Rain" in e)]

## Exercise 5:

Before starting the exercise, let's convert the precipitation column in the dataset to floating point numbers. It's currently full of strings because of the "T" value, which stands for "trace amount of precipitation."


    data.precipitation.head()

We'll replace "T" with a very small number, and convert the rest of the strings to floats:


    # Convert precipitation to floating point number
    # "T" means "trace of precipitation"
    def precipitation_to_float(precip_str):
        if precip_str == "T":
            return 1e-10  # Very small value
        return float(precip_str)
    
    data.precipitation = data.precipitation.apply(precipitation_to_float)
    data.precipitation.head()

Now on to the exercise.

What was the coldest it ever got when there was no cloud cover and no precipitation?


    

## Grouping

Besides `apply()`, another great DataFrame function is `groupby()`.
It will group a DataFrame by one or more columns, and let you iterate through each group.

As an example, let's group our DataFrame by the "cloud_cover" column (a value ranging from 0 to 8).


    cover_temps = {}
    for cover, cover_data in data.groupby("cloud_cover"):
        cover_temps[cover] = cover_data.mean_temp.mean()  # The mean mean temp!
    cover_temps

When you iterate through the result of `groupby()`, you will get a tuple.
The first item is the column value, and the second item is a filtered DataFrame (where the column equals the first tuple value).

You can group by more than one column as well.
In this case, the first tuple item returned by `groupby()` will itself be a tuple with the value of each column.


    for (cover, events), group_data in data.groupby(["cloud_cover", "events"]):
        print "Cover: {0}, Events: {1}, Count: {2}".format(cover, events, len(group_data))

## Creating New Columns

Weather events in our DataFrame are stored in strings like "Rain-Thunderstorm" to represent that it rained and there was a thunderstorm that day. Let's split them out into boolean "rain", "thunderstorm", etc. columns.

First, let's discover the different kinds of weather events we have with `unique()`.


    data.events.unique()

Looks like we have "Rain", "Thunderstorm", "Fog", and "Snow" events. Creating a new column for each of these event kinds is a piece of cake with the dictionary syntax.


    for event_kind in ["Rain", "Thunderstorm", "Fog", "Snow"]:
        col_name = event_kind.lower()  # Turn "Rain" into "rain", etc.
        data[col_name] = data.events.apply(lambda e: event_kind in e)
    data

Our new columns show up at the bottom. We can access them now with the dot syntax.


    data.rain

We can also do cool things like find out how many `True` values there are (i.e., how many days had rain)...


    data.rain.sum()

...and get all the days that had both rain and snow!


    data[data.rain & data.snow]

## Exercise 6:

Was the mean temperature more variable on days with rain and snow than on days with just rain or just snow?

*Hint: don't forget the `std()` function*


    

## Plotting

We've already seen how the `hist()` function makes generating histograms a snap. Let's look at the `plot()` function now.


    data.max_temp.plot()

That one line of code did a **lot** for us. First, it created a nice looking line plot using the maximum temperature column from our DataFrame. Second, because we used `datetime` objects in our index, pandas labeled the x-axis appropriately.

Pandas is smart too. If we're only looking at a couple of days, the x-axis looks different:


    data.max_temp.tail().plot()

Prefer a bar plot? Pandas has got your covered.


    data.max_temp.tail().plot(kind="bar", rot=10)

The `plot()` function returns a matplotlib `AxesSubPlot` object. You can pass this object into subsequent calls to `plot()` in order to compose plots.

Although `plot()` takes a variety of parameters to customize your plot, users familiar with matplotlib will feel right at home with the `AxesSubPlot` object.


    ax = data.max_temp.plot(title="Min and Max Temperatures")
    data.min_temp.plot(style="red", ax=ax)
    ax.set_ylabel("Temperature (F)")

## Exercise 7:

Add the mean temperature to the previous plot using a green line. Also, add a legend with the `legend()` method of `ax`.

## Getting Data Out

Writing data out in pandas is as easy as getting data in. To save our DataFrame out to a new csv file, we can just do this:


    data.to_csv("data/weather-mod.csv")

Want to make that tab separated instead? No problem.


    data.to_csv("data/weather-mod.tsv", sep="\t")

There's also support for [reading and writing Excel files](http://pandas.pydata.org/pandas-docs/stable/io.html#excel-files), if you need it.

## Miscellanea

We've only covered a small fraction of the pandas library here.
Before I wrap up, however, there are a few miscellaneous tips I'd like to go over.

First, it can be confusing to know when an operation will modify a DataFrame and when it will return a copy to you.
Pandas behavior here is entirely dictated by NumPy, and some situations are unintuitive.

For example, what do you think will happen here?


    for idx, row in data.iterrows():
        row["max_temp"] = 0
    data.max_temp.head()

Contrary to what you might expect, modifying `row` did **not** modify `data`!
This is because `row` is a copy, and does not point back to the original DataFrame.

Here's the right way to do it:


    for idx, row in data.iterrows():
        data.ix[idx, "max_temp"] = 0
    any(data.max_temp != 0)  # Any rows with max_temp not equal to zero?

Just to make you even more confused, this also **doesn't** work:


    for idx, row in data.iterrows():
        data.ix[idx]["max_temp"] = 100
    data.max_temp.head()

When using `apply()`, the default behavior is to go over columns.


    data.apply(lambda c: c.name)

You can make `apply()` go over rows by passing `axis=1`


    data.apply(lambda r: r["max_pressure"] - r["min_pressure"], axis=1)

When you call `drop()`, though, it's flipped. To drop a column, you need to pass `axis=1`


    data.drop("events", axis=1).columns
