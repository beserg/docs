# Window functions in {{ datalens-short-name }}

[Window functions](../function-ref/window-functions.md) are similar to aggregate functions. They allow you to get additional information about the original sample. For example, you can calculate the cumulative total and the moving average or rank values.

The difference is that when calculating window functions, the rows are not combined into one but continue to be separate. The result of the calculation is displayed in each row. The original number of rows doesn't change. For more information about how data aggregation and grouping work in {{ datalens-short-name }}, see [{#T}](aggregation-tutorial.md#datalens-aggregation).

In these examples, [Selling.csv](https://disk.yandex.com/d/pT2f56G7bNaFvg?lang=en) is the data source for sales in cities.

## Applying window functions {#usage-window-function}

In {{ datalens-short-name }}, only [measures](dataset/data-model.md#field) can be the arguments of window functions. The groups of values that a function is calculated for are specified as a list of [dimensions](dataset/data-model.md#field) and are called windows. For [groupings](#grouping), only the dimensions used in the built chart can be applied. These include all the dimensions in one chart section.

Let's take a look at the `Selling` table with data on sales in cities:

| # | City | Category | Date | Sales | Profit | Day's discount |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | Detroit | Office Supplies | 2014-01-02 | 10 | 7 | 0.05 |
| 2 | Portland | Office Supplies | 2014-04-05 | 14 | 10 | 0.00 |
| 3 | Portland | Office Supplies | 2014-01-21 | 20 | 12 | 0.20 |
| 4 | San Francisco | Office Supplies | 2014-03-11 | 8 | 3 | 0.10 |
| 5 | Detroit | Furniture | 2014-01-01 | 12 | 3 | 0.00 |
| 6 | Portland | Furniture | 2014-01-21 | 7 | 2 | 0.05 |
| 7 | San Francisco | Technology | 2014-01-02 | 7 | 3 | 0.10 |
| 8 | San Francisco | Technology | 2014-01-17 | 13 | 5 | 0.20 |

**Example 1**

In the [chart](https://datalens.yandex/ryw9h5g0ecc8k) based on the `Selling` table with grouping by `City` and `Category`, you need to calculate the total sales amount (`TotalSales`) and each category's share of the total amount (`% Total`) for the city. To do this, you need to create two measures using the [SUM](../function-ref/SUM_WINDOW.md) window function:

* TotalSales: `SUM(SUM([Sales]) TOTAL)`
* % Total: `SUM([Sales]) / [TotalSales]`

For example, for the **Table** chart, the result looks like this:

![image](../../_assets/datalens/concepts/tutorial/window-func-1.png)

**Example 2**

You need to arrange the rows in the `Selling` table based on the sales amount. To do this, you can use the [RANK](../function-ref/RANK.md) window function `RANK(SUM([Sales]))`. As a result, each row is assigned a sequence number: the row with the largest sales amount is 1, and the smallest is 6.

![image](../../_assets/datalens/concepts/tutorial/window-func-2.png)

Window functions can be nested. You can also specify a custom grouping for each function used in the formula.

**Example**

You need to arrange the rows in the `Selling` table based on the the average sales amount for all dates in the city. The average sales amount for the city can be calculated using the [AVG](../function-ref/AVG_WINDOW.md) function: `AVG(SUM([Sales]) WITHIN [City])`. City names repeat in the table, so for ranking, use the [RANK_DENSE](../function-ref/RANK_DENSE.md) function. It doesn't skip sequence numbers for rows with the same value. The resulting formula is `RANK_DENSE(AVG(SUM([Sales]) WITHIN [City]) TOTAL)`.

![image](../../_assets/datalens/concepts/tutorial/window-func-11.png)

## Grouping in window functions {#grouping}

Just like aggregate functions, window functions can be calculated:

* For a [single window](#one-window-grouping).
* For [multiple windows](#some-window-grouping).

For more information about grouping in window functions, see [{#T}](../function-ref/window-functions.md#syntax-grouping).

### Grouping for a single window {#one-window-grouping}

With this grouping option, the function is calculated for a single window that includes all the rows. The `TOTAL` grouping is used. It enables you to calculate totals, rank rows, and perform other operations that require information about all the source data.

**Example**

You need to calculate the average sales amount (`AvgSales`) and deviations from it for each category in the city (`DeltaFromAvg`). The best function for this is [AVG](../function-ref/AVG_WINDOW.md):

* AvgSales: `AVG(SUM([Sales]) TOTAL)`.
* DeltaFromAvg: `SUM([Sales]) - [AvgSales]`.

![image](../../_assets/datalens/concepts/tutorial/window-func-3.png)

### Grouping for multiple windows {#some-window-grouping}

Sometimes the window function needs to be calculated separately by group, and not across all records. In this case, the `WITHIN` and `AMONG` groupings are used.

#### WITHIN {#within}

`WITHIN`: Similar to `GROUP BY` in `SQL`. It lists all the dimensions by which splitting into windows is performed. In `WITHIN`, you can also use measures. In this case their values are similarly included in the window grouping.

{% note warning %}

In `WITHIN`, the [dimensions](aggregation-tutorial.md#dimensions-and-measures) that aren't included in chart grouping are ignored. For example, in a chart grouped by the `City` and `Category` dimensions for the `SUM(SUM([Sales]) WITHIN [Date])` measure, the `Date` dimension is ignored and it becomes the same as the `SUM(SUM([Sales]) TOTAL)` measure.

{% endnote %}

**Example**

Calculating the share of each category (`% Total`) of the total sales amount by city (`TotalSales`):

* TotalSales: `SUM(SUM([Sales]) WITHIN [City])`.
* % Total: `SUM([Sales]) / [TotalSales]`.

For example, this is the result for the **Column chart**:

![image](../../_assets/datalens/concepts/tutorial/window-func-4.png)

#### AMONG {#among}

In this case, splitting into windows is performed for all dimensions that are included in the chart grouping but are not listed in `AMONG`. That's why this grouping type is contrary to `WITHIN`. When calculating the function, `AMONG` transforms to `WITHIN`, which performs grouping by all dimensions that are not listed in `AMONG`.

For example, for a chart with grouping by the `City` and `Category` dimensions, the following measures are the same:

* `SUM(SUM([Sales]) AMONG [Category])` and `SUM(SUM([Sales]) WITHIN [City])`.
* `SUM(SUM([Sales]) AMONG [City], [Category])`and `SUM(SUM([Sales]) TOTAL)`.

This option is provided only for convenience and is used when you don't know which dimensions the chart will be built across in advance, but you need to exclude certain dimensions from the window grouping.

{% note warning %}

The dimensions listed in `AMONG` should be added to the chart sections. Otherwise, the chart returns an error.

{% endnote %}

## Sorting {#order-by}

Some window functions support [sorting](../function-ref/window-functions.md#syntax-order-by), the direction of which affects the calculation value. To specify sorting for the window function:

* Specify dimensions or measures in the `ORDER BY` section.
* In the chart, move the dimensions or measures to the **Sorting** section.

Dimensions and measures for sorting are first taken from the `ORDER BY` section in the formula and then from the **Sorting** chart section.

**Example**

You need to calculate the change in the total sales amount (`IncTotal`) for the entire period, from the earliest to the latest date. To do this, you can use the [RSUM](../function-ref/RSUM.md) function sorted by the `Date` dimension: `RSUM(SUM([Sales]) TOTAL ORDER BY [Date])`.

Result for an example **Line chart**:

![image](../../_assets/datalens/concepts/tutorial/window-func-5.png)

You'll get a similar result if you set the `IncTotal` measure with the `RSUM(SUM([Sales]) TOTAL)` formula and add the `Date` dimension to the **Sorting** section.

## Filtering {#before-filter-by}

Function values in charts are calculated after applying [filters](chart/settings.md#filter) across the dimensions and measures added to the **Filters** section. For window functions, you can override this order. To do this, specify the necessary dimensions or measures in the `BEFORE FILTER BY` section of the formula. In this case, the function value is calculated before filtering is applied.

The calculation order is changed when you need to calculate the function value for the original dataset but the chart data is limited by the filter.

**Example**

You need to calculate the change in the total sales amount (`IncTotal`) for the period from `17.01.2014` through `11.03.2014`. If you add a `Date` filter and create the `RSUM(SUM([Sales]) TOTAL ORDER BY [Date])` measure, the function is calculated only for the data limited by the filter:

![image](../../_assets/datalens/concepts/tutorial/window-func-6.png)

To calculate a function for all the data, but only display the result for a certain period, you need to add the `Date` dimension to the `BEFORE FILTER BY` section: `RSUM(SUM([Sales]) TOTAL ORDER BY [Date] BEFORE FILTER BY [Date])`.

![image](../../_assets/datalens/concepts/tutorial/window-func-7.png)

## Questions and answers {#qa}

{% cut "How do I arrange values when calculating cumulative totals or moving averages?" %}

   For functions that depend on the order of entries in the window (for example, [RSUM](../function-ref/RSUM.md), [MAVG](../function-ref/MAVG.md), [LAG](../function-ref/LAG.md), [LAST](../function-ref/LAST.md), or [FIRST](../function-ref/FIRST.md)) to work correctly, you must specify sorting. To do this:

   * Drag the dimension or measure to sort the chart by to the **Sorting** section.
   * Set sorting for a specific function using `ORDER BY`.

{% endcut %}

{% cut "How do I properly calculate cumulative totals after adding a field to the Color section?" %}

As an example, let's look at a line chart that shows changes in the total sales amount by date (see the [Selling](#usage-window-function) table). The cumulative total (`IncTotal`) is calculated using the [RSUM](../function-ref/RSUM.md) window function: `RSUM(SUM([Sales]))`.

![image](../../_assets/datalens/concepts/tutorial/window-func-8.png)

To display the change in the sales amount for each product category, add the `Category` dimension to the **Colors** section.

![image](../../_assets/datalens/concepts/tutorial/window-func-9.png)

After that, the chart displays a separate graph for each category but the totals are calculated incorrectly: for `Furniture`, it's 49 instead of 19, for `Office Supplies`, 91 instead of 52, for `Technology`, 42 instead of 20. This is because the dimension in the **Colors** (`Category`) section is included in the grouping the same way as the dimension in the **X** section (`Date`). To calculate the amount correctly, you need to add the `Category` dimension to the `WITHIN` section or the `Date` dimension to the `AMONG` section: `RSUM(SUM([Sales]) WITHIN [Category])` or `RSUM(SUM([Sales]) AMONG [Date])`.

![image](../../_assets/datalens/concepts/tutorial/window-func-10.png)

{% endcut %}

{% cut "How do I properly calculate a window function if a grouping for dates is given in the chart?" %}

When adding a grouping (rounding) for a date in the chart, the original field is replaced with an automatically generated one. For example, when rounding to a month, the `[Date]` dimension is replaced with a new field using the `DATETRUNC([Date], "month")` formula. Because the original `[Date]` field disappears from the list of chart dimensions, the window function it's used in no longer works. For the function to work correctly, you need to round the original `[Date]` dimension in the formula using the [DATETRUNC](../function-ref/DATETRUNC.md) function.

{% endcut %}
