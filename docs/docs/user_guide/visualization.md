# 可视化

在引用matplotlib API时，我们会使用标准的习惯用法:

``` python
In [1]: import matplotlib.pyplot as plt

In [2]: plt.close('all')
```

We provide the basics in pandas to easily create decent looking plots. 我们同时也在pandas中提供基础的功能，帮助我们能够轻松地画出漂亮的图表。
参见 [ecosystem](https://pandas.pydata.org/pandas-docs/stable/ecosystem.html#ecosystem-visualization) 的visualizaiton库，获取更多使用说明。

::: tip 提示

所有的``np.random``调用，都已123456作为随机种子。

:::

## 基础图表: ``plot``

我们在这里展示一些基础的用法，参见 [cookbook](cookbook.html#cookbook-plotting) 获取更多高级用法。

 method on 在Series 和 DataFrame 上使用的``plot`` 方法仅仅是一个简单的
[``plt.plot()``](https://matplotlib.org/api/_as_gen/matplotlib.axes.Axes.plot.html#matplotlib.axes.Axes.plot)的包装:

``` python
In [3]: ts = pd.Series(np.random.randn(1000),
   ...:                index=pd.date_range('1/1/2000', periods=1000))
   ...: 

In [4]: ts = ts.cumsum()

In [5]: ts.plot()
Out[5]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65d8c0ac50>
```

![series_plot_basic](/static/images/series_plot_basic.png)

如果索引中包含有时间，它会调用 [``gcf().autofmt_xdate()``](https://matplotlib.org/api/_as_gen/matplotlib.figure.Figure.html#matplotlib.figure.Figure.autofmt_xdate)去尝试将时间尽量恰当地展示在x轴.

在针对 DataFrame上, [``plot()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.plot.html#pandas.DataFrame.plot) 可以非常简单地将每一列中的数据，附上标签，并绘制在图表上：

``` python
In [6]: df = pd.DataFrame(np.random.randn(1000, 4),
   ...:                   index=ts.index, columns=list('ABCD'))
   ...: 

In [7]: df = df.cumsum()

In [8]: plt.figure();

In [9]: df.plot();
```

![frame_plot_basic](/static/images/frame_plot_basic.png)

在[``plot()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.plot.html#pandas.DataFrame.plot)中使用 *x* 和 *y*关键字，你可以在一个图标上绘制多个列的数据:

``` python
In [10]: df3 = pd.DataFrame(np.random.randn(1000, 2), columns=['B', 'C']).cumsum()

In [11]: df3['A'] = pd.Series(list(range(len(df))))

In [12]: df3.plot(x='A', y='B')
Out[12]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65d97c1668>
```

![df_plot_xy](/static/images/df_plot_xy.png)

::: tip 提示

更多格式化和样式选项，参见[formatting](#visualization-formatting).

:::

## 其他图表

Plotting methods allow for a handful of plot styles other than the
default line plot绘图方法提供除了默认线图以外的很多图表样式。这些方法可以通过使``kind``关键字参数传入 [``plot()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.plot.html#pandas.DataFrame.plot)，他们包括:

- [‘bar’](#visualization-barplot) 或 [‘barh’](#visualization-barplot) 条状图
- [‘hist’](#visualization-hist) 柱状图
- [‘box’](#visualization-box) 箱型图
- [‘kde’](#visualization-kde) 或 [‘density’](#visualization-kde) 密度图
- [‘area’](#visualization-area-plot) 面积图
- [‘scatter’](#visualization-scatter) 散点图
- [‘hexbin’](#visualization-hexbin) 六边形热力图
- [‘pie’](#visualization-pie) 饼状图

例如，条状图可以用下列方法绘制:

``` python
In [13]: plt.figure();

In [14]: df.iloc[5].plot(kind='bar');
```

![bar_plot_ex](/static/images/bar_plot_ex.png)

你也可以使用``DataFrame.plot.``来替代向 ``kind``关键字传递参数的方式来绘制。这样可以使你更快地了解各种绘图方法以及他们所需要的参数:

``` python
In [15]: df = pd.DataFrame()

In [16]: df.plot.<TAB>  # noqa: E225, E999
df.plot.area     df.plot.barh     df.plot.density  df.plot.hist     df.plot.line     df.plot.scatter
df.plot.bar      df.plot.box      df.plot.hexbin   df.plot.kde      df.plot.pie
```

除了这些``kind``(种类),我们还有 [DataFrame.hist()](#visualization-hist),
和[DataFrame.boxplot()](#visualization-box)方法，他们将会使用独立的界面.

最后，还有一些函数[plotting functions](#visualization-tools)在``pandas.plotting``下面。他们接受一个[``Series``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.html#pandas.Series) 或 [``DataFrame``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html#pandas.DataFrame) 作文他们的参数。他们包括：:

- [Scatter Matrix](#visualization-scatter-matrix)
- [Andrews Curves](#visualization-andrews-curves)
- [Parallel Coordinates](#visualization-parallel-coordinates)
- [Lag Plot](#visualization-lag)
- [Autocorrelation Plot](#visualization-autocorrelation)
- [Bootstrap Plot](#visualization-bootstrap)
- [RadViz](#visualization-radviz)

图表也可以添加[errorbars](#visualization-errorbars)
或 [tables](#visualization-table).

### 条状图

对于有标签的，非时序的数据，你有可能会想要使用条状图:

``` python
In [17]: plt.figure();

In [18]: df.iloc[5].plot.bar()
Out[18]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65da446a90>

In [19]: plt.axhline(0, color='k');
```

![bar_plot_ex](/static/images/bar_plot_ex.png)

调用DataFrame的 [``plot.bar()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.plot.bar.html#pandas.DataFrame.plot.bar) 方法，可以生成多个条状图:

``` python
In [20]: df2 = pd.DataFrame(np.random.rand(10, 4), columns=['a', 'b', 'c', 'd'])

In [21]: df2.plot.bar();
```

![bar_plot_multi_ex](/static/images/bar_plot_multi_ex.png)

要想绘制堆积条状图，可以使用 ``stacked=True``参数:

``` python
In [22]: df2.plot.bar(stacked=True);
```

![bar_plot_stacked_ex](/static/images/bar_plot_stacked_ex.png)

想要获得水平条状图，可以使用 ``barh`` 方法:

``` python
In [23]: df2.plot.barh(stacked=True);
```

![barh_plot_stacked_ex](/static/images/barh_plot_stacked_ex.png)

### 柱状图

可以通过 [``DataFrame.plot.hist()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.plot.hist.html#pandas.DataFrame.plot.hist) 和 [``Series.plot.hist()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.plot.hist.html#pandas.Series.plot.hist) 方法来绘制柱状图：.

``` python
In [24]: df4 = pd.DataFrame({'a': np.random.randn(1000) + 1, 'b': np.random.randn(1000),
   ....:                     'c': np.random.randn(1000) - 1}, columns=['a', 'b', 'c'])
   ....: 

In [25]: plt.figure();

In [26]: df4.plot.hist(alpha=0.5)
Out[26]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65da345e48>
```

![hist_new](/static/images/hist_new.png)

柱状图可以堆积，使用``stacked=True``. 每个桶的大小可以通过使用``bins``关键字进行调整.

``` python
In [27]: plt.figure();

In [28]: df4.plot.hist(stacked=True, bins=20)
Out[28]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65da30b9b0>
```

![hist_new_stacked](/static/images/hist_new_stacked.png)

你也可以向``hist``关键字传入其他matplotlib支持的参数.  例如,
水平柱状图和累计柱状图可以通过``orientation='horizontal'``以及 ``cumulative=True``参数设置.

``` python
In [29]: plt.figure();

In [30]: df4['a'].plot.hist(orientation='horizontal', cumulative=True)
Out[30]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65da69fd68>
```

![hist_new_kwargs](/static/images/hist_new_kwargs.png)

参见 [``hist``](https://matplotlib.org/api/_as_gen/matplotlib.axes.Axes.hist.html#matplotlib.axes.Axes.hist) 方法和
[matplotlib hist documentation](http://matplotlib.org/api/pyplot_api.html#matplotlib.pyplot.hist) 方法以获得更多的帮助.

现有的``DataFrame.hist``接口仍然可以使用来绘制柱状图.

``` python
In [31]: plt.figure();

In [32]: df['A'].diff().hist()
Out[32]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65dac9d240>
```

![hist_plot_ex](/static/images/hist_plot_ex.png)

[``DataFrame.hist()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.hist.html#pandas.DataFrame.hist) 可以一次在多个子图中绘制很多列数据的柱状图:

``` python
In [33]: plt.figure()
Out[33]: <Figure size 640x480 with 0 Axes>

In [34]: df.diff().hist(color='k', alpha=0.5, bins=50)
Out[34]: 
array([[<matplotlib.axes._subplots.AxesSubplot object at 0x7f6601550cc0>,
        <matplotlib.axes._subplots.AxesSubplot object at 0x7f66079a9400>],
       [<matplotlib.axes._subplots.AxesSubplot object at 0x7f65f87ac828>,
        <matplotlib.axes._subplots.AxesSubplot object at 0x7f6604bd6b70>]],
      dtype=object)
```

![frame_hist_ex](/static/images/frame_hist_ex.png)

``by`` 关键字可以用来绘制分组的柱状图:

``` python
In [35]: data = pd.Series(np.random.randn(1000))

In [36]: data.hist(by=np.random.randint(0, 4, 1000), figsize=(6, 4))
Out[36]: 
array([[<matplotlib.axes._subplots.AxesSubplot object at 0x7f6601550ef0>,
        <matplotlib.axes._subplots.AxesSubplot object at 0x7f65d9b82438>],
       [<matplotlib.axes._subplots.AxesSubplot object at 0x7f65dc30ba58>,
        <matplotlib.axes._subplots.AxesSubplot object at 0x7f65f63c2320>]],
      dtype=object)
```

![grouped_hist](/static/images/grouped_hist.png)

### 箱状图

绘制箱状图时，可以使用 [``Series.plot.box()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.plot.box.html#pandas.Series.plot.box) 和 [``DataFrame.plot.box()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.plot.box.html#pandas.DataFrame.plot.box),
或 [``DataFrame.boxplot()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.boxplot.html#pandas.DataFrame.boxplot) 来可视化地展示每一个列中数据的分布.

例如，这里有一个箱状图，展示了一个均匀分布[0,1)的五次实验，每次10个观察的数据图.

``` python
In [37]: df = pd.DataFrame(np.random.rand(10, 5), columns=['A', 'B', 'C', 'D', 'E'])

In [38]: df.plot.box()
Out[38]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65da17f898>
```

![box_plot_new](/static/images/box_plot_new.png)

箱状图可以通过设置 ``color``关键字来改变颜色。你可以传入一个 ``dict``，他的键分别为
``boxes``, ``whiskers``, ``medians`` 和 ``caps``来改变颜色。
如果某个键缺失，那么对应的部分就会使用默认颜色。并且，箱状图还有一个 ``sym``关键字，用于设定异常值的类型。

当你通过 ``color``关键字传递其他类型的参数时，他们将会直接送到matplotlib的全部 ``boxes``, ``whiskers``, ``medians`` 和 ``caps``中，用于色彩的设置.

所有的配色将会用于每一个箱。如果你希望使用个复杂的配色，你可以使用[return_type](#visualization-box-return).

``` python
In [39]: color = {'boxes': 'DarkGreen', 'whiskers': 'DarkOrange',
   ....:          'medians': 'DarkBlue', 'caps': 'Gray'}
   ....: 

In [40]: df.plot.box(color=color, sym='r+')
Out[40]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65da880b00>
```

![box_new_colorize](/static/images/box_new_colorize.png)

另外，你也可以传入其他matplotlib的 ``boxplot``方法所支持的参数。例如，水平和自定义位置的箱型图可以通过``vert=False`` 和 ``positions`` 关键字设置。

``` python
In [41]: df.plot.box(vert=False, positions=[1, 4, 5, 6, 8])
Out[41]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65db18ffd0>
```

![box_new_kwargs](/static/images/box_new_kwargs.png)

参见 [``boxplot``](https://matplotlib.org/api/_as_gen/matplotlib.axes.Axes.boxplot.html#matplotlib.axes.Axes.boxplot) 方法和[matplotlib boxplot documentation](http://matplotlib.org/api/pyplot_api.html#matplotlib.pyplot.boxplot) 获取更多信息.

现有的``DataFrame.boxplot``方法仍然可以使用.

``` python
In [42]: df = pd.DataFrame(np.random.rand(10, 5))

In [43]: plt.figure();

In [44]: bp = df.boxplot()
```

![box_plot_ex](/static/images/box_plot_ex.png)

你可以使用 ``by``关键字来生成组，并绘制分层箱型图。例如：

``` python
In [45]: df = pd.DataFrame(np.random.rand(10, 2), columns=['Col1', 'Col2'])

In [46]: df['X'] = pd.Series(['A', 'A', 'A', 'A', 'A', 'B', 'B', 'B', 'B', 'B'])

In [47]: plt.figure();

In [48]: bp = df.boxplot(by='X')
```

![box_plot_ex2](/static/images/box_plot_ex2.png)

你也可以截取列的一部分或者多列的分组来绘图：

``` python
In [49]: df = pd.DataFrame(np.random.rand(10, 3), columns=['Col1', 'Col2', 'Col3'])

In [50]: df['X'] = pd.Series(['A', 'A', 'A', 'A', 'A', 'B', 'B', 'B', 'B', 'B'])

In [51]: df['Y'] = pd.Series(['A', 'B', 'A', 'B', 'A', 'B', 'A', 'B', 'A', 'B'])

In [52]: plt.figure();

In [53]: bp = df.boxplot(column=['Col1', 'Col2'], by=['X', 'Y'])
```

![box_plot_ex3](/static/images/box_plot_ex3.png)

::: danger 警告

从v0.19.0起，默认从``'dict'`` 更改为 ``'axes'``。

:::

在``boxplot``中,返回值的类型可以通过``return_type``关键字来更改。可以使用的类型包括 ``{"axes", "dict", "both", None}``.

通过使用``DataFrame.boxplot`` 的``by``关键字创建的分面（一页多图），也会影响输出的类型：

return_type= | Faceted | Output type
---|---|---
None | No | axes
None | Yes | 2-D ndarray of axes
'axes' | No | axes
'axes' | Yes | Series of axes
'dict' | No | dict of artists
'dict' | Yes | Series of dicts of artists
'both' | No | namedtuple
'both' | Yes | Series of namedtuples

``Groupby.boxplot`` 永远都会返回 ``return_type``的一个 ``Series`` .

``` python
In [54]: np.random.seed(1234)

In [55]: df_box = pd.DataFrame(np.random.randn(50, 2))

In [56]: df_box['g'] = np.random.choice(['A', 'B'], size=50)

In [57]: df_box.loc[df_box['g'] == 'B', 1] += 3

In [58]: bp = df_box.boxplot(by='g')
```

![boxplot_groupby](/static/images/boxplot_groupby.png)

上面的子图，首先按照数字列分割，然后再按照``g``列分割。下面的子图则是首先按照``g``列分割，然后再按照数字列分割。

``` python
In [59]: bp = df_box.groupby('g').boxplot()
```

![groupby_boxplot_vis](/static/images/groupby_boxplot_vis.png)

### 面积图

你可以通过使用 [``Series.plot.area()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.plot.area.html#pandas.Series.plot.area) 和 [``DataFrame.plot.area()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.plot.area.html#pandas.DataFrame.plot.area)来建立面积图.
面积图默认是层积的。要想绘制层积面积图，每个列中的值都必须全部为正或者全部为负。

如果输入的数据中包含*NaN*, 它们将会自动以0填充。如果你想剔除，或者填充其他数据，请在绘图前使用``dataframe.dropna()`` 或 ``dataframe.fillna()`` 处理你的数据。

``` python
In [60]: df = pd.DataFrame(np.random.rand(10, 4), columns=['a', 'b', 'c', 'd'])

In [61]: df.plot.area();
```

![area_plot_stacked](/static/images/area_plot_stacked.png)

如果要生成非层积图，请传入 ``stacked=False``。

``` python
In [62]: df.plot.area(stacked=False);
```

![area_plot_unstacked](/static/images/area_plot_unstacked.png)

### 散点图

使用[``DataFrame.plot.scatter()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.plot.scatter.html#pandas.DataFrame.plot.scatter)方法，生成散点图。

散点图需要数值型的列来作为x和y的值。他们可以通过 ``x`` 和``y`` 关键字来输入。

``` python
In [63]: df = pd.DataFrame(np.random.rand(50, 4), columns=['a', 'b', 'c', 'd'])

In [64]: df.plot.scatter(x='a', y='b');
```

![scatter_plot](/static/images/scatter_plot.png)

To plot multiple column groups in a single axes, repeat ``plot`` method specifying target ``ax``.
It is recommended to specify ``color`` and ``label`` keywords to distinguish each groups.

要想在同一组坐标系下绘制多组数据（来自多组列），需要重复使用 ``plot`` 方法，并且将 ``ax``关键字设置为你想要重复使用的坐标系。强烈建议公式使用 ``color`` 和 ``label`` 关键字来区别每个组的数据。

``` python
In [65]: ax = df.plot.scatter(x='a', y='b', color='DarkBlue', label='Group 1');

In [66]: df.plot.scatter(x='c', y='d', color='DarkGreen', label='Group 2', ax=ax);
```

![scatter_plot_repeated](/static/images/scatter_plot_repeated.png)

关键字 ``c`` 可以传入一个列明，用于指明每一个点的颜色。

``` python
In [67]: df.plot.scatter(x='a', y='b', c='c', s=50);
```

![scatter_plot_colored](/static/images/scatter_plot_colored.png)

你也可以传入其他matplotlib的[``scatter``](https://matplotlib.org/api/_as_gen/matplotlib.axes.Axes.scatter.html#matplotlib.axes.Axes.scatter)所支持的其他参数。下面的例子展示了如何通过一个 ``DataFrame`` 的列来指明气泡大小，从而绘制气泡图。

``` python
In [68]: df.plot.scatter(x='a', y='b', s=df['c'] * 200);
```

![scatter_plot_bubble](/static/images/scatter_plot_bubble.png)

请参见 [``scatter``](https://matplotlib.org/api/_as_gen/matplotlib.axes.Axes.scatter.html#matplotlib.axes.Axes.scatter) 方法，以及[matplotlib scatter documentation](http://matplotlib.org/api/pyplot_api.html#matplotlib.pyplot.scatter)，获得更多信息。

### Hexagonal bin plot

You can create hexagonal bin plots with [``DataFrame.plot.hexbin()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.plot.hexbin.html#pandas.DataFrame.plot.hexbin).
Hexbin plots can be a useful alternative to scatter plots if your data are
too dense to plot each point individually.

``` python
In [69]: df = pd.DataFrame(np.random.randn(1000, 2), columns=['a', 'b'])

In [70]: df['b'] = df['b'] + np.arange(1000)

In [71]: df.plot.hexbin(x='a', y='b', gridsize=25)
Out[71]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65d96e4fd0>
```

![hexbin_plot](/static/images/hexbin_plot.png)

A useful keyword argument is ``gridsize``; it controls the number of hexagons
in the x-direction, and defaults to 100. A larger ``gridsize`` means more, smaller
bins.

By default, a histogram of the counts around each ``(x, y)`` point is computed.
You can specify alternative aggregations by passing values to the ``C`` and
``reduce_C_function`` arguments. ``C`` specifies the value at each ``(x, y)`` point
and ``reduce_C_function`` is a function of one argument that reduces all the
values in a bin to a single number (e.g. ``mean``, ``max``, ``sum``, ``std``).  In this
example the positions are given by columns ``a`` and ``b``, while the value is
given by column ``z``. The bins are aggregated with NumPy’s ``max`` function.

``` python
In [72]: df = pd.DataFrame(np.random.randn(1000, 2), columns=['a', 'b'])

In [73]: df['b'] = df['b'] = df['b'] + np.arange(1000)

In [74]: df['z'] = np.random.uniform(0, 3, 1000)

In [75]: df.plot.hexbin(x='a', y='b', C='z', reduce_C_function=np.max, gridsize=25)
Out[75]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65d98ea390>
```

![hexbin_plot_agg](/static/images/hexbin_plot_agg.png)

See the [``hexbin``](https://matplotlib.org/api/_as_gen/matplotlib.axes.Axes.hexbin.html#matplotlib.axes.Axes.hexbin) method and the
[matplotlib hexbin documentation](http://matplotlib.org/api/pyplot_api.html#matplotlib.pyplot.hexbin) for more.

### Pie plot

You can create a pie plot with [``DataFrame.plot.pie()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.plot.pie.html#pandas.DataFrame.plot.pie) or [``Series.plot.pie()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.plot.pie.html#pandas.Series.plot.pie).
If your data includes any ``NaN``, they will be automatically filled with 0.
A ``ValueError`` will be raised if there are any negative values in your data.

``` python
In [76]: series = pd.Series(3 * np.random.rand(4),
   ....:                    index=['a', 'b', 'c', 'd'], name='series')
   ....: 

In [77]: series.plot.pie(figsize=(6, 6))
Out[77]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65da5ff278>
```

![series_pie_plot](/static/images/series_pie_plot.png)

For pie plots it’s best to use square figures, i.e. a figure aspect ratio 1.
You can create the figure with equal width and height, or force the aspect ratio
to be equal after plotting by calling ``ax.set_aspect('equal')`` on the returned
``axes`` object.

Note that pie plot with [``DataFrame``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html#pandas.DataFrame) requires that you either specify a
target column by the ``y`` argument or ``subplots=True``. When ``y`` is
specified, pie plot of selected column will be drawn. If ``subplots=True`` is
specified, pie plots for each column are drawn as subplots. A legend will be
drawn in each pie plots by default; specify ``legend=False`` to hide it.

``` python
In [78]: df = pd.DataFrame(3 * np.random.rand(4, 2),
   ....:                   index=['a', 'b', 'c', 'd'], columns=['x', 'y'])
   ....: 

In [79]: df.plot.pie(subplots=True, figsize=(8, 4))
Out[79]: 
array([<matplotlib.axes._subplots.AxesSubplot object at 0x7f65d915b0b8>,
       <matplotlib.axes._subplots.AxesSubplot object at 0x7f65d9493a90>],
      dtype=object)
```

![df_pie_plot](/static/images/df_pie_plot.png)

You can use the ``labels`` and ``colors`` keywords to specify the labels and colors of each wedge.

::: danger Warning

Most pandas plots use the ``label`` and ``color`` arguments (note the lack of “s” on those).
To be consistent with [``matplotlib.pyplot.pie()``](https://matplotlib.org/api/_as_gen/matplotlib.pyplot.pie.html#matplotlib.pyplot.pie) you must use ``labels`` and ``colors``.

:::

If you want to hide wedge labels, specify ``labels=None``.
If ``fontsize`` is specified, the value will be applied to wedge labels.
Also, other keywords supported by [``matplotlib.pyplot.pie()``](https://matplotlib.org/api/_as_gen/matplotlib.pyplot.pie.html#matplotlib.pyplot.pie) can be used.

``` python
In [80]: series.plot.pie(labels=['AA', 'BB', 'CC', 'DD'], colors=['r', 'g', 'b', 'c'],
   ....:                 autopct='%.2f', fontsize=20, figsize=(6, 6))
   ....: 
Out[80]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65da0be4a8>
```

![series_pie_plot_options](/static/images/series_pie_plot_options.png)

If you pass values whose sum total is less than 1.0, matplotlib draws a semicircle.

``` python
In [81]: series = pd.Series([0.1] * 4, index=['a', 'b', 'c', 'd'], name='series2')

In [82]: series.plot.pie(figsize=(6, 6))
Out[82]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65d92543c8>
```

![series_pie_plot_semi](/static/images/series_pie_plot_semi.png)

See the [matplotlib pie documentation](http://matplotlib.org/api/pyplot_api.html#matplotlib.pyplot.pie) for more.

## Plotting with missing data

Pandas tries to be pragmatic about plotting ``DataFrames`` or ``Series``
that contain missing data. Missing values are dropped, left out, or filled
depending on the plot type.

Plot Type | NaN Handling
---|---
Line | Leave gaps at NaNs
Line (stacked) | Fill 0’s
Bar | Fill 0’s
Scatter | Drop NaNs
Histogram | Drop NaNs (column-wise)
Box | Drop NaNs (column-wise)
Area | Fill 0’s
KDE | Drop NaNs (column-wise)
Hexbin | Drop NaNs
Pie | Fill 0’s

If any of these defaults are not what you want, or if you want to be
explicit about how missing values are handled, consider using
[``fillna()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.fillna.html#pandas.DataFrame.fillna) or [``dropna()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.dropna.html#pandas.DataFrame.dropna)
before plotting.

## Plotting Tools

These functions can be imported from ``pandas.plotting``
and take a [``Series``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.html#pandas.Series) or [``DataFrame``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html#pandas.DataFrame) as an argument.

### Scatter matrix plot

You can create a scatter plot matrix using the
``scatter_matrix`` method in ``pandas.plotting``:

``` python
In [83]: from pandas.plotting import scatter_matrix

In [84]: df = pd.DataFrame(np.random.randn(1000, 4), columns=['a', 'b', 'c', 'd'])

In [85]: scatter_matrix(df, alpha=0.2, figsize=(6, 6), diagonal='kde')
Out[85]: 
array([[<matplotlib.axes._subplots.AxesSubplot object at 0x7f65dc209da0>,
        <matplotlib.axes._subplots.AxesSubplot object at 0x7f65d9df9588>,
        <matplotlib.axes._subplots.AxesSubplot object at 0x7f65d8fb4b38>,
        <matplotlib.axes._subplots.AxesSubplot object at 0x7f65d9834128>],
       [<matplotlib.axes._subplots.AxesSubplot object at 0x7f65db04d6d8>,
        <matplotlib.axes._subplots.AxesSubplot object at 0x7f65d8cdcc88>,
        <matplotlib.axes._subplots.AxesSubplot object at 0x7f65d94e8278>,
        <matplotlib.axes._subplots.AxesSubplot object at 0x7f65d9a67860>],
       [<matplotlib.axes._subplots.AxesSubplot object at 0x7f65d9a67898>,
        <matplotlib.axes._subplots.AxesSubplot object at 0x7f65da9f43c8>,
        <matplotlib.axes._subplots.AxesSubplot object at 0x7f65dacb7978>,
        <matplotlib.axes._subplots.AxesSubplot object at 0x7f65daddaf28>],
       [<matplotlib.axes._subplots.AxesSubplot object at 0x7f65dbe47518>,
        <matplotlib.axes._subplots.AxesSubplot object at 0x7f65d9016ac8>,
        <matplotlib.axes._subplots.AxesSubplot object at 0x7f65d99540b8>,
        <matplotlib.axes._subplots.AxesSubplot object at 0x7f65d9f84668>]],
      dtype=object)
```

![scatter_matrix_kde](/static/images/scatter_matrix_kde.png)

### Density plot

You can create density plots using the [``Series.plot.kde()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.plot.kde.html#pandas.Series.plot.kde) and [``DataFrame.plot.kde()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.plot.kde.html#pandas.DataFrame.plot.kde) methods.

``` python
In [86]: ser = pd.Series(np.random.randn(1000))

In [87]: ser.plot.kde()
Out[87]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65d909c828>
```

![kde_plot](/static/images/kde_plot.png)

### Andrews curves

Andrews curves allow one to plot multivariate data as a large number
of curves that are created using the attributes of samples as coefficients
for Fourier series, see the [Wikipedia entry](https://en.wikipedia.org/wiki/Andrews_plot)
for more information. By coloring these curves differently for each class
it is possible to visualize data clustering. Curves belonging to samples
of the same class will usually be closer together and form larger structures.

**Note**: The “Iris” dataset is available [here](https://raw.github.com/pandas-dev/pandas/master/pandas/tests/data/iris.csv).

``` python
In [88]: from pandas.plotting import andrews_curves

In [89]: data = pd.read_csv('data/iris.data')

In [90]: plt.figure()
Out[90]: <Figure size 640x480 with 0 Axes>

In [91]: andrews_curves(data, 'Name')
Out[91]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65da6e5518>
```

![andrews_curves](/static/images/andrews_curves.png)

### Parallel coordinates

Parallel coordinates is a plotting technique for plotting multivariate data,
see the [Wikipedia entry](https://en.wikipedia.org/wiki/Parallel_coordinates)
for an introduction.
Parallel coordinates allows one to see clusters in data and to estimate other statistics visually.
Using parallel coordinates points are represented as connected line segments.
Each vertical line represents one attribute. One set of connected line segments
represents one data point. Points that tend to cluster will appear closer together.

``` python
In [92]: from pandas.plotting import parallel_coordinates

In [93]: data = pd.read_csv('data/iris.data')

In [94]: plt.figure()
Out[94]: <Figure size 640x480 with 0 Axes>

In [95]: parallel_coordinates(data, 'Name')
Out[95]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65d96fbc88>
```

![parallel_coordinates](/static/images/parallel_coordinates.png)

### Lag plot

Lag plots are used to check if a data set or time series is random. Random
data should not exhibit any structure in the lag plot. Non-random structure
implies that the underlying data are not random. The ``lag`` argument may
be passed, and when ``lag=1`` the plot is essentially ``data[:-1]`` vs.
``data[1:]``.

``` python
In [96]: from pandas.plotting import lag_plot

In [97]: plt.figure()
Out[97]: <Figure size 640x480 with 0 Axes>

In [98]: spacing = np.linspace(-99 * np.pi, 99 * np.pi, num=1000)

In [99]: data = pd.Series(0.1 * np.random.rand(1000) + 0.9 * np.sin(spacing))

In [100]: lag_plot(data)
Out[100]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65da8b5e10>
```

![lag_plot](/static/images/lag_plot.png)

### Autocorrelation plot

Autocorrelation plots are often used for checking randomness in time series.
This is done by computing autocorrelations for data values at varying time lags.
If time series is random, such autocorrelations should be near zero for any and
all time-lag separations. If time series is non-random then one or more of the
autocorrelations will be significantly non-zero. The horizontal lines displayed
in the plot correspond to 95% and 99% confidence bands. The dashed line is 99%
confidence band. See the
[Wikipedia entry](https://en.wikipedia.org/wiki/Correlogram) for more about
autocorrelation plots.

``` python
In [101]: from pandas.plotting import autocorrelation_plot

In [102]: plt.figure()
Out[102]: <Figure size 640x480 with 0 Axes>

In [103]: spacing = np.linspace(-9 * np.pi, 9 * np.pi, num=1000)

In [104]: data = pd.Series(0.7 * np.random.rand(1000) + 0.3 * np.sin(spacing))

In [105]: autocorrelation_plot(data)
Out[105]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65d19556d8>
```

![autocorrelation_plot](/static/images/autocorrelation_plot.png)

### Bootstrap plot

Bootstrap plots are used to visually assess the uncertainty of a statistic, such
as mean, median, midrange, etc. A random subset of a specified size is selected
from a data set, the statistic in question is computed for this subset and the
process is repeated a specified number of times. Resulting plots and histograms
are what constitutes the bootstrap plot.

``` python
In [106]: from pandas.plotting import bootstrap_plot

In [107]: data = pd.Series(np.random.rand(1000))

In [108]: bootstrap_plot(data, size=50, samples=500, color='grey')
Out[108]: <Figure size 640x480 with 6 Axes>
```

![bootstrap_plot](/static/images/bootstrap_plot.png)

### RadViz

RadViz is a way of visualizing multi-variate data. It is based on a simple
spring tension minimization algorithm. Basically you set up a bunch of points in
a plane. In our case they are equally spaced on a unit circle. Each point
represents a single attribute. You then pretend that each sample in the data set
is attached to each of these points by a spring, the stiffness of which is
proportional to the numerical value of that attribute (they are normalized to
unit interval). The point in the plane, where our sample settles to (where the
forces acting on our sample are at an equilibrium) is where a dot representing
our sample will be drawn. Depending on which class that sample belongs it will
be colored differently.
See the R package [Radviz](https://cran.r-project.org/package=Radviz/)
for more information.

**Note**: The “Iris” dataset is available [here](https://raw.github.com/pandas-dev/pandas/master/pandas/tests/data/iris.csv).

``` python
In [109]: from pandas.plotting import radviz

In [110]: data = pd.read_csv('data/iris.data')

In [111]: plt.figure()
Out[111]: <Figure size 640x480 with 0 Axes>

In [112]: radviz(data, 'Name')
Out[112]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65f66bd630>
```

![radviz](/static/images/radviz.png)

## Plot Formatting

### Setting the plot style

From version 1.5 and up, matplotlib offers a range of pre-configured plotting styles. Setting the
style can be used to easily give plots the general look that you want.
Setting the style is as easy as calling ``matplotlib.style.use(my_plot_style)`` before
creating your plot. For example you could write ``matplotlib.style.use('ggplot')`` for ggplot-style
plots.

You can see the various available style names at ``matplotlib.style.available`` and it’s very
easy to try them out.

### General plot style arguments

Most plotting methods have a set of keyword arguments that control the
layout and formatting of the returned plot:

``` python
In [113]: plt.figure();

In [114]: ts.plot(style='k--', label='Series');
```

![series_plot_basic2](/static/images/series_plot_basic2.png)

For each kind of plot (e.g. *line*, *bar*, *scatter*) any additional arguments
keywords are passed along to the corresponding matplotlib function
([``ax.plot()``](https://matplotlib.org/api/_as_gen/matplotlib.axes.Axes.plot.html#matplotlib.axes.Axes.plot),
[``ax.bar()``](https://matplotlib.org/api/_as_gen/matplotlib.axes.Axes.bar.html#matplotlib.axes.Axes.bar),
[``ax.scatter()``](https://matplotlib.org/api/_as_gen/matplotlib.axes.Axes.scatter.html#matplotlib.axes.Axes.scatter)). These can be used
to control additional styling, beyond what pandas provides.

### Controlling the legend

You may set the ``legend`` argument to ``False`` to hide the legend, which is
shown by default.

``` python
In [115]: df = pd.DataFrame(np.random.randn(1000, 4),
   .....:                   index=ts.index, columns=list('ABCD'))
   .....: 

In [116]: df = df.cumsum()

In [117]: df.plot(legend=False)
Out[117]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65dbdbc0f0>
```

![frame_plot_basic_noleg](/static/images/frame_plot_basic_noleg.png)

### Scales

You may pass ``logy`` to get a log-scale Y axis.

``` python
In [118]: ts = pd.Series(np.random.randn(1000),
   .....:                index=pd.date_range('1/1/2000', periods=1000))
   .....: 

In [119]: ts = np.exp(ts.cumsum())

In [120]: ts.plot(logy=True)
Out[120]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65dbefdf98>
```

![series_plot_logy](/static/images/series_plot_logy.png)

See also the ``logx`` and ``loglog`` keyword arguments.

### Plotting on a secondary y-axis

To plot data on a secondary y-axis, use the ``secondary_y`` keyword:

``` python
In [121]: df.A.plot()
Out[121]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65f8ef6b00>

In [122]: df.B.plot(secondary_y=True, style='g')
Out[122]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65f6297780>
```

![series_plot_secondary_y](/static/images/series_plot_secondary_y.png)

To plot some columns in a ``DataFrame``, give the column names to the ``secondary_y``
keyword:

``` python
In [123]: plt.figure()
Out[123]: <Figure size 640x480 with 0 Axes>

In [124]: ax = df.plot(secondary_y=['A', 'B'])

In [125]: ax.set_ylabel('CD scale')
Out[125]: Text(0, 0.5, 'CD scale')

In [126]: ax.right_ax.set_ylabel('AB scale')
Out[126]: Text(0, 0.5, 'AB scale')
```

![frame_plot_secondary_y](/static/images/frame_plot_secondary_y.png)

Note that the columns plotted on the secondary y-axis is automatically marked
with “(right)” in the legend. To turn off the automatic marking, use the
``mark_right=False`` keyword:

``` python
In [127]: plt.figure()
Out[127]: <Figure size 640x480 with 0 Axes>

In [128]: df.plot(secondary_y=['A', 'B'], mark_right=False)
Out[128]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65f6102390>
```

![frame_plot_secondary_y_no_right](/static/images/frame_plot_secondary_y_no_right.png)

### Suppressing tick resolution adjustment

pandas includes automatic tick resolution adjustment for regular frequency
time-series data. For limited cases where pandas cannot infer the frequency
information (e.g., in an externally created ``twinx``), you can choose to
suppress this behavior for alignment purposes.

Here is the default behavior, notice how the x-axis tick labeling is performed:

``` python
In [129]: plt.figure()
Out[129]: <Figure size 640x480 with 0 Axes>

In [130]: df.A.plot()
Out[130]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65dc39f978>
```

![ser_plot_suppress](/static/images/ser_plot_suppress.png)

Using the ``x_compat`` parameter, you can suppress this behavior:

``` python
In [131]: plt.figure()
Out[131]: <Figure size 640x480 with 0 Axes>

In [132]: df.A.plot(x_compat=True)
Out[132]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65dc39f1d0>
```

![ser_plot_suppress_parm](/static/images/ser_plot_suppress_parm.png)

If you have more than one plot that needs to be suppressed, the ``use`` method
in ``pandas.plotting.plot_params`` can be used in a *with statement*:

``` python
In [133]: plt.figure()
Out[133]: <Figure size 640x480 with 0 Axes>

In [134]: with pd.plotting.plot_params.use('x_compat', True):
   .....:     df.A.plot(color='r')
   .....:     df.B.plot(color='g')
   .....:     df.C.plot(color='b')
   .....:
```

![ser_plot_suppress_context](/static/images/ser_plot_suppress_context.png)

### Automatic date tick adjustment

*New in version 0.20.0.* 

``TimedeltaIndex`` now uses the native matplotlib
tick locator methods, it is useful to call the automatic
date tick adjustment from matplotlib for figures whose ticklabels overlap.

See the ``autofmt_xdate`` method and the
[matplotlib documentation](http://matplotlib.org/users/recipes.html#fixing-common-date-annoyances) for more.

### Subplots

Each ``Series`` in a ``DataFrame`` can be plotted on a different axis
with the ``subplots`` keyword:

``` python
In [135]: df.plot(subplots=True, figsize=(6, 6));
```

![frame_plot_subplots](/static/images/frame_plot_subplots.png)

### Using layout and targeting multiple axes

The layout of subplots can be specified by the ``layout`` keyword. It can accept
``(rows, columns)``. The ``layout`` keyword can be used in
``hist`` and ``boxplot`` also. If the input is invalid, a ``ValueError`` will be raised.

The number of axes which can be contained by rows x columns specified by ``layout`` must be
larger than the number of required subplots. If layout can contain more axes than required,
blank axes are not drawn. Similar to a NumPy array’s ``reshape`` method, you
can use ``-1`` for one dimension to automatically calculate the number of rows
or columns needed, given the other.

``` python
In [136]: df.plot(subplots=True, layout=(2, 3), figsize=(6, 6), sharex=False);
```

![frame_plot_subplots_layout](/static/images/frame_plot_subplots_layout.png)

The above example is identical to using:

``` python
In [137]: df.plot(subplots=True, layout=(2, -1), figsize=(6, 6), sharex=False);
```

The required number of columns (3) is inferred from the number of series to plot
and the given number of rows (2).

You can pass multiple axes created beforehand as list-like via ``ax`` keyword.
This allows more complicated layouts.
The passed axes must be the same number as the subplots being drawn.

When multiple axes are passed via the ``ax`` keyword, ``layout``, ``sharex`` and ``sharey`` keywords
don’t affect to the output. You should explicitly pass ``sharex=False`` and ``sharey=False``,
otherwise you will see a warning.

``` python
In [138]: fig, axes = plt.subplots(4, 4, figsize=(6, 6))

In [139]: plt.subplots_adjust(wspace=0.5, hspace=0.5)

In [140]: target1 = [axes[0][0], axes[1][1], axes[2][2], axes[3][3]]

In [141]: target2 = [axes[3][0], axes[2][1], axes[1][2], axes[0][3]]

In [142]: df.plot(subplots=True, ax=target1, legend=False, sharex=False, sharey=False);

In [143]: (-df).plot(subplots=True, ax=target2, legend=False,
   .....:            sharex=False, sharey=False);
   .....:
```

![frame_plot_subplots_multi_ax](/static/images/frame_plot_subplots_multi_ax.png)

Another option is passing an ``ax`` argument to [``Series.plot()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.plot.html#pandas.Series.plot) to plot on a particular axis:

``` python
In [144]: fig, axes = plt.subplots(nrows=2, ncols=2)

In [145]: df['A'].plot(ax=axes[0, 0]);

In [146]: axes[0, 0].set_title('A');

In [147]: df['B'].plot(ax=axes[0, 1]);

In [148]: axes[0, 1].set_title('B');

In [149]: df['C'].plot(ax=axes[1, 0]);

In [150]: axes[1, 0].set_title('C');

In [151]: df['D'].plot(ax=axes[1, 1]);

In [152]: axes[1, 1].set_title('D');
```

![series_plot_multi](/static/images/series_plot_multi.png)

### Plotting with error bars

Plotting with error bars is supported in [``DataFrame.plot()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.plot.html#pandas.DataFrame.plot) and [``Series.plot()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.plot.html#pandas.Series.plot).

Horizontal and vertical error bars can be supplied to the ``xerr`` and ``yerr`` keyword arguments to [``plot()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.plot.html#pandas.DataFrame.plot). The error values can be specified using a variety of formats:

- As a [``DataFrame``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html#pandas.DataFrame) or ``dict`` of errors with column names matching the ``columns`` attribute of the plotting [``DataFrame``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html#pandas.DataFrame) or matching the ``name`` attribute of the [``Series``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.html#pandas.Series).
- As a ``str`` indicating which of the columns of plotting [``DataFrame``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html#pandas.DataFrame) contain the error values.
- As raw values (``list``, ``tuple``, or ``np.ndarray``). Must be the same length as the plotting [``DataFrame``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html#pandas.DataFrame)/[``Series``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.html#pandas.Series).

Asymmetrical error bars are also supported, however raw error values must be provided in this case. For a ``M`` length [``Series``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.html#pandas.Series), a ``Mx2`` array should be provided indicating lower and upper (or left and right) errors. For a ``MxN`` [``DataFrame``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html#pandas.DataFrame), asymmetrical errors should be in a ``Mx2xN`` array.

Here is an example of one way to easily plot group means with standard deviations from the raw data.

``` python
# Generate the data
In [153]: ix3 = pd.MultiIndex.from_arrays([
   .....:     ['a', 'a', 'a', 'a', 'b', 'b', 'b', 'b'],
   .....:     ['foo', 'foo', 'bar', 'bar', 'foo', 'foo', 'bar', 'bar']],
   .....:     names=['letter', 'word'])
   .....: 

In [154]: df3 = pd.DataFrame({'data1': [3, 2, 4, 3, 2, 4, 3, 2],
   .....:                     'data2': [6, 5, 7, 5, 4, 5, 6, 5]}, index=ix3)
   .....: 

# Group by index labels and take the means and standard deviations
# for each group
In [155]: gp3 = df3.groupby(level=('letter', 'word'))

In [156]: means = gp3.mean()

In [157]: errors = gp3.std()

In [158]: means
Out[158]: 
             data1  data2
letter word              
a      bar     3.5    6.0
       foo     2.5    5.5
b      bar     2.5    5.5
       foo     3.0    4.5

In [159]: errors
Out[159]: 
                data1     data2
letter word                    
a      bar   0.707107  1.414214
       foo   0.707107  0.707107
b      bar   0.707107  0.707107
       foo   1.414214  0.707107

# Plot
In [160]: fig, ax = plt.subplots()

In [161]: means.plot.bar(yerr=errors, ax=ax, capsize=4)
Out[161]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65d1048400>
```

![errorbar_example](/static/images/errorbar_example.png)

### Plotting tables

Plotting with matplotlib table is now supported in  [``DataFrame.plot()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.plot.html#pandas.DataFrame.plot) and [``Series.plot()``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.plot.html#pandas.Series.plot) with a ``table`` keyword. The ``table`` keyword can accept ``bool``, [``DataFrame``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html#pandas.DataFrame) or [``Series``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.html#pandas.Series). The simple way to draw a table is to specify ``table=True``. Data will be transposed to meet matplotlib’s default layout.

``` python
In [162]: fig, ax = plt.subplots(1, 1)

In [163]: df = pd.DataFrame(np.random.rand(5, 3), columns=['a', 'b', 'c'])

In [164]: ax.get_xaxis().set_visible(False)   # Hide Ticks

In [165]: df.plot(table=True, ax=ax)
Out[165]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65d0ff7550>
```

![line_plot_table_true](/static/images/line_plot_table_true.png)

Also, you can pass a different [``DataFrame``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html#pandas.DataFrame) or [``Series``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.html#pandas.Series) to the
``table`` keyword. The data will be drawn as displayed in print method
(not transposed automatically). If required, it should be transposed manually
as seen in the example below.

``` python
In [166]: fig, ax = plt.subplots(1, 1)

In [167]: ax.get_xaxis().set_visible(False)   # Hide Ticks

In [168]: df.plot(table=np.round(df.T, 2), ax=ax)
Out[168]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65d0efdcc0>
```

![line_plot_table_data](/static/images/line_plot_table_data.png)

There also exists a helper function ``pandas.plotting.table``, which creates a
table from [``DataFrame``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html#pandas.DataFrame) or [``Series``](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.html#pandas.Series), and adds it to an
``matplotlib.Axes`` instance. This function can accept keywords which the
matplotlib [table](http://matplotlib.org/api/axes_api.html#matplotlib.axes.Axes.table) has.

``` python
In [169]: from pandas.plotting import table

In [170]: fig, ax = plt.subplots(1, 1)

In [171]: table(ax, np.round(df.describe(), 2),
   .....:       loc='upper right', colWidths=[0.2, 0.2, 0.2])
   .....: 
Out[171]: <matplotlib.table.Table at 0x7f65d0e61b38>

In [172]: df.plot(ax=ax, ylim=(0, 2), legend=None)
Out[172]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65d0eab358>
```

![line_plot_table_describe](/static/images/line_plot_table_describe.png)

**Note**: You can get table instances on the axes using ``axes.tables`` property for further decorations. See the [matplotlib table documentation](http://matplotlib.org/api/axes_api.html#matplotlib.axes.Axes.table) for more.

### Colormaps

A potential issue when plotting a large number of columns is that it can be
difficult to distinguish some series due to repetition in the default colors. To
remedy this, ``DataFrame`` plotting supports the use of the ``colormap`` argument,
which accepts either a Matplotlib [colormap](http://matplotlib.org/api/cm_api.html)
or a string that is a name of a colormap registered with Matplotlib. A
visualization of the default matplotlib colormaps is available [here](https://matplotlib.org/examples/color/colormaps_reference.html).

As matplotlib does not directly support colormaps for line-based plots, the
colors are selected based on an even spacing determined by the number of columns
in the ``DataFrame``. There is no consideration made for background color, so some
colormaps will produce lines that are not easily visible.

To use the cubehelix colormap, we can pass ``colormap='cubehelix'``.

``` python
In [173]: df = pd.DataFrame(np.random.randn(1000, 10), index=ts.index)

In [174]: df = df.cumsum()

In [175]: plt.figure()
Out[175]: <Figure size 640x480 with 0 Axes>

In [176]: df.plot(colormap='cubehelix')
Out[176]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65d0defdd8>
```

![cubehelix](/static/images/cubehelix.png)

Alternatively, we can pass the colormap itself:

``` python
In [177]: from matplotlib import cm

In [178]: plt.figure()
Out[178]: <Figure size 640x480 with 0 Axes>

In [179]: df.plot(colormap=cm.cubehelix)
Out[179]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65d0c22a90>
```

![cubehelix_cm](/static/images/cubehelix_cm.png)

Colormaps can also be used other plot types, like bar charts:

``` python
In [180]: dd = pd.DataFrame(np.random.randn(10, 10)).applymap(abs)

In [181]: dd = dd.cumsum()

In [182]: plt.figure()
Out[182]: <Figure size 640x480 with 0 Axes>

In [183]: dd.plot.bar(colormap='Greens')
Out[183]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65d0a5de10>
```

![greens](/static/images/greens.png)

Parallel coordinates charts:

``` python
In [184]: plt.figure()
Out[184]: <Figure size 640x480 with 0 Axes>

In [185]: parallel_coordinates(data, 'Name', colormap='gist_rainbow')
Out[185]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65d08e5eb8>
```

![parallel_gist_rainbow](/static/images/parallel_gist_rainbow.png)

Andrews curves charts:

``` python
In [186]: plt.figure()
Out[186]: <Figure size 640x480 with 0 Axes>

In [187]: andrews_curves(data, 'Name', colormap='winter')
Out[187]: <matplotlib.axes._subplots.AxesSubplot at 0x7f65d06b6518>
```

![andrews_curve_winter](/static/images/andrews_curve_winter.png)

## Plotting directly with matplotlib

In some situations it may still be preferable or necessary to prepare plots
directly with matplotlib, for instance when a certain type of plot or
customization is not (yet) supported by pandas. ``Series`` and ``DataFrame``
objects behave like arrays and can therefore be passed directly to
matplotlib functions without explicit casts.

pandas also automatically registers formatters and locators that recognize date
indices, thereby extending date and time support to practically all plot types
available in matplotlib. Although this formatting does not provide the same
level of refinement you would get when plotting via pandas, it can be faster
when plotting a large number of points.

``` python
In [188]: price = pd.Series(np.random.randn(150).cumsum(),
   .....:                   index=pd.date_range('2000-1-1', periods=150, freq='B'))
   .....: 

In [189]: ma = price.rolling(20).mean()

In [190]: mstd = price.rolling(20).std()

In [191]: plt.figure()
Out[191]: <Figure size 640x480 with 0 Axes>

In [192]: plt.plot(price.index, price, 'k')
Out[192]: [<matplotlib.lines.Line2D at 0x7f65da5f8710>]

In [193]: plt.plot(ma.index, ma, 'b')
Out[193]: [<matplotlib.lines.Line2D at 0x7f65d9ab9518>]

In [194]: plt.fill_between(mstd.index, ma - 2 * mstd, ma + 2 * mstd,
   .....:                  color='b', alpha=0.2)
   .....: 
Out[194]: <matplotlib.collections.PolyCollection at 0x7f65d9ab9128>
```

![bollinger](/static/images/bollinger.png)

## Trellis plotting interface

::: danger Warning

The ``rplot`` trellis plotting interface has been **removed**. Please use
external packages like [seaborn](https://github.com/mwaskom/seaborn) for
similar but more refined functionality and refer to our 0.18.1 documentation
[here](http://pandas.pydata.org/pandas-docs/version/0.18.1/visualization.html)
for how to convert to using it.

:::
