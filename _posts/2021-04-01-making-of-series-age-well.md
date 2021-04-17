---
layout: post
title: "The Making of 'TV Series that Age Well'"
date: 2021-04-01 10:38:55 +0100
categories: imdb tvseries sqlite powerbi
---
Although it isn't widely known, it has been possible to download the core information of [IMDb](https://www.imdb.com/) even before the web was born, even before IMDb became a website (yeah, it's that old!), even after being purchased by [Amazon](https://www.amazon.com/)... If you have Python installed, you can use a utility that automates the entire process and outputs a single SQLite file:

```
pip install imdb-sqlite
imdb-sqlite
```

Beware that many minutes can pass before all the data is downloaded! But once everything is available locally, you can start playing with the database immediately! Let us say we wish to know which are the 10 best series that have already aired?

```sql
SELECT r.title_id, r.rating, r.votes, t.primary_title, t.premiered, t.ended
FROM ratings r
    INNER JOIN titles t
        ON r.title_id = t.title_id
WHERE t.type IN ('tvSeries', 'tvMiniSeries')
    AND r.votes > 100000 -- At least somewhat popular
    AND t.ended IS NOT NULL -- Only series that have finished
ORDER BY r.rating DESC
    , r.votes DESC
LIMIT 50;
```

The first rows returned are:

title_id | rating | votes | primary_title | premiered | ended
-------- | ------ | ----- | ------------- | --------- | -----
tt0903747 | 9.5 | 1486251 | Breaking Bad | 2008 | 2013
tt7366338 | 9.4 | 555775 | Chernobyl | 2019 | 2019
tt0185906 | 9.4 | 380863 | Band of Brothers | 2001 | 2001
tt0795176 | 9.4 | 168531 | Planet Earth | 2006 | 2006
tt0944947 | 9.3 | 1788628 | Game of Thrones | 2011 | 2019
tt0306414 | 9.3 | 288748 | The Wire | 2002 | 2008
tt2395695 | 9.3 | 111913 | Cosmos | 2014 | 2014
tt0141842 | 9.2 | 307075 | The Sopranos | 1999 | 2007
tt0417299 | 9.2 | 252438 | Avatar: The Last Airbender | 2005 | 2008
tt1475582 | 9.1 | 815308 | Sherlock | 2010 | 2017

How do the ratings for _Breaking Bad_ hold as the seasons pass?

```sql
SELECT e.season_number, AVG(r.rating)
FROM episodes e
    INNER JOIN ratings r
        ON e.episode_title_id = r.title_id
WHERE e.show_title_id = 'tt0903747' -- Breaking Bad
GROUP BY e.season_number
ORDER BY e.season_number;
```

The results show consistent quality:

season_number | AVG(r.rating)
------------- | -------------
1 | 8.78571428571429
2 | 8.86153846153846
3 | 8.80769230769231
4 | 9.02307692307693
5 | 9.43125

And what about _Game of Thrones_? Well, the last season is clearly a disappointment!

season_number | AVG(r.rating)
------------- | -------------
1 | 9.1
2 | 8.96
3 | 9.05
4 | 9.31
5 | 8.83
6 | 9.06
7 | 9.1
8 | 6.33333333333333

Now we can combine both these queries using a CTE:

```sql
WITH best_tvseries
AS (
    SELECT r.title_id, r.rating, r.votes, t.primary_title, t.premiered, t.ended
    FROM ratings r
        INNER JOIN titles t
            ON r.title_id = t.title_id
    WHERE t.type IN ('tvSeries', 'tvMiniSeries')
        AND r.votes > 100000 -- At least somewhat popular
        AND t.ended IS NOT NULL -- Only series that have finished
    ORDER BY r.rating DESC, r.votes DESC
    LIMIT 50
)
SELECT b.title_id, b.rating, b.votes, b.primary_title, b.premiered, b.ended, e.season_number, AVG(r.rating)
FROM best_tvseries b
    INNER JOIN episodes e
        ON b.title_id = e.show_title_id
    INNER JOIN ratings r
        ON e.episode_title_id = r.title_id
GROUP BY b.title_id, e.season_number
ORDER BY b.rating DESC, b.votes DESC, b.title_id, e.season_number;
```

So at least both data can now be seen simultaneously:

title_id | rating | votes | primary_title | premiered | ended | season_number |  AVG(r.rating)
-------- | ------ | ----- | ------------- | --------- | ----- | ------------- | --------------
tt0903747 | 9.5 | 1486251 | Breaking Bad | 2008 | 2013 | 1 | 8.78571428571429
tt0903747 | 9.5 | 1486251 | Breaking Bad | 2008 | 2013 | 2 | 8.86153846153846
tt0903747 | 9.5 | 1486251 | Breaking Bad | 2008 | 2013 | 3 | 8.80769230769231
tt0903747 | 9.5 | 1486251 | Breaking Bad | 2008 | 2013 | 4 | 9.02307692307693
tt0903747 | 9.5 | 1486251 | Breaking Bad | 2008 | 2013 | 5 | 9.43125
tt7366338 | 9.4 | 555775 | Chernobyl | 2019 | 2019 | 1 | 9.62
tt0185906 | 9.4 | 380863 | Band of Brothers | 2001 | 2001 | 1 | 9.08
tt0795176 | 9.4 | 168531 | Planet Earth | 2006 | 2006 | 1 | 8.78181818181818
tt0944947 | 9.3 | 1788628 | Game of Thrones | 2011 | 2019 | 1 | 9.1
tt0944947 | 9.3 | 1788628 | Game of Thrones | 2011 | 2019 | 2 | 8.96
tt0944947 | 9.3 | 1788628 | Game of Thrones | 2011 | 2019 | 3 | 9.05
tt0944947 | 9.3 | 1788628 | Game of Thrones | 2011 | 2019 | 4 | 9.31
tt0944947 | 9.3 | 1788628 | Game of Thrones | 2011 | 2019 | 5 | 8.83
tt0944947 | 9.3 | 1788628 | Game of Thrones | 2011 | 2019 | 6 | 9.06
tt0944947 | 9.3 | 1788628 | Game of Thrones | 2011 | 2019 | 7 | 9.1
tt0944947 | 9.3 | 1788628 | Game of Thrones | 2011 | 2019 | 8 | 6.33333333333333
tt0306414 | 9.3 | 288748 | The Wire | 2002 | 2008 | 1 | 8.62307692307692
tt0306414 | 9.3 | 288748 | The Wire | 2002 | 2008 | 2 | 8.53333333333333
tt0306414 | 9.3 | 288748 | The Wire | 2002 | 2008 | 3 | 8.75833333333333
tt0306414 | 9.3 | 288748 | The Wire | 2002 | 2008 | 4 | 8.78461538461538
tt0306414 | 9.3 | 288748 | The Wire | 2002 | 2008 | 5 | 8.76
tt2395695 | 9.3 | 111913 | Cosmos | 2014 | 2014 | 1 | 9.11538461538462
tt0141842 | 9.2 | 307075 | The Sopranos | 1999 | 2007 | 1 | 8.69230769230769
tt0141842 | 9.2 | 307075 | The Sopranos | 1999 | 2007 | 2 | 8.67692307692308

To visualize all this, Power BI to the rescue, using an appropriate [ODBC driver](http://www.ch-werner.de/sqliteodbc/)!

![Connecting Power BI to SQLite](/assets/2021-04-01-making-of-series-age-well_imdb-sqlite-powerbi.png)

The ratings were multiplied by 10 because, somehow, the decimals were truncated (ODBC driver bug?) and are available as "rating100".

![Configuration of Power BI visualization](/assets/2021-04-01-making-of-series-age-well_imdb-sqlite-powerbi-config.png)

The visualization is basically a **Matrix** component (circled in red) and its configuration consisted of dragging "rating)" to **Values**, "RTitle" (which is "rating100" concatenated with "primary_title") to **Rows** and "season_number" to **Columns**, as the red arrows show.

![Configuration of Power BI visualization](/assets/2021-04-01-making-of-series-age-well_imdb-sqlite-powerbi-condfmt.png)

Finally, the colors are set by specifying a Conditional Formatting; only the **Minimum** had to be customized to 6 instead of the default as the last season of _House of Cards_ really skewed the results and didn't allow any meaningful analysis for the other TV series.

Although this effort was inspired on a [Reddit post](https://www.reddit.com/r/dataisbeautiful/comments/f3drhm/oc_movie_trilogy_ranks_based_off_online_ratings/) on movie trilogies, which uses bars, they don't work very well with so many cells. Instead, background colors provide a better visual feedback.